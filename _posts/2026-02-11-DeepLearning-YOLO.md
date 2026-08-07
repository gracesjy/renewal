---
title: YOLO Object Detection
img_path: /assets/images/
author: Alex
math: true
date: 2026-02-11
category: [Manufacturing, Deep Learning, YOLO Object Detection]
tags:
 - Deep Learning
 - YOLO
layout: post
---

## CNN 등장

Machine Learning 에서는 보통 사람이 Feature 를 생성 하지만, 예를 들어 사람의 얼굴의 경우 다양한 특징 (Feature) 들이 있어서 이러한 Feature 생성이 불가능하다.

그래서 등장한 것이 CNN (Convolutional Neural Network) 가 등장하였고 이는 Deep Learning 의 범주안에 포함되는 것이다.  즉, 이미지의 특징을 추출하는데 최적화 되어 있는 딥러닝 (Deep Learning) 모델이다.

## 주요 구조

간단하게 구조를 보면 다음과 같이 축약하여 설명될 수 있다.

- 합성곱 층(Convolutional Layer): 필터를 사용해 이미지의 선, 모양 같은 작은 특징을 찾는다.
- 풀링 층(Pooling Layer): 이미지의 크기를 줄여서 중요한 특징만 남긴다.
- 완전 연결 층(Fully Connected Layer): 추출된 특징을 모아서 최종적으로 어떤 이미지인지 맞힌다.

## YOLO 란 ?

YOLO(You Only Look Once) 은 CNN(합성곱 신경망) 기술을 기반으로 알고리즘을 개선한 실시간 객체 탐지(Object Detection) 모델이며, 초당 수십~수백 프레임을 분석할 수 있어 자율주행차, CCTV 보안 시스템, 실시간 드론 관제 등에 많이 사용된다.

## YOLO 학습 환경

GPU 가속을 활용할 수 있는 파이썬(Python) 기반 환경이 필수적이며, 개인적으로는 NVIDIA RTX3060(12GB VRAM) 으로 어느 정도 커버가 될 수 있습니다.  

개인 컴퓨터의 GPU 성능이 부족하다면, 웹 브라우저에서 곧바로 무료/유료 GPU를 쓸 수 있는 구글 코랩(Google Colab)을 추천된다.

## YOLO 추론 환경

GPU 가 없는 노트북으로도 간단한 추론은 테스트 할 수 있다.  Python 기반의 YOLO 패키지의 의존성 때문에 Anaconda 환경에서 별도의 가상 환경을 꾸미는 것을 추천한다.

```bash
conda create -n yolo_webrtc -c conda-forge python=3.10 -y
conda activate yolo_webrtc
```

반드시 가상 환경을 activate 한 후 pip 와 python 이 지금 만든 yolo_webrtc 아래의 bin 에 위치하고 있는지 확인 해야 한다.  아래와 같다면 문제가 없다.
```bash

(yolo_webrtc) raymond@ASUS:~/yolo/aquarium$ which pip
/home/raymond/anaconda3/envs/yolo_webrtc/bin/pip
(yolo_webrtc) raymond@ASUS:~/yolo/aquarium$ which python
/home/raymond/anaconda3/envs/yolo_webrtc/bin/python
```

카메라 영상을 방송형 (MediaMTX) 기반으로 받는 것이 범용성을 위해서 좋으며, GPU 가 없기 때문에 YOLO 가 의존하는 PyTorch 는 CPU 기반으로 가볍게 설치하는 것을 추천한다.

또한 영상 처리를 위한 ffmpeg 등도 아래와 같이 conda 에서 설치해야 의존성 문제가 없으니 주의할 것.

```bash
pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu
pip install opencv-python ultralytics
conda install -c conda-forge ffmpeg -y
python -c "import cv2, numpy, ultralytics; print('Install Success ! OpenCV:', cv2.__version__, 'NumPy:', numpy.__version__)"
```


## YOLO Object Detection
다음은 화면이 있는 YOLO 를 이용한 객체 탐지 예제이다.  먼저 노트북에서 MediaMTX 와 FFMPEG 을 실행 시켜서 방송을 시작해야 한다.  또는 스마트폰의 방송형 앱을 이용해서 방송을 시작해야 한다.

### YOLO Model Download

아래 소스에서 yolov8n.pt 는 실행되는 하위에 없다면 실시간으로 다운을 받는다.  이는 우리가 학습한 것이 아니라 기본적인 학습 모델이다.   만약 스스로 학습한 객체라면 best.pt 나 별도의 이름의 파일을 로딩하면 된다.

```python
model = YOLO("yolov8n.pt")
```

### YOLO Simple Example
가장 간단한 YOLO 예제이다.  버스(Bus) 와 사람이 포함된 bus.jpg 이미지에서 YOLO 를 이용해서 Object Detection 을 한 뒤에 result.jpg 로 저장하는 예제이다.  이때 result.jpg 는 bus.jpg 와 달리 인식된 객체에 Box 와 확률이 그려져 있다.

```python
from ultralytics import YOLO

# 1. 사전 학습된 경량 모델 로드 (최초 실행 시 자동 다운로드)
model = YOLO("yolov8n.pt")

# 2. 이미지 추론 실행 (device='cpu' 명시)
# 온라인 샘플 이미지 URL 또는 로컬 파일 경로 지정 가능
results = model("https://ultralytics.com/images/bus.jpg", device="cpu")

# 3. 추론 결과 확인 및 저장
for result in results:
    result.show()  # 바운딩 박스가 그려진 이미지 결과창 표시
    result.save(filename="result.jpg")  # 로컬 파일로 저장

print("추론 완료! result.jpg 파일이 저장되었습니다.")
```

### WebRTC
영상으로 실시간으로 객체를 탐지하려면 NVIDIA GPU 같은 것이 있는 것이 좋다. GPU 가 없는 경우에는 Latency 를 각오해야 한다.  특히 실시간으로 BOX 로 인식된 것을 보여주는 경우 매우 Latency 가 심하니 주의해야 한다.

MediaMTX 에서 http://172.18.34.132:8888/live 로 카메라 영상이 뜨는지 웹브라우저 (크롬, 에지)에서 먼저 확인해 보아야 한다.  IP 주소는 방화벽 문제가 없어야 한다. 자신의 로컬 주소라면 큰 문제는 없다.


http://172.18.34.132:8888/live/index.m3u8" 에서 index.m3u8 은 HLS(HTTP Live Streaming) 프로토콜에서 동영상 조각들의 정보가 담긴 '인덱스(목록) 파일'이며 보통 웹사이트에서 주소 뒤에 index.html을 붙이는 것과 유사한 원리이다.

```python
import cv2
from ultralytics import YOLO

model = YOLO("yolov8n.pt")

# MediaMTX 기본 HLS HTTP 주소 (http://<IP>:8888/<path>/index.m3u8)

hls_url = "http://172.18.34.132:8888/live/index.m3u8"

cap = cv2.VideoCapture(hls_url)

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        print("프레임을 가져올 수 없습니다.")
        break

    # CPU 추론
    results = model.predict(frame, device="cpu", imgsz=320, verbose=False)
    
    # 결과 시각화
    annotated_frame = results[0].plot()
    cv2.imshow("YOLO WebRTC-HLS Stream", annotated_frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```

### RTSP 를 이용한 방법

RTSP는 HLS처럼 파일 목록(index.m3u8)을 파싱하는 과정이 없으므로, 지연 시간(Latency)이 거의 없고(0.1~0.3초 내외) OpenCV에서 매우 직관적인 URL 형식으로 읽어올 수 있다.

아래는 화면은 없고, 단순히 객체 감지 (Object Detection) 한 라벨 (Text) 만 보여준다.

```python
import os
import cv2
from ultralytics import YOLO

# 1. RTSP 연결 안정성을 위한 TCP 옵션 설정 (패킷 손실 방지)
os.environ["OPENCV_FFMPEG_CAPTURE_OPTIONS"] = "rtsp_transport;tcp"

# 2. YOLO CPU 모델 로드
model = YOLO("yolov8n.pt")

# 3. MediaMTX RTSP 주소 설정 (포트 8554, 경로 live)
rtsp_url = "rtsp://172.18.34.132:8554/live"

cap = cv2.VideoCapture(rtsp_url)

if not cap.isOpened():
    print(f"[오류] RTSP 스트림 연결 실패: {rtsp_url}")
    print("MediaMTX 서버 및 송출 경로('live')를 확인하세요.")
    exit(1)

print("=" * 60)
print(f"MediaMTX RTSP 스트림 연결 성공! ({rtsp_url})")
print("감지 객체 리스트 실시간 콘솔 출력을 시작합니다. (종료: Ctrl + C)")
print("=" * 60)

frame_count = 0

try:
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            print("[경고] 프레임을 가져올 수 없습니다. 스트림 연결을 재확인하세요.")
            break

        frame_count += 1

        # CPU 성능 부담을 줄이기 위해 3프레임마다 1번씩만 추론
        if frame_count % 3 != 0:
            continue

        # YOLO CPU 추론 실행
        results = model.predict(frame, device="cpu", imgsz=320, verbose=False)
        boxes = results[0].boxes

        if len(boxes) > 0:
            # 감지된 객체의 이름과 신뢰도(Confidence) 추출
            detected_items = []
            for box in boxes:
                cls_id = int(box.cls[0])
                cls_name = model.names[cls_id]       # 클래스 이름 (예: person, car 등)
                confidence = float(box.conf[0])     # 신뢰도 (예: 0.85)
                detected_items.append(f"{cls_name}({confidence:.2f})")

            # 리스트 형태로 출력
            items_str = ", ".join(detected_items)
            print(f"[{frame_count:05d} 프레임] 감지 ({len(boxes)}개): [ {items_str} ]")
        else:
            print(f"[{frame_count:05d} 프레임] 감지된 객체 없음")

except KeyboardInterrupt:
    print("\n[알림] 사용자에 의해 프로그램을 종료합니다.")

finally:
    cap.release()
    print("RTSP 리소스가 정상적으로 해제되었습니다.")

```

결과는 다음과 같이 나온다.

```bash
[00483 프레임] 감지 (2개): [ person(0.80), person(0.65) ]
[00486 프레임] 감지 (2개): [ person(0.80), person(0.65) ]
[00489 프레임] 감지 (2개): [ person(0.80), person(0.65) ]
[00492 프레임] 감지 (2개): [ person(0.80), person(0.64) ]
[00495 프레임] 감지 (2개): [ person(0.80), person(0.65) ]
```

## YOLO 학습

개인적으로 YOLO 학습을 위한 데이터 셋을 만들려면 AI 에게 물어 보면서 하면 되지만 예전에는 LabelImg / CVAT 라는 툴을 사용해서 일일히 박스와 라벨을 준비해야 했다.

RoboFlow 라는 웹 기반도 있지만 ... 속 편하게 LabelImg 툴을 사용해서 했던 것 같다.

RoboFlow 에서는 YOLO 를 위한 데이터 셋을 제공하기도 한다.  아래 예제는 RoboFlow 에서 다운받은 데이터 셋으로 학습한 것을 보여준다.

### Aquarium 데이터 학습

```python
from ultralytics import YOLO
import os
model = YOLO("yolov8n.pt")
model.train(data=r'G:\YOLO\Aquarium\Aquarium Combined.v2-raw-1024.yolov8\data.yaml', epochs=100,patience=30,batch=128,imgsz=416)
```

#### model.train 파라미터

간단히 정리하면 

> **"지정된 수족관(Aquarium) 이미지 데이터셋을 활용하여, 이미지를 416 크기로 줄이고 128장씩 묶어서 최대 100번 반복 학습시키되, 30번 동안 성능 향상이 없으면 중간에 종료해라."** 라는 뜻입니다.

자세히 정리하면 

- data=r'G:\YOLO\Aquarium\Aquarium Combined.v2-raw-1024.yolov8\data.yaml'의미: 
학습할 데이터의 정보가 담긴 설정 파일(data.yaml)의 경로입니다.역할: 이 파일 안에는 이미지들이 어디에 저장되어 있는지(Train/Val 경로), 그리고 맞혀야 할 물체의 종류(클래스 이름: 예: 물고기, 상어, 해파리 등)가 무엇인지 적혀 있습니다. 앞의 r은 윈도우 경로의 백슬래시(\)를 문자로 그대로 인식하게 만드는 파이썬 문법입니다.
- epochs=100의미: 전체 데이터셋을 총 100번 반복해서 학습하겠다는 설정입니다.특징: 100번을 다 돌기 전에 아래 설정한 patience 조건에 걸리면 조기 종료될 수 있습니다.
- patience=30의미: 조기 종료(Early Stopping)를 위한 대기 횟수입니다.역할: 학습 도중 검증 데이터(Validation) 기준 모델 성능이 30번의 에포크(Epoch) 동안 연속으로 좋아지지 않으면, 시간 낭비를 막고 과적합(Overfitting)을 방지하기 위해 100번을 다 채우지 않고 학습을 자동으로 중단합니다.
- batch=128의미: 모델이 한 번에 메모리에 올려서 처리하는 이미지의 묶음 크기입니다.주의점: 128은 꽤 큰 수치입니다. 이 설정을 구동하려면 컴퓨터의 그래픽카드 메모리(VRAM)가 매우 넉넉해야 합니다(최소 16GB~24GB 이상 권장). 만약 학습을 시작했을 때 Out of Memory (OOM) 에러가 난다면 이 값을 64, 32, 16 등으로 줄여야 합니다.
- imgsz=416의미: 입력 이미지의 해상도를 416x416 픽셀 크기로 리사이즈(축소)하여 학습하라는 설정입니다.특징: 원래 데이터셋 이름에 1024가 적혀 있는 것으로 보아 원본은 큰 이미지 같지만, 학습 속도를 높이고 메모리를 아끼기 위해 416으로 낮추어 학습하는 설정을 주셨습니다. 크기가 작아지면 학습은 빨라지지만, 너무 작은 물체는 잘 못 찾을 수도 있습니다.

NVIDIA GPU 기반에서는 비교적 빠르게 학습이 진행 된다.

![2026-08-07-111545.png](/assets/images/2026-08-07-111545.png)


그리고 이때 Task Manager 로 본 성능은 다음과 같다.
![2026-08-07-111724.png](/assets/images/2026-08-07-111724.png)

#### 모델(Model) 저장

```python
import torch
import pickle

model.save(r'G:\YOLO\aquarium_model_best.pt')
```
#### 학습후 생기는 폴더(runs\train000)


##### Confusion Matrix

> 이 행렬은 **"모델이 물체를 얼마나 정확하게 분류했는가, 그리고 어떤 물체와 가장 헷갈려하는가?"** 를 한눈에 보여주는 성적표.

![2026-08-07-112435.png](/assets/images/2026-08-07-112435.png)

###### 📊 이 이미지(Confusion Matrix)를 보는 방법

* **축의 의미**
  * **True (가로축)**: 실제 정답 물체의 종류입니다.
  * **Predicted (세로축)**: AI 모델이 예측한 물체의 종류입니다.
* **대각선 (왼쪽 위 ➡️ 오른쪽 아래)**
  * 이 대각선에 위치한 숫자들이 **모델이 똑바로 맞힌 개수**입니다. 
  * 예를 들어, 실제 물고기(`True: fish`)를 모델이 물고기(`Predicted: fish`)라고 정확히 맞힌 건수가 **329건**으로 가장 성능이 좋습니다.



##### F1-Confidence Curve
> 이 행렬은 **"AI가 얼마나 확신(Confidence)을 가지고 물체를 찾을 때, 전체적인 검출 정확도(F1-score)가 가장 높은가?"** 를 한눈에 보여주는 지표.

![2026-08-07-112737.png](/assets/images/2026-08-07-112737.png)

###### 📊 그래프 핵심 정보 해석

* **축의 의미**
  * **Confidence (가로축)**: AI가 "이것은 물고기다!"라고 확신하는 확률 기준값(0~1)입니다.
  * **F1 (세로축)**: 정밀도(Precision)와 재현율(Recall)을 종합한 **종합 예측 점수(0~1)**입니다. 1에 가까울수록 완벽한 모델입니다.
* **파란색 굵은 선 (`all classes`)**
  * 수족관 안의 모든 생물 클래스를 통합한 **전체 평균 점수**입니다.
  * **`all classes 0.72 at 0.302`**: AI의 확신 점수(Confidence) 커트라인을 **0.302**로 설정했을 때, 이 모델의 종합 성적인 F1-score가 **0.72(72%)**로 가장 높다는 뜻입니다.

##### Label 지표 그래프

![2026-08-07-113217.png](/assets/images/2026-08-07-113217.png)

###### 📊 Labels 그래프 핵심 정보 해석

* **좌측 상단 (Classes 분포 막대그래프)**
  * **의미**: 각 생물별 데이터(바운딩 박스)의 개수입니다.
  * **해석**: `fish`가 약 2,000개로 압도적으로 많고, `starfish`나 `stingray`는 상대적으로 개수가 매우 적습니다. 데이터 불균형(Class Imbalance)이 심해 모델이 물고기는 잘 맞히지만 적은 클래스는 놓치기 쉬운 환경임을 보여줍니다.
* **우측 상단 (Bounding Box 중첩도)**
  * **의미**: 정답 박스들을 한곳에 겹쳐놓은 모양입니다.
  * **해석**: 박스들이 중심부에 밀집해 있고 가로가 긴 직사각형 형태가 많음을 뜻합니다. 사진 중앙부에 물체들이 주로 배치되어 있음을 알 수 있습니다.
* **좌측 하단 (x, y 산점도)**
  * **의미**: 이미지 내에서 물체 중심점의 좌표(0~1 사이 비율) 분포입니다.
  * **해석**: 점들이 전체 화면에 고르게 퍼져 있습니다. 수족관 안에서 생물들이 특정 구석에 몰려있지 않고 골고루 등장한다는 뜻입니다.
* **우측 하단 (width, height 산점도)**
  * **의미**: 물체 박스의 가로(width)와 세로(height) 크기 분포입니다.
  * **해석**: 왼쪽 아래(0.0~0.2 구간)에 진한 파란색이 몰려 있습니다. 이는 데이터셋에 크기가 매우 작은 소형 물체(작은 물고기 등)가 대다수를 차지하고 있음을 의미합니다.

##### 결과들

###### P Curve (Precision-Confidence Curve)
![2026-08-07-113550.png](/assets/images/2026-08-07-113550.png)

###### PR Curve
![2026-08-07-113606.png](/assets/images/2026-08-07-113606.png)

###### Recall Confidence Curve
![2026-08-07-113644.png](/assets/images/2026-08-07-113644.png)

###### Result 
![2026-08-07-113401.png](/assets/images/2026-08-07-113401.png)

####### 📊 Results 그래프 핵심 정보 해석

* **왼쪽 6개 그래프 (Loss 지표 - 낮을수록 우수)**
  * **box_loss (박스 오차)**: 물체의 위치를 사각형으로 얼마나 정확하게 잡았는지 나타냅니다. 학습(`train`)과 검증(`val`) 모두 우상향 없이 전반적으로 안정적으로 하향 곡선을 그리고 있습니다.
  * **cls_loss (클래스 분류 오차)**: 물체의 종류(물고기, 해파리 등)를 얼마나 정확하게 맞혔는지 나타냅니다. `train`과 `val` 모두 0에 가깝게 매끄럽게 떨어지고 있어 분류 학습이 아주 잘 되었음을 의미합니다.
  * **dfl_loss (위치 경계 분포 오차)**: 물체 박스의 경계선을 미세 조정하는 오차입니다. 검증(`val`) 단계에서 다소 진동이 있지만, 전체적으로 낮아지는 추세입니다.

* **오른쪽 4개 그래프 (Metrics 지표 - 높을수록 우수)**
  * **metrics/precision(B) (정밀도)**: 모델이 물체라고 예측한 것 중 실제 정답인 비율입니다. 초반에 급격히 상승한 후 0.8(80%) 선에서 안정적으로 유지됩니다.
  * **metrics/recall(B) (재현율)**: 실제 물체들을 모델이 놓치지 않고 찾아낸 비율입니다. 약 10 에포크 부근에서 일시적으로 떨어졌다가 이후 100 에포크까지 0.65(65%) 수준으로 꾸준히 상승했습니다.
  * **mAP50(B) / mAP50-95(B) (종합 정확도)**: 객체 탐지 모델의 최종 점수입니다. 두 지표 모두 우상향을 그리며 끝까지 상승하고 있으며, 특히 mAP50은 0.72 수준까지 도달했습니다.

**💡 학습 진단 및 총평**

* **과적합 없는 안정적인 학습**: 검증 손실값(`val_loss`)들이 중반 이후 다시 치솟는 현상이 없고, 정확도 지표들(`mAP`)이 끝까지 완만하게 상승하는 것으로 보아 과적합(Overfitting) 없이 성공적으로 학습이 완료되었습니다.
* **추가 개선 가능성**: 100 에포크 시점에서도 mAP 그래프가 완전히 평평해지지 않고 미세하게 상승 곡선을 유지하고 있습니다.
* **향후 권장 사항**: 다음 학습 시에는 에포크를 `150~200` 정도로 조금 더 늘려주고, 앞서 분석한 대로 이미지 크기를 `imgsz=640`으로 키워준다면 소형 클래스의 검출율까지 함께 높일 수 있을 것으로 기대됩니다.


- Train Batch
![2026-08-07-113502.png](/assets/images/2026-08-07-113502.png)
- Validation Batch
![2026-08-07-113516.png](/assets/images/2026-08-07-113516.png)

#### 추론 (테스트)

이미 RoboFlow 에서 다운받은 Aquarium 데이터는 train, valid, test 데이터 셋으로 나눠져 있기 때문에 테스트를 수행할 수 있다.

```python
result = model.predict(source=r'G:\YOLO\Aquarium\Aquarium Combined.v2-raw-1024.yolov8\test\images', save=True)
```
![2026-08-07-112004.png](/assets/images/2026-08-07-112004.png)