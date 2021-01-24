---
layout: post
title:  "Atom + Common Lisp 윈도 개발환경, SLIMA"
date:   2021-01-24 20:34:00 +0900
categories: lisp
---

Windows + Common Lisp 개발에서 사용하기 위해 [Atom](https://atom.io/)에서 [SLIMA](https://github.com/neil-lindquist/SLIMA)를 써보기로 결정했다.

다른 환경들은 [MSYS2](https://www.msys2.org/) 등을 설치하는 등 번거로운 부분이 많아 포기했다.

```
apm install slima
apm install language-lisp
apm install lisp-paredit
apm install parinfer
```

[CLISP](https://clisp.sourceforge.io/)이 설치되어 있기 때문에 그걸 쓰기로 했다. 설치된 Lisp 구현이 없으면 [Roswell](https://github.com/roswell/roswell)을 이용해 설치하면 될 것 이다. 윈도 환경에 [Scoop](https://scoop.sh/)과 Roswell을 사용하는 것은 [지난 글](/lisp/2021/01/24/win-vscode-common-lisp.html)에서 다루었다.

```
scoop install roswell
ros install sbcl
```

[SLIME](https://common-lisp.net/project/slime/)을 [릴리즈](https://github.com/slime/slime/releases)에서 받음.

`Settings`(Control + ,)에서 `Packages`, `SLIMA`을 찾아서 `Setting` 버튼을 누름. `Lisp Process`에 `clisp`을 `SLIME Path`에 아까 SLIME을 설치했던 경로 `D:\slime-2.26.1`를 지정

역시 설정에서 `Keybindings` > `your keymap file`을 선택 후 아래의 내용을 입력.

```
'atom-text-editor[data-grammar~="lisp"]:not(.autocomplete-active)':
    'tab': 'lisp-paredit:indent'
```

`autocomplete-plus` 패키지에서 `Keymap For Confirming A Suggestion`의 선택을 `Tab and Enter`에서 `Tab`으로 변경.

`bracket-matcher` 패키지에서  `Autocomplete Brackets`을 비활성화함.