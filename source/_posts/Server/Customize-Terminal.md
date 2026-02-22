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
- ghostty
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

# 7. Ghostty 터미널

[Ghostty](https://ghostty.org)는 2024년 말 공개된 터미널 에뮬레이터로, GPU 가속 렌더링과 네이티브 UI를 결합하여 빠른 속도와 높은 완성도를 자랑합니다.
macOS와 Linux를 모두 지원하며, 설정 파일 하나로 간결하게 관리할 수 있습니다.

설정 파일 위치는 `~/.config/ghostty/config`입니다.

## 테마 및 폰트

```ini
theme = Solarized Dark Higher Contrast
font-family = "SourceCodePro+Powerline+Awesome Regular"
font-family = "Apple SD Gothic Neo"
font-style-bold = false
font-style-bold-italic = false
```

- `theme`: 기본 제공 테마 이름을 입력합니다. `Solarized Dark Higher Contrast`는 Solarized Dark보다 대비가 높아 가독성이 좋습니다.
- `font-family`: 우선순위 순서로 여러 개 지정할 수 있습니다. 첫 번째 폰트에 없는 글자(한글 등)는 두 번째 폰트로 자동 대체됩니다.
- `font-style-bold` / `font-style-bold-italic`: `false`로 설정하면 볼드·이탤릭 서체를 별도로 렌더링하지 않고 일반 서체로 통일합니다.

  이 옵션을 설정하지 않으면 `ls` 명령어 결과가 이상하게 출력됩니다.
  `ls`는 기본적으로 디렉토리 이름을 볼드체로 출력하는데, `SourceCodePro+Powerline+Awesome Regular`는 **Regular 웨이트만 존재하는 패치 폰트**입니다.
  볼드 렌더링이 요청되면 Ghostty가 해당 폰트의 Bold 변형을 찾지 못해, Powerline·Awesome 글리프가 포함되지 않은 **시스템의 다른 Source Code Pro Bold 폰트**로 대체됩니다.
  그 결과 프롬프트의 특수 기호들이 깨지거나, 폰트 메트릭 차이로 인해 글자 간격이 틀어져 보입니다.
  `font-style-bold = false`를 설정하면 볼드 요청이 와도 항상 Regular 폰트를 사용하므로 이 문제가 사라집니다.

## 커서

```ini
cursor-style = block
cursor-color = #708284
shell-integration-features = no-cursor
```

- `cursor-style`: 커서 모양을 지정합니다. `block`, `bar`, `underline`, `block_hollow` 중 선택할 수 있습니다. 기본값은 `block`입니다. `Solarized Dark Higher Contrast` 테마는 `cursor-style`을 별도로 정의하지 않으므로 Ghostty 기본값인 `block`이 그대로 적용됩니다.
- `cursor-color`: 커서 색상을 hex 코드로 지정합니다. `Solarized Dark Higher Contrast` 테마의 기본 커서 색상은 `#f34b00`(주황빛 빨강)인데, 눈에 너무 튀어서 회색 계열인 `#708284`로 변경했습니다.
- `shell-integration-features = no-cursor`: Ghostty의 쉘 통합 기능 중 커서 제어만 비활성화합니다. zsh/powerlevel10k가 커서를 직접 관리하도록 맡깁니다.

  이 옵션을 설정하지 않으면 `cursor-style`과 `cursor-color` 설정이 적용되지 않습니다.
  Ghostty는 쉘 통합(shell integration) 기능을 활성화하면 zsh에 훅(hook)을 자동으로 주입하여, 명령어 실행 중·대기 중 등 **쉘의 상태에 따라 동적으로 커서를 변경하는 ANSI 이스케이프 시퀀스**를 프롬프트에 삽입합니다.
  이 동적 시퀀스가 config 파일의 정적 커서 설정을 덮어쓰기 때문에, 아무리 `cursor-style`과 `cursor-color`를 지정해도 반영되지 않는 것입니다.
  `no-cursor`를 설정하면 Ghostty가 커서 관련 이스케이프 시퀀스 주입을 생략하므로, config 파일에 지정한 커서 스타일과 색상이 그대로 유지됩니다.

## 배경 투명도

```ini
background-opacity = 0.85
background-blur-radius = 20
```

- `background-opacity`: 배경 투명도를 0~1 사이로 설정합니다. `0.85`는 살짝 투명하게 보이는 정도입니다.
- `background-blur-radius`: 투명 배경 뒤의 내용을 흐리게 처리하는 블러 반경입니다. macOS의 유리(glass) 효과와 함께 사용하면 깔끔합니다.

## 키 바인딩

```ini
keybind = global:cmd+grave_accent=toggle_quick_terminal
```

- `global:` 접두사를 붙이면 다른 앱이 포커스를 갖고 있어도 동작하는 전역 단축키로 등록됩니다.
- `cmd+grave_accent`는 `` Cmd+` ``에 해당합니다.
- `toggle_quick_terminal`은 화면 상단에서 슬라이드로 내려오는 Quick Terminal을 토글합니다. 어느 앱에서든 빠르게 터미널을 열 수 있어 매우 편리합니다.
