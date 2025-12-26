---
layout: post
title: Terminal 꾸미기
date: 2021-03-01 11:12:00 +0900
cover: /covers/bash.jpg
disqusId: 0011860c22129d38a8838a22fd2921333ee038a5
toc: true
category: Server
tags:
- zsh
- terminal
- ohmyzsh
---

검정 바탕에 흰 글자가 전부인 기본 Terminal은 눈을 침침하게 하고 코딩 의욕을 저하시키는 원인 중에 하나입니다.
다양한 테마와 폰트, 색상 프로필을 통해 터미널을 멋있게 꾸미는 법을 알아보겠습니다.

<!-- more -->

![맥 기본 터미널](zsh.png)

맥 기본 Terminal의 모습입니다. 검정 바탕에 흰 글자... 정말 재미도 없고 감동도 없는 그런 모습입니다.
이제 이 글을 따라하며 한 단계씩 설정해 나가면 다음과 같은 터미널을 만들 수 있습니다.

![이 글을 따라한 후 당신의 터미널](powerlevel10k.png)

# 1. zsh 설치

맥은 Catalina부터 기본 터미널이 zsh로 설정되어 있습니다. 따라서 대부분의 경우 이 단계는 건너뛰어도 됩니다.
터미널에서 `echo $0`를 입력한 결과값에 `zsh`가 포함되어 있지 않다면 아래의 명령어로 zsh를 설치하고, 기본 터미널을 zsh로 바꿔줍니다.

```shell
$ brew install zsh     # zsh 설치
$ chsh -s $(which zsh) # 기본 터미널을 zsh로 변경
```

# 2. Oh My ZSH 설치

먼저 다양한 테마와 다양한 플러그인을 설치할 수 있는 [Oh My ZSH](https://github.com/ohmyzsh/ohmyzsh)를 설치합니다.

```shell
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

위의 명령어를 통해 Oh My ZSH를 설치하면 아래와 같이 터미널이 조금 달라져 있는 것을 볼 수 있습니다.

![Oh MY ZSH 설치후 터미널](ohmyzsh.png)

# 3. Oh My ZSH 테마 변경

Oh My ZSH는 설정 파일에서 단순히 글자만 바꿈으로써 테마를 변경할 수 있습니다.
일단 여기서는 `agnoster` 테마를 적용하겠습니다.
더 많은 테마는 [여기](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)에서 확인해주세요.

```diff
- ZSH_THEME="robbyrussell"
+ ZSH_THEME="agnoster"
```

`~/.zshrc` 파일을 위와 같이 변경한 후 새 터미널을 열면 아래 사진과 같이 바뀐 테마가 적용 되어 있습니다.
명령어 입력란을 보면 좀 달라져 있는 것을 알 수 있습니다.

![폰트 문제 때문에 agnoster 테마가 잘 적용되지 않은 터미널](agnoster-bad.png)

하지만 폰트 문제 때문에 위와 같이 뭔가 이상하게 나오는 것을 알 수 있습니다.

# 4. 폰트 적용

https://github.com/powerline/fonts
에 가서 맘에드는 폰트를 설치하길 바랍니다.

저는 [Source Code Pro for Powerline](https://github.com/powerline/fonts/blob/master/SourceCodePro/Source%20Code%20Pro%20for%20Powerline.otf)을 사용했습니다.

그런 다음 터미널에서 아래와 같이 폰트를 바꾼 후 새 터미널을 열면 제대로 나오는 것을 볼 수 있습니다.

![터미널에서 폰트를 바꾸는 방법](terminal-font.jpg)

![agnoster 테마가 적용된 터미널](agnoster-good.png)

# 5. 색상 프로필 변경

여기까지만 해도 충분히 괜찮지만 검정 바탕에 흰 글자는 여전히 너무 식상합니다.
이제 색상 프로필을 변경함으로써 좀 더 그럴듯하게 만들어줄 것입니다.

먼저 [여기](https://github.com/lysyi3m/macos-terminal-themes)서 원하는 프로필을 선택한 후 다운로드 해줍니다.
저는 [Solarized Dark](https://github.com/lysyi3m/macos-terminal-themes/blob/master/themes/Solarized%20Dark.terminal) 프로필을 사용했습니다.

그런 다음 터미널에서 아래와 같이 프로필을 Import한 후 새 터미널을 열면 그럴듯한 색이 나오는 것을 볼 수 있습니다.

![터미널에서 색상 프로필을 Import 하는 방법](terminal-scheme.jpg)

![Solarized Dark가 젹용된 터미널](solarized.png)

# 6. powerlevel10k 테마

여기까지만 해도 충분이 괜찮지만 [powerlevel10k](https://github.com/romkatv/powerlevel10k) 테마를 통해 명령어 입력란을 좀 더 풍성하게 만들어 줄 수 있습니다.

1) `powerlevel10k` 테마는 `Oh My ZSH` 에서 기본으로 지원해 주지 않기 때문에 따로 아래의 명령어를 통해 `powerlevel10k` 테마를 다운로드 해줍니다.

```shell
$ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

2) `Source Code Pro for Powerline`에 `Awesome Font`가 추가된 폰트를 [여기](https://github.com/Falkor/dotfiles/blob/master/fonts/SourceCodePro%2BPowerline%2BAwesome%2BRegular.ttf)에서 다운로드한 후 설치합니다. 굳이 이 폰트가 아니더라도 Powerline과 Awesome Font가 추가된 폰트면 다 괜찮습니다.

3) 터미널 설정에 들어가 폰트를 `SourceCodePro+Powerline+Awesome Regular`로 바꿔줍니다.

4) 다음과 같이 `~/.zshrc` 파일을 변경한 후 새 터미널을 열면 아래 사진과 같이 interactive하게 설정을 할 수 있습니다.

```diff
- ZSH_THEME="agnoster"
+ ZSH_THEME="powerlevel10k/powerlevel10k"
```

![powerlevel10k 설정 화면](interactive.png)

5) 설정을 다 마치고 나면 다음과 같이 멋있고 풍성한 터미널을 볼 수 있습니다.

![powerlevel10k 테마가 적용된 터미널](powerlevel10k.png)

위의 사진처럼 `powerlevel10k` 테마에서는 명령어를 입력한 시간, 명령어가 걸린 시간, 명령어의 exit 값을 알 수 있습니다.

# 7. VSCode에 테마 적용하기

신나는 마음으로 VSCode에 들어가서 Terminal 창을 열면 아래와 같이 폰트가 다 깨져 있습니다.

![폰트가 다 깨져있는 VSCode 터미널](vscode1.png)

VSCode 설정에 들어가서 아래와 같이 입력하여 Terminal의 폰트를 바꿉니다.

![VSCode에서 터미널 폰트를 바꾸는 방법](vscode2.png)

그 후 새 터미널을 열어보면 다음과 같이 제대로 나오는 것을 알 수 있습니다.

![powerlevel10k 테마가 적용된 VSCode 터미널](vscode3.png)
