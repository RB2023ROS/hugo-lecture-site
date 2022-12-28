---
title: "Lecture9 - ROS TF and Examples"
date: 2022-12-25T13:36:19+09:00
draft: false
---

> 대부분의 로보틱스 과정들에서 가장 먼저 다루는 것이 바로 **좌표계 변환(Transformation)** 입니다. 로봇은 수많은 joint와 link로 이루어져 있기 때문에 좌표계를 다루는 일이 매우 빈번합니다.

ROS에서는 **TF**라는 특수한 형태로 이 좌표계와 시간을 함께 다루고 있습니다. 예시와 설명을 통해 ROS의 TF에 대해 배워봅시다 😊

![lec8_0](/kr/ros_basic_noetic/images9/lec8_0.png?height=300px)

- image from : [eth robot dynamics lecture notes](https://ethz.ch/content/dam/ethz/special-interest/mavt/robotics-n-intelligent-systems/rsl-dam/documents/RobotDynamics2017/RD_HS2017script.pdf)

ROS는 오픈소스이니만큼 사용자들이 원하는 기능들에 맞추어 변화가 빠릅니다. 하지만 이것이 단점이 되는 경우도 있는데, 이전 버전과 최신 버전의 호환성 문제가 종종 발생합니다.

tf 또한 tf2로 개편되면서 코드의 수정이 있었으며, 이번 강의에서는 tf2를 중심으로 살펴보겠습니다.

![lec8_1](/kr/ros_basic_noetic/images9/lec8_1.png)

- image from : [wiki.ros](http://wiki.ros.org/tf/Tutorials)

> 예시를 먼저 살펴봅시다.

### tf broadcaster

```python
# Terminal 1
rosrun turtlesim turtlesim_node
# Terminal 2
rosrun py_tf2_tutorial turtle_tf2_broadcaster.py
# Terminal 3
rosrun turtlesim turtle_teleop_key
# Terminal 4
rviz
```

- rviz를 실행한 뒤 아래와 같이 설정합니다.

![tf_broadcaster.gif](/kr/ros_basic_noetic/images9/tf_broadcaster.gif?height=400px)

rviz에서 보이는 세가지 색상의 막대가 바로 **tf** 입니다.

x,y,z의 각 축을 각기 다른 색으로 표현하였으며, 연관된 좌표계끼리는 노란 선을 통해 연결한 모습이 보입니다.

![lec8_2](/kr/ros_basic_noetic/images9/lec8_2.png?height=400px)

- Terminal 1에서 실행시킨 프로그램은 [turtlesim](http://wiki.ros.org/turtlesim)이라는 것으로, 2차원 평면에서 거북이 형태의 로봇을 시뮬레이션한 프로그램입니다.

![lec8_3](/kr/ros_basic_noetic/images9/lec8_3.png?height=300px)

- 이제, Terminal 3에 커서를 두고 거북이를 움직이면서, rviz와 turtlesim의 변화를 살펴보세요.

![tf_ts.gif](/kr/ros_basic_noetic/images9/tf_ts.gif?height=400px)

거북이를 조종함에 따라 변화하는 rviz 화면을 확인할 수 있습니다.

전체 코드는 아래 링크에서 확인할 수 있으며, 지금은 필요한 부분만 집중적으로 분석해보겠습니다.

[https://github.com/RB2023ROS/du2023-ros1/blob/main/py_tf2_tutorial/scripts/turtle_tf2_broadcaster.py](https://github.com/RB2023ROS/du2023-ros1/blob/main/py_tf2_tutorial/scripts/turtle_tf2_broadcaster.py)

---

- tf 또한 하나의 Package입니다. 이에 따라 관련된 python import가 필요합니다.

```python
import rospy

# Because of transformations
import tf_conversions

import tf2_ros
import geometry_msgs.msg
import turtlesim.msg
```

- tf의 데이터 송출은 **broadcast**라고 부릅니다. topic의 publisher와 같이 tf에서는 TransformBroadcaster를 사용하며 **sendTransform**이라는 메소드를 사용합니다.

```python
def handle_turtle_pose(msg, turtlename):
    # tf requires broadbaster
    # Be Careful, !!TF IS NOT A TOPIC!!
    br = tf2_ros.TransformBroadcaster()
		...
		br.sendTransform(t)
```

- TransformBroadcaster가 사용하는 데이터 타입은 **geometry_msgs.msg.TransformStamped**입니다. 해당 데이터 타입에는 3차원 좌표계에서의 위치, 방향, 그리고 **시간**이 포함되어 있습니다.

![lec8_4.png](/kr/ros_basic_noetic/images9/lec8_4.png?height=400px)

image from : [docs.ros.org](http://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/TransformStamped.html)

- 해당 데이터 타입에 적절한 값을 채워넣어준 다음, 최종 **broadcast**가 진행됩니다. 주의할 점으로 쿼터니언 각도 체계를 사용했다는 점입니다.

```python
# prepare tf msg
t = geometry_msgs.msg.TransformStamped()

t.header.stamp = rospy.Time.now()
t.header.frame_id = "world"

t.child_frame_id = turtlename

t.transform.translation.x = msg.x
t.transform.translation.y = msg.y
t.transform.translation.z = 0.0

q = tf_conversions.transformations.quaternion_from_euler(0, 0, msg.theta)

t.transform.rotation.x = q[0]
t.transform.rotation.y = q[1]
t.transform.rotation.z = q[2]
t.transform.rotation.w = q[3]
```

{{% notice tip %}}
쿼터니언은 직관적으로 이해하기는 힘든 각도 체계입니다. 계산의 편의를 위해 다음과 같은 사이트를 사용할 수 있습니다. > [3D Rotation Converter](https://www.andre-gaschler.com/rotationconverter/)

{{% /notice %}}

> tf의 사용 시 주의해야 할 점을 언급하고자 합니다.

- turtle_tf2_broadcaster.py 수정 - Experiment라고 되어 있는 부분을 주석 해제한 다음 다시 실행해봅시다.

```python
def handle_turtle_pose(msg, turtlename):
    # tf requires broadbaster
    # Be Careful, !!TF IS NOT A TOPIC!!
    br = tf2_ros.TransformBroadcaster()

    # prepare tf msg
    t = geometry_msgs.msg.TransformStamped()

    # t.header.stamp = rospy.Time.now()
    # Experiment, Late tf2
    t.header.stamp = rospy.Time.now() - rospy.Duration(60)
    t.header.frame_id = "world"
```

- rviz를 실행시킨 터미널에서 아래와 같은 에러가 발생합니다.

```bash
[ WARN] [1671940248.738235698]: TF_REPEATED_DATA ignoring data with redundant
timestamp for frame turtle1 at time 1671940148.738096 according to authority unknown_publisher
```

tf에는 시간 데이터가 포함되어 있습니다. 따라서 현재 시간과 tf에 담기 시간의 차이가 크다면 ROS는 이를 안정적이지 못한 것으로 판단해 무시합니다. (위 에러는 아마 로봇 개발을 하면서 마주치게 되는 Warning Top3안에 들지 않을까 싶습니다.)

- 이번 예시에는 터미널이 4개나 필요하였습니다. 예시의 빠른 실행을 위해 **launch file**을 만들 수 있습니다.

```xml
<launch>
    <node pkg="turtlesim" type="turtlesim_node" name="sim"/>

    <node pkg="turtlesim" type="turtle_teleop_key" name="teleop" output="screen"/>

    <node name="turtle1_tf2_broadcaster" pkg="py_tf2_tutorial" type="turtle_tf2_broadcaster.py" respawn="false" output="screen" >
        <param name="turtle" type="string" value="turtle1" />
    </node>

    <node name="rviz" pkg="rviz" type="rviz" args="-d $(find py_tf2_tutorial)/rviz/turtlesim_tf.rviz" />
</launch>
```

---

### tf listener

tf broadcaster의 다음으로 tf listener에 대해 배워봅시다.

- turtlesim follow demo

```xml
roslaunch py_tf2_tutorial follow_demo.launch
```

- 사진과 같이 우리가 조종하는 첫번째 거북이를 두번째 거북이가 따라오게 됩니다.
  ![tf_follow.gif](/kr/ros_basic_noetic/images9/tf_follow.gif?height=400px)

- rviz를 통해 tf들 사이에 어떠한 변화가 있는지도 직접 확인해보세요
  ![lec8_5.png](/kr/ros_basic_noetic/images9/lec8_5.png?height=400px)

- launch 파일은 다음과 같은 내용을 포함하고 있습니다.

```xml
<launch>
    <node pkg="turtlesim" type="turtlesim_node" name="sim"/>

    <node pkg="turtlesim" type="turtle_teleop_key" name="teleop" output="screen"/>

    <node name="turtle1_tf2_broadcaster" pkg="py_tf2_tutorial" type="turtle_tf2_broadcaster.py" respawn="false" output="screen" >
        <param name="turtle" type="string" value="turtle1" />
    </node>

    <node name="turtle2_tf2_broadcaster" pkg="py_tf2_tutorial" type="turtle_tf2_broadcaster.py" respawn="false" output="screen" >
        <param name="turtle" type="string" value="turtle2" />
    </node>

    <node pkg="py_tf2_tutorial" type="turtle_tf2_listener.py"
          name="listener" output="screen"/>
</launch>
```

1. turtlesim 실행
2. teleop key 실행
3. turtle1의 tf broadcaster
4. turtle2의 tf broadcaster
5. **tf listener**

> 이번 예제에서 살펴보고자 하는 것은 tf listener입니다.

- **TransformListener** 클래스는 생성되는 순간부터 **/tf** topic에 귀기울이기 시작합니다. Buffer는 정해진 사이즈만큼 tf 데이터를 품게 되며, TransformListener에 전달하게 되면, tf topic data를 받아 Buffer에 쌓아두는 것입니다.

```python
if __name__ == '__main__':

    rospy.init_node('tf2_turtle_listener')

    tfBuffer = tf2_ros.Buffer()
    listener = tf2_ros.TransformListener(tfBuffer)
```

- Buffer의 lookup_transform 메소드는 두 frame 사이의 translation, rotation 변환을 계산해줍니다.

```python
	while not rospy.is_shutdown():
        try:
            # calculate transformation btw two dynamic tfs
            # those tfs were broadcasted from TransformBroadcaster
            trans = tfBuffer.lookup_transform(turtle_name, 'turtle1', rospy.Time(), rospy.Duration(1.0))
        except (tf2_ros.LookupException, tf2_ros.ConnectivityException, tf2_ros.ExtrapolationException):
            rate.sleep()
            continue
```

- lookup_transform 메소드의 원형은 다음과 같습니다.

![lec8_6.png](/kr/ros_basic_noetic/images9/lec8_6.png)

- image from : [docs.ros2.org](https://docs.ros2.org/foxy/api/tf2_ros/classtf2__ros_1_1Buffer.html#a3ab502cc1e8b608957a96ad350815aee)

**lookup_transform**의 계산 결과를 직접 살펴보고자 print 문을 추가하였으며, 캡쳐 사진을 통해 간단히 함께 살펴봅시다.

![lec8_7.png](/kr/ros_basic_noetic/images9/lec8_7.png?height=400px)

- **translation**은 두 거북이의 frame사이 수평 거리를 보여주며
- **rotation**은 두 frame사이 회전을 쿼터니언으로 보여줍니다.
- 기준이 되는 frame과 목표 frame은 id로 구분하며, 현재 turtle1, turtle2로 구분하고 있습니다.

> 현 상황을 그림으로 간단히 정리하자면 다음과 같습니다.

_lookup_transform의 계산 결과는 target frame인 turtle2의 위치가 source frame인 turtle1 기준에서는 어떠한 좌표를 갖는지를 포함합니다._

![lec8_8.png](/kr/ros_basic_noetic/images9/lec8_8.png?height=400px)

- image from : [CMU Qatar](https://web2.qatar.cmu.edu/~gdicaro/16311-Fall17/slides/16311-2-3-Pose-representation-and-transformation.pdf)

로보틱스에서 이러한 좌표 변환은 매우 자주 사용되며, 일반적으로 **Homogeneous Matrix**의 형태로 표현합니다.

![lec8_9.png](/kr/ros_basic_noetic/images9/lec8_9.png?height=400px)

- from : [eth robotics lecture notes](https://ethz.ch/content/dam/ethz/special-interest/mavt/robotics-n-intelligent-systems/rsl-dam/documents/RobotDynamics2017/RD_HS2017script.pdf)

> 그 밖에, 코드에서 구현된 기능을 간단히 살펴보며 마무리하겠습니다.

- 두번째 거북이를 등장시키는 **service client**

```python
		# Spawn second turtle
    rospy.wait_for_service('spawn')
    spawner = rospy.ServiceProxy('spawn', turtlesim.srv.Spawn)
    turtle_name = rospy.get_param('turtle', 'turtle2')
    spawner(4, 2, 0, turtle_name)
```

- 거북이를 제어하기 위한 **Twist msg topic publisher**

```python
		# turtle2 controller
    turtle_vel = rospy.Publisher('%s/cmd_vel' % turtle_name, geometry_msgs.msg.Twist, queue_size=1)

    rate = rospy.Rate(50.0)

    while not rospy.is_shutdown():
				...
        turtle_vel.publish(msg)

        rate.sleep()
```

이렇게 tf에 대해서 turtlesim 예시와 함께 살펴보았습니다. 로보틱스에서 자주 사용되는 좌표계와 그들 사이의 변환을 시간 데이터와 함께 표현하는 것이 ROS의 tf2입니다.

> 다음 강의에 계속 이어집니다.
