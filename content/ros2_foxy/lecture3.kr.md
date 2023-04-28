---
title: "Lecture3. Core of ROS"
date: 2023-04-28T10:51:29+09:00
draft: false
---

## Husky Gazebo

강의에 앞서, Gazebo상에 ClearPath의 Husky라는 로봇을 등장시켜보고자 합니다.

![Untitled.png](/kr/ros2_foxy/images3/Untitled.png?height=300px)

image from : [clearpath robotics](https://clearpathrobotics.com/husky-unmanned-ground-vehicle-robot/)

- 예제 패키지 설치

```
sudo apt-get update
sudo apt install ros-foxy-husky-gazebo
sudo apt install ros-foxy-husky-simulator
sudo apt install ros-foxy-husky-viz
```

- 시뮬레이션 실행

```bash
# Terminal 1
ros2 launch husky_gazebo husky_playpen.launch.py
```

![Untitled1.png](/kr/ros2_foxy/images3/Untitled1.png?height=300px)

- Rviz2 + Controller 실행 (Rviz2상의 interactive marker를 사용하여 로봇을 이동시킬 수 있습니다.)

```python
# Terminal 2
ros2 launch husky_viz view_robot.launch.py
```

![Untitled2.png](/kr/ros2_foxy/images3/Untitled2.png?height=400px)

- 다음으로, 새로운 터미널에서 아래 커멘드 라인을 실행시켜 봅시다.

```bash
rqt_graph
```

![rosgraph.png](/kr/ros2_foxy/images3/rosgraph.png?height=300px)

위 그림은 방금 전 실행한 예제 내부적으로 어떠한 동작들이 이루어지고 있었는지를 보여주는 것으로, 강의를 마칠 때면 여러분들은 위 그림이 어떠한 의미를 갖는지 모두 이해하실 수 있을 것입니다.

다음으로, 터미널을 새로 실행시켜 아래의 명령어들을 실행시켜 봅시다.

- **ros2 node list**

```bash
$ ros2 node list
/controller_manager
/ekf_node
/gazebo
/gazebo_ros2_control
/gps_plugin
/husky_velocity_controller
/imu_filter
/imu_plugin
...
```

- **ros2 topic list**

```bash
$ ros2 topic list
/clicked_point
/clock
/cmd_vel
/diagnostics
/dynamic_joint_states
/e_stop
/goal_pose
/gps/data
/husky_velocity_controller/cmd_vel_unstamped
/imu/data
/imu/data_raw
/imu/mag
/initialpose
...
```

> 위 명령어들이 어떠한 의미를 갖는지 하나하나씩 함께 살펴보겠습니다.

## ROS 2 Node

> ROS 2는 각 프로세스들을 **Node**라는 단위로 관리합니다.

- 로봇을 움직이는 Node, 센서와 통신하는 Node, 시각화 Node 등 다양한 Node들이 얽혀 로봇 시스템을 구성하게 됩니다.
- Node들 사이에는 **데이터의 송수신**이 필요합니다. 이를 담당하는 ROS의 통신 메커니즘들이 있으며 각기 다른 특성을 갖고 있습니다. (topic, service, action)
- 실행중인 Node를 확인하는 방법으로 저는 크게 두가지를 사용합니다.

1. **ROS 2** **command line interface**

```bash
$ ros2 node list
/node1
/node2
/node3
...
```

1. **rqt_graph**

```bash
rqt_graph
# or
ros2 run rqt_graph rqt_graph
```

- rqt graph를 살펴보면, 동그란 **Node**와 Node들 사이의 데이터 송수신이 **화살표**로 표현된 것을 알 수 있습니다.
- Rviz2 marker를 통해 제어 데이터가 송신되어 twist_mux로 전달되고 있으며, 따라서 gazebo는 이 데이터를 통해 실제 로봇을 움직이게 되는 것입니다.

![Untitled3.png](/kr/ros2_foxy/images3/Untitled3.png?height=300px)

- 특정 Node에 대해서 더 자세한 정보를 얻고 싶다면, info 커맨드를 사용합니다.

```bash
$ ros2 node info /ekf_node
/ekf_node
  Subscribers:
    /imu/data: sensor_msgs/msg/Imu
    /odom: nav_msgs/msg/Odometry
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /set_pose: geometry_msgs/msg/PoseWithCovarianceStamped
  Publishers:
    /diagnostics: diagnostic_msgs/msg/DiagnosticArray
    /odometry/filtered: nav_msgs/msg/Odometry
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /rosout: rcl_interfaces/msg/Log
    /tf: tf2_msgs/msg/TFMessage
  Service Servers:
    /ekf_node/describe_parameters: rcl_interfaces/srv/DescribeParameters
    ...
```

- 현재 예시를 실행하며 수많은 프로그램들이 동작되어 매우 복잡한 상황입니다. 이전 예제들은 일단 종료시킨 뒤, 보다 간단한 예시를 실행해봅시다.

```bash
# Terminal 1
ros2 run examples_rclcpp_minimal_publisher publisher_member_function
# Terminal 2
ros2 run examples_rclcpp_minimal_subscriber subscriber_member_function
# Terminal 3
rqt_graph
```

![ros_1.gif](/kr/ros2_foxy/images3/ros_1.gif?height=400px)

> rqt_graph를 보면 **publisher ⇒ subscriber** 로 데이터가 전송되는 것을 알 수 있습니다.

![Untitled4.png](/kr/ros2_foxy/images3/Untitled4.png?height=300px)

이것이 바로 ROS 2가 데이터를 송수신하는 가장 기본적인 구조입니다.

1. 서로 다른 두 프로그램(Node)을 생성하고
2. ROS 2에서 사용하는 특정한 형식에 이 둘 사이에 오가야 하는 데이터를 채워넣습니다.
3. 두 프로그램 사이 통신은 DDS를 사용합니다.

![Untitled5.png](/kr/ros2_foxy/images3/Untitled5.png?height=300px)

- image from : [MDS 테크](https://blog.naver.com/PostView.naver?blogId=neos_rtos&logNo=220346180839&parentCategoryNo=&categoryNo=20&viewDate=&isShowPopularPosts=false&from=postList)

## ROS 2 Workspace 생성

- ROS 2의 Package는 관련된 파일들과 코드를 한데 모아둔 폴더이자, 빌드의 단위입니다.
- 그리고 이러한 패키지는 특정 Workspace안에 담겨있어야 합니다.

![Untitled6.png](/kr/ros2_foxy/images3/Untitled6.png?height=300px)

- image from : [packthub](https://subscription.packtpub.com/book/hardware+and+creative/9781783551798/1/ch01lvl1sec11/understanding-the-ros-file-system-level)

#### Workspace??

마치 파이썬의 가상 환경을 사용하는 것처럼, ROS 2 개발을 하다 보면 진행하는 프로젝트에 따라 하나의 PC에 여러 ROS 2 환경이 필요해지는 경우가 생깁니다.

- 강의용 WS
- 개발용 WS
- 배포용 Clean WS
- etc…

![Untitled7.png](/kr/ros2_foxy/images3/Untitled7.png?height=200px)

ROS 2에서 사용중인 colcon 시스템은 Workspace를 나눔으로 이 작업을 손쉽게 해줍니다.

이번 강의용 WS를 만들어보겠습니다. 이후 여러분들만의 WS도 만들어 작업해보세요!

- 일반적으로 ROS 2의 workspace는 **<name>\_ws**라는 이름을 갖는 것이 일반적이며, 우리는 **ros2_ws**라는 workspace를 만들어보고자 합니다.

```bash
cd ~/
# colcon workspace는 반드시 src라는 폴더가 있어야 합니다.
mkdir -p ros2_ws/src
cd ros2_ws
colcon build
```

![Untitled8.png](/kr/ros2_foxy/images3/Untitled8.png?height=200px)

- image from : [colcon.readthedocs](https://colcon.readthedocs.io/en/released/user/what-is-a-workspace.html)

```python
ros2_ws
├── build
│   └── COLCON_IGNORE
├── install
│   ├── COLCON_IGNORE
│   ├── setup.bash
├── log
└── src
```

작업을 완료하면, 위와 같은 폴더 구조가 자동 생성되었을 것입니다.

- **build** : 컴파일 된 프로그램, custom interface 등이 위치하게 됩니다.
- **install** : ros2 launch와 ros2 run 등의 명령어는 프로그램의 실행 시 이 install 폴더 내 파일들을 조회합니다. 일종의 바로가기들의 모임이라고 생각하면 됩니다.
- **log** : colcon build 시 발생하는 로드들이 위치하게 됩니다.
- **src** : 모든 소스 코드들이 위치하게 됩니다.

{{% notice note %}}
package를 지우고 싶은 경우에는 src에서 코드 원본을 지운 뒤, build와 install 폴더에서 해당 package에 해당하는 내용들도 삭제해야 합니다!
{{% /notice %}}

### Package 생성 실습

> ROS 2에서는 **colcon**을 사용하여 새로운 package를 생성합니다. 하지만, ROS 2에서는 사용하는 프로그래밍 언어에 따라 다른 build type을 사용합니다. 자주 사용되는 두 언어(python/c++)에 대해서 패키지를 생성해보겠습니다.

**ROS 2 Python Package**

- 파이썬 패키지의 생성은 다음과 같습니다.

```bash
$ ros2 pkg create --build-type ament_python <package_name>

# example
$ ros2 pkg create --build-type ament_python my_first_pkg
going to create a new package
package name: my_first_pkg
destination directory: /home/kimsooyoung/ros2_ws/src
package format: 3
version: 0.0.0
...
```

- 다음과 같은 폴더 구조가 자동 생성됩니다.

```bash
my_first_pkg/
├── my_first_pkg
│   └── __init__.py
├── package.xml
├── resource
│   └── my_first_pkg
├── setup.cfg
├── setup.py
└── test
    ├── test_copyright.py
    ├── test_flake8.py
    └── test_pep257.py
```

- 새롭게 생성된 패키지를 빌드해봅시다. (여기서의 빌드는 컴파일을 거치는 빌드가 아니라 colcon 시스템에서의 빌드를 뜻합니다.)

```bash
$ cd ~/ros2_ws
$ colcon build --packages-select my_first_pkg
Starting >>> my_first_pkg
Finished <<< my_first_pkg [1.56s]

Summary: 1 package finished [1.96s]
```

![colcon_build.gif](/kr/ros2_foxy/images3/colcon_build.gif?height=400px)

- 빌드 결과로, workspace 내부 build와 install 폴더 내부에 해당 패키지 이름을 갖는 폴더가 생성됩니다.

```bash
$ cd ~/ros2_ws/build
$ ls
**COLCON_IGNORE  my_first_pkg/

$ cd ~/ros2_ws/install
$ ls
**COLCON_IGNORE  my_first_pkg/ ...
```

### ROS 2 Cpp Package

> 대부분의 오픈소스 패키지들은 C++로 개발되어 있는 경우가 많습니다. 따라서, 실제 작업 시 파이썬을 위주로 사용하더라도 **적어도 C++ 패키지의 구조는 파악하고 있어야** 오류 상황에 대처할 수 있습니다.

- C++ 패키지의 생성은 다음과 같습니다.

```python
$ ros2 pkg create --build-type ament_cmake <package_name>

# example
$ ros2 pkg create --build-type ament_cmake my_first_cpp_pkg
going to create a new package
package name: my_first_cpp_pkg
destination directory: /home/kimsooyoung/ros2_ws/src
package format: 3
version: 0.0.0
...
```

- C++ 패키지는 다음과 같은 구조를 갖습니다. 파이썬과 달리, C++는 컴파일 언어이기 때문에 make를 위한 **CMakeLists.txt** 파일이 추가되어 있습니다.

```python
my_first_cpp_pkg
├── CMakeLists.txt
├── include
│   └── my_first_cpp_pkg
├── package.xml
└── src
```

{{% notice note %}}
일반적으로 C++ 개발을 하다보면 header와 source의 분리를 시키곤 합니다. 비슷하게, ROS 2에서도 보통 **include 폴더** 안에 헤더 파일을 위치시키고, **src 폴더** 안에 소스 코드를 위치시킵니다.

{{% /notice %}}

- 빌드와 실행은 파이썬과 동일한 커멘드 라인을 사용합니다.

```bash
$ colcon build --packages-select my_first_cpp_pkg
Starting >>> my_first_cpp_pkg
Finished <<< my_first_cpp_pkg [1.56s]

Summary: 1 package finished [1.96s]
```

이 시점에서, colcon을 사용하여 package를 빌드하는 방법들을 정리하여 소개합니다.

- **colcon build** : src 폴더 내부에 존재하는 모든 package들을 빌드합니다.
- **colcon build --packages-up-to <pkg-name>** : 해당 package의 종속성이 존재할 시, 이들을 먼저 빌드하고 pkg-name을 빌드합니다.
- **colcon build --packages-select <pkg-name>** : 해당 package만을 빌드합니다.

새로운 Package를 빌드했다면, 변경 내용을 갱신 (source) 해주어야 합니다. 이 작업에는 크게 두가지가 존재합니다.

- `source install/setup.bash` ⇒ workspace를 source하고 ROS 2시스템 전체를 갱신합니다.
- `source install/local_setup.bash` ⇒ workspace만을 sources합니다. (여러 ROS 2 workspace가 있는 경우 local_setup.bash를 사용합시다.)

> 참고로, setup.bash에는 결국 local_setup.bash가 포함되어 있습니다.

```python
# source chained prefixes
# setting COLCON_CURRENT_PREFIX avoids determining the prefix in the sourced script
COLCON_CURRENT_PREFIX="/opt/ros/foxy"
_colcon_prefix_chain_bash_source_script "$COLCON_CURRENT_PREFIX/local_setup.bash"

# source this prefix
# setting COLCON_CURRENT_PREFIX avoids determining the prefix in the sourced script
COLCON_CURRENT_PREFIX="$(builtin cd "`dirname "${BASH_SOURCE[0]}"`" > /dev/null && pwd)"
_colcon_prefix_chain_bash_source_script "$COLCON_CURRENT_PREFIX/local_setup.bash"

unset COLCON_CURRENT_PREFIX
unset _colcon_prefix_chain_bash_source_script
```

#### apt를 통한 패키지 설치하기

널리 사용되고 검증된 패키지들은 소스 코드 빌드 없이 apt 명령어 하나만으로 빌드된 형태를 사용할 수 있습니다. 다만, 같은 ROS 2일지라도 버전에 따라 재설치를 해줘야 합니다.

```python
$ sudo apt install ros-<DISTRO>-<pkg-name>
$ sudo apt install ros-foxy-turtlesim
```

{{% notice note %}}
일전, 우리가 사용했던 husky관련 패키지들은 모두 `apt install` 을 통해 설치가 가능하였기에 코드에 대한 고려 없이 바로 실행할 수 있던 것입니다.
{{% /notice %}}

> ROS 2에서는 husky를 비롯한 다양한 로보틱스 패키지들을 제공하며, 개발 초기에는 이를 사용하여 빠르게 검증을 하고, 이후 직접 Customizing해야 하는 부분은 별도 소스코드 빌드를 통해 업그레이드를 하곤 합니다.

### ROS 2 Launch

3종류의 ROS 2 프로그램을 실행한다고 해봅시다.

- A 프로그램 실행 터미널
- B 프로그램 실행 터미널
- C 프로그램 실행 터미널

터미널 3개 정도야 실행하면 그만일 수 있지만, 이 프로그램의 개수가 10개, 20개로 늘어난다고 생각해 봅시다. 😂

> 실행해야 하는 프로그램들을 모두 하나의 파일로 정리한 다음, 해당 파일만 실행시키면 어떨까요? 이것이 바로 launch의 개념입니다.

ROS 2의 launch파일은 python과 xml을 통해 구성할 수 있는데요, 저는 기본적으로 **Python** 문법을 선호합니다. 원래 사용하는 언어이기도 하고, 많은 오픈소스 프로젝트들이 Python launch를 사용하고 있기 때문입니다.

- 일전 실행했던 husky 예제 분석을 통해 launch 구조에 대해 짚고 넘어가겠습니다. ros2의 launch 명령어는 다음과 같이 구성되어 있습니다.

```bash
$ ros2 launch <package-name> <launch-file-name>
$ ros2 launch husky_gazebo husky_playpen.launch.py
```

- 해당 launch 파일을 열어보면, 아래와 같습니다.([링크](https://github.com/husky/husky/blob/foxy-devel/husky_gazebo/launch/husky_playpen.launch.py)) 참고로, python 파일의 경우 **<file-name>.launch.py**로 이름짓는 것이 암묵적인 규칙입니다.

```python
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import PathJoinSubstitution
from launch_ros.substitutions import FindPackageShare

def generate_launch_description():

    world_file = PathJoinSubstitution(
        [FindPackageShare("husky_gazebo"),
        "worlds",
        "clearpath_playpen.world"],
    )

    gazebo_launch = PathJoinSubstitution(
        [FindPackageShare("husky_gazebo"),
        "launch",
        "gazebo.launch.py"],
    )

    gazebo_sim = IncludeLaunchDescription(
        PythonLaunchDescriptionSource([gazebo_launch]),
        launch_arguments={'world_path': world_file}.items(),
    )

    ld = LaunchDescription()
    ld.add_action(gazebo_sim)

    return ld
```

- 항상 launch file의 분석은 가장 하단부터 시작합니다.

```python
    ld = LaunchDescription()
    ld.add_action(gazebo_sim)

    return ld
```

**LaunchDescription**안에 실행을 원하는 프로그램들을 전달하고, launch 명령 시 이들을 실제 실행하는 것이 launch file을 작성하는 이유입니다.

- 상단 옵션을 하나씩 살피겠습니다. launch file에서 특정 폴더, 파일에 접근하기 위해 package에 기반하여 경로를 탐색합니다.

```python
    world_file = PathJoinSubstitution(
        [FindPackageShare("husky_gazebo"),
        "worlds",
        "clearpath_playpen.world"],
    )

    gazebo_launch = PathJoinSubstitution(
        [FindPackageShare("husky_gazebo"),
        "launch",
        "gazebo.launch.py"],
    )
```

- world란 gazebo상의 환경을 나타내는 파일로 아래와 같은 텍스트 형식을 갖습니다. (이후 Gazebo 시간에 살펴보고자 합니다.)

```xml
<sdf version='1.4'>
  <world name='default'>
    <light name='sun' type='directional'>
      <cast_shadows>1</cast_shadows>
      <pose>0 0 10 0 -0 0</pose>
      <diffuse>0.8 0.8 0.8 1</diffuse>
      <specular>0.2 0.2 0.2 1</specular>
      <attenuation>
        <range>1000</range>
        <constant>0.9</constant>
        <linear>0.01</linear>
        <quadratic>0.001</quadratic>
      </attenuation>
      <direction>-0.5 0.1 -0.9</direction>
    </light>
    <model name='ground_plane'>
		...
```

- 현재 husky_playpen는 다른 launch 파일을 또다시 include하고 있습니다. 이렇게 실행시켜야 할 프로그램들이 많을 때, launch file 자체를 계층 구조로 실행시키는 것도 가능합니다.

```python
    gazebo_launch = PathJoinSubstitution(
        [FindPackageShare("husky_gazebo"),
        "launch",
        "gazebo.launch.py"],
    )

    gazebo_sim = IncludeLaunchDescription(
        PythonLaunchDescriptionSource([gazebo_launch]),
        launch_arguments={'world_path': world_file}.items(),
    )
```

- 하위 launch file인 [gazebo.launch.py](https://github.com/husky/husky/blob/foxy-devel/husky_gazebo/launch/gazebo.launch.py) 내부는 다음과 같습니다. 복잡한 형태를 갖고 있어 현재 모든 내용을 설명할 수는 없지만, launch 파일은 받드시 LaunchDescription을 포함하고, LaunchDescription으로 전달되는 프로그램들이 실행되는 주체임을 다시 한 번 강조드립니다!

```python
    ld = LaunchDescription(ARGUMENTS)
    ld.add_action(gz_resource_path)
    ld.add_action(node_robot_state_publisher)
    ld.add_action(spawn_joint_state_broadcaster)
    ld.add_action(diffdrive_controller_spawn_callback)
    ld.add_action(gzserver)
    ld.add_action(gzclient)
    ld.add_action(spawn_robot)
    ld.add_action(launch_husky_control)
    ld.add_action(launch_husky_teleop_base)

    return ld
```

### [간략버전] Launch file 작성법

> ROS 2는 워낙 방대한 오프 소스 패키지들을 갖고 있어 약간의 과장을 보태면 코드 한 줄 없이 로봇을 제작할 수도 있습니다.

하지만, 어떠한 프로그램들을 실행하고, 실행 시 옵션은 어떻게 할당해야 하는지는 알고 있어야 합니다. 이러한 이유에서, 프로그래밍 학습보다 앞서 ROS 2의 launch file 작성법을 간단하게 설명하고 넘어가겠습니다.

- 방금 전 생성한 my_first_pkg 패키지 내부에 launch 폴더를 생성하고, 아래 코드를 복사 & 붙여넣기 합니다. (my_first_pkg입니다! my_first_cpp_pkg가 아닙니다.)

```python
# my_first_launch.launch.py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():

    turtlesim_node = Node(
        package="turtlesim",
        executable="turtlesim_node",
        name="turtlesim_node",
    )

    return LaunchDescription([
        turtlesim_node,
    ])
```

| Keyword    | Description                                                                                                                                                                         |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Node       | Node 하나를 실행시킬 수 있는 옵션입니다.                                                                                                                                            |
| package    | 실행시킬 Node가 포함된 package를 선택해줍니다.                                                                                                                                      |
| executable | c++ Node의 경우, colcon build를 하면 실행 가능한 프로그램이 생성됩니다. python의 경우도 스크립트를 실행 시키게 되며, 이는 추후 코딩 실습을 거치면 완벽히 이해하실 수 있을 것입니다. |
| parameters | 실행시킬 Node의 추가 매개변수가 있다면 이 부분에 추가됩니다.                                                                                                                        |

자, 새로운 launch 파일을 실행해보겠습니다.

```python
$ ros2 launch my_first_pkg my_first_launch.launch.py
file 'my_first_launch.launch.py' was not found in the share directory of package...
```

⇒ 오류가 발생하였습니다! 이는 install 폴더 내부에 symbolic link가 생성되지 않아 발생한 오류입니다.

colcon build 시 colcon은 install/<package_name>/share 폴더 내부에 파일들의 symbolic link를 생성합니다. 더불어 어떠한 파일들에 대해 link를 만들 것인지 전달해주어야 하며, 이는 setup.py에서 설정 가능합니다.

- setup.py 수정

```python
import os
from glob import glob
from setuptools import setup

package_name = 'my_first_pkg'

setup(
    name=package_name,
    version='0.0.0',
    packages=[package_name],
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        (os.path.join('share', package_name, 'launch'), glob('launch/*.launch.py')),
    ],
    ...
```

- 다음으로 아래의 커멘드 라인을 순서대로 입력해 주세요!

```bash
# 폴더 이동
cd ~/ros2_ws

# colcon build
colcon build --packages-select my_first_pkg

# 변경 적용
source install/setup.bash

# 실행!
ros2 launch my_first_pkg my_first_launch.launch.py
```

![Untitled9.png](/kr/ros2_foxy/images3/Untitled9.png?height=400px)

이렇게 launch file을 직접 생성해 보았습니다. 주목할 점은, 제가 보여드린 것이 하나의 예시일 뿐이며, launch file은 여러 형식으로 사용된다는 것입니다.

```python
    # my_first_pkg
    return LaunchDescription([
        turtlesim_node,
    ])

    # husky_gazebo
    ld = LaunchDescription(ARGUMENTS)
    ld.add_action(gz_resource_path)
    ld.add_action(node_robot_state_publisher)
    ld.add_action(spawn_joint_state_broadcaster)
    ld.add_action(diffdrive_controller_spawn_callback)
    ld.add_action(gzserver)
    ld.add_action(gzclient)
    ld.add_action(spawn_robot)
    ld.add_action(launch_husky_control)
    ld.add_action(launch_husky_teleop_base)

    return ld
```

> ROS 2 자체가 오픈소스에 기반하기 때문에 사용자들마다 각기 다른 방식이 있습니다. 모든 방식을 다 설명할 수는 없으나 가장 널리 사용되는 것들은 대부분 같이 다루어보겠습니다!
