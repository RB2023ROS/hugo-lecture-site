---
title: "Lecture7. ROS 2 Topic and Programming"
date: 2023-04-28T10:51:28+09:00
draft: false
---

## ROS 2 Topic and Programming

강의를 열심히 따라오면서 개념을 잊어버렸을 수 있으므로 Topic에 대해서 간단히 개념 복습을 해보겠습니다.

- Topic은 Node들 사이에 데이터(Message)가 오가는 길(Bus)의 이름이며, 적절한 이름으로 데이터를 송수신하지 않으면 원하는 동작을 수행할 수 없습니다.

![topic.gif](/kr/ros2_foxy/images7/topic.gif?height=300px)

image from : [docs.ros.org](https://docs.ros.org/en/foxy/Tutorials/Topics/Understanding-ROS2-Topics.html)

- Topic의 중요한 특징으로 Topic은 여러 Node들로부터 데이터를 받을 수 있고, 전송 시에도 여러 Node들에게 전송이 가능한 방식입니다.

![topic2.gif](/kr/ros2_foxy/images7/topic2.gif?height=300px)

image from : [docs.ros.org](https://docs.ros.org/en/foxy/Tutorials/Topics/Understanding-ROS2-Topics.html)

topic을 통해 데이터가 전달되는 과정은 다음과 같습니다.

1. 데이터를 보내는 주체인 **publisher**는 데이터를 받는 주체, **subscriber**를 지정한 뒤, topic을 통해 원하는 정보를 전달합니다. 이것을 **topic publish**라고 부르지요.
2. 반대로, 데이터를 받는 주체인 **subscriber** 입장에선 topic을 통해 데이터를 받게 되며 이것은 **topic subscribe**라고 불립니다.

- Topic을 사용할 시 Publisher, Subscriber 사이에는 특정한 형식을 갖는 데이터, msg가 오갑니다. 원하는 데이터에 적합한 topic msg를 사용하면 더욱 효율적인 로봇 프로그래밍이 가능해집니다.

![Untitled.png](/kr/ros2_foxy/images7/Untitled.png?height=300px)

image from : [ethz.ch](https://ethz.ch/content/dam/ethz/special-interest/mavt/robotics-n-intelligent-systems/rsl-dam/ROS2021/lec4/ROS%20Course%20Slides%20Course%204.pdf)

- 특정 message가 어떻게 구성되어 있는지 알고 싶을 때는 다음과 같은 커멘드 라인을 사용합니다.

```bash
$ ros2 interface show geometry_msgs/msg/Twist
# This expresses velocity in free space broken into its linear and angular parts.

Vector3  linear
Vector3  angular
```

## SRC Gazebo

이번 ROS 2 강의의 실습들은 Road Balance의 차량형 로봇 SRC를 통해 진행하고자 합니다. 실습을 진행하기 전 Gazebo 환경을 함께 세팅해보겠습니다.

- src_gazebo 실행 세팅

```bash
cd ~/ros2_ws/src/du2023-ros2
./setup_scripts.sh

# Package build
cbp src_controller
source install/local_setup.bash
cbp src_gazebo
source install/local_setup.bash
```

- 모든 준비가 완료되었다면 ros2 launch를 통해 이번 실습에 사용될 Gazebo 환경을 실행합니다.

```python
ros2 launch src_gazebo empty_world.launch.py
```

- 아무것도 없는 Gazebo 환경에 차량형 로봇이 등장할 것입니다. 함께 등장하는 **rqt_robot_steering**을 통해 로봇을 움직여보세요.

![src_turn.gif](/kr/ros2_foxy/images7/src_turn.gif?height=400px)

⇒ 현재 /cmd_vel topic을 사용해서 로봇을 조종하고 있는 상황입니다. 지난 시간 학습한 rqt tool들을 사용해서 시스템 분석을 진행해봅시다.

- rqt_graph를 통해 해당 topic의 publisher / subscriber 확인

![Untitled1.png](/kr/ros2_foxy/images7/Untitled1.png?height=500px)

- topic 조회

```python
$ ros2 topic list
/clicked_point
/clock
/cmd_vel
/dynamic_joint_states
/forward_position_controller/commands
/goal_pose
/imu/data
/initialpose
/joint_states
/logi_camera_sensor/camera_info
/logi_camera_sensor/image_raw
/parameter_events
/performance_metrics
/robot_description
/rosout
/scan
/steering_angle_middle
/tf
/tf_static
/throttling_vel_middle
/velocity_controller/commands
```

- 특정 topic 정보 확인

```python
$ ros2 topic info /forward_position_controller/commands
Type: std_msgs/msg/Float64MultiArray
Publisher count: 1
Subscription count: 1
```

⇒ 이렇게 시스템에 대한 분석을 해두면 코딩 전 명확한 목표와 기능을 세울 수 있고, 많은 시간을 단축할 수 있습니다.

### ROS 2 Topic 프로그래밍 - python

#### example 1 - cmd_vel publish

- 준비된 예시를 우선 실행해봅시다.

```python
# Terminal 1
ros2 launch src_gazebo empty_world.launch.py

# Terminal 2
cd ~/ros2_ws
colcon build --packages-select py_topic_tutorial
source install/local_setup.bash
ros2 run py_topic_tutorial topic_pub_node
```

⇒ 로봇이 원을 그리며 움직이기 시작합니다.

![circle_src.gif](/kr/ros2_foxy/images7/circle_src.gif?height=400px)

## 코드 분석

- ROS 2에서 파이썬 프로그래밍을 하기 위해서, rclpy.node.Node를 import 해야 했습니다. 더불어, Topic에 사용되는 msg, **Twist**를 import하는 부분도 눈여겨봅시다.

```python
import random

from geometry_msgs.msg import Twist
import rclpy
from rclpy.node import Node
```

- 핵심이 되는 TwistPubNode 클래스의 생성자부터 살펴보겠습니다.

```python
class TwistPubNode(Node):

    def __init__(self):
        super().__init__('twist_pub_node')

        self.get_logger().info(
            f'TwistPubNode Created at {self.get_clock().now().to_msg().sec}'
        )

        # self.twist_publisher = self.create_publisher(Twist, "twist_topic", 10)
        self.twist_publisher = self.create_publisher(Twist, '/cmd_vel', 10)
```

- **create_publisher**는 topic publisher를 생성하는 함수로 3개의 매개변수를 받으며 각각에 대한 설명은 다음과 같습니다.

| Parameters   | Description                                                                                                           |
| ------------ | --------------------------------------------------------------------------------------------------------------------- |
| message type | Topic 통신에 사용될 msg Type                                                                                          |
| topic name   | 사용할 Topic 이름을 지정 (이 이름을 잘못 설정하면 존재하지 않는 topic에 Publish하는 오류 상황이 발생하니 주의합니다.) |
| queue_size   | topic 통신 시 대기열의 크기                                                                                           |

- 이어서, timer callback에서 publish가 이루어집니다. 인스턴스화한 publisher에서 **publish** 함수를 사용하며 매개변수로 message가 전달됩니다.

```python
        ...
        self.create_timer(0.2, self.timer_callback)

    def timer_callback(self):
        """Timer will run this function periodically."""
        msg = Twist()

        # Fill in msg with compatible values
        msg.linear.x = 0.5
        msg.angular.z = 1.0

        self.twist_publisher.publish(msg)
```

#### example 2 - scan subscription

- 이번 예시는 2D 라이다 데이터를 다뤄보고자 합니다. 아래의 커멘드 라인들을 실행해주세요.

```python
# Terminal 1
$ ros2 launch src_gazebo empty_world.launch.py

# Terminal 2
$ ros2 run py_topic_tutorial topic_sub_node
[INFO] [1672494602.141200628] [scan_sub_node]:
msg.ranges[0] : inf
msg.ranges[30] : inf
msg.ranges[60] : 0.7975894212722778
msg.ranges[90] : inf
msg.ranges[119] : inf
...
```

- 예시를 실행시킨 상황에서 Gazebo 상의 로봇 주변으로 물체를 등장시킨 뒤, topic node를 실행하고 있는 터미널 창의 변화를 살펴봅시다.

![Untitled2.png](/kr/ros2_foxy/images7/Untitled2.png?height=300px)

⇒ scan data의 ranges는 사진과 같은 거리 데이터를 담고 있으며, 각 지점에 해당하는 lidar data를 터미널에 표시하는 예시였습니다.

![Untitled3.png](/kr/ros2_foxy/images7/Untitled3.png?height=300px)

### 코드 분석

- 이번 예시는 2D lidar 데이터를 위해 제공되는 sensor_msgs/msg/LaserScan msg를 사용하고 있습니다.

```python
import rclpy
from rclpy.node import Node

from sensor_msgs.msg import LaserScan
```

⇒ 해당 msg에 대한 정보를 조회하는 방법, 이제는 모두 숙지하셨으리라 믿습니다.

```python
$ ros2 interface show sensor_msgs/msg/LaserScan
```

- ScanSubNode 클래스의 생성자에서는 scan topic의 **subscriber**가 생성됩니다.

```python
class ScanSubNode(Node):

    def __init__(self):

        super().__init__('scan_sub_node')

        queue_size = 10  # Queue Size

        self.pose_subscriber = self.create_subscription(
            LaserScan, 'scan', self.sub_callback, queue_size
        )
```

- **create_subscription**은 4개의 매개변수를 필요로 합니다. 각각의 매개변수를 보면서 왜 이런 값들이 필요할지 생각해보세요.

| Parameters         | Description                                                                                           |
| ------------------ | ----------------------------------------------------------------------------------------------------- |
| Message Type       | Topic 통신에 사용될 msg Type                                                                          |
| topic name         | 데이터를 Subscribe할 Topic의 이름                                                                     |
| subscribe callback | 데이터가 전달될 때마다 실행되는 callback 함수. 전달된 데이터를 통해 실행할 작업이 이 함수 안에 구현됨 |
| queue_size         | create_publisher때와 마찬가지로, 대기열의 크기                                                        |

- 더불어, sub_callback의 첫번쨰 매개변수는 항상 topic message data가 됩니다. 대부분의 구현에서는 해당 message에서 원하는 데이터를 추출한 다음, 이후의 처리를 진행합니다.

```python
		def sub_callback(self, msg):
        """Timer will run this function periodically."""
        self.get_logger().info(f' \
            \nmsg.ranges[0] : {msg.ranges[0]} \
            \nmsg.ranges[30] : {msg.ranges[30]} \
            \nmsg.ranges[60] : {msg.ranges[60]} \
            \nmsg.ranges[90] : {msg.ranges[90]} \
            \nmsg.ranges[119] : {msg.ranges[119]} \
        ')
```

#### example 3 - parking

- 이번 예시를 실행하기 전에, empty_world.launch.py를 수정해야 합니다. gazebo_ros server의 실행 시 전달되는 환경 파일을 변경하는 작업입니다.

```bash
		# gazebo
    pkg_gazebo_ros = FindPackageShare(package='gazebo_ros').find('gazebo_ros')
    pkg_path = os.path.join(get_package_share_directory('src_gazebo'))
    # world_path = os.path.join(pkg_path, 'worlds', 'empty_world.world')
    world_path = os.path.join(pkg_path, 'worlds', 'wall_world.world')
```

- 이번 예시는 일정 거리 내 벽이 검출되면 자동으로 로봇을 정지시킵니다.

```python
# Terminal 1
$ ros2 launch src_gazebo empty_world.launch.py

# Terminal 2
$ ros2 run py_topic_tutorial parking_node
...
[INFO] [1672563255.067421128] [parking_node]: Distance from Front Object : 0.6280616521835327
[INFO] [1672563255.173734128] [parking_node]: Distance from Front Object : 0.5546504259109497
[INFO] [1672563255.281276729] [parking_node]: Distance from Front Object : 0.5124648809432983
[INFO] [1672563255.389067729] [parking_node]: ==== Parking Done!!! ====
...
```

![parking.gif](/kr/ros2_foxy/images7/parking.gif?height=300px)

이 기능을 위해 Topic Publish와 Subscribe가 모두 필요합니다.

1. **/cmd_vel topic publish**
2. **/scan topic subscribe**

- 따라서, Node의 생성자에서 Publisher와 Subscriber를 모두 생성합니다.

```python
class ParkingNode(Node):

    def __init__(self):

        super().__init__('parking_node')
        queue_size = 10  # Queue Size

        self.twist_publisher = self.create_publisher(Twist, 'cmd_vel', queue_size)
        self.scan_subscriber = self.create_subscription(
            LaserScan, 'scan', self.sub_callback, queue_size
        )
```

- sub_callback에서 전방 (msg.ranges[60]) 물체와의 거리를 탐지하고, 이 거리가 0.5m 이하가 될 시 정지 topic을 publish 합니다.

```python
		def sub_callback(self, msg):

        twist_msg = Twist()
        distance_forward = msg.ranges[60]

        if distance_forward > 0.5:
            self.get_logger().info(f'Distance from Front Object : {distance_forward}')
            twist_msg.linear.x = 0.5
            self.twist_publisher.publish(twist_msg)
        else:
            self.get_logger().info('==== Parking Done!!! ====\n')
            twist_msg.linear.x = 0.0
            self.twist_publisher.publish(twist_msg)
```

### ROS 2 Topic Programming - C++

#### example 1 - cmd_vel publish

- 준비된 예시들을 빌드 해봅시다.

```bash
cbp cpp_topic_tutorial
source install/local_setup.bash

# Terminal 1
ros2 launch src_gazebo empty_world.launch.py

# Terminal 2
ros2 run cpp_topic_tutorial cpp_topic_pub_node
```

⇒ 구현 자체는 동일합니다. 로봇이 원을 그리며 움직이기 시작합니다.

### 코드 분석

- 로봇을 움직이기 위해서 **geometry_msgs/msg/Twist** 형식의 topic message type을 사용해야 함을 배운 바 있습니다. 이를 위해서 다음과 같이, 헤더를 include 하면 됩니다.

```cpp
#include "geometry_msgs/msg/twist.hpp"
#include "rclcpp/rclcpp.hpp"
```

{{% notice tip %}}
단, 파이썬과 달리 c++ 헤더는 snake_case를 취하며, 코드 사용 시 CamelCase를 사용함에 주의합니다.
{{% /notice %}}

|        | Implementation                         |
| ------ | -------------------------------------- |
| Python | from geometry_msgs.msg import Twist    |
| Cpp    | #include "geometry_msgs/msg/twist.hpp" |

- publisher는 **create_publisher** 함수를 통해 생성할 수 있습니다.

|                         | Implementation                           |
| ----------------------- | ---------------------------------------- |
| rclcpp::Publisher<type> | template을 통해 message type을 적습니다. |
| topic_name              | publish할 topic의 이름                   |
| queue size              | 일전과 동일                              |

```cpp
rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr twist_publisher;
...
twist_publisher = this->create_publisher<geometry_msgs::msg::Twist>("turtle1/cmd_vel", 10);
```

{{% notice note %}}
geometry_msgs::msg::Twist와 같이 타입이 길기 때문에 using을 사용하여 축약하곤 합니다.
{{% /notice %}}

#### example 2 - scan subscription

- 로봇 내 장착된 2D Lidar data를 적절히 표시하는 예시였습니다.

```bash
# Terminal 1
$ ros2 launch src_gazebo empty_world.launch.py

# Terminal 2
$ ros2 run cpp_topic_tutorial cpp_topic_sub_node
[INFO] [1672494602.141200628] [scan_sub_node]:
msg.ranges[0] : inf
msg.ranges[30] : inf
msg.ranges[60] : 0.7975894212722778
msg.ranges[90] : inf
msg.ranges[119] : inf
...
```

![Untitled4.png](/kr/ros2_foxy/images7/Untitled4.png?height=300px)

### 코드 분석

- subscriber는 create_subscription 함수를 통해 생성할 수 있습니다.

|                        | Implementation                               |
| ---------------------- | -------------------------------------------- |
| rclcpp::Subscription<> | template을 통해 message type을 적습니다.     |
| topic_name             | subscribe할 topic의 이름                     |
| queue size             | 일전과 동일                                  |
| callback               | std::bind를 통해 callback 함수를 전달합니다. |

- 예시에서는 callback 함수의 매개변수가 1개이기에 std::placeholders::\_1이 사용되었습니다.

```cpp
LaserSubNode() : Node("laser_sub_node")
{
  laser_subscriber = this->create_subscription<LaserScan>(
    "scan", 10,
    std::bind(&LaserSubNode::sub_callback, this, std::placeholders::_1)
  );
}
```

- callback 함수의 첫번째 매개변수인 데이터는 SharedPtr 타입이 사용된다는 것에 주의하며, 때문에 레퍼런스를 사용할 수 없습니다. (자주 있는 실수이므로 오류 상황도 함께 살펴보겠습니다.)

```cpp
void sub_callback(const LaserScan::SharedPtr msg){
  RCLCPP_INFO(this->get_logger(), "msg.ranges[0]: '%f'", msg->ranges[0]);
  RCLCPP_INFO(this->get_logger(), "msg.ranges[30]: '%f'", msg->ranges[30]);
  RCLCPP_INFO(this->get_logger(), "msg.ranges[60]: '%f'", msg->ranges[60]);
  RCLCPP_INFO(this->get_logger(), "msg.ranges[90]: '%f'", msg->ranges[90]);
  RCLCPP_INFO(this->get_logger(), "msg.ranges[119]: '%f'\n", msg->ranges[119]);
}
```

- SharedPtr은 rclcpp 내부적으로 정의되어 있는 스마트 포인터입니다. 현 상황에서 레퍼런스를 사용하기 위해서는 ConstSharedPtr을 사용하면 됩니다.

```cpp
void sub_callback(const LaserScan::ConstSharedPtr &msg){
  RCLCPP_INFO(this->get_logger(), "msg.ranges[0]: '%f'", msg->ranges[0]);
  RCLCPP_INFO(this->get_logger(), "msg.ranges[30]: '%f'", msg->ranges[30]);
  RCLCPP_INFO(this->get_logger(), "msg.ranges[60]: '%f'", msg->ranges[60]);
  RCLCPP_INFO(this->get_logger(), "msg.ranges[90]: '%f'", msg->ranges[90]);
  RCLCPP_INFO(this->get_logger(), "msg.ranges[119]: '%f'\n", msg->ranges[119]);
}
```

### example 3 - parking

- 일정 거리 내 벽이 검출되면 자동으로 로봇이 정지하는 예시였습니다.

```python
# Terminal 1
$ ros2 launch src_gazebo empty_world.launch.py

# Terminal 2
$ ros2 run cpp_topic_tutorial cpp_parking_node
...
[INFO] [1672563255.067421128] [parking_node]: Distance from Front Object : 0.6280616521835327
[INFO] [1672563255.173734128] [parking_node]: Distance from Front Object : 0.5546504259109497
[INFO] [1672563255.281276729] [parking_node]: Distance from Front Object : 0.5124648809432983
[INFO] [1672563255.389067729] [parking_node]: ==== Parking Done!!! ====
...
```

![parking.gif](/kr/ros2_foxy/images7/parking.gif?height=300px)

- 이 기능을 위해 Topic Publish와 Subscribe가 모두 필요합니다. 따라서, Node의 생성자에서 Publisher와 Subscriber를 모두 생성합니다.

```cpp
public:
  ParkingNode() : Node("laser_sub_node")
  {
    twist_publisher = this->create_publisher<Twist>("cmd_vel", 10);

    laser_subscriber = this->create_subscription<LaserScan>(
      "scan", 10,
      std::bind(&ParkingNode::sub_callback, this, std::placeholders::_1)
    );
  }
```

- sub_callback에서 전방 (msg.ranges[60]) 물체와의 거리를 탐지하고, 이 거리가 0.5m 이하가 될 시 정지 topic을 publish 합니다.

```cpp
void sub_callback(const LaserScan::SharedPtr msg){

    auto distance_forward = msg->ranges[60];

    if(distance_forward > 0.5){
      auto msg = Twist();
      msg.linear.x = 0.5;
      twist_publisher->publish(msg);
    }else{
      auto msg = Twist();
      msg.linear.x = 0.0;
      twist_publisher->publish(msg);
    }
  }
```

- rclcpp의 경우 CMakeLists.txt를 수정하여 node를 컴파일해야 했습니다. 이번 예시에서도 마찬가지 작업이 필요하며, 특히 코드에서 include 하였던 sensor_msgs geometry_msgs를 바인딩 하는 점에 유의하세요.

```cpp
# find_package를 통해 종속성들을 추가합니다.
find_package(rclcpp REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
...

add_executable(cpp_topic_pub_node src/cpp_topic_pub_node.cpp)
ament_target_dependencies(cpp_topic_pub_node rclcpp geometry_msgs)

add_executable(cpp_topic_sub_node src/cpp_topic_sub_node.cpp)
ament_target_dependencies(cpp_topic_sub_node rclcpp sensor_msgs)

add_executable(cpp_parking_node src/cpp_parking_node.cpp)
ament_target_dependencies(cpp_parking_node rclcpp sensor_msgs geometry_msgs)
```

### Custom Interface Generation

> Topic의 msg, Service의 srv, Action의 action… 각각 통신 메커니즘마다. 사용하는 데이터 타입은 이제 익숙해지셨으리라 생각합니다. 그리고 ROS 2 에서는 자주 사용되는 데이터 타입들을 제공하고 있습니다.

- 몇가지를 함께 조회해 봅시다.

```python
ros2 interface show msg / srv / action
```

이렇게 미리 제공되는 데이터 타입들은 common interfaces라는 이름으로 제공되고 있습니다.

⇒ [https://github.com/ros2/common_interfaces](https://github.com/ros2/common_interfaces)

- 그런데, 만약 나만의 새로운 데이터 타입을 만들고 싶다면 어떻게 해야 할까요? ROS 2에서는 custom interface의 생성을 제공하고 있습니다. 함께 실습해봅시다.

> ROS 2에서 custom interface를 만들기 위해서는 **C++ Package**에서 작업이 이루어져야 합니다. C++ package는 build type ament_cmake를 사용하는 package였습니다.

```bash
$ ros2 pkg create --build-type ament_cmake <package-name>
$ ros2 pkg create --build-type ament_cmake custom_interfaces
```

- 해당 패키지 내 action, msg, srv 라는 폴더를 만들고 해당 폴더 안에 나만의 인터페이스를 작성합니다.

![Untitled5.png](/kr/ros2_foxy/images7/Untitled5.png?height=300px)

- example - MyFirstMsg.msg

```python
string name
float32 height
float32 weight
uint8 age
```

- example - TurningControl.srv

```python
uint32 time_duration
float64 angular_vel_z
float64 linear_vel_x
---
bool success
```

- example - Parking.action

```python
#goal definition
bool start_flag
---
#result definition
string message
---
#feedback definition
float32 distance
```

- custom interface 내부에는 primitive type / pre-defined type이 존재 가능합니다.

⇒ primitive type은 아래의 그림을 참고하시고

⇒ pre-defined type은 해당 ROS 2 시스템에서 이미 정의된 (std_msgs, std_srvs, etc…)가 가능합니다.

![Untitled6.png](/kr/ros2_foxy/images7/Untitled6.png?height=300px)

image from : [https://design.ros2.org/articles/interface_definition.html](https://design.ros2.org/articles/interface_definition.html)

⇒ 이제 해당 interface를 rclpy, rclcpp에서 사용 가능하도록 해봅시다.

- CMakeLists.txt 수정

```cpp
rosidl_generate_interfaces(${PROJECT_NAME}
  "action/DataGen.action"
  "action/Parking.action"
  "srv/CircleTurtle.srv"
  "srv/TurtleJail.srv"
)
```

- package.xml 수정

```xml
<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

- 패키지 빌드와 소싱

```bash
$ colcon build --packages-select custom_interfaces
Starting >>> custom_interfaces
Finished <<< custom_interfaces [0.62s]

Summary: 1 package finished [0.84s]

$ source install/local_setup.bash
```

- 이제 ROS 2에서 해당 인터페이스를 사용할 수 있습니다 (단, 현재 custom_interfaces 패키지는 ros2_ws에 존재하기 때문에 해당 workspace를 소싱한 터미널에서만 조회가 가능합니다. )

```bash
$ ros2 interface show custom_interfaces/srv/TurtleJail
float32 width
float32 height
---
bool success
```

⇒ 그럼, custom interface의 실제 사용은 어떻게 할까요? 방금 생성한 custom_interfaces/srv/TurtleJail를예시로 살펴보겠습니다.

- rclpy

```python
from custom_interfaces.srv import TurtleJail
```

- rclcpp

```python
#include "custom_interfaces/srv/turtle_jail.hpp"

using TurtleJail = custom_interfaces::srv::TurtleJail;
```

- python을 사용하는 패키지에서 별도 작업 없이 사용 가능하지만 C++ 패키지는 코딩 시 CMakeLists.txt의 수정이 필요합니다.

```python
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(<custom-interface-pkg> REQUIRED)                         # CHANGE

add_executable(talker src/publisher_member_function.cpp)
ament_target_dependencies(talker rclcpp <custom-interface-pkg>)         # CHANGE

install(TARGETS
  talker
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```

실제 ROS 2 호환 카메라 패키들을 보아도 필요에 따라 custom interface들을 선언한 모습을 확인 가능합니다.

- realsense_ros

![Untitled6.png](/kr/ros2_foxy/images7/Untitled6.png?height=300px)
