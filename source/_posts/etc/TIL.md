---
layout: post
title: Today I Learned (TIL)
date: 2017-04-04 00:00:00 +0900
cover: /covers/profile.jpg
disqusId: 8f348edd3784df3c9e668f952af8e7dc934358f0
toc: true
category: 기타
---

개인적인 TIL을 모아두는 글입니다.

<!-- more -->

# 2021-08-18

query parameter가 있으면 proxy_pass로 보내고 없으면 바로 파일을 서빙

```nginx
location ~* \.(jpe?g|png)$ {
  auth_request /auth;
  proxy_set_header 'X-Forwarded-For' $remote_addr;
  proxy_set_header 'Host' $http_host;
  if ($args) {
    proxy_pass http://localhost:9001;
  }
  if ($args = '') {
    root /archive;
    add_header Access-Control-Allow-Origin 'https://video.hyunsub.kim';
    add_header Access-Control-Allow-Credentials 'true';
  }
}
```

# 2021-08-17

- 모바일 사파리에서는 전화번호같이 생긴 문자열에 대해서는 자동으로 링크를 생성하는데 아래의 meta tag로 이를 방지할 수 있다.    
`<meta name="format-detection" content="telephone=no"/>`

- 사용자가 링크를 눌렀을 때 해당 링크로 넘어가기 전에 무언가의 작업을 하고 싶다면 `window.addEventListener('pagehide', () => {})` 를 추가하면 된다.

# 2021-08-13

https://apps.timwhitlock.info/unicode/inspect?s=%E6%A8%82+%EF%A5%9C
똑같이 생긴 한자인데 유니코드가 다르다

# 2021-07-30

- scp로 큰 파일을 보내려고 할 때 `client_loop: send disconnect: Broken pipe` 라는 에러가 발생하는 경우가 있는데 이는 파일을 받는 서버가 throughput을 감당하지 못 해서 그렇다.
`-l 2000` 옵션을 붙여주면 해결된다. 여기서 `2000`은 2000kbps 속도로 보낸다는 의미이다.    
ref: <https://stackoverflow.com/questions/30020519/broken-pipe-error-on-scp>

- ubuntu에서 adoptopenjdk를 설치하고 jenv 에 추가하는 법    
`sudo apt install adoptopenjdk-11-hotspot`    
`jenv add /usr/lib/jvm/adoptopenjdk-11-hotspot-amd64`    
ref: <https://blog.benelog.net/installing-jdk.html#jenv>    

# 2021-07-18

@Aspect 어노테이션이 붙은 클래스는 상속할 수 없다. `AopConfigException: cannot extend concrete aspect` 에러가 발생한다.

# 2021-07-15

`ClassPathResource.getFile`은 IDE에서 실행하면 잘 되지만 jar 파일로 실행하면 잘 되지 않는다. 파일시스템에서 해당 파일을 찾기 때문이다.
대신 `ClassPathResource.getInputStream` 을 써야한다.

# 2021-06-21

linux에서 특정 명령어만 비밀번호 입력 없이 sudo 권한을 사용할 수 있게 하는 방법

1. `sudo visudo` 명령어를 입력하면 nano 편집기가 열린다.
2. `%sudo`로 시작하는 라인 아래에 다음과 같이 적는다

`[username] ALL=NOPASSWD: [command]`    
예) `hyunsub ALL=NOPASSWD: /usr/sbin/nginx -s reload`

여기서 `command`는 절대경로로 적어야 한다.    
여기서 `[username]`과 `ALL` 사이는 탭 문자를 써야 한다.

3. 저장하고 나가면 적용되어 있다.

# 2021-06-05

ssh 명령어로 remote server에 background process를 실행시키려고 하는 경우

`> nohup.out < /dev/null`을 붙여주어야 한다.

그렇지 않으면 ssh 세션이 끊어지지 않고 계속 유지된다.

또는 ssh 명령어에 `-f` 옵션을 붙여주면 된다.

# 2021-05-23

nginx binary를 그냥 실행히시키면 no daemon 모드로 실행이 되는데 이를 daemon 모드로 실행시키려고 하면    
`nginx -c nginx.conf -g 'daemon off;'`

# 2021-05-11

Java에서 삼항연산자 사용시, 두번째, 세번째 피연산자 중 한 쪽은 Wrapper Class이고 다른 한 쪽은 원시 자료형일 경우,
무조건 Wrapper Class를 원시 자료형으로 변환한다.

예를 들어 아래의 코드는 무조건 NullPointerException이 발생한다.

```java
Integer A = null;
Integer B = null;
Integer chosenInteger = A != null ? A.intValue() : B;  
```

여기서 두번째 피연산자의 타입은 int(원시자료형)이고, 세번째 피연산자의 타입은 Integer(Wrapper Class)이다.
따라서 B를 원시자료형으로 변환하려고 하고 이 때 NullPointerException이 발생하는 것이다.

Ref: https://stackoverflow.com/a/15082983

# 2021-05-04
nginx 에서는 dollar sign 을 escape 할 수 있는 방법이 없다.
따라서 아래와 같이 써야 한다.

```conf
// http block 에서
geo $dollar {
  default "$";
}

location / {
  return 200 '$dollar.status_code';
}
```

# 2021-04 13

safari 에서는 이미지의 높이를 auto 로 하려면 align-self: flex-start 를 해야 한다.

# 2021-03-19

nginx에서 header에 underscore가 있으면 underscores_in_headers on; 해줘야한다.

# 2021-02-14

nginx proxy_pass로 socket.io를 동작하고 싶을 때
```
location /socket.io {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_buffering off;
    proxy_pass http://localhost:8003;
}
```
