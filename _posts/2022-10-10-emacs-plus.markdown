---
layout: post
title:  "맥 OS 이맥스 플러스 설치하기"
date:   2022-10-10 03:13:00 +0900
categories: mac
---

## 맥 OS에 이맥스 플러스 설치

요즘 맥 OS에서 이맥스를 설치할 때 무엇을 설치하나 봤더니 [Emacs Plus](https://github.com/d12frosted/homebrew-emacs-plus)를 설치하는 경우가 많았다. [몇 안되는 GUI가 제대로 돌아가는 프로젝트](https://seorenn.tistory.com/182)라는 평도 있었다.

모든 설치는 Homebrew를 통해 진행했다.

```sh
brew tap d12frosted/emacs-plus
brew install emacs-plus
```

네이티브 컴파일 기능은 필요없다고 생각했기 때문에 `--with-native-comp` 등의 옵션은 사용하지 않았다.

### 로스웰로 리습 환경 설치하기

이맥스를 설치한 이유가 [Common Lisp](https://common-lisp.net/)을 설치하기 위함이었기 때문에 [Roswell](https://roswell.github.io/)도 같이 설치했다. Roswell을 이용하면 [SBCL(Steel Bank Common Lisp)](https://www.sbcl.org/)과 같은 Common Lisp 구현도 설치할 수 있고 기타 패키지도 설치할 수 있다.

```sh
brew install ros 
ros install sbcl
ros install slime
```

lem을 사용하기 위해 `.zshrc`에 다음을 추가했다.

```sh
export PATH=$PATH:~/.roswell/bin
```

`~/.emacs.d/init.el` 파일에 다음을 추가한다.

```lisp
(load (expand-file-name "~/.roswell/helper.el"))
(setq inferior-lisp-program "ros -Q run")
```