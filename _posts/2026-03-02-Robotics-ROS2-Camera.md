---
title: ROS2 Camera Publish
img_path: /assets/images/
author: Alex
math: true
date: 2025-09-24
category: [Robotics, ROS2, ROS2 Camera Publish]
tags:
 - ROS2 Camera Publish
layout: post
---

## 준비
### 1. PC 준비 (카메라 서버 준비)
먼저 노트북 등의 카메라에 MediaMTX 와 FFMPEG 을 이용해서 스트리밍을 시작한다.  먼저 도스창 (cmd.exe) 으로 mediaMTX 폴더의 run.bat 를 실행 시키고, 다른 도스창 (cmd.exe) 로 ffmpeg 디렉토리의 run.bat 를 실행시킨다.  ffmpeg 은 노트북 카메라에 따라 오류가 날 수 있는데, 장치관리자에서 카메라 이름을 수정하면 된다.  아래를 다운로드해서 적절한 폴더에 압축을 풀어 사용한다. 

[다운로드]({{ '/assets/data/Camera.zip' | relative_url }})

### 2. Vmware 의 Ubuntu 24.04 LTS (ROS2 jazzy 설치 완료된 것)
Ubuntu 24.04 Desktop 을 설치하고 ROS2 jazzy 를 모두 설치한 상태를 전제로 한다.  특히 Network 은 반드시 bridged 이어야만 한다. 

다음 소스를 다운로드 한 뒤에 적절한 Vmware 의 공유 폴더에 넣고, 이것을 Ubuntu 로 복사 해서 폴더를 만들고 zip 압축을 푼다.  그러면 ros2_ws 디렉토리가 보일 것이다.

[다운로드]({{ '/assets/data/ros2_ws_webrtc.zip' | relative_url }})

![2026-08-04-162227.png](/assets/images/2026-08-04-162227.png)

아래와 같이 빌드를 수행한다.
```bash
cd ros2_ws
colcon build
```

그 다음에 실행은 다음과 같이 한다.  whep_url 은 mediaMTX 가 있는 IP 주소, 즉 카메라가 있는 IP 주소이다.
```bash
source install/local_setup.bash
ros2 run webrtc_receiver receiver_node --ros-args -p whep_url:=http://172.18.34.132:8889/live/whep
```

## 확인
웹 브라우저로 http://127.0.0.1:8889/live/ 를 치면 MediaMTX 서버에 접근하여 카메라가 보여주는 동영상을 볼 수 있어야 한다.  그 이후에 Vmware 의 터미널에서 반드시 ros2_ws 디렉토리에 들어 간 이후에 다음 명령어를 쳐야 한다.  이때 127.0.0.1 로 하면 되지 않는다.  즉, 카메라는 Vmware 가상 머신에 있는 것이 아니고 다른 네트워크 노드, 적어도 자신의 PC 에서 구동되기 때문이다.  물론 원격의 다른 서버에서 구동되어도 상관 없다. 적어도 지금은 127.0.0.1 은 아니다.  

```bash
source install/local_setup.bash
ros2 run webrtc_receiver receiver_node --ros-args -p whep_url:=http://172.18.34.132:8889/live/whep
```

## ROS2 Publish Image 공유
위의 노드 (webrtc_receiver)가 기동하면 주기적으로 카메라 서버로 부터 받은 영상을 ROS2 포맷으로 Publish 하게 되며 이는 같은 Subnet 에서 모두 받아 볼 수 있다.

영상이므로 보통 다음의 ROS2 툴을 써서 볼 수 있다.  소스를 보면 ROS2 의 어떤 Topic 으로 publish 되었는지 알 수 있으며 그것을 콤보에서 선택하면 된다.  (아래 이미지)

```bash
ros2 run rqt_image_view rqt_image_view
```
![2026-08-04-163318.png](/assets/images/2026-08-04-163318.png)

그 외로 다음과 같은 명령으로도 확인이 가능하다.
```bash
ros2 topic list
ros2 topic echo /camera/image_raw
ros2 topic hz /camera/image_raw
```


## 소스 엿보기
ros2_ws_webrtc.zip 는 Python 으로 ROS2 Node 를 만드는 전체적인 디렉토리 구성, 환경 파일들, 소스를 엿볼 수 있다.   이것은 좋은 템플릿이 될 수 있다.

핵심 소스는 다음과 같다.  자세한 설명은 이 소스를 AI 에 주고 설명을 부탁하면 자세히 설명해 줄 것이다.
```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2
import asyncio
import threading
import requests
from aiortc import RTCPeerConnection, RTCSessionDescription, MediaStreamTrack
from av import VideoFrame

class WebRTCReceiver(Node):
    def __init__(self):
        super().__init__('webrtc_receiver')

        # ✅ 파라미터 선언 (whep_url 이 없으면 디폴트 설정)
        self.declare_parameter('whep_url', 'http://172.18.34.132:8889/live/8889/live/whep')
        self.whep_url = self.get_parameter('whep_url').get_parameter_value().string_value
        # Publish 하는 Topic 이름
        self.publisher_ = self.create_publisher(Image, 'camera/image_raw', 10)
        self.bridge = CvBridge()
        
        self.pc = RTCPeerConnection()
        self.loop = asyncio.new_event_loop()
        
        # ✅ 영상 트랙이 수신되었을 때의 콜백
        @self.pc.on("track")
        def on_track(track):
            if track.kind == "video":
                self.get_logger().info("🎥 비디오 트랙 수신 시작!")
                asyncio.run_coroutine_threadsafe(self.process_track(track), self.loop)

        # ✅ 별도 스레드에서 asyncio 루프 실행
        self.thread = threading.Thread(target=self.start_async_loop, daemon=True)
        self.thread.start()

        # ✅ WHEP 핸드셰이크 시작
        asyncio.run_coroutine_threadsafe(self.connect_whep(), self.loop)

    def start_async_loop(self):
        asyncio.set_event_loop(self.loop)
        self.loop.run_forever()

    async def connect_whep(self):
        try:
            # 1. Offer 생성 (받기 전용)
            self.pc.addTransceiver("video", direction="recvonly")
            offer = await self.pc.createOffer()
            await self.pc.setLocalDescription(offer)

            # 2. MediaMTX에 WHEP POST 요청 (curl 방식과 동일)
            headers = {'Content-Type': 'application/sdp'}
            response = requests.post(self.whep_url, data=self.pc.localDescription.sdp, headers=headers, timeout=5)

            if response.status_code == 201 or response.status_code == 200:
                # 3. Answer SDP 수신 및 적용
                answer = RTCSessionDescription(sdp=response.text, type="answer")
                await self.pc.setRemoteDescription(answer)
                self.get_logger().info("✅ WHEP 핸드셰이크 성공!")
            else:
                self.get_logger().error(f"❌ WHEP 접속 실패: {response.status_code} - {response.text}")
        except Exception as e:
            self.get_logger().error(f"❌ 연결 오류: {str(e)}")

    async def process_track(self, track):
        while rclpy.ok():
            try:
                frame = await track.recv()
                # PyAV 프레임을 numpy(BGR)로 변환
                img = frame.to_ndarray(format="bgr24")
                
                # ROS2 메시지 발행
                msg = self.bridge.cv2_to_imgmsg(img, encoding="bgr8")
                msg.header.stamp = self.get_clock().now().to_msg()
                msg.header.frame_id = "camera_link"
                self.publisher_.publish(msg)
            except Exception as e:
                self.get_logger().warn(f"⚠️ 프레임 수신 중 오류: {e}")
                break

    def stop(self):
        self.loop.stop()
        self.pc.close()

def main(args=None):
    rclpy.init(args=args)
    node = WebRTCReceiver()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.stop()
        node.destroy_node()
        rclpy.shutdown()

```
