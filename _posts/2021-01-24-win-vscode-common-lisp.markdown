---
layout: post
title:  "Common Lisp 윈도 개발환경 실패기"
date:   2021-01-24 03:00:00 +0900
categories: lisp
---

## VS Code 실패기

윈도에서 Common Lisp 개발을 할 때 VS Code를 사용해보려 [commonlisp-vscode](https://marketplace.visualstudio.com/items?itemName=ailisp.commonlisp-vscode)를 설치해보았다.

[Roswell](https://github.com/roswell/roswell)를 설치하려 [문서](https://github.com/roswell/roswell/wiki/Installation#windows)를 보니 [Scoop](https://scoop.sh/)을 통해 설치할 것을 권한다. Scoop은 커맨드라인 인스톨러라고 설명되어 있는데 윈도 버전의 [Homebrew](https://brew.sh/)인가 보다. [예전 루비를 윈도에 설치할 때 시도한 Chocolatey는 영 별로](http://dalinaum.github.io/windows/ruby/2018/03/03/windows-ruby.html) 였는데 Scoop은 로컬에 설치해서 그나마 괜찮았다.

Scoop의 설치는 생각보다 어렵지는 않았다. 파워쉘에서 다음의 커맨드를 입력한다.

```
Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
```

혹시나 문제가 생기면 아래의 커맨드를 먼저 입력한다.

```
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
```

나는 파워쉘에 익숙하지 않아 `cmd`로 돌아가 아래와 같이 roswell을 설치했다.

```
scoop install roswell
```

권한을 요구하지 않고 사용자의 홈에 설치하기 때문에 Chocolatey보다는 훨씬 마음이 놓인다.

```
ros install ailisp/linedit
ros install ailisp/prepl
ros install ailisp/cl-lsp
```

`ailisp/linedit` 설치에서 `gcc`를 찾으며 죽어버린다. vscode에 개발환경을 설치하기 위해 `gcc`([MSYS2](https://www.msys2.org/))를 설치하는 것은 좀 오버같다. 그리고 MSYS2등을 설치한다 하여 `ailisp/linedit`가 수정없이 빌드될지도 자신이 없다.

여기에서 시간을 낭비하고 싶지 않아 다른 에디터를 알아보려 한다.

## LEM 실패기

이후 알아본 환경은 스탠드얼론 환경이라는 [LEM](https://github.com/lem-project/lem/wiki/Windows-Platform) 환경이었다. MSYS2를 요구하고 설치방식이 전혀 깔끔해 보이지 않는다. 일단 보류하자.
