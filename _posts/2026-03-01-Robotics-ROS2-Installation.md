---
title: ROS2 Installation
img_path: /assets/images/
author: Alex
math: true
date: 2026-03-01
category: [Robotics, ROS2, ROS2 Install]
tags:
 - ROS2 jazzy Install
layout: post
---

## ROS2 개발 환경으로 Ubuntu를 최우선 추천하는 이유

ROS2는 Windows, macOS 등 다양한 OS를 공식 지원하지만, 실무 및 연구 분야에서는 **Ubuntu Linux**가 사실상의 표준(De Facto Standard)으로 사용됩니다. 그 주요 이유는 다음과 같습니다.

---

### 1. Official Reference OS (공식 기준 및 최우선 지원)
* **Tier 1 지원:** Open Robotics(ROS 개발 재단)는 ROS2 버전을 출시할 때 특정 **Ubuntu LTS(장기 지원) 버전**을 **Tier 1 Target Platforms**로 지정합니다.
  * *예시:* ROS2 Humble / Iron / Jazzy 등의 메인 타깃 OS는 항상 Ubuntu LTS 버전입니다.
* **최속 업데이트 및 버그 수정:** 새로운 기능, 보안 패치, 버그 수정이 Ubuntu 환경에 가장 먼저 적용되고 완벽하게 테스트됩니다.

### 2. 패키지 관리의 편의성 (`apt` 패키지 생태계)
* **간단한 설치:** Ubuntu에서는 `sudo apt install ros-<distro>-desktop` 단 한 줄 명령어로 ROS2 핵심 프레임워크와 주요 종속성 라이브러리를 손쉽게 설치할 수 있습니다.
* **Windows/macOS의 한계:** 다른 OS에서는 소스 코드 직접 빌드(Build from source)나 복잡한 환경 변수/의존성 설정이 필요한 경우가 많아 초기 구성 난이도가 높습니다.

### 3. 로봇 하드웨어 및 임베디드 디바이스 호환성
* **온보드 컴퓨터의 표준:** 로봇에 탑재되는 싱글 보드 컴퓨터(NVIDIA Jetson, Raspberry Pi, x86 온보드 PC 등)의 기본 운영체제가 대부분 **Ubuntu 기반 Linux**입니다.
* **하드웨어 드라이버 지원:** LiDAR, 카메라(RealSense 등), 모터 컨트롤러, CAN 통신 카드 등 로봇 하드웨어 제조사에서 제공하는 드라이버(SDK)가 Linux/Ubuntu를 최우선으로 지원합니다.

### 4. 커뮤니티 생태계 및 서드파티 오픈소스 (Navigation2, MoveIt 등)
* **풍부한 레퍼런스:** 전 세계 ROS2 사용자 및 연구진의 80% 이상이 Ubuntu 환경에서 개발하기 때문에, 에러 발생 시 커뮤니티(ROS Answers, GitHub Issue)에서 해결 방법을 구하기가 훨씬 쉽습니다.
* **오픈소스 패키지 호환성:** Navigation2, MoveIt2, Gazebo, SLAM 등의 핵심 툴 및 커뮤니티 오픈소스 패키지들이 Ubuntu 환경에서 가장 안정적으로 동작하도록 개발되어 있습니다.

### 5. 실시간 성능 (Real-Time Kernel: PREEMPT_RT)
* **Real-time ROS2 지원:** 모터 제어나 산업용 로봇 arm 제어 등 엄격한 결정을 내리는 하드웨어 제어에는 **실시간 OS(RTOS)** 성능이 필수적입니다.
* **Linux 커널 커스텀:** Ubuntu Linux는 커널 패치(`PREEMPT_RT`)를 적용하여 용이하게 실시간 커널 환경을 구축할 수 있으며, ROS2의 Real-Time 통신 기능을 최대로 활용할 수 있습니다.

---

### 요약

Ubuntu를 사용하는 이유는 **"ROS2 개발 재단과 라이브러리 개발자, 하드웨어 제조사, 글로벌 커뮤니티가 모두 Ubuntu를 1순위 기준으로 개발하기 때문"**입니다. 따라서 에러를 최소화하고 개발 생산성을 높이기 위해 Ubuntu 환경 구축이 권장됩니다.

## Ubuntu 및 ROS2 버전에 따른 서비스 종료 시점(EOL) 비교

Ubuntu의 LTS(Long Term Support) 버전과 ROS2의 LTS 버전은 각각 **5년 지원 주기**를 가지며 1:1로 매칭됩니다.

---

### 1. 버전별 서비스 종료 시점 비교

| 구분 | 조합 1 (이전 권장 LTS) | 조합 2 (최신 권장 LTS) |
| :--- | :--- | :--- |
| **Ubuntu 버전** | **Ubuntu 22.04 LTS** (Jammy Jellyfish) | **Ubuntu 24.04 LTS** (Noble Numbat) |
| **ROS2 버전을 위한 배포판** | **ROS2 Humble Hawksbill** | **ROS2 Jazzy Jalisco** |
| **출시일** | 2022년 5월 | 2024년 5월 |
| **서비스 종료 시점 (EOL)** | **2027년 5월** | **2029년 5월** |
| **지원 기간** | 5년 | 5년 |

---

### 2. 핵심 차이 및 권장사항

1. **ROS2 Humble (Ubuntu 22.04)**
   * **EOL:** **2027년 5월** 종료
   * **특징:** 가장 안정성이 검증되어 있으며, 수많은 서브파티 패키지와 오픈소스 라이브러리가 완벽하게 지원되는 대표적인 안정 버전입니다.

2. **ROS2 Jazzy (Ubuntu 24.04)**
   * **EOL:** **2029년 5월** 종료
   * **특징:** 최신 LTS 버전으로, Humble 대비 지원 기간이 2년 더 깁니다. 신규 프로젝트나 장기 프로젝트에 권장됩니다.
   * **주의점:** 시뮬레이터 사용 시 기존 Gazebo Classic은 지원하지 않으며, **New Gazebo(Harmonic 등)** 버전을 표준으로 사용합니다.


 ## Ubuntu 24.04 LTS 환경에서 ROS 2 Jazzy Jalisco 설치 순서

가장 안정적이고 권장되는 **APT 데비안(Debian) 패키지 방식** 설치 단계입니다. 터미널(`Ctrl + Alt + T`)을 열고 순서대로 명령어를 입력하시면 됩니다.

---

### Step 1. 로케일(Locale) UTF-8 설정
ROS 2는 UTF-8 환경을 필요로 합니다.
```bash
sudo apt update && sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

### Step 2. Ubuntu Universe 저장소 및 필수 툴 설치
```bash
sudo apt install software-properties-common -y
sudo add-apt-repository universe -y
sudo apt update && sudo apt install curl -y
```
### Step 3. ROS 2 APT 저장소 키 및 키링 등록

```bash
export ROS_APT_SOURCE_VERSION=$(curl -s [https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest](https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest) | grep -F "tag_name" | awk -F'"' '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "[https://github.com/ros-infrastructure/ros-apt-source/releases/download/$](https://github.com/ros-infrastructure/ros-apt-source/releases/download/$){ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb
```

### Step 4. 시스템 패키지 업데이트 및 ROS 개발 툴 설치
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ros-dev-tools -y

```

### Step 5. ROS 2 Jazzy 설치
보통 시각화 도구(RViz2), 데모, 튜토리얼이 모두 포함된 Desktop 버전 설치를 권장합니다.
```bash
# 데스크톱 버전 (권장: GUI, RViz2, 튜토리얼 포함)
sudo apt install ros-jazzy-desktop -y

# (참고) 최소 통신 라이브러리만 필요한 경우 (GUI 미포함):
# sudo apt install ros-jazzy-ros-base -y
```
### Step 6. 환경 변수 설정 (Auto-Sourcing)
터미널을 열 때마다 ROS 2 명령어를 사용할 수 있도록 ~/.bashrc에 자동 로드 설정을 추가합니다.

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### Step 7. 설치 정상 동작 확인 (Talker / Listener 테스트)
설치가 완료되었는지 확인하기 위해 2개의 터미널에서 간단한 통신 테스트를 진행합니다.  아래 두가지의 통신이 완성되면 Jazzy 설치가 끝났다고 볼 수 있다.  

#### 1. 첫 번째 터미널 (Talker 실행):
```bash
ros2 run demo_nodes_cpp talker
```
#### 2. 두 번째 터미널 (Listener 실행):
```bash
ros2 run demo_nodes_py listener
```
아래 캡처한 것 처럼 데이터 Pub/Sub 으로 데이터 통신이 되는 것을 확인할 수가 있다.
![2026-08-04-151419.png](/assets/images/2026-08-04-151419.png)

## Vmware 에서 Ubuntu 24.04 LTS Desktop 설치
WSL Ubuntu 에서 ROS2 를 설치해도 문제가 되는 것은 Multicast 통신 때문이다. 이것은 한계라고 볼 수 있다.  그러나, Vmware 에서 Network 을 Brideged 로 설정하면 원격의 라즈베리파이의 노드의 데이터를 받아 볼 수가 있다.  물론 Windows 방화벽 처리만 되면 말이다.   그러나, WSL Ubuntu 는 그 상태에서도 받기 어렵다.

### WSL에서 ROS2를 사용할 때의 한계
ROS2는 노드를 자동으로 찾기 위해 DDS(Data Distribution Service) 를 사용한다.  DDS는 기본적으로 Multicast를 이용하여 같은 네트워크에 있는 다른 ROS2 노드를 자동으로 발견(Discovery)한다.  따라서 일반적인 Ubuntu에서는 별도의 설정 없이도 다른 컴퓨터의 ROS2 노드를 자동으로 찾을 수 있다.

![2026-08-04-150851.png](/assets/images/2026-08-04-150851.png)

그러나, WSL 구조는 다르다.

![2026-08-04-151048.png](/assets/images/2026-08-04-151048.png)

그래서 WSL 기반 Ubuntu 에서는 DDS 데이타 통신이 되지 않는다.  따라서 ROS2 는 Vmware 를 사용하는 것을 강력히 추천한다.