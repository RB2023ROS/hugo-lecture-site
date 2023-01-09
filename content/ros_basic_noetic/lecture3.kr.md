---
title: "Lecture3 - Core of ROS"
date: 2022-12-25T13:36:31+09:00
draft: false
---

> 지난 시간 개발환경 세팅을 잘 진행하였는지 확인을 해보면서 강의를 시작해보겠습니다.

터미널 프로그램을 실행한 뒤, gazebo를 실행해 봅시다.

```bash
gazebo
```

![img0](/kr/ros_basic_noetic/images3/lec2_0.png?height=400px)


{{% notice tip %}}
위 사진과 같은 화면이 나오지 않았다면 설치가 제대로 되지 않은 것입니다. (sudo apt install libgazebo11 을 통해 다시 설치해봅시다.)
{{% /notice %}}

## About Gazebo 

> Gazebo의 특징과 기본적인 UI, 그리고 사용법을 짚고 넘어가고자 합니다.

![gazebo_logo](/kr/ros_basic_noetic/images3/gazebo_logo.png?height=200px)


* image from : [Open Robotics](https://www.openrobotics.org/)



> Gazebo는 로봇공학을 위해 제작된 전용 물리 엔진 기반의 높은 3D 시뮬레이터입니다. 
> 
> ROS를 관리하는 Open Robotics에서 비롯된 시뮬레이터인만큼 ROS와 높은 호환성을 자랑합니다.
> 
> Gazebo는 실제 로봇을 위한 설계 검증 테스트와 시뮬레이션을 용이하게 하며, 윈도우, 맥, 리눅스에서 모두 실행되지만 대부분 리눅스 시스템에서 ROS와 함께 사용됩니다.

⇒ 이후 Gazebo 사용 및 오류 발생 시 디버깅을 위해, Gazebo를 구성하는 요소들을 간단하게 짚고 넘어가겠습니다.

### gzserver & gzclient

> Gazebo는 Socket-Based Communication을 갖습니다. 따라서 서버와 Client를 분리하여 실행 가능하며 다른 기기에서의 실행 후 연동도 가능합니다. 

- **Gazebo Server**

**gzserver**는 Gazebo 동작의 대부분을 수행합니다. 시뮬레이션하려는 장면과 그 안에 있는 개체 파일을 분석하고, 그런 다음 물리 엔진과 센서 엔진을 사용하여 전체 장면을 시뮬레이션합니다.

터미널에서 다음 명령을 사용하여 서버를 독립적으로 시작할 수 있습니다.

```cpp
$ gzserver
```

- **Gazebo Client**

**gzclient**는 gzserver에 연결하여 대화형 도구와 함께 시뮬레이션을 렌더링하는 Graphic Client를 제공합니다.

다음 명령을 사용하여 gzclient를 단독으로 실행할 수 있습니다.

```cpp
$ gzclient
```

{{% notice note %}}
client만 실행하면, 연결되고 명령을 수신할 서버가 없기 때문에 (컴퓨팅 리소스를 소비하는 것을 제외하고) 아무 것도 하지 않습니다.
{{% /notice %}}


- **Combining Gazebo Server and Gazebo Client**

gzserver를 먼저 실행한 다음 gzclient를 실행하는 것이 일반적이며, 렌더링하기 전에 시뮬레이션 장면, 객체 관련 파라미터를 초기화할 수 있습니다. 

gazebo 명령어를 입력하면 내부적으로 server와 client가 순차적으로 실행됩니다.

```cpp
$ gazebo
```

{{% notice tip %}}
이러한 이유로 Gazebo를 종료했다고 생각하지만 gzserver가 깔끔하게 종료되지 않는 상황이 발생합니다. ⇒ killg를 사용합시다!
{{% /notice %}}


### World File과 Model File

![world](/kr/ros_basic_noetic/images3/world.png?height=300px)

* image from : [dronecode](https://dev.px4.io/v1.11_noredirect/en/simulation/gazebo_worlds.html)

- **World Files**

Gazebo world file에는 로봇 모델, 환경, 조명, 센서, 다른 기타 물체들까지 시뮬레이션 환경의 모든 요소가 포함되어 있습니다. 일반적으로 확장자명 **.world**를 사용합니다. 

아래 예시와 같이 Gazebo 실행 시 world 파일을 옵션으로 하여 실행이 가능합니다. 

```cpp
$ gazebo <yourworld>.world
```

- **Model Files**

Gazebo의 World는 다양한 Model들로 구성됩니다. 하지만 Model만을 가지고 gazebo를 실행시킬 수는 없습니다. (gazebo <model> 이렇게는 불가합니다.)

모델을 별도의 파일로 보관하는 이유는 다른 프로젝트에 재사용하기 위해서이며, 로봇의 모델 파일 또는 다른 모델을 월드 파일 내에 포함하려면 다음과 같이 <uri>태그를 통해 import 할 수 있습니다. 

```xml
<include>
  <uri>model://model_file_name</uri>
</include>
```

- **Environment Variables**

Gazebo에 World, Model을 연동하고, gzserver와 gzclient 간 통신을 설정하기 사용하는 많은 환경 변수가 있습니다. 

예를 들어, GAZEBO_MODEL_PATH는 Gazebo가 모델 파일을 검색할 시 참조하는 경로입니다.

- **Plugins**

Gazebo의 World, Model에 장착되는 각종 센서와 제어를 위한 다양한 플러그인이 준비되어 있으며,  이러한 플러그인은 커멘드 라인에서 로드하거나 SDF 파일 내부에 추가할 수 있습니다. (자체 Plugin을 개발할 수도 있습니다.)

### **Understanding Gazebo GUI**

이번에는 Gazebo의 기본 조작 방법과 내장 Tool들을 살펴보겠습니다.

- 우선 Gazebo를 실행시킵니다.

```xml
$ gazebo
```

- Gazebo의 시점 변환은 마우스를 사용하여 손쉽게 조작 가능합니다.
- 마우스 왼쪽 버튼을 누르고 드래그하면 시점을 이동할 수 있습니다.
- 마우스 휠을 누른 상태로 이동시키면 회전 시점을 조작할 수 있습니다.

![ui](/kr/ros_basic_noetic/images3/ui.png?height=300px)


- 최상단에 위치한 **Top Toolbar**를 살펴보겠습니다.

![toolbar](/kr/ros_basic_noetic/images3/toolbar.png?height=50px)

> 왼쪽부터 오른쪽 순서대로 각 아이콘 별 기능을 설명해보겠습니다.  

- **Select mode**

Select mode는 가장 일반적으로 사용되는 커서 모드입니다. 장면을 탐색할 수 있습니다.

- **Translate mode**

커서 모드를 선택한 다음 이동시키길 원하는 객체를 클릭합니다.
이후 등장하는 3축 중 적절한 축을 사용하여 개체를 원하는 위치로 끌기만 하면 됩니다.

![new1](/kr/ros_basic_noetic/images3/new1.png?height=300px)

- **Rotate mode**

translate mode와 마찬가지로 이 커서 모드를 사용하면 주어진 모델의 방향을 변경할 수 있습니다.

![new2](/kr/ros_basic_noetic/images3/new2.png?height=300px)

- **Scale mode**

Scale mode를 사용하면 객체의 전체 크기를 변경할 수 있습니다.

![new3](/kr/ros_basic_noetic/images3/new3.png?height=300px)

- **Undo/Redo**

앞,뒤로 되돌리는 기능입니다.

- **Simple shapes**

큐브, 구체 또는 실린더와 같은 기본 3D 모델을 환경에 삽입할 수 있습니다.

![new4](/kr/ros_basic_noetic/images3/new4.png?height=300px)

- **Lights**

스포트라이트, 포인트 라이트 또는 방향이 정해진 조명과 같은 다양한 광원을 환경에 추가합니다.

![new5](/kr/ros_basic_noetic/images3/new5.png?height=300px)

- **Copy/Paste**

모델을 복사/붙여넣을 수 있습니다. (Ctrl+C/V를 통해서도 가능합니다.) 

- **Align**

이 도구를 사용하면 x y z 축 중 하나를 따라 한 모형을 다른 모델과 정렬할 수 있습니다.

![new6](/kr/ros_basic_noetic/images3/new6.png?height=300px)

또는 두 모델을 특정한 면 기반으로 서로 붙이는 것도 가능합니다. 

![new7](/kr/ros_basic_noetic/images3/new7.png?height=300px)

- **Change view**

상단 뷰, 측면 뷰, 전면 뷰, 하단 뷰와 같은 다양한 관점에서 장면을 볼 수 있습니다.

다음으로 **Side Panel**을 살펴보겠습니다.

- **World**

현재 사용중인 조명 및 모델들이 표시됩니다. 개별 모델을 클릭하여 위치 및 방향과 같은 모델의 기본 파라미터를 보거나 편집할 수 있습니다.
또한 물리 옵션을 통해 중력 및 자기장과 같은 물성치도 변경할 수 있습니다. GUI 옵션을 사용하면 기본 카메라 뷰 각도 및 포즈에 액세스할 수 있습니다.

![new8](/kr/ros_basic_noetic/images3/new8.png?height=300px)

- **Insert**

![new9](/kr/ros_basic_noetic/images3/new9.png?height=300px)


추가할 모델을 찾을 수 있습니다. 환경 변수에 지정된 폴더에서 모델들을 검색한 뒤 배치하는 것이 가능하며, 환경 변수 설정 없이도 Add Path 옵션을 통해 정해진 포멧을 갖는 모델을 가져올 수 있습니다.

---

> 다시 본론으로 돌아와서, ROS 설치는 잘 되었는지도 확인해봅시다.

```bash
# Terminal 1
roscore
# Terminal 2
rosrun rospy_tutorials talker
```

![talker.gif](/kr/ros_basic_noetic/images3/talker.gif?height=400px)

모든 확인이 끝났다면, 예제 프로그램을 실행시켜보겠습니다.

---

## Husky Gazebo

- 예제 패키지 설치

```
sudo apt-get update
sudo apt-get install ros-noetic-husky-desktop
sudo apt-get install ros-noetic-husky-simulator
```

- 예제 프로그램 실행

```bash
# Terminal 1
roslaunch husky_gazebo husky_empty_world.launch
# Terminal 2
roslaunch husky_viz view_robot.launch
# Terminal 3
rosrun teleop_twist_keyboard teleop_twist_keyboard.py cmd_vel:=/husky_velocity_controller/cmd_vel
```

Terminal 1에서 발생하는 아래 오류는 무시해도 좋습니다.

![lec2_1.png](/kr/ros_basic_noetic/images3/lec2_1.png?height=300px)

- 모든 실행이 정상 동작하였다면, Terminal 3에서 키보드를 통해 Husky를 제어할 수 있습니다.

![husky_move.gif](/kr/ros_basic_noetic/images3/husky_move.gif?height=400px)

- 로봇이 움직임에 따라 두번째 터미널 결과였던 Rviz에 아래와 같은 변화가 생깁니다.

![lec2_2.png](/kr/ros_basic_noetic/images3/lec2_2.png?height=400px)

- 다음으로, 새로운 터미널에서 아래 커멘드 라인을 실행시켜 봅시다.

```bash
rosrun rqt_graph rqt_graph
```

![rosgraph.png](/kr/ros_basic_noetic/images3/rosgraph.png?height=150px)

위 그림은 방금 전 실행한 예제 내부적으로 어떠한 동작들이 이루어지고 있었는지를 보여주는 것으로, 강의를 마칠 때면 여러분들은 위 그림이 어떠한 의미를 갖는지 모두 이해하실 수 있을 것입니다.

다음으로, 터미널을 새로 실행시켜 `rosnode list`와 `rostopic list`를 실행시켜 봅시다.

- **rosnode list**

```bash
$ rosnode list
/base_controller_spawner
/ekf_localization
/gazebo
/gazebo_gui
/joy_teleop/joy_node
/joy_teleop/teleop_twist_joy
/robot_state_publisher
/rosout
/teleop_twist_keyboard
/twist_marker_server
/twist_mux
```

- **rostopic list**

```bash
$ rostopic list
/clock
/cmd_vel
/diagnostics
/e_stop
/gazebo/link_states
/gazebo/model_states
/gazebo/parameter_descriptions
/gazebo/parameter_updates
/gazebo/performance_metrics
/gazebo/set_link_state
/gazebo/set_model_state
...
```

> 앞으로의 강의들에서, 위 명령어들이 어떠한 의미를 갖는지 하나하나씩 함께 살펴보겠습니다.

---

## ROS Node

ROS는 각 프로세스들을 **Node**라는 단위로 관리합니다.

- 로봇을 움직이는 Node, 센서와 통신하는 Node, 시각화 Node 등 다양한 Node들이 얽혀 로봇 시스템을 구성하게 됩니다.
- Node들 사이에는 **데이터의 송수신**이 필요합니다. 이를 담당하는 ROS의 통신 메커니즘들이 있으며 각기 다른 특성을 갖고 있습니다.
- Node들끼리 데이터를 주고받기 위해서는 어떤 노드가 존재하는지, id는 몇번인지 등의 정보가 공유되어야 할 것입니다. 아래 그림의 **ROS Master**가 이를 관리해주는 것이라고 이해하시면 됩니다.

![lec2_3.png](/kr/ros_basic_noetic/images3/lec2_3.png?height=400px)

- image from : [clearpathrobotics](http://www.clearpathrobotics.com/assets/guides/kinetic/ros/Intro%20to%20the%20Robot%20Operating%20System.html)

그렇다면, 방금 우리가 실행한 예시에서도 ROS Master와 Node들이 실행되었겠군요!

> 실행되는 Node를 확인하는 방법은 크게 두 가지가 있습니다.

1. **rosnode command**

```bash
$ rosnode list
/base_controller_spawner
/ekf_localization
/gazebo
/gazebo_gui
/joy_teleop/joy_node
/joy_teleop/teleop_twist_joy
/robot_state_publisher
/rosout
/teleop_twist_keyboard
/twist_marker_server
/twist_mux
```

2. **rqt_graph**

```bash
rosrun rqt_graph rqt_graph
```

rqt graph를 살펴보면, 동그란 **Node**와 Node들 사이의 데이터 송수신이 **화살표**로 표현된 것을 알 수 있습니다. 키보드를 통해 제어 데이터를 송신하는 teleop_twist_keyboard는 gazebo node로 데이터를 보내고 있으며, 따라서 gazebo는 이 데이터를 통해 실제 로봇을 움직이게 되는 것입니다.

![lec2_4.png](/kr/ros_basic_noetic/images3/lec2_4.png?height=200px)

- 특정 Node에 대해서 더 자세한 정보를 얻고 싶다면, `rosnode info` 커맨드를 사용합니다.

```bash
$ rosnode info /base_controller_spawner
--------------------------------------------------------------------------------
Node [/base_controller_spawner]
Publications:
 * /rosout [rosgraph_msgs/Log]

Subscriptions:
 * /clock [rosgraph_msgs/Clock]

Services:
 * /base_controller_spawner/get_loggers
 * /base_controller_spawner/set_logger_level

contacting node http://192.168.55.236:33811/ ...
Pid: 63764
Connections:
 * topic: /rosout
    * to: /rosout
    * direction: outbound (43329 - 192.168.55.236:34456) [10]
    * transport: TCPROS
 * topic: /clock
    * to: /gazebo (http://192.168.55.236:33853/)
    * direction: inbound
    * transport: TCPROS
```

이전 예제들은 일단 종료시킨 뒤, 간단한 새로운 예시를 실행해봅시다.

```bash
# Terminal 1
roscore
# Terminal 2
rosrun roscpp_tutorials talker
# Terminal 3
rosrun roscpp_tutorials listener
# Terminal 4
rqt_graph
```

![talker_listener.gif](/kr/ros_basic_noetic/images3/talker_listener.gif?height=400px)

rqt_graph를 보면 **talker ⇒ listener**로 데이터가 전송되는 것을 알 수 있습니다.

이제, rqt_graph를 보는 것은 익숙해졌지요?

![lec2_5.png](/kr/ros_basic_noetic/images3/lec2_5.png)

첫번째 Gazebo 예시와 다른 점으로 roscore라는 것을 실행해주었습니다.

- roscore는 ROS Master를 실행시키는 명령어로 모든 ROS Node들은 Master에 의해 관리되기 때문에 roscore를 통해 실행시키거나, roslaunch를 사용해야 합니다.
- ROS Master가 실행되고 있지 않다면, 아래와 같이 오류가 발생합니다.

![no_master.gif](/kr/ros_basic_noetic/images3/no_master.gif?height=400px)

---

### ETH Super Mega Bot & ROS Workspace

- ROS는 **catkin**이라는 빌드 시스템을 사용합니다. 기존 c/c++ cross-platform 개발을 경험하셨다면 cmake에 익숙하실텐데, 이와 매우 비슷합니다.
- catkin을 통해 실행 가능한 프로그램 (C/C++의 빌드), 라이브러리, 인터페이스들을 만들 수 있으며, catkin 시스템을 사용하기 위해서는 **workspace**라는 특별한 폴더가 필요합니다.

![lec2_6.png](/kr/ros_basic_noetic/images3/lec2_6.png?height=150px)

- ROS 개발을 하다 보면 여러 프로젝트를 동시에 진행하는 경우가 생깁니다.
- 새로운 작업은 새로운 폴더를 만들어 작업하듯이 ROS 에서도 새로운 프로젝트는 새로운 WorkSpace에서 작업을 수행하는 것이 일반적입니다.
- 새로운 WS로 이동하게 되면 ROS에게 이러한 변화를 알려줘야 하며 이 명령어가 **source devel/setup.bash** 입니다.
- 이번 강의용 WS를 만들어보고, 이후 여러분들만의 WS도 만들어 작업해보세요!

> 일반적으로 ROS의 workspace는 **name_ws**라는 이름을 갖는 것이 일반적이며, 우리는 **catkin_ws**라는 workspace를 만들어보고자 합니다.

- 아래 커멘드 라인들을 따라해주세요

```bash
cd ~/
mkdir -p catkin_ws/src
cd catkin_ws

catkin config --init
catkin build
```

![catkin_ws.gif](/kr/ros_basic_noetic/images3/catkin_ws.gif?height=350px)

- 다음과 같이 **build, devel, src, log** 폴더가 만들어집니다.

![lec2_7.png](/kr/ros_basic_noetic/images3/lec2_7.png?height=200px)

- ROS 코드들은 모두 src 폴더 안에 위치하게 됩니다.
- src 폴더 내부에서 코드 개발 ⇒ catkin을 사용한 빌드 ⇒ build 폴더 내부에 실행 가능한 프로그램 생성의 순서로 개발이 이루어집니다.

> 실습을 통해 개발 프로세스에 익숙해져봅시다.

1. 아래 폴더를 catkin_ws/src 안에 압축 해제합니다.

{{% attachments title="Example Packages" /%}}

![lec2_8.png](/kr/ros_basic_noetic/images3/lec2_8.png?height=200px)

2. 터미널 프로그램을 실행시키고 아래 커멘드 라인을 따라합니다.

```bash
catkin build smb_description
catkin build smb_gazebo
catkin build smb_control

source devel/setup.bash
```

![catkin_build.gif](/kr/ros_basic_noetic/images3/catkin_build.gif?height=350px)

{{% notice tip %}}
source로 시작하는 마지막 라인은 새로운 빌드 후에 항상 실행해줘야 합니다. 1강을 잘 따라했다면 **sds**라는 단축어로 사용이 가능합니다.
{{% /notice %}}

3. 예제 프로그램을 실행시킵니다.

```bash
roslaunch smb_gazebo smb_gazebo.launch
```

![lec2_9.png](/kr/ros_basic_noetic/images3/lec2_9.png?height=300px)

{{% notice note %}}
실행 시 붉은 에러 메세지가 나오지만 동작만 된다면 문제 없습니다.
{{% /notice %}}

4. 이 로봇을 한번 움직여 볼까요? - teleop 실행

```bash
rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

- 파일 구조 관점에서, ROS Application은 여러 **Package**들로 이루어집니다. 이들 Package가 소스 파일을 담고 있고, catkin이 이들을 빌드하여 실행 프로그램들을 만들었습니다.
- 제가 공유한 압축 파일 안에도 3개의 Package가 포함되어있던 것이며, ROS 개발자들은 자신들의 로봇 Package를 개발하고 공유합니다.

---

## Package 생성 실습

> Package를 생성하는 방법은 다음과 같습니다.

```bash
cd <your-ws>/src
catkin_create_pkg <package_name> [depend1] [depend2] [depend3]
```

my_first_package라는 package를 시험삼아 생성해봅시다.

```
# exameple

$ catkin_create_pkg my_first_package rospy std_msgs
Created file my_first_package/package.xml
Created file my_first_package/CMakeLists.txt
Created folder my_first_package/src
Successfully created files in /home/kimsooyoung/catkin_ws/src/my_first_package. Please adjust the values in package.xml.

```

![lec2_10.png](/kr/ros_basic_noetic/images3/lec2_10.png?height=200px)

depend에는 해당 패키지의 의존성 패키지들이 나열되며, rospy는 파이썬을 통해 ROS를 사용하기 위한 의존성입니다.

미리 제공되었던 Package, smb_gazebo를 살펴봅시다.

![lec2_11.png](/kr/ros_basic_noetic/images3/lec2_11.png?height=200px)

Gazebo 실행에 필요한 모델 파일과 환경 파일 등 기능별 정리된 모습을 볼 수 있습니다.

이렇게 Package를 잘 구성해두면 이후 코드의 관리에도 편리하다는 장점이 있습니다.

---

**참고자료**

- [http://wiki.ros.org/ko/ROS/Tutorials/CreatingPackage](http://wiki.ros.org/ko/ROS/Tutorials/CreatingPackage)
- [https://rsl.ethz.ch/education-students/lectures/ros.html](https://rsl.ethz.ch/education-students/lectures/ros.html)
- [https://www.clearpathrobotics.com/assets/guides/noetic/ros/Drive a Husky.html](https://www.clearpathrobotics.com/assets/guides/noetic/ros/Drive%20a%20Husky.html)
