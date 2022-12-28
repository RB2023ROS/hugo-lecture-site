---
title: "Lecture7 - "
date: 2022-12-25T13:36:23+09:00
draft: false
---

강의 초반, 다양한 rqt tool들을 살펴본 바 있습니다. 이제는 Topic과 Service에 모두 익숙해졌기 때문에, rqt의 많은 기능들을 사용할 수 있습니다. 다시 한 번 rqt를 살펴보면서 편리한 툴들의 사용법을 익혀봅시다.

### Message Publisher & Topic Monitor

강의자를 따라 다음과 같이 화면을 구성합니다.

1. **plugins ⇒ topics ⇒ Message Publisher**
2. **pulgins ⇒ topics ⇒ Topic Monitor**

![rqt_topic.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a43eac4f-177b-4e78-a9d9-c10080fa0a9b/rqt_topic.gif)

message publisher를 사용하면 코딩 없이 cmd_vel을 publish가 가능합니다.

Topic Msg에 원하는 데이터를 채워넣은 뒤, 주기를 선택한 후 체크박스를 클릭하면 로봇이 움직이기 시작합니다.

![rqt_robot_move.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1e3f188b-d11b-4210-8cc8-0e6728f06c9e/rqt_robot_move.gif)

Topic Monitor를 사용하면, 여러 데이터들을 효과적으로 모니터링 가능합니다.

Topic Publisher와 동일하게 체크박스를 눌러 topic을 선택한 뒤, 변하는 데이터를 확인해봅시다.

![rqt_topic_mnt.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d0acbd49-e200-4ec2-b49b-07afb01fb0ce/rqt_topic_mnt.gif)

코딩 없이 간단히 값의 확인과 동작 여부를 확인할 수 있는 툴들이었습니다.

---

### RQT Multiplot

수치 데이터를 그래프로 보고싶은 경우 rqt의 multiplot이 유용하게 사용됩니다.

```bash
rosrun rqt_multiplot rqt_multiplot
```

/odom topic의 X,Y position을 기준으로 그래프를 그려보도록 하겠습니다. 아래의 gif를 통해 모든 과정을 기록하였으니 차근차근 따라와주세요.

![rqt_multiplot.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/13738b3b-f33c-44e2-957a-eb848d62ed8c/rqt_multiplot.gif)

### RQT Console

지금까지 ROS의 콘솔 로그를 위해 rospy.loginfo()를 사용하였습니다. 사실 ROS에는 loginfo말고도 다양한 level의 logger level이 존재합니다. 실습을 통해 살펴봅시다.

- rospy logger level

```bash
# Terminal 1
roscore
# Terminal 2
rosrun my_first_package logger_level.py
```

![logger.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ca24034c-6db6-4820-af39-d1904516f822/logger.gif)

- 코드의 내용과 함께 예시를 살펴봅시다.

```python
		def hello_du(self, event=None):
        hello_du = f"hello du {rospy.get_time()}, counter: {self.counter_}"
        rospy.logdebug(hello_du)
        rospy.loginfo(hello_du)
        rospy.logwarn(hello_du)
        rospy.logerr(hello_du)
        rospy.logfatal(hello_du)
        self.counter_ += 1
```

ROS는 총 5가지의 logger level을 갖추고 있으며, Debug 부터 Fatal로 갈수록 더 높은 level을 갖는다고 보시면 됩니다. Info level 부터 콘솔 출력이 이루어지며, Python의 stdout를 사용합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0c3aea3c-24c9-4665-bf9d-bce6be4a728c/Untitled.png)

- image from : [51CTO](https://blog.51cto.com/u_7784550/5675313)

상황에 따라 각기 다른 level의 log를 사용하도록 하면, 실제 로봇 개발시에도 큰 도움이 됩니다.

rqt에는 이러한 다양한 level을 갖는 ros의 log를 필터링하는 rqt console이라는 툴이 있습니다. 사용 방법을 함께 알아봅시다.

![rqt_logger.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3d019893-51d1-4a90-bba7-f88e84e0ba17/rqt_logger.gif)

그 밖에도 수많은 rqt 도구들이 있지만, 모두 살펴보는 대신 링크로 대체하겠습니다.

[Wiki](http://wiki.ros.org/rqt)

### ROS Bags

rqt 툴에 속하지는 않지만, 개발 시 매우 유용한 ROS의 기능을 하나 더 소개시켜드리고자 합니다.

**rosbag**은 프로그램 동작 중 발생하는 message 데이터를 기록하고 복기할 수 있게 해주는 툴입니다. 로봇 알고리즘을 개발할 때, 같은 상황에 대해 성능을 비교하는 경우, 혹은 교육 목적으로 데이터셋을 제공하는 경우 등에 매우 유용하게 사용할 수 있습니다.

rosbag 사용법을 함께 실습해보겠습니다.

- smb gazebo를 실행합니다.

```bash
roslaunch smb_gazebo smb_gazebo.launch world:=big_map_summer_school
```

- rosbag의 사용 시 여러 옵션들이 있습니다.
  - **-o** 옵션으로 rosbag의 이름을 지정합니다.
  - **-a** 옵션 사용 시 모든 topic을 저장합니다.
  - rosbag의 종료는 **ctrl + c**를 사용합니다.

```bash
rosbag record -o first_rosbag /scan /tf /tf_static
```

/tf와 /tf_static은 왜 저장하는 것일까요? 생각해봅시다.

- rosbag info를 통해 저장을 마친 rosbag의 정보를 조회할 수 있습니다.

```bash
$ rosbag info first_rosbag_<time-format-sth>.bag
path:        first_rosbag_2022-12-27-15-51-55.bag
version:     2.0
duration:    4.8s
start:       Jan 01 1970 09:07:35.31 (455.31)
end:         Jan 01 1970 09:07:40.12 (460.12)
size:        83.8 KB
messages:    49
compression: none [1/1 chunks]
types:       sensor_msgs/LaserScan [90c7ef2dc6895d81024acba2ac42f369]
topics:      /scan   49 msgs    : sensor_msgs/LaserScan
```

- 저장 완료된 rosbag을 다시 복기해봅시다.

```bash
$ rosbag play first_rosbag_2022-12-27-15-56-23.bag
[ INFO] [1672124296.822088842]: Opening first_rosbag_2022-12-27-15-56-23.bag

Waiting 0.2 seconds after advertising topics... done.
...
```

- rviz를 통해 시각화까지 해봅시다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edf02019-06c3-462d-9124-4d99aa371dcc/Untitled.png)

rosbag은 기본적으로 topic을 저장합니다.

rviz 화면을 살펴보면 아래와 같은 Warning이 발생할 것입니다. 그런데 이 문구, 익숙하지 않나요?

```bash
[ WARN] [1672124550.336965013]: TF_REPEATED_DATA ignoring data with redundant timestamp for frame LF_WHEEL at time 472.056000 according to authority unknown_publisher
[ WARN] [1672124550.336981403]: TF_REPEATED_DATA ignoring data with redundant timestamp for frame LH_WHEEL at time 472.056000 according to authority unknown_publisher
[ WARN] [1672124550.336991753]: TF_REPEATED_DATA ignoring data with redundant timestamp for frame RF_WHEEL at time 472.056000 according to authority unknown_publisher
[ WARN] [1672124550.337003964]: TF_REPEATED_DATA ignoring data with redundant timestamp for frame RH_WHEEL at time 472.056000 according to authority unknown_publisher
[ WARN] [1672124550.356035553]: TF_REPEATED_DATA ignoring data with redundant timestamp for frame base_link at time 472.076000 according to authority unknown_publisher
[ WARN] [1672124550.356945321]: TF_REPEATED_DATA ignoring data with redundant timestamp for frame LF_WHEEL at time 472.076000 according to authority unknown_publisher
[ WARN] [1672124550.356960231]: TF_REPEATED_DATA ignoring data with redundant timestamp for frame LH_WHEEL at time 472.076000 according to authority unknown_publisher
[ WARN] [1672124550.356970561]: TF_REPEATED_DATA ignoring data with redundant timestamp for frame RF_WHEEL at time 472.076000 according to authority unknown_publisher
```

### ROS Time

일전 강의에서 언급한 바와 같이, 이번 강의에 **ROS의 시간 체계**에 대해서 다잡고 가고자 합니다.

- rospy.loginfo를 통해 콘솔에 출력되는 시간과 같이 ROS에서 기본적으로 사용되는 시간의 기준은 PC의 Clock입니다. (이를 wall timer라고 부릅니다.)
- ROS 프로그램은 일정 주기를 갖고 무한히 반복되는 상황이 잦습니다. 이때 사용하는 주기가 정확해야 할 것입니다.
- 우리는 2023년 00월 00일이라는 시간체계를 사용하지만, Gazebo와 같은 시뮬레이션 툴은 시작되는 시점이 곧 0분 0초가 됩니다. 이러한 시간의 차이로 인해 warning과 error가 빈번하게 발생합니다.

그럼, 실질적으로 ROS에서 시각과 주기, 시간은 어떻게 다루는지 예시와 함께 python 코드를 살펴봅시다.

- **ros_time.py**

```bash
$ rosrun my_first_package ros_time.py
[INFO] [1672127477.793024]: Current time 1672127477 792964935
[INFO] [1672127477.793593]: Current time to_sec 1672127477
[INFO] [1672127477.794082]: Past time 1672127477 292964935
[INFO] [1672127477.893132]: Current time 1672127477 893082141
[INFO] [1672127477.893684]: Current time to_sec 1672127477
[INFO] [1672127477.894148]: Past time 1672127477 393082141
...
```

코드는 다음 위치에서 확인이 가능합니다.

[https://github.com/RB2023ROS/du2023-ros1/blob/main/my_first_package/scripts/ros_time.py](https://github.com/RB2023ROS/du2023-ros1/blob/main/my_first_package/scripts/ros_time.py)

- rospy Time instance

rospy에서는 `Time`이라는 클래스로 시간을 표현합니다. 가장 많이 사용되는 현재 시간은 `rospy.Time.now()` 로 파악할 수 있으며, 이는 sec와 nsec등의 클래스 변수를 갖고 있습니다.

```python
def hello_du(self, event=None):

    now  = rospy.Time.now()
    seconds = now.to_sec()

    rospy.loginfo("Current time %i %i", now.secs, now.nsecs)
    rospy.loginfo("Current time to_sec %i", seconds)
```

- rospy Time Duration

시간 간격을 나타내는 클래스로 Duration이 사용되며, Time 인스턴스와 +,- 연산이 가능합니다.

```bash
		delta = rospy.Duration(0.5)
    past  = now - delta
    rospy.loginfo("Past time %i %i", past.secs, past.nsecs)
```

- rospy Rate

while loop와 Rate를 사용하여 일정 주기다마 반복되는 구현이 가능합니다. 이때 사용되는 시간 기준은 PC의 Clock입니다.

```bash
		r = rospy.Rate(10) # 10hz
    while not rospy.is_shutdown():
        time_ex.hello_du()
        r.sleep()
```

### Topic vs Service and Action

지금까지 ROS의 통신 메커니즘으로 Topic과 Service에 대해 배워보았습니다. 그런데, 사실 ROS의 통신 메커니즘에는 Action이라는 한가지가 더 있습니다.

Action은 Topic과 Service 모두의 특징을 갖고 있는 진보된 통신 메커니즘입니다. Action은 Feedback이라는 것으로 Goal Request 이후 계속적인 데이터 송수신이 가능합니다. Action은 ROS 2 강의에서 살펴볼 예정으로 어떻게 사용될 수 있을지 한번 고민해보세요.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/583e145d-cfd1-4d08-96ad-c3d459a79172/action.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/583e145d-cfd1-4d08-96ad-c3d459a79172/action.gif)

image from : [docs.rog.org](https://docs.ros.org/en/foxy/Tutorials/Understanding-ROS2-Actions.html)

### Deal with Open Source Projects

이번 시간에는 오픈 소스 프로젝트를 사용하는 방법에 대해 배워보겠습니다. 보다 실질적인 사용 방법을 보여드리기 위해 저 또한 여러분들과 같은 상황에서 처음부터 하나씩 같이 해보겠습니다.

오늘 데모하고자 하는 로봇 소프트웨어는 **드론 시뮬레이션**입니다. 지금까지 지상을 움직이는 바퀴 로봇만을 다루었기 때문에 새로운 플렛폼을 동작시켜보고자 합니다.

- 항상 시작은 구글링부터!!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b5c77f85-35a8-40b9-a841-ff18815c14c2/Untitled.png)

- 검색 결과 원하는 패키지를 찾은 것 같습니다.

[https://github.com/RAFALAMAO/hector-quadrotor-noetic](https://github.com/RAFALAMAO/hector-quadrotor-noetic)

**목적에 부합하는 오픈소스를 찾기 위해서 아래와 같은 기본적인 내용을 고려해야 합니다.**

- 버전 호환성
- 구체적인 목표에 부합하는지
- Star, Fork를 통해 검증된 소스코드임을 확인
- Issue를 통해 사용 중 문제가 있지는 않은지

Readme를 따라 package build를 진행하고 최초 실행을 해보겠습니다.

```python
# Terminal 1
roslaunch hector_quadrotor_gazebo quadrotor_empty_world.launch

# Terminal 2
rosrun hector_ui ui_hector_quad.py
```

![quadrotor.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3b620eb4-c594-4c38-a794-95060f601583/quadrotor.gif)

동작에는 문제가 없어보입니다. 그럼 이 프로젝트가 내부적으로 어떻게 구현되어있는지 분석해봅시다.

```python
rqt_graph
```

![quadrotor.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8de9907f-f79e-485b-b7f0-6181621b069a/quadrotor.png)

로봇의 위치를 알려주는 topic인 **/ground_truth/state**와 **/ground_truth_to_tf/pose**를 파악할 수 있습니다. gazebo에서 물체의 절대적인 위치를 알려주기 때문에 이를 ground truth라고 부르고 있습니다.

- 이번에는 조종 프로그램의 소스코드를 확인해봅시다. - hector_quadrotor_noetic/hector_ui/src

```python
#Callback de pose y orientacion simulador
def pose_callback(data):
    x_p.set("{0:.2f}".format(data.pose.pose.position.x))
    y_p.set("{0:.2f}".format(data.pose.pose.position.y))
    z_p.set("{0:.2f}".format(data.pose.pose.position.z))

def rot_callback(data):
    z_o.set("{0:.2f}".format( math.degrees(quaterionToRads(data)) ))

rospy.init_node('HectorQ_GUI', anonymous=False)
#Subscribers
posicionLider_sub = rospy.Subscriber("/ground_truth/state", Odometry , pose_callback)
orientaLider_sub = rospy.Subscriber("/ground_truth_to_tf/pose", PoseStamped , rot_callback)
```

두 종류의 subscriber가 존재하며 각각 UI의 상태를 업데이트하는 것 같이 보입니다.

- 이번에는 launch file을 분석해봅시다 - hector_quadrotor/hector_quadrotor_gazebo/launch/quadrotor_empty_world.launch

```xml
<?xml version="1.0"?>

<launch>
  <arg name="paused" default="false"/>
  <arg name="use_sim_time" default="true"/>
  <arg name="gui" default="true"/>
  <arg name="headless" default="false"/>
  <arg name="debug" default="false"/>

  <include file="$(find gazebo_ros)/launch/empty_world.launch">
    <arg name="paused" value="$(arg paused)"/>
    <arg name="use_sim_time" value="$(arg use_sim_time)"/>
    <arg name="gui" value="$(arg gui)"/>
    <arg name="headless" value="$(arg headless)"/>
    <arg name="debug" value="$(arg debug)"/>
  </include>

  <include file="$(find hector_quadrotor_gazebo)/launch/spawn_quadrotor.launch" />
    <arg name="model" value="$(find hector_quadrotor_description)/urdf/quadrotor_hokuyo_utm30lx.gazebo.xacro"/>
</launch>
```

gazebo_ros의 empty_world.launch, hector_quadrotor_gazebo의 spawn_quadrotor.launch가 실행되며, model이라는 argument를 갖습니다.

spawn_quadrotor.launch에는 다음과 같은 node들이 실행됩니다.

- spawn_model
- robot_state_publisher
- ground_truth_to_tf
- controller.launch ⇒ controller_spawner

이렇게 rqt 툴들과 코드의 구조를 파헤치면서 전체 구조를 파악할 수 있으며 원하는 부분만을 고치면서 Package를 발전시켜나가는 것입니다.

launch file의 응용을 실습해봅시다. 일전 배워본 husky gazebo와 hector gazebo를 함께 사용해보는 것입니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3a456b55-a456-4eb1-a9fc-3e3f68089926/Untitled.png)

- [husky gazebo](https://github.com/husky/husky/blob/noetic-devel/husky_gazebo/launch/husky_empty_world.launch) 는 다음과 같은 내용을 담고 있었습니다.
  - gazebo_ros를 실행시키고
  - husky model을 spawn하는 또다른 launch file을 include 하였습니다.

```xml
<launch>

  <arg name="world_name" default="worlds/empty.world"/>

  <include file="$(find gazebo_ros)/launch/empty_world.launch">
    <arg name="world_name" value="$(arg world_name)"/> <!-- world_name is wrt GAZEBO_RESOURCE_PATH environment variable -->
    <arg name="paused" value="false"/>
    <arg name="use_sim_time" value="true"/>
    <arg name="gui" value="true"/>
    <arg name="headless" value="false"/>
    <arg name="debug" value="false"/>
  </include>

  <include file="$(find husky_gazebo)/launch/spawn_husky.launch">
  </include>

</launch>
```

- 그럼, 겹치는 부분을 제외하고 quadrotor_empty_world.launch의 내용을 추가하여 새로운 launch file을 만들어 봅시다. (hetero_spawn.launch 생성)

```xml
<launch>

  <include file="$(find husky_gazebo)/launch/husky_empty_world.launch" />

  <include file="$(find hector_quadrotor_gazebo)/launch/spawn_quadrotor.launch" />
    <arg name="model" value="$(find hector_quadrotor_description)/urdf/quadrotor_hokuyo_utm30lx.gazebo.xacro"/>

</launch>
```

- 아래와 같은 오류가 발생하네요, 이는 robot_state_publisher node가 중복되기 때문에 발생하는 문제입니다.

```bash
RLException: roslaunch file contains multiple nodes named [/robot_state_publisher].
Please check all <node> 'name' attributes to make sure they are unique.
Also check that $(anon id) use different ids.
The traceback for the exception was written to the log file
```

- 이를 해결하기 위해서, 저는 다른 launch file(spawn_two_quadrotors.launch)을 참고해보았습니다. group 태그를 사용하면 같은 로봇의 중복 선언을 namespace를 통해 구분할 수 있게됩니다.

```xml
<?xml version="1.0"?>
<launch>
   <arg name="model" default="$(find hector_quadrotor_description)/urdf/quadrotor.gazebo.xacro" />

   <group ns="uav1">
     <include file="$(find hector_quadrotor_gazebo)/launch/spawn_quadrotor.launch">
       ...
   </group>

   <group ns="uav2">
     <include file="$(find hector_quadrotor_gazebo)/launch/spawn_quadrotor.launch">
       ...
   </group>
</launch>
```

- 이제, hetero_spawn.launch를 최종 수정해봅시다.

```xml
<launch>

  <include file="$(find husky_gazebo)/launch/husky_empty_world.launch" />

  <arg name="model" default="$(find hector_quadrotor_description)/urdf/quadrotor.gazebo.xacro" />
  <group ns="uav1">
    <include file="$(find hector_quadrotor_gazebo)/launch/spawn_quadrotor.launch">
      <arg name="name" value="uav1" />
      <arg name="tf_prefix" value="uav1" />
      <arg name="model" value="$(arg model)" />
      <arg name="z" value="1.0" />
    </include>
  </group>

</launch>
```

실행 후 topic을 조회해보면, 아래와 같이 quadrotor의 topic 앞에 namespace가 추가되어 있는 모습이 확인 가능합니다. 여러 로봇을 사용하게 되면 /cmd_vel, /scan등 사용하는 topic의 이름이 중복될 수 있어 **namespace**를 추가 설정하는 것이 일반적입니다.

```bash
$ rostopic list
/clock
/cmd_vel
/husky_velocity_controller/cmd_vel
/husky_velocity_controller/odom
/husky_velocity_controller/parameter_descriptions
/husky_velocity_controller/parameter_updates
/imu/data
/odometry/filtered
...
/uav1/aerodynamics/wrench
/uav1/cmd_vel
/uav1/command/motor
/uav1/command/twist
/uav1/command/wrench
/uav1/fix_velocity
...
```

마지막으로, 로봇을 제어해보면서 이번 세션을 마무리해보겠습니다. gazebo의 world를 바꾸거나, husky가 아닌 smb로 모델을 바꾸는 등 응용 예시들을 직접 해보면서 launch file의 사용에 익숙해지시기 바랍니다.

```bash
# husky 제어
rosrun teleop_twist_keyboard teleop_twist_keyboard.py
********************************# quadrotor 제어
rosrun hector_ui ui_hector_quad_leader.py
```

![hetero.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4044b7e6-cb3c-40bd-ad3d-b39cc89ad708/hetero.gif)

---

### **ROS Custom Interfac**es

ROS에서 기본 제공되는 msg와 srv도 훌륭하지만, 상황에 따라 나만의 custom msg/srv를 사용해야 하는 경우가 있습니다. 이번 시간에는 custom interface를 만들어보고, 사용해보겠습니다.

드론의 이착륙을 제어하는 srv를 만들어보고자 합니다.

- 이륙/착륙을 구분하는 string이 필요할 것이며,
- 기준은 시간을 사용하고자 합니다. ( ex - 2초간 이륙 )
- service request는 성공 여부인 bool type으로 해보겠습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e438d786-8f8e-47bd-975d-2cefca20c418/Untitled.png)

- custom interface를 만들기 위해 저는 별도의 package를 생성하였습니다.

```xml
cd ~/catkin_ws/src/du2023-ros1
catkin_create_pkg custom_interfaces
```

- package 내부에 msg 혹은 srv라는 폴더를 만들고, custom interface를 정의하는것이 추천됩니다.

```bash
cd custom_interfaces
mkdir srv

# QuadrotorControl.srv 생성
string command
uint8 seconds
---
bool success
```

- custom_interfaces package의 package.xml을 수정합니다.

```xml
  <build_depend>message_generation</build_depend>
  <exec_depend>message_runtime</exec_depend>
```

- custom_interfaces package의 CMakeLists.txt를 수정합니다.

```bash
# 1. find_package 수정
find_package(catkin REQUIRED COMPONENTS
   roscpp
   rospy
   std_msgs
   message_generation
)

# 2. catkin_package 주석 해제 후 수정
catkin_package(
  ...
  CATKIN_DEPENDS message_runtime ...
  ...)

# 3. add_service_files에 파일 반영
add_service_files(
  FILES
  QuadrotorControl.srv
)

# 4. generate_messages 주석 해제 후 수정
generate_messages(
  DEPENDENCIES
  std_msgs  # Or other packages containing msgs
)
```

- custom interface를 빌드하고 생성을 확인해봅시다.

```bash
$ catkin build custom_interfaces

$ rossrv show custom_interfaces/QuadrotorControl
string command
uint8 seconds
---
bool success
```

지금 생성한 QuadrotorControl은 catkin_ws에서만 사용 가능한 srv라는 점에 유의합니다. 다른 workspace에서는 QuadrotorControl에 대해 알 길이 없습니다.

---

### **Custom Interfaces 사용해보기**

- 작성한 QuadrotorControl를 사용하여 Service Server를 만들어봅시다! 정해진 시간동안 takeoff와 land 움직임을 수행하는 Service입니다.

```bash
# Terminal 1
roslaunch hector_quadrotor_gazebo quadrotor_empty_world.launch

# Terminal 2
rosrun py_service_pkg quadrotor_custom_srv.py

# Terminal 3
rqt
```

![custom_srv.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2d0b6a26-beae-4076-8726-97b2ba1af5af/custom_srv.gif)

소스 코드는 [링크](https://github.com/RB2023ROS/du2023-ros1/blob/main/py_service_pkg/scripts/quadrotor_custom_srv.py)에서 확인이 가능합니다.

- custom_interfaces package에서 **QuadrotorControl srv**를 import 하며, 로봇의 제어를 위해 Twist msg도 import 하였습니다.

```python
import rospy

from geometry_msgs.msg import Twist
from custom_interfaces.srv import QuadrotorControl, QuadrotorControlResponse
```

- Service Server와 Topic Publisher를 생성합시다.

```python
class QuadRotorUpDown(object):

    def __init__(self):
        self.cmd_vel_pub_ = rospy.Publisher("cmd_vel", Twist, queue_size=10)
        self.stop_server_ = rospy.Service("up_down", QuadrotorControl, self.up_down_cb)

        self.twist_msg_   = Twist()
        self.response_    = QuadrotorControlResponse()

        rospy.loginfo("Quadrotor Up-Down Server Started")
```

- callback 함수인 up_down_cb입니다. command가 land/takeoff일 때의 경우를 나누고, 그 이외의 입력은 오류로 판명합니다.

```python
def up_down_cb(self, request):

    if request.command == "land":
        self.twist_msg_.linear.z  = -0.5
        self.response_.success = True
    elif request.command == "takeoff":
        self.twist_msg_.linear.z  = 0.5
        self.response_.success = True
    else:
        rospy.logwarn("Unknown Command")
        self.response_.success = False
        return self.response_
```

- request의 seconds 시간동안 로봇이 움직여야 할 것이며, 이를 위해 now를 갱신하며 지나간 시간을 계속해서 tracking 합니다.

```python
		start = rospy.Time.now()
    now = rospy.Time.now()
    while (now - start).secs < request.seconds:
        now = rospy.Time.now()
        self.cmd_vel_pub_.publish(self.twist_msg_)
```

- 모든 동작이 완료된 이후에는 로봇을 다시 정지시킵니다.

```python
		rospy.loginfo(f"{request.command} done, quadrotor stop")
    self.twist_msg_.linear.z = 0.0
    self.cmd_vel_pub_.publish(self.twist_msg_)

    return self.response_
```

**참고자료**

- [http://wiki.ros.org/rospy/Overview/Time](http://wiki.ros.org/rospy/Overview/Time)
- [https://github.com/RAFALAMAO/hector-quadrotor-noetic](https://github.com/RAFALAMAO/hector-quadrotor-noetic)
- [https://docs.ros.org/en/foxy/index.html](https://docs.ros.org/en/foxy/index.html)
- [http://wiki.ros.org/ROS/Tutorials/CreatingMsgAndSrv](http://wiki.ros.org/ROS/Tutorials/CreatingMsgAndSrv)
