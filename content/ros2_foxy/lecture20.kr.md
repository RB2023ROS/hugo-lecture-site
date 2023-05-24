---
title: "Lecture20. NVIDIA Jetson Platform"
date: 2023-04-28T10:40:29+09:00
draft: false
---

## 데모 시연

> 이번 시간에는 지금까지 학습한 내용들을 다시 한 번 되짚어보면서 ROS 2를 사용하고자 하는 고객들 측면에서의 개발, 기능들을 이야기해보겠습니다.

- Gazebo 시뮬레이션 데모
- Real World와 시뮬레이션 연동
- 실물 센서의 ROS 2 연동
- 소형 로봇 데모를 통해 살펴보는 로봇 시스템

## Gazebo 시뮬레이션 데모 - Fusionbot SLAM

지금까지 학습한 내용들을 통해 구현할 수 있는 Application을 데모해보고자 합니다.

- Gazebo World 생성
- URDF를 통한 로봇 Structure 구성
- Gazebo Control Plugin과 Sensor Plugin

처음 마주친 환경에서, 우리는 현재 자신의 위치와 주변 환경의 Mapping을 동시에 진행합니다. 이와 같이 SLAM이란, Localization과 Mapping을 Simultaneously 실행하는 것으로 로봇의 자율주행에 사용되는 지도 생성에 주로 사용됩니다.

![Untitled.png](/kr/ros2_foxy/images20/Untitled.png?height=300px)

2D Lidar를 통해 Gazebo 상의 Fusionbot을 통해 환경의 지도를 생성해보는 실습을 진행해보겠습니다.

{{% notice tip %}}
SLAM은 사용하는 센서, 방식에 따라 여러 종류의 구현이 가능합니다. 반드시 2D Lidar SLAM만 존재하는 것이 아님을 밝힙니다.
{{% /notice %}}

- Fusionbot SLAM

```python
# Terminal 1 - gazebo world launch
ros2 launch fusionbot_description office_construction.launch.py

# Terminal 2 - slam toolbox launch
cbp fusionbot_slam && source install/local_setup.bash
ros2 launch fusionbot_slam online_async_launch.py

# Terminal 2 - control node
rqt_robot_steering
```

![Untitled1.png](/kr/ros2_foxy/images20/Untitled1.png?height=300px)

⇒ rqt_robot_steering을 통해 로봇을 움직임에 따라 rviz2 상에서 완성되는 지도를 확인할 수 있습니다.

- 생성된 지도를 저장하고 싶다면 rviz2에서 경로를 입력한 뒤, Save Map 버튼을 클릭합니다.

![Untitled2.png](/kr/ros2_foxy/images20/Untitled2.png?height=300px)

⇒ 이렇게 생성된 지도는 이후 자율주행, world 생성, 3D reconstruction등 다양한 Application에 사용됩니다.

## Real World와 시뮬레이션 연동

> 이번 데모는 SLAMTEC의 Rplidar를 사용하여 사무실을 Mapping하고, 이를 바탕으로 Gazebo World를 만들어보겠습니다.

- rplidar a3

![Untitled3.png](/kr/ros2_foxy/images20/Untitled3.png?height=400px)

2D 라이다 제품군 중 비교적 저렴한 가격에 최대 25m라는 합리적인 탐지거리와 해상도를 갖고 있어 로봇 프로젝트에서 널리 사용되는 제품 중 하나입니다.

- 실제 실외 배송로봇에 이러한 형태의 라이다가 사용된 바 있습니다.

![Untitled4.png](/kr/ros2_foxy/images20/Untitled4.png?height=300px)

- rplidar a3를 통해 실제 사무실을 SLAM 해보겠습니다. (하단 로그는 제가 데모하기 위한 명령어입니다.)

```python
# rplidar a3
cd /dev
sudo chmod 777 ttyUSB0
ros2 launch rplidar_ros view_rplidar_a3_launch.py

# Terminal 2
ros2 launch demo_pkg rf2o_odom.launch.py
# Terminal 3
ros2 launch demo_pkg online_async_launch_real.py
```

- 실시간으로 그려지는 지도를 볼 수 있습니다. 지도 작성을 마쳤다면, 이제 Gazebo에서 이 지도를 활용해 Building을 만들어 보겠습니다.

![Untitled5.png](/kr/ros2_foxy/images20/Untitled5.png?height=300px)

SLAM map은 pgm 포맷을 가지므로 png로 변환이 필요합니다. https://convertio.co/pgm-png/

- 이제 Gazebo의 building Editor를 사용하여 사무실과 동일한 크기의 시뮬레이션 환경을 제작합니다.

![Untitled6.png](/kr/ros2_foxy/images20/Untitled6.png?height=300px)

⇒ 지금까지 학습한 내용을 바탕으로 이제 여러분만의 로봇, Object를 추가하여 풍성한 시뮬레이션을 만들 수 있을 것입니다.

## 실물 센서의 ROS 2 연동

> 아무리 시뮬레이션에서 완벽하게 동작하더라도 실제 센서를 사용하는 것과는 큰 차이를 갖습니다. 이번 데모에서는 다양한 센서들을 ROS 2 연동하여 실행해보고, 성능을 비교해보겠습니다. (하단 로그는 제가 데모하기 위한 명령어입니다.)

- Orbbec Astra+

```bash
cd ~/gemini_ws/
source install/local_setup.bash
ros2 launch orbbec_camera astra_pro_plus.launch.xml

rviz2 & rqt

top
```

- Orbbec Gemini2

```bash
ros2 launch orbbec_camera gemini2.launch.xml
```

- Intel Realsense2 D455

```bash
cd ~/realsense_ws/
source install/local_setup.bash

ros2 launch realsense2_camera rs_launch.py pointcloud.enable:=true
```

## 소형 로봇 데모를 통해 살펴보는 로봇 시스템

- nanosaur 소개

Nanosaur는 탱크 타입의 로봇으로 NVIDIA의 개발자 **Raffaello Bonghi**가 배포한 오픈소스에 기반하여 Road Balance가 교육용 키트로 재설계한 로봇입니다. 자세한 스펙과 기능은 링크를 통해 확인할 수 있습니다. - [link](https://nanosaur.ai/)

![Untitled7.png](/kr/ros2_foxy/images20/Untitled7.png?height=300px)

- nanosaur 제어

```python
ROS_DOMAIN_ID=30 ros2 run nanosaur_hardware nanosaur
rqt_robot_steering
```

- nanosaur 카메라

```python
ROS_DOMAIN_ID=30 ros2 run nanosaur_camera nanosaur_camera
ros2 run rqt_image_view rqt_image_view
```

- 이번 데모를 통해 ROS 2의 장점을 엿볼 수 있는데요. ROS 2는 DDS라는 표준 프로토콜을 사용하기 때문에 동일한 네트워크를 사용하는 시스템 내부에서 동작하는 시스템들끼리는 자동으로 상호 인지가 가능합니다.

![Untitled8.png](/kr/ros2_foxy/images20/Untitled8.png?height=300px)

- 이러한 장점을 가진 ROS 2는 로봇 뿐만 아니라 자율주행 분야로 확대되어 현재 다양한 도메인에서 수요가 많은 상황입니다.

![Untitled9.png](/kr/ros2_foxy/images20/Untitled9.png?height=300px)

- 비롯 지금은 매우 작은 Nanosaur를 살펴보았지만 실제 로봇을 만들기까지 로봇 공학자들은 다양한 과정을 거치고 수정, 검증을 반복합니다.

![Untitled10.png](/kr/ros2_foxy/images20/Untitled10.png?height=300px)

설계와 검증까지 구현하기도 벅찬 와중, 센서 연동까지 개발하는 것은 매우 힘들기에 대부분 ROS 개발자들은 검증된 코드와 예시를 제공하는 제품들을 사용합니다.

- [Intel Realsense](https://dev.intelrealsense.com/docs/ros-wrapper)
- [ZED Stereo Camera](https://www.stereolabs.com/docs/ros2/)
- [Oak-D AI Camera](https://docs.luxonis.com/en/latest/)
- [Livox 3D Lidar](https://github.com/Livox-SDK/livox_ros_driver)

단순 ROS 2 패키지가 존재하는 것 뿐만 아니라, 카메라와 같이 메모리와 처리 속도에 민감한 센서는 효율적인 코드를 제공하는지 여부가 무척 중요합니다.

![Untitled11.png](/kr/ros2_foxy/images20/Untitled11.png?height=300px)

NVIDIA에서 제공하는 ROS 2 카메라 코드 페이지를 살펴보면 예시와 함께 사용된 환경, 해상도 별 성능을 제시하고 있습니다.

![Untitled12.png](/kr/ros2_foxy/images20/Untitled12.png?height=300px)

> 따라서 센서를 개발하거나, 로봇을 개발하는 입장 모두 사용자 호환성을 고려하여 설계하고, 수많은 시행착오를 통해 올바른 제품에 대한 척도를 깨우쳐야 합니다.
