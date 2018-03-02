---
layout: post
title:  "윈도우즈 루비 환경의 괴랄함"
date:   2018-03-03 01:00:00 +0900
categories: windows ruby
---

~~오버워치 머신~~ PC에 루비를 설치하고자 [rbenv](https://github.com/rbenv/rbenv)를 찾아봤지만 윈도우즈에서는 되지 않더라. 그래서 대안을 찾다가 패키지 시스템을 찾아 보았다. 윈도우즈용 패키지 시스템을 써봤는데 [Chocolately](https://chocolatey.org/)는 앱을 설치할 때 마다 관리자 쉘에서 수행해야 하는 것이 좀 별로였다. 그래서 굿 바이.

많은 사람들이 사용하는 것으로 보이는 [RubyInstaller](https://rubyinstaller.org/)를 설치했다. 한번에 설치가 잘 안되고 에러가 나던데 [MSYS2](http://www.msys2.org/)에서 다운로드를 받다가 에러가 생겼다. 조금 지나서 다시 시도하니 정상적으로 받아졌다. MSYS2는 윈도우즈 환경에서 네이티브로 GNU 도구를 사용하는 것을 목표로 하는 프로젝트다.

MSYS2는 자체의 패키지를 다루기 위해서 [pacman package manager](https://wiki.archlinux.org/index.php/pacman)를 [사용한다](https://github.com/msys2/msys2/wiki/Using-packages). 내가 필요한 의존성과 다른 도구에 의해 필요한 의존성이 팩맨에 의해 관리되는 것이다.

그런데 루비 인스톨러는 자신이 필요한 패키지를 다루고 위해 [ridk라는 커맨드라인](https://github.com/oneclick/rubyinstaller2#the-ridk-command) 도구를 사용하여 MSYS2를 다룬다.

루비를 시작하기 전에 수 많은 개념이 내 머리속에 스택 형태로 쌓였다. 그냥 맥북을 열 걸.
