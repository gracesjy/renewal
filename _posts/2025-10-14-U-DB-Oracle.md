---
title: Oracle Database Handling for Analysis
img_path: /assets/images/
author: Raymond
date: 2025-10-14
category: [Utility, Database, Oracle]
tags:
 - PostgreSQL
layout: post
---

## File -> Oracle Database Table with Python
여러 DB 툴이 있기 때문에 그것들을 사용해도 된다. Oracle DB 설치는 Docker 등을 활용하면 좋다.

--
여기에서는 Python 으로 간략하게 만드는 것을 보여준다.

CSV 파일의 맨 상단에 헤더가 있는지 확인하고 가급적이면 공백이 없는 것으로 잘 수정하거나 입력 해 둔다.  이것은 컬럼 이름이 되기 때문이다.


### Package Installation

예전에는 cx_Oracle 등을 사용했었지만, 지금은 oracledb 패키지를 사용하면 된다.

```bash
pip install pandas sqlalchemy oracledb
```

### import csv to PostgreSQL
Oracle 은 컬럼 타입에 주의해야 한다. 

```python
import pandas as pd
import numpy as np
from sqlalchemy import create_engine
from sqlalchemy.dialects.oracle import NUMBER, FLOAT, VARCHAR2

# 1. 오라클 DB 접속 정보
DB_USER = "000"
DB_PASS = "000"
DB_HOST = "000"
DB_PORT = "1521"
DB_SERVICE = "FREE"

# 2. SQLAlchemy 엔진 생성
db_url = f"oracle+oracledb://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/?service_name={DB_SERVICE}"
engine = create_engine(db_url)

# 3. 파일 및 테이블명 설정
CSV_FILE_PATH = "pca.csv"
TABLE_NAME = "pca_sensor_data"

try:
    # 4. CSV 파일 읽기
    df = pd.read_csv(CSV_FILE_PATH, encoding='utf-8-sig')
    print(f"CSV 파일 읽기 완료: 총 {len(df):,}행, {len(df.columns)}개 컬럼")

    # 컬럼명 대문자 변환 (오라클 권장)
    df.columns = [col.upper() for col in df.columns]

    # 5. 데이터 타입 매핑 사전(dict) 자동 생성
    # Pandas 컬럼 타입 분석 후 소수점(float) 데이터는 Oracle의 NUMBER 타입으로 자동 지정
    dtype_mapping = {}
    for col in df.columns:
        if pd.api.types.is_float_dtype(df[col]):
            # float 타입은 Oracle의 NUMBER 타입으로 지정하여 에러 방지
            dtype_mapping[col] = NUMBER
        elif pd.api.types.is_integer_dtype(df[col]):
            dtype_mapping[col] = NUMBER
        elif pd.api.types.is_datetime64_any_dtype(df[col]):
            pass # 기본 datetime 변환 사용
        else:
            # 문자열 컬럼은 가변 길이 고려하여 VARCHAR2 지정 (기본 255 또는 필요시 조율)
            dtype_mapping[col] = VARCHAR2(255)

    print("생성된 컬럼 매핑:", dtype_mapping)

    # 6. 오라클 DB로 데이터 적재 (to_sql)
    df.to_sql(
        name=TABLE_NAME,
        con=engine,
        if_exists='replace',
        index=False,
        chunksize=5000,           # 대용량(90만건) 데이터이므로 5000 정도로 지정
        dtype=dtype_mapping       # ★ 핵심: 오라클 타입 매핑 전달
    )

    print(f"성공적으로 오라클 DB의 '{TABLE_NAME}' 테이블에 데이터를 적재했습니다.")

except Exception as e:
    print(f"오류 발생: {e}")

```

### 확인
DB 툴 (sqldeveloper 또는 DBeaver)로 해당 테이블을 확인한다.

## Oracle to Pandas DataFrame

```python

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sqlalchemy import create_engine
from sqlalchemy.dialects.oracle import NUMBER, VARCHAR2

# 1. DB 접속 정보 설정 및 엔진 생성
DB_USER = "000"
DB_PASS = "000"
DB_HOST = "localhost"
DB_PORT = "1521"
DB_SERVICE = "FREE"

db_url = f"oracle+oracledb://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/?service_name={DB_SERVICE}"
engine = create_engine(db_url)

# 2. 오라클에서 Long Format 데이터 읽어오기
# (컬럼명이 대문자로 들어갔을 수 있으므로 쿼리 작성)
query = """
    SELECT sequence, variable, value
    FROM pca_sensor_data order by variable, sequence
"""

print("DB에서 데이터를 읽는 중...")
df_long = pd.read_sql(query, con=engine)

# 컬럼명 소문자 통일 (조작 편의성)
df_long.columns = [col.lower() for col in df_long.columns]
print(f"원본 데이터 (Long Format): {df_long.shape} 행")

# 3. Pivot: Row 기반 -> Column 기반 (Wide Format 변환)
# index: 시계열 순서(sequence), columns: 센서명(variable), values: 측정값(value)
print("데이터 Pivot 변환 중...")
df_wide = df_long.pivot(index='sequence', columns='variable', values='value')

print(f"변환된 데이터 (Wide Format): {df_wide.shape} (행: sequence, 열: 센서)")
print(df_wide.head())

# 4. 결측치 처리 (필요시)
# 시계열 데이터 특성상 센서별 주기 차이 등으로 결측치가 있을 경우 보정
df_wide = df_wide.ffill().bfill() # 앞/뒤 값으로 채우기

# 5. 상관관계 행렬(Correlation Matrix) 계산 (Pearson 상관계수)
# -1 ~ 1 사이 값을 가지며 1에 가까울수록 양의 상관관계, -1에 가까울수록 음의 상관관계
corr_matrix = df_wide.corr(method='pearson')

print("\n--- 상관관계 행렬 (상위 5개 센서) ---")
print(corr_matrix.iloc[:5, :5])

# 6. scikit-learn / Seaborn을 이용한 히트맵(Heatmap) 시각화
plt.figure(figsize=(12, 10))
sns.heatmap(
    corr_matrix,
    annot=False,      # 센서 수가 많으면 숫자는 끄는 것이 깔끔합니다 (필요시 True)
    cmap='coolwarm',  # 빨간색(양의 상관관계) - 파란색(음의 상관관계)
    vmax=1.0,
    vmin=-1.0,
    linewidths=0.5
)
plt.title('Sensor Data Correlation Matrix', fontsize=15)
plt.xlabel('Sensors')
plt.ylabel('Sensors')
plt.tight_layout()

# 7. (응용) 가장 상관관계가 높은 / 낮은 센서 쌍 찾아보기
# 자기 자신과의 상관관계(1.0)를 제외하기 위해 상삼각 행렬(Upper Triangle)만 추출
sol = corr_matrix.where(np.triu(np.ones(corr_matrix.shape), k=1).astype(bool))
sol_unstacked = sol.unstack().dropna()

# 절대값이 높은 순으로 정렬
top_correlations = sol_unstacked.reindex(sol_unstacked.abs().sort_values(ascending=False).index)

print("\n--- 가장 강한 상관관계를 가진 센서 탑 10 ---")
print(top_correlations.head(10))

threshold = 0.9

strong_corr = (
    top_correlations[
        top_correlations.abs() >= threshold
    ]
)

print(strong_corr)
plt.show()

```
## Oracle to Pandas DataFrame and CSV file
```python
import pandas as pd
from sqlalchemy import create_engine

# 1. DB 접속 정보 설정
DB_USER = "rquser"       # 오라클 사용자 이름
DB_PASS = "nebula"   # 비밀번호
DB_HOST = "localhost"     # 호스트 주소 (예: 127.0.0.1 또는 IP)
DB_PORT = "1521"          # 오라클 기본 포트
DB_SERVICE = "FREE"       # 서비스 이름 (Service Name 또는 SID)

# 2. SQLAlchemy 엔진 생성 (oracledb 드라이버 사용)
# oracle+oracledb://[USER]:[PASSWORD]@[HOST]:[PORT]/?service_name=[SERVICE_NAME]
db_url = f"oracle+oracledb://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/?service_name={DB_SERVICE}"
engine = create_engine(db_url)

# 3. 테이블 이름 및 저장할 CSV 파일 경로 설정
TABLE_NAME = "pca_test"   # 읽어올 테이블명
CSV_FILE_PATH = "pca.csv"   # 저장할 CSV파일명

try:
    # 4. DB 테이블을 데이터프레임으로 읽기
    # 단순 전체 조회시 테이블명 바로 입력 가능: pd.read_sql_table(TABLE_NAME, con=engine)
    # 특정 조건/쿼리가 필요하면 pd.read_sql 사용:
    df = pd.read_sql(f"SELECT * FROM {TABLE_NAME}", con=engine)
    print(f"성공적으로 데이터를 읽어왔습니다. (총 {len(df)} 행)")

    # 5. 데이터프레임을 CSV 파일로 저장
    # index=False: 파이썬의 인덱스 번호는 CSV에 저장하지 않음
    # encoding='utf-8-sig': 한글이 깨지지 않도록 UTF-8 BOM 설정 (엑셀 호환)
    df.to_csv(CSV_FILE_PATH, index=False, encoding='utf-8-sig')
    print(f"CSV 파일 저장 완료: {CSV_FILE_PATH}")

except Exception as e:
    print(f"오류 발생: {e}")

```

## Pivot
```python
import pandas as pd
from sqlalchemy import create_engine

# 1. 오라클 DB 접속 설정
DB_USER = "000"
DB_PASS = "000"
DB_HOST = "localhost"
DB_PORT = "1521"
DB_SERVICE = "FREE"

db_url = f"oracle+oracledb://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/?service_name={DB_SERVICE}"
engine = create_engine(db_url)

# 2. DB에서 데이터 읽기 (sequence 순서대로 정렬)
query = """
    SELECT sequence, variable, value
    FROM pca_sensor_data
    ORDER BY sequence
"""
df_long = pd.read_sql(query, con=engine)
df_long.columns = [col.lower() for col in df_long.columns]

# 3. Pivot 변환 (Row -> Column)
# index: 시간/순서(sequence), columns: 센서명(variable), values: 수치(value)
df_wide = df_long.pivot(index='sequence', columns='variable', values='value')

# 4. 각 variable별로 NumPy Array 담기 (Dictionary 형태)
sensor_arrays = {}
for col in df_wide.columns:
    # .dropna() 또는 .fillna()로 결측치 처리 후 array 변환 가능
    sensor_arrays[col] = df_wide[col].to_numpy()

# --- 결과 확인 ---
# 예: 특정 센서 'param_001'의 Array 출력
sample_sensor = list(sensor_arrays.keys())[0]
print(f"센서 이름: {sample_sensor}")
print(f"데이터 타입: {type(sensor_arrays[sample_sensor])}")
print(f"Array Shape: {sensor_arrays[sample_sensor].shape}")
print(f"상위 5개 값: {sensor_arrays[sample_sensor][:5]}")
```

## Array to Oracle
```python
import pandas as pd
import numpy as np
from sqlalchemy import create_engine
from sqlalchemy.types import VARCHAR, CLOB

# 1. 오라클 DB 접속 정보 설정
DB_USER = "000"
DB_PASS = "000"
DB_HOST = "localhost"
DB_PORT = "1521"
DB_SERVICE = "FREE"

db_url = f"oracle+oracledb://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/?service_name={DB_SERVICE}"
engine = create_engine(db_url)

try:
    # 2. 원본 데이터 읽어오기 (sequence 순서 보장)
    query = """
        SELECT sequence, variable, value
        FROM pca_sensor_data
        ORDER BY sequence
    """
    print("DB에서 데이터를 읽어오는 중...")
    df_long = pd.read_sql(query, con=engine)
    df_long.columns = [col.lower() for col in df_long.columns]

    # 3. Pivot 변환 (sequence x variable)
    print("Pivot 변환 및 시계열 문자열 생성 중...")
    df_wide = df_long.pivot(index='sequence', columns='variable', values='value')

    # 4. 각 variable(센서)별 값을 콤마(,) 구분 문자열로 변환하여 새로운 DataFrame 생성
    # 예시: '12.5, 13.1, 14.0, ...' 형태의 긴 시계열 문자열
    ts_list = []

    for var_name in df_wide.columns:
        # 결측치가 있을 경우를 대비해 처리 후 문자열로 결합 (NaN 제거/보정 필요시 ffill() 등 추가)
        values_array = df_wide[var_name].dropna().to_numpy()

        # Array 요소들을 쉼표(,)로 연결하여 긴 텍스트 생성
        ts_string = ",".join(map(str, values_array))

        ts_list.append({
            'variable': str(var_name),
            'ts_data': ts_string
        })

    # 새로운 DataFrame 생성 (variable, ts_data)
    df_result = pd.DataFrame(ts_list)

    print(f"변환 완료! 생성된 센서 수: {len(df_result)}개")
    print(df_result.head())  # 상위 5개 데이터 미리보기

    # 5. 오라클에 새로운 테이블 생성 및 데이터 입력
    target_table_name = "pca_sensor_ts_summary"
    print(f"타겟 테이블 '{target_table_name}' 생성 및 저장 중...")

    # SQLAlchemy 데이터 타입 매핑 설정 (variable: VARCHAR2, ts_data: CLOB)
    dtype_mapping = {
        'variable': VARCHAR(250),
        'ts_data': CLOB
    }

    # if_exists='replace': 기존 테이블이 있으면 드롭 후 새로 생성
    df_result.to_sql(
        name=target_table_name,
        con=engine,
        if_exists='replace',
        index=False,
        dtype=dtype_mapping
    )

    print(f"성공적으로 '{target_table_name}' 테이블이 생성되었으며 데이터를 저장했습니다.")

except Exception as e:
    print(f"오류 발생: {e}")
```
![Array Table Result](/assets/images/2026-07-30-114517.png)

## Oracle CLOB Array Column to Dataframe
```python
import pandas as pd
import numpy as np
from sqlalchemy import create_engine

# 1. 오라클 DB 접속 설정
DB_USER = "000"
DB_PASS = "000"
DB_HOST = "localhost"
DB_PORT = "1521"
DB_SERVICE = "FREE"

db_url = f"oracle+oracledb://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/?service_name={DB_SERVICE}"
engine = create_engine(db_url)

try:
    # 2. 요약 테이블(pca_sensor_ts_summary) 읽어오기
    # 이미 1개 센서 = 1개 Row로 모여있어 정렬이나 Pivot이 필요 없습니다.
    query = "SELECT variable, ts_data FROM pca_sensor_ts_summary"
    print("DB에서 요약 테이블(CLOB)을 읽어오는 중...")
    df_summary = pd.read_sql(query, con=engine)
    df_summary.columns = [col.lower() for col in df_summary.columns]

    # 3. CLOB 텍스트 데이터를 숫자(float) NumPy Array로 복원
    sensor_arrays = {}

    print("CLOB 시계열 텍스트 -> NumPy Array 변환 중...")
    for idx, row in df_summary.iterrows():
        var_name = row['variable']
        clob_text = row['ts_data']

        if clob_text and isinstance(clob_text, str):
            # 쉼표(,) 기준 분할 후 float32/float64 배열로 변환
            # (ValueError 방지를 위해 strip() 적용)
            arr = np.array([float(val.strip()) for val in clob_text.split(',') if val.strip()], dtype=np.float64)
        else:
            arr = np.array([], dtype=np.float64)

        sensor_arrays[var_name] = arr

    # 4. (선택) 이전 방식처럼 완전히 동일한 시계열 DataFrame (sequence x variable)으로 복원하기
    # 모든 센서의 시계열 길이가 동일하다면 아래 1줄로 원본 df_wide 형태 완성!
    df = pd.DataFrame(sensor_arrays)
    df.index.name = 'sequence'

    # --- 결과 확인 ---
    print("\n" + "="*50)
    print(" [변환 결과 확인] ")
    print("="*50)

    # A. 딕셔너리 형태의 Array 확인
    sample_sensor = list(sensor_arrays.keys())[0]
    print(f"1. 샘플 센서 이름: {sample_sensor}")
    print(f"   데이터 타입: {type(sensor_arrays[sample_sensor])}")
    print(f"   Array Shape: {sensor_arrays[sample_sensor].shape}")
    print(f"   상위 5개 값: {sensor_arrays[sample_sensor][:5]}")

    # B. 복원된 DataFrame 형태 확인
    print(f"\n2. 최종 복원된 DataFrame (df) Shape: {df.shape}")
    print(df.head())

except Exception as e:
    print(f"오류 발생: {e}")
```

## Pandas DataFrame 
분석 서버들 (FastAPI, DJango) 에서 리턴을 JSON 등으로 하기 전에 Pandas DataFrame 에 결과를 담아 보는 예제

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sqlalchemy import create_engine
import io
import base64

# 1. DB 접속 정보 설정
DB_USER = "000"
DB_PASS = "000"
DB_HOST = "localhost"
DB_PORT = "1521"
DB_SERVICE = "FREE"

db_url = f"oracle+oracledb://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/?service_name={DB_SERVICE}"
engine = create_engine(db_url)

try:
    # 2. DB에서 데이터 읽어오기
    query = """
        SELECT sequence, variable, value
        FROM pca_sensor_data
    """
    print("DB에서 데이터를 읽어오는 중...")
    df_long = pd.read_sql(query, con=engine)
    df_long.columns = [col.lower() for col in df_long.columns]

    # 3. Pivot 변환
    print("Pivot 변환 중...")
    df_wide = df_long.pivot(index='sequence', columns='variable', values='value')

    # [데이터 정제 1] 무한대(Inf) 값을 NaN으로 변경 후 결측치 처리
    df_wide = df_wide.replace([np.inf, -np.inf], np.nan).ffill().bfill().fillna(0)

    # [데이터 정제 2] 표준편차가 0이거나 NaN인 죽은 센서 완전 제거
    std_series = df_wide.std()
    valid_sensors = std_series[(std_series > 1e-12) & (std_series.notna())].index
    df_wide = df_wide[valid_sensors]

    print(f"변환 및 정제 완료: {df_wide.shape[1]}개 유효 센서, {df_wide.shape[0]}개 시계열 포인트")

    # 4. 상관계수 행렬 계산
    print("상관계수 계산 중...")
    corr_matrix = df_wide.corr(method='pearson')

    # [데이터 정제 3] 이상치 및 범위(-1.0 ~ 1.0) 조율
    corr_matrix = corr_matrix.replace([np.inf, -np.inf], np.nan).fillna(0)
    corr_matrix = corr_matrix.clip(lower=-1.0, upper=1.0)

    # read-only 메모리 이슈 방지를 위한 안전한 대각선(1.0) 처리
    matrix_values = corr_matrix.to_numpy(copy=True)
    np.fill_diagonal(matrix_values, 1.0)
    corr_matrix = pd.DataFrame(matrix_values, index=corr_matrix.index, columns=corr_matrix.columns)

    # 5. 계층적 군집화 계산 (객체 생성)
    print("계층적 군집화 연산 중...")
    g = sns.clustermap(
        corr_matrix,
        method='average',
        metric='euclidean',
        cmap='vlag',
        vmin=-1, vmax=1,
        figsize=(14, 14),
        xticklabels=False,
        yticklabels=False,
        cbar_kws={'label': 'Correlation'}
    )
    plt.suptitle('Hierarchical Clustering of Sensors (Cleaned)', y=1.02, fontsize=16)

    # 6. 콘솔 출력 및 CSV 저장 (plt.show() 전에 실행)
    print("\n" + "="*60)
    print(" [센서 배치 및 정보 출력] ")
    print("="*60)

    # [A] 실제 군집화에 사용된 전체 센서 리스트
    participated_sensors = corr_matrix.columns.tolist()
    print(f"1. 군집화 최종 참여 센서 수: 총 {len(participated_sensors)}개")

    # [B] X축(좌->우) / Y축(상->하)에 재배치된 센서 순서 리스트
    axis_sensor_order = corr_matrix.columns[g.dendrogram_row.reordered_ind].tolist()

    print("\n2. 차트 [좌측 상단] 붉은 블록부터 배치된 센서 순서 (Top 30 예시):")
    for idx, sensor_name in enumerate(axis_sensor_order[:30], 1):
        print(f"   위치 {idx:03d}: {sensor_name}")

    # [C] 전체 순서를 CSV로 저장
    df_order = pd.DataFrame({
        'plot_position_index': range(1, len(axis_sensor_order) + 1),
        'sensor_name': axis_sensor_order
    })
    df_order.to_csv("clustermap_sensor_order.csv", index=False, encoding='utf-8-sig')
    print(f"\n3. 전체 {len(axis_sensor_order)}개 센서의 차트 배치 순서가 'clustermap_sensor_order.csv' 파일로 저장되었습니다.")
    print("="*60)
    print("콘솔 확인 완료. 차트 창을 엽니다...\n")

    output_filename = "clustermap_result.png"
    plt.savefig(output_filename, dpi=300, bbox_inches='tight')
    print(f"차트 이미지가 '{output_filename}' 파일로 저장되었습니다.")

    # ----------------------------------------------------
    # 7. 차트 이미지를 Base64 문자열로 변환
    # ----------------------------------------------------
    img_buf = io.BytesIO()
    plt.savefig(img_buf, format='png', dpi=300, bbox_inches='tight')
    img_buf.seek(0)
    base64_image_str = base64.b64encode(img_buf.read()).decode('utf-8')
    img_buf.close()

    # 8. 최종 요약 정보 Pandas DataFrame 생성
    # 요청하신 각 항목 매핑:
    # - EQP_ID: EQP_101
    # - Total_No_Sensor: 원본 DB 전체 센서 개수
    # - Total_Row: 전체 시계열 행(Sequence) 수
    # - Cluster_No_Sensor: 실제 군집화에 참여한 유효 센서 수
    # - Cluster_Sensor_List: 재배치된 센서 이름들의 numpy array
    # - Cluster_Result_Image: Base64 변환 이미지 문자열

    result_data = [{
        'EQP_ID': 'EQP_101',
        'Total_No_Sensor': int(df_long['variable'].nunique()),
        'Total_Row': int(df_wide.shape[0]),
        'Cluster_No_Sensor': int(len(axis_sensor_order)),
        'Cluster_Sensor_List': np.array(axis_sensor_order, dtype=object),
        'Cluster_Result_Image': base64_image_str
    }]

    df_result_info = pd.DataFrame(result_data)

    print("\n" + "="*60)
    print(" [생성된 최종 Pandas DataFrame (df_result_info)] ")
    print("="*60)
    print(df_result_info[['EQP_ID', 'Total_No_Sensor', 'Total_Row', 'Cluster_No_Sensor']])
    print(f"Cluster_Sensor_List 타입: {type(df_result_info['Cluster_Sensor_List'].iloc[0])}")
    print(f"Cluster_Result_Image 길 이: {len(df_result_info['Cluster_Result_Image'].iloc[0])} 자 (Base64 String)")
    print("="*60 + "\n")

    # 9. 차트 출력
    plt.show()

except Exception as e:
    print(f"오류 발생: {e}")
```

## Pandas (CLOB, BLOB) to Oracle Database
Panads DataFrame 에 모든 분석 결과들을 넣고, 이것을 Oracle Database 에 넣는 예제

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sqlalchemy import create_engine
from sqlalchemy.types import VARCHAR, INTEGER, CLOB, BLOB
import io

# 1. DB 접속 정보 설정
DB_USER = "000"
DB_PASS = "000"
DB_HOST = "localhost"
DB_PORT = "1521"
DB_SERVICE = "FREE"

db_url = f"oracle+oracledb://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/?service_name={DB_SERVICE}"
engine = create_engine(db_url)

try:
    # 2. DB에서 데이터 읽어오기
    query = """
        SELECT sequence, variable, value
        FROM pca_sensor_data
    """
    print("DB에서 데이터를 읽어오는 중...")
    df_long = pd.read_sql(query, con=engine)
    df_long.columns = [col.lower() for col in df_long.columns]

    # 3. Pivot 변환
    print("Pivot 변환 중...")
    df_wide = df_long.pivot(index='sequence', columns='variable', values='value')

    # [데이터 정제 1] 무한대(Inf) 값을 NaN으로 변경 후 결측치 처리
    df_wide = df_wide.replace([np.inf, -np.inf], np.nan).ffill().bfill().fillna(0)

    # [데이터 정제 2] 표준편차가 0이거나 NaN인 죽은 센서 완전 제거
    std_series = df_wide.std()
    valid_sensors = std_series[(std_series > 1e-12) & (std_series.notna())].index
    df_wide = df_wide[valid_sensors]

    print(f"변환 및 정제 완료: {df_wide.shape[1]}개 유효 센서, {df_wide.shape[0]}개 시계열 포인트")

    # 4. 상관계수 행렬 계산
    print("상관계수 계산 중...")
    corr_matrix = df_wide.corr(method='pearson')

    # [데이터 정제 3] 이상치 및 범위(-1.0 ~ 1.0) 조율
    corr_matrix = corr_matrix.replace([np.inf, -np.inf], np.nan).fillna(0)
    corr_matrix = corr_matrix.clip(lower=-1.0, upper=1.0)

    # read-only 메모리 이슈 방지를 위한 안전한 대각선(1.0) 처리
    matrix_values = corr_matrix.to_numpy(copy=True)
    np.fill_diagonal(matrix_values, 1.0)
    corr_matrix = pd.DataFrame(matrix_values, index=corr_matrix.index, columns=corr_matrix.columns)

    # 5. 계층적 군집화 계산 (객체 생성)
    print("계층적 군집화 연산 중...")
    g = sns.clustermap(
        corr_matrix,
        method='average',
        metric='euclidean',
        cmap='vlag',
        vmin=-1, vmax=1,
        figsize=(14, 14),
        xticklabels=False,
        yticklabels=False,
        cbar_kws={'label': 'Correlation'}
    )
    plt.suptitle('Hierarchical Clustering of Sensors (Cleaned)', y=1.02, fontsize=16)

    # 6. 재배치된 센서 리스트 추출
    participated_sensors = corr_matrix.columns.tolist()
    axis_sensor_order = corr_matrix.columns[g.dendrogram_row.reordered_ind].tolist()

    # 7. 차트 이미지를 BLOB(Raw Bytes) 형태로 추출
    img_buf = io.BytesIO()
    plt.savefig(img_buf, format='png', dpi=300, bbox_inches='tight')
    blob_image_bytes = img_buf.getvalue()
    img_buf.close()

    # 8. 최종 요약 정보 Pandas DataFrame 생성
    sensor_list_str = ",".join(axis_sensor_order)

    result_data = [{
        'eqp_id': 'EQP_101',
        'total_no_sensor': int(df_long['variable'].nunique()),
        'total_row': int(df_wide.shape[0]),
        'cluster_no_sensor': int(len(axis_sensor_order)),
        'cluster_sensor_list': sensor_list_str,
        'cluster_result_image': blob_image_bytes
    }]

    df_result_info = pd.DataFrame(result_data)

    # ----------------------------------------------------
    # 9. DB에 새로운 테이블 생성 및 저장
    # ----------------------------------------------------
    target_table_name = "cluster_analysis_result"
    print(f"\n타겟 테이블 '{target_table_name}' 생성 및 저장 중...")

    # SQLAlchemy 컬럼 타입 매핑 설정 (CLOB, BLOB 처리)
    dtype_mapping = {
        'eqp_id': VARCHAR(50),
        'total_no_sensor': INTEGER,
        'total_row': INTEGER,
        'cluster_no_sensor': INTEGER,
        'cluster_sensor_list': CLOB,
        'cluster_result_image': BLOB
    }

    # DB에 저장 (이미 테이블이 존재하면 Drop 후 새로 생성)
    df_result_info.to_sql(
        name=target_table_name,
        con=engine,
        if_exists='replace',
        index=False,
        dtype=dtype_mapping
    )

    print(f" 성공적으로 오라클 DB에 '{target_table_name}' 테이블이 생성되었으며 결과가 저장되었습니다.")
    print("="*60)

    # 10. 차트 출력
    plt.show()

except Exception as e:
    print(f"오류 발생: {e}")
```
![All Data Type](/assets/images/2026-07-30-120439.png)

-----------------
[pca.csv 파일 다운로드](/assets/data/pca.csv)