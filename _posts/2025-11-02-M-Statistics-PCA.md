---
title: PCA
img_path: /assets/images/
author: Raymond
date: 2025-11-02
category: [Manufacturing, Statistics, PCA Analysis]
tags:
 - Correlation
 - Sensor Data
 - PostgreSQL
layout: post
---

## 1. 개요
반도체 엔지니어나 데이터 분석가로 일하다 보면 가장 흔하게 마주하는 난제 중 하나가 바로 "센서 파라미터가 너무 많다"는 점입니다.

하나의 설비(EQP) 안에서도 온도, 압력, RF Power, Gas Flow 등 수백~수천 개의 센서(Parameter)가 초 단위로 시계열 데이터를 쏟아냅니다. 하지만 이 수많은 파라미터들을 그대로 분석 모델에 넣으면 '차원의 덧(Curse of Dimensionality)'에 빠지게 되고, 변수 간 다중공선성(Multicollinearity) 문제나 계산 비용 폭증 문제에 직면하게 됩니다.

오늘은 이러한 수많은 반도체 센서 데이터 중에서 진짜 핵심 데이터만 선별하기 위한 필수 관문인 'PCA(주성분 분석)' 기술을 PostgreSQL DB 연동 환경에서 파이썬으로 구현한 과정을 공유합니다.

## 2. PCA(Principal Component Analysis 주성분 분석)란 무엇인가?
PCA(Principal Component Analysis, 주성분 분석)는 고차원의 데이터를 정보 손실을 최소화하면서 저차원으로 축소하는 대표적인 비지도 학습(Unsupervised Learning) 기법입니다.

💡 반도체 데이터에서 PCA가 필요한 이유
변수 간 높은 상관관계 제거: 예컨대 챔버 내부 온도 센서 A와 B는 거의 유사하게 움직입니다. PCA는 이렇게 중복되는 정보들을 하나로 묶어줍니다.

소수의 핵심 파라미터 도출: 1000개가 넘는 센서 데이터 중, 설비 상태 변화 정보의 90% 이상을 설명할 수 있는 상위 몇 개의 주성분(PC)으로 차원을 축소합니다.

노이즈 및 고장난 센서 필터링: 수치 변화가 전혀 없는 죽은 센서(Dead Sensor)나 극단적 노이즈를 전처리 과정에서 제거합니다.

## 3. 파이프라인 전체 구조
[PostgreSQL DB] 
   └── (1) Long-Format 시계열 센서 데이터 추출
          │
[Python Data Processing]
   ├── (2) Pivot 변환 (Long → Wide)
   ├── (3) 결측치 및 죽은 센서(Dead Sensor) 자동 제거
   ├── (4) Z-score 표준화 (Standardization)
   ├── (5) PCA 분석 수행 (정보량 90% 이상을 만족하는 PC 자동 추려내기)
   └── (6) Loading 분석으로 '핵심 중요 센서(Parameters)' 피처 추출
          │
[Data Visualization & DB Persistence]
   ├── (7) PCA Scree Plot 차트 생성

## 4. 핵심 분석 단계 및 로직 설명

① 데이터 정제 및 죽은 센서(Dead Sensor) 필터링

센서 수치가 영원히 동일하거나 분산이 0에 가까운 센서들은 PCA 분석에 악영향을 줍니다.
```python
# 분산이 0에 가까운(변화가 없는) 죽은 센서 필터링
std_series = df_wide.std()
valid_sensors = std_series[(std_series > 1e-12) & (std_series.notna())].index
df_cleaned = df_wide[valid_sensors]
```

② 주성분(PC) 개수 자동 설정 (정보 보존율 90%)

주성분 개수를 사용자가 임의로 3개, 5개 정하는 것이 아니라, 전체 데이터 변동성의 90% 이상을 설명해 주는 개수를 PCA 모델이 자동으로 선별하도록 설정했습니다.

```python
# 설명 분산 비율 90% 이상을 만족하는 주성분 자동 선택
pca = PCA(n_components=0.90)
X_pca = pca.fit_transform(X_scaled)

```

③ Loading(가중치) 분석을 통한 핵심 센서(Parameter) 발굴

PCA로 축소된 주성분(PC1, PC2 등)이 "실제 어떤 원본 센서의 영향을 가장 많이 받았는가?"를 역추적하는 단계입니다. Loading 절대값이 클수록 해당 PC를 형성하는 데 결정적인 역할을 한 핵심 센서입니다.

결과 예시:

PC1 (설비 대표 변동성): TEMP_SENSOR_01, PRESS_SENSOR_03
PC2 (RF Power 및 압력 미세 변동): RF_MATCH_POS_02, GAS_FLOW_05

이렇게 찾아낸 Top 센서들의 목록은 이후 설비 이상 감지(FDC)나 수명 예측(PHM) 모델의 최우선 입력 변수(Feature)로 활용됩니다.

## 5. 시각화 및 결과 저장

📈 PCA Scree Plot (스클리 플롯)
차원 축소가 얼마나 효율적으로 이루어졌는지 직관적으로 보여주는 차트입니다.

막대그래프 (Individual Variance): 각 주성분(PC1, PC2...)이 개별적으로 설명하는 정보의 양(%)

꺾은선그래프 (Cumulative Variance): 주성분들이 누적되어 전체 정보의 몇 %를 설명하는지 나타내는 누적 곡선 (목표치인 90% 달성 시점 확인 가능)

💾 PostgreSQL DB로 저장 및 관리 (LargeBinary / BYTEA)
분석된 결과(추출된 주요 센서 목록, 누적 설명력, 그리고 시각화된 Scree Plot 이미지 차트)는 단순 파일 형태가 아닌, PostgreSQL 데이터베이스 테이블(pca_analysis_result)에 다이렉트로 저장됩니다.

이미지 바이너리는 PostgreSQL의 BYTEA (SQLAlchemy의 LargeBinary) 타입으로 저장되어, 추후 웹 대시보드나 모니터링 시스템에서 DB 호출만으로 차트를 띄울 수 있도록 설계했습니다.

![Chart](/assets/images/2026-07-30-123539.png)

![PostgreSQL DB](/assets/images/2026-07-30-123620.png)