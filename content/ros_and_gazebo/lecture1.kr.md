---
title: "Lecture1 - Gazebo and Robot"
date: 2022-01-19T04:41:42+09:00
draft: false
---

## FusionBot 소개

> 이번 실습을 위해 모바일 로봇에서 가장 많이 사용되는 타입의 로봇, FusionBot을 준비하였습니다. FusionBot을 통해 ROS에서의 로봇 표현 방법을 익힌 뒤, 실제 CAD 파일에서 Gazebo 상의 로봇을 구현하는 실습을 진행해봅시다.

![fb0.png](/kr/ros_and_gazebo/images1/fb0.png?height=400px)

- 실습을 위한 git repo를 clone한 뒤 종속성들을 설치합시다.

```bash
cd ~/ros2_ws/src
git clone https://github.com/RB2023ROS/du2023-gz.git

cd du2023-gz
./setup_scripts.sh
```

### URDF와 Robot Description

> 일반적으로, 로봇은 Links와 Joints 두가지 요소로 이루어집니다.

- Link : 단단하게 고정된 강체(rigid-body)이며, 사람의 골격에 해당합니다.
- Joint : link 사이를 결합해주고 이들 사이 운동을 결정짓습니다. 사람의 관절에 해당합니다.

![fb1.png](/kr/ros_and_gazebo/images1/fb1.png?height=400px)

다양한 종류의 joint들이 존재하지만, 이론적으로 이들은 결국 prismatic + revolute joint의 결합으로 설명될 수 있습니다.

- revolute joint : **회전** 운동을 갖는 joint
- prismatic joint : **수평** 병진 운동을 갖는 joint

그리고 ROS에서는 개발 상 편의를 위해 크게 6가지의 joint를 사용하고 있지요.

![fb2.png](/kr/ros_and_gazebo/images1/fb2.png?height=300px)

- **revolute** - 각도 제한을 가진 회전
- **continuous** - 각도 제한이 없는 회전
- **prismatic** - 수평 운동
- **fixed** - 고정 강체
- **floating** - 6 DOF joint로 gazebo에선 사용 불가
- **Planar** – 평면을 미끄러지는 운동 (물리적 구성이 쉽지 않아 거의 사용되지 않음)

> Link와 Joint로 결합된 로봇을 결국 **텍스트로 표현**할 수 있지 않겠냐는 기본 전제 하에, 로봇 공학자들은 **URDF - Unified Robot Description Format**라는 표준을 만들게 됩니다. 실제로 urdf만 있다면 시뮬레이터 종류에 상관 없이 동일한 로봇 외형을 등장시킬 수 있게 됩니다.

![fb3.png](/kr/ros_and_gazebo/images1/fb3.png?height=400px)

- image from : [Martin Androvich](https://www.youtube.com/watch?v=cy7ZICqaFVQ)

> URDF는 XML 문법을 사용하고 있으며 ROS 1의 launch file과 같이 다양한 tag를 통해 로봇을 표현하게 됩니다. 예시를 통해 URDF에 대한 이해도를 가져봅시다.

URDF의 link가 가질 수 있는 속성들은 다음과 같습니다.

- **inertial** : 해당 link의 질량, 관성 모멘트와 같은 물성치를 포함합니다.
- **visual** : 로봇이 겉으로 보여지는 시각적인 요소를 설정합니다. STL과 같은 3D 모델링 파일을 사용할 수 있습니다.
- **collision** : visual은 겉으로 보여지는 모습일 뿐, 실제 해당 link가 자치하는 부피는 collision에서 지정됩니다. Visual과 collision을 일치시킬수록 좋기 때문에 3D 모델링 파일을 사용하기도 합니다. 종종 계산 단순화를 위해 간소화된 모델을 사용하기도 합니다.

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
</visual>
<collision>
<origin xyz="0 0 0.125" rpy="0 0 0" />
<geometry>
<box size="0.25 0.25 0.25" />
</geometry>
<material name="black" />
</collision>
</link>
```

URDF의 Joint가 가질 수 있는 속성들은 다음과 같습니다.

- **name** : joint의 이름은 후에 tf publish 시 그대로 사용되기 때문에 혼란을 야기하지 않도록 설정해야 합니다.
- **type** : 예시에서 사용중인 joint type은 fixed와 continuous로. fixed는 단단히 결합된 joint를, continuous는 무한히 돌아갈 수 있는 joint를 뜻합니다.
- **origin** : parent link의 원점을 기준으로 한 joint의 위치를 지정하게 되며, 이러한 수치는 모델링 파일을 통해 미리 조사된 이후 URDF로 변환됩니다.
- **axis** : 회전하는 joint의 경우 어떠한 축을 기준으로 회전되는지 설정이 필요합니다.
- **parent, child** : 해당 joint의 전, 후 link를 설정합니다.
- **기타 속성들** (limit, dynamics, calibration, mimic, safety_controller …) : 각종 역학적인 속성을 표현합니다.

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

{{% notice note %}}
urdf의 joint는 절대 좌표를 기준으로 하는 extrinsic 체계를 갖습니다.
{{% /notice %}}

![fb4.png](/kr/ros_and_gazebo/images1/fb4.png?height=200px)

기타 속성들

- **origin** : 해당 요소의 원점을 기준으로, 위치와 방향을 결정합니다. 3축 직교 좌표계를 기준으로 x,y,z 축과 roll pitch yaw 회전각을 사용하고 있습니다.
- **geometry** : visual과 collision의 기하학적 요소를 결정하는 태그입니다. urdf에서는 box, cylinder, sphere와 같이 단순한 도형을 제공하고 있습니다. 별도 stl 파일을 사용해도 되지만 로봇을 단순화하고 싶은 경우 이를 통해 간소화가 가능합니다.
- **material** : **color, texture**등을 지정할 수 있으며, 외향적인 디자인을 위한 요소입니다.

### Xacro

> URDF 스크립트를 사람이 모두 작성하기는 매우 비효율적입니다. 더불어, Gazebo에서만 사용하는 속성을 따로 분리하고 싶은 경우, 파일을 나누어 관리하고 싶을 것입니다. 이러한 욕구를 충족시키기 위해서 ROS는 URDF의 작성을 보다 편하게 해주는 **XML Macro, Xacro**를 지원하고 있습니다.

특히 xacro는 수식, 조건을 사용 가능하기 때문에 로봇 파일을 다루기 매우 용이하며, 특정 요소를 모듈화 후 재사용하는 등 효율적인 URDF 작성이 가능하도록 도와줍니다.

```xml
<xacro:macro name="pr2_arm" params="suffix parent reflect">
<pr2_upperarm suffix="${suffix}" reflect="${reflect}" parent="${parent}" />
<pr2_forearm suffix="${suffix}" reflect="${reflect}" parent="elbow_flex_${suffix}" />
</xacro:macro>
<xacro:pr2_arm suffix="left" reflect="1" parent="torso" />
<xacro:pr2_arm suffix="right" reflect="-1" parent="torso" />
...

<pr2_upperarm suffix="left" reflect="1" parent="torso" />
<pr2_forearm suffix="left" reflect="1" parent="elbow_flex_left" />
<pr2_upperarm suffix="right" reflect="-1" parent="torso" />
<pr2_forearm suffix="right" reflect="-1" parent="elbow_flex_right" />
```

CMake를 통해 손쉽게 Xacro 파일들을 URDF로 자동 변환하는 작업을 실습해봅시다.

![fb5.png](/kr/ros_and_gazebo/images1/fb5.png?height=200px)

1. 해당 패키지 빌드

```bash
colcon build --packages-select fusionbot_description
source install/setup.bash
```

> 패키지 빌드가 발생하면 모든 관련 파일들의 symbolic link가 workspace의 **install/<repo-name>/share** 폴더에 생성됩니다. CMake는 이 share 폴더 내에 있는 symbolic link를 위주로 작업하므로 새로운 파일이 생겼다면 주기적으로 colcon build를 실행하는 것을 추천합니다.

![fb6.png](/kr/ros_and_gazebo/images1/fb6.png?height=100px)

2. CMakelists.txt 수정 - 주석되어 있는 라인을 주석 해제합니다.

```python
find_package(xacro REQUIRED)
# Xacro files
file(GLOB xacro_files urdf/*.urdf.xacro)

foreach(it ${xacro_files})
  # remove .xacro extension
  string(REGEX MATCH "(.*)[.]xacro$" unused ${it})
  set(output_filename ${CMAKE_MATCH_1})

  # create a rule to generate ${output_filename} from {it}
  xacro_add_xacro_file(${it} ${output_filename})

  list(APPEND urdf_files ${output_filename})
endforeach(it)

add_custom_target(media_files ALL DEPENDS ${urdf_files})
```

3. 패키지를 다시 빌드하면, 자동으로 생성된 urdf 파일을 확인할 수 있습니다.

```bash
sudo apt install ros-foxy-xacro -y
colcon build --packages-select fusionbot_description
```

```xml
<?xml version="1.0" ?>
<!-- =================================================================================== -->
<!-- |    This document was autogenerated by xacro from /home/swimming/nav2_ws/src/nav2_rosdevday_2021/djhrd_ros2/fusionbot_description/urdf/fusionbot.urdf.xacro | -->
<!-- |    EDITING THIS FILE BY HAND IS NOT RECOMMENDED                                 | -->
<!-- =================================================================================== -->
<robot name="fusionbot">
  <material name="silver">
    <color rgba="0.700 0.700 0.700 1.000"/>
  </material>
...
```

{{% notice note %}}
기존 fusionbot.urdf를 제거한 뒤 자동 생성되는 모습을 확인해보세요.

{{% /notice %}}

{{% notice note %}}
CMake는 사용자의 편의를 위해 추가해둔 것일 뿐 키워드를 통해 Xacro ⇒ URDF로의 변환도 가능합니다. 편한 방법을 사용하시기 바랍니다. (workspace를 sourcing 하셔야 실행 가능합니다.)

{{% /notice %}}

```bash
xacro fusionbot.urdf.xacro > fusionbot.urdf
```

### joint state publisher와 robot state publisher

> urdf를 모두 작성했다면, 이제 ROS에게 이 정보를 전달해야 합니다. 이를 담당하는 패키지인 joint state publisher와 robot state publisher에 대해 학습해 봅시다.

- 개념 학습 이전, 예시를 먼저 실행해보겠습니다.

```xml
ros2 launch fusionbot_description description.launch.py
```

![fb7.png](/kr/ros_and_gazebo/images1/fb7.png?height=300px)

{{% notice note %}}
WSL2 + Windows 시스템에서는 로봇 외관이 보이지 않을 수 있습니다.
{{% /notice %}}

- rviz 상에 로봇이 등장했으며, 로봇과 tf를 확인할 수 있습니다.
- joint state publisher gui라는 작은 창이 등장하며, 이 안에 위치한 작은 슬라이드바를 움직이면 로봇의 바퀴에 박힌 tf도 회전합니다.
- ROS에서는 이렇게 tf와 description의 형태로 로봇을 인지합니다.

#### joint state publisher

> joint state publisher는 로봇 내 존재하는 다양한 joint 값들을 실시간으로 갱신하여 **/joint_states**라는 topic으로 publish 합니다. 해당 topic은 [sensor_msgs/msg/JointState](http://docs.ros.org/en/noetic/api/sensor_msgs/html/msg/JointState.html) msg를 사용하며, 각 joint들의 이름, 현재 위치, 속도와 힘을 배열 형태로 담게 됩니다.

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

#### joint state publisher gui

> 예제 실행 시 등장했던 조그만 창은 joint_state_publisher_gui라고 불리며, 로봇 내 조작이 가능한 joint들을 간단하게 제어할 수 있도록 해주는 작은 프로그램입니다. 이를 통해 구성한 URDF의 방향은 알맞게 설정되었는지, 원점은 잘 맞는지 등을 확인 가능합니다.

- topic echo를 실행시킨 뒤, joint_state_publisher_gui를 조작해보면서 변화를 살펴봅시다.

```bash
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

⇒ **right_wheel_joint는** 위치 1.9188를 갖고 있으며, 속도와 힘은 제어되고 있지 않고 있습니다.

⇒ **left_wheel_joint는** 위치 -1.4093를 갖고 있으며, 속도와 힘은 제어되고 있지 않고 있습니다.

![fb8.png](/kr/ros_and_gazebo/images1/fb8.png?height=300px)

> 이렇게 joint state publisher는 현재 로봇이 가진 움직일 수 있는 모든 **joint들을** 예의주시하고 topic으로 publish하는 node입니다.

#### robot state publisher

> URDF의 joint는 joint state publisher가 담당했다면, robot state publisher는 모든 link와 joint 값을 지속적으로 Subscribe하여 전체 로봇의 구조를 tf 형식으로 publish합니다. 더불어, robot state publisher가 publish하는 **/robot_description** topic은 rviz에서 로봇의 시각화를 위해 사용되고, gazebo에서 로봇을 등장시키기 위해 사용됩니다.

![fb9.png](/kr/ros_and_gazebo/images1/fb9.png?height=300px)

{{% notice tip %}}
참고로, ROS 2에서 tf tree를 얻기 위해서는 아래와 같은 커멘드 라인을 사용합니다. (몇 초간 tf listen을 거친 뒤 pdf 형태로 tf tree를 도출해줍니다.)
{{% /notice %}}

```bash
$ ros2 run tf2_tools view_frames.py
```

![fb10.png](/kr/ros_and_gazebo/images1/fb10.png?height=300px)

지금까지 배운 내용들을 복습해봅시다.

- 로봇 모델이 ROS에게 전달되기 위해 urdf를 사용했고, 이는 link와 joint로 이루어졌습니다.
- 로봇 내 joint는 joint state publisher가 담당하며,
- robot state publisher는 joint state와 함께 link를 결합하여 최종 tf를 broadcast했습니다.

> 각 Node간 연결을 rqt_graph를 통해 살펴보면 아래와 같습니다.

![fb11.png](/kr/ros_and_gazebo/images1/fb11.png?height=300px)

### Launch file 분석

> description.launch.py를 분석해봅시다.

- rviz description을 위해 필요한 Node는 총 3가지입니다. (현재는 joint_state_publish_gui가 사용되고 있지만 이후에는 일반 joint_state_publisher가 사용됩니다.)

```python
return LaunchDescription([
    joint_state_publisher_gui,
    robot_state_publisher,
    TimerAction(
        period=5.0,
        actions=[rviz]
    ),
])
```

- joint_state_publisher_gui는 일반적인 Node 실행 옵션과 동일합니다.

```python
# Joint State Publisher
joint_state_publisher_gui = Node(
    package='joint_state_publisher_gui',
    executable='joint_state_publisher_gui',
    name='joint_state_publisher_gui'
)
```

- robot_state_publisher는 urdf를 argument로 요구합니다. urdf 파일의 위치는 os.path.join를 통해 전달하고 있으며, 여기서의 path는 src 폴더가 아닌 install 폴더의 symbolic link를 참조한다는 것에 주의합니다.

```python
    # Prepare Robot State Publisher Params
    urdf_file = os.path.join(description_pkg_path, 'urdf', 'fusionbot_description.urdf')
        # Robot State Publisher
        robot_state_publisher = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        name='robot_state_publisher',
        output='screen',
        parameters=[{'use_sim_time': True}],
        arguments=[urdf_file],
    )
```

- 마지막으로 rviz를 3초 후 실행되도록 합니다. 이는 launch.actions에서 제공하는 TimerAction 기능을 사용한 것으로, urdf의 내용이 너무 많으면 joint state와 robot state를 로드하는데 시간이 소요되기 때문에 적당한 Delay를 갖도록 한 것입니다.

```python
# Launch RViz
    rviz_config_file = os.path.join(description_pkg_path, 'rviz', 'description.rviz’)

    rviz = Node(
        package='rviz2',
        executable='rviz2',
        name='rviz2',
        output='screen',
        arguments=['-d', rviz_config_file]
    )
    ...
    TimerAction(
        period=3.0,
        actions=[rviz]
    ),
```

### Make a Mobile Robot

> 지금까지의 예시들은 모두 제가 미리 작성해둔 xacro를 사용하였는데요, 그럼 이런 xacro와 urdf는 어떻게 만들 수 있는지, 설계된 모바일 로봇을 바탕으로 Gazebo, ROS와 연동하는 방법에 대해 알아보겠습니다.

- 로봇을 설계하기 위한 수많은 설계 프로그램들이 존재합니다.

![fb12.png](/kr/ros_and_gazebo/images1/fb12.png?height=300px)

이번 예시는 무료 사용이 가능한 [Autodesk의 Fusion 360](https://www.autodesk.co.kr/products/fusion-360/overview?term=1-YEAR&tab=subscription)을 사용하였으며, Fusion 360에 Third Party ADD_IN을 추가하여 URDF를 생성하고, 생성된 URDF파일을 사용하여 Rviz 및 Gazebo와 연동하고자 합니다.

{{% notice tip %}}
사용한 Add In 링크는 다음과 같습니다. ⇒ [https://github.com/syuntoku14/fusion2urdf](https://github.com/syuntoku14/fusion2urdf)
{{% /notice %}}

{{% notice tip %}}
링크를 통해 FusionBot의 외형을 확인할 수 있습니다. ⇒ [https://a360.co/3gdOajr](https://a360.co/3gdOajr)
{{% /notice %}}

완성된 로봇으로부터 URDF를 추출하면 첨부파일과 같은 패키지를 얻을 수 있습니다.

📂 [fusionbot_description.zip](https://drive.google.com/file/d/1vltZMsTH3wkMysQQRmKJ5kiC4adzJxH0/view?usp=sharing)

![fb13.png](/kr/ros_and_gazebo/images1/fb13.png?height=300px)

ROS 1의 경우 바로 사용이 가능하지만, ROS 2는 몇가지 추가 설정들이 필요합니다. 따라서, 이번 예시에서는 ROS 2 환경을 기준으로 설정을 변경해보면서 URDF와 ROS 연동에 대해 학습해보겠습니다.

- 실습을 위해 새로운 ROS 2 패키지를 생성합니다. (ament_cmake 이용)

```bash
$ ros2 pkg create --build-type ament_cmake <package_name>
ros2 pkg create --build-type ament_cmake temp_description
```

{{% notice note %}}
결국 우리가 만들고자 하는 패키지는 제가 배포한 fusionbot_description과 동일합니다. 예제를 따라오시면서 발생하는 문제는 fusionbot_description을 참고하여 해결하시면 됩니다.
{{% /notice %}}

- 새롭게 생성한 ROS 2 패키지에 fusionbot_description의 **meshes**와 **urdf** 폴더를 이동시킵니다. (meshes에는 표면 질감 등 시각화 관련 파일들이 담겨 있습니다.)

![fb14.png](/kr/ros_and_gazebo/images1/fb14.png?height=200px)

> urdf 폴더 내부의 파일들을 수정해봅시다.

- **fusionbot.gazebo을 아래와 같이 수정**합니다.

```xml
<?xml version="1.0" ?>
<robot name="fusionbot" xmlns:xacro="http://www.ros.org/wiki/xacro" >

<xacro:property name="body_color" value="Gazebo/Silver" />

<gazebo>
  <plugin filename="libgazebo_ros_control.so" name="control"/>
</gazebo>
<gazebo reference="base_link">
  <material>${body_color}</material>
  <mu1>0.1</mu1>
  <mu2>0.1</mu2>
  <selfCollide>true</selfCollide>
  <gravity>true</gravity>
</gazebo>

<gazebo reference="lidar_1">
  <material>${body_color}</material>
  <mu1>0.2</mu1>
  <mu2>0.2</mu2>
  <selfCollide>true</selfCollide>
</gazebo>

<gazebo reference="right_wheel_1">
  <material>${body_color}</material>
  <mu1>100000.0</mu1>
  <mu2>100000.0</mu2>
  <selfCollide>true</selfCollide>
</gazebo>

<gazebo reference="left_wheel_1">
  <material>${body_color}</material>
  <mu1>1500</mu1>
  <mu2>1500</mu2>
  <selfCollide>true</selfCollide>
</gazebo>

</robot>
```

{{% notice tip %}}
libgazebo_ros_control는 ros control의 gazebo 버전입니다. ros control interface를 통해 gazebo 상의 로봇과 실제 로봇의 움직임을 별도의 추가 개발 없이 편리하게 Swap 가능합니다.
{{% /notice %}}

{{% notice tip %}}

mu1, mu2는 마찰력과 관련된 변수로 자세한 설명은 아래 링크를 참고하세요.
{{% /notice %}}

=> [https://answers.gazebosim.org//question/13083/explain-gazebo-friction-coefficients-mu-and-mu2/](https://answers.gazebosim.org//question/13083/explain-gazebo-friction-coefficients-mu-and-mu2/)

다음으로, 아래 작업들을 수행합니다.

=> **fusionbot.trans를 삭제합니다.** 해당 파일은 ROS 1에서만 필요한 파일입니다. 이는 ros_control을 gazebo에서 실행할 때 필요한 파일로 ROS 1 시간에 함께 살펴보겠습니다.

=> **fusionbot.xacro를 fusionbot.urdf.xacro으로 이름을 변경합니다.** (이후 자동 URDF 생성을 위함입니다.)

=> **변경한** **fusionbot.urdf.xacro를 수정합니다.**

---

- 이제, fusionbot.urdf.xacro에서 fusionbot.trans 부분을 삭제합니다.

```xml
<xacro:include filename="$(find fusionbot_description)/urdf/materials.xacro" />
<!-- <xacro:include filename="$(find fusionbot_description)/urdf/fusionbot.trans" /> -->
<xacro:include filename="$(find fusionbot_description)/urdf/fusionbot.gazebo" />
```

- ȸ��7 과 같이 잘못 인코딩된 joint 이름을 적절히 수정합니다.

```xml
# From
<joint name="��ü3" type="fixed">
  <origin rpy="0 0 0" xyz="0.05 0.0 0.11"/>
  <parent link="base_link"/>
  <child link="lidar_1"/>
</joint>

<joint name="ȸ��7" type="continuous">
  <origin rpy="0 0 0" xyz="0.0 -0.1 0.05"/>
  <parent link="base_link"/>
  <child link="right_wheel_1"/>
  <axis xyz="-0.0 -1.0 0.0"/>
</joint>

<joint name="ȸ��8" type="continuous">
  <origin rpy="0 0 0" xyz="0.0 0.1 0.05"/>
  <parent link="base_link"/>
  <child link="left_wheel_1"/>
  <axis xyz="-0.0 1.0 0.0"/>
</joint>

# To
<joint name="lidar_joint" type="fixed">
  <origin rpy="0 0 0" xyz="0.05 0.0 0.11" />
  <parent link="base_link" />
  <child link="lidar_1" />
</joint>

<joint name="right_wheel_joint" type="continuous">
  <origin rpy="0 0 0" xyz="0.0 -0.1 0.05" />
  <parent link="base_link" />
  <child link="right_wheel_1" />
  <axis xyz="-0.0 -1.0 0.0" />
</joint>

<joint name="left_wheel_joint" type="continuous">
  <origin rpy="0 0 0" xyz="0.0 0.1 0.05" />
  <parent link="base_link" />
  <child link="left_wheel_1" />
  <axis xyz="-0.0 1.0 0.0" />
</joint>
```

- joint 축 방향을 수정합니다.

```xml
<joint name="right_wheel_joint" type="continuous">
  <origin rpy="0 0 0" xyz="0.0 -0.1 0.05" />
  <parent link="base_link" />
  <child link="right_wheel_1" />
  <axis xyz="0.0 1.0 0.0" />
</joint>

<joint name="left_wheel_joint" type="continuous">
  <origin rpy="0 0 0" xyz="0.0 0.1 0.05" />
  <parent link="base_link" />
  <child link="left_wheel_1" />
  <axis xyz="0.0 1.0 0.0" />
</joint>
```

- 파일 상단 **base_footprint** link를 추가하고 base_footprint와 base_link를 연결하는 joint를 추가합니다. base_footprint는 추후 자율주행 시 필요한 tf ID로 로봇 중심에서 바닥면에 projection한 위치를 뜻합니다.

```xml
<link name='base_footprint' />
...
<joint name='base_link_joint' type='fixed'>
  <origin rpy="0 0 0" xyz="0 0 0" />
  <parent link="base_footprint" />
  <child link="base_link" />
</joint>
```

- mesh 파일의 uri와 xacro 라인의 **package명을 여러분의 패키지로 수정**합니다.

```xml
# From
<xacro:include filename="$(find fusionbot_description)/urdf/materials.xacro" />
...
<mesh filename="package://fusionbot_description..." />

# To
<xacro:include filename="$(find temp_description)/urdf/materials.xacro" />
...
<mesh filename="package://temp_description..." />
```

- 색상을 입히고 싶다면 materials.xacro를 수정합니다. (Gazebo는 고유한 색상 체계를 갖고 있습니다. 이는 [링크](http://classic.gazebosim.org/tutorials?tut=color_model&cat=)를 참고합니다.)

```xml
<?xml version="1.0"?>
<robot name="fusionbot" xmlns:xacro="http://www.ros.org/wiki/xacro">

  <material name="silver">
    <color rgba="0.700 0.700 0.700 1.000" />
  </material>
  <material name="white">
    <color rgba="1 1 1 0.6" />
  </material>
  <material name="black">
    <color rgba="0 0 0 0.7" />
  </material>
  <material name="red">
    <color rgba="1 0 0 1" />
  </material>
  <material name="blue">
    <color rgba="0 0 0.8 1"/>
  </material>

</robot>
```

- CMakeLists.txt를 수정합니다. ament_package() 이전 아래 코드를 추가하시면 됩니다.

```bash
## Install
install(DIRECTORY meshes urdf
  DESTINATION  share/${PROJECT_NAME}
)
**
ament_package()
```

- 작업을 마친 뒤, **colcon build**를 한차례 실시합니다.

```xml
cd ~/ros2_ws
colcon build --packages-select temp_description
source install/local_setup.bash
```

- **CMakeLists.txt**에 다음 라인을 추가한 뒤 다시 한 번 colcon build를 실시합니다.

```bash
find_package(xacro REQUIRED)
# Xacro files
file(GLOB xacro_files urdf/*.urdf.xacro)

foreach(it ${xacro_files})
  # remove .xacro extension
  string(REGEX MATCH "(.*)[.]xacro$" unused ${it})
  set(output_filename ${CMAKE_MATCH_1})

  # create a rule to generate ${output_filename} from {it}
  xacro_add_xacro_file(${it} ${output_filename})

  list(APPEND urdf_files ${output_filename})
endforeach(it)

add_custom_target(media_files ALL DEPENDS ${urdf_files})
```

⇒ 이 과정이 필요한 이유는 다음과 같습니다.

1. 첫번째 colcon build로 필요 파일들의 symbolic link가 install 폴더 내 생성됩니다.
2. 두번째 colcon build 시 CMakeLists.txt에 작성한 xacro 빌드 스크립트를 통해 xacro ⇒ urdf로 변환되며, 이 과정에서 CMake는 install 폴더 내 symbolic link들을 필요로 합니다.
3. 만약 첫번째 작업이 없다면, mesh, urdf 관련 파일이 없다는 에러가 발생합니다.

- 위 과정을 마치면 자동으로 fusionbot.urdf가 생성되며, 이후 해당 urdf는 description과 gazebo 등 다양한 목적으로 사용할 수 있습니다.

```xml
<?xml version="1.0" ?>
<!-- =================================================================================== -->
<!-- |    This document was autogenerated by xacro from /home/kimsooyoung/ros2_ws/src/du2023-gz/test_description/urdf/fusionbot.urdf.xacro | -->
<!-- |    EDITING THIS FILE BY HAND IS NOT RECOMMENDED                                 | -->
<!-- =================================================================================== -->
<robot name="fusionbot">
  <material name="silver">
    <color rgba="0.700 0.700 0.700 1.000"/>
  </material>
...
```

### Description launch 실행해보기

> **robot state publisher**와 **joint_state_publisher_gui**를 통해 robot_description topic을 생성하고 rviz를 통해 이를 시각화해봅시다.

- 예시를 위해 아래와 같은 폴더 구조가 필요합니다. (launch 폴더와 rviz 폴더를 추가하였습니다.)

![fb15.png](/kr/ros_and_gazebo/images1/fb15.png?height=300px)

- launch 폴더를 생성하고, 해당 폴더 내부에 launch 파일을 작성해볼 것입니다. 필요한 Node는 아래와 같은 3개 이며, [fusionbot_description.launch.py](https://github.com/RB2023ROS/du2023-gz/blob/main/fusionbot_description/launch/fusionbot_gazebo.launch.py)를 참고하여 직접 launch file을 작성해보세요!

```bash
return LaunchDescription([
    joint_state_publisher_gui,
    robot_state_publisher,
    rviz,
])
```

- rviz 폴더도 생성하고 기존 fusionbot_description에서 config 파일을 가져옵니다.

![fb16.png](/kr/ros_and_gazebo/images1/fb16.png?height=200px)

- 새로운 폴더가 추가되면 항상 **CMakeLists.txt**를 수정해야 합니다.

```bash
## Install
install(DIRECTORY meshes urdf launch rviz
  DESTINATION  share/${PROJECT_NAME}
)
```

- 패키지를 빌드한 뒤, 여러분이 작성한 launch file을 실행해보세요!

```xml
colcon build --symlink-install --packages-select temp_description
source install/local_setup.bash

ros2 launch temp_description description.launch.py
```

![fb17.png](/kr/ros_and_gazebo/images1/fb17.png?height=400px)
