---
title: Chi-Square Test
img_path: /assets/images/
author: Raymond
date: 2025-11-02
category: [Manufacturing, Statistics, Descriptive, Chi-Square Test]
tags:
 - Chi-Square Test
 - Equipment Significant Difference
 - PostgreSQL
layout: post
---

## 개요
제조 현장에서 "특정 설비의 불량률이 유독 높은 것 같은데, 이게 진짜 설비 문제일까? 아니면 단지 어쩌다 생긴 운(우연)일까?"라는 고민을 자주 하게 됩니다.

이러한 의문을 통계학적으로 명확히 밝혀내는 대표적인 방법이 바로 카이제곱 검정(Chi-Square Test)입니다. 오늘은 PostgreSQL 데이터를 가져와 파이썬으로 설비 간 유의차를 검정하고 시각화하는 전체 프로세스를 알아보겠습니다.

## 카이제곱 검정(Chi-Square Test)이란?

① 개념 요약카이제곱 검정은 범주형 데이터(예: 설비A/B/C, 불량/양품) 간에 "서로 관련이 있는지(독립성)" 또는 "관측한 값이 이론적 기대값과 차이가 있는지"를 검정하는 통계적 기법입니다.이번 분석에서는 카이제곱 독립성 검정을 사용합니다.

> 가설 설정
귀무가설 (H0): 설비 종류와 불량 발생은 아무 관계가 없다. (모든 설비의 불량률 차이는 우연이다.)

대립가설 (H1): 설비 종류에 따라 불량 발생에 차이가 있다. (특정 설비의 불량률이 유의미하게 높거나 낮다.)

② p-value(유의확률) 읽는 법
 p-value < 0.05 (유의수준 5% 미만):
 귀무가설을 기각합니다. 즉, "우연이라고 보기 어렵고, 진짜로 설비 간 불량률에 통계적인 차이가 존재한다"고 결론을 내립니다. (해당 설비 점검 필요!)

 p-value > 0.05:
 귀무가설을 채택합니다. 차이가 있어 보여도 통계적으로는 "우연히 발생할 수 있는 범위 안의 차이"로 봅니다.

## 파이썬 분석 코드 흐름
작성된 전체 코드는 데이터 추출 $\rightarrow$ 통계 검정 $\rightarrow$ 시각화 3단계로 구성되어 있습니다. 각 파트별 핵심 의미를 살펴봅니다.

1. 집계 (PostgreSQL DB)
SQL 문을 통해 단순히 LOT별 데이터를 나열하는 것이 아니라, 설비(machine_id)별로 총 양품 수와 총 불량 수를 집계합니다.

```sql
SELECT 
    machine_id,
    SUM(ng_qty)                         AS observed_ng,   -- 관측된 불량수
    SUM(out_qty - ng_qty)               AS observed_good, -- 관측된 양품수
    SUM(out_qty)                        AS total_qty,     -- 총 생산량
    ROUND(SUM(ng_qty)::numeric / NULLIF(SUM(out_qty), 0) * 100, 2) AS defect_rate_pct
FROM lot_sum
WHERE out_qty > 0
GROUP BY machine_id
ORDER BY defect_rate_pct DESC;

```
![2026-07-30-134707.png](/assets/images/2026-07-30-134707.png)


2. 카이제곱 검정 수행

Pandas로 읽어온 데이터를 Scipy의 통계 모듈로 넘겨 계산을 수행하는 핵심 부분입니다
```python
# 분할표(2D 배열) 생성
contingency_table = df[['observed_ng', 'observed_good']].values

# 카이제곱 검정 실행
chi2_stat, p_val, dof, expected = stats.chi2_contingency(contingency_table)
```

3. 대용량 설비(19개 이상)를 위한 시각화 최적화

```python
df = df.sort_values(by='defect_rate_pct', ascending=False)
```
Chart Size 조절
```python
fig, axes = plt.subplots(1, 2, figsize=(18, 7)) # 가로 폭을 18인치로 넓힘
axes[0].set_xticklabels(axes[0].get_xticklabels(), rotation=45, ha='right')
```
관측값 등 비교
```python
axes[1].bar(x - width/2, df['observed_ng'], ... ) # 실제 불량 수
axes[1].bar(x + width/2, df['expected_ng'], ... ) # 이론상 기대 불량 수
```

![2026-07-30-134922.png](/assets/images/2026-07-30-134922.png)

## 결과
==================================================
1. 카이제곱 통계량 (Chi2 Statistic) : 191.7006
2. 자유도 (Degrees of Freedom)    : 18
3. 유의확률 (p-value)            : 4.543450e-31
==================================================
결과: p-value < 0.05 이므로 '설비 간 불량률에 통계적으로 유의미한 차이가 있다'고 판단합니다.


## Machine Quality Comparison

LOT 단위의 불량률(Defect Rate) 을 이용하여 문제 설비를 찾아본다.

```sql
SELECT 
    lot_id,
    machine_id,
    out_qty,
    ng_qty,
    ROUND((ng_qty::numeric / NULLIF(out_qty, 0)) * 100, 2) AS defect_rate_pct
FROM lot_sum
WHERE out_qty > 0
```

```sql
SELECT 
    lot_id,
    machine_id,
    SUM(out_qty) AS out_qty,
    SUM(ng_qty)  AS ng_qty,
    ROUND((SUM(ng_qty)::numeric / NULLIF(SUM(out_qty), 0)) * 100, 2) AS defect_rate_pct
FROM lot_sum
WHERE out_qty > 0
GROUP BY lot_id, machine_id;
```

앞서 48PARA-03 가 Defect 비율이 높았기 때문에 다른 설비들과 비교를 해 보자. 또한 특별히 불량율이 높은 Lot ID 에 대해서도 검색을 해 보자.

```python

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import sqlalchemy
from sqlalchemy import create_engine

# ----------------------------------------------------
# 1. DB 연결 및 lot_id 포함 데이터 조회
# ----------------------------------------------------
DB_USER = "raymond"
DB_PASS = "bdae_password"
DB_HOST = "localhost"
DB_PORT = "5432"
DB_NAME = "bdae_knowledge"

db_url = f"postgresql+psycopg2://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/{DB_NAME}"
engine = create_engine(db_url)

# lot_id를 포함한 쿼리 실행
query = """
SELECT 
    lot_id,
    machine_id,
    out_qty,
    ng_qty,
    ROUND((ng_qty::numeric / NULLIF(out_qty, 0)) * 100, 2) AS defect_rate_pct
FROM lot_sum
WHERE out_qty > 0;
"""

df = pd.read_sql(query, engine)

# ----------------------------------------------------
# 2. 문제 설비 (48PARA-03)의 이상 LOT 추적 (Top 5)
# ----------------------------------------------------
print("=" * 60)
print(" [48PARA-03 설비] 불량률 Highest Top 5 LOT 목록")
print("=" * 60)
worst_lots = df[df['machine_id'] == '48PARA-03'].sort_values(by='defect_rate_pct', ascending=False).head(5)
print(worst_lots[['lot_id', 'machine_id', 'out_qty', 'ng_qty', 'defect_rate_pct']].to_string(index=False))
print("=" * 60)


# ----------------------------------------------------
# 3. 산포 시각화 (KDE Curve & Boxplot)
# ----------------------------------------------------
plt.style.use('seaborn-v0_8-whitegrid')
fig, axes = plt.subplots(1, 2, figsize=(16, 6))

df_target = df[df['machine_id'] == '48PARA-03']
df_others = df[df['machine_id'] != '48PARA-03']

# [차트 1] 커널 밀도 곡선 (KDE Plot)
sns.kdeplot(data=df_target['defect_rate_pct'], ax=axes[0], color='red', 
            fill=True, alpha=0.3, linewidth=2.5, label='48PARA-03 (High Variation)')
sns.kdeplot(data=df_others['defect_rate_pct'], ax=axes[0], color='blue', 
            fill=True, alpha=0.3, linewidth=2.5, label='Other Machines (Stable)')

axes[0].set_title('Defect Rate Distribution by LOT\n48PARA-03 vs Other Machines', fontsize=14, fontweight='bold')
axes[0].set_xlabel('Defect Rate (%)', fontsize=12)
axes[0].set_ylabel('Density', fontsize=12)
axes[0].legend(fontsize=11)

# [차트 2] 박스플롯 (Box Plot)
sns.boxplot(data=df, x='machine_id', y='defect_rate_pct', ax=axes[1], palette='Set3')
axes[1].set_title('LOT Defect Rate Spread Across All Machines', fontsize=14, fontweight='bold')
axes[1].set_xlabel('Machine ID', fontsize=12)
axes[1].set_ylabel('Defect Rate (%)', fontsize=12)
axes[1].set_xticklabels(axes[1].get_xticklabels(), rotation=45, ha='right')

plt.tight_layout()
plt.show()

```

![2026-07-30-140417.png](/assets/images/2026-07-30-140417.png)

lot_id가 들어가면 단순히 "48PARA-03 설비의 산포가 크다"에서 끝나는 것이 아니라, "48PARA-03 설비에서 불량률을 크게 끌어올린 원인 LOT(worst_lots)이 무엇인가?"까지 파이썬 콘솔 창에서 바로 찾아내어 원인 분석(Root Cause Analysis)으로 이어갈 수 있습니다.



-----


[다운로드]({{ '/assets/data/pca.csv' | relative_url }})