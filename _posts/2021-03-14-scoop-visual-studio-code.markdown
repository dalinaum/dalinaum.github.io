---
layout: post
title:  "윈도우에 Visual Studio Code 패키지로 설치하기. - Scoop"
date:   2021-03-14 02:52:00 +0900
categories: windows
---

나는 웹 브라우저로 무언가 받는 것을 좋아하지 않는다. 21세기인데 응용 프로그램의 설치 파일을 웹 브라우저에서 받거나 커맨드라인에서 받아 위저드 도구로 설치하는 것은 너무 이상한 것 같다. 설마 마이크로소프트의 최신 에디터를 [Visual Studio Code](https://code.visualstudio.com/) 윈도우에서 구닥다리 방법으로만 설치해야하지는 않을거라 생각했고 답을 찾았다.

윈도용 패키지 관리자 [Scoop](https://scoop.sh/)을 이용하면 Visual Studio Code를 설치할 수 있었다. Scoop은 파워셀에서 아래 두 커맨드를 입력하면 설치가 된다. 커맨드라인(`cmd`)이 아니라는 점을 유의하자. 파워셀을 모르면 윈도우 버튼을 눌러 PowerShell을 검색하자.

```sh
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
iwr -useb get.scoop.sh | iex
```

그런데 `scoop install vscode`로는 설치되지 않는다. scoop에서 앱이 모여있는 장소(저장소)를 `bucket`이라고 부른다. [Homebrew](https://brew.sh)에서 `tap`을 바꾸어 가며 비공식 앱을 설치하는 것 처럼 scoop에서는 bucket을 변경하며 다른 곳에서 받을 수 있다.

양조(brew)하는 애들은 꼭지(tap)를 바꿔가며 다른 맥주를 마시는 것이고, 아이스크림 한 스쿱(scoop, 아이스크림은 다른 통(bucket)에서 한 숟갈(scoop)씩 퍼가는 것이다.

Visual Studio Code는 `extras` 버켓에 있다.

```sh
scoop install git
scoop bucket add extras
scoop install vscode
```

버킷 기능을 이용하기 위해 `git`이 필요해 `git`도 설치하고, `extras` 버켓을 추가해 Visual Studio Code를 설치했다.

이제 Visual Studio Code를 사용할 수 있다. 커맨드 라인에 `code`를 치거나 시작 메뉴에서 Visual Studio Code를 찾자.