---
title: ROS2 ESP-IDF
img_path: /assets/images/
author: Alex
math: true
date: 2026-03-04
category: [Robotics, ROS2, ESP-IDF]
tags:
 - ROS2
 - SLAM
 - ESP32
 - PCB
layout: post
---

## 개요

micro-ROS는 리소스가 제한된 임베디드 마이크로컨트롤러(MCU) 환경에서 ROS2 도메인과 직접 통신할 수 있도록 설계된 프레임워크이며, ESP32 계열 칩셋을 개발하는 ESP-IDF(Espressif IoT Development Framework) 환경은 FreeRTOS 기반의 강력한 네트워크 및 하드웨어 제어 기능을 제공하므로, micro-ROS를 탑재하기에 매우 이상적인 조합이다.

![2026-08-06-132307.png](/assets/images/2026-08-06-132307.png)

- Client (ESP32 / ESP-IDF): rclc C 클라이언트 라이브러리를 통해 퍼블리셔/서브스크라이버/서비스를 생성합니다.

- Transport (통신 방식): UART(Serial), Wi-Fi(UDP/TCP), 또는 Custom Transport(CAN 등)를 사용해 데이터를 전송합니다.

- Agent (PC / ROS 2 Host): ESP32로부터 들어오는 XRCE-DDS 메세지를 해석하여 메인 ROS 2 도메인 네트워크(DDS)로 중계합니다.


## ESP32 칩셋
ESP32는 중국의 Espressif Systems가 개발한 Low-Power, Low-Cost 32비트 마이크로컨트롤러(MCU) 시스템 온 칩(SoC) 시리즈로, Wi-Fi와 Bluetooth(또는 Zigbee/Thread) 통신 모듈을 칩 내부에 기본 탑재하고 있어 IoT(사물인터넷), 스마트홈, 로보틱스, 엣지 AI 등의 분야에서 사실상의 표준 MCU로 널리 사용되고 있다.


### 라인업 및 시리즈별 특징

초기 오리지널 ESP32 모델 이후 성능, 용도, 가격에 맞춰 여러 계열로 라인업이 다변화되었습니다.

| 시리즈 | 코어 아키텍처 | 무선 통신 | 주요 타겟 및 특징 |
| :--- | :--- | :--- | :--- |
| **ESP32 (Original)** | Xtensa LX6 (듀얼/싱글코어) | Wi-Fi 4 + BT Classic + BLE | 풍부한 기존 레퍼런스, 클래식 블루투스가 필요한 경우 |
| **ESP32-S3** | Xtensa LX7 (듀얼코어) | Wi-Fi 4 + BLE 5.0 | **고성능 / AI / 카메라**: 벡터 명령어(Vector Ext.) 기반 엣지 AI, 카메라/디스플레이 제어 |
| **ESP32-C3** | RISC-V (싱글코어) | Wi-Fi 4 + BLE 5.0 | **보급형 / 저전력**: ESP8266의 직계 후속작, 초저가 센서 노드 구축에 적합 |
| **ESP32-C6** | RISC-V (싱글코어) | **Wi-Fi 6** + BLE 5.3 + Thread/Zigbee | **차세대 스마트홈**: Matter 프로토콜 완벽 지원, Wi-Fi 6 네트워크 탑재 |
| **ESP32-H2** | RISC-V (싱글코어) | BLE + Thread/Zigbee **(Wi-Fi 없음)** | **초저전력 메쉬 네트워크**: 배터리로 동작하는 Matter/Zigbee 전용 스마트 디바이스 |
| **ESP32-P4** | RISC-V (고성능 듀얼코어) | 무선 기능 없음 (Ethernet 등 유선/외부모듈) | **HMI / 고성능 연산**: 최대 400MHz, 복잡한 UI/음성인식 처리용 |

### ESP32 모델별 micro-ROS 적합성 비교

ESP32 주요 모델별 micro-ROS 지원 여부 및 시스템 사양 비교입니다.

| 구분 | 오리지널 ESP32 (Xtensa 듀얼코어) | ESP32-C3 (RISC-V 싱글코어) | ESP32-S3 (Xtensa 듀얼코어) |
| :--- | :--- | :--- | :--- |
| **micro-ROS 지원 여부** | **완벽 지원** (검증된 레퍼런스 다수) | **지원** | **완벽 지원** (자율주행/AMR 표준) |
| **SRAM (내장)** | 520 KB | 400 KB | 512 KB |
| **PSRAM (외장 확장)** | 최대 4MB~8MB (Quad SPI) | 미지원 | **최대 8MB~16MB (Octal SPI)** |
| **태스크 분리** | Core 0 (Wi-Fi/micro-ROS) / Core 1 (제어) | 싱글코어로 시분할 처리 필요 | Core 0 (Wi-Fi/micro-ROS) / Core 1 (제어) |
| **연산 유닛** | FPU 지원 | FPU 없음 (소프트웨어 연산) | **FPU + Vector Extension** |
| **시리얼 통신** | 외부 USB-to-UART 칩 경유 | Native USB CDC | **Native USB-JTAG/CDC** (고속 Mbaud) |
| **추천 용도** | 일반 모바일 로봇, 엔코더/센서 노드 | 단일 센서 전용 소형 노드 | **자율주행 차량(AMR), 복잡한 IK 연산, 카메라 노드** |

### ESP32-S3 와 micro-ROS

자율주행 자동차와 같은 경우에는 ***ESP32-S3-N16R8*** 같은 16MB Flash / 8MB PSRAM 모델을 추천한다. 대부분의 ESP32 자율주행보드들은 대부분 이것을 채용하고 있다.

구매할 때 반드시 ***ESP32-S3-N16R8*** 을 선택하는 것을 강력히 추천한다.


## 개발환경

- ESP-IDF: Espressif의 공식 C/C++ SDK로, 메모리와 성능 제어에 최적화된 프레임워크.  
- Arduino IDE: 초보자 및 빠른 프로토타이핑에 매우 용이한 환경.
- MicroPython / CircuitPython: 파이썬 언어로 손쉽게 하드웨어 제어 가능.

### Arduino vs ESP-IDF 비교

ESP32 기반 개발 시 가장 대표적인 두 프레임워크의 환경 비교.

| 구분 | Arduino (micro_ros_arduino) | ESP-IDF (micro_ros_esp_idf_component) |
| :--- | :--- | :--- |
| **개발 난이도** | 낮음 (빠른 프로토타이핑 및 진입 장벽 낮음) | 보통~높음 (C 기반 CMake/Kconfig 빌드 시스템 구조) |
| **태스크 및 멀티코어 제어** | 제약 있음 (기본 `loop()` 구조 중심) | FreeRTOS 태스크, 큐, 세마포어, 코어별 할당 완전 활용 |
| **통신 스택 및 페리페럴** | 기본 라이브러리 추상화 레이어에 의존 | Wi-Fi Power Save, IP 스택(lwIP), 하드웨어 드라이버 상세 세팅 가능 |
| **메모리 최적화** | 컴파일 타임 최적화 및 바이너리 다이어트에 한계 | Kconfig로 미사용 기능 제거 가능, RAM/Flash 사용량 최소화 |
| **개발 도구 및 디버깅** | Arduino IDE, PlatformIO | VS Code ESP-IDF Extension, GDB Debugging, SystemView |
| **추천 용도** | 개인 프로젝트, 빠른 개념 검증(PoC), 단일 노드 | **양산 제품 개발, 실시간 멀티태스크 로봇 제어, 자율주행 AMR** |

## ESP-IDF 환경

결국 micro-ROS 개발 환경은 ESP-IDF 환경에서 하는 것이 보편적인 듯 하다.  자율주행 상용 제품들의 소스들과 자율주행 환경이 대부분 이것으로 되어 있는 것이 그 방증이 아닐까 한다.

### Vmware 환경
ESP-IDF 환경으로는 Ubuntu 환경을 접했다.  그런데 WSL Ubuntu 는 문제가 있다.  ESP32 개발보드에 Firmware 업로드 시에 문제가 있다. 그래서 Vmware 를 추천한다.  이러한 일들이 가장 좋다.  호스트 (PC)의 USB 를 그대로 게스트로 가져올 수 있기 때문이다.

Vmware Workstation 을 설치하고 Ubuntu 24.04 LTS 를 설치하고 이에 맞는 ROS2 Jazzy 를 설치한 상태에서 ESP-IDF 환경을 설정하는 것이 좋겠다.  어짜피 micro-ROS 만으로는 안되기 때문에 micro-ROS agent 가 필요하기 때문이기도 하고, ROS2 의 rviz 및 다양한 UI 가 필요한 것들이 모두 필요하기 때문이다.

ROS2 humble(under Ubuntu 22.04 LTS)의 만료일(EOL)은 2027년 5월 23일이므로 이제는 ROS2 jazzy 로 가야 한다.  그리고 이것은 Ubuntu 24.04 LTS 기반이기 때문이다.

### Vmware Network 환경
Vmware 에서 Ubuntu 를 설치할 때 Network 는 반드시 다음과 같은 ***bridged*** 로 해야 한다.  그건 ROS2 의 DDS 통신 때문이다.
![2026-08-06-134655.png](/assets/images/2026-08-06-134655.png)

### 설치된 것들
작업한 Ubuntu 셀의 이력들이다.  순서대로 보면 될 듯하다.

```bash
sudo shutdown -h now
sudo apt update
sudo apt install open-vm-tools-desktop -y
sudo shutdown -h now

sudo apt update
sudo apt install terminator -y
terminator
sudo apt install net-tools
sudo apt purge update-notifier
sudo apt update && sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
sudo apt install software-properties-common -y
sudo add-apt-repository universe -y
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update
sudo apt upgrade -y
sudo apt install ros-jazzy-desktop -y
sudo apt install ros-dev-tools -y
vi .bashrc
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source . ./.bashrc
source ~/.bashrc
ls -rtl
rviz2
idf.py
cd esp
cp -r $IDF_PATH/examples/get-started/hello_world .
get_idf
sudo apt update
sudo apt install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0 -y
mkdir -p ~/esp
cd esp
git clone --recursive -b v5.3.1 https://github.com/espressif/esp-idf.git
cd esp-idf
./install.sh esp32
cd
echo "alias get_idf='. \$HOME/esp/esp-idf/export.sh'" >> ~/.bashrc
source ~/.bashrc
which idf.py
# 다이얼아웃(dialout) 그룹에 현재 로그인된 사용자 추가
sudo usermod -a -G dialout $USER
get_idf
cd cp -r $IDF_PATH/examples/get-started/hello_world .
cp -r $IDF_PATH/examples/get-started/hello_world .
cd hell*
ls -rtl
idf.py build
idf.py flash monitor
ls -rtl /dev/ttyACM*
sudo usermod -a -G dialout $USER
cd micro*
ls -rtl
rm -rf build log install
colcon build
ls -rtl
source install/local_setup.bash
vi run.sh
chmod 755 *.sh
./run.sh
ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888
ros2 run micro_ros_setup create_agent_ws.sh
ros2 run micro_ros_setup build_agent.sh
ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888
# 1. micro-ROS 작업 공간으로 이동
cd ~/microros_ws
# 2. ★ 중요: 새로 빌드 완료된 에이전트 패키지 경로를 현재 터미널 세션에 '다시 로드'합니다.
source install/local_setup.bash
# 3. 이제 다시 와이파이(UDP) 에이전트를 실행합니다!
ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888

# ESP-IDF 5.1 
git clone --recursive -b v5.1.4 https://github.com/espressif/esp-idf.git esp-idf-v5.1
cd esp-idf-v5.1
./install.sh esp32s3
cd
source ~/.bashrc
get_idf5.1
alias
get_idf51

ls -rtl
idf.py build
idf.py flash monitor
idf.py menuconfig
idf.py build
sudo apt update
sudo apt install -y python3-colcon-common-e
```


### 칩셋의 설정
통상적으로 최신 ESP-IDF 를 설치했으나, 기존 것들의 빌드 문제로 5.1 버전도 같이 설치해서 아래와 같이 alias 형태로 구분을 할 수 있도록 했다.

```bash

alias get_idf='. $HOME/esp/esp-idf/export.sh'
alias get_idf51='. $HOME/esp/esp-idf-v5.1/export.sh'

```
ESP-IDF 환경을 시작할 때 터미널을 열고 . $HOME/esp/esp-idf/export.sh 등을 해야 하는데, 이미 alias 가 되어 있으니 간단히 get_idf 나 get_idf51 을 실행시키면 된다.

```bash
go@ubuntu2404:~/esp/projects/atlas_new_esp$ get_idf51
Setting IDF_PATH to '/home/go/esp/esp-idf-v5.1'
Detecting the Python interpreter
Checking "python3" ...
Python 3.12.3
"python3" has been detected
Checking Python compatibility
Checking other ESP-IDF version.
Adding ESP-IDF tools to PATH...
Checking if Python packages are up to date...
Constraint file: /home/go/.espressif/espidf.constraints.v5.1.txt
Requirement files:
 - /home/go/esp/esp-idf-v5.1/tools/requirements/requirements.core.txt
Python being checked: /home/go/.espressif/python_env/idf5.1_py3.12_env/bin/python
Python requirements are satisfied.
Added the following directories to PATH:
  /home/go/esp/esp-idf-v5.1/components/espcoredump
  /home/go/esp/esp-idf-v5.1/components/partition_table
  /home/go/esp/esp-idf-v5.1/components/app_update
  /home/go/.espressif/tools/xtensa-esp-elf-gdb/12.1_20231023/xtensa-esp-elf-gdb/bin
  /home/go/.espressif/tools/xtensa-esp32s3-elf/esp-12.2.0_20230208/xtensa-esp32s3-elf/bin
  /home/go/.espressif/tools/riscv32-esp-elf/esp-12.2.0_20230208/riscv32-esp-elf/bin
  /home/go/.espressif/tools/esp32ulp-elf/2.35_20220830/esp32ulp-elf/bin
  /home/go/.espressif/tools/openocd-esp32/v0.12.0-esp32-20230921/openocd-esp32/bin
  /home/go/.espressif/tools/xtensa-esp-elf-gdb/12.1_20231023/xtensa-esp-elf-gdb/bin
  /home/go/.espressif/tools/xtensa-esp32s3-elf/esp-12.2.0_20230208/xtensa-esp32s3-elf/bin
  /home/go/.espressif/tools/riscv32-esp-elf/esp-12.2.0_20230208/riscv32-esp-elf/bin
  /home/go/.espressif/tools/esp32ulp-elf/2.35_20220830/esp32ulp-elf/bin
  /home/go/.espressif/tools/openocd-esp32/v0.12.0-esp32-20230921/openocd-esp32/bin
  /home/go/.espressif/python_env/idf5.1_py3.12_env/bin
  /home/go/esp/esp-idf-v5.1/tools

Detected installed tools that are not currently used by active ESP-IDF version.
For removing old versions of openocd-esp32, xtensa-esp-elf, esp32ulp-elf, esp-rom-elfs, xtensa-esp-elf-gdb use command 'python /home/go/esp/esp-idf-v5.1/tools/idf_tools.py uninstall'
To free up even more space, remove installation packages of those tools. Use option 'python3 /home/go/esp/esp-idf-v5.1/tools/idf_tools.py uninstall --remove-archives'.

Done! You can now compile ESP-IDF projects.
Go to the project directory and run:

  idf.py build

```

이제 프로젝트 폴더를 만들고, 최상위 프로젝트 디렉토리에서 
```bash
idf.py menuconfig
```
를 치면 다음과 같은 화면이 나온다.

![2026-08-06-140446.png](/assets/images/2026-08-06-140446.png)

대략 설정해야 할 것들은 다음과 같다.
---

| 메뉴 경로 | 설정 항목 | 값 | 비고 |
| :--- | :--- | :--- | :--- |
| **Serial flasher config** | Flash SPI mode | **QIO** | EFUSE 미설정 시 OPI 대신 QIO 사용 |
| | Flash SPI speed | **40MHz** | 안정성을 위해 40MHz 권장 |
| | Flash size | **16MB** | N16 모델 제원 |
| **Component config → ESP PSRAM** | Support for external RAM | **[*] (Enable)** | |
| | SPI RAM config | **Octal Mode PSRAM** | R8 모델 (8MB Octal) |
| | Set RAM clock speed | **40MHz / 80MHz** | 현재 40MHz로 안정화 |
| **Component config → ESP32S3-Specific** | VDD_SDIO Power VCC | **3.3V** | N16R8 필수 전압 |

---

| 메뉴 경로 | 설정 항목 | 값 | 비고 |
| :--- | :--- | :--- | :--- |
| **micro-ROS Settings** | WiFi SSID | **alex_2.4** | Raymond님 공유기 이름 |
| | WiFi Password | **PASSWORD1** | 공유기 비밀번호 |
| | Agent IP Address | **192.168.0.53** | 가상머신(VM)의 IP |
| | Agent Port | **8888** | UDP 기본 포트 |
| **Component config → LWIP** | Local main task stack size | **10240** | 네트워크 스택 안정화 (권장) |

---

| 메뉴 경로 | 설정 항목 | 값 | 비고 |
| :--- | :--- | :--- | :--- |
| **Component config → FreeRTOS** | Number of Cores | **2** | S3의 듀얼코어 활용 |
| | Common ESP-related | **[*] (Enable)** | 멀티코어 최적화 |
| **Component config → Common ESP** | Channel for console output | **UART0** | Baud rate 115200 확인 |

이 외에 블루투스 (BLE) 를 설정하려면 NIMBLE 를 선택하고,
![2026-08-06-140705.png](/assets/images/2026-08-06-140705.png)

컴포넌트를 다운로드해서 배치해야 한다.
![2026-08-06-140734.png](/assets/images/2026-08-06-140734.png)


모두 설정 후 저장을 하면 idf.py menuconfig 를 실행한 위치에 sdkconfig 파일이 생성된다.

### 프로젝트 빌드 후

프로젝트를 적절히 만들어서 빌드, 업로드, 모니터링은 다음과 같다.
프로젝트 빌드는 다음
```bash
idf.py build
```
업로드 및 모니터링은 다음
```bash
idf.py flash monitor
```

## micro-ROS agent 설치 및 실행
ESP32 micro-ROS 만으로는 ROS2 와 어떠한 통신도 이뤄지지 않는다. 이것을 연계하려면 micro-ROS 와 ROS2 를 위한 agent 가 존재해야 한다.  그것이 다음이다.

### 1. ROS 2 워크스페이스 생성 및 이동
```bash
mkdir -p ~/microros_ws/src
cd ~/microros_ws/src
```

### 2. micro-ROS Setup 패키지 클론
```bash
git clone -b humble https://github.com/micro-ROS/micro_ros_setup.git
```
### 3. 의존성 설치 및 워크스페이스 빌드
```bash
cd ~/microros_ws
rosdep update
rosdep install --from-paths src --ignore-src -y
colcon build
source install/setup.bash
```
### 4. Agent 워크스페이스 생성 및 빌드 (시간이 수 분 소요됨)
```bash
ros2 run micro_ros_setup create_agent_workspace.sh
ros2 run micro_ros_setup build_agent.sh
source install/setup.bash
```
### 5. 실행
```bash
source install/local_setup.bash

# UDPv4 에이전트 구동 (기본 포트: 8888)
ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888
```
위의 8888 은 idf.py menuconfig 에서 설정한 포트 번호 이어야만 한다. !! (주의)

이것을 실행하고 ESP32 보드에 전원을 켜면 ESP32 에서 idf.py menuconfig 에서 설정한 서버로 자동으로 접속이 이뤄지면 바로 위의 micro-ROS agent 와 통신이 시작된다.

이후에는 ROS2 기반 툴과 명령들을 사용할 수 있다.




