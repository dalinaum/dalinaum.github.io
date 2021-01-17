---
layout: post
title:  "동영상 편집을 위한 루틴"
date:   2021-01-17 22:35:00 +0900
categories: video
---

유튜브 영상을 만들기 위한 루틴을 적어봅니다.

퀵 타임으로 맥에서 내가 작업하거나 발표를 진행하고 이를 PC에서 편집하고 있습니다.

마이크는 [Sony ICD-TX650](https://www.sony.co.kr/electronics/voice-recorders/icd-tx650) 모델을 사용합니다.

## 핸드브레이크 설정

퀵 타임으로 녹화된 영상은 VBR (Variable bitrate)이기 때문에 어도비 프리미어에서 편집할 때 문제가 생겼다 다음과 같이 [핸드브레이크](https://handbrake.fr/)를 이용해서 변환했다.

1. 소스 영상을 선택.
2. `Dimensions` 탭의 `Anamorphic`은 `Loose`.
3. `Video` 탭에서 `Framerate` 항목에서 `Constant Framerate` 선택.
4. `Chapters` 탭에서 `Create chapter markers` 비활성화.
5. `Save As`에서 파일명 선택 후 `Start Encode`

## 프리미어

1. mp4(핸드브레이크에서 편집된 영상) 파일과 mp3(TX650으로 녹음된 음성) 파일을 모두 추가해서 작업 준비.
2. `창` > `오디오 트랙믹서`에서 왼쪽 상단의 `>`를 클릭.
3. 두 번째 박스 첫 번째 슬롯에 `노이즈 감소/복원` > `자동 클릭 제거`
4. 해당 슬롯 더블 클릭, `사전 설정:` `중간`
5. 다섯 번째 슬롯, `진폭 및 압축` > `선택적 제한`
6. 해당 슬롯을 더블 클릭, `입력 부스트` `6db` 정도로 (들어보면서 조정)

이후 편집을 진행한다.