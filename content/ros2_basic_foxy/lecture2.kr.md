---
title: "Lecture2 - ROS 2 Node, Package"
date: 2023-01-02T12:39:13+09:00
draft: false
---

> ROS Noetic 강의를 이미 학습하였기 때문에, Node, Package, launch file, Topic, Message, Service와 같은 개념은 이미 알고 계실 것이라 생각합니다. (맞죠?)

혹시나 잊어버리셨을 수 있으니 강의 중간중간 제가 간단히 리뷰를 하면서 진행하겠습니다.

### ROS 2 Node

![lec2_0.png](/kr/ros2_basic_foxy/images2/lec2_0.png?height=300px)

- image from : [docs.rog.org](https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html)

> ROS에서의 Node는 하나의 **실행 가능한 프로그램**을 이야기하며, 각각의 Node는 본인 고유의 기능을 담당합니다.

서로 다른 역할을 담당하는 Node들이 각자의 동작을 수행하고 서로 간의 데이터를 주고받게 됨으로 시스템을 형성하게 되고, 따라서, ROS를 통한 로봇 개발은 곧, 적절한 Node들의 개발이라고도 말할 수 있습니다.

- ROS 2 Node 실행 - ROS 2에서 Node의 실행은 `ros2 run <pkg-name> <executable-name>`입니다.

```bash
sudo apt install ros-foxy-turtlesim -y
ros2 run turtlesim turtlesim_node
```

![lec2_1.png](/kr/ros2_basic_foxy/images2/lec2_1.png?height=300px)

- ROS 2의 Node를 다루는 명령어는 ROS 1과 크게 다르지 않습니다.

```bash
$ ros2 node list
/teleop_turtle
/turtlesim

$ ros2 node info /turtlesim
/turtlesim
  Subscribers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /turtle1/cmd_vel: geometry_msgs/msg/Twist
  Publishers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /rosout: rcl_interfaces/msg/Log
    /turtle1/color_sensor: turtlesim/msg/Color
    /turtle1/pose: turtlesim/msg/Pose
  Service Servers:
    /clear: std_srvs/srv/Empty
  …
```

- rqt_graph도 동일하게 사용 가능합니다.

```bash
$ rqt_graph
```

![lec2_2.png](/kr/ros2_basic_foxy/images2/lec2_2.png?height=200px)

### ROS 2 Package

> ROS의 Package는 파일 관점에서 관련된 소스코드, 라이브러리, 모델링 파일들, 설정 파일들을 한데 모아둔 폴더로 생각할 수 있으며, 기능 관점에서 시뮬레이션, 하드웨어 제어, 센서 다루기 등으로 분리시킨 모듈로 생각할 수 있습니다.

![lec2_3.png](/kr/ros2_basic_foxy/images2/lec2_3.png?height=300px)

- image from : [packthub](https://subscription.packtpub.com/book/hardware+and+creative/9781783551798/1/ch01lvl1sec11/understanding-the-ros-file-system-level)

새로운 패키지를 생성하는 과정에서, ROS 2는 ROS 1과 차이점이 있습니다.

ROS 1에서 **catkin_create_package**를 사용했던 것처럼, ROS 2에서는 **colcon**을 사용하여 새로운 package를 생성합니다. 하지만, ROS 2에서는 사용하는 프로그래밍 언어(python/c++)에 따라 다른 build type을 지정할 수 있습니다.

### ROS 2 Python Package

- 파이썬 패키지의 생성은 다음과 같습니다.

```bash
$ ros2 pkg create --build-type ament_python <package_name>
$ ros2 pkg create --build-type ament_python my_first_pkg
going to create a new package
package name: my_first_pkg
destination directory: /home/kimsooyoung/ros2_ws/src
package format: 3
version: 0.0.0
...
```

- 그리고, 다음과 같은 폴더 구조가 자동 생성됩니다.

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

- 새롭게 만든 패키지를 빌드해봅시다.

```bash
$ cd ~/ros2_ws
$ colcon build --packages-select my_first_pkg
Starting >>> my_first_pkg
Finished <<< my_first_pkg [1.56s]

Summary: 1 package finished [1.96s]

$ source install/local_setup.bash
```

![colcon_build.gif](/kr/ros2_basic_foxy/images2/colcon_build.gif?height=300px)

생성한 파이썬 코드 패키지에서, 실제 파이썬 코드는 어디에 위치할까요?

- 새로운 패키지를 생성하면 해당 패키지 폴더 안에 패키지 이름과 동일한 폴더가 하나 위치하게 됩니다. 이 폴더 안에는 **init.py** 파일이 기본적으로 위치합니다.

![lec2_4.png](/kr/ros2_basic_foxy/images2/lec2_4.png?height=300px)

참고로, wsl에서 파일 탐색기를 열기 위해서는 `explorer.exe .` 를 입력하시면 됩니다.

![lec2_5.png](/kr/ros2_basic_foxy/images2/lec2_5.png?height=200px)

- 강의 예시 소스코드를 하나 끌어와 빌드해보겠습니다. 아주 간단한 파이썬 예시가 실행될 것입니다.

```bash
$ cd ~/ros2_sw

$ colcon build --packages-select py_node_tutorial
Starting >>> py_node_tutorial
Finished <<< py_node_tutorial [1.85s]

Summary: 1 package finished [2.14s]

$ source install/local_setup.bash
$ ros2 run py_node_tutorial example_node_1
[INFO] [1672462290.252773294] [example_node_1]: ==== Hello ROS 2 ====
```

![ros2_build_ex.gif](/kr/ros2_basic_foxy/images2/ros2_build_ex.gif?height=300px)

{{% notice note %}}
여기서 주목할 점은, node의 실행 시 **roscore가 없어도 된다**는 점입니다. DDS를 사용함으로 각각의 node가 분산 처리가 가능하기 때문에 가능한 것입니다.
{{% /notice %}}

![lec2_6.png](/kr/ros2_basic_foxy/images2/lec2_6.png?height=300px)

- image from : [design.ros2.org](https://design.ros2.org/articles/ros_on_dds.html)

### ROS 2 C++ Package

> 대부분의 오픈소스 패키지들은 C++로 개발되어 있는 경우가 많습니다. turtlesim의 소스코드를 확인해봅시다.

[https://github.com/ros/ros_tutorials](https://github.com/ros/ros_tutorials)

모두 C++로 개발되어 있습니다. 따라서, 실제 작업 시 파이썬을 위주로 사용하더라도 **적어도 C++ 패키지의 구조는 파악하고 있어야** 오류 상황에 대처할 수 있습니다.

- C++ 패키지의 생성은 다음과 같습니다.

```python
$ ros2 pkg create --build-type ament_cmake <package_name>
$ ros2 pkg create --build-type ament_cmake my_first_cpp_pkg
going to create a new package
package name: my_first_cpp_pkg
destination directory: /home/kimsooyoung/ros2_ws/src
package format: 3
version: 0.0.0
...
```

- C++ 패키지는 다음과 같은 구조를 갖습니다. 파이썬과 달리, C++는 컴파일 언어이기 때문에 CMake를 위한 **CMakeLists.txt** 파일이 추가되어 있습니다.

```python
my_first_cpp_pkg
├── CMakeLists.txt
├── include
│   └── my_first_cpp_pkg
├── package.xml
└── src
```

일반적으로 C++ 개발을 하다보면 header와 source의 분리를 시키곤 합니다.

이를 위해 보통 **include 폴더** 안에 헤더 파일을 위치시키고, **src 폴더** 안에 소스 코드를 위치시킵니다.

- 빌드와 실행은 C++ 패키지도 동일한 커멘드 라인을 사용합니다.

```bash
$ colcon build --packages-select cpp_node_tutorial
$ source install/local_setup.bash
Starting >>> cpp_node_tutorial
Finished <<< cpp_node_tutorial [0.42s]

Summary: 1 package finished [0.64s]

$ ros2 run cpp_node_tutorial example_node_1
[INFO] [1672462275.581606122] [example_node_1]: ==== Hello ROS 2 ====
```

#### apt를 통한 패키지 사용하기

> 널리 사용되고 검증된 패키지들은 소스 코드 빌드 없이 명령어 하나만으로 사용할 수 있습니다. 다만, ROS 1과 ROS 2, 그리고 같은 ROS 2일지라도 버전에 따라 재설치를 해줘야 합니다.

```python
$ sudo apt install ros-<DISTRO>-<pkg-name>
$ sudo apt install ros-foxy-turtlesim
```

지금까지 우리가 사용했던 **turtlesim**, **turtle_teleop_key**는 모두 `apt install turtlesim`을 통해 설치가 가능하였기에 코드에 대한 고려 없이 바로 실행할 수 있던 것입니다.

이를 사용하여 ROS 개발 초기에는 바로 설치 가능한 패키지들을 조합하여 빠르게 검증을 하고, 이후 직접 Customizing해야 하는 부분은 별도 소스코드 빌드를 통해 업그레이드를 하곤 합니다.

### ROS 2 Launch

> ROS 2의 launch파일은 기본적으로 **Python** 문법을 사용합니다. 기존 ROS 1에서 xml 문법을 사용하던 것에서 비교하면, 보다 손쉽게 접근할 수 있어졌습니다.

- 예제를 통해 기본 구조에 대해 짚고 넘어가겠습니다.

```bash
$ cd ~/ros2_ws
$ colcon build --packages-select src_description
$ source install/local_setup.bash

$ ros2 launch src_description src_description.launch.py
```

사진과 같이 RViz2와 joint state publisher gui가 등장할 것입니다. joint state publisher gui의 슬라이드바를 이리저리 옮겨보세요.

ROS 1 강의에서 배워보았던 tf에 대해서도 복습할 수 있는 기회입니다.

![lec2_7.png](/kr/ros2_basic_foxy/images2/lec2_7.png?height=300px)

- ros2의 launch 명령어는 다음과 같이 구성되어 있습니다.

```bash
$ ros2 launch <package-name> <launch-file-name>
```

패키지 내에 존재하는 특정 launch 파일을 실행해라, 라는 뜻이 됩니다.

예제 launch file을 살펴봅시다. ros2의 launch file은 python과 xml을 지원합니다. python 파일의 경우 **<file-name>.launch.py**로 이름짓는 것이 일반적입니다.

- 항상 launch file의 분석은 가장 하단부터 시작합니다.

```python
return LaunchDescription([
        TimerAction(
            period=3.0,
            actions=[rviz2]
        ),

        robot_state_publisher,
        joint_state_publisher_gui,
    ])
```

**LaunchDescription[]** 안에 위치한 프로그램들을 동시에 실행하는 것이 launch file을 작성하는 이유입니다.

- 상단 옵션을 하나씩 살피겠습니다. launch file에서 특정 폴더, 파일에 접근하기 위해 package에 기반하여 경로를 탐색합니다.

```python
def generate_launch_description():

    pkg_path = os.path.join(get_package_share_directory("src_description"))
    rviz_config_file = os.path.join(pkg_path, "rviz", "description.rviz")
    urdf_file = os.path.join(pkg_path, "urdf", "src_body.urdf")
```

- **urdf - Unified Robotics Description Format**란 로봇의 형상을 파일로 표현하는 일종의 포멧입니다.

```xml
<?xml version="1.0" ?>
<!-- =================================================================================== -->
<!-- |    This document was autogenerated by xacro from /home/kimsooyoung/ros2_ws/src/du2023-ros2/src_gazebo/urdf/src_body.urdf.xacro | -->
<!-- |    EDITING THIS FILE BY HAND IS NOT RECOMMENDED                                 | -->
<!-- =================================================================================== -->
<robot name="src_body">
  <material name="silver">
    <color rgba="0.700 0.700 0.700 1.000"/>
  </material>
  <material name="black">
    <color rgba="1.000 1.000 1.000 1.000"/>
  </material>
  <material name="blue">
    <color rgba="0.0 0.0 0.8 1.0"/>
  </material>
  <material name="green">
    <color rgba="0.0 0.8 0.0 1.0"/>
  </material>
  <material name="grey">
    <color rgba="0.2 0.2 0.2 1.0"/>
  </material>
	...
```

![lec2_8.png](/kr/ros2_basic_foxy/images2/lec2_8.png?height=300px)

- image from : [http://library.isr.ist.utl.pt/](<http://library.isr.ist.utl.pt/docs/roswiki/urdf(2f)XML(2f)Link.html>)

{{% notice note %}}
src의 urdf와 description 결과를 함께 살펴보면서 urdf에 대한 개념을 익혀봅시다.
{{% /notice %}}

> urdf를 생성하는 여러 방법들이 있습니다.

- xacro와 함께 직접 텍스트를 작성하기
- CAD 프로그램을 사용하기
  - [SOLIDWORKS](https://www.solidworks.com/ko)
  - [Fusion 360](https://www.autodesk.co.kr/products/fusion-360/overview)
  - [blender](https://www.blender.org/)
- 시뮬레이션 프로그램 내 model builder 사용하기
  - Gazebo
  - [Colleliasim](https://www.coppeliarobotics.com/)

저의 경우, **Fusion 360**에서 사용할 수 있는 오픈소스 urdf exporter를 통해 시뮬레이션을 개발하였습니다.

[https://github.com/syuntoku14/fusion2urdf](https://github.com/syuntoku14/fusion2urdf)

- urdf를 사용하여 ROS에게 로봇에 대한 정보를 전달하고 지속 주시하는 것이 아래 두 Node의 역할입니다.

```python
joint_state_publisher_gui = Node(
    package="joint_state_publisher_gui",
    executable="joint_state_publisher_gui",
    name="joint_state_publisher_gui",
)

robot_state_publisher = Node(
    package='robot_state_publisher',
    node_executable='robot_state_publisher',
    node_name='robot_state_publisher',
    output='screen',
    parameters=[{'use_sim_time': True}],
    arguments=[urdf_file],
)
```

- launch file 분석의 마지막으로 Rviz2 실행입니다. Rviz와 동일하게 launch file에서 **config option**을 줄 수 있으며, **TimerAction**을 통해 3초의 여유를 갖고 실행하도록 하였습니다.

```python
# Launch RViz
rviz2 = Node(
    package="rviz2",
    executable="rviz2",
    name="rviz2",
    output="screen",
    arguments=["-d", rviz_config_file],
)

return LaunchDescription([
    TimerAction(
        period=3.0,
        actions=[rviz2]
    ),
```

- ROS 2의 launch file 작성법을 간단하게 설명하고 넘어가겠습니다.

```python
robot_state_publisher = Node(
    package='robot_state_publisher',
    executable='robot_state_publisher',
    output='screen',
    parameters=[robot_description]
)
```

- **Node** : Node 하나를 실행시킬 수 있는 옵션입니다.
- **package** : 실행시킬 Node가 포함된 package를 선택해줍니다.
- **executable** : c++ Node의 경우, colcon build를 하면 실행 가능한 프로그램이 생성됩니다. python의 경우도 스크립트를 실행 시키게 되며, 추후 코딩 실습을 거치면 완벽히 이해하실 수 있을 것입니다.
- **parameters** : 실행시킬 Node의 추가 매개변수가 있다면 이 부분에 추가됩니다.

---

> 이렇게 **launch file 하나를 분석해 보았습니다.** 하지만, 제가 보여드린 것은 하나의 예시일 뿐이며, launch file은 여러 형식으로 사용되고 있습니다.

일례로, 자율 주행 시 사용되는 nav2 패키지의 lanuch file은 다음과 같은 형태를 갖습니다.

```python
declare_namespace_cmd = DeclareLaunchArgument(
        'namespace',
        default_value='',
        description='Top-level namespace')
		...
		# Create the launch description and populate
		ld = LaunchDescription()

		# Set environment variables
		ld.add_action(stdout_linebuf_envvar)

		# Declare the launch options
		ld.add_action(declare_namespace_cmd)
		ld.add_action(declare_use_namespace_cmd)
		ld.add_action(declare_slam_cmd)
		ld.add_action(declare_map_yaml_cmd)
		ld.add_action(declare_use_sim_time_cmd)
		ld.add_action(declare_params_file_cmd)
		ld.add_action(declare_autostart_cmd)
		ld.add_action(declare_bt_xml_cmd)

		# Add the actions to launch all of the navigation nodes
		ld.add_action(bringup_cmd_group)

		return ld
```

아직 프로그래밍을 시작하지도 않았기 때문에, launch file에 대해서 모두 이해하려 하지는 않으셔도 됩니다. 남은 강의들에서 계속해서 살펴보겠습니다.
