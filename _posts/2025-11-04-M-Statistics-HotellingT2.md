---
title: Hotelling T2
img_path: /assets/images/
author: Alex
date: 2025-11-04
category: [Manufacturing, Statistics, Hotelling T2]
tags:
 - Hotelling T2
 - Control Chart
layout: post
---

## 1. Hotelling's $T^2$ 검정 개요
### 1.1 개념 정의
Hotelling's $T^2$ 검정(호텔링 $T^2$ 검정)은 단변량 통계 분석 기법인 Student's $t$-검정을 다변량(Multivariate) 데이터 영역으로 확장한 통계적 검정 방법입니다.반도체, 화학, 정밀 제조 공정과 같이 온도, 압력, RF Power 등 여러 수치형 변수가 동시에 상관관계를 갖고 변동하는 시스템에서, 단일 변수별로 분석하지 않고 여러 변수의 평균 벡터 및 변수 간 공분산(상관관계)을 통합하여 통계적 차이나 이상 발생 여부를 검정합니다.
### 1.2 단변량 $t$-검정과의 주요 차이점
단변량 $t$-검정: 한 번에 하나의 변수(예: 단일 공정 온도) 평균 및 변동만 개별적으로 비교합니다.Hotelling's $T^2$ 검정: 여러 변수(예: 온도, 압력, RF Power)의 평균 벡터와 변수 간 상관성(공분산 행렬)을 동시에 고려하여 시스템 전체의 이상을 한 번에 탐지합니다.
마하나라노비스 거리(Mahalanobis Distance) 활용: 데이터 간 단순 거리가 아닌, 변수 간 상관관계를 반영하여 중심점으로부터 개별 측정 데이터가 떨어진 정도를 정확하게 수치화합니다.

## 2. 수학적 수식 및 구조
### 2.1 $T^2$ 통계량 연산식
단일 표본 또는 특정 시점 $t$의 측정값 벡터 $\mathbf{x}_t = [\text{온도}_t, \text{압력}_t, \text{RF Power}_t]^T$에 대한 Hotelling's $T^2$ 통계량은 다음과 같은 이차형식(Quadratic Form) 수식으로 정의됩니다.$$T^2 = (\mathbf{x}_t - \bar{\mathbf{x}})^T \mathbf{S}^{-1} (\mathbf{x}_t - \bar{\mathbf{x}})$$$\bar{\mathbf{x}}$: 기준 공정(Phase I)에서의 변수별 평균 벡터 ($p \times 1$)$\mathbf{S}$: 기준 공정(Phase I)에서의 표본 공분산 행렬 ($p \times p$)$\mathbf{S}^{-1}$: 공분산 행렬의 역행렬 ($p \times p$)$p$: 공정 관리 변수의 개수
### 2.2 공분산 행렬 및 역행렬의 원시 연산 구조
원시 데이터(Raw Data)로부터 공분산 행렬 $\mathbf{S}$를 직접 산출하는 방식은 다음과 같습니다.
#### 1. 편차 행렬 산출: 전체 표본 데이터 $\mathbf{X}$에서 각 변수별 평균 벡터 $\bar{\mathbf{x}}$를 차감하여 중심화(Centered) 데이터를 구합니다.$$\mathbf{X}_{\text{centered}} = \mathbf{X} - \bar{\mathbf{x}}$$
#### 2. 공분산 행렬 계산:
$$\mathbf{S} = \frac{\mathbf{X}_{\text{centered}}^T \mathbf{X}_{\text{centered}}}{n - 1}$$
#### 3. 이차형식 스칼라 변환: 
각 샘플의 편차 벡터 $(\mathbf{x}_i - \bar{\mathbf{x}})$에 역행렬 $\mathbf{S}^{-1}$을 곱하여, 다변량 데이터를 단 1개의 $T^2$ 스칼라 지표로 축쇄합니다.

## 3. Hotelling's $T^2$ 관리도 (Control Chart)
### 3.1 관리도의 원리 및 역할
다변량 공정 관리(MSPC, Multivariate Statistical Process Control)에서는 개별 변수별 관리도(Shewhart Chart)를 다수 사용할 경우 1종 오류(False Alarm)가 누적되거나, 변수 간의 잔여 상관관계가 깨지는 이상을 탐지할 수 없습니다.Hotelling's $T^2$ 관리도는 공정 내 수많은 변수를 하나로 통합한 $T^2$ 지표를 시간 흐름에 따라 추적 시각화합니다.하한 관리한계선 (LCL, Lower Control Limit): $T^2$ 값은 제곱 연산 특성상 항상 양수이므로 $0$입니다.상한 관리한계선 (UCL, Upper Control Limit): $F$-분포를 바탕으로 지정된 유의수준(예: $\alpha = 0.01$, 신뢰수준 99%)에 맞추어 설정합니다.

### 3.2 상한 관리한계선(UCL) 산출식
기준 데이터(Phase I, 표본 수 $m$, 변수 개수 $p$) 기반으로 실시간 모니터링(Phase II)을 진행할 때의 UCL 수식은 다음과 같습니다.$$\text{UCL} = \frac{p(m + 1)(m - 1)}{m(m - p)} F_{\alpha, p, m - p}$$$F_{\alpha, p, m - p}$: 자유도가 $(p, m - p)$이고 상위 꼬리 확률이 $\alpha$인 $F$-분포의 임계값

## 4. 간단 예제
```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats

# ==========================================
# 1. 원시 Raw Data 생성 (가상 공정 센서 데이터)
# ==========================================
np.random.seed(42)

# 공정 인자: [온도(Temp), 압력(Press), RF Power(Power)]
p_vars = ['Temp', 'Press', 'Power']

# [Phase I] 정상 공정 Raw Data 생성 (100개 샘플)
mean_true = [150.0, 10.0, 300.0]
cov_true = [[4.0, 0.8, 1.2], 
            [0.8, 0.25, 0.3], 
            [1.2, 0.3, 9.0]]

df_raw_phase1 = pd.DataFrame(
    np.random.multivariate_normal(mean_true, cov_true, size=100), 
    columns=p_vars
)

# [Phase II] 실시간 모니터링 Raw Data 생성 (50개 샘플)
# 1~30번: 정상 공정 / 31~50번: 이상 발생 공정 (Power 상승 및 연동 변동)
raw_normal = np.random.multivariate_normal(mean_true, cov_true, size=30)
raw_abnormal = np.random.multivariate_normal([151.0, 10.1, 309.0], cov_true, size=20)

df_raw_phase2 = pd.DataFrame(
    np.vstack([raw_normal, raw_abnormal]), 
    columns=p_vars
)


# ==========================================
# 2. Raw Data로부터 통계량 직접 계산 (Phase I)
# ==========================================
print("=== Step 1. Phase I Raw Data로부터 모수 추정 ===")

# (1) 평균 벡터 계산 (\bar{x})
X_p1 = df_raw_phase1.values
n_p1, p = X_p1.shape
mean_vec = np.mean(X_p1, axis=0)

# (2) 공분산 행렬 직접 계산 (S)
# 수식: S = (X - \bar{x})^T (X - \bar{x}) / (n - 1)
X_centered = X_p1 - mean_vec
cov_mat = (X_centered.T @ X_centered) / (n_p1 - 1)

# (3) 공분산 행렬의 역행렬 계산 (S^-1)
cov_inv = np.linalg.inv(cov_mat)

print(f"■ 추정된 평균 벡터 (\bar{{x}}):\n {mean_vec}")
print(f"■ 추정된 공분산 행렬 (S):\n {cov_mat}")
print(f"■ 공분산 역행렬 (S^-1):\n {cov_inv}\n")


# ==========================================
# 3. Phase II 실시간 $T^2$ 산출 및 관리한계선(UCL)
# ==========================================
X_p2 = df_raw_phase2.values
n_p2 = len(X_p2)

# (1) 각 샘플별 T^2 통계량 연산
# 수식: T^2 = (x_i - \bar{x})^T * S^-1 * (x_i - \bar{x})
t2_values = []
for i in range(n_p2):
    diff = X_p2[i] - mean_vec
    t2 = diff.T @ cov_inv @ diff
    t2_values.append(t2)

t2_values = np.array(t2_values)

# (2) 상한 관리한계선(UCL) 계산 (F-분포 기반)
alpha = 0.01  # 신뢰수준 99%
f_crit = stats.f.ppf(1 - alpha, p, n_p1 - p)
UCL = (p * (n_p1 + 1) * (n_p1 - 1)) / (n_p1 * (n_p1 - p)) * f_crit

print(f"=== Step 2. 관리 한계선 설정 ===")
print(f"■ 상한 관리한계선 (UCL, alpha=0.01): {UCL:.4f}\n")


# ==========================================
# 4. 변수별 기여도(Contribution) 산출
# ==========================================
# 이상치 탐지 시 어떤 인자(Temp, Press, Power)가 T^2를 높였는지 분해
def calculate_contribution(x_sample, mean_vec, cov_inv):
    diff = x_sample - mean_vec
    # d_j * (S^-1 * d)_j
    contributions = diff * (cov_inv @ diff)
    return contributions

# 이상이 탐지된 샘플들의 평균 기여도 산출
outlier_indices = np.where(t2_values > UCL)[0]
if len(outlier_indices) > 0:
    outlier_contributions = [calculate_contribution(X_p2[i], mean_vec, cov_inv) for i in outlier_indices]
    avg_contribution = np.mean(outlier_contributions, axis=0)


# ==========================================
# 5. 시각화 (Control Chart & Contribution Plot)
# ==========================================
fig, ax = plt.subplots(1, 2, figsize=(15, 5))

# (Chart 1) Hotelling T2 Control Chart
ax[0].plot(range(1, n_p2 + 1), t2_values, marker='o', color='b', label='$T^2$ Stat')
ax[0].axhline(y=UCL, color='r', linestyle='--', label=f'UCL ({UCL:.2f})')
ax[0].axvline(x=30, color='gray', linestyle=':', label='Fault Occurred')
ax[0].plot(outlier_indices + 1, t2_values[outlier_indices], 'ro', label='Out of Control')
ax[0].set_title("Hotelling $T^2$ Control Chart (From Raw Data)")
ax[0].set_xlabel("Sample Number")
ax[0].set_ylabel("$T^2$ Value")
ax[0].grid(True, alpha=0.3)
ax[0].legend()

# (Chart 2) Variable Contribution Plot (이상 발생 구간)
if len(outlier_indices) > 0:
    ax[1].bar(p_vars, avg_contribution, color=['skyblue', 'orange', 'crimson'])
    ax[1].set_title("Out-of-Control Contribution Plot")
    ax[1].set_xlabel("Process Variables")
    ax[1].set_ylabel("Average Contribution to $T^2$")
    ax[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```
```bash
(alex) raymond@ASUS:~/stats$ /home/raymond/anaconda3/envs/alex/bin/python /home/raymond/stats/HotellingT2.py
=== Step 1. Phase I Raw Data로부터 모수 추정 ===
■ 추정된 평균 벡터 ar{x}):
 [150.27522865  10.07583855 299.6435186 ]
■ 추정된 공분산 행렬 (S):
 [[3.55998239 0.8001149  0.35462088]
 [0.8001149  0.29037817 0.1911418 ]
 [0.35462088 0.1911418  6.28772052]]
■ 공분산 역행렬 (S^-1):
 [[ 0.7404541  -2.05387434  0.02067534]
 [-2.05387434  9.21114766 -0.16417533]
 [ 0.02067534 -0.16417533  0.16286488]]

=== Step 2. 관리 한계선 설정 ===
■ 상한 관리한계선 (UCL, alpha=0.01): 12.3395
```
![2026-08-05-104829.png](/assets/images/2026-08-05-104829.png)


## 5. 어느 데이터가 가장 기여를 했나 ?

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats

# ==========================================
# 1. Raw Data 생성 (Phase I 정상 / Phase II 이상)
# ==========================================
np.random.seed(42)

p_vars = ['Temp', 'Press', 'Power']
p = len(p_vars)

mean_true = [150.0, 10.0, 300.0]
cov_true = [[4.0, 0.8, 1.2], 
            [0.8, 0.25, 0.3], 
            [1.2, 0.3, 9.0]]

# Phase I 정상 데이터 (100개)
df_phase1 = pd.DataFrame(
    np.random.multivariate_normal(mean_true, cov_true, size=100), 
    columns=p_vars
)

# Phase II 실시간 모니터링 데이터 (50개: 1~30번 정상 / 31~50번 Power 및 온도 이상)
raw_normal = np.random.multivariate_normal(mean_true, cov_true, size=30)
raw_abnormal = np.random.multivariate_normal([152.0, 10.1, 312.0], cov_true, size=20)
df_phase2 = pd.DataFrame(np.vstack([raw_normal, raw_abnormal]), columns=p_vars)


# ==========================================
# 2. Raw Data 통계량 및 T2 계산
# ==========================================
X_p1 = df_phase1.values
n_p1 = len(X_p1)

# Phase I 기준 매개변수 (평균 벡터, 공분산 행렬, 역행렬)
mean_vec = np.mean(X_p1, axis=0)
X_centered = X_p1 - mean_vec
cov_mat = (X_centered.T @ X_centered) / (n_p1 - 1)
cov_inv = np.linalg.inv(cov_mat)

# Phase II T2 통계량 산출
X_p2 = df_phase2.values
n_p2 = len(X_p2)

t2_values = []
for i in range(n_p2):
    diff = X_p2[i] - mean_vec
    t2 = diff.T @ cov_inv @ diff
    t2_values.append(t2)

t2_values = np.array(t2_values)

# 상한 관리한계선(UCL, alpha=0.01)
alpha = 0.01
f_crit = stats.f.ppf(1 - alpha, p, n_p1 - p)
UCL = (p * (n_p1 + 1) * (n_p1 - 1)) / (n_p1 * (n_p1 - p)) * f_crit


# ==========================================
# 3. 이상 샘플들의 변수별 기여도(Contribution) 산출
# ==========================================
def calculate_contribution(x_sample, mean_vec, cov_inv):
    diff = x_sample - mean_vec
    # d_j * (S^-1 * d)_j
    return diff * (cov_inv @ diff)

outlier_indices = np.where(t2_values > UCL)[0]

if len(outlier_indices) > 0:
    outlier_contributions = [calculate_contribution(X_p2[i], mean_vec, cov_inv) for i in outlier_indices]
    avg_contribution = np.mean(outlier_contributions, axis=0)
else:
    avg_contribution = np.zeros(p)


# ==========================================
# 4. 시각화 (T2 Control Chart & Radar Contribution Chart)
# ==========================================
fig = plt.figure(figsize=(15, 6))

# --- (1) 일반 T2 관리도 ---
ax1 = fig.add_subplot(1, 2, 1)
ax1.plot(range(1, n_p2 + 1), t2_values, marker='o', color='b', label='$T^2$ Stat')
ax1.axhline(y=UCL, color='r', linestyle='--', label=f'UCL ({UCL:.2f})')
ax1.axvline(x=30, color='gray', linestyle=':', label='Fault Occurred')
ax1.plot(outlier_indices + 1, t2_values[outlier_indices], 'ro', label='Out of Control')
ax1.set_title("Hotelling $T^2$ Control Chart", fontsize=13)
ax1.set_xlabel("Sample Number")
ax1.set_ylabel("$T^2$ Value")
ax1.grid(True, alpha=0.3)
ax1.legend()


# --- (2) 원형 레이더 차트 (Polar Radar Chart) ---
ax2 = fig.add_subplot(1, 2, 2, polar=True)

# 레이더 차트 각도 계산 (원형으로 닫히도록 첫 번째 각도를 끝에 추가)
angles = np.linspace(0, 2 * np.pi, p, endpoint=False).tolist()
angles += angles[:1]

# 기여도 데이터 원형으로 닫기
radar_values = avg_contribution.tolist()
radar_values += radar_values[:1]

# 변수 라벨 배치
labels = p_vars + [p_vars[0]]

# 레이더 축 및 라벨 설정
ax2.set_theta_offset(np.pi / 2)  # 시작 각도를 12시 방향(북쪽)으로 설정
ax2.set_theta_direction(-1)       # 시계 방향으로 각도 증가
plt.xticks(angles[:-1], p_vars, fontsize=11, fontweight='bold')

# 데이터 다각형 묘사 및 영역 색상 채우기
ax2.plot(angles, radar_values, color='crimson', linewidth=2, linestyle='solid', label='Avg Contribution')
ax2.fill(angles, radar_values, color='crimson', alpha=0.25)

# 레이더 배경 가이드라인 설정
ax2.set_rlabel_position(0)
ax2.grid(True, linestyle='--', alpha=0.6)
ax2.set_title("Out-of-Control Radar Contribution Plot", fontsize=13, pad=20)
ax2.legend(loc='upper right', bbox_to_anchor=(1.3, 1.1))

plt.tight_layout()
plt.show()
```
![2026-08-05-105251.png](/assets/images/2026-08-05-105251.png)

### 설명

개별적으로는 단변량일 경우 문제가 없을 수 있지만, 이 Hotelling T2 로 보면 다르다.  그리고 어떤 변수가 가장 큰 영향을 끼친 것을 우측의 Radar 로 볼 수 있다.  Power 가 가장 큰 영향을 끼쳤다.


## 이상 발생 기여도 예제
```python

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats

# ==========================================
# 1. Raw Data 생성 (Phase I 정상 / Phase II 이상)
# ==========================================
np.random.seed(42)

p_vars = ['Temp', 'Press', 'Power']
p = len(p_vars)

mean_true = [150.0, 10.0, 300.0]
cov_true = [[4.0, 0.8, 1.2], 
            [0.8, 0.25, 0.3], 
            [1.2, 0.3, 9.0]]

# Phase I 정상 데이터 (100개)
df_phase1 = pd.DataFrame(
    np.random.multivariate_normal(mean_true, cov_true, size=100), 
    columns=p_vars
)

# Phase II 실시간 모니터링 데이터 (50개: 1~30번 정상 / 31~50번 Power 및 온도 이상)
raw_normal = np.random.multivariate_normal(mean_true, cov_true, size=30)
raw_abnormal = np.random.multivariate_normal([152.0, 10.1, 312.0], cov_true, size=20)
df_phase2 = pd.DataFrame(np.vstack([raw_normal, raw_abnormal]), columns=p_vars)


# ==========================================
# 2. Raw Data 통계량 및 T2 계산
# ==========================================
X_p1 = df_phase1.values
n_p1 = len(X_p1)

# Phase I 기준 매개변수 (평균 벡터, 공분산 행렬, 역행렬)
mean_vec = np.mean(X_p1, axis=0)
X_centered = X_p1 - mean_vec
cov_mat = (X_centered.T @ X_centered) / (n_p1 - 1)
cov_inv = np.linalg.inv(cov_mat)

# Phase II T2 통계량 산출
X_p2 = df_phase2.values
n_p2 = len(X_p2)

t2_values = []
for i in range(n_p2):
    diff = X_p2[i] - mean_vec
    t2 = diff.T @ cov_inv @ diff
    t2_values.append(t2)

t2_values = np.array(t2_values)

# 상한 관리한계선(UCL, alpha=0.01)
alpha = 0.01
f_crit = stats.f.ppf(1 - alpha, p, n_p1 - p)
UCL = (p * (n_p1 + 1) * (n_p1 - 1)) / (n_p1 * (n_p1 - p)) * f_crit


# ==========================================
# 3. 이상 샘플들의 변수별 기여도(Contribution) 산출
# ==========================================
def calculate_contribution(x_sample, mean_vec, cov_inv):
    diff = x_sample - mean_vec
    # d_j * (S^-1 * d)_j
    return diff * (cov_inv @ diff)

outlier_indices = np.where(t2_values > UCL)[0]

if len(outlier_indices) > 0:
    outlier_contributions = [calculate_contribution(X_p2[i], mean_vec, cov_inv) for i in outlier_indices]
    avg_contribution = np.mean(outlier_contributions, axis=0)
else:
    avg_contribution = np.zeros(p)


# ==========================================
# 4. 시각화 (T2 Control Chart & Bar Contribution Chart)
# ==========================================
fig, ax = plt.subplots(1, 2, figsize=(15, 5))

# --- (1) 일반 T2 관리도 ---
ax[0].plot(range(1, n_p2 + 1), t2_values, marker='o', color='b', label='$T^2$ Stat')
ax[0].axhline(y=UCL, color='r', linestyle='--', label=f'UCL ({UCL:.2f})')
ax[0].axvline(x=30, color='gray', linestyle=':', label='Fault Occurred')
ax[0].plot(outlier_indices + 1, t2_values[outlier_indices], 'ro', label='Out of Control')
ax[0].set_title("Hotelling $T^2$ Control Chart", fontsize=13)
ax[0].set_xlabel("Sample Number")
ax[0].set_ylabel("$T^2$ Value")
ax[0].grid(True, alpha=0.3)
ax[0].legend()


# --- (2) 변수별 기여도 막대그래프 (Bar Contribution Plot) ---
colors = ['#4C72B0', '#DD8452', '#C44E52']  # Temp, Press, Power 구분 색상
bars = ax[1].bar(p_vars, avg_contribution, color=colors, edgecolor='black', alpha=0.85)

# 수치 데이터 막대 위에 표시
for bar in bars:
    height = bar.get_height()
    ax[1].annotate(f'{height:.2f}',
                    xy=(bar.get_x() + bar.get_width() / 2, height),
                    xytext=(0, 3),  # 3pt vertical offset
                    textcoords="offset points",
                    ha='center', va='bottom', fontsize=10, fontweight='bold')

ax[1].set_title("Out-of-Control Contribution Plot (Bar Chart)", fontsize=13)
ax[1].set_xlabel("Process Variables", fontsize=11)
ax[1].set_ylabel("Average Contribution to $T^2$", fontsize=11)
ax[1].grid(True, axis='y', linestyle='--', alpha=0.5)

plt.tight_layout()
plt.show()

```
이상발생에 대한 기여도를 보여준다.
![2026-08-05-110454.png](/assets/images/2026-08-05-110454.png)