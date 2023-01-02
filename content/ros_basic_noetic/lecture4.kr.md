---
title: "Lecture4 - ROS Launch, RViz"
date: 2022-12-25T13:36:28+09:00
draft: false
---

## ROS Launch

> 일전 예시 실행에서 다음과 같은 커멘드 라인을 사용했었습니다.

```bash
roslaunch smb_gazebo smb_gazebo.launch
```

**roslaunch**란, 다수의 ROS Node들을 한번에 실행할 수 있도록 해주는 툴 입니다.

roslaunch를 사용하기 위해서는 **xml**이라는 포멧을 사용하는 **launch file**이 있어야 하며, 이는 보통 패키지의 launch 폴더에 위치하고 있습니다.

![lec3_0.png](/kr/ros_basic_noetic/images4/lec3_0.png?height=200px)

> launch file의 구조를 파악해봅시다.

- launch파일은 `xml`이라는 문법을 사용합니다. html을 사용해보셨다면 아시겠지만, <>를 이용하여 라인을 구분하는 포멧입니다.
- 한 라인에서 끝나는 경우 />로 맺을 수 있지만, 여러 라인이 필요한 경우에는 여는 태그와 닫는 태그를 사용하여 구분합니다.

```xml
<tag />

or

<tag (value)>
...
</tag>
```

launch file은 시작과 끝, **<launch> </launch>** 태그로 감싸집니다.

- **node** 태그는 실행되는 ROS Node를 지칭합니다.
- **name** 태그는 node를 실행할 때의 이름을 설정하는 부분으로 자유롭게 지정 가능합니다.
- **pkg** 태그에는 해당 node가 속해있는 package를 적습니다.
- **type** 태그에는 실행가능한 파일, 혹은 프로그램을 적게 되며, c++의 경우 빌드된 프로그램, 파이썬의 경우 파이썬 파일이 됩니다.
- **output** 태그는 로그가 출력되는 위치를 지정하며 screen일 시 터미널에 출력됩니다.

```xml
<launch>
  <node name="listener" pkg="rospy_tutorials" type="listener.py" output="screen"/>
  <node name="talker" pkg="rospy_tutorials" type="talker.py" output="screen"/>
</launch>
```

이번에는 `smb_gazebo.launch`를 살펴봅시다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<launch>
  <!-- GAZEBO ARGUMENTS -->
  <!-- Run Gazebo headless -->
  <arg name="headless"                              default="false"/>
  <arg name="model_path"                            default="$(find smb_gazebo)/"/>
  <arg name="robot_namespace"                       default=""/>
  <arg name="robot_model_name"                      default="smb"/>
  <arg name="enable_ekf"                            default="true"/>

  ...

  <!-- Load Gazebo world -->
  <include file="$(find gazebo_ros)/launch/empty_world.launch">
    <env name="GAZEBO_MODEL_PATH" value="$(arg model_path)"/>
    <arg name="world_name"        value="$(arg world_file)"/>
    <arg name="paused"            value="$(arg paused)"/>
    <arg name="use_sim_time"      value="$(arg use_sim_time)"/>
    <arg name="gui"               value="$(arg run_gui)"/>
    <arg name="headless"          value="$(arg headless)"/>
    <arg name="debug"             value="$(arg debug)"/>
    <arg name="verbose"           value="$(arg verbose)"/>
  </include>
```

**arg**란, argument의 약자이며 launch 파일에서 인자로 작용하는 일종의 변수입니다.

- arg 태그를 통해 argument를 선언하고 **default**를 통해 초기값을 정할 수 있습니다.
- argument의 선언 후 사용은 **$(arg "argument name")** 입니다.
- roslaunch 시 argument를 바꿔 실행이 가능합니다.

예를 들어, smb_gazebo.launch 실행 시. world를 바꾸어 아래와 같이 사용이 가능합니다.

```xml
roslaunch smb_gazebo smb_gazebo.launch world:=big_map_summer_school
```

![lec3_1.png](/kr/ros_basic_noetic/images4/lec3_1.png?height=400px)

{{% notice note %}}
초기 실행 시 시간이 다소 걸릴 수 있습니다.
{{% /notice %}}

현재 예시에서 제공되는 world는 다음 3가지 입니다.

![lec3_2.png](/kr/ros_basic_noetic/images4/lec3_2.png?height=200px)

```bash
roslaunch smb_gazebo smb_gazebo.launch
```

![lec3_3.png](/kr/ros_basic_noetic/images4/lec3_3.png?height=400px)

제작한 launch file을 다시 다른 launch file에서 불러오는 경우가 더러 있습니다.

이때, **include** 태그를 사용하며, 패키지 단위를 기반으로 파일의 경로를 가져오게 됩니다.

include하는 launch file의 내부에도 여러 argument들이 있을 것입니다. 이들은 arg 태그를 통해 접근할 수 있습니다.

```xml
<include file="$(find gazebo_ros)/launch/empty_world.launch">
  <env name="GAZEBO_MODEL_PATH" value="$(arg model_path)"/>
  <arg name="world_name"        value="$(arg world_file)"/>
  <arg name="paused"            value="$(arg paused)"/>
  <arg name="use_sim_time"      value="$(arg use_sim_time)"/>
  <arg name="gui"               value="$(arg run_gui)"/>
  <arg name="headless"          value="$(arg headless)"/>
  <arg name="debug"             value="$(arg debug)"/>
  <arg name="verbose"           value="$(arg verbose)"/>
</include>
```

html과 마찬가지로 `<!--  -->` 사이에 오는 코드는 주석으로 무시됩니다.

단, 주의사항 하나 있습니다. launch 파일을 사용하다보면 `--`가 종종 쓰이곤 하는데요. 이 경우 주석에 오류가 나니 주의하시기 바랍니다.

![lec3_4.png](/kr/ros_basic_noetic/images4/lec3_4.png?height=400px)

> Launch File을 다루는 연습을 해봅시다.

**smb_gazebo.launch를 다음과 같이 수정합니다.**

1. world를 big_map_summer_school로 수정합니다.
2. 로봇이 등장하는 위치를 다음과 같이 수정합니다.
   - xyz : (-0.5, -1.0, 0.4)
   - yaw angle : 90도 (3.1415를 1 radian으로 잡습니다.)
3. launch file에 rospy_tutorials Package에 있는 talker Node를 추가합니다.

![lec3_5.png](/kr/ros_basic_noetic/images4/lec3_5.png?height=400px)

---

## Rqt와 RViz

ROS에는 로봇의 다양한 센서 데이터들을 시각화해주는 3D 툴이 있으며, 이는 RViz라고 불립니다.

Rviz의 사용법을 알아봅시다.

- 예제 실행

```xml
# Terminal 1
roslaunch smb_gazebo smb_gazebo.launch world:=big_map_summer_school

# Terminal 2
rviz
```

사진과 같이 Gazebo와 RViz가 잘 실행된 상황에서 강의를 따라합니다.

![lec3_6.png](/kr/ros_basic_noetic/images4/lec3_6.png?height=400px)

rviz에서 센서 데이터를 시각화하기 전, 우선 어떤 좌표계를 기준으로 시각화할지 설정해주어야 합니다.

같은 센서라도 원점 좌표계에서 본 모습과, 센서 좌표계에서 본 모습이 다르기 때문입니다.

![lec3_7.png](/kr/ros_basic_noetic/images4/lec3_7.png?height=300px)

- img from : [mathworks](https://kr.mathworks.com/help/ros/ug/access-the-tf-transformation-tree-in-ros.html)

이는 RViz의 Fixed Frame에서 설정 가능합니다. (odom으로 설정해보겠습니다.)

![frame_rviz.gif](/kr/ros_basic_noetic/images4/frame_rviz.gif?height=400px)

이제 다양한 시각화 기능들을 사용해보려 합니다.

- 기본적으로 데이터의 추가는 왼쪽 하단 Add 버튼으로 실행합니다.

![lec3_8.png](/kr/ros_basic_noetic/images4/lec3_8.png?height=300px)

- tf 시각화

![rviz_tf.gif](/kr/ros_basic_noetic/images4/rviz_tf.gif?height=300px)

- odometry 시각화

![rviz_odom.gif](/kr/ros_basic_noetic/images4/rviz_odom.gif?height=300px)

- point cloud 시각화

![rviz_point_cloud.gif](/kr/ros_basic_noetic/images4/rviz_point_cloud.gif?height=300px)

> 이렇게 잘 설정해둔 RViz는 config 포멧으로 추출하여 이후에 다시 사용할 수 있습니다.

**File ⇒ Save Config**를 통해 config를 저장하고, Open Config를 통해 저장한 config를 불러올 수 있습니다.

![rviz_config.gif](/kr/ros_basic_noetic/images4/rviz_config.gif?height=300px)

아래와 같이 다양한 Plugin을 통해 여러 센서, 로봇 데이터를 시각화할 수 있으며, 자신만의 Plugin을 제작할 수도 있습니다.

![lec3_9.png](/kr/ros_basic_noetic/images4/lec3_9.png?height=300px)

지금까지 여러분들이 만든 RViz 설정을 저장해보고, launch file에 통합해봅시다.

- RViz의 좌측 상단 File 옵션을 사용하여 config file을 저장합니다.

![lec3_10.png](/kr/ros_basic_noetic/images4/lec3_10.png?height=200px)

- 저장 위치는 smb_gazebo/rviz로 지정하겠습니다. (새롭게 폴더를 만들어주었습니다.)

![lec3_11.png](/kr/ros_basic_noetic/images4/lec3_11.png?height=200px)

- 이제, launch file을 수정합시다. 파일 하단 launch 태그가 닫히기 전 부분에 아래와 같은 라인을 추가합니다.

```xml
	<node name="rviz" pkg="rviz" type="rviz" args="-d $(find smb_gazebo)/rviz/my_config.rviz" />
</launch>
```

이제, 다시 Gazebo launch를 해봅시다.

```xml
roslaunch smb_gazebo smb_gazebo.launch world:=big_map_summer_school
```

---

## Rqt Tools

지금까지 사용한 ROS 툴은 rqt_graph와 RViz가 있었습니다.

사실 ROS에는 수많은 추가 툴들이 존재하며 이들을 묶어 rqt tools라고 부릅니다.

- rqt image view

![lec3_12.png](/kr/ros_basic_noetic/images4/lec3_12.png?height=400px)

image from : [wiki.ros.org](http://wiki.ros.org/rqt_image_view)

- rqt multiplot

![lec3_13.png](/kr/ros_basic_noetic/images4/lec3_13.png?height=400px)

image from : [project march](https://docs.projectmarch.nl/doc/using_the_march_exoskeleton/how_to_view_live_data.html)

- rqt console

![lec3_14.png](/kr/ros_basic_noetic/images4/lec3_14.png?height=400px)

image from : [wiki.ros.org](http://wiki.ros.org/rqt_console)

- rqt robot steering

![lec3_15.png](/kr/ros_basic_noetic/images4/lec3_15.png?height=400px)

- rqt tf tree

![lec3_16.png](/kr/ros_basic_noetic/images4/lec3_16.png?height=400px)

image from : [rqt tf tree](http://wiki.ros.org/rqt_tf_tree)

> 이러한 수많은 툴들이 있어 ROS 개발을 편리하게 해주고 있으며, 함께 ROS를 공부하면서 하나씩 같이 살펴보고 사용해보려 합니다.

---

**참고자료**

- [http://wiki.ros.org/roslaunch](http://wiki.ros.org/roslaunch)
- [https://www.clearpathrobotics.com/assets/guides/noetic/ros/Drive a Husky.html](https://www.clearpathrobotics.com/assets/guides/noetic/ros/Drive%20a%20Husky.html)
- [https://rsl.ethz.ch/education-students/lectures/ros.html](https://rsl.ethz.ch/education-students/lectures/ros.html)
