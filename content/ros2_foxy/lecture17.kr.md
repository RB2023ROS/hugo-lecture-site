---
title: "Lecture17. Real Hardware Examples"
date: 2023-04-28T10:51:09+09:00
draft: false
---

## Intel realsense2 camera ROS 2

real hardware 강의의 두번째 시간으로 Intel의 realsense2 카메라 패키지를 실행하고, 분석해 보겠습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9781b781-5675-481a-a51c-30a24ef3e9d3/Untitled.png)

- 예제 실행을 위해 필요한 종속성들을 설치하고 realsense sdk인 librealsense를 설치합니다.

```bash
sudo apt-get update && sudo apt-get upgrade

# Register the server's public key
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
> gpg: Total number processed: 1

# Add the server to the list of repositories
sudo add-apt-repository "deb https://librealsense.intel.com/Debian/apt-repo $(lsb_release -cs) main" -u

# Install the libraries (see section below if upgrading packages)
sudo apt-get install librealsense2-dkms -y
sudo apt-get install librealsense2-utils -y
```

- 실제 카메라 연결을 통해 realsense2 패키지 설치를 검증해봅시다.

```bash
# Reconnect the Intel RealSense depth camera and run
realsense-viewer

modinfo uvcvideo | grep "version:"
# should include realsense string
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0d2475b8-f2bb-41f5-8acc-cfe3258fb669/Untitled.png)

⇒ 위와 같은 이미지 출력을 얻었다면 성공입니다.

- 사실, realsense camera는 intel이 개발한 만큼 ubuntu 배포판 패키지로 ros2를 지원하고 있습니다. 사용만 하고 싶다면 아래와 같이 손쉽게 설치가 가능합니다.

```bash
# Install Dependencies
sudo apt-get install ros-foxy-cv-bridge ros-foxy-librealsense2 ros-foxy-message-filters ros-foxy-image-transport -y
sudo apt install ros-foxy-diagnostic-updater -y
sudo apt-get install -y libssl-dev libusb-1.0-0-dev pkg-config libgtk-3-dev
sudo apt-get install -y libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev
```

⇒ 하지만 우리는 패키지 분석이 목적이므로 직접 소스코드 빌드와 분석을 해보겠습니다. 아래 링크를 통해 release version을 다운받고 ros2 workspace에 압축을 해제합니다.

[https://github.com/IntelRealSense/realsense-ros/releases](https://github.com/IntelRealSense/realsense-ros/releases)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/822510e1-52ce-441d-9f94-911c6e0f77c4/Untitled.png)

- 폴더 이동 후 빌드는 아래 순서대로 해주세요!

```bash
cbp realsense2_camera_msgs && source install/local_setup.bash
cbp realsense2_description && source install/local_setup.bash
cbp realsense2_camera && source install/local_setup.bash
```

- 우선 빌드가 잘 되었는지 예시부터 실행해봅시다.

```bash
ros2 launch realsense2_camera rs_launch.py
```

- topic list, rviz2를 통해 정상 동작을 확인합니다.

```bash
$ ros2 topic list
/camera/color/camera_info
/camera/color/image_raw
/camera/color/metadata
/camera/depth/camera_info
/camera/depth/image_rect_raw
/camera/depth/metadata
/camera/extrinsics/depth_to_color
/camera/extrinsics/depth_to_depth
/camera/imu
/parameter_events
/rosout
/tf_static
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/75d32614-48c0-4c0b-bd78-bb68604e7424/Untitled.png)

- point cloud topic을 활성화하기 위해선 아래와 같은 launch가 필요합니다.

```bash
ros2 launch realsense2_camera rs_launch.py pointcloud.enable:=true
```

- rviz2를 통해 pointcloud를 시각화해봅시다. (메가커피에서 강의를 제작해서 손흥민 선수 얼굴이 보이네요 ㅎㅎ)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b46035cd-cb20-452c-b844-15905d1952ab/Untitled.png)

- rviz2에서 tf2 data도 확인할 수 있습니다. fixed frame은 camera_link로 두시면 됩니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/56877e49-f609-449a-af5e-0568f260b7e4/Untitled.png)

- tf2 tree도 확인해봅시다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7edd658a-c845-42cd-abbd-1aa8a8d7bd82/Untitled.png)

## Composition launch

realsense ros2 package는 Intra-communication을 사용하여 메모리 최적화를 적용한 composition launch도 지원하고 있습니다. 이를 사용하기 위해서는 패키지 빌드를 다시 해야 하며, 이러한 이유로 저는 realsense_ws라는 별도의 workspace를 만들었습니다.

```bash
cd ~/
mkdir -p realsense_ws/src
cd realsense_ws

# locate packages & build again
colcon build --cmake-args '-DBUILD_TOOLS=ON'
```

- Composition 예시를 실행해봅시다!

```bash
$ ros2 launch realsense2_camera rs_intra_process_demo_launch.py
...
[component_container-1] [INFO] [1682508794.621382739] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0597926 [sec]
[component_container-1] [INFO] [1682508794.689360200] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0610729 [sec]
[component_container-1] [INFO] [1682508794.755558725] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0605604 [sec]
[component_container-1] [INFO] [1682508794.822757399] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0610629 [sec]
[component_container-1] [INFO] [1682508794.889473916] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0610784 [sec]
[component_container-1] [INFO] [1682508794.955772739] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0606767 [sec]
[component_container-1] [INFO] [1682508795.022029595] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0602304 [sec]
[component_container-1] [INFO] [1682508795.088818104] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0603308 [sec]
[component_container-1] [INFO] [1682508795.156870084] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0616579 [sec]
[component_container-1] [INFO] [1682508795.223072425] [frame_latency]:
```

- rqt_image_view를 통해 이미지도 확인해 보겠습니다.

```bash
$ ros2 run rqt_image_view rqt_image_view
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4cf0f610-5316-4928-931d-c7ab5cac1d12/Untitled.png)

- 실제 일반 node 실행과 composition 사용 시 점유하는 메모리를 비교해 보았습니다.

```bash
PID   USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 2530 kimsooy+  20   0  897252  82312  50300 R  12.3   0.5   1:09.80 x-terminal-emul
 1247 kimsooy+  20   0 6284836 344712 109988 S  11.3   2.1   7:17.14 gnome-shell
41892 kimsooy+  20   0 1183960  85788  57140 S   5.3   0.5   0:03.28 component_conta
```

```bash
PID   USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 1247 kimsooy+  20   0 6333092 344816 109988 S  11.6   2.1   8:00.07 gnome-shell
 2530 kimsooy+  20   0  897252  82664  50300 S  10.9   0.5   1:33.14 x-terminal-emul
42240 kimsooy+  20   0 1457552  92104  57096 S   8.9   0.6   0:03.88 realsense2_came
```

<aside>
💡 노트북 환경이어 겉보기에 큰 차이는 없지만, 실제 edge device에서 실행 + 다중 Node들이 결합되는 경우 Composition 사용 여부가 큰 차이를 가질 것입니다.

</aside>

[Launch file 분석 - Publish 분석 (1)](https://www.notion.so/Launch-file-Publish-1-745759fd131546a9a4ec0955605edce7)

## realsense2 Description

description이라는 이름을 가진 패키지는 CAD, URDF 파일과 같은 물성치, 외관에 대한 데이터를 담고 있습니다. realsense의 description package를 분석해봅시다.

- 기본 launch file을 실행해 보겠으며, d455 옵션으로 실행해보겠습니다.

```cpp
ros2 launch realsense2_description view_model.launch.py model:=<sth>
ros2 launch realsense2_description view_model.launch.py model:=test_d455_camera.urdf.xacro
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/92e8988d-f8d2-4c04-a537-3180dea74664/Untitled.png)

⇒ 센서 외관을 비롯하여 다양한 내장 센서들에 대한 tf2도 확인할 수 있습니다.

- tf2 tree를 통해 이에 대한 자세한 구조를 파악할 수 있으며, d455는 카메라 뿐만 아니라 가속도 센서, 자이로 센서를 포함하고 있어 아래와 같이 복잡한 구조를 갖게 됩니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/32523da2-2872-44d5-8428-87d2183ed979/Untitled.png)

⇒ 지금 tf2 tree에서는 base_link가 최상위 Node인데요. 실제 로봇 시스템에 장착 시 로봇의 base_link가 이미 존재한다면 의도치 않은 tf2 error가 발생할 수 있습니다.

- 실제 test_d455_camera.urdf.xacro 파일에서도 이를 손쉽게 제거할 수 있도록 아래와 같이 base_link를 별도 분리해둔 모습을 볼 수 있습니다.

```xml
<?xml version="1.0"?>
<robot name="realsense2_camera" xmlns:xacro="http://ros.org/wiki/xacro">
  <xacro:arg name="use_nominal_extrinsics" default="false"/>
  <xacro:include filename="$(find realsense2_description)/urdf/_d455.urdf.xacro" />

  <link name="base_link" />
  <xacro:sensor_d455 parent="base_link" use_nominal_extrinsics="$(arg use_nominal_extrinsics)">
    <origin xyz="0 0 0" rpy="0 0 0"/>
  </xacro:sensor_d455>
</robot>
```

## Description Launch file 분석

- view_model.launch.py - 실제 실행되는 Node는 robot_state_publisher와 rviz2 입니다. (센서는 움직이는 파츠가 없으니 joint_state_publisher는 굳이 필요 없겠지요?)

```python
rviz_node = Node(
    package='rviz2',
    executable='rviz2',
    node_name='rviz2',
    output = 'screen',
    arguments=['-d', rviz_config_dir],
    parameters=[{'use_sim_time': False}]
    )
model_node = Node(
    node_name='model_node',
    package='robot_state_publisher',
    executable='robot_state_publisher',
    namespace='',
    output='screen',
    arguments = [urdf]
    )
return launch.LaunchDescription([rviz_node, model_node])
```

⇒ robot_state_publisher의 argument로 urdf 파일들이 전달되며, 사용자로부터 mode:= 값을 받아 해당 urdf file이 실행되는 형식입니다. (저라면 Launch Argument를 사용했을 것 같습니다.)

- urdf 폴더를 살펴보면, 모든 모델에 대한 기본 urdf file과, description launch를 위해 base_link가 결합된 test urdf들이 자동생성되어있는 모습을 확인할 수 있습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aa167fb0-896d-43e7-81fb-7b27f43d08bc/Untitled.png)

- 마지막으로 카메라 센서의 tf2 사용 시 주의할 점이 있는데요. 일반적으로 컴퓨터 비전 도메인에서 사용하는 좌표계와, ROS 2에서 사용하는 좌표계는 아래와 같이 차이를 갖습니다. gazebo plugin 시간에 살펴보았던 것처럼 rviz2를 통해 검증을 한 뒤 사용하기를 추천드립니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c5ee9fbd-3b5c-4708-8720-142ce11e9145/Untitled.png)

## Custom Message

- 대부분 센서 패키지들은 sdk에서 정의된 자료구조와 ROS 2의 연동을 위해 custom interface를 구현합니다. realsense2에서도 IMUInfo를 비롯한 Custom Interface들을 제공하고 있습니다.

```python
$ ros2 interface show realsense2_camera_msgs/msg/IMUInfo
# header.frame_id is either set to "imu_accel" or "imu_gyro"
# to distinguish between "accel" and "gyro" info.
std_msgs/Header header
float64[12] data
float64[3] noise_variances
float64[3] bias_variances
```

⇒ 이렇게 custom interface가 있다면, 다른 패키지의 종속성이 될 확률이 높이 때문에 먼저 빌드해줘야 합니다.

⇒ 더불어 custom interface는 해당 workspace에만 적용되기 때문에 workspace가 바뀌면 적용되지 않습니다.

- CMakeLists.txt를 통해 빌드되는 모든 IDL들을 확인할 수 있습니다.

```cpp
set(msg_files
  "msg/IMUInfo.msg"
  "msg/Extrinsics.msg"
  "msg/Metadata.msg"
)
rosidl_generate_interfaces(${PROJECT_NAME}
  ${msg_files}
  "srv/DeviceInfo.srv"
  DEPENDENCIES builtin_interfaces std_msgs
  ADD_LINTER_TESTS
)
```

- 일전 이야기한 것과 같이 다른 소스코드에서 custom interface를 사용하고 있어 선행 빌드 후 소싱이 꼭 필요합니다.

```cpp
#pragma once

#include <librealsense2/rs.hpp>
#include <librealsense2/rsutil.h>
#include "constants.h"
#include <cv_bridge/cv_bridge.h>

#include <diagnostic_updater/diagnostic_updater.hpp>
#include <diagnostic_updater/publisher.hpp>
#include "realsense2_camera_msgs/msg/imu_info.hpp"
#include "realsense2_camera_msgs/msg/extrinsics.hpp"
#include "realsense2_camera_msgs/msg/metadata.hpp"
#include "realsense2_camera_msgs/srv/device_info.hpp"
```

## velodyne Lidar ROS 2 실행

이번에는 point cloud 데이터를 센싱할 수 있는 3D Lidar 중 가장 보편적으로 사용되는 VLP-16의 ROS 2 패키지를 분석해보겠습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1b47994d-5c1a-4eb7-a182-acbff616c97a/Untitled.png)

- velodyne lidar 또한 ubuntu package를 제공하고 있으며 자체적인 Gazebo package와 driver 패키지를 모두 제공합니다.

```cpp
$ sudo apt install ros-foxy-velodyne-
ros-foxy-velodyne-description            ros-foxy-velodyne-laserscan-dbgsym
ros-foxy-velodyne-driver                 ros-foxy-velodyne-msgs
ros-foxy-velodyne-driver-dbgsym          ros-foxy-velodyne-msgs-dbgsym
ros-foxy-velodyne-gazebo-plugins         ros-foxy-velodyne-pointcloud
ros-foxy-velodyne-gazebo-plugins-dbgsym  ros-foxy-velodyne-pointcloud-dbgsym
ros-foxy-velodyne-laserscan              ros-foxy-velodyne-simulator
```

- 이번에도 우리는 학습을 위해 소스코드 빌드를 진행하겠습니다.

```bash
cd ~/ros2_ws/src

# package clone
git clone -b foxy-devel https://github.com/ros-drivers/velodyne.git

# dependencies install
sudo apt-get update && sudo apt-get install libpcap-dev

# source code build
cbp velodyne_msgs && source install/local_setup.bash
cbp velodyne_laserscan && source install/local_setup.bash
cbp velodyne_pointcloud && source install/local_setup.bash
cbp velodyne_driver && source install/local_setup.bash
cbp velodyne && source install/local_setup.bash
```

- velodyne lidar는 Ethernet 인터페이스를 제공합니다. 따라서 Ubuntu의 네트워크 설정에서 고정 IP 설정을 해주며, VLP-16의 기본 IP는 **192.168.1.100**입니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/634f1e8d-0f87-40b6-97e1-161785338d9d/Untitled.png)

- 설정 이후 launch는 다음과 같습니다.

```cpp
ros2 launch velodyne velodyne-all-nodes-VLP16-launch.py
```

- launch rviz2를 통한 시각화를 해봅시다. fixed frame을 velodyne으로 설정해야 합니다.

![velo1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d24c7d19-2831-4a68-a259-9e3020198aad/velo1.png)

- tf2 tree를 살펴보면 아무것도 조회되지 않습니다. velodyne tf2만 broadcast되고 있기 때문에 확인할 수 없는 것입니다. (저였다면 stasic transform broadcaster를 통해 world > velodyne tf2를 하나 생성했을 것 같습니다.)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/09ccef10-82ec-4995-9cbb-7b49d995e8a6/Untitled.png)

- velodyne lidar 또한 composed-launch라는 comspoition 기반 실행을 지원합니다.

```bash
$ ros2 launch velodyne velodyne-all-nodes-VLP16-composed-launch.py
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/velodyne_driver_node' in container '/velodyne_container'
[component_container-1] [INFO] [1682566114.537928654] [velodyne_container]: Load Library: /home/kimsooyoung/ros2_ws/install/velodyne_pointcloud/lib/libconvert.so
[component_container-1] [INFO] [1682566114.540579179] [velodyne_container]: Found class: rclcpp_components::NodeFactoryTemplate<velodyne_pointcloud::Convert>
[component_container-1] [INFO] [1682566114.540601962] [velodyne_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<velodyne_pointcloud::Convert>
[component_container-1] [INFO] [1682566114.542606795] [velodyne_convert_node]: correction angles: /home/kimsooyoung/ros2_ws/install/velodyne_pointcloud/share/velodyne_pointcloud/params/VLP16db.yaml
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/velodyne_convert_node' in container '/velodyne_container'
[component_container-1] [INFO] [1682566114.548422161] [velodyne_container]: Load Library: /home/kimsooyoung/ros2_ws/install/velodyne_laserscan/lib/libvelodyne_laserscan.so
[component_container-1] [INFO] [1682566114.549147973] [velodyne_container]: Found class: rclcpp_components::NodeFactoryTemplate<velodyne_laserscan::VelodyneLaserScan>
[component_container-1] [INFO] [1682566114.549171818] [velodyne_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<velodyne_laserscan::VelodyneLaserScan>
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/velodyne_laserscan_node' in container '/velodyne_container'
[component_container-1] [INFO] [1682566114.597621115] [velodyne_laserscan_node]: Latched ring count of 16
[component_container-1] [WARN] [1682566114.597755427] [velodyne_laserscan_node]: PointCloud2 fields in unexpected order. Using slower generic method.
```

- composed-launch와 일반 launch 시 차이점을 rqt_graph로 확인해봅시다.

![velo_rqt.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8b6d6a84-0c0a-4209-ac68-2453360f3c77/velo_rqt.png)

![veloc_rqt.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/53459cf8-605e-438a-951f-1a47dc8a3ca8/veloc_rqt.png)

## Launch file 분석

- velodyne-all-nodes-VLP16-launch.py를 분석해봅시다. velodyne_driver_node, velodyne_convert_node, velodyne_laserscan_node이 실행되며 velodyne_driver_node의 종료 시 launch 자체가 종료되도록 Event를 걸어주었습니다.

```python
return launch.LaunchDescription([
    velodyne_driver_node,
    velodyne_convert_node,
    velodyne_laserscan_node,

    launch.actions.RegisterEventHandler(
        event_handler=launch.event_handlers.OnProcessExit(
            target_action=velodyne_driver_node,
            on_exit=[launch.actions.EmitEvent(
                event=launch.events.Shutdown())],
        )),
    ])
```

- velodyne-all-nodes-VLP16-composed-launch.py에서는 component_container와 velodyne_driver::VelodyneDriver, velodyne_pointcloud::Convert, velodyne_laserscan::VelodyneLaserScan 3개의 composition이 동작합니다.

```python
	container = ComposableNodeContainer(
	  name='velodyne_container',
	  namespace='',
	  package='rclcpp_components',
	  executable='component_container',
	  composable_node_descriptions=[
	    ComposableNode(
	      package='velodyne_driver',
	      plugin='velodyne_driver::VelodyneDriver',
	      name='velodyne_driver_node',
	      parameters=[driver_params]),
	    ComposableNode(
	      package='velodyne_pointcloud',
	      plugin='velodyne_pointcloud::Convert',
	      name='velodyne_convert_node',
	      parameters=[convert_params]),
	    ComposableNode(
	      package='velodyne_laserscan',
	      plugin='velodyne_laserscan::VelodyneLaserScan',
	      name='velodyne_laserscan_node',
	      parameters=[laserscan_params]),
	  ],
	  output='both',
	)

	return LaunchDescription([container])
```

⇒ 이제 방금 전의 실행 코드들(velodyne_driver_node, velodyne_convert_node, velodyne_laserscan_node)을 분석해 봅시다.

## velodyne_driver_node

- velodyne package의 CMakeLists.txt에서 velodyne_driver_node의 코드를 확인할 수 있습니다.

```python
add_executable(velodyne_driver_node src/driver/velodyne_node.cpp)
ament_target_dependencies(velodyne_driver_node rclcpp)
target_link_libraries(velodyne_driver_node velodyne_driver)
install(TARGETS velodyne_driver_node DESTINATION lib/${PROJECT_NAME})
```

- velodyne_node.cpp 자체는 일반적인 rclcpp node programming이 구현되어 있습니다. (대신 NodeOptions을 매개변수로 전달하는데 이는 VelodyneDriver가 Composition형태로 구현되어 있기 때문입니다.)

```cpp
#include <rclcpp/rclcpp.hpp>
#include <memory>
#include "velodyne_driver/driver.hpp"

int main(int argc, char ** argv)
{
  // Force flush of the stdout buffer.
  setvbuf(stdout, nullptr, _IONBF, BUFSIZ);

  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<velodyne_driver::VelodyneDriver>(rclcpp::NodeOptions()));
  rclcpp::shutdown();

  return 0;
}
```

- VelodyneDriver::VelodyneDriver에서는 velodyne sdk => ROS 2로의 형태 변환이 이루어집니다. Composition을 고려한 형태로 Node 프로그래밍 되어있습니다.

```cpp
namespace velodyne_driver
{
class VelodyneDriver final : public rclcpp::Node
{
public:
  explicit VelodyneDriver(const rclcpp::NodeOptions & options);
  ~VelodyneDriver() override;
  ...

private:
  bool poll();
  void pollThread();

  // configuration parameters
  struct config_;
  ...
  std::unique_ptr<Input> input_;
  rclcpp::Publisher<velodyne_msgs::msg::VelodyneScan>::SharedPtr output_;
  int last_azimuth_;
  ...
};
}  // namespace velodyne_driver

#endif  // VELODYNE_DRIVER__DRIVER_HPP_
```

- velodyne sdk => ROS 2로의 데이터 형태 변환은 std::thread 통해 구현되어 있습니다. ROS 2와의 충돌을 막기 위해 소멸자에서 join하는 방식을 사용하였습니다. (사실 이렇게 하면 이벤트 발생 시 다시 deadlock 문제가 발생합니다. 저라면 일전의 callback group & executor를 사용했을 것 같습니다.)

```cpp
std::thread poll_thread_;

// node 생성자에서 thread 시작
VelodyneDriver::VelodyneDriver(const rclcpp::NodeOptions & options)
: rclcpp::Node("velodyne_driver_node", options),
  diagnostics_(this, 0.2)
{
  poll_thread_ = std::thread(&VelodyneDriver::pollThread, this);
}

// thread 함수에서 무한 poll 호출
void VelodyneDriver::pollThread()
{
  std::future_status status;

  do {
    poll();
    status = future_.wait_for(std::chrono::seconds(0));
  } while (status == std::future_status::timeout);
}

// poll 함수가 마치 ROS 2 callback 처럼 동작
bool VelodyneDriver::poll()
```

- velodyne_driver => ROS 2로의 데이터 변환을 정리해보았습니다. (poll 함수의 내용입니다.)

1. velodyne*driver 데이터를 std::unique_ptr<Input> input*로 파싱
2. input*에서 config*으로 데이터 변환
3. config\_에서 std::unique_ptr<velodyne_msgs::msg::VelodyneScan> scan으로 데이터 변환
4. output\_->publish(std::move(scan));

⇒ 상당히 메모리 형변환이 잦은데 이전 코드에서의 레거시가 있는 것 같습니다.

- diagnostic_updater ⇒ 중간중간 보이는 diagnostic 관련 코드들은 ROS 2의 diagnostic_updater API로 device drivers의 여러 상태를 topic 형태로 publish할 수 있는 기능입니다.

```cpp
diagnostic_updater::Updater diagnostics_;
std::unique_ptr<diagnostic_updater::TopicDiagnostic> diag_topic_;
```

## Visualization

Outside of this repository, there is `[rqt_robot_monitor](https://index.ros.org/p/rqt_robot_monitor/)` to visualize diagnostic messages that have been aggregated by the `diagnostic_aggregator`.

Diagnostics messages that are not aggregated can be visualized by `[rqt_runtime_monitor](https://index.ros.org/p/rqt_runtime_monitor/)`.

## velodyne_convert_node

rviz2에서 보았던 것처럼 제대로 된 Pointcloud topic을 위해서 velodyne_driver에서의 raw data들을 누적하고, 다시 sensor_msgs/msg/PointCloud2로 변환하는 작업이 필요합니다. velodyne_convert_node에서 이 내용이 구현되어 있습니다.

![velo1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d24c7d19-2831-4a68-a259-9e3020198aad/velo1.png)

- velodyne_pointcloud package의 CMakeLists.txt에서 velodyne_convert_node의 코드를 확인할 수 있습니다.

```python
add_executable(velodyne_convert_node src/conversions/convert_node.cpp)
ament_target_dependencies(velodyne_convert_node rclcpp)
target_link_libraries(velodyne_convert_node convert)
install(TARGETS velodyne_convert_node
  DESTINATION lib/${PROJECT_NAME}
)
```

- convert_node.cpp 자체는 단순 Node 생성 후 spin을 담고 있습니다.

```cpp
int main(int argc, char ** argv)
{
  // Force flush of the stdout buffer.
  setvbuf(stdout, nullptr, _IONBF, BUFSIZ);

  rclcpp::init(argc, argv);

  // handle callbacks until shut down
  rclcpp::spin(
    std::make_shared<velodyne_pointcloud::Convert>(
      rclcpp::NodeOptions()));

  rclcpp::shutdown();

  return 0;
}
```

- Convert 클래스는 velodyne_msgs::msg::VelodyneScan topic을 subscribe하여 누적하고, sensor_msgs::msg::PointCloud2 topic으로 다시 publish합니다.

```cpp
namespace velodyne_pointcloud
{
class Convert final
  : public rclcpp::Node
{
public:
  explicit Convert(const rclcpp::NodeOptions & options);
...

private:
  void processScan(const velodyne_msgs::msg::VelodyneScan::SharedPtr scanMsg);

  std::unique_ptr<velodyne_rawdata::RawData> data_;
  rclcpp::Subscription<velodyne_msgs::msg::VelodyneScan>::SharedPtr velodyne_scan_;
  rclcpp::Publisher<sensor_msgs::msg::PointCloud2>::SharedPtr output_;
  tf2_ros::Buffer tf_buffer_;
  std::unique_ptr<velodyne_rawdata::DataContainerBase> container_ptr_;
...

};
}  // namespace velodyne_pointcloud
```

- velodyne_packets의 subscribe callback으로 processScan이 바인딩되어있으며, processScan에서는 일전 언급한 데이터 누적과 publish가 이루어집니다.

```cpp
// subscribe to VelodyneScan packets
velodyne_scan_ =
  this->create_subscription<velodyne_msgs::msg::VelodyneScan>(
  "velodyne_packets", rclcpp::QoS(10),
  std::bind(&Convert::processScan, this, std::placeholders::_1));

void Convert::processScan(const velodyne_msgs::msg::VelodyneScan::SharedPtr scanMsg)
{
  ...
  // allocate a point cloud with same time and frame ID as raw data
  container_ptr_->setup(scanMsg);

  // process each packet provided by the driver
  for (size_t i = 0; i < scanMsg->packets.size(); ++i) {
    data_->unpack(scanMsg->packets[i], *container_ptr_);
  }

  // publish the accumulated cloud message
  diag_topic_->tick(scanMsg->header.stamp);
  output_->publish(container_ptr_->finishCloud());
}
```

- velodyne_convert_node에서 tf2 데이터를 다루는 부분이 구현되어 있습니다. 하지만 편의를 위해 별도의 컨테이너를 만들고 구현은 추상화해둔 모습을 확인할 수 있습니다.

```cpp
tf2_ros::Buffer tf_buffer_;

if (organize_cloud) {
  container_ptr_ = std::make_unique<OrganizedCloudXYZIR>(
    min_range, max_range, target_frame, fixed_frame,
    data_->numLasers(), data_->scansPerPacket(), tf_buffer_);
} else {
  container_ptr_ = std::make_unique<PointcloudXYZIR>(
    min_range, max_range, target_frame, fixed_frame,
    data_->scansPerPacket(), tf_buffer_);
}
```

- DataContainerBase 클래스에서는 tf2 lookupTransform을 통해 scan data의 좌표를 미리 계산해둡니다.

```cpp
std::unique_ptr<velodyne_rawdata::DataContainerBase> container_ptr_;

class DataContainerBase
{
public:
  ...
	void computeTransformation(const rclcpp::Time & time)
	  {
	    geometry_msgs::msg::TransformStamped transform;
	    try {
	      const std::chrono::nanoseconds dur(time.nanoseconds());
	      std::chrono::time_point<std::chrono::system_clock, std::chrono::nanoseconds> time(dur);
	      transform = tf_buffer_.lookupTransform(config_.target_frame, cloud.header.frame_id, time);
	    } catch (tf2::LookupException & e) {
	      return;
	    } catch (tf2::ExtrapolationException & e) {
	      return;
	    }

	    tf2::Quaternion quaternion(
	      transform.transform.rotation.x,
	      transform.transform.rotation.y,
	      transform.transform.rotation.z,
	      transform.transform.rotation.w);
	    Eigen::Quaternionf rotation(quaternion.w(), quaternion.x(), quaternion.y(), quaternion.z());

	    Eigen::Vector3f eigen_origin;
	    tf2::Vector3 origin(
	      transform.transform.translation.x,
	      transform.transform.translation.y,
	      transform.transform.translation.z);
	    vectorTfToEigen(origin, eigen_origin);
	    Eigen::Translation3f translation(eigen_origin);
	    transformation = translation * rotation;
	  }
```

## velodyne_laserscan_node

velodyne_laserscan_node는 sensor_msgs::msg::PointCloud2 data를 sensor_msgs::msg::LaserScan로 변환 후 /scan topic으로 publish 하는 node입니다. 3D pointcloud만 사용한다면 일반적으로 필요 없는 기능입니다.

- 클래스 header file을 통해 간단히 기능만 리뷰해봅시다.

```cpp
namespace velodyne_laserscan
{

class VelodyneLaserScan final
  : public rclcpp::Node
{
public:
  explicit VelodyneLaserScan(const rclcpp::NodeOptions & options);
  ...
private:
  void recvCallback(const sensor_msgs::msg::PointCloud2::SharedPtr msg);

  rclcpp::Subscription<sensor_msgs::msg::PointCloud2>::SharedPtr sub_;
  rclcpp::Publisher<sensor_msgs::msg::LaserScan>::SharedPtr pub_;

  uint16_t ring_count_{0};
  int ring_;
  double resolution_;
};

}  // namespace velodyne_laserscan
```

⇒ PointCloud2 topic을 subscribe 받고

⇒ 이 데이터를 LaserScan으로 변환합니다. (subscribe callback)

⇒ 마지막으로 변환된 데이터를 scan topic으로 다시 publish합니다.

- 코드를 분석하면서 보시다시피 모든 Node 구현은 Composition을 고려하여 만들어졌습니다. 따라서 CMakeLists.txt를 보면 아래와 같이 rclcpp_components_register_nodes 키워드들을 통해 같은 코드로 Composition을 생성할 수 있습니다.

```cpp
target_link_libraries(velodyne_driver velodyne_input)

# install runtime and library files
install(TARGETS velodyne_driver
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

rclcpp_components_register_nodes(velodyne_driver "velodyne_driver::VelodyneDriver")
```

## velodyne_msgs

velodyne_msgs 패키지에는 VelodynePacket, VelodyneScan라는 두 종류의 custom message를 정의하고 있습니다. 이는 주로 velodyne 자체의 프로토콜 패킷을 다루기 위한 것으로 ROS 2 데이터 변환 시 중간 데이터 구조로 사용됩니다.

```bash
> VelodynePacket.msg
# Raw Velodyne LIDAR packet.

builtin_interfaces/Time stamp              # packet timestamp
uint8[1206] data        # packet contents

> VelodyneScan.msg
# Velodyne LIDAR scan packets.

std_msgs/Header header         # standard ROS message header
VelodynePacket[] packets        # vector of raw packets
```

## orbbec astra camera ROS 2 패키지 분석

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/31429758-cabe-4ae0-84e8-8f290a640673/Untitled.png)

이번 시간에는 Orbbec Astra+ Development Kit의 ROS 2 실행과 분석을 진행해보겠습니다. 실제 사용될 카메라와 코드이므로 좀 더 자세하게 분석해보려 합니다. 이전 센서 패키지들과 비교하면서 따라와 주세요!

## 개발환경 설정

- 필요한 apt package들과 ROS 2 package들을 설치합니다.

```bash
sudo apt install libgflags-dev nlohmann-json3-dev
sudo apt install ros-foxy-image-transport ros-foxy-image-publisher
```

- 빌드 종속성인 glog을 설치합니다.

```bash
wget -c https://github.com/google/glog/archive/refs/tags/v0.6.0.tar.gz  -O glog-0.6.0.tar.gz
tar -xzvf glog-0.6.0.tar.gz
cd glog-0.6.0
mkdir build && cd build
cmake .. && make -j4
sudo make install
sudo ldconfig  # Refreshing the link library
```

- 다음으로, 빌드 종속성인 magic_enum을 설치합니다.

```bash
wget -c https://github.com/Neargye/magic_enum/archive/refs/tags/v0.8.0.tar.gz -O  magic_enum-0.8.0.tar.gz

tar -xzvf magic_enum-0.8.0.tar.gz
cd magic_enum-0.8.0
mkdir build && cd build
cmake .. && make -j4
sudo make install
sudo ldconfig # Refreshing the link library
```

이번 실습을 위해 사용된 코드는 2022-07-15 버전 Orbbec SDK Beta for ROS입니다. ( 이후 업데이트 시 코드 변경이 있을 수 있어 명확히 명시하겠습니다. ⇒ [download link](https://orbbec3d.com/index/download.html))

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/77ee1c93-3d46-4153-bdbf-abf3d83e0609/Untitled.png)

- 다운로드 받은 source code를 원하는 workspace에 위치시킨 뒤, 개발문서에서 요구하는 usb rule 추가 작업을 진행합니다.

```bash
cd orbbec_camera/scripts
sudo bash install.sh
sudo udevadm control --reload-rules && sudo udevadm trigger
```

- 해당 작업 후 혹시 모를 오류를 방지하기 위해 재부팅을 권장드리며, 카메라를 연결한 뒤 장치 인식이 되는지 확인해봅시다.

```bash
$ ls /dev | grep Astra
Astra+
Astra+_rgb
```

- 소스코드를 빌드하며, 공식 문서에서는 Release build를 제시하고 있어 그대로 진행해 보겠습니다.

```bash
cd ~/ros2_ws/src
colcon build --event-handlers  console_direct+  --cmake-args  -DCMAKE_BUILD_TYPE=Release
```

<aside>
💡

</aside>

빌드 중 오류가 발생했다면 패키지를 하나하나씩 빌드하며 원인을 분석해봅니다.

```bash
colcon build --packages-select orbbec_camera_msgs --event-handlers  console_direct+  --cmake-args  -DCMAKE_BUILD_TYPE=Release
source install/local_setup.bash
colcon build --packages-select orbbec_camera --event-handlers  console_direct+  --cmake-args  -DCMAKE_BUILD_TYPE=Release
source install/local_setup.bash
```

## **Getting started**

- 빌드가 완료되었다면 기본 예시를 사용해봅시다.

<aside>
💡

</aside>

제공되는 초기 코드 parameter가 아닌, default parameter를 사용하겠습니다. launch file에서 아래와 같이 parameter를 설정을 제거하고 실행합시다.

```python
from launch import LaunchDescription
from launch_ros.actions import Node
from ament_index_python import get_package_share_directory

def generate_launch_description():
    ob_params_file = (
        get_package_share_directory("orbbec_camera") + "/params/astra_plus_params.yaml"
    )
    return LaunchDescription(
        [
            Node(
                package="orbbec_camera",
                namespace="camera",
                name="camera",
                executable="orbbec_camera_node",
                output="screen",
                # parameters=[ob_params_file],
            ),
        ]
    )
```

- astra_plus.launch 예제를 실행해봅시다.

```python
cd <my_ws>
source install/local_setup.bash
ros2 launch orbbec_camera astra_plus.launch.py
...
[orbbec_camera_node-1] [WARN] [1683197732.405664220] [camera.camera]: Using default profile instead.
[orbbec_camera_node-1] [WARN] [1683197732.405680938] [camera.camera]: default FPS 15
[orbbec_camera_node-1] [INFO] [1683197732.413525012] [camera.camera]:  stream color is enabled - width: 2048, height: 1536, fps: 30, Format: OB_FORMAT_MJPG
[orbbec_camera_node-1] [WARN] [1683197732.427355527] [camera.camera]: Publishing dynamic camera transforms (/tf) at 10 Hz
```

<aside>
💡

</aside>

custom interface들을 사용하기 때문에 workspace sourcing이 반드시 필요합니다.

- publish되고 있는 topic들은 다음과 같습니다.

```bash
$ ros2 topic list
/camera/color/camera_info
/camera/color/image_raw
/camera/depth/camera_info
/camera/depth/color/points
/camera/depth/image_raw
/camera/depth/points
/camera/extrinsic/depth_to_color
/camera/ir/camera_info
/camera/ir/image_raw
/clicked_point
/goal_pose
/initialpose
/parameter_events
/rosout
/tf
/tf_static
```

- 특이한 점으로, 카메라 사이 좌표계 값을 depth_to_color라는 이름의 topic으로 publish하고 있습니다.

```python
$ ros2 topic echo --qos-durability=transient_local /camera/extrinsic/depth_to_color  --qos-profile=services_default
header:
  stamp:
    sec: 0
    nanosec: 0
  frame_id: depth_to_color_extrinsics
rotation:
- 0.9999811053276062
- -0.006122155115008354
- -0.0005165112670511007
- 0.006124507170170546
- 0.9999702572822571
- 0.004682439845055342
- 0.0004878292966168374
- -0.004685514606535435
- 0.9999889135360718
translation:
- -25.19551658630371
- -0.30450138449668884
- 1.0939441919326782
---
```

- rviz2를 통해 데이터를 시각화해봅시다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/977294ae-caa4-478e-9f9f-0f8f6b195d95/Untitled.png)

<aside>
💡

</aside>

주의해야 할 점으로, pointcloud2 topic의 DDS QoS를 아래와 같이 잘 맞춰주어야 합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5ad8252-d09e-4c2a-b1fd-9458652ac328/Untitled.png)

<aside>
💡

</aside>

더불어, color pointcloud2 topic의 색상을 확인하기 위해서는 rviz2 옵션을 아래와 같이 맞춰주어야 합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f0d2804c-5bbf-46a4-b550-53bd779326eb/Untitled.png)

<aside>
💡

</aside>

혹은, 제가 제공드리는 rviz파일을 사용하여 configuration 하셔도 좋습니다. ⇒ [astra rviz download](https://drive.google.com/file/d/1heFUJDr7Q2N9jWhP_jYCV9EG_KTUjiQq/view?usp=sharing)

- 카메라 센서인 만큼 tf2를 다룰 시 조심해야 할 부분이 있습니다. tf2 tree는 다음과 같습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c6097385-c226-4d9d-9046-fd535b8ddff4/Untitled.png)

- rviz2를 통해 확인해보면, optical frame들은 회전된 camera 좌표 체계를 갖는 것을 확인 가능합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea5ce636-0c9c-4895-8047-3398593a6685/Untitled.png)

- 마지막으로, astra node의 parameter들을 조회해봅시다.

```bash
$ ros2 param list
/camera/camera:
  camera_link_frame_id
  color_format
  color_fps
  color_frame_id
  color_height
  color_optical_frame_id
  color_width
  d2c_mode
  ...

$ ros2 param get /camera/camera color_format
String value is: YUYV
```

해당 parameter들에 따라 코드 실행에 어떤 변화가 있는지 분석을 통해 살펴보겠습니다.

## 코드 분석

- launch file을 통해 확인했던 orbbec_camera_node가 어떻게 생성되었는지 추적해봅시다.

```python
Node(
    package="orbbec_camera",
    namespace="camera",
    name="camera",
    executable="orbbec_camera_node",
    output="screen",
    # parameters=[ob_params_file],
),
```

- CMakeLists.txt를 살펴보면 rclcpp_components_register_node를 통해 생성되는 orbbec_camera_node를 확인할 수 있습니다. (component와 executable을 모두 빌드하기 위함입니다.)

```bash
ament_target_dependencies(${PROJECT_NAME}
  ${dependencies}
  )
rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "orbbec_camera::OBCameraNodeFactory"
  EXECUTABLE orbbec_camera_node
)
```

- orbbec_camera_node을 구성하는 코드들은 다음과 같습니다.

```bash
add_library(${PROJECT_NAME} SHARED
  src/dynamic_params.cpp
  src/ob_camera_node_factory.cpp
  src/ob_camera_node.cpp
  src/ros_param_backend.cpp
  src/ros_service.cpp
  src/utils.cpp
)
```

- File별 기능을 분석하면 다음과 같습니다.

| Code                   | Description                                                                                                                                                                               |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ros_param_backend      | parameter 변경 시 특정 callback이 실행되도록 binding 해주는 decorator입니다.                                                                                                              |
| dynamic_params         | ROS 2에서 제공되는 기본적인 set_parameter와 유사한 역할을 수행하는 각종 setParameter API들을 구현한 부분입니다. (아마도 별도의 parameter 변경 reject 로직을 구현하고 싶었던 것 같습니다.) |
| ob_camera_node         | 일반적인 ROS 2 Node의 로직과, orbbec sdk의 pipeline을 융합하여 구현한 코드입니다. 실질적인 topic publish, service server들이 구현되어 있습니다.                                           |
| ros_service            | service server들에 대한 cpp 구현을 별도로 분리한 코드입니다.                                                                                                                              |
| ob_camera_node_factory | ob_camera_node에서 구현한 OBCameraNode node를 포인터로 갖는 별도의 Component를 구현하였습니다. (이 부분은 ROS 2 구현 측면에서 다소 미숙한 부분이라고 볼 수 있습니다.)                     |

주요 구현은 ob_camera_node에 작성되어 있습니다. 해당 파일 내 메소드들을 호출 계층 구조에 따라 정리하면 아래와 같습니다.

- ob_camera_node

  - Constructor
    - setupTopics
      - getParameters
      - setupDevices
      - setupProfiles
      - setupDefaultStreamCalibData
        - findDefaultCameraParam
        - updateStreamCalibData
      - setupCameraCtrlServices
      - setupPublishers
      - publishStaticTransforms
    - startPipeline
      - ob::Pipeline 실행
      - frameSetCallback
        - publishColorFrame
        - publishDepthFrame
        - publishIRFrame
        - publishPointCloud
          - publishColorPointCloud
          - publishDepthPointCloud

- setupTopics에서 호출되는 함수들은 다음과 같은 역할을 갖습니다.

| Function                    | Description                                                                                                                                                                                                                                                   |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| getParameters               | dynamic_params에 구현된 setParam를 사용하여 각종 매개변수들을 파싱하고, 세팅합니다.                                                                                                                                                                           |
| setupDevices                | orbbec sdk를 사용하여 camera profile, stream data등에 대한 정보를 조회하고 저장합니다.                                                                                                                                                                        |
| setupProfiles               | getParameters와 setupDevices에서 얻은 정보를 바탕으로 orbbec 카메라를 셋업합니다. d2c_mode등 orbbec만의 고유 설정들을 셋업하는 코드가 구현되어 있습니다.                                                                                                      |
| setupDefaultStreamCalibData | 내부적으로 findDefaultCameraParam / updateStreamCalibData라는 메소드를 호출하며, 카메라 calibration data를 받아 셋업하는 부분입니다.                                                                                                                          |
| setupCameraCtrlServices     | topic이 아닌 service server들에 대한 선언과 callback mapping들이 이루어집니다.                                                                                                                                                                                |
| setupPublishers             | depth/color/points, depth/points, camera_info, extrinsic/depth_to_color, image_raw 등의 topic publisher들이 선언되고 세팅됩니다.                                                                                                                              |
| publishStaticTransforms     | dynamic*tf_broadcaster*와 static*tf_broadcaster*라는 두개의 tf2 broadcast가 구현되어 있습니다. dynamic*tf_broadcaster*의 경우 publishDynamicTransforms라는 메소드를 별도 thread로 실행합니다. (하지만 실제 카메라 내부에서 동적으로 변화하는 tf2는 없습니다.) |

- 대부분 설정과 관련된 구현이지만, ROS 2 구현 측면에서 setupPublishers를 살펴보겠습니다.

QoS profile을 적용한 topic publisher들과 tf broadcaster가 생성되고 있습니다.

```cpp
void OBCameraNode::setupPublishers() {
  static_tf_broadcaster_ = std::make_shared<tf2_ros::StaticTransformBroadcaster>(node_);
  using PointCloud2 = sensor_msgs::msg::PointCloud2;
  using CameraInfo = sensor_msgs::msg::CameraInfo;
  point_cloud_publisher_ = node_->create_publisher<PointCloud2>(
      "depth/color/points", rclcpp::QoS{1}.best_effort().keep_last(1));
  depth_point_cloud_publisher_ = node_->create_publisher<PointCloud2>(
      "depth/points", rclcpp::QoS{1}.best_effort().keep_last(1));
  for (const auto& stream_index : IMAGE_STREAMS) {
    std::string name = stream_name_[stream_index.first];
    std::string topic = name + "/image_raw";
    image_publishers_[stream_index] = image_transport::create_publisher(node_, topic);
    topic = name + "/camera_info";
    camera_info_publishers_[stream_index] =
        node_->create_publisher<CameraInfo>(topic, rclcpp::QoS{1}.best_effort());
  }
  extrinsics_publisher_ = node_->create_publisher<orbbec_camera_msgs::msg::Extrinsics>(
      "extrinsic/depth_to_color", rclcpp::QoS{1}.transient_local());
}
```

- publishStaticTransforms 메소드는 다음과 같습니다. dynamic transform이라는 구현이 보이는데, 사실 카메라 내부적으로는 움직이는 부분이 없습니다.

```cpp
void OBCameraNode::publishStaticTransforms() {
  calcAndPublishStaticTransform();
  if (tf_publish_rate_ > 0) {
    tf_thread_ = std::make_shared<std::thread>([this]() { publishDynamicTransforms(); });
  } else {
    static_tf_broadcaster_->sendTransform(static_tf_msgs_);
  }
}
```

- 코드 확인 결과, transform의 의미보다 시간 동기화를 위해 구현해둔 것으로 보입니다.

```cpp
void OBCameraNode::publishDynamicTransforms() {
  RCLCPP_WARN(logger_, "Publishing dynamic camera transforms (/tf) at %g Hz", tf_publish_rate_);
  std::mutex mu;
  std::unique_lock<std::mutex> lock(mu);
  while (rclcpp::ok() && is_running_) {
    tf_cv_.wait_for(lock, std::chrono::milliseconds((int)(1000.0 / tf_publish_rate_)),
                    [this] { return (!(is_running_)); });
    {
      rclcpp::Time t = node_->now();
      for (auto& msg : static_tf_msgs_) {
        msg.header.stamp = t;
      }
      dynamic_tf_broadcaster_->sendTransform(static_tf_msgs_);
    }
  }
}
```

- startPipeline에서 호출되는 함수들은 다음과 같은 역할을 갖습니다.

| Function          | Description                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------- |
| ob::Pipeline 실행 | orbbec sdk로부터 pipeline을 다루는 핸들러를 받아 image stream을 시작합니다.                       |
| frameSetCallback  | setupPublishers에서 설정한 각종 topic publisher들이 실질적인 topic publish를 수행하는 부분입니다. |

- 코드를 확인해보면, orbbec SDK의 pipeline을 실행한 뒤, frame callback과 binding하고 있는 모습을 확인 가능합니다.

```cpp
void OBCameraNode::startPipeline() {
  if (pipeline_ != nullptr) {
    pipeline_.reset();
  }
  pipeline_ = std::make_unique<ob::Pipeline>(device_);
  pipeline_->start(config_, [this](std::shared_ptr<ob::FrameSet> frame_set) {
    frameSetCallback(std::move(frame_set));
  });
}
```

- frame이 갱신될 떄마다 실행되는 frameSetCallback에서는 갱신된 이미지 데이터를 사용하여 각종 topic publish를 진행합니다.

```cpp
void OBCameraNode::frameSetCallback(std::shared_ptr<ob::FrameSet> frame_set) {
  auto color_frame = frame_set->colorFrame();
  auto depth_frame = frame_set->depthFrame();
  auto ir_frame = frame_set->irFrame();
  if (color_frame && enable_[COLOR]) {
    publishColorFrame(color_frame);
  }
  if (depth_frame && enable_[DEPTH]) {
    publishDepthFrame(depth_frame);
  }
  if (ir_frame && enable_[INFRA0]) {
    publishIRFrame(ir_frame);
  }
  publishPointCloud(frame_set);
}
```

- publishColorFrame / publishDepthFrame / publishIRFrame에서는 cv::Mat 형식의 데이터를 ROS 2 Image msg로 변환하여 publish하고 있습니다.

```cpp
void OBCameraNode::publishColorFrame(std::shared_ptr<ob::ColorFrame> frame) {
  ...
  sensor_msgs::msg::Image::SharedPtr img;
  img = cv_bridge::CvImage(std_msgs::msg::Header(), encoding_.at(stream), image).toImageMsg();

  img->width = width;
  img->height = height;
  img->is_bigendian = false;
  img->step = width * unit_step_size_[stream];
  img->header.frame_id = optical_frame_id_[COLOR];
  img->header.stamp = timestamp;
  auto& image_publisher = image_publishers_.at(stream);
  image_publisher.publish(img);
}
```

- publishPointCloud에서는 특정 조건에 따라 color pointcloud, depth point cloud를 준비시킵니다. 이 부분 때문에 parameter설정이 중요합니다.

```cpp
void OBCameraNode::publishPointCloud(std::shared_ptr<ob::FrameSet> frame_set) {
  try {
    if (align_depth_ && (format_[COLOR] == OB_FORMAT_YUYV || format_[COLOR] == OB_FORMAT_I420)) {
      if (frame_set->depthFrame() != nullptr && frame_set->colorFrame() != nullptr) {
        publishColorPointCloud(frame_set);
      }
    }
    if (frame_set->depthFrame() != nullptr) {
      publishDepthPointCloud(frame_set);
    }
```

- pointcloud를 publish하는 로직을 살펴보면 공통적으로 아래와 같은 iterator 구현을 확인할 수 있습니다. 이는 ROS 2의 PointCloud2 type에서 제공하는 API로 손쉽게 각 point data를 append 할 수 있습니다.

```cpp
  ...
  sensor_msgs::PointCloud2Iterator<float> iter_x(point_cloud_msg_, "x");
  sensor_msgs::PointCloud2Iterator<float> iter_y(point_cloud_msg_, "y");
  sensor_msgs::PointCloud2Iterator<float> iter_z(point_cloud_msg_, "z");
  size_t valid_count = 0;
  for (size_t point_idx = 0; point_idx < point_size; point_idx++, points++) {
    bool valid_pixel(points->z > 0);
    if (valid_pixel) {
      *iter_x = static_cast<float>(points->x / 1000.0);
      *iter_y = -static_cast<float>(points->y / 1000.0);
      *iter_z = static_cast<float>(points->z / 1000.0);
      ++iter_x;
      ++iter_y;
      ++iter_z;
      ++valid_count;
    }
  }
```

- 마지막으로 ros_service.cpp 코드에는 각종 설정을 service로 제어할 수 있는 로직이 구현되어 있습니다. 자세한 내용을 살펴보지는 않겠지만, service server를 설정하고 binding되는 callback을 구현하는 부분이 반복됩니다.

```cpp
service_name = "set_" + stream_name + "_exposure";
set_exposure_srv_[stream_index] = node_->create_service<SetInt32>(
    service_name,
    [this, stream_index = stream_index](const std::shared_ptr<SetInt32::Request> request,
                                        std::shared_ptr<SetInt32::Response> response) {
      setExposureCallback(request, response, stream_index);
    });
...

void OBCameraNode::setExposureCallback(const std::shared_ptr<SetInt32::Request>& request,
                                       std::shared_ptr<SetInt32::Response>& response,
                                       const stream_index_pair& stream_index) {
  auto stream = stream_index.first;
  ...
```

- OBCameraNodeFactory는 OBCameraNode, ob::Device, ob::DeviceInfo 인스턴스를 스마트 포인터로 포함하고 있습니다. 대부분의 구현들도 해당 스마트 포인터 내부의 메소드를 호출하는 식으로 구성됩니다.

```cpp
void init();
void startDevice();
void getDevice(const std::shared_ptr<ob::DeviceList>& list);
void updateDeviceInfo();
void deviceConnectCallback(const std::shared_ptr<ob::DeviceList>& device_list);
void deviceDisconnectCallback(const std::shared_ptr<ob::DeviceList>& device_list);
void printDeviceInfo(const std::shared_ptr<ob::DeviceInfo>& device_info);

std::unique_ptr<OBCameraNode> ob_camera_node_;
std::shared_ptr<ob::Device> device_;
std::shared_ptr<ob::DeviceInfo> device_info_;
```

<aside>
💡

</aside>

이 부분은 구조적으로 다소 비효율적이라고 생각합니다. orbbec 관련 인스턴스를 클래스 변수로 갖는 것은 이해가 되지만, 저라면 Node를 스마트 포인터로 갖지 않고, Composition 형태로 바로 구현했을 것 같습니다.

지금까지 orbbec astra plus model의 ROS 2 패키지를 분석해보았습니다.
