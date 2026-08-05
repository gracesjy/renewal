---
title: Outlier Detection in Manufacturing Data
img_path: /assets/images/
math: true
author: Alex
date: 2025-11-04
category: [Manufacturing, Statistics, Outlier Detection]
tags:
 - Tukey Method
 - Carling Method
 - Outlier Detection
layout: post
---

## 개요
제조 공정에서는 매 순간 수많은 데이터가 생성됩니다. 반도체 장비에서는 온도(Temperature), 압력(Pressure), RF Power, 가스 유량(Gas Flow)와 같은 공정 변수뿐 아니라 두께(Thickness), 식각 깊이(Etch Depth), 저항(Resistance) 등 다양한 품질 데이터가 지속적으로 측정됩니다.

이러한 데이터는 공정 상태를 이해하고 품질을 개선하는 데 중요한 역할을 합니다. 하지만 현실의 데이터는 항상 깨끗하지 않습니다. 센서 오류, 장비 이상, 공정 불안정, 작업자의 실수 등으로 인해 정상적인 분포에서 크게 벗어나는 값이 나타날 수 있으며, 이를 이상치(Outlier) 라고 합니다.

## 이상치는 왜 중요한가?
많은 사람들이 이상치를 단순히 "잘못 측정된 값"이라고 생각합니다. 하지만 제조 현장에서는 반드시 그렇지 않습니다.

이상치는 다음과 같은 중요한 정보를 포함할 수 있습니다.

- 센서의 오동작
- 장비 고장 초기 징후
- 공정 조건의 변화
- 원재료 품질 문제
- 새로운 불량 패턴의 발생

즉, 이상치는 단순히 제거해야 하는 데이터가 아니라 공정 이상을 가장 먼저 알려주는 신호일 수도 있습니다.

예를 들어, RF Power가 평소보다 크게 증가한 한 번의 측정값은 측정 오류일 수도 있지만, RF Generator의 이상이나 Matching Network 문제를 의미할 수도 있습니다.

## 이상치를 무조건 제거하면 안 되는 이유
데이터 분석을 시작하면 가장 먼저 이상치를 제거하려는 경우가 많습니다. 그러나 제조 데이터에서는 이러한 접근이 위험할 수 있습니다.

예를 들어 다음과 같은 상황을 생각해 보겠습니다.

- 동일한 장비에서 특정 Lot 이후부터 이상치가 반복적으로 발생한다.
- 특정 Recipe에서만 큰 값이 나타난다.
- 특정 Chamber에서만 이상치가 집중된다.

이러한 현상은 공정 이상을 의미할 가능성이 높습니다.

따라서 이상치를 제거하기 전에 먼저 왜 발생했는지 분석하는 과정이 반드시 필요합니다.

## 이상치를 탐지하는 대표적인 방법
| 방법                        | 특징                       |
| ------------------------- | ------------------------ |
| Z-score                   | 평균과 표준편차 기반              |
| Modified Z-score          | Median 기반으로 이상치에 강인함     |
| IQR (Interquartile Range) | 사분위수를 이용한 비모수 방법         |
| Tukey Method              | IQR 기반의 가장 널리 사용되는 방법    |
| Carling Method            | Tukey를 개선한 방법으로 표본 수를 고려 |

이번 글에서는 제조 데이터에서 가장 널리 사용되는 Tukey 방법과 이를 보완한 Carling 방법을 중심으로 살펴보겠습니다.

### Tukey Method

Tukey Method는 사분위수(Quartile)와 사분위 범위(Interquartile Range, IQR)를 이용하여 이상치(Outlier)를 탐지하는 대표적인 비모수(Non-parametric) 통계 기법입니다.

평균과 표준편차에 의존하지 않기 때문에 이상치의 영향을 적게 받으며, 제조 데이터와 같이 분포를 사전에 알기 어려운 데이터 분석에 널리 사용됩니다.

#### Interquartile Range (IQR)

IQR은 제3사분위수(\(Q_3\))와 제1사분위수(\(Q_1\))의 차이로 정의됩니다.

$$
IQR = Q_3 - Q_1
$$

여기서

- \(Q_1\) : 제1사분위수 (25%)
- \(Q_2\) : 중앙값(Median)
- \(Q_3\) : 제3사분위수 (75%)

#### Lower Fence

아래 경계는 다음과 같이 계산합니다.

$$
Lower\ Fence = Q_1 - 1.5 \times IQR
$$

#### Upper Fence

위쪽 경계는 다음과 같이 계산합니다.

$$
Upper\ Fence = Q_3 + 1.5 \times IQR
$$

#### Outlier Decision Rule

관측값 \(x\)가 다음 조건을 만족하면 이상치로 판단합니다.

$$
x < Q_1 - 1.5 \times IQR
$$

또는

$$
x > Q_3 + 1.5 \times IQR
$$

#### Summary

Tukey Method는 평균과 표준편차를 사용하지 않고 사분위수만을 이용하므로, 이상치가 포함된 데이터에서도 비교적 안정적인 이상치 탐지가 가능합니다.

특히 제조 공정 데이터, 품질 데이터, 센서 데이터와 같이 데이터 분포를 사전에 가정하기 어려운 경우 가장 널리 사용되는 이상치 탐지 방법 중 하나입니다.


### Carling Method

Carling Method는 Tukey의 IQR(Interquartile Range) 방법을 개선한 이상치 탐지 기법입니다.

Tukey Method는 모든 데이터에 대해 동일한 \(1.5 \times IQR\) 기준을 적용하지만, Carling Method는 **표본 수(Sample Size)**를 고려하여 이상치 탐지 경계를 보정합니다.

특히 표본 수가 적거나 중간 규모인 데이터에서는 Tukey Method보다 보다 안정적인 이상치 탐지가 가능하다고 알려져 있으며, 제조 데이터 분석이나 품질 관리 분야에서도 활용됩니다.

#### Interquartile Range (IQR)

Carling Method 역시 기본적으로 IQR을 사용합니다.

$$
IQR = Q_3 - Q_1
$$

여기서

- \(Q_1\) : 제1사분위수 (25%)
- \(Q_2\) : 중앙값(Median)
- \(Q_3\) : 제3사분위수 (75%)

#### Carling Correction Factor

Carling은 Tukey의 고정 계수(1.5)를 사용하지 않고, 표본 수 \(n\)에 따라 다음과 같은 보정 계수를 적용합니다.

$$
k(n)=\frac{17.63n-23.64}{7.74n-3.71}
$$

#### Lower Boundary

$$
Lower\ Boundary = Q_1 - k(n)\times IQR
$$

#### Upper Boundary

$$
Upper\ Boundary = Q_3 + k(n)\times IQR
$$

#### Outlier Decision Rule

관측값 \(x\)가 다음 조건을 만족하면 이상치로 판단합니다.

$$
x < Q_1 - k(n)\times IQR
$$

또는

$$
x > Q_3 + k(n)\times IQR
$$

#### Characteristics

Carling Method는 표본 수에 따라 이상치 경계를 자동으로 조정하므로, Tukey Method보다 작은 표본에서 보다 신뢰성 있는 결과를 제공할 수 있습니다.

반면 표본 수가 충분히 커질 경우에는 Tukey Method와 유사한 결과를 나타내는 경우가 많습니다.

따라서 제조 공정 데이터, 실험 데이터, 품질 데이터와 같이 표본 수가 제한적인 경우 Carling Method는 Tukey Method를 보완하는 이상치 탐지 기법으로 활용됩니다.


## 예제
```python
import numpy as np
import matplotlib.pyplot as plt

# 1. 오른쪽으로 쏠린 가상 데이터 생성 (Log-normal distribution)
np.random.seed(42)
data = np.random.lognormal(mean=1.5, sigma=0.55, size=100)
n = len(data)

# 2. 통계량 산출
median = np.median(data)
q25, q75 = np.percentile(data, [25, 75])
iqr = q75 - q25

# (1) Tukey 방식 경계
tukey_lower = q25 - 1.5 * iqr
tukey_upper = q75 + 1.5 * iqr

# (2) Carling 방식 경계
k = (17.63 * n - 23.64) / (10.72 * n - 17.54)  # n=100일 때 약 1.638
carling_lower = median - k * iqr
carling_upper = median + k * iqr

# 3. 시각화
plt.figure(figsize=(12, 6))

# 데이터 개별 점 및 분포 (Jittering 적용)
y_jitter = np.random.normal(1, 0.04, size=n)
plt.scatter(data, y_jitter, alpha=0.5, color='gray', label='Data Points')

# Tukey 경계선 (Red)
plt.axvline(x=tukey_upper, color='crimson', linestyle='--', linewidth=2, label=f'Tukey Upper ({tukey_upper:.2f})')

# Carling 경계선 (Blue)
plt.axvline(x=carling_upper, color='navy', linestyle='-', linewidth=2, label=f'Carling Upper ({carling_upper:.2f})')

# 데이터 중심 표시 (Median)
plt.axvline(x=median, color='green', linestyle=':', linewidth=2, label=f'Median ({median:.2f})')

plt.title("Outlier Boundary Comparison: Tukey vs. Carling Method", fontsize=14)
plt.xlabel("Value", fontsize=12)
plt.yticks([])
plt.grid(True, alpha=0.3)
plt.legend(fontsize=11)
plt.tight_layout()
plt.show()

```
![Outlier Boundary Comparison](/assets/images/2026-08-05-115714.png)

## Tukey Method와 Carling Method 비교(위 예제)

위의 그림은 동일한 데이터에 대해 **Tukey Method**와 **Carling Method**가 계산한 이상치(Outlier) 경계를 비교한 결과입니다.

### 데이터 분포

회색 점(Gray Dots)은 각 관측값(Observation)을 나타냅니다.

예를 들어 제조 공정에서는 다음과 같은 측정값이 될 수 있습니다.

- Wafer Thickness
- Etch Rate
- RF Power
- Chamber Pressure
- Temperature

각 점은 하나의 측정 데이터를 의미하며, 가로축(X-axis)은 측정값(Value)을 나타냅니다.

---

### Median (중앙값)

초록색 점선은 데이터의 **중앙값(Median)**을 나타냅니다.

본 예제에서는

$$
Median = 4.18
$$

입니다.

중앙값은 평균(Mean)보다 이상치의 영향을 적게 받기 때문에 이상치 탐지에서 자주 사용됩니다.

---

### Tukey Method

빨간 점선은 Tukey Method가 계산한 상한 경계(Upper Fence)를 나타냅니다.

본 예제에서는

$$
Upper\ Fence = 9.18
$$

입니다.

즉,

$$
Value > 9.18
$$

인 데이터는 Tukey Method에서 이상치(Outlier)로 판단됩니다.

---

### Carling Method

파란 실선은 Carling Method가 계산한 상한 경계입니다.

본 예제에서는

$$
Upper\ Boundary = 8.11
$$

입니다.

즉,

$$
Value > 8.11
$$

인 데이터는 Carling Method에서 이상치로 판단됩니다.

---

### 결과 해석

그림을 보면 Carling Method의 경계가 Tukey Method보다 더 왼쪽에 위치하고 있습니다.

즉,

$$
8.11 < 9.18
$$

이므로 동일한 데이터에서도 Carling Method가 더 많은 데이터를 이상치로 탐지하게 됩니다.

예를 들어 측정값이

$$
Value = 8.5
$$

라고 가정하면

### Tukey Method

$$
8.5 < 9.18
$$

따라서 정상 데이터로 판단합니다.

### Carling Method

$$
8.5 > 8.11
$$

따라서 이상치로 판단합니다.

즉, 동일한 데이터라도 선택한 이상치 탐지 기법에 따라 결과가 달라질 수 있습니다.

---

### 왜 이런 차이가 발생할까?

Tukey Method는 모든 데이터에 대해 동일한

$$
1.5 \times IQR
$$

기준을 적용합니다.

반면 Carling Method는 표본 수(Sample Size)를 고려하여 경계를 보정합니다.

따라서 표본 수가 적거나 중간 규모인 데이터에서는 Tukey Method보다 현실적인 이상치 경계를 제공하는 경우가 많습니다.

---

### 제조 데이터에서의 의미

제조 공정에서는 이상치가 반드시 잘못된 데이터(Error)를 의미하는 것은 아닙니다.

이상치는 다음과 같은 중요한 정보를 포함할 수도 있습니다.

- 센서 오동작
- 장비 이상
- 공정 조건 변화
- 새로운 불량 발생
- 품질 이상 징후

따라서 이상치를 무조건 제거하기보다는 **왜 이러한 값이 발생했는지를 먼저 분석하는 과정이 중요합니다.**

---

## 정리

| 항목 | Tukey Method | Carling Method |
|------|--------------|----------------|
| 기준 | \(Q_3 + 1.5 \times IQR\) | 표본 수를 고려한 보정 계수 적용 |
| 계산 난이도 | 매우 쉬움 | 다소 복잡 |
| 표본 수 고려 | ✗ | ✓ |
| 작은 표본에서의 안정성 | 보통 | 우수 |
| 이상치 탐지 | 다소 완화 | 다소 엄격 |
| 제조 데이터 활용 | 매우 많이 사용 | Tukey를 보완하는 방법 |

이번 예제에서는 Carling Method가 Tukey Method보다 더 작은 상한 경계를 계산하였으며, 그 결과 더 많은 데이터를 이상치로 탐지하는 것을 확인할 수 있습니다.

이처럼 두 방법은 모두 IQR 기반의 강건한(Robust) 이상치 탐지 기법이지만, **Carling Method는 표본 수를 고려하여 Tukey Method를 개선한 방법**이라는 점이 가장 큰 차이점입니다.