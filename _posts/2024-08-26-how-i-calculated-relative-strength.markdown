---
layout: post
title:  "나는 어떻게 상대 강도를 계산하였는가?"
date:   2024-08-26 01:03:00 +0900
categories: investment
---

[윌리엄 오닐의 Relative Strength Rating](https://www.williamoneil.com/proprietary-ratings-and-rankings/)에 기반한 [달리나음의 코스피 상대 강도](https://dalinaum.github.io/rs/)을 만든 다음 몇 번의 문의를 받았다. 어떻게 상대 강도를 계산했는지에 대한 질문이었다. 이에 대한 답변을 여기에 적어본다.

## 윌리엄 오닐의 상대 강도 등급

먼저 윌리엄 오닐의 상대 강도 등급에 대해서 살펴보자. 계산 방식에 대한 윌리엄 오닐 사이트의 설명은 다음과 같다.

RELATIVE STRENGTH RATING

This technical tool is one of the most popular ways for clients to see the market’s top performers. The Relative Strength Rating is the result of calculating a stock’s percentage price change over the last 12 months. A 40% weight is assigned to the latest three-month period; the remaining three quarters each receive 20% weight. All stocks are arranged in order of greatest price percentage change and assigned a percentile rank from 99 (highest) to 1 (lowest).

마켓 주도주를 알고 싶은 사람에게 가장 유용한 방법을 제공하는 기술적 도구이다. 상대 강도 등급(Reative Strehing)은 최근 12개월 동안 주식의 가격 변동률을 계산한 결과이다. 최근 3개월 기간에 40%의 가중치가 할당된다. 나머지 세 분기는 각각 20%의 가중치를 받는다. 모든 종목은 가격 변동율이 큰 순서대로 정렬되고 99(최고)부터 1(최저)의 백분위 순위가 할당된다.

## 계산 방식

오닐의 설명에 따라 4분기의 가격 변동률을 계산해 최근 1분기만 2배로 반영하고 나머지는 곱하지 않고 합산했다.

파이썬 코드로 나타내면 다음과 같다.

```py
today = data.loc[data.index[day]]
one_quarter_ago = data.loc[data.index[day - (quater)]]
two_quarter_ago = data.loc[data.index[day - (quater * 2)]]
three_quarter_ago = data.loc[data.index[day - (quater * 3)]]
four_quarter_ago = data.loc[data.index[day - (quater * 4)]]

score_1 = today.Close / one_quarter_ago.Close
score_2 = one_quarter_ago.Close / two_quarter_ago.Close
score_3 = two_quarter_ago.Close / three_quarter_ago.Close
score_4 = three_quarter_ago.Close / four_quarter_ago.Close

total_score = (score_1 * 2) + score_2 + score_3 + score_4
```

전체 코드는 [여기](https://github.com/dalinaum/rs/blob/main/calc-kospi-rs.py#L80C1-L83C67)를 참고하라.

이렇게 스코어를 계산한 후는 순위를 매기고 1등에게 99점, 꼴등에게 1점을 할당하도록 균등하게 점수를 매겼다.

```py
rs_df['Rank'] = rs_df['Score'].rank()
rs_df['RS'] = (rs_df['Rank'] * 98 / len(rs_df)).apply(np.int64) + 1
```

전체 점수로 순위를 매긴 다음에 전체 종목의 수로 나누면 0부터 1.0 사이의 값이 된다. 이 값에 98을 곱하면 0부터 98까지의 범위가 되고 여기에 1을 더하면 1부터 99까지의 범위가 된다.

굉장히 단순하지만 무식한 방법으로 집계를 했는데 윌리엄 오닐의 방식도 단순하고 직관적으로 집계했을 것이라고 생각한다.

## 피드백

해당 사이트에 대한 피드백은 [dalinaum@gmail.com](mailto:dalinaum@gmail.com)이나 [@dalinaum](https://x.com/dalinaum)으로 피드백을 달라.