---
title: "Lecture6 - ROS Service, Parameter"
date: 2022-12-25T13:36:25+09:00
draft: false
---

> 지난 시간 마지막 예시였던 장애물 회피 코드부터 간단하게 리뷰해보고자 합니다.

![lec5_0.png](/kr/ros_basic_noetic/images6/lec5_0.png?height=400px)

제가 작성한 로직은 다음과 같습니다.

- 과제를 해보셨다면 아시겠지만, 측정 범위를 벗어나게 되면 data.ranges는 inf 값을 갖게 됩니다. 이를 걸러내는 코드가 아래 부분입니다.

```python
        for i, point in enumerate(data.ranges):
            if not math.isinf(point) and point < 1.0:
```

![lec5_1.png](/kr/ros_basic_noetic/images6/lec5_1.png?height=400px)

- 저의 로직은, 정면을 기점으로 왼쪽과 오른쪽 각각 inf가 아닌 데이터의 개수를 카운팅합니다. 전체 데이터가 362개이고 마지막 데이터는 사용하기 않는 값이기 때문에, 180을 기점으로 잡았습니다.

```python
        left_side_count = 0
        right_side_count = 0

        for i, point in enumerate(data.ranges):
            if not math.isinf(point) and point < 1.0:
                if i > 180:
                    left_side_count += 1
                else:
                    right_side_count += 1
```

- 이제, 제어 데이터를 만들어줍니다. ROS를 비롯하여 로봇 시스템에서는 대부분 **오른손 좌표계**를 사용합니다. 따라서 위에서 보았을 rospy.loginfo(hello_du)때 오른손이 감기는 **반시계 방향이 + 부호를** 갖게 됩니다. 이를 고려하여 각속도를 정했습니다. 180은 magic number, 일종의 변환 상수입니다.

![lec5_2.png](/kr/ros_basic_noetic/images6/lec5_2.png?height=300px)

image from : [오로카](https://cafe.naver.com/openrt/24274)

```python
		def laser_cb(self, data):

        left_side_count = 0
        right_side_count = 0

        for i, point in enumerate(data.ranges):
            if not math.isinf(point) and point < 1.0:
                if i > 180:
                    left_side_count += 1
                else:
                    right_side_count += 1

        self.twist_.linear.x = 0.3
        self.twist_.angular.z = (right_side_count - left_side_count) / 100

        self.cmd_vel_pub_.publish(self.twist_)

```

지난 강의에서 이야기한 것과 같이 이 문제의 정답은 없습니다.

다만, Topic의 Pub / Sub을 모두 사용할 수 있는지 스스로 점검해볼 수 있을 것입니다.

---

### ROS Parameter

앞선 저의 예시에서 마지막 속도로 변환하는 부분 수식에 나누기 100이 있었던 것을 기억하시나요? 이런 상수를 직접 코드에 적는 것은 사실 추천되지 않습니다. 개발 이후 해당 상수를 변경하고자 하였을 시, 직접 코드를 수정하고 다시 실행해야 하기 때문에 불편을 야기합니다.

> 이러한 문제의 해결 방법으로 ROS의 매개변수, **parameter**를 다루는 방법을 알아보겠습니다.

- **py_param_pkg/scripts/various_params.py**

```python
#!/usr/bin/env python3

import rospy

class ParamNode:

    def __init__(self):
        self.str_param_ = rospy.get_param('~str_param', 'hello_world')
        self.int_param_ = rospy.get_param('~int_param', 2023)
        self.double_param_ = rospy.get_param('~double_param', 3.14)
        self.bool_param_ = rospy.get_param('~bool_param', True)
        self.list_of_float_param_ = rospy.get_param('~list_of_float_param', [1., 2., 3., 4.])

        rospy.loginfo(f"""
        self.str_param_ = {self.str_param_}
        self.int_param_ = {self.int_param_}
        self.double_param_ = {self.double_param_}
        self.bool_param_ = {self.bool_param_}
        self.list_of_float_param_ = {self.list_of_float_param_}
        """)

def param_node():
    rospy.init_node('param_node', anonymous=True)
    param_node = ParamNode()
    rospy.spin()

if __name__ == '__main__':
    try:
        param_node()
    except rospy.ROSInterruptException:
        pa
```

- 실행 결과는 다음과 같습니다.

```python
$ rosrun py_param_pkg various_params.py
[INFO] [1672014267.630578]:
        self.str_param_ = hello_world
        self.int_param_ = 2023
        self.double_param_ = 3.14
        self.bool_param_ = True
        self.list_of_float_param_ = [1.0, 2.0, 3.0, 4.0]
```

- 매개변수를 선언하고 기본값을 지정하는 방법은 **rospy.get_param()**을 사용하는 것입니다. 두번째 기본값을 잘 보면 어떤 타입을 사용하는지 알 수 있습니다.

```python
    def __init__(self):
        self.str_param_ = rospy.get_param('~str_param', 'hello_world')
        self.int_param_ = rospy.get_param('~int_param', 2023)
        self.double_param_ = rospy.get_param('~double_param', 3.14)
        self.bool_param_ = rospy.get_param('~bool_param', True)
        self.list_of_float_param_ = rospy.get_param('~list_of_float_param', [1., 2., 3., 4.])
```

- parameter 앞에 붙는 물결 표시 (~)는 **private parameter**를 의미합니다. 이에 대해 궁금하다면 아래의 추가 자료를 학습해보세요.

```python
global_name = rospy.get_param("/global_name")
relative_name = rospy.get_param("relative_name")
private_param = rospy.get_param('~private_name')
default_param = rospy.get_param('default_param', 'default_value')
```

추가 자료 : [wiki.ros](http://wiki.ros.org/rospy/Overview/Parameter%20Server)

- 매개변수를 변경하는 가장 보변적인 방법은 **launch file**을 사용하는 것입니다. launch file의 **param** 태그를 사용하여 Node에 원하는 parameter를 전달할 수 있습니다.

```xml
<launch>
  <node name="various_param_node" pkg="py_param_pkg" type="various_params.py" respawn="false" output="screen" >
    <param name="str_param" type="string" value="roslaunch changed me" />
  </node>
</launch>
```

ROS 프로그래밍을 하다 보면 매개변수가 아주 많이 필요한 경우가 있습니다. 이럴 때마다 launch file에 param 태그 라인을 추가하는 것은 매우 귀찮은 일입니다.

**yaml**이라는 형식의 파일로 매개변수를 한번에 관리할 수 있습니다.

- **py_param_pkg/param/config.yaml**

```yaml
str_param: "yaml string"
int_param: 9
double_param: 2.71828
bool_param: "false"
list_of_float_param: [3., 2., 1.]
```

- **py_param_pkg/launch/param_with_yaml.launch**

```xml
<launch>
  <node name="various_param_node" pkg="py_param_pkg" type="various_params.py" respawn="false" output="screen" >
    <rosparam command="load" file="$(find py_param_pkg)/param/config.yaml" />
    <param name="str_param" type="string" value="roslaunch changed me" />
  </node>
</launch>
```

launch file에 rosparam 태그를 추가하고, load command를 사용하며 사용하는 yaml 파일의 위치를 file 옵션을 통해 전달합니다.

### **ROS parameter Commands**

rostopic, rosnode와 같이 parameter 또한 터미널 명령어를 갖고 있습니다.

- 접근 가능한 모든 parameter들을 나열합니다.

```xml
rosparam list
```

- 특정 paramter의 값을 얻고자하면 아래 키워드를 사용합니다.

```xml
rosparam get <parameter_name>
```

- 선언되어 있는 parameter의 값을 변경하고 싶은 경우 아래 키워드를 사용합니다.

```xml
rosparam set <parameter_name> <value>
```

여러분들이 작성한 회피 프로그램에도 매개변수로 작용하는 상수들이 있을 것입니다. 이를 parameter로 변경하여 launch file과 yaml file로 업데이트하는 작업을 해보세요

---

### ROS Service

> Topic에 이어 ROS의 통신 메커니즘 두번째로 Service를 배워보겠습니다.

Service가 동작하는 방식은 아래와 같습니다.

그림과 같이 Client Node가 Server Node로 **request**를 주면, 해당 request에 대응하는 적절한 **response**가 다시 Client에게로 전달됩니다. 이 과정을 **Service Call**이라고 부릅니다.

![service1.gif](/kr/ros_basic_noetic/images6/service1.gif?height=300px)

image from : [docs.ros.org](https://docs.ros.org/en/foxy/Tutorials/Services/Understanding-ROS2-Services.html)

- 하나의 Service Server에는 여러 Client Node가 request 할 수 있지만, **Server는 동시에 여러 request를 처리하지는 못합니다.** 두 Node에서 동시에 request가 왔다면, 조금이라도 먼저 통신한 Node의 작업을 우선 진행하고, 그동안 다른 Node는 기다리고 있어야 합니다.

![service2.gif](/kr/ros_basic_noetic/images6/service2.gif?height=300px)

image from : [https://docs.ros.org/en/foxy/Tutorials/Services/Understanding-ROS2-Services.html](https://docs.ros.org/en/foxy/Tutorials/Services/Understanding-ROS2-Services.html)

**Topic과 비교하여 Service의 특징을 알아봅시다.**

- 1:1 통신 : Topic publish를 하면 여러 Node가 Subscribe 가능합니다. 반면, **Service는 request가 온 대상에게만 response를 줍니다.**
- 순차적 통신 : Service Server는 동시에 여러 request를 처리할 수 없습니다. **현재 작업중인 request가 처리될 때 까지 다른 request는 기다리고 있어야 합니다.**
- 단발성 : Topic은 대부분 **지속적으로** publish를 진행하는 반면, Service는 **1회성** 통신입니다.

> 실제 로봇 프로그램에서 Service는 어떻게 사용될 수 있을지, 예시를 통해 살펴봅시다.

- 예제 패키지 빌드

```xml
cd ~/catkin_ws
catkin build py_service_pkg
source devel/setup.bash
```

---

### Service Client

- **예제 실행 - 아르키메데스 나선**

```bash
# Terminal 1
roslaunch py_service_pkg empty_gazebo.launch
# Terminal 2
rosrun py_service_pkg spawn_model_client.py
```

![helix.gif](/kr/ros_basic_noetic/images6/helix.gif?height=500px)

방금 실행한 예시는 Gazebo에게 box를 등장시켜달라고 하는 **Service Client**를 포함하고 있습니다.

Box가 등장하는 위치를 아래 사진과 같은 수식에 맞추어 설정한 것 뿐입니다.

![lec5_3.png](/kr/ros_basic_noetic/images6/lec5_3.png?height=300px)

> 그럼, 코드를 분석해 볼까요?

- 필요한 파이썬 패키지들을 import 합니다.

```bash
import math
import rospy
import rospkg
from geometry_msgs.msg import Pose
from gazebo_msgs.srv import SpawnModel
```

여기서 중요한 점은 **msg**와 **srv** 부분입니다. topic에서 사용되는 데이터 타입이 Message였고, 프로그래밍 시에는 msg로 사용하였습니다.

- Service에서는 msg가 아니라 **srv**라는 데이터 타입을 사용합니다.

![lec5_4.png](/kr/ros_basic_noetic/images6/lec5_4.png?height=300px)

image from : [rsl.eth](https://rsl.ethz.ch/education-students/lectures/ros.html)

- 이 srv는 msg와 다른 점이 있는데, **request**와 **response**로 나뉘어 있다는 점입니다. **---** 표시를 기점으로 위쪽은 Server에게 전달하는 request, 아래쪽은 Server가 다시 회답하는 response 부분입니다.

![lec5_5.png](/kr/ros_basic_noetic/images6/lec5_5.png?height=200px)

- 이번 예시에서 사용한 **gazebo_msgs/SpawnModel**도 아래와 같은 구조를 갖습니다.

![lec5_6.png](/kr/ros_basic_noetic/images6/lec5_6.png?height=300px)

image from : [docs.ros.org](http://docs.ros.org/en/jade/api/gazebo_msgs/html/srv/SpawnModel.html)

> gazebo_msgs/SpawnModel를 살펴보면 파란 글자로 geometry_msgs/Pose라는 부분이 있습니다. 이와 같이 srv는 다른 msg를 품을 수도 있고, 이렇게 만든 srv를 또다시 다른 srv에 포함시킬 수도 있습니다.

코드 구현 관점에서, geometry_msgs/Pose는 Model을 등장시킬 초기 위치를 지정하는데 사용됩니다.

```python
# initial_pose
initial_pose = Pose()
initial_pose.position.x = 0.0
initial_pose.position.y = -1
initial_pose.position.z = 0.2

# z rotation -pi/2 to Quaternion
initial_pose.orientation.z = -0.707
initial_pose.orientation.w = 0.707
```

- **Service Client**의 생성과 사용은 아래와 같습니다.

```python
spawn_model_prox = rospy.ServiceProxy("gazebo/spawn_urdf_model", SpawnModel)
...
result = spawn_model_prox(
    entity_name, model_xml, robot_namespace, initial_pose, reference_frame
)
```

`rospy.ServiceProxy()`는 2개의 매개변수를 필요로 합니다.

- **service 이름**
- **service 데이터 타입 (srv)**

생성한 client로 request를 하기 위해서는 생성한 인스턴스에 매개변수를 전달하기만 하면 됩니다. 마치 함수 호출처럼 말이지요. 이는 ServiceProxy 내부적으로 **call** 메소드가 구현되어있기 때문입니다.

service call의 결과로 result가 반환되며, 예시에서는 성공 여부를 반환하도록 되어 있습니다.

- 추가적으로, model을 불러오는 부분을 간단하게 설명하고자 합니다.

```python
    # model_xml
    rospack = rospkg.RosPack()
    model_path = rospack.get_path("py_service_pkg") + "/urdf/"

    with open(model_path + model_name + ".urdf", "r") as xml_file:
        model_xml = xml_file.read().replace("\n", "")
```

Gazebo는 urdf라는 파일을 전달하면 해당 파일을 기반으로 시뮬레이션에 물체를 등장시켜줍니다. 이 urdf라는 것은 로봇을 표현하기 위한 일종의 약속된 파일 확장명입니다.

![lec5_7.png](/kr/ros_basic_noetic/images6/lec5_7.png?height=300px)

image from : [spart](https://spart.readthedocs.io/en/latest/Tutorial_Robot.html)

- 세상 모든 로봇들은 joint와 link로 표현 가능합니다.
- 이러한 개념을 바탕으로 로봇의 특성을 텍스트 파일로 표현하는 형식이 바로 urdf이며, 아래와 같이 여러 태그와 속성을 사용하여 작성 가능합니다.

```xml
<?xml version="1.0"?>
<robot name="box">
  <link name="box">
    <inertial>
      <mass value="1"/>
      <!-- Inertia values were calculated to be consistent with the mass and
           geometry size, assuming a uniform density. -->
      <inertia ixx="0.0108" ixy="0" ixz="0" iyy="0.0083" iyz="0" izz="0.0042"/>
    </inertial>
    <visual>
      <geometry>
        <box size=".1 .2 .3"/>
      </geometry>
    </visual>
    <collision name="box">
      <geometry>
        <box size=".1 .2 .3"/>
      </geometry>
    </collision>
  </link>
</robot>
```

### ROS Service Commands

**gazebo/spawn_urdf_model**과 같은 service는 gazebo_ros를 사용할 때 자동으로 함께 실행됩니다. 이렇게 현재 어떠한 service가 존재하며, 또 구체적인 정보는 어떻게 조회하는지 알아봅시다.

- 현재 사용 가능한 모든 service를 조회해봅시다.

```python
$ ros2 service list
/delete_entity
/gazebo/describe_parameters
/gazebo/get_parameter_types
/gazebo/get_parameters
/gazebo/list_parameters
/gazebo/set_parameters
...
```

{{% notice tip %}}
리눅스의 grep 명령어를 함께 사용해 보세요.
{{% /notice %}}

- 특정 service가 어떤 srv 타입을 사용하는지 검색하고 싶다면 다음 커멘드 라인을 사용합니다.

```bash
$ rosservice type /gazebo/spawn_urdf_model
****
```

- 이렇게 검색된 srv는 rossrv show와 결합할 때 더욱 진가를 발휘합니다.

```python
$ rossrv show `rosservice type /gazebo/spawn_urdf_model`

```

- 특정 srv 타입에 대한 자세한 정보는 다음과 같이 조회할 수 있습니다.

```bash
$ rosservice info /gazebo/spawn_urdf_model

```

gazebo_ros에서 제공하는 다양한 service들이 있습니다. rosservice 커멘드를 사용하여 조회해보고 여러분들만의 Application을 생각해 보세요.

---

## Service Server

- 예제 실행 - 긴급 정지

```python
# Terminal 1
roslaunch smb_gazebo smb_gazebo.launch
# Terminal 2
rosrun py_service_pkg emergency_stop.py
# Terminal 3
rqt
```

- 두번째 Node 실행 시, 로봇이 원을 그리며 움직이기 시작합니다.
- 세번째 명령어를 통해 등장하는 rqt는 아래와 같이 사용 가능합니다. 로봇에게 긴급 정지 명령을 내려보겠습니다.

![srv_stop.gif](/kr/ros_basic_noetic/images6/srv_stop.gif?height=400px)

실제 로봇 개발시에도 Service는 이렇게 **단발성이고, 빠르게 실행되어야 하는 동작**에 주로 사용됩니다. 더불어, 지금 실행한 예시가 Service Server임을 다시 한 번 상기시켜드립니다.

![lec5_4.png](/kr/ros_basic_noetic/images6/lec5_4.png?height=300px)

image from : [rsl.eth](https://rsl.ethz.ch/education-students/lectures/ros.html)

> 코드를 분석해 봅시다.

- py_service_pkg/scripts/emergency_stop.py

```python
from geometry_msgs.msg import Twist
from std_srvs.srv import SetBool, SetBoolResponse
```

이번에 사용하는 데이터 타입은 크게 2 종류입니다.

- 로봇 제어 topic에 사용되는 **Twist**
- 긴급 정지 service에 사용될 **SetBool**

![lec5_8.png](/kr/ros_basic_noetic/images6/lec5_8.png?height=400px)

image from : [docs.ros.org](http://docs.ros.org/en/api/std_srvs/html/srv/SetBool.html)

**SetBoolResponse**이라는 것은 SetBool srv 중 response 부분에 해당합니다. 기본 데이터 타입 이름 + Response를 붙여주기만 하면 사용 가능합니다.

ROS의 msg, srv는 다양한 언어와 상황을 고려하도록 만들어져 있으며, ROS 2에서는 **IDL**이라는 이름으로 더욱 발전하였습니다. 이후의 커스텀 데이터 타입 제작을 통해 이 과정을 다시 살펴봅시다.

- 다음으로 통신 메커니즘을 생성합니다.

```python
class EmergencyStopNode(object):

    def __init__(self):
        self.cmd_vel_pub_ = rospy.Publisher("cmd_vel", Twist, queue_size=10)
        self.stop_server_ = rospy.Service("emergency_stop", SetBool, self.stop_cb)
```

로봇 제어를 위한 topic publisher와 service server를 생성합니다.

`rospy.Service()`를 통해 Service Server를 생성할 수 있으며 다음과 같은 매개변수를 필요로합니다.

- **Service 이름**
- **srv 타입**
- **Client로부터 request가 올 시 실행되는 callback 함수**

callback 함수는 일전 subscriber에서 살펴본 바 있습니다. service server의 callback 함수는 항상 매개변수로 request srv를 받습니다. 그리고 return 값은 항상 response가 됩니다.

```python
def stop_cb(self, request):
	  ...
    return self.response_
```

- request 데이터 중 boolean 값을 갖는 data의 true / false 여부에 따라 로봇의 정지 여부가 결정됩니다.

```python
    if request.data is True:
        self.twist_msg_.linear.x = 0.0
        self.twist_msg_.angular.z = 0.0
        self.cmd_vel_pub_.publish(self.twist_msg_)

        self.response_.success = True
        self.response_.message = "Successfully Stopped"
    else:
        self.response_.success = False
        self.response_.message = "Stop Failed"
```

마지막에 사용한 rqt의 service caller는 별도의 프로그래밍이나 복잡한 터미널 명령어 없이도 손쉽게 service를 다룰 수 있게 해주는 ROS의 툴입니다.

![lec5_9.png](/kr/ros_basic_noetic/images6/lec5_9.png?height=400px)

> 지금까지 ROS Service에 대해 배워보았습니다. Topic과 더불어 많이 사용되는 통신 메커니즘이므로 잘 숙지하고 복습하시기 바랍니다.

---

**참고자료**

- [https://rsl.ethz.ch/education-students/lectures/ros.html](https://rsl.ethz.ch/education-students/lectures/ros.html)
- [https://ko.wikipedia.org/wiki/아르키메데스\_와선](https://ko.wikipedia.org/wiki/%EC%95%84%EB%A5%B4%ED%82%A4%EB%A9%94%EB%8D%B0%EC%8A%A4_%EC%99%80%EC%84%A0)
- [https://docs.ros.org/en/foxy/index.html](https://docs.ros.org/en/foxy/index.html)
- [http://wiki.ros.org/Services](http://wiki.ros.org/Services)
