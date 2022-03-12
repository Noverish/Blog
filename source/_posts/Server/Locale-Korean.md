---
layout: post
title: 달러 기호($)와 함께 세자리수로 한글이 깨지는 경우
date: 2022-03-12 11:52:00 +0900
cover: /covers/bash.jpg
disqusId: 34ce6bcdfdf2137eb342ff2084032bb02b5348ac
toc: true
category: Server
tags:
- charset
---

`''$'\354\235\264\353\240\245\354\204\234''`와 같이 달러 기호($)와 함께 세자리수로 한글이 깨지는 경우 해결 방법을 알아보겠습니다.
(feat. locale 설정)

<!-- more -->

![깨져있는 한글 예시](./broken-korean.png)

# 1. 현재 설정되어 있는 locale 보기

`locale` 명령어를 입력하여 현재 설정되어 있는 locale 볼 수 있습니다.

```shell
$ locale
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

아마 위와 같이 나올 것입니다. 지금 locale 설정이 `en_US.UTF-8`로 설정되어 있는 것을 알 수 있습니다. 이 때문에 한글이 깨지는 것입니다.

# 2. 사용 가능한 locale 목록 보기

`locale -a` 명령어를 통해 사용 가능한 locale 목록을 볼 수 있습니다.

```
$ locale -a
C
C.UTF-8
en_US
en_US.utf8
...
POSIX
```

여기서 `ko_KR.UTF-8`이 있나 확인 합니다. 만약 존재하지 않다면 다음 명령어를 통해 추가합니다.

> $ locale-gen ko_KR.UTF-8

<article class="message message-immersive is-primary">
  <div class="message-body">
    <i class="fas fa-info-circle mr-2"></i>
    설정 가능한 locale 목록은 <code>/usr/share/i18n/SUPPORTED</code> 파일에서 확인할 수 있습니다.
  </div>
</article>

# 3. Locale charset을 한글로 바꾸기

다음 명령어를 통해 Locale charset을 한글로 바꿉니다.

> $ sudo update-locale LC_CTYPE=ko_KR.UTF-8

그 다음에 ssh 연걸을 끊었다가 다시 연결하면 다음과 같이 한글이 잘 나타나는 것을 알 수 있습니다.

![잘 표시되는 한글 예시](./fixed-korean.png)

[Reference](https://www.thomas-krenn.com/en/wiki/Configure_Locales_in_Ubuntu)

# 부록. 다양한 LC_* locale 환경 변수들

우리는 위에서 `LC_CTYPE` locale 환경 변수만 변경했습니다.
다른 값들도 변경하면 다양한 방면으로 시스템에 영향을 미칩니다.

예를 들어 `LC_TIME=ko_KR.UTF-8`로 변경하면 `date` 명령어의 결과 값이 다음과 같이 달라집니다.


```shell shell
# LC_TIME=en_US.UTF-8 인 경우
$ date
Sat Mar 12 12:24:41 KST 2022

# LC_TIME=ko_KR.UTF-8 인 경우
$ date
2022. 03. 12. (토) 12:28:30 KST
```

아래와 같이 locale 환경 변수는 다양한 kernel 함수에 영향을 미칩니다.

- LC_TIME : 시간과 날짜의 표현(년, 월, 일에 대한 명칭 등)을 조절합니다. 영향을 미치는 함수: strftime(), strptime()
- LC_MONETARY : 금액 표현(천단위 구분 문자, 소수점 문자, 금액 표시 문자, 그 위치 등)을 조절합니다. 영향을 미치는 함수: strfmon()
- LC_NUMERIC : 금액이 아닌 숫자 표현(천단위, 소수점, 숫자 그룹핑 등)을 조절합니다. 영향을 미치는 함수: strtod(), atof().
- LC_COLLATE : 문자열의 정렬 순서를 조절합니다. 영향을 미치는 함수: strcoll(), wcscoll(), strxfrm()

더 자세한 정보는
[IBM 문서](https://www.ibm.com/docs/en/aix/7.1?topic=locales-understanding-locale-environment-variables),
[cpp 문서](https://en.cppreference.com/w/cpp/locale/LC_categories),
[커피닉스 문서](http://coffeenix.net/doc/misc/locale.html)를 참고해주세요.
