---
title: "Lecture11. About Gazebo - 1"
date: 2023-04-28T10:51:09+09:00
draft: false
---

## URDF와 Robot Description

로봇에 센서를 부착하고, 외형을 정의하기 위해 가장 기본이 되는 것이 바로 URDF입니다. 이번 시간, 실습을 통해 URDF에 대한 개념과 robot desciption을 실습해 보겠습니다.

- 일반적으로, 로봇은 Links와 Joints 두가지 요소로 이루어집니다.

|       | Description                                                                     |
| ----- | ------------------------------------------------------------------------------- |
| Link  | 단단하게 고정된 강체(rigid-body)이며, 사람의 골격에 해당합니다.                 |
| Joint | link 사이를 결합해주고 이들 사이 운동을 결정짓습니다. 사람의 관절에 해당합니다. |

![Untitled.png](/kr/ros2_foxy/images11/Untitled.png?height=300px)

다양한 종류의 joint들이 존재하지만, 이론적으로 이들은 결국 prismatic + revolute joint의 결합으로 설명될 수 있습니다.

|                 | Description                 |
| --------------- | --------------------------- |
| revolute joint  | 회전 운동을 갖는 joint      |
| prismatic joint | 수평 병진 운동을 갖는 joint |

⇒ ROS 2에서는 개발 상 편의를 위해 크게 6가지의 joint를 사용하고 있습니다만 그림으로만 보고 넘어가겠습니다.

![Untitled1.png](/kr/ros2_foxy/images11/Untitled1.png?height=250px)

- Link와 Joint로 결합된 로봇을 결국 **텍스트로 표현**할 수 있지 않겠냐는 기본 전제 하에, 로봇 공학자들은 **URDF(Unified Robot Description Format)**라는 표준을 만들게 됩니다. 실제로 urdf만 있다면 시뮬레이터 종류에 상관 없이 동일한 로봇 외형을 등장시킬 수 있습니다.

![Untitled2.png](/kr/ros2_foxy/images11/Untitled2.png?height=300px)

image from : [Martin Androvich](https://www.youtube.com/watch?v=cy7ZICqaFVQ)

⇒ URDF는 XML 문법을 사용하고 있으며 다양한 tag를 통해 로봇을 표현하게 됩니다. 예시를 통해 URDF에 대한 이해도를 가져봅시다.

- URDF Link Example

```xml
<link name="base_link">
    <inertial>
        <origin xyz="0 0 0" rpy="0 0 0" />
        <mass value="8.3" />
        <inertia ixx="5.249466E+13" ixy="-1.398065E+12" ixz="-3.158592E+12" iyy="5.786727E+13" iyz="-5.159120E+11" izz="3.114993E+13" />
    </inertial>
    <visual>
        <origin xyz="0 0 0" rpy="0 3.1415 3.1415" />
        <geometry>
            <mesh filename="package://neuronbot2_description/meshes/neuronbot2/base_link.stl" scale="0.001 0.001 0.001" />
        </geometry>
        <material name="black" />
    </visual
    <collision>
        <origin xyz="0 0 0.125" rpy="0 0 0" />
        <geometry>
            <box size="0.25 0.25 0.25" />
        </geometry>
        <material name="black" />
    </collision>
</link>
```

|           | Description                                                                                                                                                                                                                                   |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| inertial  | 해당 link의 질량, 관성 모멘트와 같은 물성치를 포함합니다.                                                                                                                                                                                     |
| visual    | 로봇이 겉으로 보여지는 시각적인 요소를 설정합니다. STL과 같은 3D 모델링 파일을 사용할 수 있습니다.                                                                                                                                            |
| collision | visual은 겉으로 보여지는 모습일 뿐, 실제 해당 link가 자치하는 부피는 collision에서 지정됩니다. Visual과 collision을 일치시킬수록 좋기 때문에 3D 모델링 파일을 사용하기도 합니다. (종종 계산 단순화를 위해 간소화된 모델을 사용하기도 합니다.) |

- URDF JointExample

```xml
<joint name="r_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="wheel_right_link"/>
    <origin xyz="0.0 -0.09 0.0415" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
</joint>
<joint name="l_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="wheel_left_link"/>
    <origin xyz="0.0 0.109 0.0415" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
</joint>
```

|               | Description                                                                                                                              |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| name          | joint의 이름은 후에 tf publish 시 그대로 사용되기 때문에 혼란을 야기하지 않도록 설정해야 합니다.                                         |
| type          | 예시에서 사용중인 joint type은 fixed와 continuous로. fixed는 단단히 결합된 joint를, continuous는 무한히 돌아갈 수 있는 joint를 뜻합니다. |
| origin        | parent link의 원점을 기준으로 한 joint의 위치를 지정하게 되며, 이러한 수치는 모델링 파일을 통해 미리 조사된 이후 URDF로 변환됩니다.      |
| axis          | 회전하는 joint의 경우 어떠한 축을 기준으로 회전되는지 설정이 필요합니다.                                                                 |
| parent, child | 해당 joint의 전, 후 link를 설정합니다.                                                                                                   |
| 기타 속성들   | limit, dynamics, calibration, mimic, safety_controller등이 있으며, 각종 역학적인 속성을 표현합니다.                                      |

{{% notice note %}}
urdf의 joint는 절대 좌표를 기준으로 하는 extrinsic 체계를 갖습니다.
{{% /notice %}}

- 기타 속성들

|          | Description                                                                                                                                                                                                                   |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| origin   | 해당 요소의 원점을 기준으로, 위치와 방향을 결정합니다. 3축 직교 좌표계를 기준으로 x,y,z 축과 roll pitch yaw 회전각을 사용하고 있습니다.                                                                                       |
| geometry | visual과 collision의 기하학적 요소를 결정하는 태그입니다. urdf에서는 box, cylinder, sphere와 같이 단순한 도형을 제공하고 있습니다. 별도 stl 파일을 사용해도 되지만 로봇을 단순화하고 싶은 경우 이를 통해 간소화가 가능합니다. |
| material | color, texture등을 지정할 수 있으며, 외향적인 디자인을 위한 요소입니다.                                                                                                                                                       |

## Basic Stick

> URDF를 설명하긴 했는데… 실제로 와닿지 않지요? 이러한 이유로 2가지 예시를 통해 URDF에 대한 개요를 확실히 짚고 넘어가보겠습니다.

1. Basic Stick
2. Fusion Bot

![Untitled3.png](/kr/ros2_foxy/images11/Untitled3.png?height=300px)

### basic stick

- 2개의 link로 구성되어 있는 아주 기본적인 모델을 함께 분석해볼 것입니다. launch file을 실행해봅시다.

```python
cbp basic_stick && source install/local_setup.bash

ros2 launch basic_stick description.launch.py
```

![Untitled4.png](/kr/ros2_foxy/images11/Untitled4.png?height=350px)

{{% notice note %}}
전에 배웠던 tf2 기억하시죠?? 까먹으면 안됩니다…
{{% /notice %}}

- lauch file 분석

⇒ launch file은 항상 최하단부터 분석합니다. 지금 2개의 node가 실행되고 있습니다.

```python
return LaunchDescription([
        robot_state_publisher,
        rviz,
    ])
```

- robot_state_publisher node는 xacro file을 받아 일련의 처리 과정을 거친 결과를 매개변수로 받습니다.

```python
# Get URDF via xacro
robot_description_content = Command(
    [
        PathJoinSubstitution([FindExecutable(name="xacro")]),
        " ",
        PathJoinSubstitution(
            [FindPackageShare("basic_stick"), "urdf", "basic_stick_desc.xacro"]
        ),
    ]
)
robot_description = {"robot_description": robot_description_content}

# Robot State Publisher
robot_state_publisher = Node(
    package='robot_state_publisher',
    executable='robot_state_publisher',
    name='robot_state_publisher',
    output='screen',
    parameters=[robot_description],
)
```

- rviz2 launch는 간단하지만 복습을 위해 다시 살펴봅시다.

```python
pkg_path = os.path.join(get_package_share_directory('basic_stick'))
rviz_config_file = os.path.join(pkg_path, 'rviz', 'desc.rviz')

# Launch RViz
rviz = Node(
    package='rviz2',
    executable='rviz2',
    name='rviz2',
    output='screen',
    arguments=["-d", rviz_config_file],
)
```

### robot state publisher

- robot state publisher는 모든 link와 joint 값을 지속적으로 받아 전체 로봇의 구조를 tf2 형식으로 변환합니다.

![Untitled5.png](/kr/ros2_foxy/images11/Untitled5.png?height=300px)

{{% notice note %}}
tf2 예시에서 static/dynamic tf2에 대해서 살펴보았지요? 지금 basic stick은 움직이는 부분이 없기 때문에 모든 파트가 static이라는 점을 안내드립니다.

{{% /notice %}}

- robot state publisher는 URDF file의 joint/link 데이터를 기반으로 tf2 broadcast를 합니다. 때문에 tf2 tree를 살펴보아도 아래와 같은 그림을 볼 수 있습니다.

![Untitled6.png](/kr/ros2_foxy/images11/Untitled6.png?height=300px)

- robot state publisher가 도출하는 또 다른 것은 robot_description topic입니다. string만 가득한 topic이지만 이를 사용하여 시각화, Gazebo spawn 등 여러 작업에서 사용됩니다.

```python
$ ros2 topic info /robot_description
Type: std_msgs/msg/String
Publisher count: 1
Subscription count: 0

data: <?xml version="1.0" ?> <!-- =================================================================================== --> <!-- |    Th...
---
```

### basic_stick_desc.xacro

> URDF 스크립트를 사람이 모두 작성하기는 매우 비효율적입니다. 더불어, Gazebo에서만 사용하는 속성을 따로 분리하고 싶은 경우, 파일을 나누어 관리하고 싶을 것입니다. 이러한 욕구를 충족시키기 위해서 ROS 2는 URDF의 작성을 보다 편하게 해주는 **XML Macro, Xacro**를 지원하고 있습니다.

특히 xacro는 수식, 조건을 사용 가능하기 때문에 로봇 파일을 다루기 매우 용이하며, 특정 요소를 모듈화 후 재사용하는 등 효율적인 URDF 작성이 가능하도록 도와줍니다.

- basic_stick_desc.xacro의 초반부를 보면 xacro 변수가 설정되고 있는 것을 확인할 수 있습니다.

```xml
<?xml version="1.0"?>
<robot name="basic_stick" xmlns:xacro="http://www.ros.org/wiki/xacro">
  <xacro:property name="PI" value="3.14159"/>
  <xacro:property name="mass1" value="10" />
  <xacro:property name="mass2" value="1" />
  <xacro:property name="width1" value="0.1" /> <!--link_1 radius-->
  <xacro:property name="width2" value="0.1" /> <!--link_2 radius-->
  <xacro:property name="length0" value="0.1" /> <!--link_1 length-->
  <xacro:property name="length1" value="1.5" /> <!--link_2 length-->
  <xacro:property name="length2" value="0.1" /> <!--link_3 length-->
  <xacro:property name="mass_camera" value="0.2" />
```

- 본론으로 돌아와 현재 basic stick은 link 3개 joint 2개를 갖는, 모든 joint가 fixed joint인 로봇입니다.

```xml
  <!--Links-->
  <link name="world"/>

  <link name="link_1">

  <link name="link_2">
```

- 각 링크의 3가지 속성을 통해 물성치와 시각적 특성을 구현하고, Gazebo에서 이를 바탕으로 우리에게 시각화해주게 되는 원리입니다.

```xml
<link name="link_1">
  <visual>
  <collision>
  <inertial>
</link>
```

- joint에서는 항상 parent와 child의 연결이 완성되어 있어야 합니다. xacro property를 사용한 모습도 확인 가능합니다.

```xml
<!--Joints-->
<joint name="fixed_base_joint" type="fixed">
  <parent link="world"/>
  <child link="link_1"/>
  <origin xyz="0 0 ${length1/2}" rpy="0 0 0"/>
</joint>

<joint name="pan_joint" type="fixed">
  <parent link="link_1"/>
  <child link="link_2"/>
  <!-- <axis xyz="0 0 1"/> -->
  <origin xyz="0 0 ${length1/2+length2/2}" rpy="0 0 -${pi/2}"/>
</joint>
```

- 이렇게 basic stick을 완성해보았습니다. 그럼, Gazebo에서 이를 등장시켜봅시다.

![Untitled7.png](/kr/ros2_foxy/images11/Untitled7.png?height=300px)

## Gazebo launch

- 예시를 실행합니다.

```xml
ros2 launch basic_stick gz.launch.py
```

![Untitled8.png](/kr/ros2_foxy/images11/Untitled8.png?height=300px)

⇒ 색도 없고, 밋밋한 basic stick이 동장할 것입니다.

- 이 launch file을 분석해 보면서 gazebo 연동의 개념을 잡아봅시다.

```python
return LaunchDescription(
    [
        start_gazebo_server_cmd,
        start_gazebo_client_cmd,
        robot_state_publisher,
        spawn_entity,
    ]
)
```

- start_gazebo_server_cmd / start_gazebo_client_cmd는 각각 gzserver와 client를 실행하는 부분입니다. 더불어, world file을 gzserver에게 전달할 수 있습니다.

```python
def generate_launch_description():

    pkg_path = os.path.join(get_package_share_directory('basic_stick'))

    pkg_gazebo_ros = FindPackageShare(package='gazebo_ros').find('gazebo_ros')
    world_path = os.path.join(pkg_path, 'worlds', 'empty_world.world')

    # Start Gazebo server
    start_gazebo_server_cmd = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(os.path.join(pkg_gazebo_ros, 'launch', 'gzserver.launch.py')),
        launch_arguments={'world': world_path}.items()
    )

    # Start Gazebo client
    start_gazebo_client_cmd = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(os.path.join(pkg_gazebo_ros, 'launch', 'gzclient.launch.py'))
    )
```

- 메인 파트인 spawn 부분입니다. spawn node는 robot state publisher의 output인 robot_description topic을 subscribe하여 gazebo 상에 물체를 등장시킵니다.

⇒ 등장시킬 시의 이름, 위치와 방향을 지정할 수 있습니다.

```python
    # Spawn Robot
    spawn_entity = Node(
        package='gazebo_ros',
        executable='spawn_entity.py',
        arguments=[
            '-topic', 'robot_description',
            '-entity', 'sensor_stick',
            '-x', str(0),
            '-y', str(0.0),
            '-Y', str(0.0),
        ],
        output='screen'
    )
```

- 방금 전, Gazebo 상에서 sensor stick이 밋밋했지요? 색상을 추가하기 위해서 다른 xacro file을 include 합시다.

⇒ basic_stick.xacro에서 basic_stick.gazebo.xacro를 include 합니다.

```xml
<?xml version="1.0"?>
<robot>
  <!--base_link-->
  <gazebo reference="base_link">
    <material>Gazebo/White</material>
  </gazebo>
  ...

</robot>
```

```xml
<!--Import gazebo elements-->
<xacro:include filename="$(find basic_stick)/urdf/basic_stick.gazebo.xacro" />
```

- 색상이 반영된 새로운 xacro file을 다시 gazebo spawn 시켜봅시다.

```python
robot_description_content = Command(
    [
        PathJoinSubstitution([FindExecutable(name="xacro")]),
        " ",
        PathJoinSubstitution(
            [FindPackageShare("basic_stick"), "urdf", "basic_stick.xacro"]
        ),
    ]
)
```

![Untitled9.png](/kr/ros2_foxy/images11/Untitled9.png?height=300px)

Basic Stick을 통해 배운 내용들을 복습하자면

- ROS 2에게 로봇의 정보를 전달하기 위해 urdf라는 포멧을 사용했고, 이는 link와 joint로 이루어집니다.
- robot state publisher는 URDF를 받아 tf2를 broadcast하며 robot_description publish를 합니다.
- robot_description topic을 통해 gazebo에서 로봇을 등장시킬 수 있었습니다.

### fusion bot

> 일전 sensor stick은 움직이는 파츠가 없이 정적인 모델이었습니다. 그래서 이번에는 움직이는 joint를 갖는 모바일 로봇의 예시를 살펴보겠습니다.

![Untitled10.png](/kr/ros2_foxy/images11/Untitled10.png?height=300px)

- 예시를 실행해봅시다. rviz2에 등장하는 joint들을 볼 수 있습니다.

```python
cbp fusionbot_description && source install/local_setup.bash

ros2 launch fusionbot_description description.launch.py
```

![Untitled11.png](/kr/ros2_foxy/images11/Untitled11.png?height=300px)

- tf2 tree도 확인해봅시다.

```bash
$ ros2 run tf2_tools view_frames.py
```

![Untitled12.png](/kr/ros2_foxy/images11/Untitled12.png?height=300px)

- 그리고 함께 등장하는 joint_state_publisher_gui의 슬라이드바를 움직이면, rviz2 상의 tf가 변화하는 것을 볼 수 있습니다.

![Untitled13.png](/kr/ros2_foxy/images11/Untitled13.png?height=400px)

- fusionbot_description.urdf을 보면 알 수 있듯이, 이번 로봇은 continuous joint를 갖습니다. 이는 바퀴와 같이 끝없이 돌아갈 수 있는 joint입니다.

```xml
<?xml version="1.0"?>
<robot name="fusionbot" xmlns:xacro="http://www.ros.org/wiki/xacro">

  <xacro:include filename="$(find fusionbot_description)/urdf/materials.xacro" />
  <!-- <xacro:include filename="$(find fusionbot_description)/urdf/fusionbot.gazebo" /> -->

  <link name='base_footprint' />
  <link name="base_link">
  <link name="right_wheel">
  ...

  <joint name='base_link_joint' type='fixed'>
  <joint name="right_wheel_joint" type="continuous">
  <joint name="left_wheel_joint" type="continuous">
  ...

</robot>
```

### **joint state publisher**

> joint state publisher는 로봇 내 존재하는 다양한 joint 값들을 실시간으로 갱신하여 **/joint_states**라는 topic으로 publish 하고 tf2 broadcast도 담당합니다.

- /joint_states topic은 [sensor_msgs/msg/JointState](http://docs.ros.org/en/noetic/api/sensor_msgs/html/msg/JointState.html) msg를 사용하며, 각 joint들의 이름, 현재 위치, 속도와 힘을 배열 형태로 담게 됩니다.

```bash
$ ros2 interface show sensor_msgs/msg/JointState
# This is a message that holds data to describe the state of a set of torque controlled joints.
#
#
# The state of each joint (revolute or prismatic) is defined by:
#  * the position of the joint (rad or m),
#  * the velocity of the joint (rad/s or m/s) and
#  * the effort that is applied in the joint (Nm or N).
#
...

std_msgs/Header header

string[] name
float64[] position
float64[] velocity
float64[] effort
```

- 예제 실행 시 등장했던 조그만 창은 joint_state_publisher_gui라고 불리며, 로봇 내 조작이 가능한 joint들을 간단하게 제어할 수 있도록 해주는 작은 프로그램입니다. 이를 통해 구성한 URDF의 방향은 알맞게 설정되었는지, 원점은 잘 맞는지 등을 확인 가능합니다.

![Untitled14.png](/kr/ros2_foxy/images11/Untitled14.png?height=300px)

- topic echo를 걸어둔 상태에서 gui의 슬라이드바를 움직여봅시다.

```xml
$ ros2 topic echo /joint_states
header:
  stamp:
frame_id: ''
name:
- right_wheel_joint
- left_wheel_joint
position:
- 1.918884792812646
- -1.409318464400381
velocity: []
effort: []
---
```

![Untitled14.png](/kr/ros2_foxy/images11/Untitled14.png?height=300px)

⇒ 이렇게 joint state publisher는 현재 로봇이 가진 움직일 수 있는 모든 joint들을 예의주시하고 있습니다.

- rviz2 상에서 보시다시피 tf2 frame도 회전하기 때문에, 이는 로봇팔을 포함 움직이는 모든 로봇에게 필수적인 데이터입니다.

![Untitled15.png](/kr/ros2_foxy/images11/Untitled15.png?height=300px)

- 그렇다면 robot state publisher, joint state publisher, tf2는 어떠한 관계를 가질까요? rqt_graph를 통해 살펴봅시다.

![Untitled16.png](/kr/ros2_foxy/images11/Untitled16.png?height=300px)

### FusionBot Gazebo

- fusionbot의 gazebo 예시도 실행해봅시다.

```xml
ros2 launch fusionbot_description gz.launch.py
```

![Untitled17.png](/kr/ros2_foxy/images11/Untitled17.png?height=300px)

{{% notice note %}}
현재 로봇이 움직일 수는 없습니다. 이는 다음 시간 plugin을 사용하여 구현해보겠습니다.
{{% /notice %}}

- 이번 launch file의 하단을 보면서 이번 시간 배운 내용들을 다시금 정리해봅시다.

```python
  return LaunchDescription(
        [
            start_gazebo_server_cmd,
            start_gazebo_client_cmd,
            robot_state_publisher,
            joint_state_publisher,
            spawn_entity,
        ]
    )
```

지금까지 배운 내용을 총정리해봅시다.

- ROS 2와 소통하기 위해 urdf를 사용했고,이는 link와 joint로 이루어집니다.
- joint는 joint state publisher가 담당하며,
- robot state publisher는 joint state와 함께 link를 결합하여 최종 tf를 구성합니다.
- 이러한 데이터들에 기반하여 gazebo는 시뮬레이션을 합니다.

## Gazebo Basic

> Gazebo의 특징과 기본적인 UI, 그리고 사용법을 짚고 넘어가고자 합니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/gazebo_logo.png?height=200px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/gazebo_logo.png?height=200px)

- image from : [Open Robotics](https://www.openrobotics.org/)

Gazebo는 로봇공학을 위해 제작된 전용 물리 엔진 기반의 높은 3D 시뮬레이터로 ROS를 관리하는 Open Robotics에서 비롯된 시뮬레이터인 만큼 ROS와 높은 호환성을 자랑합니다. 이후 Gazebo 사용 및 오류 발생 시 디버깅을 위해, Gazebo를 구성하는 요소들을 간단하게 짚고 넘어가겠습니다.

### gzserver & gzclient

> Gazebo는 Socket-Based Communication을 갖습니다. 따라서 서버와 Client를 분리하여 실행 가능하며 다른 기기에서의 실행 후 연동도 가능합니다.

#### Gazebo Server

- gzserver는 Gazebo 동작의 대부분을 수행합니다. 시뮬레이션하려는 장면과 그 안에 있는 개체 파일을 분석하고, 그런 다음 물리 엔진과 센서 엔진을 사용하여 전체 장면을 시뮬레이션합니다.
  ⇒ 터미널에서 gzserver를 독립적으로 실행할 수 있습니다.

```python
$ gzserver
```

#### Gazebo Client

- **gzclient**는 gzserver에 연결하여 대화형 도구와 함께 시뮬레이션을 렌더링하는 Graphic Client를 제공합니다.

```python
$ gzserver
```

⇒ client만 실행하면, 연결되고 명령을 수신할 서버가 없기 때문에 (컴퓨팅 리소스를 소비하는 것을 제외하고) 아무 것도 하지 않습니다.

- gazebo 명령어를 입력하면 내부적으로 server와 client가 순차적으로 실행됩니다.

```bash

# Terminal1 - gazebo 실행
$ gazebo

# Terminal2 - gazebo process check
$ ps faux | grep gz
```

⇒ 이러한 이유로 Gazebo를 종료했다고 생각하지만 gzserver가 깔끔하게 종료되지 않아 정상 동작하던 예시에서 오류를 얻는 상황이 발생합니다.

### World File and Model File

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/world.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/world.png?height=300px)

- image from : [dronecode](https://dev.px4.io/v1.11_noredirect/en/simulation/gazebo_worlds.html)

#### **World Files**

⇒ Gazebo world file에는 로봇 모델, 환경, 조명, 센서, 다른 기타 물체들까지 시뮬레이션 환경의 모든 요소가 포함되어 있으며, 이는 일반적으로 확장자명 **.world**를 사용합니다. 아래 예시와 같이 Gazebo 실행 시 world 파일을 옵션으로 하여 단독 실행이 가능합니다.

```bash
$ gazebo <yourworld>.world
```

#### **Model Files**

⇒ Gazebo의 World는 다양한 Model들로 구성됩니다. 하지만 Model 파일만으로 gazebo를 실행시킬 수는 없습니다.

⇒ 모델을 별도의 파일로 보관하는 이유는 다른 프로젝트에 재사용하기 위해서이며, 로봇의 모델 파일 또는 다른 모델을 월드 파일 내에 포함하려면 다음과 같이 태그를 통해 import 할 수 있습니다.

```xml
<include><uri>model://model_file_name</uri></include>
```

#### **Plugins**

Gazebo의 World, Model에 장착되는 각종 센서와 제어를 위한 다양한 플러그인이 준비되어 있으며, 이러한 플러그인은 커멘드 라인에서 로드하거나 SDF 파일 내부에 추가할 수 있습니다. (자체 Plugin을 개발할 수도 있습니다.)

### **Understanding Gazebo GUI**

> 이번에는 Gazebo의 기본 조작 방법과 내장 Tool들을 살펴보겠습니다.

1. Gazebo의 시점 변환은 마우스를 사용하여 손쉽게 조작 가능합니다.
2. 마우스 왼쪽 버튼을 누르고 드래그하면 시점을 이동할 수 있습니다.
3. 마우스 휠을 누른 상태로 이동시키면 회전 시점을 조작할 수 있습니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/ui.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/ui.png?height=300px)

- 내장 tool들을 살펴보겠습니다. 우선, 최상단에 위치한 **Top Toolbar**부터 살펴보겠습니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/toolbar.png?height=50px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/toolbar.png?height=50px)

⇒ 왼쪽부터 오른쪽 순서대로 각 아이콘 별 기능을 설명해보겠습니다.

- **Select mode**

Select mode는 가장 일반적으로 사용되는 커서 모드입니다. 장면을 탐색할 수 있습니다.

- **Translate mode**

커서 모드를 선택한 다음 이동시키길 원하는 객체를 클릭합니다. 이후 등장하는 3축 중 적절한 축을 사용하여 개체를 원하는 위치로 끌기만 하면 됩니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new1.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new1.png?height=300px)

- **Rotate mode**

translate mode와 마찬가지로 이 커서 모드를 사용하면 주어진 모델의 방향을 변경할 수 있습니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new2.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new2.png?height=300px)

- **Scale mode**

Scale mode를 사용하면 객체의 전체 크기를 변경할 수 있습니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new3.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new3.png?height=300px)

- **Undo/Redo**

앞,뒤로 되돌리는 기능입니다.

- **Simple shapes**

큐브, 구체 또는 실린더와 같은 기본 3D 모델을 환경에 삽입할 수 있습니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new4.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new4.png?height=300px)

- **Lights**

스포트라이트, 포인트 라이트 또는 방향이 정해진 조명과 같은 다양한 광원을 환경에 추가합니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new5.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new5.png?height=300px)

- **Copy/Paste**

모델을 복사/붙여넣을 수 있습니다. (Ctrl+C/V를 통해서도 가능합니다.)

- **Align**

이 도구를 사용하면 x y z 축 중 하나를 따라 한 모형을 다른 모델과 정렬할 수 있습니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new6.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new6.png?height=300px)

⇒ 또는 두 모델을 특정한 면 기반으로 서로 붙이는 것도 가능합니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new7.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new7.png?height=300px)

- **Change view**

> 상단 뷰, 측면 뷰, 전면 뷰, 하단 뷰와 같은 다양한 관점에서 장면을 볼 수 있습니다.

다음으로 **Side Panel**을 살펴보겠습니다.

- **World**

현재 사용중인 조명 및 모델들이 표시됩니다. 개별 모델을 클릭하여 위치 및 방향과 같은 모델의 기본 파라미터를 보거나 편집할 수 있습니다. 또한 물리 옵션을 통해 중력 및 자기장과 같은 물성치도 변경할 수 있습니다. GUI 옵션을 사용하면 기본 카메라 뷰 각도 및 포즈에 액세스할 수 있습니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new8.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new8.png?height=300px)

- **Insert**

추가할 모델을 찾을 수 있습니다. 환경 변수에 지정된 폴더에서 모델들을 검색한 뒤 배치하는 것이 가능하며, 환경 변수 설정 없이도 Add Path 옵션을 통해 정해진 포맷을 갖는 모델을 가져올 수 있습니다.

![https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new9.png?height=300px](https://rb2023ros.github.io/kr/ros_basic_noetic/images3/new9.png?height=350px)

### Gazebo and ROS 2

> Gazebo는 ROS 2와 호환이 가장 좋은 로봇 시뮬레이션 프로그램입니다. gazebo_ros에 대해서 살펴봅시다.

- gazebo_ros 실행

```xml
$ ros2 launch gazebo_ros gazebo.launch.py
```

![Untitled18.png](/kr/ros2_foxy/images11/Untitled18.png?height=300px)

{{% notice note %}}
겉보기에는 일반 gazebo 실행과 차이가 없는 것 같지만, 내부적으로 이는 큰 차이를 갖습니다. => 바로 ROS 2와 호환할 수 있는 다양한 API를 포함한 Gazebo가 실행된다는 점입니다.

{{% /notice %}}

```xml
$ ros2 topic list
/clock
/parameter_events
/performance_metrics
/rosout

$ ros2 service list
/apply_joint_effort
/apply_link_wrench
/clear_joint_efforts
/clear_link_wrenches
/delete_entity
/gazebo/describe_parameters
/gazebo/get_parameter_types
/gazebo/get_parameters
/gazebo/list_parameters
/gazebo/set_parameters
/gazebo/set_parameters_atomically
/get_model_list
/pause_physics
/reset_simulation
/reset_world
/spawn_entity
/unpause_physics
```

- 지금까지 실행했던 launch file을 살펴보면 모두 gazebo_ros를 실행함을 알 수 있으며, 특히 server와 client를 나누어 실행하고 있음을 확인 가능합니다.

```python
# Start Gazebo server
start_gazebo_server_cmd = IncludeLaunchDescription(
    PythonLaunchDescriptionSource(os.path.join(pkg_gazebo_ros, 'launch', 'gzserver.launch.py')),
    launch_arguments={'world': world_path}.items()
)

# Start Gazebo client
start_gazebo_client_cmd = IncludeLaunchDescription(
    PythonLaunchDescriptionSource(os.path.join(pkg_gazebo_ros, 'launch', 'gzclient.launch.py'))
)
```

⇒ server와 client를 나눈 이유는 server단에서 world 파일이 전달되기 때문입니다.

- gazebo_ros를 실행한 뒤 조회되었던 topic과 service들 중 의미있는 것들을 다시 한 번 살펴봅시다. (우리에게 익숙한 것도 있습니다.)

| Topic Name | Description                                                       |
| ---------- | ----------------------------------------------------------------- |
| /clock     | Gazebo 자체의 시간으로 시뮬레이션이 시작되는 시점이 0초가 됩니다. |

---

| Service Name      | Description                                 |
| ----------------- | ------------------------------------------- |
| /delete_entity    | 모델을 등장시킵니다.                        |
| /spawn_entity     | 존재하는 모델을 제거합니다.                 |
| /get_model_list   | 전체 모델 리스트를 조회합니다.              |
| /pause_physics    | 중력, 바람 등 물리량들을 모두 정지시킵니다. |
| /unpause_physics  | 물리량들을 다시 시작합니다.                 |
| /reset_simulation | World 전체를 Reset                          |
| /reset_world      | World내 Model Pose를 Reset                  |

### Gazebo Building Editor

- gazebo를 실행시킨 뒤, 상단 Edit 옵션에서 Building Editor를 실행합니다.

![gz_building_editor.gif](/kr/ros2_foxy/images11/gz_building_editor.gif?height=300px)

- 상단 모눈종이에 스케치를 통해 건물 벽을 생성할 수 있으며, 하단 view를 통해 실시간으로 업데이트되는 건물을 확인할 수 있습니다. 혹여 실수를 했거나, 보다 정확한 수치를 입력하고 싶은 경우, 모눈종이 위의 검은 선을 더블 클릭하면 사진과 같이 구체적인 설정을 변경할 수 있는 탭이 등장합니다.

![Untitled19.png](/kr/ros2_foxy/images11/Untitled19.png?height=300px)

- 수정이 완료되었다면, 왼쪽 탭에서 door, stairs도 사용해보고, Texture도 입혀봅시다.

![Untitled20.png](/kr/ros2_foxy/images11/Untitled20.png?height=300px)

- 모든 작업을 마친 뒤, File 탭에서 Save As를 선택하여 완성한 물체를 저장합니다. (저는 wall 이라는 이름으로 저장해보겠습니다.)

![gz_save_model.gif](/kr/ros2_foxy/images11/gz_save_model.gif?height=400px)

- 저장된 폴더 내부는 다음과 같은 구조를 갖게 됩니다.

```python
├─  wall
   ├── model.config
   └── model.sdf
```

|              | Description                                                |
| ------------ | ---------------------------------------------------------- |
| model.config | 해당 model의 이름, 작성자 등 기본적인 정보들이 기입됩니다. |
| model.sdf    | 실직적인 sdf 형식의 모델이 위치합니다.                     |

이렇게 생성한 물체를 Gazebo에서 두고두고 사용하는 몇가지 방법들이 있습니다.

- GAZEBO_MODEL_PATH에 추가하기 ⇒ **~/.gazebo/gui.ini** 파일 내 model_paths에 building 폴더에 절대 경로를 추가합니다.

```python
$ gedit ~/.gazebo/gui.ini

[geometry]
x=0
y=0
[model_paths]
filenames=/home/kimsooyoung/<your-folder-location>
```

- launch file에서 GAZEBO_MODEL_PATH 수정하기

{{% notice note %}}
대신 이 방법을 사용하면, 해당 launch file을 사용해야 GAZEBO_MODEL_PATH가 반영됩니다.

{{% /notice %}}

```python
if 'GAZEBO_MODEL_PATH' in os.environ:
    os.environ['GAZEBO_MODEL_PATH'] += ":" + gazebo_model_path
else:
    os.environ['GAZEBO_MODEL_PATH'] = gazebo_model_path
```

- 위 방법을 적용한 뒤 다시 Gazebo를 실행시키면, 사진과 같이 Insert 부분에 building이 추가되어있음을 확인 가능합니다. 이를 클릭 후 gazebo 환경으로 커서를 옮기면 원하는 위치에 wall을 위치시킬 수 있습니다.

![Untitled21.png](/kr/ros2_foxy/images11/Untitled21.png?height=300px)

---

### 3D Gems

![Untitled22.png](/kr/ros2_foxy/images11/Untitled22.png?height=300px)

이번 시간에는 오픈소스 데이터셋 3DGEMS를 사용하여 나만의 World를 꾸미는 예시를 진행해보겠습니다. => 📁 [3DGEMS.zip](https://drive.google.com/file/d/14P83HSgLmN5iC4Yx6TMl7XvbIKuFr7as/view?usp=sharing)

- 제공되는 3DGEMS 폴더를 압축해제한 뒤, **~/.gazebo/models** 폴더에 위치시킵니다.

{{% notice note %}}
WSL2를 사용하시는 분들께서는 터미널에서 **explorer.exe .** 를 입력하면 윈도우 파일 탐색기를 실행 가능합니다.
{{% /notice %}}

- 다시 한 번 Gazebo를 실행시켜 보면 Insert 탭에 추가된 새로운 모델들을 확인할 수 있습니다.

{{% notice tip %}}
잠시 시간을 갖고 나만의 building과 model을 추가하여 나만의 world를 만들어 봅시다!
{{% /notice %}}

- 만든 모델은 launch 실습을 위해 gz_ros2_examples ⇒ my_world ⇒ worlds 폴더에 저장해둡니다. **File ⇒ Save World As**를 클릭하여 완성된 world 파일을 저장합시다.

![Untitled23.png](/kr/ros2_foxy/images11/Untitled23.png?height=300px)

- 여러분들이 만든 world 파일과 기본 empty world를 비교해보세요.

```xml
<sdf version='1.7'>
  <world name='default'>

    <include>
      <uri>model://ground_plane</uri>
    </include>

    <include>
      <uri>model://sun</uri>
    </include>

  </world>
</sdf>
```

⇒ world 파일은 SDF라는 포멧을 사용하여 작성되었으며, 이는 Gazebo에서 쓰이는 독특한 문법입니다.

- 여러분이 만든 launch file을 gazebo_ros를 통해 실행해보세요.

```python
pkg_path = os.path.join(get_package_share_directory('my_world'))
# world_path = os.path.join(pkg_path, 'worlds', <your-world-file>)

# Start Gazebo server
start_gazebo_server_cmd = IncludeLaunchDescription(
    PythonLaunchDescriptionSource(os.path.join(pkg_gazebo_ros, 'launch', 'gzserver.launch.py')),
    launch_arguments={'world': world_path}.items()
)

# Start Gazebo client
start_gazebo_client_cmd = IncludeLaunchDescription(
    PythonLaunchDescriptionSource(os.path.join(pkg_gazebo_ros, 'launch', 'gzclient.launch.py'))
)
```

- 새로운 파일이 추가되었으니, package를 빌드하고, launch file을 다시 실행해봅시다.

1. my_world package에 worlds 폴더 추가
2. worlds 폴더 안에 여러분의 world 파일을 위치시킵니다.
3. 새로운 folder가 추가되었으니 CMakeLists.txt를 수정하고 빌드합니다.
4. launch file 수정 (직접 작성하셔도 좋고, 제가 제공드리는 template을 사용하셔도 됩니다.)
5. ros2 launch를 통해 gazebo_ros에서 여러분의 world를 불러와봅시다!

```python
colcon build --packages-select src_gazebo
source install/local_setup.bash

ros2 launch my_world template.py
```

> world 제작 시 사용된 SDF 포멧은 Blender라는 프로그램에서 export 할 수 있어 많은 사람들이 사용하고 있습니다. 우리는 이제 world 파일을 볼 수 있으니 Github에 있는 수많은 예시들을 가져다 쓸 수 있게 된 것입니다!
