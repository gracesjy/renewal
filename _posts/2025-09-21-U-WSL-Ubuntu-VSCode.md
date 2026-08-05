---
title: VS Code under WSL Ubuntu
img_path: /assets/images/
author: Alex
math: true
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

## WSL Ubuntu 에 Git 관련 설정
공개 레포지터리(public repository) 가 아닌 자신의 비공개 등에 접근하기 위해서는 다음의 순서로 설치를 먼저 해 주어야 한다.

```bash
sudo apt update && sudo apt install -y gh
sudo apt update && sudo apt install -y wslu
sudo bash -c 'echo -e "[interop]\nenabled=true\nappendWindowsPath=true" >> /etc/wsl.conf'
```

다음에 로그인을 해야 한다.
```bash
gh auth login
```

주의할 것은 디폴트 선택을 하면 무난하다.  웹 브라우저가 떠서 위에서 보여주는 콘솔의 코드를 붙여 넣기를 해야 한다는 점(1), 그리고 2FA 때문에 모바일에서 Confirm 을 해 주어야 최종적으로 모든 것이 마무리 된다.

아래 WSL Ubuntu 에서 VS Code 를 실행 시키고 거기에서 commit 을 하려면 반드시 다음 명령어를 WSL Ubuntu 의 콘솔에서 입력해야 한다.  안하면 VS Code 의 Commit 을 누르면 오류가 난다.

```bash
git config --global user.name "본인이름 또는 GitHub 계정명"
git config --global user.email "GitHub이메일주소@example.com"
```

## VS Code 의 Python 설정

최초에 파이썬 파일 (.py) 를 선택하면 다음 처럼 실행 버튼이 없다. 이는 Python Interpreter 와 연결이 안되어 있다는 증거라서 설정해 주어야 한다.

![2026-08-04-120503.png](/assets/images/2026-08-04-120503.png)


VS Code 에서 Ctrl + Shift + P 를 누르면 Python: Select Interpreter 를 치면 나와야 한다. 그런데, 안나오는 경우가 있다.  그때에는 아래 처럼 패키지에서 일단 Python 을 하나 설치하면 된다.  

패키지에서 Python 을 치면 WSL Ubuntu 관련 패키지들이 나오는데, 
![2026-08-04-122002.png](/assets/images/2026-08-04-122002.png)

그 이후에는 Ctrl + Shift + P 를 누르고 Python: Select Interpreter 를 치면 다음 처럼 목록이 나온다. 이것들은 WSL Ubuntu 에서 conda create -n .. 으로 만든 가상 환경이다. 이 중에 맞는 것을 선택하면 된다.  만약 이것이 안되면 Enter interpreter path.. 를 눌러서 직접 Python 위치를 정확히 넣으면 된다.
![2026-08-04-135155.png](/assets/images/2026-08-04-135155.png)

잘 세팅되면 다음과 같이 실행 버튼이 생긴다.
![2026-08-04-140524.png](/assets/images/2026-08-04-140524.png)


