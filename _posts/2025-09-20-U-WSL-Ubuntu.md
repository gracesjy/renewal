---
title: WSL Ubuntu, Anaconda, Oracle Import
img_path: /assets/images/
author: Alex
math: true
date: 2025-09-20
category: [Utility, WSL Ubuntu, Ubuntu 24.04 LTS]
tags:
 - WSL
 - Ubuntu 24.04 LTS
 - Anaconda
 - Oracle
layout: post
---

## WSL 설치 후 Power Shell

WSL 에 여러개의 Ubuntu 및 다른 Linux 를 설치할 수 있다.  먼저 Power Shell 을 기동 시킨다. 이때 굳이 Administrator 권한으로 할 필요는 없다.  만약 필요하면 그때 다시 해도 늦지 않다.

아래는 Ubuntu 24.04 LTS 나 Ubuntu 22.04 LTS, 또 다른 WSL 기반 Linux 버전을 설치할 수 있는 목록을 확인하기 위한 것이다.

```shell
wsl --list --online
```

![2026-08-04-103450.png](/assets/images/2026-08-04-103450.png)


## Ubuntu 24.04 LTS 설치
이것을 설치하려면 위의 설치 가능한 것을 보고 그대로 아래와 같이 타이핑 한다.

```shell
wsl --install -d Ubuntu-24.04
```
설치가 다 되면 바로 Ubuntu-24.04 에 들어가니 exit 로 빠져 나온다. 
![2026-08-04-103608.png](/assets/images/2026-08-04-103608.png)

## WSL 설치된 목록 확인

설치된 WSL 기반 SW 목록은 다음과 같이 확인한다.
```shell
wsl -l -v
```

![2026-08-04-103716.png](/assets/images/2026-08-04-103716.png)

## Ubuntu 24.04 바로 가기

윈도우 검색 창에서 Ubuntu 를 찾아서 파일 목록이나, 작업 표시줄 등에 등록하면 손쉽게 다음부터는 아이콘 등을 눌러서 Ubuntu 를 실행할 수 있을 것이다.

![2026-08-04-104036.png](/assets/images/2026-08-04-104036.png)

Power Shell 에서 바로 들어가기 위해서는 다음과 같이 한다.  만약 Ubuntu 의 2가지 버전을 모두 쓸 때에는 주의를 기울여야 한다.  아이콘 클릭에 들어 갔을 때 State 이 Running 쪽으로 갈 수 있기 때문이다.  아래 처럼 명시적으로 wsl -d Ubuntu-24.04 등으로 하는 것이 안전하다.

```shell
(base) PS C:\Users\raymond> wsl -l -v
  NAME              STATE           VERSION
* Ubuntu            Running         2
  Ubuntu-24.04      Running         2
  docker-desktop    Running         2
(base) PS C:\Users\raymond> wsl -d Ubuntu-24.04
(base) raymond@ASUS:/mnt/c/Users/raymond$ cd
(base) raymond@ASUS:~$ pwd
/home/raymond
(base) raymond@ASUS:~$ conda env list
# conda environments:
#
base                  *  /home/raymond/anaconda3

(base) raymond@ASUS:~$
```

## Anaconda 설치

anaconda 를 설치하기 위해서는 다음을 기본적으로 설정해야 한다.  이는 Ubuntu 24.04 LTS 기반이다.
```shell
sudo apt update && sudo apt install -y wget bzip2 libgl1 libglx-mesa0
cd ~
wget https://repo.anaconda.com/archive/Anaconda3-2024.02-1-Linux-x86_64.sh
bash Anaconda3-2024.02-1-Linux-x86_64.sh
```

## 첫번째 Python 개발 환경 설치
먼저 간단하게 Oracle Database 기반으로 CSV 파일을 테이블로 만드는 간단한 예제를 해 보자.

### 가상환경
```bash
(base) raymond@ASUS:~$ conda env list
(base) raymond@ASUS:~$ conda create -n alex python=3.12

#
# To activate this environment, use
#
#     $ conda activate alex
#
# To deactivate an active environment, use
#
#     $ conda deactivate

(base) raymond@ASUS:~$ conda env list
# conda environments:
#
base                  *  /home/raymond/anaconda3
alex                     /home/raymond/anaconda3/envs/alex

(base) raymond@ASUS:~$ conda activate alex
(alex) raymond@ASUS:~$ mkdir oracledb
(alex) raymond@ASUS:~$ cd oracledb
(alex) raymond@ASUS:~/oracledb$

```

### 윈도우의 CSV 파일을 위의 새로 만든 디렉토리에 넣기

아래 처럼 실행하면 탐색기가 기동한다.  CSV 파일을 복사해서 넣으면 된다.
```bash
(alex) raymond@ASUS:~/oracledb$ explorer.exe .

```
![2026-08-04-113548.png](/assets/images/2026-08-04-113548.png)

### CSV to Oracle DB 
이제 이 작업을 위한 Python 패키지들을 먼저 설치해 보자. 그 전에 다음을 확인 !! 이것을 확인하는 것은 엉뚱한 곳에 Python 패키지가 저장되는 것을 막기 위해서이다.  alex 라는 가상 환경에 관련된 pip 프로그램 위치를 정확하게 확인해야 한다.  아래는 OK.
```bash
(alex) raymond@ASUS:~/oracledb$ which pip
/home/raymond/anaconda3/envs/alex/bin/pip
```

오라클 데이터베이스 (Oracle Database) 작업을 위해서는 다음을 설치해야 한다.
```bash
pip install pandas sqlalchemy oracledb
```
![2026-08-04-114156.png](/assets/images/2026-08-04-114156.png)
![2026-08-04-114219.png](/assets/images/2026-08-04-114219.png)

파이썬 소스는 다음과 같다.  이것은 AI 가 만들어 준 것이고 오라클 연결 정보만 맞게 수정하면 된다.
```python
import pandas as pd
import numpy as np
from sqlalchemy import create_engine
from sqlalchemy.dialects.oracle import NUMBER, FLOAT, VARCHAR2

# 1. 오라클 DB 접속 정보
DB_USER = "alex"
DB_PASS = "alex1020"
DB_HOST = "localhost"
DB_PORT = "1521"
DB_SERVICE = "FREEPDB1"

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

위의 소스를 pca.csv 위치와 동일하게 위치해서 pcs_import.py 등 적당한 파일이름으로 한뒤에 아래를 실행해 보면

```bash
(alex) raymond@ASUS:~/oracledb$ ls -rtl *.py
-rw-r--r-- 1 raymond raymond 2150 Aug  4 11:38 pca_import.py
(alex) raymond@ASUS:~/oracledb$ python ./pca_import.py
CSV 파일 읽기 완료: 총 924,530행, 3개 컬럼
생성된 컬럼 매핑: {'SEQUENCE': <class 'sqlalchemy.dialects.oracle.types.NUMBER'>, 'VARIABLE': VARCHAR2(length=255), 'VALUE': <class 'sqlalchemy.dialects.oracle.types.NUMBER'>}
성공적으로 오라클 DB의 'pca_sensor_data' 테이블에 데이터를 적재했습니다.
```

DBeaver 로 접속해서 다음과 같이 Refresh(F5) 해 본다.
![2026-08-04-114536.png](/assets/images/2026-08-04-114536.png)

이제 새로 생긴 테이블이 존재함을 알 수 있다.
![2026-08-04-114614.png](/assets/images/2026-08-04-114614.png)