---
title: "Lecture10. TF and ROS 2"
date: 2023-04-28T10:51:09+09:00
draft: false
---

## Introduction to tf2

> 대부분의 로보틱스 과정들에서 가장 먼저 다루는 것이 바로 **좌표계 변환 - Transformation**입니다. 로봇은 수많은 joint와 link로 이루어져 있으며, 센서와 움직임을 다루기 때문에 좌표계를 다루는 일이 매우 빈번합니다.

ROS 2에서는 TF2라는 특수한 형태로 이 좌표계와 시간을 함께 다루고 있습니다. 예시와 설명을 통해 ROS 2의 TF2에 대해 배워봅시다 😊

![Untitled.png](/kr/ros2_foxy/images10/Untitled.png?height=300px)

- image from : [eth robot dynamics lecture notes](https://ethz.ch/content/dam/ethz/special-interest/mavt/robotics-n-intelligent-systems/rsl-dam/documents/RobotDynamics2017/RD_HS2017script.pdf)

### Husky Again

> 일전 husky 예시를 통해 tf2에 대한 개요을 파악해보고자 합니다.

```python
# Terminal 1 - husky gazebo
ros2 launch husky_gazebo husky_playpen.launch.py

# Terminal 2 - rviz
rviz2
```

![Untitled1.png](/kr/ros2_foxy/images10/Untitled1.png?height=400px)

⇒ rviz에서 보이는 세가지 색상의 막대가 바로 tf2 입니다. x,y,z의 각 축을 각기 다른 색으로 표현하였으며, 연관된 좌표계끼리는 노란 선을 통해 연결한 모습이 보입니다.

![Untitled2.png](/kr/ros2_foxy/images10/Untitled2.png?height=300px)

- rviz2 화면에서 fixed frame을 변경해보면, 모든 데이터, 좌표들의 원점이 바뀌는 것을 볼 수 있습니다.

![Untitled3.png](/kr/ros2_foxy/images10/Untitled3.png?height=300px)

⇒ 이렇게 좌표변환에 따른 데이터 시각화가 가능하려면 모든 좌표계들 사이의 회전, 평행 이동 변환 정보를 알고 있어야 할 것입니다. ROS 2에서 이를 어떻게 다룰 수 있는지 또 하나의 예시를 살펴보겠습니다.

## TF2 Tree

- tf2 view frame - 이전 husky는 실행 상태로 유지합니다.

```bash
# Terminal1 - husky gazebo
ros2 launch husky_gazebo husky_playpen.launch.py

# Terminal2 - tf2 view frame
cd ~/ros2_ws
ros2 run tf2_tools view_frames.py
```

- 약 5초간 대기 후, 폴더를 조회해보면 아래와 같은 tree 사진이 저장된 것을 확인할 수 있습니다.

![frames.png](/kr/ros2_foxy/images10/frames.png?height=300px)

tree를 확대해서 살펴봅시다.

- 모든 node들끼리는 하나의 연결 상태를 갖습니다.
- rviz2의 fixed frame에서는 바로 이 node를 선택했던 것입니다.
- 해당 연결을 담당하는 Broadcaster 정보, 평균 rate, buffer length 등의 정보를 확인 가능합니다.

![Untitled4.png](/kr/ros2_foxy/images10/Untitled4.png?height=300px)

{{% notice note %}}
만약 tf tree가 온전히 연결되어 있지 않다면 어떤 일이 발생할까요?
{{% /notice %}}

![Untitled5.png](/kr/ros2_foxy/images10/Untitled5.png?height=300px)

⇒ tf2간 연결이 끊어지게 되면, 끊어진 아래 시점에 대해서는 그 어떠한 정보도 알 수 없습니다. 따라서 rviz2에서의 시각화나, SLAM, 자율주행 시 이는 큰 문제가 됩니다.

- 더불어, tf2가 제대로 연결되어 있더라고 정확한 시간과 정확한 좌표 변환을 갖는 것이 매우 중요합니다. 예를 들어, 라이다의 tf를 180도 반대로 설정해버리면 후방에 있는 장애물을 전방 장애물로 잘못 인식할 수 있습니다.

![Untitled6.png](/kr/ros2_foxy/images10/Untitled6.png?height=300px)

image from : [answers.ros.org](https://answers.ros.org/question/265846/how-to-build-tf-topic-frame-tree-for-hector_slam-or-gmapping/)

- 때문에 sensor topic message들은 공통적으로 tf2 정보를 포함하고 있습니다.

```bash
$ ros2 interface show sensor_msgs/msg/PointCloud2
**std_msgs/Header header**
uint32 height
uint32 width
sensor_msgs/PointField[] fields
bool is_bigendian
uint32 point_step
uint32 row_step
uint8[] data
bool is_dense

$ ros2 interface show std_msgs/msg/Header
uint32 seq
time stamp
string frame_id
```

⇒ frame_id와 stamp이 바로 tf2와 관련된 데이터로 이렇게 센서 데이터들은 대부분 좌표와 시간에 대한 정보를 포함하게 됩니다.

이번 시간에는 크게 3가지 예시를 통해 tf2에 대해 학습할 예정입니다.

- static transform ⇒ 움직이지 않는 좌표계
- tf2 broadcast ⇒ tf2 데이터의 송신
- tf2 listen ⇒ tf2 데이터의 수신

{{% notice note %}}
topic publish를 하는데 왜 또 tf2 boradcast가 필요할까요?
{{% /notice %}}

⇒ 바로 위 예시처럼 topic에 담긴 정보는 실제 좌표변환이 아닌, frame id입니다. 이는 topic data를 받은 subscriber node 입장에서 센서 데이터를 다루는 상황을 생각해보면 이해하기 쉽습니다.

1. topic sub
2. sub msg내 frame id 확인
3. tf2 데이터에서 해당 frame의 변환 listen
4. 이후 로직 구현

그럼, 예시와 코드를 통해 tf2를 마스터해봅시다!

### static broadcaster

#### Orbit World

> tf2의 학습을 위해 gazebo 환경을 만들어 두었습니다. 함께 실행해봅시다.

```python
cd ~/ros2_ws
cbp tf2_world && source install/local_setup.bash

ros2 launch tf2_world tf2_world.launch.py
```

> 마치 행성과 위성 같은 환경을 볼 수 있습니다.

![orbit.gif](/kr/ros2_foxy/images10/orbit.gif?height=300px)

- 현 상황에서 tf2 tree를 조회해봅시다.

```bash
ros2 run tf2_tools view_frame.py
```

![Untitled7.png](/kr/ros2_foxy/images10/Untitled7.png?height=200px)

{{% notice note %}}
tf2 데이터는 아무것도 없지요? 이번 시간의 목표는 우리가 직접 코딩과 launch file을 통해 tree를 구성해 보는 것입니다.
{{% /notice %}}

- launch file 실행 터미널을 자세히 살펴보면, 아래와 같은 문구를 볼 수 있습니다.

```python
[gzserver-1] [INFO] [1681981785.437867364] [gazebo_ros_state]: Publishing states of gazebo models at [/model_states]
[gzserver-1] [INFO] [1681981785.438120429] [gazebo_ros_state]: Publishing states of gazebo links at [/link_states]
```

- 현 gazebo 환경에서의 존재하는 모든 모델의 state를 /model_state topic을 통해 알 수 있습니다. topic monitor를 통해 살펴봅시다.

![Untitled8.png](/kr/ros2_foxy/images10/Untitled8.png?height=300px)

### static transform publisher

![Untitled9.png](/kr/ros2_foxy/images10/Untitled9.png?height=200px)

- image from : [theegeek](https://theegeek.com/static-members-and-static-concept/)

> 움직이지 않는 static object의 tf2 데이터는 static transform publisher가 담당합니다. 덕분에 우리는 굳이 코딩을 하지 않아도 미리 준비된 node를 실행하기만 하면 됩니다.

- launch file을 통해 static transform publisher를 사용해봅시다.

```bash
    static_transform_publisher = Node(
        package = "tf2_ros",
        executable = "static_transform_publisher",
        arguments = ["0", "0", "0.5", "0", "0", "0", "world", "unit_sphere"],
    )

    return LaunchDescription(
        [
            static_transform_publisher,
        ]
    )
```

- arguments에 들어가는 매개변수들은 순서대로 (x y z roll pitch yaw parent_frame child_frame)이 됩니다.

⇒ world frame 기준 unit_sphere frame를 z축으로 0.5만큼 떨어뜨려 tf2 data가 생성됩니다.

{{% notice note %}}
로봇 개발자들은 암묵적으로 모든 상황의 원점이 되는 절대 좌표계를 world frame이라고 정합니다.
{{% /notice %}}

- 다시 한 번 tf2 tree를 살펴봅시다

```python
ros2 run tf2_tools view_frame.py
```

![Untitled10.png](/kr/ros2_foxy/images10/Untitled10.png?height=300px)

⇒ world to unit_sphere tf가 성공적으로 생성되었습니다!

- 이제 rviz2 화면을 통해서도 tf2를 시각화 할 수 있습니다.

![Untitled11.png](/kr/ros2_foxy/images10/Untitled11.png?height=400px)

{{% notice note %}}
움직이지 않는 물체 뿐만 아니라, 로봇 내부에서 고정된 부분에서도 static_transform_publisher를 사용할 수 있습니다. 자주 탈부착되는 센서 frame이 대표적인 예시입니다.
{{% /notice %}}

![Untitled12.png](/kr/ros2_foxy/images10/Untitled12.png?height=300px)

- image from : [articulatedrobotics](https://articulatedrobotics.xyz/ready-for-ros-6-tf/)

### TF2 Broadcaster & Lisenter & Add frame

> 지난 강의에서 고정된 물체에 대한 tf2 데이터를 다루는 방법을 알아보았습니다. 이번 시간부터 tf2를 다루는 코드를 작성해 보겠으며, 최종적으로 아래와 같은 시스템을 만들어보겠습니다.

![orbit_frame.gif](/kr/ros2_foxy/images10/orbit_frame.gif?height=400px)

## TF2 Python Programming

### TF2 Broadcaster (Python)

> topic에서는 publish/subscribe라는 용어를 사용하였지만 tf2에서는 broadcast/listen이라는 용어를 사용합니다. topic과 tf2는 완전 별개의 개념입니다! 혼란되지 않도록 주의하세요.

- 우선, 예시를 실행해봅시다.

```bash
# Terminal1 - gazebo
ros2 launch tf2_world tf2_world.launch.py

# Terminal2 - tf2 broadcaster
ros2 run py_tf2_tutorial tf2_broadcaster
```

- rviz2를 통해 움직이는 구의 tf2를 확인할 수 있습니다. 이를 구현하는 것이 이번 예제의 목표입니다.

![tf2_bc.gif](/kr/ros2_foxy/images10/tf2_bc.gif?height=400px)

### 코드 분석

- 이전 강의에서 gazebo 내 모든 물체들의 정보를 담은 /model_state topic을 살펴본 바 있습니다. 이를 코드 구현에 사용하기 위해 정보를 조회해봅시다.

```python
$ ros2 interface show gazebo_msgs/msg/ModelStates
# broadcast all model states in world frame
string[] name                 # model names
geometry_msgs/Pose[] pose     # desired pose in world frame
geometry_msgs/Twist[] twist   # desired twist in world frame
```

⇒ 이 모양을 기억하면서 코드로 진입해보겠습니다.

- 더불어, tf2의 데이터 타입은 geometry_msgs/msg/TransformStamped 입니다. ⇒ [link](http://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/TransformStamped.html)

```python
$ ros2 interface show geometry_msgs/msg/TransformStamped
# This expresses a transform from coordinate frame header.frame_id
# to the coordinate frame child_frame_id
#
# This message is mostly used by the
# tf package.
# See its documentation for more information.

Header header
string child_frame_id # the frame id of the child frame
Transform transform
```

- 코드로 돌아와 import 부분입니다. tf2 데이터를 다루기 위한 패키지인 tf2_ros, ModelStates 데이터 타입을 import합니다.

```python
import rclpy
from rclpy.node import Node
from tf2_ros import TransformBroadcaster

from gazebo_msgs.msg import ModelStates
from geometry_msgs.msg import TransformStamped
```

- 우리의 목표를 정리해봅시다.

1. model_state topic에서 움직이는 물체의 정보를 조회합니다.
2. 해당 데이터를 정제하여 TF2의 형태로 변형한 뒤 broadcast 합니다.

⇒ 따라서 topic sub과 tf2 broadcast가 필요한 상황입니다.

```python
class OrbitBroadcaster(Node):

  def __init__(self):
    super().__init__('tf2_frame_broadcaster')

    # Initialize the transform broadcaster
    self.tf_broadcaster = TransformBroadcaster(self)

    # Subscribe to a turtle{1}{2}/pose topic and call handle_turtle_pose
    # callback function on each message
    self.m_model_state_sub = self.create_subscription(
        ModelStates, "model_states", self.handle_turtle_pose, 1
    )
```

- 코드의 핵심은 sub callback에 담겨 있습니다. model_state msg중 우리가 원하는 animated_sphere의 정보를 파싱하여 TransformStamped msg에 집어넣은 뒤, broadcast 합니다.

```python
def handle_turtle_pose(self, msg):

    index = [i for i in range(len(msg.name)) if msg.name[i] == "animated_sphere"][0]

    t = TransformStamped()

    # Read message content and assign it to
    # corresponding tf variables
    t.header.stamp = self.get_clock().now().to_msg()
    t.header.frame_id = 'world'
    t.child_frame_id = 'animated_sphere'

    t.transform.translation.x = msg.pose[index].position.x
    t.transform.translation.y = msg.pose[index].position.y
    t.transform.translation.z = msg.pose[index].position.z

    t.transform.rotation.x = msg.pose[index].orientation.x
    t.transform.rotation.y = msg.pose[index].orientation.y
    t.transform.rotation.z = msg.pose[index].orientation.z
    t.transform.rotation.w = msg.pose[index].orientation.w

    # Send the transformation
    self.tf_broadcaster.sendTransform(t)
```

- tf2 broadcastt를 위한 API를 정리하고 넘어갑시다. 다시 한 번 말하지만 topic과 헷갈리면 안됩니다!

|                        | Description      |
| ---------------------- | ---------------- |
| TransformBroadcaster() | broadcaster 생성 |
| TransformStamped()     | tf2 msg 생성     |
| sendTransform(msg)     | tf2 broadcast    |

### TF2 Listener [Python]

- rqt_monitor를 통해 model_state topic을 조회한 결과입니다. pose는 잘 보이는데, 속도 데이터인 twist는 비어있는 모습을 볼 수 있습니다.

⇒ TF2 Listener를 사용하여 이를 구현해봅시다!

![Untitled13.png](/kr/ros2_foxy/images10/Untitled13.png?height=300px)

- 예시 실행 ⇒ 선속도와 각속도를 콘솔에서 확인할 수 있습니다.

```python
# Terminal1 - gazebo
$ ros2 launch tf2_world tf2_world.launch.py

# Terminal2 - tf2 broadcaster
$ ros2 run py_tf2_tutorial tf2_broadcast

# Terminal3 - tf2 listener
$ ros2 run py_tf2_tutorial tf2_listener
[INFO] [1681998128.822169076] [tf2_frame_listener]: linear vel : 2.542703, angular vel : 1.810120
[INFO] [1681998128.922124745] [tf2_frame_listener]: linear vel : 2.501199, angular vel : 1.610775
...
```

- 코드의 import 부분입니다. tf2 listen을 위해 TransformListener와 Buffer를 import하고 있습니다.

```python
import math

from geometry_msgs.msg import Twist

import rclpy
from rclpy.node import Node

from tf2_ros import TransformException
from tf2_ros.buffer import Buffer
from tf2_ros.transform_listener import TransformListener
```

- 현재 우리가 구현해야 하는 작업은 다음과 같습니다.

1. tf2 데이터 listen ⇒ 이를 통해 움직이는 물체의 좌표 변화를 감지합니다.
2. 속도 계산 ⇒ 위치를 통해 선속도와 각속도를 계산합니다.
3. topic publish

```python
class TF2Listener(Node):

  def __init__(self):
    super().__init__('tf2_frame_listener')

    self.tf_buffer = Buffer()
    self.tf_listener = TransformListener(self.tf_buffer, self)

    # Create turtle2 velocity publisher
    self.publisher = self.create_publisher(Twist, 'sphere_vel', 10)

    # Call timer_cb function every 10 seconds
    self.timer = self.create_timer(0.1, self.timer_cb)
```

⇒ TransformListener의 생성 이후, 모든 tf2의 listen을 시작하면 이를 Buffer에 담아두는 식입니다.

⇒ 주기적으로 listen, publish를 위해 timer를 사용합니다.

다음으로, timer_cb 함수를 뜯어봅시다.

- tf2 listen을 위해 lookup_transform이라는 함수를 사용하며 3개의 매개변수를 필요로 합니다.
  1.  **Target frame**
  2.  **Source frame**
  3.  **time**

```python
    from_frame_rel = 'animated_sphere'
    to_frame_rel = 'world'

    try:
        t = self.tf_buffer.lookup_transform(
            from_frame_rel, to_frame_rel, rclpy.time.Time())
    except TransformException as ex:
        self.get_logger().info(
            f'Could not transform {to_frame_rel} to {from_frame_rel}: {ex}')
        return
```

- tf2 데이터에서 우리가 원하는 속도를 계산하기 위해 간단한 연산을 추가하였습니다.

```python
    msg = Twist()

    scale_rotation_rate = 1.0
    msg.angular.z = scale_rotation_rate * math.atan2(
        t.transform.translation.y,
        t.transform.translation.x)

    scale_forward_speed = 0.5
    msg.linear.x = scale_forward_speed * math.sqrt(
        t.transform.translation.x ** 2 +
        t.transform.translation.y ** 2)
```

- 최종 완성된 Twist msg를 publish 합니다.

```python
    self.get_logger().info(f"linear_vel: {msg.linear.x}, angular_vel: {msg.angular.z}")
    self.publisher.publish(msg)
```

⇒ 현재 예시이므로 정확한 3차원 속도를 계산하지는 않았지만, gazebo의 model_state 데이터에 기반하여 환경의 ground_truth로 사용할 수 있습니다.

### TF2 Add Frame (Python)

- tf2 구현의 마지막 예시입니다.

```python
# Terminal1 - gazebo
$ ros2 launch tf2_world tf2_world.launch.py

# Terminal2 - tf2 broadcaster
$ ros2 run py_tf2_tutorial tf2_broadcast

# Terminal3 - tf2 add frame
$ ros2 run py_tf2_tutorial tf2_add_frame
```

- animated_sphere의 주위를 위성처럼 궤도 운동하는 새로운 frame이 보입니다.

![orbit_frame.gif](/kr/ros2_foxy/images10/orbit_frame.gif?height=400px)

⇒ 이번 예시는 새로운 dynamic frame을 추가하는 것입니다. 일전 정적 frame은 일전 launch file에서 static_transform_publisher를 사용하였지요? dynamics frame은 코드로 구현해보겠습니다.

- tf2 데이터를 broadcast해야 하는 것은 일전 예시와 동일합니다. 따라서 import와 생성자 부분에 대한 설명은 생략하겠습니다.

```python
import rclpy
import numpy as np
from rclpy.node import Node

from tf2_ros import TransformBroadcaster
from geometry_msgs.msg import TransformStamped

class InverOrbitFrame(Node):

   def __init__(self):
      super().__init__('inverse_orbit_frame_tf2_broadcaster')

      self.count = 0
      self.freq_factor = 2* np.pi / 20;

      self.tf_broadcaster = TransformBroadcaster(self)
      self.timer = self.create_timer(0.1, self.broadcast_timer_callback)
```

- timer에 의해 반복 실행되는 callback 함수를 살펴봅시다.

⇒ animated_sphere을 기준으로 원 운동을 하는 새로운 frame을 만드는 것이 목표입니다. 따라서 (cos(theta), sin(theta))의 좌표를 갖는 tf2 data를 구축하고 이를 시간에 따라 broadcast 하였습니다.

```python
def broadcast_timer_callback(self):

   t = TransformStamped()

   t.header.stamp = self.get_clock().now().to_msg()
   t.header.frame_id = 'animated_sphere'
   t.child_frame_id = 'inverse_orbit_frame'

   t.transform.translation.x = np.cos(-self.count * self.freq_factor)
   t.transform.translation.y = np.sin(-self.count * self.freq_factor)
   t.transform.translation.z = 0.5

   t.transform.rotation.x = 0.0
   t.transform.rotation.y = 0.0
   t.transform.rotation.z = 0.0
   t.transform.rotation.w = 1.0

   self.tf_broadcaster.sendTransform(t)
   self.count += 1
   if self.count >= 20:
       self.count = 0
```

### TF2 C++ Programming

#### TF2 Broadcaster (C++)

> 실제 개발에서 tf2 데이터는 실시간성이 무척 중요한 좌표변환을 다루므로, c++ 코드를 사용하는 비중이 더 큽니다. 따라서, 우선 모든 예시를 살펴보고 센서 패키지에 기반하여 실 사용 사례도 확인해보겠습니다.

- 예시를 실행해봅시다.

```bash
# Terminal1 - gazebo
ros2 launch tf2_world tf2_world.launch.py

# Terminal2 - tf2 broadcaster
ros2 run cpp_tf2_tutorial tf2_broadcaster
```

- rviz2를 통해 움직이는 구의 tf2를 확인할 수 있습니다.

![tf2_bc.gif](/kr/ros2_foxy/images10/tf2_bc.gif?height=400px)

### 코드 분석

- 코드로 돌아와 include 부분입니다. tf2 데이터를 다루기 위한 패키지인 tf2_ros, ModelStates 데이터 타입을 import합니다.

```cpp
#include "rclcpp/rclcpp.hpp"

#include "tf2/LinearMath/Quaternion.h"
#include "tf2_ros/transform_broadcaster.h"
#include "gazebo_msgs/msg/model_states.hpp"
#include "geometry_msgs/msg/transform_stamped.hpp"
```

- tf_broadcaster를 초기화하는 코드 API에서 unique_ptr이 사용됩니다. 개발자의 실수로 tf2_broadcaster가 여러개 생길 시 이를 디버깅하기 힘들어 사전에 방지한 경우입니다.

```cpp
class OrbitBroadcaster : public rclcpp::Node
{
private:
  rclcpp::Subscription<ModelStates>::SharedPtr m_model_state_sub;
  std::unique_ptr<tf2_ros::TransformBroadcaster> m_tf_broadcaster;

public:
  OrbitBroadcaster() : Node("tf2_frame_broadcaster"){

    m_model_state_sub = this->create_subscription<ModelStates>(
      "model_states", 10,
      std::bind(&OrbitBroadcaster::model_state_sub_cb, this, std::placeholders::_1)
    );

    // Initialize the transform broadcaster
    m_tf_broadcaster =
      std::make_unique<tf2_ros::TransformBroadcaster>(*this);
  }
```

- model_states 타입은 3개의 std::vector로 구성됩니다. 따라서 우리가 원하는 값, animated_sphere의 index를 먼저 알아낸 뒤, pose 데이터를 파싱합니다.

```cpp
private:
  void model_state_sub_cb(const ModelStates::ConstSharedPtr msg)
  {
    geometry_msgs::msg::TransformStamped t;

    std::string model_name = "animated_sphere";
    auto it = find(msg->name.begin(), msg->name.end(), model_name);
    auto index = it - msg->name.begin();
```

- 이번 코드에서 다루지는 않지만, tf2는 기본적으로 오일러 각도 체계가 아닌 쿼터니언을 사용합니다. 오일러 각도 ⇒ 쿼터니언의 변환과 같은 API는 tf2 패키지에 구현되어 있습니다.

```cpp
#include "tf2/LinearMath/Quaternion.h"

    // tf2::Quaternion q;
    // q.setRPY(0, 0, msg->theta);

    t.transform.rotation.x = msg->pose[index].orientation.x;
    t.transform.rotation.y = msg->pose[index].orientation.y;
    t.transform.rotation.z = msg->pose[index].orientation.z;
    t.transform.rotation.w = msg->pose[index].orientation.w;

    // Send the transformation
    m_tf_broadcaster->sendTransform(t);
  }
```

- cpp 코드인만큼 CMakeLists.txt도 살펴보겠습니다.

```python
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(gazebo_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)

add_executable(tf2_broadcast src/tf2_broadcast.cpp)
ament_target_dependencies(tf2_broadcast tf2 tf2_ros gazebo_msgs geometry_msgs)
```

- tf2 broadcast를 위한 API를 정리하고 넘어갑시다.

|                                                 | Description      |
| ----------------------------------------------- | ---------------- |
| std::make_unique<tf2_ros::TransformBroadcaster> | broadcaster 생성 |
| geometry_msgs::msg::TransformStamped            | tf2 msg 생성     |
| tf_broadcaster->sendTransform(t)                | tf2 broadcast    |

### TF2 Listener (C++)

- animated_sphere의 tf2 listen을 통해 속도 topic을 publish하는 예시였습니다.

```python
# Terminal1 - gazebo
$ ros2 launch tf2_world tf2_world.launch.py

# Terminal2 - tf2 broadcaster
$ ros2 run py_tf2_tutorial tf2_broadcast

# Terminal3 - tf2 listener
$ ros2 run py_tf2_tutorial tf2_listener
[INFO] [1681998128.822169076] [tf2_frame_listener]: linear vel : 2.542703, angular vel : 1.810120
[INFO] [1681998128.922124745] [tf2_frame_listener]: linear vel : 2.501199, angular vel : 1.610775
...
```

- 코드의 include 부분입니다. tf2 listen을 위해 tf2_ros/transform_listener.h를 사용합니다.

```cpp
#include "rclcpp/rclcpp.hpp"
#include "tf2/exceptions.h"
#include "tf2_ros/transform_listener.h"
#include "tf2_ros/buffer.h"

#include "geometry_msgs/msg/twist.hpp"
#include "geometry_msgs/msg/transform_stamped.hpp"

using namespace std::chrono_literals;
using Twist = geometry_msgs::msg::Twist;
using TransformStamped = geometry_msgs::msg::TransformStamped;
```

- listener의 생성 시 Buffer가 필요한 부분도 동일하며, 대신, buffer가 unique_ptr 타입을 취하고 있어 pointer가 아닌 value를 전달함에 유의합니다.

```cpp
public:
  TF2Listener(): Node("tf2_frame_listener"){
    m_tf_buffer = std::make_unique<tf2_ros::Buffer>(this->get_clock());
    m_tf_listener = std::make_shared<tf2_ros::TransformListener>(*m_tf_buffer);
```

- python과 동일하게 lookupTransform은 3가지 매개변수를 필요로 합니다.
  1. Target frame
  2. Source frame
  3. time

```cpp
void timer_cb(){
  // Store frame names in variables that will be used to
  // compute transformations

  TransformStamped t;

  try {
    t = m_tf_buffer->lookupTransform(
      "animated_sphere", "world", tf2::TimePointZero
    );
```

### TF2 Add Frame (C++)

- tf2 구현의 마지막 예시입니다.

```python
# Terminal1 - gazebo
$ ros2 launch tf2_world tf2_world.launch.py

# Terminal2 - tf2 broadcaster
$ ros2 run cpp_tf2_tutorial tf2_broadcast

# Terminal3 - tf2 add frame
$ ros2 run cpp_tf2_tutorial tf2_add_frame
```

![orbit_frame.gif](/kr/ros2_foxy/images10/orbit_frame.gif?height=300px)

⇒ tf2 broadcaster와 큰 차이가 없으므로, 구체적인 설명은 생략하겠습니다.

## TF2 Example

이번 시간에는 실제 센서 데이터 ROS 2 패키지에서 tf2가 사용된 모습을 확인해보겠습니다.

![Untitled14.png](/kr/ros2_foxy/images10/Untitled14.png?height=300px)

두 센서 패키지를 참고해보려 합니다.

1. [intel realsense ros2 package](https://github.com/IntelRealSense/realsense-ros/tree/f18ca43483f216a02d1f1ea010ad52c9999e6c97)
2. [my ahrs imu package](https://github.com/CLOBOT-Co-Ltd/myahrs_ros2_driver/tree/master)

- 우선, my ahrs 코드입니다. publish*tf*라는 매개변수에 따라 tf2 데이터의 broadcast 여부가 달라집니다. 이렇게 해둔 이유는, 센서 자체를 통한 개발 시에는 tf2의 사용이 필요 없기 때문입니다.

```cpp
// publish tf
if (publish_tf_) {
  geometry_msgs::msg::TransformStamped tf;
  tf.header.stamp = now;
  tf.header.frame_id = parent_frame_id_;
  tf.child_frame_id = frame_id_;
  tf.transform.translation.x = 0.0;
  tf.transform.translation.y = 0.0;
  tf.transform.translation.z = 0.0;
  tf.transform.rotation = imu_data_msg.orientation;

  broadcaster_->sendTransform(tf);
}
```

⇒ [link1](https://github.com/CLOBOT-Co-Ltd/myahrs_ros2_driver/blob/d6a48f5121984aaa5ab850960eb363134c5f3ac9/myahrs_ros2_driver/src/myahrs_ros2_driver.cpp#L199)

- 다음으로, realsense-ros 패키지입니다. 이번에도 마찬가지로, 상황에 따라 statics tf / odom tf를 구별하여 구현해둔 모습을 볼 수 있습니다.

```cpp
void BaseRealSenseNode::publish_static_tf(const rclcpp::Time& t,
                                          const float3& trans,
                                          const tf2::Quaternion& q,
                                          const std::string& from,
                                          const std::string& to)
{
    geometry_msgs::msg::TransformStamped msg;
    msg.header.stamp = t;
    msg.header.frame_id = from;
    msg.child_frame_id = to;

    // Convert x,y,z (taken from camera extrinsics)
    // from optical cooridnates to ros coordinates
    msg.transform.translation.x = trans.z;
    msg.transform.translation.y = -trans.x;
    msg.transform.translation.z = -trans.y;

    msg.transform.rotation.x = q.getX();
    msg.transform.rotation.y = q.getY();
    msg.transform.rotation.z = q.getZ();
    msg.transform.rotation.w = q.getW();
    _static_tf_msgs.push_back(msg);
}

void BaseRealSenseNode::publishStaticTransforms(std::vector<rs2::stream_profile> profiles)
{
    // Publish static transforms
    if (_publish_tf)
    {
        for (auto &profile : profiles)
        {
            calcAndPublishStaticTransform(profile, _base_profile);
        }
        if (_static_tf_broadcaster)
            _static_tf_broadcaster->sendTransform(_static_tf_msgs);
    }
}
...

void BaseRealSenseNode::pose_callback(rs2::frame frame)
{
    ...
    geometry_msgs::msg::PoseStamped pose_msg;
    pose_msg.pose.position.x = -pose.translation.z;
    ...

    static tf2_ros::TransformBroadcaster br(_node);
    geometry_msgs::msg::TransformStamped msg;
    msg.header.stamp = t;
    msg.header.frame_id = DEFAULT_ODOM_FRAME_ID;
    msg.child_frame_id = FRAME_ID(POSE);
    msg.transform.translation.x = pose_msg.pose.position.x;
    msg.transform.translation.y = pose_msg.pose.position.y;
    msg.transform.translation.z = pose_msg.pose.position.z;
    msg.transform.rotation.x = pose_msg.pose.orientation.x;
    msg.transform.rotation.y = pose_msg.pose.orientation.y;
    msg.transform.rotation.z = pose_msg.pose.orientation.z;
    msg.transform.rotation.w = pose_msg.pose.orientation.w;

    if (_publish_odom_tf) br.sendTransform(msg);
```

⇒ [참고 링크 1](https://github.com/IntelRealSense/realsense-ros/blob/f18ca43483f216a02d1f1ea010ad52c9999e6c97/realsense2_camera/src/base_realsense_node.cpp#L994)

⇒ [참고 링크 2](https://github.com/IntelRealSense/realsense-ros/blob/f18ca43483f216a02d1f1ea010ad52c9999e6c97/realsense2_camera/src/base_realsense_node.cpp#L511)
