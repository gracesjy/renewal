---
title: PostgreSQL Handling for Analysis
img_path: /assets/images/
author: Raymond
date: 2025-10-15
category: [Utility, Database, PostgreSQL]
tags:
 - PostgreSQL
layout: post
---

## File -> PostgreSQL with Python

여러 DB 툴이 있기 때문에 그것들을 사용해도 된다. PostgreSQL 설치는 Docker 를 활용하면 손쉽게 설치가 가능하다.

--
여기에서는 Python 으로 간략하게 만드는 것을 보여준다.

CSV 파일의 맨 상단에 헤더가 있는지 확인하고 가급적이면 공백이 없는 것으로 잘 수정하거나 입력 해 둔다.  이것은 컬럼 이름이 되기 때문이다.

### Package Installation

```bash
pip install pandas sqlalchemy psycopg2-binary
```

### import csv to PostgreSQL
```python
import pandas as pd
from sqlalchemy import create_engine

# 1. 전달해주신 DB 접속 정보 설정
DB_USER = "raymond"
DB_PASS = "bdae_password"
DB_HOST = "localhost"
DB_PORT = "5432"
DB_NAME = "bdae_knowledge"

# 2. SQLAlchemy 엔진 생성
db_url = f"postgresql+psycopg2://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/{DB_NAME}"
engine = create_engine(db_url)

# 3. CSV 파일 및 저장할 DB 테이블명 설정
CSV_FILE_PATH = "pca.csv"
TABLE_NAME = "pca_sensor_data"

try:
    # 4. CSV 파일 읽기
    # 오라클에서 넘어온 CSV 한글 깨짐 방지를 위해 encoding='utf-8-sig' 또는 'utf-8' 지정
    df = pd.read_csv(CSV_FILE_PATH, encoding='utf-8-sig')
    print(f"CSV 읽기 완료! 데이터 크기: {df.shape} (행, 열)")

    # (선택 사항) Column 명 소문자 변환
    # PostgreSQL은 컬럼명에 대문자가 섞여있으면 큰따옴표("")로 감싸야 해서,
    # 아래처럼 모두 소문자로 변경해주면 추후 SQL 쿼리 작성이 편해집니다.
    df.columns = [col.lower() for col in df.columns]

    # 5. PostgreSQL에 데이터프레임을 테이블로 입력 (to_sql)
    # if_exists 옵션:
    #   - 'fail': 테이블이 이미 존재하면 에러 발생 (기본값)
    #   - 'replace': 기존 테이블이 있으면 삭제 후 새로 생성
    #   - 'append': 기존 테이블 뒤에 데이터 덧붙이기
    df.to_sql(
        name=TABLE_NAME,
        con=engine,
        if_exists='replace',   # 처음 생성하거나 덮어쓸 때 사용
        index=False,           # Pandas index 번호는 DB에 넣지 않음
        method='multi',        # 여러 행을 한 번에 insert하여 속도 향상
        chunksize=10000        # 데이터가 클 경우 1만 건씩 나누어 insert
    )

    print(f"성공적으로 PostgreSQL의 '{TABLE_NAME}' 테이블에 데이터를 밀어 넣었습니다.")

except Exception as e:
    print(f"오류 발생: {e}")

```

### 확인
DB 툴 (pgAdmin 또는 DBeaver)로 해당 테이블을 확인한다.

## PostgreSQL to Pandas DataFrame

```python
import pandas as pd
from sqlalchemy import create_engine, text

# 1. DB 접속 정보 설정
DB_USER = "raymond"       # 데이터베이스 사용자 이름
DB_PASS = "bdae_password"   # 비밀번호
DB_HOST = "localhost"     # 호스트 주소 (예: 127.0.0.1 또는 IP)
DB_PORT = "5432"          # 기본 포트번호
DB_NAME = "bdae_knowledge"   # 데이터베이스 이름

# 2. SQLAlchemy 엔진 생성 (연결 문자열 구조)
# postgresql+psycopg2://[USER]:[PASSWORD]@[HOST]:[PORT]/[DB_NAME]
db_url = f"postgresql+psycopg2://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/{DB_NAME}"
engine = create_engine(db_url)

# 3-A. 전체 테이블을 데이터프레임으로 가져오는 경우
df_table = pd.read_sql("manual_chunks", con=engine)
print(df_table.head())

# 3-B. 조건이 포함된 SQL 쿼리로 조회하는 경우 (권장)
query = """
SELECT id, doc_title, chunk_order, content, embedding FROM manual_chunks WHERE chunk_order=:chunk_order
"""

# 파라미터 바인딩을 위해 text() 함수 사용 (SQL Injection 예방)
df_query = pd.read_sql(
    sql=text(query),
    con=engine,
    params={"chunk_order": 1}
)

print(df_query)
```
## PostgreSQL to Pandas DataFrame and CSV file
```python
import pandas as pd
from sqlalchemy import create_engine

# 1. PostgreSQL DB 접속 정보 설정
DB_USER = "your_username"      # DB 사용자명 (예: postgres)
DB_PASS = "your_password"      # DB 비밀번호
DB_HOST = "localhost"          # DB 호스트 (예: localhost 또는 IP 주소)
DB_PORT = "5432"               # PostgreSQL 기본 포트
DB_NAME = "your_dbname"        # 데이터베이스 이름

# 2. SQLAlchemy 연결 엔진 생성
# 접속 URL 형식: postgresql://사용자명:비밀번호@호스트:포트/DB이름
db_url = f"postgresql://{DB_USER}:{DB_PASS}@{DB_HOST}:{DB_PORT}/{DB_NAME}"
engine = create_engine(db_url)

try:
    # 3. 가져올 테이블 이름 설정
    table_name = "sensor_data"  # 가져오고자 하는 PostgreSQL 테이블명
    
    print(f"PostgreSQL '{table_name}' 테이블 데이터를 읽어오는 중...")
    
    # 방법 A: 테이블 전체를 가져오는 경우
    df = pd.read_sql_table(table_name, con=engine)
    
    # 방법 B: 특정 조건이나 SQL 쿼리로 가져오고 싶은 경우 (주석 해제 후 사용)
    # query = f"SELECT * FROM {table_name} WHERE value > 10"
    # df = pd.read_sql_query(query, con=engine)

    print(f"데이터 로드 완료! (행: {df.shape[0]}개, 열: {df.shape[1]}개)")

    # 4. DataFrame을 CSV 파일로 저장
    output_csv = "exported_sensor_data.csv"
    
    # index=False : DataFrame의 인덱스 번호(0, 1, 2...)는 CSV에 저장하지 않음
    # encoding='utf-8-sig' : 엑셀(Excel)에서 한글 깨짐 없이 안전하게 열기 위한 옵션
    df.to_csv(output_csv, index=False, encoding='utf-8-sig')
    
    print(f"'{output_csv}' 파일로 성공적으로 저장되었습니다.")

except Exception as e:
    print(f"오류 발생: {e}")
```

---------

[pca.csv 파일 다운로드](/assets/data/pca.csv)