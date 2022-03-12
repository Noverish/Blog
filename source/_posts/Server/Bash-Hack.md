---
layout: post
title: Bash Hack - 멋있게 Bash 쓰기
date: 2020-04-26 14:33:00 +0900
cover: /covers/bash.jpg
disqusId: 6350540701e6e17afa89e62b36436d631a2c0def
toc: true
category: Server
tags:
- bash
---

자주 마주치는 8가지 상황에서 Bash의 다양한 기능을 이용해 적게 타이핑 하고 문제를 해결해 보자!

<!-- more -->

# 1. 깜빡하고 sudo를 입력하지 않음

```shelll
$ ln -s /path/to/command/file /usr/bin/command
$ sudo !!
```

`!!`는 바로 전에 입력한 명령어를 의미합니다.
`!-n`은 n번째 전에 입력한 명령어를 의미합니다.
따라서 `!!`와 `!-1`은 같은 의미입니다.

# 2. 명령어를 입력하는 데 오타 발생

```shell
$ some very long long lung long command
$ ^lung^long^
```

`^old^new^`는 가장 최근에 입력한 명령어 중에서 처음으로 찾은 `old`를 `new`로 바꾸어 줍니다.

# 3. 폴더를 만든 후 바로 들어 가기

```shell
$ mkdir verylonglongfoldername && cd verylonglongfoldername
$ mkdir verylonglongfoldername && cd $_

$ ls -al /path/to/list/files
$ cd $_
```

`$_`는 전에 입력한 명령어 중에 가장 마지막 파라미터를 의미합니다.
`ls` 명령어로 뭐가 있는지 확인하고 그 경로로 들어갈 때도 유용합니다.

# 4. 파일 이름을 일부분만 수정하기

```
$ mv /path/to/verylongfilename.txt /path/to/veryshortfilename.txt
$ mv /path/to/verylongfilename.txt !#$:s/long/short/
```

조금 복잡해 보이는데 분해해서 설명하면 다음과 같습니다.
- `!#`: 지금까지 입력란에 입력한 단어들을 의미합니다. 여기서는 `mv /path/to/verylongfilename` 입니다.
- `$`: 앞에서 찾은 단어들 중에 가장 마지막 단어를 의미합니다.
- `:s/old/new/`: 앞에서 찾은 단어에서 `old`를 `new`로 바꿉니다.

# 5. 지금까지 입력했던 명령어 검색

`Ctrl + r`을 누른 후 검색어를 입력하면 지금까지 입력했던 명령어 중에 검색어가 존재하는 가장 최근 명령어를 보여줍니다.
여기서 `Ctrl + r`을 누르면 그 다음 최근 명령어를 보여줍니다.

# 6. 하나의 명령어를 여러 줄에 입력

```shell
$ docker run \
    -e port=8080 \
    -e host=0.0.0.0 \
    -p 8080:8080 \
    service:latest
```

위와 같이 역슬래쉬를 입력하고 다음 라인에 명령어를 계속 입력할 수 있습니다.

# 7. 명령어를 한 번에 여러 개 실행

```shell
$ command1 && command2
$ command1 || command2
$ command1; command2
```

- `&&`: command1이 성공하면 command2도 실행됩니다.
- `||`: command1이 실패하면 command2가 실행됩니다.
- `;`: command1의 성공여부와 상관 없이 command2는 실행됩니다.

명령어를 그룹화 하고 싶으면 중괄호를 입력하면 됩니다. 중괄호로 그룹화한 명령어들 중에 가장 마지막 명령어 뒤에 세미콜른은 필수적으로 붙여주어야 합니다.

```shell
$ command1 && command2 && command3 > file
$ { command1 && command2 && command3; } > file
```

위의 명령어는 `command3`의 결과만 file에 기록되고 `command1`과 `command2`의 결과는 stdout에 출력됩니다.
아래의 명령어는 모든 명령어의 결과가 file에 기록됩니다.

# 8. 명령어를 백그라운드에서 실행

```shell
$ some command &
```

명령어 뒤에 `&`을 붙이면 그 명령어는 백그라운드에서 실행됩니다. 하지만 다음 두 가지의 단점이 있습니다.

- 명령어가 실행된 쉘을 종료하면 백그라운드로 실행되는 명령어도 종료됩니다.
- 명령어의 output이 현재 쉘에 그대로 출력됩니다.

따라서 `nohup`을 이용하는 편이 좋습니다.

```shell
$ nohup some command &
$ nohup some command &
```

명령어 앞에 `nohup`을 뒤에 `&`를 붙이면 위의 두 가지 단점이 사라집니다.

- 명령어가 실행된 쉘을 종료해도 백그라운드로 실행되는 명령어는 유지됩니다.
- 명령어의 output은 명령어가 실행된 경로에서 `nohup.out` 이라는 파일에 기록됩니다.

# Appendix

## 1) 현재 사용하고 있는 Shell 이름 알아내기

- `echo $0`: 현재 사용하고 있는 Shell 이름 알아내기
- `echo $SHELL`: 기본으로 설정된 Shell 이름 알아내기

## 2) 단축키 모음

- `Ctrl + a`, `Home`: 커서를 맨 앞으로 옮기기
- `Ctrl + e`, `End`: 커서를 맨 뒤로 옮기기
- `Ctrl + l`: clear 명령어와 같은 동작을 합니다.
- `Ctrl + t`: 현재 커서에 있는 글자와 그 앞에 있는 글자의 위치를 바꿉니다 (오타 수정용)
- `Ctrl + r`: 지금까지 입력했던 명령어를 역순으로 검색하기 (reverse-search-history)
- `Ctrl + d`: 명령창에 아무 글자도 없을 때 이 키를 입력하면 exit 명령어와 같은 동작을 합니다.
- `Ctrl + k`: 현재 커서 뒤에 있는 글자 들을 삭제합니다.
- `Ctrl + u`: 현재 커서 앞에 있는 글자 들을 삭제합니다.
- `Ctrl + w`: 현재 커서 앞에 있는 가장 가까운 공백까지 삭제합니다.
- `Ctrl + y`: 최근에 삭제한 글자들을 현재 커서 뒤에 입력합니다.

`bind -p` 명령어로 단축키를 확인할 수 있습니다.

## 3) 히스토리 이용하기

### 명령어 지칭

- `!!`: 바로 전에 입력했던 명령어
- `!n`: history에서 n번째 명령어
- `!-n`: n번째로 전에 입력했던 명령어
- `!string`: string으로 시작하는 가장 최근 명령어
- `!?string`: string을 포함하는 가장 최근 명령어
- `^string1^string2^`: 가장 최근 명령어에서 string1을 string2로 바꾼 명령어
- `!#`: 현재 입력란에서 지금까지 입력한 글자들을 지칭함

### 단어 지칭

위의 명령어 지칭을 사용한 후에 `:`를 입력하여 단어 지칭을 할 수 있습니다.
단어 지칭이 `^`, `$`, `*`, `-`, `%`로 시작하는 경우 `:`를 생략할 수 있습니다.

- `n`: 지칭된 단어들 중에 n번째 단어를 지칭함 (0 부터 시작함)
  - `mkdir folder; cd !#:1`
  - `docker stop service; docker rm !#:2`
- `$`: 지칭된 단어들 중에 마지막 단어를 지칭함
- `^`: 지칭된 단어들 중에 두번째 단어를 지칭함 (명령어를 제외한 첫 번째 argument)
- `x-y`: x번째 부터 y번째 까지의 단어
- `-y`: 0번째 부터 y번째 까지의 단어
- `*`: 첫 번째 단어를 제외한 모든 단어
- `x*`, `x-`: `x-$`와 같은 의미

명령어 지칭이 없이 단어 지칭을 사용하는 경우 바로 전에 입력했던 명령어에서 단어 지칭을 합니다.

### 단어 수정

optional word designator 다음에 단어 수정자를 한 개 이상 나열할 수 있습니다. 각각은 `:` 뒤에 옵니다.

- `h`: 가장 마지막 pathname component를 삭제합니다.
- `t`: 가장 마지막 pathname component만 남깁니다.
- `r`: 확장자를 삭제합니다.
- `e`: 확장자만 남깁니다.
- `s/old/new/`: 처음 등장하는 old를 new로 바꿉니다. `/` 대신 다른 구분자를 사용할 수 있고 역슬래쉬를 통해 구분자를 이스케이프 할 수 있습니다. 만약 `&`가 new에 등장합니다면 이는 old로 치환됩니다. & 또한 역슬래쉬로 이스케이프 할 수 있습니다. 맨 마지막 구분자가 입력 라인의 가장 마지막 글자라면 생략할 수 있습니다.
- `a`, `g`: 치환을 전체 라인에 대해 적용합니다. `s`나 `&` 앞에서 사용합니다. 예) `gs/old/new/`, `g&`
- `G`: 앞으로 등장하는 모든 치환을 모든 라인에 대해 적용합니다.
- `q`: 모든 단어를 한 번에 따옴표로 감쌉니다.
- `x`: 모든 단어를 각각 따옴표로 감쌉니다. 단어들은 공백, 탭, 개행으로 구분합니다.
- `&`: 가장 최근에 실행한 치환을 다시 반복합니다.
- `p`: 새로운 명령어를 출력하고 실행하지는 않습니다.

## 4) quote(')와 double quote(")의 차이

### quote(')

quote(') 사이의 어떤 특수문자도 아무런 처리를 하지 않습니다. 따라서 backslash(\\)로 quote를 escape 할 수 없습니다.

```shell
$ echo '$PORT'
$PORT

$ echo '!!'
!!

# quote 문자열에서 quote를 출력하고 싶으면 아래처럼 해야 합니다.
$ echo 'I'\''m human'
I'm human
```

### double quote(")

double quote(") 사이의 특수문자(!, $, # 등)는 기존 처리방법대로 처리합니다.

```shell
$ PORT=8080

$ echo "$PORT"
8080

$ echo "!!"
echo 8080
```

참고 자료

- <https://ss64.com/bash/syntax-keyboard.html>
- <http://tldp.org/LDP/abs/html/special-chars.html>
- <http://www.gnu.org/software/bash/manual/bash.html>
- <https://www.lesstif.com/system-admin/bash-readline-line-reverse-search-6717494.html>
