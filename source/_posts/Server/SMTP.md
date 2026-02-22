---
layout: post
title: Docker로 SMTP 서버 직접 구축하기
date: 2026-02-22 00:00:00 +0900
cover: /covers/docker.png
disqusId: 3a7f2c1e9b4d8e6a0f5c3b2d1a9e8f7c4b6d2e1a
toc: true
category: Server
tags:
- smtp
- docker
- postfix
- dkim
---

외부 릴레이(Gmail SMTP 등) 없이 직접 수신 메일 서버에 전달하는 SMTP 서버를 Docker로 구축하는 법을 알아보겠습니다.
`boky/postfix` 이미지를 사용하며, SSL 인증서(Let's Encrypt)가 이미 발급되어 있다고 가정합니다.

<!-- more -->

## 전제 조건

| 항목 | 비고 |
|---|---|
| 공인 IP | ISP에서 포트 25 아웃바운드 차단 여부 확인 필요 (`nc -zv smtp.gmail.com 25`) |
| SSL 인증서 | Let's Encrypt 등. `/etc/letsencrypt/live/DOMAIN/` 위치 가정 |
| 도메인 | DNS 레코드 직접 편집 가능해야 함 |
| Docker | 설치되어 있어야 함 |

# 1. DKIM 키 생성

```bash
mkdir -p /path/to/postfix/opendkim/keys/your.domain

# 컨테이너를 현재 유저(uid 1000)로 실행해야 파일 소유자가 root가 되지 않음
docker run --rm \
  -u $(id -u):$(id -g) \
  -v /path/to/postfix/opendkim/keys/your.domain:/keys \
  --entrypoint opendkim-genkey \
  instrumentisto/opendkim \
  -b 2048 -d your.domain -s mail -D /keys/
```

생성 결과:
- `mail.private` — 서명 키 (컨테이너가 사용)
- `mail.txt` — DNS에 등록할 공개키

## flat 키 파일도 함께 생성

`boky/postfix`는 DKIM 키를 `/etc/opendkim/keys/DOMAIN.private` 경로(flat 파일)로 탐색한다.
서브디렉터리 구조(`hyunsub.kim/mail.private`)를 인식하지 못하므로, 아래와 같이 복사본을 만들어둔다.

```bash
cp /path/to/postfix/opendkim/keys/your.domain/mail.private \
   /path/to/postfix/opendkim/keys/your.domain.private
```

# 2. OpenDKIM 설정 파일 생성

`boky/postfix`가 시작 시 이 파일들에 내용을 **추가(append)** 한다.
빈 파일로 시작해도 되고, 아래처럼 기본값을 미리 채워도 된다.

### `opendkim/KeyTable`
```
mail._domainkey.your.domain  your.domain:mail:/etc/opendkim/keys/your.domain.private
```

### `opendkim/SigningTable`
```
*@your.domain  mail._domainkey.your.domain
```

### `opendkim/TrustedHosts`
```
127.0.0.1
localhost
172.16.0.0/12
192.168.0.0/16
```

> **주의**: 세 파일 모두 컨테이너 시작 시 스크립트가 내용을 추가하므로 볼륨 마운트를 `:ro`로 걸면 안 된다.

# 3. docker-compose.yml

```yaml
  postfix:
    image: boky/postfix
    container_name: postfix
    restart: always
    hostname: mail.your.domain
    ports:
      - "127.0.0.1:587:587"   # 외부에서 접근 불가, 로컬 앱 전용
    environment:
      - HOSTNAME=mail.your.domain
      - ALLOWED_SENDER_DOMAINS=your.domain
      - POSTFIX_mynetworks=127.0.0.0/8 172.16.0.0/12 192.168.0.0/16
      - POSTFIX_smtp_tls_security_level=may
      - POSTFIX_smtp_tls_cert_file=/certs/live/your.domain/fullchain.pem
      - POSTFIX_smtp_tls_key_file=/certs/live/your.domain/privkey.pem
      - POSTFIX_smtpd_tls_cert_file=/certs/live/your.domain/fullchain.pem
      - POSTFIX_smtpd_tls_key_file=/certs/live/your.domain/privkey.pem
      - POSTFIX_smtpd_tls_security_level=may
    volumes:
      - /etc/letsencrypt:/certs:ro   # live/ 내 파일이 ../../archive/ 심볼릭링크이므로 상위 디렉터리 전체 마운트
      - ./postfix/opendkim/keys:/etc/opendkim/keys          # :ro 금지 (chown 필요)
      - ./postfix/opendkim/KeyTable:/etc/opendkim/KeyTable  # :ro 금지 (startup 시 append)
      - ./postfix/opendkim/SigningTable:/etc/opendkim/SigningTable
      - ./postfix/opendkim/TrustedHosts:/etc/opendkim/TrustedHosts
```

## 주의사항 — `DKIM_AUTOGENERATE`

`boky/postfix`의 스크립트는 이 변수를 bash `-n` 조건으로 체크한다.

```bash
if [ -n "$DKIM_AUTOGENERATE" ]; then  # 값이 "false"여도 non-empty이므로 true!
```

따라서 `DKIM_AUTOGENERATE=false`로 설정하면 **자동생성이 비활성화되지 않는다**.
비활성화하려면 **변수 자체를 아예 선언하지 않아야** 한다.

# 4. DNS 레코드 등록

| 타입 | 이름 | 값 |
|---|---|---|
| A | `mail` | 서버 공인 IP |
| MX | `@` | `mail.your.domain.` (우선순위 10, 끝에 `.` 필수) |
| TXT | `@` | `v=spf1 a:mail.your.domain ~all` |
| TXT | `mail._domainkey` | `mail.txt` 파일 내용 (아래 참고) |
| TXT | `_dmarc` | `v=DMARC1; p=none; rua=mailto:admin@your.domain` |

## DKIM 공개키 (TXT 레코드) 포맷

DNS 레지스트라에 따라 TXT 값이 250자 이하인 문자열만 허용하는 경우가 있다.
2048비트 키는 전체 길이가 400자 이상이므로 반드시 **multi-line(따옴표로 분리)** 으로 입력해야 한다.

`mail.txt` 파일 내용에서 따옴표와 공백을 정리해 하나의 문자열로 합친 뒤, 250자 단위로 나눈다:

```python
full = 'v=DKIM1; k=rsa; p=<공개키>'
parts = [full[i:i+250] for i in range(0, len(full), 250)]
print(' '.join(f'"{p}"' for p in parts))
```

출력 예시 (DNS 레지스트라 입력란에 그대로 붙여넣기):
```
"v=DKIM1; k=rsa; p=...첫번째 250자..." "...나머지..."
```

## PTR 레코드 (역방향 DNS)

ISP 또는 호스팅사에 `공인IP → mail.your.domain` PTR 레코드 설정을 요청해야 한다.
없으면 Gmail 등 주요 메일 서버에서 스팸으로 분류될 수 있다.

# 5. 컨테이너 시작 및 검증

```bash
# 포트 25 아웃바운드 확인 (차단 시 직접 발송 불가)
nc -zv smtp.gmail.com 25

# 컨테이너 기동
docker compose up -d postfix
docker logs postfix

# 정상 기동 확인 로그 키워드
# ✅ DKIM-Signature field added (s=mail, d=your.domain)
# ✅ supervisord: opendkim/postfix entered RUNNING state

# 테스트 메일 발송
swaks --to test@gmail.com --from noreply@your.domain \
      --server localhost:587 --tls-optional \
      --header "Subject: DKIM test"

# 메일 발송 로그 확인
docker logs postfix 2>&1 | grep -E "(DKIM|status=sent|status=bounced)"
```

# 6. Spring Boot 연동

`application-prod.yml`:

```yaml
spring:
  mail:
    host: localhost
    port: 587
    properties:
      mail:
        smtp:
          starttls:
            enable: true
            required: false
          auth: false        # localhost 전용이라 인증 불필요
          connectiontimeout: 5000
          timeout: 10000
          writetimeout: 10000
```

# 7. 인증서 갱신 시 자동 reload

`/etc/letsencrypt/renewal-hooks/post/reload-postfix.sh`:

```bash
#!/bin/bash
docker exec postfix postfix reload
```

```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/post/reload-postfix.sh
```

# 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| 컨테이너가 재시작 반복 | `DKIM_AUTOGENERATE=false`로 설정했는데 자동생성 시도 → read-only 마운트에 쓰기 실패 | `DKIM_AUTOGENERATE` 변수 자체를 제거 |
| `TrustedHosts: Read-only file system` | startup 스크립트가 파일에 append 시도 | KeyTable, SigningTable, TrustedHosts 마운트에서 `:ro` 제거 |
| `Skipping DKIM for domain` + KeyTable 비어있음 | boky/postfix는 `/etc/opendkim/keys/DOMAIN.private` flat 경로만 탐색 | `keys/your.domain/mail.private`를 `keys/your.domain.private`로 복사 |
| `key data is not secure` WARNING | 키 파일 권한이 opendkim 사용자 관점에서 느슨함 | 기능 무관, 무시해도 됨 |
| `/etc/letsencrypt/live/` 인증서 읽기 실패 | `live/` 내 파일이 `../../archive/`를 가리키는 심볼릭링크 | `/etc/letsencrypt` 전체를 마운트 (하위 디렉터리만 마운트하면 링크 깨짐) |

---

> 이 글은 [Claude Code](https://claude.ai/claude-code) (Model: **Claude Sonnet 4.6** / `claude-sonnet-4-6`)가 생성했습니다.
