---
title: VS Code under WSL Ubuntu
img_path: /assets/images/
author: Alex
date: 2025-09-20
category: [Utility, WSL Ubuntu, Ubuntu 24.04 LTS]
tags:
 - WSL
 - Ubuntu 24.04 LTS
 - VS Code
layout: post
---

## WSL, WSL Ubuntu, Anancoda 를 모두 설치한 이후에 진행
WSL 설치, WSL Ubuntu, Anacoda 를 모두 설치한 이후에 다음을 진행 한다.

```bash
(base) raymond@ASUS:~$ conda env list
# conda environments:
#
base                  *  /home/raymond/anaconda3
alex                     /home/raymond/anaconda3/envs/alex

(base) raymond@ASUS:~$ conda activate alex
(alex) raymond@ASUS:~$
(alex) raymond@ASUS:~$
(alex) raymond@ASUS:~$ cd oracledb
(alex) raymond@ASUS:~/oracledb$ ls -rtl
total 19052
-rw-r--r-- 1 raymond raymond 19501401 Jul 30 15:37 pca.csv
-rw-r--r-- 1 raymond raymond     2150 Aug  4 11:38 pca_import.py
(alex) raymond@ASUS:~/oracledb$ code .
Installing VS Code Server for Linux x64 (e4c7e7b1d6d060162f4aa7f8225271b67ce1df75)
Downloading: 100%
Unpacking: 100%
Unpacked 3598 files and folders to /home/raymond/.vscode-server/bin/e4c7e7b1d6d060162f4aa7f8225271b67ce1df75.
Looking for compatibility check script at /home/raymond/.vscode-server/bin/e4c7e7b1d6d060162f4aa7f8225271b67ce1df75/bin/helpers/check-requirements.sh
Running compatibility check script
Compatibility check successful (0)
```

이제 VS Code 가 기동되었을 것이다.  윈도우에서 기동한 것과는 약간 다르며 반드시 VS Package 에 WSL Extension 이 설치되어 있어야 한다.

![2026-08-04-115637.png](/assets/images/2026-08-04-115637.png)

최초 기동 시에는 다음과 같다.  주의 해야 할 사항은 왼쪽 하단의 WSL:ubuntu-24.04 와 Restricted Mode 이다.  먼저 WSL 이 없다면 이것은 윈도우에서 평범하게 실행된 모습이고, WSL 있다면 원격과 연결된 것이다. 

![2026-08-04-115931.png](/assets/images/2026-08-04-115931.png)

그리고 Restricted Mode 는 반드시 Trust 로 만들어주어야 한다. Restricted Mode 를 누르면 아래 처럼 보이게 되는데 거기에서 Trust 를 누르면 된다.  
![2026-08-04-120159.png](/assets/images/2026-08-04-120159.png)

그러면 Restricted Mode 는 없어지고 여기에서 정상적으로 파일을 편집하고 생성 하는 등의 작업을 할 수 있게 된다.


## VS Code 의 Python 설정

최초에 파이썬 파일 (.py) 를 선택하면 다음 처럼 실행 버튼이 없다. 이는 Python Interpreter 와 연결이 안되어 있다는 증거라서 설정해 주어야 한다.

![2026-08-04-120503.png](/assets/images/2026-08-04-120503.png)

패키지에서 Python 을 치면 WSL Ubuntu 관련 패키지들이 나오는데, 
![2026-08-04-122002.png](/assets/images/2026-08-04-122002.png)