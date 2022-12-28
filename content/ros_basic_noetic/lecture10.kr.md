---
title: "Lecture10 - TF2 Examples, Outro"
date: 2022-12-24T13:48:05+09:00
draft: false
---

### tf2 사례

> 제가 강조해서 자꾸 좌표계가 tf가 중요하다고 말하고 있는데, 그 이유를 예시와 함께 좀 더 자세히 살펴보고자 합니다.

- husky slam

```bash
# 예시 종속성 설치
sudo apt install ros-noetic-slam-gmapping
# Terminal 1
roslaunch smb_gazebo smb_gazebo.launch world:=big_map_summer_school
# Terminal 2
roslaunch py_tf2_tutorial slam_gmapping.launch
# Terminal 3
rosrun teleop_twist_keyboard teleop_twist_keyboard.py

# [option] rviz (이전 예시에서 rviz를 추가하였다면 넘어가셔도 좋습니다.)
rviz
```

- rviz를 다음과 같이 세팅합니다.

![slam_rviz.gif](/kr/ros_basic_noetic/images9/slam_rviz.gif?height=400px)

- 이제 teleop을 통해 로봇을 이동시키면서, rviz 화면의 변화를 확인해봅시다.

![slam_ex.gif](/kr/ros_basic_noetic/images9/slam_ex.gif?height=400px)

> 로봇이 움직이면서 자신의 위치를 파악함과 동시에 **지도를 생성하는 예시**입니다.

- 마지막으로, rqt를 실행하여 tf tree를 실행시킵니다.

![tf_rqt_tree.gif](/kr/ros_basic_noetic/images9/tf_rqt_tree.gif?height=400px)

- tf tree는 tf 관련 상태를 시각화하여 한번에 볼 수 있게 해주는 고마운 툴입니다.

![frames.png](/kr/ros_basic_noetic/images9/frames.png?height=300px)

#### tree를 확대해서 살펴보자면,

- slam_gmapping은 **map → odom**으로의 **tf broadcast**를 담당하고 있습니다.
- 더불어 map은 모든 **tf의 최상단에** 존재하고 있습니다.
- 이러한 이유로 rviz에서 **fixed frame**을 map으로 설정한 것입니다.

![lec8_10.png](/kr/ros_basic_noetic/images9/lec8_10.png?height=300px)

{{% notice note %}}
퀴즈: 만약 tf tree가 온전히 연결되어 있지 않다면 어떤 일이 발생할까요?
{{% /notice %}}

![lec8_11.png](/kr/ros_basic_noetic/images9/lec8_11.png?height=300px)

- image from : [answers.ros.org](https://answers.ros.org/question/316747/cartographer-slam-tf-tree-errors/)

**센서** 입장에서도 tf는 매우 중요합니다.

같은 데이터라도 그 기준이 어딘지에 따라서 전혀 다른 의미를 가질 수 있기 때문입니다.

예를 들어, 라이다의 **tf를 180도 반대로 설정**해버리면 후방에 있는 장애물을 **전방 장애물로 잘못 인식**할 수 있습니다.

![lec8_12.png](/kr/ros_basic_noetic/images9/lec8_12.png?height=300px)

- image from : [answers.ros.org](https://answers.ros.org/question/265846/how-to-build-tf-topic-frame-tree-for-hector_slam-or-gmapping/)

더불어, 로봇 팔과 같은 관절로봇에게도 tf는 무척 중요한 의미를 갖습니다. 각 joint의 상태를 통해 tf를 계산하고 이를 통해 최종적으로 로봇 팔의 끝점이 어디에 위치하는지 계산할 수 있습니다.

---

### MoveIt! 실습해보기

이번 시간에는 조금 쉬어가는 느낌으로 유용한 ROS Package를 소개해드리고자 합니다.

![lec8_14.png](/kr/ros_basic_noetic/images9/lec8_14.png?height=100px)

- image from : [moveit github](https://github.com/ros-planning/moveit)

**MoveIt**은 다관절 로봇의 모션 제어를 위한 프레임워크입니다. 이름만 들어서는 감이 잘 오지 않지요? 간단한 예시를 통해 살펴봅시다.

- 우리 인간은 팔을 이용하여 물건을 잡는 것이 매우 쉽고 간단하지만, 사실 이는 기구학적으로, 동역학적으로, 에너지 차원에서 매우 최적화된 움직임입니다.
- 로봇 팔의 경우 장착된 모터의 방향각이 제한된 경우도 있고, 자기 자신과 얽혀버리는 문제도 발생할 수 있으며, 같은 목표를 갖더라고 다양한 경로로 움직일 수 있기 때문에 최적의 경로에 대한 기준도 고려해야 합니다.

![lec8_15.png](/kr/ros_basic_noetic/images9/lec8_15.png?height=400px)

- image from : [mecademic](https://www.mecademic.com/en/what-are-singularities-in-a-six-axis-robot-arm)

#### 로봇 팔의 주요 구성

- **Base** : 고정된 지지부
- **Arm** : 실질적인 로봇 팔
- **End Effector** : Arm 끝에 부착되는 기구의 통칭, 일반적으로 물체를 잡고 놓는 동작을 수행

![lec8_16.png](/kr/ros_basic_noetic/images9/lec8_16.png?height=400px)

MoveIt은 관절 로봇의 기본 구성과 Mass Matrix, 각 모터의 제한과 원하는 움직임을 지정해주면 이에 따라 각 관절의 위치, 속도, 가속도 경로를 최적화(Planning) 해주는 프레임워크이며, 그 밖에도, **물체 인지, 장애물 회피, End Effector에 가해지는 힘**까지 고려 가능한 거대한 오픈소스 프로젝트입니다.

[MoveIt Motion Planning Framework](https://moveit.ros.org/)

> 이번 예제로 저와 함께 MoveIt의 가장 기본적인 데모를 함께 실행해보겠습니다. 예시에 사용되는 로봇은 FRANKA EMIKA의 **PANDA**라는 로봇입니다. [FRANKA EMIKA - PANDA](https://wego-robotics.com/franka-emika-panda/)

아래 커멘드 라인을 함께 따라와주세요.

- apt 패키지 설치

```bash
sudo apt install ros-noetic-moveit-setup-assistant
sudo apt install ros-noetic-moveit

sudo apt install ros-noetic-gazebo-ros-control joint-state-publisher
sudo apt install ros-noetic-controller-manager
sudo apt install ros-noetic-ros-controllers
sudo apt install ros-noetic-ros-control
sudo apt install ros-noetic-robot-state-publisher
```

- 예제 패키지 Clone

```bash
cd ~/catkin_ws
git clone https://github.com/ros-planning/moveit_tutorials.git -b master
git clone https://github.com/ros-planning/panda_moveit_config.git -b noetic-devel
```

- 관련 종속성 설치 - rosdep 추가 설명

```bash
cd ~/catkin_ws/src
rosdep install -y --from-paths . --ignore-src --rosdistro noetic
```

- 패키지 빌드

```bash
cd ~/catkin_ws
catkin build
source devel/setup.bash
```

- 데모 실행

```bash
roslaunch panda_moveit_config demo_gazebo.launch
```

> 여기까지 잘 따라오셨나요? 그렇다면 강의자의 설명에 따라 **RViz Motion Planning Plugin**을 사용해봅니다.

![moveit_gazebo.gif](/kr/ros_basic_noetic/images9/moveit_gazebo.gif?height=400px)

**참고자료**

- [https://rsl.ethz.ch/education-students/lectures/ros.html](https://rsl.ethz.ch/education-students/lectures/ros.html)
- [https://ros-planning.github.io/moveit_tutorials/](https://ros-planning.github.io/moveit_tutorials/)
- [http://wiki.ros.org/tf/Tutorials](http://wiki.ros.org/tf/Tutorials)
