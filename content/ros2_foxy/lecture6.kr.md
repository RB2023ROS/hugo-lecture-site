---
title: "Lecture6. ROS 2 Tools"
date: 2023-04-28T10:51:28+09:00
draft: false
---

![Untitled.png](/kr/ros2_foxy/images6/Untitled.png?height=200px)

ROS 2는 rqt라는 프로그램의 집합을 제공합니다. rqt의 GUI를 사용하면 데이터 시각화, 코딩 없이 통신 메커니즘과 상호 작용, tf frame의 시각화 등 ROS 2 프로그래밍이 매우 매우 편해집니다.

> 이번 시간을 통해 몇가지 자주 사용되는 rqt들을 살펴보고, 약간의 실습도 진행해보겠습니다.

- rqt의 실행은 다음과 같습니다.

```python
rqt
```

![Untitled1.png](/kr/ros2_foxy/images6/Untitled1.png?height=300px)

⇒ 초기 실행 시 빈 화면이 등장하지만, 상단 탭을 통해 수많은 응용 프로그램들을 확인할 수 있습니다.

## rqt_graph

> rqt_graph는 node와 topic에 대해 시각화하여 한 장의 이미지를 보여주는 툴로, 가장 많이 사용되는 Rqt Tool입니다.

- rqt_graph는 자주 사용되는 만큼 보통 개별적으로 실행합니다.

```python
rqt_graph
```

- 사진에 보이는 동그라미는 node를, 네모는 topic을 의미합니다. 더불어 화살표의 방향을 통해 topic의 송/수신자를 구별할 수 있습니다.

![Untitled2.png](/kr/ros2_foxy/images6/Untitled2.png?height=300px)

- 더불어 rqt_graph는 다양한 시각화 옵션들을 제공하고 있습니다.

| Node Option           | Hide Option | ETC          |
| --------------------- | ----------- | ------------ |
| Node Only             | Dead sinks  | Group Option |
| Nodes/Topics (Active) | Leaf Topics | Actions      |
| Nodes/Topics (All)    | Debug       | tf           |
|                       | tf          | Images       |
|                       | Unreachable | Highlight    |
|                       | Params      | Fit          |

⇒ 참고로, rqt_graph의 node에 마우스 커서를 가져가면 관련 topic들의 색상을 바꿔줍니다. (Highlight)

{{% notice tip %}}
Topic 이름을 잘못 생성하거나 겹치는 경우, 코드 차원에서 디버깅이 힘들지만 rqt_graph를 사용하면 손쉽게 판별 가능합니다.
{{% /notice %}}

## Topic Tools - Message Publisher & Topic Monitor

> rqt 상단의 탭에서 다음과 같이 화면을 구성합니다.

1. **plugins ⇒ topics ⇒ Message Publisher**
2. **pulgins ⇒ topics ⇒ Topic Monitor**

![Untitled3.png](/kr/ros2_foxy/images6/Untitled3.png?height=300px)

- message publisher를 사용하면 코딩 없이 원하는 topic을 publish 가능합니다. Topic Msg에 원하는 데이터를 채워넣은 뒤, 체크박스를 클릭하면 로봇이 움직이기 시작합니다. 실습을 위해 일전 husky gazebo를 실행시키고 로봇을 제어해 보겠습니다.

```bash
# Terminal 1
ros2 launch husky_gazebo husky_playpen.launch.py

# Terminal 2
rqt
```

![Untitled4.png](/kr/ros2_foxy/images6/Untitled4.png?height=400px)

- Topic Monitor를 사용하면, Topic 데이터들을 효과적으로 모니터링 가능합니다. Topic Publisher와 동일하게 체크박스를 눌러 topic을 선택한 뒤, 변하는 데이터를 확인해봅시다.

![Untitled5.png](/kr/ros2_foxy/images6/Untitled5.png?height=400px)

## Service Tools - Service Caller

> rqt 상단의 탭에서 다음과 같이 화면을 구성합니다.

- **plugins ⇒ services ⇒ Service Caller**

![Untitled6.png](/kr/ros2_foxy/images6/Untitled6.png?height=300px)

- Service Caller를 통해 코딩 없이 간편하게 Service Call이 가능합니다. Service Srv에 원하는 데이터를 채워넣은 뒤, Call 버튼을 클릭하면 Request와 Response를 받을 수 있습니다. Gazebo상의 물체를 제거하는 간단한 실습을 해봅시다.

1. gazebo_ros와 rqt를 실행합니다.

```python
# Terminal 1 - Run Gazebo ROS
ros2 launch gazebo_ros gazebo.launch.py

# Terminal 2 - rqt & Service Caller
rqt
```

2. rqt Service Caller를 실행시킨 뒤, /delete_entity service를 선택합니다.

![Untitled7.png](/kr/ros2_foxy/images6/Untitled7.png?height=400px)

3. gazebo상에서 모델을 등장시킨 뒤, 해당 모델의 이름을 service srv를 통해 전달하면 물체가 사라집니다!

![rqt_srv.gif](/kr/ros2_foxy/images6/rqt_srv.gif?height=200px)

## Rviz2

> Rviz2는 ROS 2 개발 중 발생하는 수많은 로봇 데이터들을 시각화해주는 고마운 툴입니다. 다른 rqt 프로그램들은 몰라도, Rviz2의 사용법은 반드시 익혀두시기를 추천합니다.

- rviz2 예시를 위해 좀 더 갖춰진 gazebo 환경을 실행해보겠습니다.

```python
# package build
cd ~/ros2_ws
cbp src_gazebo

# demo env launch
ros2 launch src_gazebo racecourse.launch.py
```

![Untitled8.png](/kr/ros2_foxy/images6/Untitled8.png?height=400px)

⇒ 현재는 제가 미리 준비해둔 rviz2 화면이 등장했지만, 실습을 통해 이런 rviz2 화면을 구성하고 launch file을 통해 실행하는 법까지 다루어 보겠습니다.

- Rviz2의 개별 실행은 터미널에서 가능합니다.

```python
rviz2
```

- rviz에서 view를 전환하는 방식은 gazebo와는 다르게 마우스 휠 클릭이 평행이동, 왼쪽 클릭이 회전이동입니다. 더불어, 우측 상단의 view를 통해 시점을 바꿀 수도 있습니다.

⇒ Orbit이 기본이며, 종종 TopDownOrtho도 사용됩니다.

![Untitled9.png](/kr/ros2_foxy/images6/Untitled9.png?height=100px)

- 좌측 Displays 패널을 통해 각종 Topic, TF 데이터들을 추가, 수정, 옵션 변경할 수 있습니다.

![Untitled10.png](/kr/ros2_foxy/images6/Untitled10.png?height=300px)

- 새로운 display를 추가하기 위해서, 패널 하단 Add 버튼을 사용합니다.

![Untitled11.png](/kr/ros2_foxy/images6/Untitled11.png?height=150px)

{{% notice tip %}}
rviz2에서는 By display type / By Topic의 두가지 add option을 제공합니다.
{{% /notice %}}

- Topic type을 사용하면 rviz2가 현재 실행되고 있는 topic을 인식하여 적합한 시각화를 해주게됩니다.

- Display type은 원하는 시각화 타입을 먼저 선택하고, 해당 타입을 갖는 topic을 사용자가 매칭하는 방식입니다.

![Untitled12.png](/kr/ros2_foxy/images6/Untitled12.png?height=300px)

- 시각화 시, 색상/크기/투명도 등 다양한 옵션을 바꿀 수 있습니다.

![Untitled13.png](/kr/ros2_foxy/images6/Untitled13.png?height=300px)

- ros2 launch로 rviz2 실행하기

rviz2 또한 ros2 node입니다. 그렇기에 launch file에서의 실행은 일반 node와 동일합니다.

```python
    # Launch RViz
    rviz = Node(
        package='rviz2',
        executable='rviz2',
        name='rviz2',
        output='screen',
    )
```

- 하지만, 이렇게 실행 시, 매번 시각화할 데이터를 선택하고, 옵션을 바꾸는 등의 수고스러움이 생깁니다. 따라서 rviz2는 설정을 저장하고 불러오는 기능을 제공합니다. (이는 rviz2 상단 패널에서도 확인 가능합니다.)

![Untitled14.png](/kr/ros2_foxy/images6/Untitled14.png?height=300px)

- save config를 통해 나만의 설정을 저장하고, launch file을 통해 실행하는 실습을 진행해봅시다.

1. 여러분만의 rviz2 화면을 구성합니다.

![Untitled15.png](/kr/ros2_foxy/images6/Untitled15.png?height=400px)

2. 해당 설정(config file)을 특정 패키지 내부에 저장하고, 위치를 기억합니다.

![Untitled16.png](/kr/ros2_foxy/images6/Untitled16.png?height=300px)

3. 패키지 내 새로운 launch file을 작성하고 rviz2 node를 추가한 뒤, config file을 옵션으로 전달합니다.

```python
import os

from ament_index_python.packages import get_package_share_directory

from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():

    pkg_path = os.path.join(get_package_share_directory('src_gazebo'))
    rviz_config_file = os.path.join(pkg_path, 'rviz', 'gazebo_racecourse.rviz')

    # Launch RViz
    rviz = Node(
        package='rviz2',
        executable='rviz2',
        name='rviz2',
        output='screen',
        arguments=['-d', rviz_config_file]
    )

    return LaunchDescription([
        rviz
    ])
```

- 참고로, 새로운 폴더가 추가되었을 시 CMakeLists.txt에 해당 폴더를 추가하여야 이후 launch file에서 접근 가능합니다.

```python
install(
  DIRECTORY
    launch
    meshes
    models
    config
    worlds
    rviz
    urdf
  DESTINATION
    share/${PROJECT_NAME}/
)
```

- 새로운 파일, 폴더가 추가되었으므로 colcon build를 수행한 뒤, 예제를 실행합니다.

```python
# Package build again
cbp src_gazebo
source install/local_setup.bash

# execute your launch file
ros2 launch src_gazebo <your-launch-file>
```

## rqt_robot_steering

- rqt_robot_steering를 사용하면 움직이는 로봇을 조종하기 위해 별도의 코딩을 하지 않고도 GUI를 통한 조종이 가능합니다.

```python
# package install
sudo apt install ros-foxy-rqt-robot-steering

# run node
rqt_robot_steering
```

- GUI의 가로 슬라이드는 angular velocity, 세로 슬라이드는 linear velocity를 뜻하며 로봇의 topic이름을 맞춰준 뒤 원하는 조종이 가능합니다.

![Untitled17.png](/kr/ros2_foxy/images6/Untitled17.png?height=400px)

- 실습을 위해 이전 husky 예시를 실행하고 rqt_robot_steering를 통해 로봇을 조종해보겠습니다.

```python
# Terminal 1
ros2 launch husky_gazebo husky_playpen.launch.py

# Terminal 2
rqt_robot_steering
```

![Untitled18.png](/kr/ros2_foxy/images6/Untitled18.png?height=400px)

## **rosbag2**

> rosbag은 프로그램 동작 중 발생하는 message 데이터를 기록하고 복기할 수 있게 해주는 툴입니다. 로봇 알고리즘을 개발할 때, 같은 상황에 대해 성능을 비교하는 경우, 혹은 교육 목적으로 데이터셋을 제공하는 경우 등에 매우 유용하게 사용할 수 있습니다.

- rosbag 사용법을 함께 실습해보겠습니다! 실습을 위해 gazebo 예시를 빌드하고 실행합니다.

```bash
# Example package clone
cd ~/ros2_ws/src
git clone https://github.com/RB2023ROS/gz_ros2_examples
cd ~/ros2_ws

# Example package build
cbp lidar_world

# Run Example package
ros2 launch lidar_world lidar_world.launch.py
```

![lidar_world_gz.gif](/kr/ros2_foxy/images6/lidar_world_gz.gif?height=400px)

⇒ 매우 복잡한 형상의 환경과 동적 물체들이 등장합니다. 아마 고사양의 PC가 아닌 이상 이번 예제는 실행되지 않을 것입니다.

⇒ 더불어 이런 여러 기능을 가진 시뮬레이션은 매우 느리게 동작하기 때문에 정확한 알고리즘의 검증이 불가능합니다.

- ROS 개발자들은 이러한 이유로 **rosbag2**를 통해 환경에서의 데이터셋을 제작합니다. 실제로 컴퓨터 비전 학회나 로봇 학회에서도 본인의 테스트베드를 rosbag으로 저장한 뒤 publish하는 경우가 많습니다.

![Untitled19.png](/kr/ros2_foxy/images6/Untitled19.png?height=200px)

- 그럼 실습을 통해 rosbag2의 사용 방법을 알아봅시다.

```python
# Terminal 1 - gazebo env
ros2 launch lidar_world lidar_world.launch.py

# Terminal 2 - rosbag2 install & run
sudo apt-get install ros-foxy-ros2bag ros-foxy-rosbag2*

# create seperate dir for rosbag2
mkdir ~/ros2_ws/rosbag2
cd ~/ros2_ws/rosbag2

ros2 bag record -a -o lidar_world

...
[INFO] [1681816765.163369771] [rosbag2_transport]: Listening for topics...
[INFO] [1681816765.164494950] [rosbag2_transport]: Subscribed to topic '/tf_static'
[INFO] [1681816765.165085302] [rosbag2_transport]: Subscribed to topic '/clicked_point'
[INFO] [1681816765.165696583] [rosbag2_transport]: Subscribed to topic '/tf'
[INFO] [1681816765.166304849] [rosbag2_transport]: Subscribed to topic '/goal_pose'
```

⇒ record의 종료는 ctrl + c를 사용합니다.

- info 옵션을 통해 저장을 마친 rosbag2의 정보를 조회할 수 있습니다. 16초 동안 약 570MB의 데이터가 저장된 것을 알 수 있습니다.

```bash
$ ros2 bag info lidar_world/lidar_world_0.db3
[INFO] [1681817128.588731891] [rosbag2_storage]: Opened database 'lidar_world/lidar_world_0.db3' for READ_ONLY.

Files:             lidar_world/lidar_world_0.db3
Bag size:          569.4 MiB
Storage id:        sqlite3
Duration:          16.325s
Start:             Apr 18 2023 20:19:25.172 (1681816765.172)
End:               Apr 18 2023 20:19:41.497 (1681816781.497)
Messages:          1068
Topic information: Topic: /clock | Type: rosgraph_msgs/msg/Clock | Count: 173 | Serialization Format: cdr
                   Topic: /performance_metrics | Type: gazebo_msgs/msg/PerformanceMetrics | Count: 82 | Serialization Format: cdr
...
```

- 저장 완료된 rosbag2를 다시 복기해보고 rviz2를 통해 시각화까지 해봅시다.

```bash
# Terminal 1 - ros2 bag play
$ ros2 bag play lidar_world/lidar_world_0.db3
[INFO] [1681817182.011798950] [rosbag2_storage]: Opened database 'lidar_world/lidar_world_0.db3' for READ_ONLY.
...

# Terminal 2 - rviz2
$ rviz2
```

⇒ Fixed Frame을 link_1으로 변경해야 포인트들을 확인 가능합니다.

![Untitled20.png](/kr/ros2_foxy/images6/Untitled20.png?height=400px)

한차례 실습을 진행해 보았는데요, 이제 각각의 커멘드 라인 옵션들을 살펴보겠습니다.

- ros2 bag 키워드

| Command | Description                               |
| ------- | ----------------------------------------- |
| record  | topic data를 저장합니다.                  |
| info    | 저장된 rosbag 데이터의 정보를 조회합니다. |
| play    | 저장된 rosbag 데이터를 재생합니다.        |

- ros2 bag record 옵션

| Command | Description                                                                         |
| ------- | ----------------------------------------------------------------------------------- |
| -a      | 모든 topic data를 저장합니다.                                                       |
| -o      | 저장될 rosbag의 이름을 지정합니다.                                                  |
| -b      | rosbag 저장 시, 최대 사이즈를 지정하며 사이즈 초과 시 새로운 rosbag으로 저장됩니다. |
| -d      | rosbag 저장 시, 최대 시간을 지정하며 사이즈 초과 시 새로운 rosbag으로 저장됩니다.   |

{{% notice note %}}
더불어, convert 기능을 통해 rosbag2를 쪼개거나 합할 수 있습니다. 자세한 사용법은 링크로 남겨두겠습니다. [https://github.com/ros2/rosbag2#converting-bags](https://github.com/ros2/rosbag2#converting-bags)
{{% /notice %}}

> 실제 로봇 제품이 없더라도 기록된 rosbag2 데이터를 통해 알고리즘의 검증과 시연이 가능합니다. 이번 강의에서도 rosbag2 데이터를 적극 사용하고자 하니 잘 기억해주세요!
