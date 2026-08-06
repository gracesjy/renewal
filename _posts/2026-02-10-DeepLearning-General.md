---
title: Deep Learning General
img_path: /assets/images/
author: Alex
math: true
date: 2025-09-24
category: [Manufacturing, Deep Learning, Deep Learning General]
tags:
 - Docker Desktop
 - Docker export
 - Docker import
layout: post
---

## Deep Learning은 왜 등장했을까?

Machine Learning은 오랫동안 데이터 분석과 예측 분야에서 매우 뛰어난 성능을 보여 왔다. 특히 SVM(Support Vector Machine), Decision Tree, Random Forest와 같은 알고리즘은 지금도 다양한 산업 분야에서 널리 사용되고 있다.

그렇다면 이미 훌륭한 Machine Learning 알고리즘들이 존재했는데도 왜 Deep Learning이라는 새로운 접근법이 등장하게 되었을까?

이번 글에서는 동일한 설비 데이터를 예로 들어 Machine Learning과 Deep Learning의 차이점을 살펴본다.

---

## 동일한 설비 데이터

예를 들어 반도체 공정 장비에서 다음과 같은 센서 데이터를 수집한다고 가정해 보자.

| Temperature | Pressure | RF Power | Gas Flow | Result |
|-------------|----------|----------|----------|--------|
| 200 | 1.10 | 500 | 100 | OK |
| 205 | 1.15 | 510 | 102 | OK |
| 210 | 1.40 | 530 | 110 | NG |
| ... | ... | ... | ... | ... |

입력 데이터(X)

- Temperature
- Pressure
- RF Power
- Gas Flow

출력 데이터(Y)

- OK
- NG

목표는 센서 데이터를 이용하여 제품의 양품(OK)과 불량(NG)을 예측하는 것이다.

---

## Machine Learning 접근 방법

Machine Learning에서는 입력 데이터를 이용하여 가장 적절한 **Decision Boundary(결정 경계)** 또는 **Hyperplane**을 찾는다.

대표적인 알고리즘이 SVM(Support Vector Machine)이다.

```python
from sklearn.svm import SVC

model = SVC(kernel="rbf")

model.fit(X_train, y_train)

pred = model.predict(X_test)
```

학습 과정은 다음과 같이 생각할 수 있다.

```
Sensor Data
      │
      ▼
 SVM 학습
      │
      ▼
최적의 Decision Boundary
      │
      ▼
 OK / NG
```

SVM은 매우 강력한 알고리즘이며 데이터가 많지 않은 경우에도 높은 성능을 보여준다.

---

## 하지만 한계가 있었다

Machine Learning의 가장 큰 특징은 **Feature Engineering**이다.

즉,

사람이 데이터를 분석하여 중요한 특징(Feature)을 먼저 만들어 주어야 한다.

예를 들어

- 평균
- 최대값
- 최소값
- 표준편차
- FFT
- PCA

등을 사람이 설계한 후 SVM에 입력한다.

```
Raw Data

↓

Feature Engineering

↓

SVM

↓

Prediction
```

문제는 데이터가 점점 복잡해졌다는 것이다.

예를 들어

- 이미지
- 음성
- 자연어
- 반도체 장비의 수천 개 센서 데이터

에서는 어떤 Feature를 만들어야 하는지 사람이 결정하기 어려워졌다.

---

## Deep Learning의 등장

Deep Learning은 바로 이러한 문제를 해결하기 위해 등장하였다.

핵심 아이디어는

> 사람이 Feature를 만드는 대신, 모델이 Feature를 스스로 학습하도록 하자는 것이다.

동일한 데이터를 Deep Learning으로 학습하면 다음과 같은 구조가 된다.

```python
model = Sequential([
    Dense(16, activation="relu"),
    Dense(8, activation="relu"),
    Dense(1, activation="sigmoid")
])

model.compile(...)

model.fit(...)
```

구조는 다음과 같다.

```
Sensor Data

↓

Hidden Layer

↓

Hidden Layer

↓

Hidden Layer

↓

Prediction
```

여기서 중요한 점은

각 Hidden Layer가 점점 더 의미 있는 Feature를 자동으로 생성한다는 것이다.

즉,

```
Raw Data

↓

Feature 생성

↓

더 좋은 Feature 생성

↓

더 추상적인 Feature 생성

↓

Prediction
```

이라는 과정을 모델이 스스로 수행한다.

---

## Machine Learning과 Deep Learning 비교

| Machine Learning | Deep Learning |
|------------------|---------------|
| 사람이 Feature를 만든다 | 모델이 Feature를 학습한다 |
| 비교적 적은 데이터에서도 강하다 | 많은 데이터에서 강하다 |
| 모델 구조가 비교적 단순하다 | 여러 개의 Layer를 사용한다 |
| 학습 속도가 빠르다 | 학습 시간이 길다 |
| 해석이 상대적으로 쉽다 | 내부 동작을 해석하기 어렵다 |

---

## 같은 데이터를 학습하지만 접근 방식이 다르다

SVM도

```python
model.fit(...)
```

을 사용한다.

Deep Learning도

```python
model.fit(...)
```

을 사용한다.

겉으로 보기에는 거의 동일하다.

하지만 내부는 완전히 다르다.

SVM은

```
Data

↓

Optimization

↓

Decision Boundary
```

를 찾는다.

반면 Deep Learning은

```
Data

↓

Layer

↓

Layer

↓

Layer

↓

Prediction

↑

Loss

↑

Backpropagation
```

을 수백~수천 번 반복하면서 모든 가중치를 학습한다.

---

## Deep Learning의 Layer는 무엇을 하는가?

흥미로운 사실은 Deep Learning의 Layer 하나도 결국 다음과 같은 계산을 수행한다는 것이다.

```
y = Wx + b
```

즉,

Layer 하나는 선형 모델과 매우 유사한 계산을 수행한다.

차이점은

```
Linear

↓

Activation

↓

Linear

↓

Activation

↓

Linear
```

를 반복하면서 복잡한 비선형 문제를 해결할 수 있다는 점이다.

---

## 핵심 차이

Machine Learning은

```
Raw Data

↓

사람이 Feature 생성

↓

Machine Learning

↓

Prediction
```

반면

Deep Learning은

```
Raw Data

↓

Layer

↓

Layer

↓

Layer

↓

Prediction
```

과 같이 Feature 자체를 학습한다.

---

## 결론

Deep Learning은 SVM이나 기존 Machine Learning을 대체하기 위해 등장한 것이 아니다.

오히려 사람이 Feature를 설계하기 어려운 문제를 해결하기 위해 발전한 기술이다.

특히

- 이미지
- 음성
- 자연어
- 대규모 제조 데이터
- 반도체 공정 데이터

처럼 데이터가 매우 복잡한 환경에서는 사람이 직접 Feature를 만드는 것보다 Deep Learning이 Feature를 자동으로 학습하는 것이 훨씬 효과적인 경우가 많다.

---

## 한 문장으로 정리

> **Machine Learning은 사람이 만든 Feature로 학습하고, Deep Learning은 Feature까지 스스로 학습한다.**

이 한 문장이 두 접근법의 가장 큰 차이를 가장 잘 설명해 준다.

## 참고 (Feature Engineering)
Feature Engineering 은 입력 변수들(X 인자들) 에 대한 것으로 다음으로 분류되곤 한다.

![2026-08-06-164632.png](/assets/images/2026-08-06-164632.png)

### Correlation Analysis (상관 분석)

아래의 경우 RF Power 는 제거될 수도 있는 것으로 이 경우는 Feature Selection (특징 선택)이다.

| Feature     | Result와 상관계수 |
| ----------- | ------------ |
| Temperature | 0.92         |
| Pressure    | 0.87         |
| RF Power    | 0.18         |
| Gas Flow    | 0.81         |

### PCA (주성분 분석)

Temperature,Pressure,Voltage,Current,Humidity의 5개 Feature 의 경우 PCA 를 수행하면 PC1, PC2 처럼 새로운 Feature 로 압축될 수 있다. 이것은 기존 5개의 Feature 를 변환하는 것으로 이를 Feature Extraction(특징 추출)이라고 한다.

### Machine Learning vs Deep Learning

기존 Machine Learning은 사람이 Feature Engineering 과정에 적극적으로 관여하고, Deep Learning은 Feature Representation 자체를 모델이 학습한다.

### Domain Knowledge

사람이 관여하는 Machine Learning 은 결국 Domain Knowledge 의 엔지니어가 중요한 역할을 수행하지만, Deep Learning 은 이러한 것들을 Network Layer 가 데이터로부터 학습을 하는 것이 차이가 난다.

