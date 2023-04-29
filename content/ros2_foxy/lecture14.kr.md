---
title: "Lecture14. Useful ROS 2 Examples (Control Part)"
date: 2023-04-28T10:51:09+09:00
draft: false
---

## Gazebo ROS 2 Control

![Untitled.png](/kr/ros2_foxy/images14/Untitled.png?height=300px)

지난 fusionbot 예시에서, Differential Drive만 예외적으로 gazebo pluin을 제공한다고 하였습니다. 일반적으로 Gazebo내에서 움직이는 (=모터를 가진) 모델을 구현하기 위해서는 gazebo_ros2_control을 사용합니다.

다만, ROS 2 Control 전체 시스템을 이해하기 위해서는 상당한 배경 지식과 시간이 필요하므로, 이번 예시에서는 딱 하나, 위치 제어를 하는 가이드를 드리도록 하겠습니다.

- 일전 sensor_stick의 URDF에는 tilt_joint라는 joint가 있었습니다. 우리의 목표는 이 tilt_joint를 말 그대로 기울여질 수 있게 해보는 것입니다.

```xml
<joint name="tilt_joint" type="revolute">
  <origin xyz="0.05 0 ${length2/2+0.02}" rpy="0 0.5 0" />
  <parent link="link_2" />
  <child link="camera_link" />
  <limit lower="-1.5707" upper="1.5707" effort="10" velocity="100" />
  <axis xyz="0 1 0"/>
</joint>
```

- 예시 실행

```bash
# Terminal 1
$ ros2 launch basic_stick moving_stick.launch.py
> gazebo world 내에 table등 원하는 물체를 추가합니다.

# Terminal 2
ros2 topic pub /forward_position_controller/commands std_msgs/msg/Float64MultiArray "data:
- 0.1"
> 마지막 data value를 바꿔가면서 원하는 기울기로 조정해보세요!
```

![Untitled1.png](/kr/ros2_foxy/images14/Untitled1.png?height=300px)

gazebo_ros의 ForwardCommandController를 사용하여 제작한 예시이며, ros2 control에 대해 학습하지 않았으므로, 이를 위한 절차만 간단히 설명드리겠습니다.

1. URDF 내 움직일 수 있는 joint 추가
2. gazebo_ros2_control을 정의하는 xacro 추가
3. gazebo_ros2_control controller 매개변수를 정의하는 yaml file 추가

- URDF 내 움직일 수 있는 joint 추가

```xml
<joint name="tilt_joint" type="revolute">
  <origin xyz="0.05 0 ${length2/2+0.02}" rpy="0 0.5 0" />
  <parent link="link_2" />
  <child link="camera_link" />
  <limit lower="-1.5707" upper="1.5707" effort="10" velocity="100" />
  <axis xyz="0 1 0"/>
</joint>
```

|                     | Description                                                                          |
| ------------------- | ------------------------------------------------------------------------------------ |
| revolute            | 문고리와 같이 limit을 갖는 joint 입니다.                                             |
| limit - lower/upper | revolute joint의 limit을 설정합니다.                                                 |
| effort              | 최대 토크를 정의합니다. (이론은 모르셔도 됩니다. 지금은 적당히 넉넉히 해두었습니다.) |
| velocity            | 최대 속도를 정의합니다.                                                              |
| axis                | joint가 움직일 축 방향을 지정합니다.                                                 |

- gazebo_ros2_control을 정의하는 xacro 추가 - ros2_control.gazebo.xacro이라는 이름으로 추가하였으며, 최소한의 내용만 짚고 넘어가겠습니다.

```xml
<?xml version="1.0"?>
<!-- ref github - diffbot_system.ros2_control.xacro -->
<robot xmlns:xacro="http://www.ros.org/wiki/xacro">

  <ros2_control name="GazeboSystem" type="system">
    <!-- Gazebo -->
    <hardware>
      <plugin>gazebo_ros2_control/GazeboSystem</plugin>
    </hardware>

    <joint name="tilt_joint">
      <command_interface name="position">
        <param name="min">-1.5707</param>
        <param name="max">1.5707</param>
      </command_interface>
      <state_interface name="position">
        <param name="initial_value">0.0</param>
      </state_interface>
      <state_interface name="velocity"/>
    </joint>
  </ros2_control>

  <gazebo>
    <plugin name="gazebo_ros2_control" filename="libgazebo_ros2_control.so">
        <parameters>$(find basic_stick)/config/joint_controller.yaml</parameters>
    </plugin>
  </gazebo>

</robot>
```

|                   | Description                                                                                                      |
| ----------------- | ---------------------------------------------------------------------------------------------------------------- |
| joint name        | control이 필요한 joint name을 지정합니다.                                                                        |
| command_interface | position 제어를 위해 name=position, 이에 따른 최대, 최소 각도를 지정합니다.                                      |
| state_interface   | topic을 통해 피드백받을 데이터 종류를 선택합니다. 지금은 위치 피드백을 받을 예정이며, 초기값을 0도로 두었습니다. |
| plugin parameters | controller 실행을 위해 추가로 필요한 매개변수들을 yaml file로 전달합니다.                                        |

- 이번 예시에서는 control, state feedback, 총 2개의 plugin이 사용됩니다. 각 pluin들에 대해서 필수로 정의해줘야 하는 매개변수들이 있으며, forward_position_controller의 경우 joints를 필요로 합니다.

```yaml
controller_manager:
  ros__parameters:
    update_rate: 100 # Hz

    joint_state_broadcaster:
      type: joint_state_broadcaster/JointStateBroadcaster

    forward_position_controller:
      type: forward_command_controller/ForwardCommandController

forward_position_controller:
  ros__parameters:
    joints:
      - tilt_joint
    interface_name: position
```

⇒ 여기까지 한번에 완성되었다면, 매우 순조로운 것입니다. 보통 robot_state_publisher의 실행을 통해 하나씩 검증하며 개발이 이루어집니다.

- 마지막으로, launch file을 살펴봅시다. ros2_control을 관리하는 controller_manager에게 필요한 plugin을 전달하면 spawner.py가 이들을 실행해주는 형태를 취하고 있습니다.

```python
fp_controller = Node(
    package="controller_manager",
    executable="spawner.py",
    arguments=["forward_position_controller"],
    output="screen",
)

jsb_controller = Node(
    package="controller_manager",
    executable="spawner.py",
    arguments=["joint_state_broadcaster"],
    output="screen",
)
```

> 이번 예시와 지난 dataset_gen action 예시, 그리고 take picture service 예시와 my world 생성까지 모두 결합하면, 무궁무진한 여러분들만의 gazebo 센서 환경을 갖추실 수 있을 것입니다.
