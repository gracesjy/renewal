---
title: Sensor Relationship Analysis using Correlation Matrix and Hierarchical Clustering
img_path: /assets/images/
author: Raymond
date: 2025-11-02
category: [Manufacturing, Statistics, Correlation Analysis]
tags:
 - Correlation
 - Sensor Data
 - PostgreSQL
 - Oracle
layout: post
---

## 제조 데이터에서 Correlation Heatmap만으로 충분할까?

반도체 제조 설비에서는 수백 개에서 수천 개의 센서가 동시에 측정된다.  특히 반도체 FAB 의 경우에는 FDC(Fault Detection and Classification) 시스템이 존재하고 데이터를 수집해서 데이터베이스에 실시간으로 적재된다.

예를 들어

- Temperature
- Pressure
- Gas Flow
- RF Power
- RF Voltage
- ESC Temperature

등 다양한 센서들이 일정한 Sampling Time마다 측정된다.  반도체의 경우 Semi Standard 의 SECS/GEM 프로토콜과 함께 EDA (Equipment Data Acquisition) 으로 0.1초, 0.01초 등의 주기로 샘플링이 진행 되기도 한다.  따라서 많은 센서별 시계열 데이터가 수집되어 설비로부터 날라와서 적재된다.

이러한 센서 데이터를 기반으로, 특히 신규 설비의 경우 가장 먼저 수행되는 분석 중 하나가  **Correlation Analysis(상관분석)** 이다.  설비의 모든 센서를 수집하는 것은 의미가 없기 때문이고 부하를 가중시키기 때문이다.  따라서 DCP (Data Collection Plan)이 허용되고 정하는 데 이러한 분석은 도움이 된다.

---

## Correlation Matrix

상관계수는 두 변수간의 선형 관계를 나타낸다.

| Correlation | Meaning |
|-------------|---------|
| +1 | 완전한 양의 상관관계 |
| 0 | 상관관계 없음 |
| -1 | 완전한 음의 상관관계 |

Python에서는 매우 간단하게 계산할 수 있다.

```python
corr_matrix = df_wide.corr(method='pearson')
```

하지만 문제가 있다.

센서가 300개만 되어도

Correlation Matrix는

```
300 x 300 = 90,000
```

개의 셀을 가진다.

Heatmap은 다음과 같이 거의 해석이 어려운 그림이 된다.

---

## 단순 Heatmap의 한계

Heatmap은

- 어떤 센서끼리 관계가 있는지
- 어떤 그룹을 이루는지

를 직관적으로 알기 어렵다.

특히

- 200 Sensors
- 500 Sensors
- 1000 Sensors

에서는 거의 의미를 잃는다.

따라서

**센서를 재배치(Reordering)** 하는 과정이 필요하다.

---

# Hierarchical Clustering

Correlation Matrix에

Hierarchical Clustering을 적용하면

비슷한 센서끼리 자동으로 묶인다.

```python
g = sns.clustermap(
    corr_matrix,
    method='average',
    metric='euclidean',
    cmap='vlag',
    figsize=(14,14),
    xticklabels=False,
    yticklabels=False
)
```

그러면

Heatmap이 아니라

**Sensor Relationship Map**

으로 바뀐다.

---

## 결과 해석

가장 먼저 눈에 들어오는 것은

좌측 상단의 큰 붉은 블록이다.

```
██████████
██████████
██████████
```

이 의미는

> 서로 매우 높은 상관관계를 가지는 센서 그룹

이라는 뜻이다.

실제 설비에서는

- Heater
- Wall Temperature
- Chuck Temperature

등 하나의 Physical System일 가능성이 높다.

---

그 다음

여러 개의 작은 붉은 사각형이 보인다.

이들은

Subsystem을 의미한다.

예를 들어

- Pressure
- Gas Flow
- Throttle Valve

처럼 하나의 제어 Loop를 구성하는 센서일 수도 있다.

---

파란색 영역은

음의 상관관계를 의미한다.

즉

한 센서가 증가하면

다른 센서는 감소하는 특성을 가진다.

Valve Opening과 Chamber Pressure가 대표적인 예이다.

---

## Dead Sensor 제거

실제 제조 데이터에서는

모든 센서가 정상적으로 동작하지 않는다.

고정값만 출력하는

Dead Sensor도 존재한다.

이러한 센서는 반드시 제거해야 한다.

```python
std_series = df_wide.std()

valid_sensors = std_series[
    (std_series > 1e-12)
].index

df_wide = df_wide[valid_sensors]
```

표준편차가 0인 센서는

Correlation

Regression

PCA

모든 분석 결과를 왜곡시킨다.

---

## Long Format → Wide Format

대부분의 제조 데이터는

다음과 같은 형태이다.

| Sequence | Variable | Value |
|----------|----------|------:|
|1|Pressure|1.2|
|1|Temperature|398|
|1|RFPower|350|

이 형태를

```python
pivot()
```

을 이용하여

```
Sequence

↓

Sensor1
Sensor2
Sensor3
```

형태로 변환해야 한다.

이 과정은

Manufacturing Data Analytics에서

가장 먼저 수행되는 데이터 전처리 과정이다.

---

## 왜 Clustering을 하는가?

Correlation Matrix를 보기 위한 것이 아니다.

목적은

**Feature Engineering**이다.

예를 들어

```
Temperature Sensors

↓

Cluster A
```

```
Pressure Sensors

↓

Cluster B
```

```
Gas Sensors

↓

Cluster C
```

처럼

센서를 자동으로 그룹화할 수 있다.

---

## PCA와의 연결

Hierarchical Clustering은

끝이 아니다.

다음 단계는

PCA이다.

```
Raw Sensors

↓

Correlation

↓

Hierarchical Clustering

↓

Feature Selection

↓

PCA

↓

Machine Learning
```

즉

Correlation Analysis는

Machine Learning을 위한

Feature Engineering의 첫 단계이다.

---

# 정리

Manufacturing Data Analytics에서

Correlation Matrix는

단순히 상관계수를 계산하는 작업이 아니다.

실제 목적은

- 센서간 관계 이해
- 센서 그룹 발견
- Feature Selection
- PCA 전처리
- Machine Learning 입력 변수 선정

이다.

수백 개 이상의 센서를 다루는 제조 데이터에서는

단순 Heatmap보다

**Hierarchical Clustering을 이용한 Sensor Relationship Analysis**가

훨씬 효과적인 분석 방법이다.

---

## 다음 글

> **Principal Component Analysis(PCA) for High-Dimensional Manufacturing Data**

Correlation으로 센서 관계를 파악했다면,

다음 단계는

수백 개의 센서를

몇 개의 대표 변수(Principal Components)로 압축하는 과정이다.

