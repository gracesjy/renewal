---
title: Pandas Transpose/Pivot/Melt
img_path: /assets/images/
author: Alex
math: true
date: 2025-10-16
category: [Utility, Pandas, Pivot/Melt]
tags:
 - Pandas
 - Transpose
 - Pivot
 - Melt(Unpivot)
layout: post
---

## Transpose 와 Pivot

### 원본 데이터

|    | name | subject | score |
|---:|:-----|:--------|------:|
|  0 | Kim  | Math    |    90 |
|  1 | Kim  | English |    80 |
|  2 | Lee  | Math    |    70 |

### Transpose

원리: 행(0, 1, 2)과 열(name, subject, score)을 1:1로 단순히 바꿉니다.

|         | 0    | 1       | 2    |
|:--------|:-----|:--------|:-----|
| **name**    | Kim  | Kim     | Lee  |
| **subject** | Math | English | Math |
| **score**   | 90   | 80      | 70   |

### Pivot

원리: name을 행, subject를 열, score를 값으로 재배치합니다.

| name | English | Math |
|:-----|--------:|-----:|
| **Kim**  |    80.0 | 90.0 |
| **Lee**  |     NaN | 70.0 |

## Unpivot / Melt 적용 후 (pd.melt)

단순하게 적용하면 아래처럼 의도되지 않게 만들어진다. 의미가 없다.

| name | variable | value   |
|:-----|:---------|:--------|
| Kim  | subject  | Math    |
| Kim  | subject  | English |
| Lee  | subject  | Math    |
| Kim  | score    | 90      |
| Kim  | score    | 80      |
| Lee  | score    | 70      |

### name은 고정(id_vars), Math와 English 컬럼을 세로로 Melt

아래처럼, 의도된 형태로 사용해야 의미있게 된다.

df_melt = pd.melt(df, id_vars=['name'], var_name='subject', value_name='score')

| name | subject | score   |
|:-----|:---------|:--------|
| Kim  | Math  | 90    |
| Kim  | English  | 80 |
| Lee  | Math  | 70    |

...

## Database 와 Pivot

데이터베이스에서는 Transpose 키워드가 대부분 없고 대신 Pivot 키워드가 있다.  SQL 문으로 Pivot 을 할 수 있는다. Oracle Database 와 SQL Server 가 있다.

### Oracle Database Pivot
```sql
SELECT *
FROM scores
PIVOT (
    SUM(score) 
    FOR subject IN ('Math' AS MATH, 'English' AS ENGLISH)
);
```

### 표준 SQL 공통 방식: CASE WHEN + 집계함수 (GROUP BY)

```sql
SELECT 
    name,
    MAX(CASE WHEN subject = 'Math' THEN score END) AS MATH,
    MAX(CASE WHEN subject = 'English' THEN score END) AS ENGLISH
FROM scores
GROUP BY name;
```

### Unpivot SQL
```sql
SELECT name, subject, score
FROM student_grades
UNPIVOT (
    score FOR subject IN (MATH, ENGLISH)
);
```

## Titanic 예제

### Titanic CSV Top 10 Rows

|   PassengerId |   Survived |   Pclass | Name                                                | Sex    |   Age |   SibSp |   Parch | Ticket           |    Fare | Cabin   | Embarked   |
|--------------:|-----------:|---------:|:----------------------------------------------------|:-------|------:|--------:|--------:|:-----------------|--------:|:--------|:-----------|
|             1 |          0 |        3 | Braund, Mr. Owen Harris                             | male   |    22 |       1 |       0 | A/5 21171        |  7.25   | nan     | S          |
|             2 |          1 |        1 | Cumings, Mrs. John Bradley (Florence Briggs Thayer) | female |    38 |       1 |       0 | PC 17599         | 71.2833 | C85     | C          |
|             3 |          1 |        3 | Heikkinen, Miss. Laina                              | female |    26 |       0 |       0 | STON/O2. 3101282 |  7.925  | nan     | S          |
|             4 |          1 |        1 | Futrelle, Mrs. Jacques Heath (Lily May Peel)        | female |    35 |       1 |       0 | 113803           | 53.1    | C123    | S          |
|             5 |          0 |        3 | Allen, Mr. William Henry                            | male   |    35 |       0 |       0 | 373450           |  8.05   | nan     | S          |
|             6 |          0 |        3 | Moran, Mr. James                                    | male   |   nan |       0 |       0 | 330877           |  8.4583 | nan     | Q          |
|             7 |          0 |        1 | McCarthy, Mr. Timothy J                             | male   |    54 |       0 |       0 | 17463            | 51.8625 | E46     | S          |
|             8 |          0 |        3 | Palsson, Master. Gosta Leonard                      | male   |     2 |       3 |       1 | 349909           | 21.075  | nan     | S          |
|             9 |          1 |        3 | Johnson, Mrs. Oscar W (Elisabeth Vilhelmina Berg)   | female |    27 |       0 |       2 | 347742           | 11.1333 | nan     | S          |
|            10 |          1 |        2 | Nasser, Mrs. Nicholas (Adele Achem)                 | female |    14 |       1 |       0 | 237736           | 30.0708 | nan     | C          |


...


[titanic.csv 다운로드]({{ '/assets/data/titanic.csv' | relative_url }})


### 기술 통계량(Descriptive Statistics)

```python
import pandas as pd

# 1. titanic.csv 파일 불러오기 (파일 경로에 맞게 수정)
# 파일이 스크립트와 같은 폴더에 있다면 파일명만 적으시면 됩니다.
df = pd.read_csv("titanic.csv")

# 2. 컬럼명을 모두 소문자로 일괄 변환 (대소문자 차이로 인한 KeyError 방지)
df.columns = df.columns.str.lower()

# 3. 필요한 6개 수치형 컬럼만 선택
target_cols = ['survived', 'pclass', 'age', 'sibsp', 'parch', 'fare']
df_sub = df[target_cols]

# 4. describe() 실행 후 reset_index()로 인덱스(count, mean 등)를 일반 컬럼으로 변환
df_desc = df_sub.describe().reset_index()

# 5. 자동으로 생성된 'index' 컬럼명을 'vars'로 변경
df_desc.rename(columns={'index': 'vars'}, inplace=True)

# 6. melt를 통한 세로(Long Format) 구조 변환
df_melt = pd.melt(df_desc, id_vars=['vars'])

# 결과 확인
print("=== df_melt 상위 10개 행 ===")
print(df_melt.head(10))

print("\n=== df_melt 전체 결과 ===")
print(df_melt)

```
![2026-08-07-145834.png](/assets/images/2026-08-07-145834.png)

### Melt (Unpivot) 을 한 이유

만약 데이터베이스 (Oracle)에 titanic 테이블이 있고, 이를 이용한다고 한 후에 기술 통계량을 다시 Oracle Database 에 저장한다고 한다면 ?

반도체 설비의 센서들에 대한 기술 통계량을 구해서 이를 다시 저장한다고 한다면 ?  Row 기반의 Unpivot (melt) 없이는 저장할 수 없는 이유는 바로 최대 컬럼 1,000 개 이기 때문이다.

$$\text{컬럼 수} = (\text{파라미터 수 } \approx 300) \times (\text{기술통계량 수 } 8) = 2,400\text{개}$$

또 Column 이름들이 제각각이기 때문에 매핑의 어려움도 있다.  따라서 데이터베이스(RDBMS) 저장을 위한 정규화 (EAV 패턴) 으로 저장하는 것이 효율적이다.  여기에서 EAV 는 Entity - Attribute - Value의 약자로, 개체-속성-값 모델을 의미한다. 따라서 이 테이블은 범용 테이블로서 가치가 있는 것이고 Column 형태로 즉, pivot 형태로 테이블에 저장하면 데이터 형태마다 컬럼이름을 정하든지 아니면 매핑 테이블을 또 따로 두는 등 복잡해 진다.



물론 Row 기반으로 저장되어 있다면 계산이 용이한 점도 크다. 특히 필터링 같은 경우가 대표적이다.
```python
# 통계지표가 'mean'이면서 그 값이 30 이상인 데이터만 필터링
df_melt[(df_melt['vars'] == 'mean') & (df_melt['value'] >= 30)]
```



```python
import pandas as pd
import numpy as np
from sqlalchemy import create_engine
from sqlalchemy.dialects.oracle import NUMBER, FLOAT, VARCHAR2

# 1. 오라클 DB 접속 정보
DB_USER = "rquser"
DB_PASS = "nebula"
DB_HOST = "172.30.224.1"
DB_PORT = "1521"
DB_SERVICE = "FREE"

# 2. SQLAlchemy 엔진 생성
db_url = f"oracle+oracledb://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/?service_name={DB_SERVICE}"
engine = create_engine(db_url)

query = """
        SELECT * FROM TITANIC
    """

    # Engine을 연동하여 Pandas DataFrame으로 수집
df = pd.read_sql(query, con=engine)
print(df)
df.columns = df.columns.str.lower()
print(df.columns)

# -------------------------------------------------------------
# 3. 데이터 전처리 및 구조 변환 (df_melt)
# -------------------------------------------------------------

target_cols = ['survived', 'pclass', 'age', 'sibsp', 'parch', 'fare']
df_target = df[target_cols]

# 3-1. 수치형 컬럼 요약 통계량 추출
print(df_target)
df_desc = df_target.describe().reset_index()

print("------------------")

print(df_desc)
# 3-3. 컬럼명 정제 (소문자 표준화)
df_desc.rename(columns={'index': 'vars'}, inplace=True)
#df_desc.columns = ['vars', 'survived', 'pclass', 'age', 'sibsp', 'parch', 'fare']

# 3-4. melt를 통한 세로 형태(Long Format) 변환
df_melt = pd.melt(df_desc, id_vars=['vars'])

# 결과 확인
print(df_melt.head(10))

```
![2026-08-07-150045.png](/assets/images/2026-08-07-150045.png)


## Text 기반 Pivot/Unpivot
Text 기반에서 한번 더 Pivot 과 Unpivot(Melt) 를 연습해 보자.

```python
import pandas as pd

# 1. 엑셀/텍스트 형태의 원본 데이터 (Pivot / Wide Format)
data = {
    'PCS_ID': ['LOT_A01', 'LOT_A01', 'LOT_A02', 'LOT_A02', 'LOT_B01', 'LOT_B01'],
    'SEQ': [1, 2, 1, 2, 1, 2],
    'REF_VAR1': ['PASS', 'PASS', 'FAIL', 'PASS', 'PASS', 'PASS'],
    'REF_VAR2': ['OK', 'NG', 'OK', 'OK', 'NG', 'OK'],
    'REF_VAR3': ['HIGH', 'NORMAL', 'LOW', 'NORMAL', 'HIGH', 'NORMAL'],
    'REF_VAR4': ['STEP1', 'STEP2', 'STEP1', 'STEP2', 'STEP1', 'STEP2']
}
df = pd.DataFrame(data)

print(df)
print("----------------")

# 2. 'REF_VAR'로 시작하는 변수 컬럼들만 자동으로 추출
ref_cols = [col for col in df.columns if col.startswith('REF_VAR')]

# 3. melt(unpivot) 실행
df_pcs_melt = pd.melt(
    df,
    id_vars=['PCS_ID', 'SEQ'],      # 식별자 컬럼 고정
    value_vars=ref_cols,            # 녹일 컬럼 목록
    var_name='PARAM_NAME',          # 컬럼명이 입력될 새 컬럼명
    value_name='PARAM_VALUE'        # 실제 값이 입력될 새 컬럼명
)

# 4. 보기 좋게 LOT/SEQ/파라미터 순으로 정렬
df_pcs_melt = df_pcs_melt.sort_values(by=['PCS_ID', 'SEQ', 'PARAM_NAME']).reset_index(drop=True)
print(df_pcs_melt)

# df_pcs_melt 데이터를 다시 원래 표 모양으로 복원
df_pivoted = df_pcs_melt.pivot(
    index=['PCS_ID', 'SEQ'],    # 기준이 될 행(Row) 식별자
    columns='PARAM_NAME',       # 새 컬럼명이 될 항목
    values='PARAM_VALUE'        # 채워 넣을 데이터 값
).reset_index()

# 컬럼의 상위 이름(PARAM_NAME) 지우기 (인덱스 정리)
df_pivoted.columns.name = None

# 완벽하게 일치하면 True, 하나라도 다르면 False
is_same = df.equals(df_pivoted)
print("두 데이터프레임이 동일한가요?:", is_same)
# 출력: True

```