---
title: Statistics Approach
img_path: /assets/images/
author: Alex
date: 2025-10-28
category: [Manufacturing, Statistics, Statistics Approach]
tags:
 - Statistics
 - ANOVA

layout: post
---

## 개요
### 제조 데이터 분석에서 적절한 통계 검정을 선택하는 방법

제조 현장에서는 매일 수많은 데이터가 생성됩니다. 반도체 공정, 자동차 제조, 디스플레이 생산, 화학 공정 등 다양한 산업에서는 설비에서 수집되는 센서 데이터와 품질 검사 결과를 이용하여 공정을 개선하고 불량을 줄이는 작업이 이루어집니다.

하지만 데이터를 수집했다고 해서 바로 통계 분석을 수행할 수 있는 것은 아닙니다. 가장 먼저 결정해야 할 사항은 **어떤 통계 기법을 적용해야 하는가**입니다.

예를 들어 다음과 같은 질문을 생각해 볼 수 있습니다.

* 설비 A와 설비 B의 평균 두께는 동일한가?
* 네 개의 생산 라인 중 어느 라인의 품질이 가장 우수한가?
* 새로운 공정 조건이 기존 공정보다 실제로 개선되었는가?
* 불량 유형의 발생 비율은 생산 라인마다 차이가 있는가?

이러한 질문은 모두 통계적으로 검정할 수 있지만, **모든 문제에 동일한 검정을 사용할 수는 없습니다.**

데이터가 연속형인지 범주형인지, 비교 대상이 두 그룹인지 세 그룹 이상인지, 데이터가 정규분포를 따르는지, 그룹 간 분산이 동일한지에 따라 적용해야 하는 통계 검정이 달라집니다. 잘못된 검정을 선택하면 분석 결과가 왜곡되거나 잘못된 의사결정을 내릴 수도 있습니다.

이번 글에서는 제조 데이터를 분석할 때 가장 많이 사용하는 통계 검정들을 하나의 의사결정 트리(Decision Tree)로 정리해 보겠습니다. 이 트리를 따라가면 현재 가지고 있는 데이터의 특성에 맞는 검정 방법을 쉽게 선택할 수 있으며, 이후 Python을 이용한 실습으로도 자연스럽게 연결할 수 있습니다.

다음 그림은 제조 데이터 분석에서 통계 검정을 선택하는 전체 흐름을 나타낸 것입니다.

![2026-08-05-094133.png](/assets/images/2026-08-05-094133.png)


## 2 대의 설비, 정규분포, 등분산성의 경우 (Sample T-Test)
설비 데이터는 난수를 가지고 만들었음. 흐름 파악이 우선임.

```python
import numpy as np
import pandas as pd
from scipy import stats

# 1. 재현성을 위한 난수 시드 설정 및 2개 설비 데이터 생성
np.random.seed(42)

# 설비 A, B 데이터 생성 (정규분포를 따르며, 분산이 유사하도록 설정)
# target_mean_A = 10.0, target_mean_B = 10.2, std = 0.5, n = 35
data_A = np.random.normal(loc=10.0, scale=0.5, size=35)
data_B = np.random.normal(loc=10.2, scale=0.5, size=35)

df = pd.DataFrame({
    'Facility_A': data_A,
    'Facility_B': data_B
})

print("=== 1. 데이터 기초 통계량 ===")
print(f"설비 A - 평균: {np.mean(data_A):.4f}, 표준편차: {np.std(data_A, ddof=1):.4f}")
print(f"설비 B - 평균: {np.mean(data_B):.4f}, 표준편차: {np.std(data_B, ddof=1):.4f}")
print("-" * 50)

# 2. Shapiro-Wilk 정규성 검정 (Normality Test)
# 귀무가설(H0): 데이터가 정규분포를 따른다. (p >= 0.05 이면 정규성 만족)
stat_A, p_val_A = stats.shapiro(data_A)
stat_B, p_val_B = stats.shapiro(data_B)

print("=== 2. Shapiro-Wilk 정규성 검정 ===")
print(f"설비 A: W-stat = {stat_A:.4f}, p-value = {p_val_A:.4f}")
print(f"설비 B: W-stat = {stat_B:.4f}, p-value = {p_val_B:.4f}")

is_normal_A = p_val_A >= 0.05
is_normal_B = p_val_B >= 0.05

if is_normal_A and is_normal_B:
    print("▶ 판정: 두 설비 데이터 모두 정규성 만족 (Yes)")
else:
    print("▶ 판정: 정규성 미만족 (No)")
print("-" * 50)

# 3. Levene 등분산성 검정 (Homogeneity of Variance Test)
# 귀무가설(H0): 두 집단의 분산이 같다. (p >= 0.05 이면 등분산성 만족)
stat_lev, p_val_lev = stats.levene(data_A, data_B)

print("=== 3. Levene 등분산성 검정 ===")
print(f"Levene-stat = {stat_lev:.4f}, p-value = {p_val_lev:.4f}")

is_equal_var = p_val_lev >= 0.05

if is_equal_var:
    print("▶ 판정: 등분산성 만족 (Yes)")
else:
    print("▶ 판정: 등분산성 미만족 (No)")
print("-" * 50)

# 4. 2-Sample t-test 수행 (등분산성 만족 시 equal_var=True)
if is_normal_A and is_normal_B and is_equal_var:
    # equal_var=True -> Student's t-test 수행
    t_stat, p_val_ttest = stats.ttest_ind(data_A, data_B, equal_var=True)
    
    print("=== 4. 2-Sample t-test 결과 ===")
    print(f"t-statistic = {t_stat:.4f}")
    print(f"p-value     = {p_val_ttest:.6f}")
    
    alpha = 0.05
    if p_val_ttest < alpha:
        print(f"▶ 최종 결론: p-value < {alpha} 이므로, 두 설비 간 평균에 통계적으로 유의미한 차이가 있습니다.")
    else:
        print(f"▶ 최종 결론: p-value >= {alpha} 이므로, 두 설비 간 평균 차이가 유의미하지 않습니다.")
```
결과는 다음과 같다.
```bash
(alex) raymond@ASUS:~/stats$ /home/raymond/anaconda3/envs/alex/bin/python /home/raymond/stats/SampleTTest.py
=== 1. 데이터 기초 통계량 ===
설비 A - 평균: 9.9337, 표준편차: 0.4659
설비 B - 평균: 10.1316, 표준편차: 0.4399
--------------------------------------------------
=== 2. Shapiro-Wilk 정규성 검정 ===
설비 A: W-stat = 0.9783, p-value = 0.7043
설비 B: W-stat = 0.9700, p-value = 0.4436
▶ 판정: 두 설비 데이터 모두 정규성 만족 (Yes)
--------------------------------------------------
=== 3. Levene 등분산성 검정 ===
Levene-stat = 0.0014, p-value = 0.9699
▶ 판정: 등분산성 만족 (Yes)
--------------------------------------------------
=== 4. 2-Sample t-test 결과 ===
t-statistic = -1.8274
p-value     = 0.072025
▶ 최종 결론: p-value >= 0.05 이므로, 두 설비 간 평균 차이가 유의미하지 않습니다.
```
## 2 대의 설비, Mann-Whitney U Test
2개 설비의 데이터가 Shapiro-Wilk 정규성 검정을 통과하지 못해(P < 0.05), 비모수 검정 기법인 Mann-Whitney U 검정(Mann-Whitney U Test)을 수행하는 완성형 Python 소스 코드입니다. 정규성 검정에서 "미만족(No)" 판정이 나오도록 일부러 치우친 데이터(지수분포/감마분포 형태의 비정규 데이터)를 생성하도록 난수 시드를 설정했음.

```python
import numpy as np
import pandas as pd
from scipy import stats

# 1. 재현성을 위한 난수 시드 설정 및 비정규 분포(지수분포) 데이터 생성
np.random.seed(101)

# 치우침이 심한 데이터 생성 (지수분포: Exponential Distribution)
# 설비 A, B 간의 위치(중위수) 차이를 두기 위해 scale 파라미터 조정
data_A = np.random.exponential(scale=1.0, size=35)
data_B = np.random.exponential(scale=1.8, size=35)

df = pd.DataFrame({
    'Facility_A': data_A,
    'Facility_B': data_B
})

print("=== 1. 데이터 기초 통계량 (비모수는 '중위수'가 핵심) ===")
print(f"설비 A - 평균: {np.mean(data_A):.4f}, 중위수(Median): {np.median(data_A):.4f}")
print(f"설비 B - 평균: {np.mean(data_B):.4f}, 중위수(Median): {np.median(data_B):.4f}")
print("-" * 55)

# 2. Shapiro-Wilk 정규성 검정 (Normality Test)
# 귀무가설(H0): 데이터가 정규분포를 따른다. (p < 0.05 이면 정규성 미만족)
stat_A, p_val_A = stats.shapiro(data_A)
stat_B, p_val_B = stats.shapiro(data_B)

print("=== 2. Shapiro-Wilk 정규성 검정 ===")
print(f"설비 A: W-stat = {stat_A:.4f}, p-value = {p_val_A:.6f}")
print(f"설비 B: W-stat = {stat_B:.4f}, p-value = {p_val_B:.6f}")

is_normal_A = p_val_A >= 0.05
is_normal_B = p_val_B >= 0.05

if not (is_normal_A and is_normal_B):
    print("▶ 판정: 정규성 미만족 (No) -> 모수 검정(t-test) 불가, 비모수 검정 선택")
else:
    print("▶ 판정: 정규성 만족 (Yes)")
print("-" * 55)

# 3. Mann-Whitney U 검정 수행 (비모수 검정)
# 귀무가설(H0): 두 설비 데이터의 분포/중위수에 차이가 없다.
# 대립가설(H1): 두 설비 데이터의 분포/중위수에 차이가 있다.
if not (is_normal_A and is_normal_B):
    u_stat, p_val_mw = stats.mannwhitneyu(data_A, data_B, alternative='two-sided')
    
    print("=== 3. Mann-Whitney U Test 결과 ===")
    print(f"U-statistic = {u_stat:.4f}")
    print(f"p-value     = {p_val_mw:.6f}")
    
    alpha = 0.05
    if p_val_mw < alpha:
        print(f"▶ 최종 결론: p-value < {alpha} 이므로, 두 설비 간 중위수(분포)에 통계적으로 유의미한 차이가 있습니다.")
    else:
        print(f"▶ 최종 결론: p-value >= {alpha} 이므로, 두 설비 간 유의미한 차이가 없습니다.")
```
```bash
-------------------------------------------------------
=== 2. Shapiro-Wilk 정규성 검정 ===
설비 A: W-stat = 0.7981, p-value = 0.000019
설비 B: W-stat = 0.8554, p-value = 0.000306
▶ 판정: 정규성 미만족 (No) -> 모수 검정(t-test) 불가, 비모수 검정 선택
-------------------------------------------------------
=== 3. Mann-Whitney U Test 결과 ===
U-statistic = 516.0000
p-value     = 0.259478
▶ 최종 결론: p-value >= 0.05 이므로, 두 설비 간 유의미한 차이가 없습니다.
```

## 2 대의 설비, 정규분포 그러나, 등분산성 아님 Welch's T-Test
2개 설비 데이터가 Shapiro-Wilk 정규성 검정은 통과($P \ge 0.05$)했지만, Levene 등분산성 검정에서 실패($P < 0.05$)하여 Welch's t-test(웰치의 t-검정)를 수행하는 완성형 Python 소스 코드입니다.실행 시 등분산성이 미만족(No)되도록 두 설비의 표준편차(산포)를 다르게 설정하였습니다.

```python
import numpy as np
import pandas as pd
from scipy import stats

# 1. 난수 시드 및 파라미터 조정
# seed를 123으로 변경하고, 표본 수(size)를 50으로 설정
np.random.seed(123)

# 설비 A: 산포가 작음 (std=0.2)
# 설비 B: 산포가 큼 (std=0.8)
data_A = np.random.normal(loc=10.0, scale=0.2, size=50)
data_B = np.random.normal(loc=10.2, scale=0.8, size=50)

df = pd.DataFrame({
    'Facility_A': data_A,
    'Facility_B': data_B
})

print("=== 1. 데이터 기초 통계량 ===")
print(f"설비 A - 평균: {np.mean(data_A):.4f}, 표준편차(Std): {np.std(data_A, ddof=1):.4f}")
print(f"설비 B - 평균: {np.mean(data_B):.4f}, 표준편차(Std): {np.std(data_B, ddof=1):.4f}")
print("-" * 55)

# 2. Shapiro-Wilk 정규성 검정 (Normality Test)
stat_A, p_val_A = stats.shapiro(data_A)
stat_B, p_val_B = stats.shapiro(data_B)

print("=== 2. Shapiro-Wilk 정규성 검정 ===")
print(f"설비 A: W-stat = {stat_A:.4f}, p-value = {p_val_A:.4f}")
print(f"설비 B: W-stat = {stat_B:.4f}, p-value = {p_val_B:.4f}")

is_normal_A = p_val_A >= 0.05
is_normal_B = p_val_B >= 0.05

if is_normal_A and is_normal_B:
    print("▶ 판정: 두 설비 데이터 모두 정규성 만족 (Yes)")
else:
    print("▶ 판정: 정규성 미만족 (No)")
print("-" * 55)

# 3. Levene 등분산성 검정 (Homogeneity of Variance Test)
stat_lev, p_val_lev = stats.levene(data_A, data_B)

print("=== 3. Levene 등분산성 검정 ===")
print(f"Levene-stat = {stat_lev:.4f}, p-value = {p_val_lev:.6f}")

is_equal_var = p_val_lev >= 0.05

if is_equal_var:
    print("▶ 판정: 등분산성 만족 (Yes)")
else:
    print("▶ 판정: 등분산성 미만족 (No) -> Welch's t-test 실행 대상")
print("-" * 55)

# 4. Welch's t-test 수행 (조건 불일치 시 원인 출력 추가)
if is_normal_A and is_normal_B and (not is_equal_var):
    t_stat, p_val_welch = stats.ttest_ind(data_A, data_B, equal_var=False)
    
    print("=== 4. Welch's t-test 결과 ===")
    print(f"t-statistic = {t_stat:.4f}")
    print(f"p-value     = {p_val_welch:.6f}")
    
    alpha = 0.05
    if p_val_welch < alpha:
        print(f"▶ 최종 결론: p-value < {alpha} 이므로, 두 설비 간 평균에 통계적으로 유의미한 차이가 있습니다.")
    else:
        print(f"▶ 최종 결론: p-value >= {alpha} 이므로, 두 설비 간 평균 차이가 통계적으로 유의미하지 않습니다.")
else:
    print("=== 4. 검정 미실행 원인 ===")
    print(f"- 정규성 A 만족 여부: {is_normal_A} (p={p_val_A:.4f})")
    print(f"- 정규성 B 만족 여부: {is_normal_B} (p={p_val_B:.4f})")
    print(f"- 등분산성 미만족 여부: {not is_equal_var} (p={p_val_lev:.4f})")
```
```bash
▶ 판정: 두 설비 데이터 모두 정규성 만족 (Yes)
-------------------------------------------------------
=== 3. Levene 등분산성 검정 ===
Levene-stat = 75.3193, p-value = 0.000000
▶ 판정: 등분산성 미만족 (No) -> Welch's t-test 실행 대상
-------------------------------------------------------
=== 4. Welch's t-test 결과 ===
t-statistic = -1.8238
p-value     = 0.073460
▶ 최종 결론: p-value >= 0.05 이므로, 두 설비 간 평균 차이가 통계적으로 유의미하지 않습니다.
```

## One Way ANOVA
3개 설비(A, B, C) 데이터를 생성하여 Shapiro-Wilk 정규성 검정 통과($P \ge 0.05$), Levene 등분산성 검정 통과($P \ge 0.05$) 후 One-Way ANOVA를 수행하고, 유의미한 차이가 있을 경우 Tukey HSD 사후 분석까지 진행하는 완성형 Python 소스 코드입니다.실행 시 3개 그룹 모두 정규성과 등분산성을 확실하게 통과하도록 무작위 난수 시드(np.random.seed(303)) 및 파라미터를 맞추어 작성했습니다.

```python
import numpy as np
import pandas as pd
from scipy import stats
from statsmodels.stats.multicomp import pairwise_tukeyhsd

# 1. 재현성을 위한 난수 시드 설정 및 3개 설비 데이터 생성
np.random.seed(303)

# 3개 설비 데이터 생성 (정규분포를 따르며, 분산이 유사하도록 설정)
# target_mean: A=10.0, B=10.15, C=10.30, std = 0.4, n = 40
data_A = np.random.normal(loc=10.0, scale=0.4, size=40)
data_B = np.random.normal(loc=10.15, scale=0.4, size=40)
data_C = np.random.normal(loc=10.30, scale=0.4, size=40)

# DataFrame 구성
df = pd.DataFrame({
    'Facility': ['Facility_A']*40 + ['Facility_B']*40 + ['Facility_C']*40,
    'Thickness': np.concatenate([data_A, data_B, data_C])
})

print("=== 1. 설비별 기초 통계량 ===")
for facility in ['Facility_A', 'Facility_B', 'Facility_C']:
    sub_data = df[df['Facility'] == facility]['Thickness']
    print(f"{facility} - 평균: {sub_data.mean():.4f}, 표준편차: {sub_data.std():.4f}")
print("-" * 60)

# 2. Shapiro-Wilk 정규성 검정 (Normality Test)
# 귀무가설(H0): 데이터가 정규분포를 따른다. (p >= 0.05 이면 정규성 만족)
print("=== 2. Shapiro-Wilk 정규성 검정 ===")
is_normal_all = True

for facility in ['Facility_A', 'Facility_B', 'Facility_C']:
    sub_data = df[df['Facility'] == facility]['Thickness']
    stat, p_val = stats.shapiro(sub_data)
    is_normal = p_val >= 0.05
    print(f"{facility}: W-stat = {stat:.4f}, p-value = {p_val:.4f} (만족: {is_normal})")
    if not is_normal:
        is_normal_all = False

if is_normal_all:
    print("▶ 판정: 모든 설비 데이터가 정규성을 만족합니다. (Yes)")
else:
    print("▶ 판정: 일부 설비 데이터가 정규성을 만족하지 않습니다. (No)")
print("-" * 60)

# 3. Levene 등분산성 검정 (Homogeneity of Variance Test)
# 귀무가설(H0): 모든 집단의 분산이 동일하다. (p >= 0.05 이면 등분산성 만족)
stat_lev, p_val_lev = stats.levene(data_A, data_B, data_C)

print("=== 3. Levene 등분산성 검정 ===")
print(f"Levene-stat = {stat_lev:.4f}, p-value = {p_val_lev:.4f}")

is_equal_var = p_val_lev >= 0.05

if is_equal_var:
    print("▶ 판정: 등분산성을 만족합니다. (Yes) -> One-Way ANOVA 실행")
else:
    print("▶ 판정: 등분산성을 만족하지 않습니다. (No) -> Welch's ANOVA 고려")
print("-" * 60)

# 4. One-Way ANOVA 수행 및 사후 분석 (Tukey HSD)
if is_normal_all and is_equal_var:
    # f_oneway 수행
    f_stat, p_val_anova = stats.f_oneway(data_A, data_B, data_C)
    
    print("=== 4. One-Way ANOVA 결과 ===")
    print(f"F-statistic = {f_stat:.4f}")
    print(f"p-value     = {p_val_anova:.6f}")
    
    alpha = 0.05
    if p_val_anova < alpha:
        print(f"▶ 최종 결론: p-value < {alpha} 이므로, 3개 설비 중 적어도 하나의 평균에 유의미한 차이가 있습니다.")
        
        # 5. 사후 분석 (Post-hoc Test) - Tukey HSD
        print("\n=== 5. Tukey HSD 사후 분석 (어느 설비 간에 차이가 있는지 확인) ===")
        tukey = pairwise_tukeyhsd(endog=df['Thickness'], groups=df['Facility'], alpha=0.05)
        print(tukey)
    else:
        print(f"▶ 최종 결론: p-value >= {alpha} 이므로, 3개 설비 간 평균 차이가 통계적으로 유의미하지 않습니다.")
else:
    print("=== 4. 검정 미실행 원인 ===")
    print(f"- 정규성 전원 만족 여부: {is_normal_all}")
    print(f"- 등분산성 만족 여부: {is_equal_var}")
```
```bash
(alex) raymond@ASUS:~/stats$ /home/raymond/anaconda3/envs/alex/bin/python /home/raymond/stats/OneWayAnova.py
=== 1. 설비별 기초 통계량 ===
Facility_A - 평균: 9.9478, 표준편차: 0.3945
Facility_B - 평균: 10.2426, 표준편차: 0.3732
Facility_C - 평균: 10.2769, 표준편차: 0.3240
------------------------------------------------------------
=== 2. Shapiro-Wilk 정규성 검정 ===
Facility_A: W-stat = 0.9881, p-value = 0.9428 (만족: True)
Facility_B: W-stat = 0.9644, p-value = 0.2363 (만족: True)
Facility_C: W-stat = 0.9448, p-value = 0.0503 (만족: True)
▶ 판정: 모든 설비 데이터가 정규성을 만족합니다. (Yes)
------------------------------------------------------------
=== 3. Levene 등분산성 검정 ===
Levene-stat = 1.5575, p-value = 0.2150
▶ 판정: 등분산성을 만족합니다. (Yes) -> One-Way ANOVA 실행
------------------------------------------------------------
=== 4. One-Way ANOVA 결과 ===
F-statistic = 9.8214
p-value     = 0.000114
▶ 최종 결론: p-value < 0.05 이므로, 3개 설비 중 적어도 하나의 평균에 유의미한 차이가 있습니다.

=== 5. Tukey HSD 사후 분석 (어느 설비 간에 차이가 있는지 확인) ===
    Multiple Comparison of Means - Tukey HSD, FWER=0.05    
===========================================================
  group1     group2   meandiff p-adj   lower  upper  reject
-----------------------------------------------------------
Facility_A Facility_B   0.2948 0.0013   0.101 0.4886   True
Facility_A Facility_C   0.3291 0.0003  0.1353 0.5229   True
Facility_B Facility_C   0.0343 0.9075 -0.1595 0.2281  False
-----------------------------------------------------------
```
결과에 대한 해석은 다음과 같다.
  > One-Way ANOVA: P-value(0.014169)가 0.05보다 작으므로 설비 간 평균 치수에 통계적으로 유의미한 차이가 존재합니다.

  > Tukey HSD 사후 분석: reject 열을 통해 Facility_A와 Facility_C 간에만 유의미한 평균 차이(reject = True)가 존재하고, A-B 및 B-C 간의 차이는 우연적 오차 범주 내에 있음을 해석할 수 있습니다.

  ## Welch's ANOVA

  ```python
import numpy as np
import pandas as pd
from scipy import stats
from itertools import combinations

# 1. 난수 시드 및 데이터 생성
np.random.seed(1234)

data_A = np.random.normal(loc=10.0, scale=0.2, size=50)
data_B = np.random.normal(loc=10.3, scale=0.8, size=50)
data_C = np.random.normal(loc=10.8, scale=1.5, size=50)

groups = {'Facility_A': data_A, 'Facility_B': data_B, 'Facility_C': data_C}

print("=== 1. 설비별 기초 통계량 ===")
for name, data in groups.items():
    print(f"{name} - 평균: {np.mean(data):.4f}, 표준편차: {np.std(data, ddof=1):.4f}")
print("-" * 65)

# 2. Shapiro-Wilk 정규성 검정
print("=== 2. Shapiro-Wilk 정규성 검정 ===")
is_normal_all = True
for name, data in groups.items():
    stat, p_val = stats.shapiro(data)
    is_normal = p_val >= 0.05
    print(f"{name}: W-stat = {stat:.4f}, p-value = {p_val:.4f} (만족: {is_normal})")
    if not is_normal:
        is_normal_all = False

print("-" * 65)

# 3. Levene 등분산성 검정
stat_lev, p_val_lev = stats.levene(data_A, data_B, data_C)
print("=== 3. Levene 등분산성 검정 ===")
print(f"Levene-stat = {stat_lev:.4f}, p-value = {p_val_lev:.6f}")

is_equal_var = p_val_lev >= 0.05
print(f"▶ 등분산성 만족 여부: {is_equal_var}")
print("-" * 65)


# --- Welch's ANOVA 함수 ---
def custom_welch_anova(*args):
    k = len(args)
    ni = np.array([len(a) for a in args])
    mi = np.array([np.mean(a) for a in args])
    vi = np.array([np.var(a, ddof=1) for a in args])
    
    wi = ni / vi
    w = np.sum(wi)
    
    m_prime = np.sum(wi * mi) / w
    
    df1 = k - 1
    numerator = np.sum(wi * (mi - m_prime) ** 2) / df1
    
    lambda_term = np.sum((1 - wi / w) ** 2 / (ni - 1))
    denominator = 1 + (2 * (k - 2) / (k ** 2 - 1)) * lambda_term
    
    f_stat = numerator / denominator
    df2 = (k ** 2 - 1) / (3 * lambda_term)
    p_val = stats.f.sf(f_stat, df1, df2)
    
    return f_stat, df1, df2, p_val


# --- Games-Howell 사후검정 함수 ---
def custom_games_howell(groups_dict, alpha=0.05):
    results = []
    group_names = list(groups_dict.keys())
    
    for g1, g2 in combinations(group_names, 2):
        x1, x2 = groups_dict[g1], groups_dict[g2]
        n1, n2 = len(x1), len(x2)
        m1, m2 = np.mean(x1), np.mean(x2)
        v1, v2 = np.var(x1, ddof=1), np.var(x2, ddof=1)
        
        diff = m1 - m2
        se = np.sqrt(v1 / n1 + v2 / n2)
        t_stat = np.abs(diff) / se
        
        # Welch-Satterthwaite 자유도
        df = (v1 / n1 + v2 / n2) ** 2 / ((v1 / n1) ** 2 / (n1 - 1) + (v2 / n2) ** 2 / (n2 - 1))
        
        # Studentized Range Distribution 기반 p-value
        k = len(groups_dict)
        q_stat = t_stat * np.sqrt(2)
        p_val = stats.studentized_range.sf(q_stat, k, df)
        
        results.append({
            'Group 1': g1, 'Group 2': g2,
            'Diff': round(diff, 4), 't-stat': round(t_stat, 4),
            'df': round(df, 2), 'p-value': round(p_val, 6),
            'Reject H0': p_val < alpha
        })
    return pd.DataFrame(results)


# 4. 검정 및 사후검정 무조건 실행
f_stat, df1, df2, p_val_welch = custom_welch_anova(data_A, data_B, data_C)

print("=== 4. Welch's ANOVA 결과 ===")
print(f"F-statistic : {f_stat:.4f}")
print(f"df1, df2    : {df1}, {df2:.2f}")
print(f"p-value     : {p_val_welch:.6f}")

if p_val_welch < 0.05:
    print("\n▶ 최종 결론: p-value < 0.05 이므로 설비 간 평균 차이가 통계적으로 유의미합니다.")
    print("\n=== 5. Games-Howell 사후검정 결과 ===")
    gh_df = custom_games_howell(groups)
    print(gh_df.to_string(index=False))
else:
    print("\n▶ 최종 결론: p-value >= 0.05 이므로 설비 간 평균 차이가 유의미하지 않습니다.")

```

```bash
(alex) raymond@ASUS:~/stats$ /home/raymond/anaconda3/envs/alex/bin/python /home/raymond/stats/WelchsANOVA.py
=== 1. 설비별 기초 통계량 ===
Facility_A - 평균: 10.0149, 표준편차: 0.1946
Facility_B - 평균: 10.2965, 표준편차: 0.8289
Facility_C - 평균: 10.9708, 표준편차: 1.4997
-----------------------------------------------------------------
=== 2. Shapiro-Wilk 정규성 검정 ===
Facility_A: W-stat = 0.9654, p-value = 0.1497 (만족: True)
Facility_B: W-stat = 0.9584, p-value = 0.0764 (만족: True)
Facility_C: W-stat = 0.9760, p-value = 0.3996 (만족: True)
-----------------------------------------------------------------
=== 3. Levene 등분산성 검정 ===
Levene-stat = 32.9290, p-value = 0.000000
▶ 등분산성 만족 여부: False
-----------------------------------------------------------------
=== 4. Welch's ANOVA 결과 ===
F-statistic : 12.3083
df1, df2    : 2, 69.76
p-value     : 0.000026

▶ 최종 결론: p-value < 0.05 이므로 설비 간 평균 차이가 통계적으로 유의미합니다.

=== 5. Games-Howell 사후검정 결과 ===
   Group 1    Group 2    Diff  t-stat    df  p-value  Reject H0
Facility_A Facility_B -0.2816  2.3388 54.38 0.058858      False
Facility_A Facility_C -0.9558  4.4694 50.65 0.000129       True
Facility_B Facility_C -0.6742  2.7823 76.38 0.018438       True
```

Games-Howell 사후검정 결과표에 표시되는 각 컬럼은 **"두 집단 간 평균 차이가 우연에 의한 것인지, 실제 통계적으로 의미 있는 차이인지"**를 판정하기 위한 통계량들입니다.

---

### 1. 각 컬럼의 통계적 의미

#### ① Group 1 & Group 2 (비교 대상)
* **의미**: 서로 비교를 수행하는 두 개의 집단(설비)입니다.
* **해석**: A와 B, A와 C, B와 C처럼 가능한 모든 쌍(Pair)에 대해 1:1 비교를 수행합니다.

#### ② Diff (Mean Difference, 평균 차이)
* **의미**: 두 집단의 샘플 평균 간 차이 (Group 1 평균 - Group 2 평균)입니다.
* **해석**: 
  * 양수(+): Group 1의 평균이 Group 2보다 높음을 의미합니다.
  * 음수(-): Group 1의 평균이 Group 2보다 낮음을 의미합니다.
  * *예시*: Diff가 `-0.9559`라면 Facility A의 평균이 Facility C보다 `0.9559`만큼 작다는 뜻입니다.

#### ③ t-stat (t-통계량)
* **의미**: 두 그룹 평균 차이를 표준 오차(Standard Error)로 나눈 값입니다.
  $$\text{t-stat} = \frac{|\bar{X}_1 - \bar{X}_2|}{\sqrt{\frac{S_1^2}{n_1} + \frac{S_2^2}{n_2}}}$$
* **해석**: 두 그룹 간 평균 차이가 표본의 표준오차 대비 얼마나 큰지를 나타냅니다. 이 값이 **크면 클수록 두 집단 간 평균 차이가 명확**하다는 것을 의미합니다.

#### ④ df (Degrees of Freedom, 수정 자유도)
* **의미**: Welch-Satterthwaite 공식을 사용해 보정한 자유도입니다.
* **해석**: Games-Howell 검정은 **두 집단의 분산이 서로 다르고 표본 크기가 달라도 적용 가능**하도록 설계되었기 때문에, 정수 형태의 일반 자유도가 아닌 **소수점을 포함한 자유도**가 산출됩니다.

#### ⑤ p-value (유의확률)
* **의미**: "두 집단의 실제 평균이 같다(귀무가설)"는 전제 하에, 현재 관측된 평균 차이 이상의 차이가 우연히 발생할 확률입니다.
* **특징**: 단순 t-검정을 여러 번 할 때 발생하는 **1종 오류(Type I error, 거짓 긍정) 누적 문제를 방지**하기 위해 Studentized Range 분포(q-분포) 기반으로 p-value를 상향 조정(보정)합니다.
* **해석**:
  * p-value < 0.05: 두 집단의 차이가 우연일 확률이 5% 미만이므로 **통계적으로 유의미한 차이가 있음**을 나타냅니다.
  * p-value >= 0.05: 두 집단 간 평균 차이가 우연히 발생했을 수 있으므로 **유의미한 차이가 없음**을 나타냅니다.

#### ⑥ Reject H0 (귀무가설 기각 여부)
* **의미**: 유의수준 alpha = 0.05 기준으로 귀무가설(H0: 두 집단의 평균은 같다)을 기각할지 여부입니다.
* **해석**:
  * **True**: p-value < 0.05 상황입니다. **"두 설비 간 평균 차이가 존재함"**을 의미합니다.
  * **False**: p-value >= 0.05 상황입니다. **"두 설비 간 평균 차이가 통계적으로 입증되지 않음"**을 의미합니다.

---

### 2. 사후검정 결과표 실전 해석 예시

다음과 같은 결과표가 나왔다고 가정해 보겠습니다.

| Group 1 | Group 2 | Diff | t-stat | df | p-value | Reject H0 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Facility_A** | **Facility_B** | -0.2816 | 2.3275 | 56.63 | 0.058319 | **False** |
| **Facility_A** | **Facility_C** | -0.9559 | 4.4751 | 51.68 | 0.000122 | **True** |
| **Facility_B** | **Facility_C** | -0.6743 | 2.8028 | 76.24 | 0.017234 | **True** |

#### 보고서용 종합 해석 예시

1. **Facility_A vs Facility_B** (`Reject H0 = False`)
   * p-value = 0.0583으로 유의수준 0.05보다 크므로, 두 설비 간 평균 차이(-0.2816)는 통계적으로 유의미하지 않습니다. (즉, 두 설비의 성능/수치는 서로 비슷합니다.)
2. **Facility_A vs Facility_C** (`Reject H0 = True`)
   * p-value = 0.0001로 유의수준 0.05보다 매우 작으므로, 두 설비 간 평균 차이(-0.9559)는 통계적으로 매우 유의미합니다. Facility C가 Facility A보다 유의하게 높은 평균을 가집니다.
3. **Facility_B vs Facility_C** (`Reject H0 = True`)
   * p-value = 0.0172로 유의수준 0.05보다 작으므로, 두 설비 간 평균 차이(-0.6743) 역시 통계적으로 유의미합니다.

> **최종 요약**: 설비 A와 B는 서로 유의미한 차이가 없지만, **설비 C는 A, B 두 설비 모두와 유의미한 차이를 보이며 확연히 높은 평균값을 나타냅니다.**

## Kruskal-Wallis & Dunn's Test 
3개 이상의 설비 데이터가 정규분포를 따르지 않을 때(Shapiro-Wilk 검정에서 $p < 0.05$), 비모수(Non-parametric) 통계 방법인 Kruskal-Wallis 검정과 사후검정인 Dunn's test를 수행하는 코드입니다.별도의 외부 패키지(scikit-posthocs 등) 설치 없이, numpy, pandas, scipy만 사용하여 실행할 수 있습니다.

```python
import numpy as np
import pandas as pd
from scipy import stats
from itertools import combinations

# 1. 난수 시드 및 데이터 생성 (비정규 분포 유도: Exponential 분포 사용)
np.random.seed(42)

# 서로 다른 척도(scale)를 부여하여 세 설비의 중앙값 차이를 생성
data_A = np.random.exponential(scale=1.0, size=50)
data_B = np.random.exponential(scale=1.5, size=50)
data_C = np.random.exponential(scale=2.5, size=50)

groups = {'Facility_A': data_A, 'Facility_B': data_B, 'Facility_C': data_C}

print("=== 1. 설비별 기초 통계량 (비모수: 중앙값 중심) ===")
for name, data in groups.items():
    print(f"{name} - 중앙값(Median): {np.median(data):.4f}, IQR: {stats.iqr(data):.4f}")
print("-" * 65)

# 2. Shapiro-Wilk 정규성 검정
print("=== 2. Shapiro-Wilk 정규성 검정 ===")
is_normal_any = False
for name, data in groups.items():
    stat, p_val = stats.shapiro(data)
    is_normal = p_val >= 0.05
    print(f"{name}: W-stat = {stat:.4f}, p-value = {p_val:.4f} (정규성 만족: {is_normal})")
    if is_normal:
        is_normal_any = True

print("-" * 65)


# --- 3. Pure Python / Scipy 기반 Dunn's Post-hoc Test 함수 ---
def custom_dunn_test(groups_dict, p_adjust='bonferroni', alpha=0.05):
    """Scipy 기반 Dunn's 사후검정 (Bonferroni p-value 보정 적용)"""
    group_names = list(groups_dict.keys())
    all_data = []
    group_labels = []
    
    for g_name, data in groups_dict.items():
        all_data.extend(data)
        group_labels.extend([g_name] * len(data))
    
    # 전체 데이터를 결합하여 순위(Rank) 부여 (동점 평균 처리)
    ranks = stats.rankdata(all_data)
    
    df_ranks = pd.DataFrame({'Group': group_labels, 'Rank': ranks})
    rank_means = df_ranks.groupby('Group')['Rank'].mean()
    n_i = df_ranks.groupby('Group')['Rank'].count()
    
    N = len(all_data)
    m = len(group_names)
    num_comparisons = m * (m - 1) / 2  # 총 비교 쌍의 수
    
    # 동점(Ties) 보정항 산출
    _, tie_counts = np.unique(ranks, return_counts=True)
    tie_sum = np.sum(tie_counts**3 - tie_counts)
    c_factor = 1 - (tie_sum / (N**3 - N)) if (N**3 - N) != 0 else 1
    
    results = []
    for g1, g2 in combinations(group_names, 2):
        r_diff = rank_means[g1] - rank_means[g2]
        
        # 표준 오차(Standard Error) 산출
        se = np.sqrt((N * (N + 1) / 12 - tie_sum / (12 * (N - 1))) * (1 / n_i[g1] + 1 / n_i[g2]))
        z_stat = np.abs(r_diff) / se
        
        # 양측 검정 p-value 산출
        p_unadj = 2 * (1 - stats.norm.cdf(z_stat))
        
        # Bonferroni 보정
        if p_adjust == 'bonferroni':
            p_adj = min(1.0, p_unadj * num_comparisons)
        else:
            p_adj = p_unadj
            
        results.append({
            'Group 1': g1,
            'Group 2': g2,
            'Rank Diff': round(r_diff, 4),
            'z-stat': round(z_stat, 4),
            'p-unadj': round(p_unadj, 6),
            'p-adj (Bonferroni)': round(p_adj, 6),
            'Reject H0': p_adj < alpha
        })
        
    return pd.DataFrame(results)


# 4. 검정 및 사후검정 실행
print("=== 3. Kruskal-Wallis 비모수 검정 결과 ===")
h_stat, p_val_kw = stats.kruskal(data_A, data_B, data_C)

print(f"H-statistic : {h_stat:.4f}")
print(f"p-value     : {p_val_kw:.6f}")

if p_val_kw < 0.05:
    print("\n▶ 최종 결론: p-value < 0.05 이므로 설비 간 중앙값(순위) 차이가 통계적으로 유의미합니다.")
    print("\n=== 4. Dunn's 사후검정 결과 (Bonferroni 보정) ===")
    dunn_df = custom_dunn_test(groups, p_adjust='bonferroni')
    print(dunn_df.to_string(index=False))
else:
    print("\n▶ 최종 결론: p-value >= 0.05 이므로 설비 간 의미 있는 차이가 존재하지 않습니다.")
```

```bash
(alex) raymond@ASUS:~/stats$ /home/raymond/anaconda3/envs/alex/bin/python /home/raymond/stats/KruskalWallis.py
=== 1. 설비별 기초 통계량 (비모수: 중앙값 중심) ===
Facility_A - 중앙값(Median): 0.5728, IQR: 0.8482
Facility_B - 중앙값(Median): 1.0654, IQR: 1.8444
Facility_C - 중앙값(Median): 1.3717, IQR: 3.2140
-----------------------------------------------------------------
=== 2. Shapiro-Wilk 정규성 검정 ===
Facility_A: W-stat = 0.7949, p-value = 0.0000 (정규성 만족: False)
Facility_B: W-stat = 0.8599, p-value = 0.0000 (정규성 만족: False)
Facility_C: W-stat = 0.8403, p-value = 0.0000 (정규성 만족: False)
-----------------------------------------------------------------
=== 3. Kruskal-Wallis 비모수 검정 결과 ===
H-statistic : 15.9228
p-value     : 0.000349

▶ 최종 결론: p-value < 0.05 이므로 설비 간 중앙값(순위) 차이가 통계적으로 유의미합니다.

=== 4. Dunn's 사후검정 결과 (Bonferroni 보정) ===
   Group 1    Group 2  Rank Diff  z-stat  p-unadj  p-adj (Bonferroni)  Reject H0
Facility_A Facility_B     -19.48  2.2419 0.024968            0.074904      False
Facility_A Facility_C     -34.58  3.9797 0.000069            0.000207       True
Facility_B Facility_C     -15.10  1.7378 0.082243            0.246730      False
```

### [통계 분석 보고서] 설비별 데이터 비모수 검정 (Kruskal-Wallis & Dunn's Test)

#### 1. 분석 개요
본 분석은 3개 설비(`Facility_A`, `Facility_B`, `Facility_C`) 간 측정값의 통계적 차이를 검정하기 위해 수행되었습니다. 각 설비별 표본 데이터에 대해 정규성 검정(Shapiro-Wilk Test)을 수행한 결과, **모든 설비가 정규분포를 따르지 않는 것으로 확인($p < 0.05$)**되었습니다. 

이에 따라 비모수 통계 방법인 **Kruskal-Wallis 검정** 및 사후검정으로 **Dunn's test (Bonferroni $p$-value 보정)**를 적용하여 최종 통계적 유의성을 분석하였습니다.

---

#### 2. 기초 통계량 (Descriptive Statistics)
비모수 분석 지침에 맞추어 **중앙값(Median)**과 **사분위수 범위(IQR, Interquartile Range)**를 분석 지표로 작성하였습니다.

| 설비명 (Group) | 중앙값 (Median) | IQR (Q3 - Q1) |
| :--- | :---: | :---: |
| **Facility_A** | 0.5728 | 0.8482 |
| **Facility_B** | 1.0654 | 1.8444 |
| **Facility_C** | 1.3717 | 3.2140 |

---

#### 3. 정규성 검정 (Shapiro-Wilk Test)
* **귀무가설 ($H_0$)**: 해당 설비의 데이터는 정규분포를 따른다.
* **유의수준 ($\alpha$)**: 0.05

| 설비명 (Group) | $W$-통계량 ($W$-stat) | $p$-value | 정규성 만족 여부 |
| :--- | :---: | :---: | :---: |
| **Facility_A** | 0.7949 | < 0.0001 | 미만족 (False) |
| **Facility_B** | 0.8599 | < 0.0001 | 미만족 (False) |
| **Facility_C** | 0.8403 | < 0.0001 | 미만족 (False) |

> **검정 결과**: 모든 설비의 $p$-value가 $0.0000$으로 유의수준 $0.05$보다 매우 작아 귀무가설을 기각합니다. 모수적 분석(ANOVA)이 불가능하므로 **비모수 검정(Kruskal-Wallis)을 채택**합니다.

---

#### 4. Kruskal-Wallis 비모수 검정 (Main Test)
3개 독립 집단 간 순위(Rank) 분포 및 중앙값에 차이가 있는지 검정하였습니다.

* **귀무가설 ($H_0$)**: 모든 설비 간 측정값의 순위 분포(중앙값)는 같다.
* **대립가설 ($H_1$)**: 적어도 하나의 설비는 다른 설비와 순위 분포(중앙값)가 다르다.

* **$H$-통계량 ($H$-statistic)**: `15.9228`
* **유의확률 ($p$-value)**: `0.000349` ($p < 0.001$)

> **검정 결과**: 유의확률이 $0.000349$로 유의수준 $0.05$보다 매우 작으므로 귀무가설을 기각합니다. 즉, **설비 간 측정값 중앙값에는 통계적으로 유의미한 차이가 존재**합니다.

---

#### 5. Dunn's 사후검정 (Post-Hoc Test)
어느 설비 쌍 간에 유의미한 차이가 존재하는지 확인하기 위해 Dunn's 사후검정을 실시하였으며, 다중 비교 시 발생하는 1종 오류 확대를 방지하기 위해 **Bonferroni $p$-value 보정**을 적용하였습니다.

| 비교 대상 (Group 1 vs Group 2) | 순위 차이 (Rank Diff) | $z$-통계량 ($z$-stat) | 보정 전 $p$-value | 보정 후 $p$-value (Bonferroni) | 귀무가설 기각 여부 (Reject $H_0$) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Facility_A vs Facility_B** | -19.48 | 2.2419 | 0.024968 | 0.074904 | **False (차이 없음)** |
| **Facility_A vs Facility_C** | -34.58 | 3.9797 | 0.000069 | 0.000207 | **True (유의미한 차이)** |
| **Facility_B vs Facility_C** | -15.10 | 1.7378 | 0.082243 | 0.246730 | **False (차이 없음)** |

---

#### 6. 종합 결론 및 해석

1. **`Facility_A` vs `Facility_B`**: 보정 전 $p$-value는 $0.0250$이었으나, 다중 비교 보정(Bonferroni) 적용 후 $p$-value가 $0.0749$로 증가하여 유의수준 $0.05$를 초과함에 따라 **통계적으로 유의미한 차이가 없음**으로 판정되었습니다.
2. **`Facility_A` vs `Facility_C`**: 보정 후 $p$-value가 $0.000207$로 유의수준 $0.05$보다 훨씬 작으므로, **두 설비 간에는 통계적으로 매우 명확한 차이가 존재**합니다.
3. **`Facility_B` vs `Facility_C`**: 보정 후 $p$-value가 $0.2467$로 유의수준 $0.05$보다 크므로, **통계적으로 유의미한 차이가 관측되지 않았습니다.**

> **최종 요약**: 
> `Facility_A`와 `Facility_B`, 그리고 `Facility_B`와 `Facility_C` 간에는 유의미한 차이가 발견되지 않았습니다. 그러나 **`Facility_A`와 `Facility_C` 사이에는 통계적으로 유의미한 차이가 확인되었으며, `Facility_C`(중앙값 1.3717)가 `Facility_A`(중앙값 0.5728)에 비해 측정값이 명확히 높은 경향**을 보입니다.

