---
layout: post
title:  "윈도우 터미널 설치하기"
date:   2021-03-10 21:15:00 +0900
categories: windows
---

윈도우 터미널을 설치해보았다.

## Windows Packager Manager 설치하기

윈도우 터미널을 설치하기 위해 윈도우 패키지 매니저가 필요하다.

[참가자 프로그램 사이트](https://insider.windows.com/en-us/)에서 먼저 참가자 등록을 진행했다.

윈도우즈 시작 창에서 `설정` > `업데이트 및 보안` > `Windows 참가자 프로그램`에서 참가자 프로그램을 등록.

[앱 설치 관리자](https://www.microsoft.com/en-us/p/app-installer/9nblggh4nns1)를 수행.

이제 `winget` 커맨드를 수행할 수 있다.

```sh
$ winget
Windows Package Manager v0.2.10191 미리 보기
Copyright (c) Microsoft Corporation. All rights reserved.

원넷 명령줄 유틸리티를 사용하면 명령줄에서 응용 프로그램 및 기타 패키지를 설치할 수 있습니다.

사용: winget [<명령>] [<옵션>]

다음 명령을 사용할 수 있음
  install   지정된 패키지를 설치합니다.
  show      패키지에 대한 정보 표시
  source    패키지 원본 관리
  search    패키지의 기본 정보를 찾아 표시
  hash      해시 설치 관리자 파일 도우미
  validate  매니페스트 파일의 유효성 검사
  settings  설정 열기
  features  실험적 기능의 상태 표시

특정 명령에 대한 자세한 내용을 보려면 도움말 인수에 해당 명령을 전달합니다. [-?]

다음 선택 사항을 사용할 수 있음
  -v,--version  도구의 버전을 표시
  --info        도구의 일반 정보를 표시

자세한 도움말은 다음의 위치에서 찾아볼 수 있습니다. https://aka.ms/winget-command-help
```

## Windows Terminal 설치하기

이제 [Windows Terminal](https://github.com/Microsoft/Terminal) 을 설치합니다.

```sh
winget install --id=Microsoft.WindowsTerminal -e
```

`install` 커맨드를 할 때 `id`


이제 Windwos Terminal을 쓸 수 있는데 조금 심심하다. 반투명한 배경을 사용해보자.

`Control`+`,`을 누르면 설정을 바꿀 수 있는데 JSON을 편집할 에디터를 요구한다. 일단 노트패드를 이용해 변경한다.

```json
...
{
    ...
    "profiles":
    {
        "defaults":
        {
            // Put settings here that you want to apply to all profiles.
            "useAcrylic": true, 
            "acrylicOpacity": 0.5
        },
        ...
```

`useAcrylic`과 `acrylicOpacity`를 넣어주면 반투명한 터미널을 이용할 수 있다.

![윈도우 터미널 반투명]({{ "/assets/7-windows-terminal.png" | absolute_url }})