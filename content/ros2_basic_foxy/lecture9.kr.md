---
title: "Lecture9 - C++ Programming Again, Outro"
date: 2023-01-02T12:38:58+09:00
draft: false
---

## C++ Programming Again

> 실질적으로 C++을 통해 ROS 2 개발이 많이 이루어진다 이야기하였습니다. Topic Publish와 Subscriber의 C++코드들도 turtlesim 예제와 함꼐 간단하게 정리해보았습니다.

### Topic Publish 예제 - Random Movement Turtle

- 예제 실행

```bash
cbp cpp_topic_tutorial
source install/local_setup.bash

# Terminal 1
ros2 run turtlesim turtlesim_node
# Terminal 2
ros2 run cpp_topic_tutorial topic_pub_node
```

![lec9_0.png](/kr/ros2_basic_foxy/images9/lec9_0.png?height=400px)

⇒ 거북이가 마구잡이로 움직이기 시작할 것입니다

- 로봇을 움직이기 위해서는 **geometry_msgs/msg/Twist** 형식의 topic message type을 사용해야 함을 배운 바 있습니다. 이를 위해서 다음과 같이, 헤더를 include 하면 됩니다.

```cpp
#include "geometry_msgs/msg/twist.hpp"
#include "rclcpp/rclcpp.hpp"
```

{{% notice note %}}
단, 파이썬과 달리 c++ 헤더는 snake_case를 취하며, 코드 사용 시 CamelCase를 사용합니다.

{{% /notice %}}

```cpp
rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr twist_publisher;
...
twist_publisher = this->create_publisher<geometry_msgs::msg::Twist>("turtle1/cmd_vel", 10);
```

publisher는 **create_publisher** 함수를 통해 생성할 수 있습니다.

- **<>** 안에는 message type을 적어주고
- 첫 번째 매개변수는 **생성할 topic의 이름**
- 두 번째 매개변수로 **queue size**를 전달합니다.

{{% notice tip %}}

geometry_msgs::msg::Twist와 같이 타입이 길기 때문에 using을 사용하여 축약하곤 합니다.
{{% /notice %}}

### Topic Subscribe 예제 - Turtle Pose Sub

- turtlesim 상의 거북이의 위치를 Subscribe 받습니다.

```bash
# Terminal 1
ros2 run turtlesim turtlesim_node
# Terminal 2
ros2 run cpp_topic_tutorial topic_sub_node
[INFO] [1666434438.709117200] [turtlepose_sub_node]: x: 5.544/ y: 5.544/ theta: 0.000/ lin_vel: 0.000, ang_vel: 0.000
[INFO] [1666434438.725522800] [turtlepose_sub_node]: x: 5.544/ y: 5.544/ theta: 0.000/ lin_vel: 0.000, ang_vel: 0.000
[INFO] [1666434438.741218300] [turtlepose_sub_node]: x: 5.544/ y: 5.544/ theta: 0.000/ lin_vel: 0.000, ang_vel: 0.000
...
```

- 로직 자체는 어렵지 않기에 API에 집중하여 분석해보겠습니다.

```cpp
TwistPubNode() : Node("twist_pub_node") {
    twist_publisher = this->create_publisher<geometry_msgs::msg::Twist>("turtle1/cmd_vel", 10);
    timer = this->create_wall_timer(
        std::chrono::milliseconds(500),
        std::bind(&TwistPubNode::timer_callback, this)
    );
  }
```

subscriber는 create_subscription 함수를 통해 생성할 수 있습니다.

- <> 안에는 **message type**을 적어주고
- 첫 번째 매개변수는 subscribe **topic의 이름**
- 두 번째 매개변수로 **queue size**
- 세 번째 매개변수로는 std::bind를 통해 callback 함수를 전달합니다.

callback의 매개변수가 1개이기에 이를 알려야 하며, **std::placeholders::\_1**이 사용되었습니다.

sub_callback의 첫번째 매개변수인 데이터는 SharedPtr 타입이 사용된다는 것에 주의하며, 때문에 레퍼런스를 사용할 수 없습니다.

> Service Client와 Server의 C++코드들도 분석해봅시다.

### Service Client 예제 - Turtle Spawn

- Turtle Spawn 예제 - 영상과 같이 우리가 내린 명령대로 거북이가 등장한 모습을 볼 수 있습니다.

```bash
cbp cpp_service_tutorial
source install/local_setup.bash

# Terminal 1
ros2 run turtlesim turtlesim_node

# Terminal 2
ros2 run cpp_service_tutorial turtle_spawn_client
> Turtle X position : 5.0
> Turtle Y position : 5.0
> Turtle Angle : 0.0
> Turtle Name : my_turtle
[INFO] [1666434554.012857700] [spawn_turtle_node]: Turtle Named : my_turtle Spawned Successfully.
```

![spawn_turtle.gif](/kr/ros2_basic_foxy/images9/spawn_turtle.gif?height=300px)

**create_client**는 다음과 같은 정보를 요합니다.

- 사용하는 **srv 타입**
- request할 **service 이름**

**wait_for_service**를 통해 Service Server의 존재 여부를 체크할 수 있습니다.

```cpp
...
public:
  SpawnTurtle() : Node("spawn_turtle_node"){
    spawn_client = this->create_client<Spawn>("spawn");

    while (!spawn_client->wait_for_service(1s)) {
      if (!rclcpp::ok()) {
        RCLCPP_ERROR(this->get_logger(),
                    "Interrupted while waiting for the service. Exiting.");
        exit(0);
      }
      RCLCPP_INFO(this->get_logger(), "service not available, waiting again...");
    }
  }
```

- **send_request**를 호출함으로 Service call이 가능합니다. 이때, send_request의 반환값을 살펴보면, 비동기 실행을 하고 있음을 알 수 있습니다.

```cpp
auto send_request(){
	get_user_input("> Turtle X position : ", spawn_request->x);
	get_user_input("> Turtle Y position : ", spawn_request->y);
	get_user_input("> Turtle Angle : ", spawn_request->theta);

	get_user_input("> Turtle Name : ", spawn_request->name);

	return spawn_client->async_send_request(spawn_request);
}
```

- **spin_until_future_complete**는 async_send_request의 비동기 promise를 받아 실행 완료까지 대기하게 됩니다.

```cpp
int main(int argc, char **argv) {
  rclcpp::init(argc, argv);

  auto node = std::make_shared<SpawnTurtle>();
  auto result = node->send_request();

  // Wait for the result.
  if (rclcpp::spin_until_future_complete(node, result) == rclcpp::FutureReturnCode::SUCCESS) {
    RCLCPP_INFO(node->get_logger(), "Turtle Named : %s Spawned Successfully.", result.get()->name.c_str());
  } else {
    RCLCPP_ERROR(node->get_logger(), "Failed to call service add_two_ints");
  }

  rclcpp::shutdown();

  return 0;
}
```

- promise의 반환 완료값은 enumerator type을 갖고 있으며, 공식 문서를 참고하였습니다.

![lec9_1.png](/kr/ros2_basic_foxy/images9/lec9_1.png?height=200px)

image from : [docs.ros2.org](https://docs.ros2.org/foxy/api/rclcpp/namespacerclcpp.html#a7b4ff5f1e516740d7e11ea97fe6f5532)

### Service Server 예제 - Turtle Turn

- 거북이가 원을 그리며 돌기 시작하다가, 일정 시간이 지나면 움직임을 멈추게 됩니다. rqt의 **Service Caller**를 사용하며, **turtle_turn** service를 사용하시면 됩니다.

```bash
# Terminal 1
ros2 run turtlesim turtlesim_node
# Terminal 2
ros2 run cpp_service_tutorial turtle_circle_server
# Terminal 3
[INFO] [1666435009.330619900] [turtle_circle_server]: Turtle Turning Server Started, Waiting for Request...
[INFO] [1666435026.497839300] [turtle_circle_server]: 0.00 Seconds Passed
[INFO] [1666435026.498143700] [turtle_circle_server]: 0.00 Seconds Passed
[INFO] [1666435026.498297800] [turtle_circle_server]: 0.00 Seconds Passed
[INFO] [1666435026.498427400] [turtle_circle_server]: 0.00 Seconds Passed
...
```

![turtle_circle.gif](/kr/ros2_basic_foxy/images9/turtle_circle.gif?height=300px)

- 전체 코드 중 Service Server를 생성하는 핵심을 위주로 살펴봅시다.

```cpp
class TurtleCircleNode : public rclcpp::Node {
private:
  rclcpp::Service<SetBool>::SharedPtr bool_server;

  void turtle_circle(){
    ...
    }
    ...
  }

  void server_callback(const std::shared_ptr<SetBool::Request> request,
                      const std::shared_ptr<SetBool::Response> response){
    if (request->data) {
      turtle_circle();
    }

    response->success = true;
    response->message = "Turtle successfully drawed Circle";
  }

public:
  TurtleCircleNode() : Node("turtle_circle_server"){
    bool_server = this->create_service<SetBool>(
      "turtle_circle",
      std::bind(&TurtleCircleNode::server_callback, this, std::placeholders::_1, std::placeholders::_2)
```

**create_service**는 다음과 같은 정보를 필요로 합니다.

- 사용하는 **srv 타입** - SetBool
- 생성할 **service 이름** - turtle_circle
- request 시 실행될 **callback** - server_callback

service callback은 request와 response 두 데이터를 매개변수로 받습니다. 때문에, **std::placeholders::\_1, \_2**가 명시된 것이 보입니다. 더불어, 거북이를 움직이기 위해 Publisher도 하나 생성하였습니다.

이렇게 C++ 코드까지 살펴보았는데요, Action의 C++ 코드는 이번 강의에서는 넘어가도록 하겠으며, 링크를 남겨두겠습니다.

[https://docs.ros.org/en/foxy/Tutorials/Intermediate/Writing-an-Action-Server-Client/Cpp.html](https://docs.ros.org/en/foxy/Tutorials/Intermediate/Writing-an-Action-Server-Client/Cpp.html)

---

## About Real Robot

> 지금까지 우리의 실습을 책임졌던 src 로봇이 어떻게 만들어졌는지 살펴보면서, 실제 로봇 제품의 개발 과정을 살펴봅시다.

![lec9_2.png](/kr/ros2_basic_foxy/images9/lec9_2.png?height=300px)

항상 로봇 시스템을 제작하기 전, 저는 부품의 수급부터 계획합니다.

- **메인 PC** - 라즈베리파이, 젯슨 나노, 라떼판다, 오드로이드, Mini PC, etc…
- **MCU 보드** - 아두이노, ESP 시리즈, Teensy, Node MCU etc…
- **Actuator** - 서보/스텝 모터, BLDC 모터, (엔코더 장착 여부, 전압 고려) …
- **Sensor** - Lidar, Camera (Monocular, Stereo, RGB-D, 360도 카메라…), Lidar, Range Sensor, etc…
- **외관** - 알루미늄, 3D 프린팅, 절곡, 카본 플레이트, CNC 가공…

이러한 이유로 로봇 제작에는 일정 비용과 경험이 요구됩니다. 하지만 모두 피와 살이 되니 아낌없이 투자하시기 바랍니다.

![lec9_3.png](/kr/ros2_basic_foxy/images9/lec9_3.png?height=300px)

- image from : [로봇 신문](http://m.irobotnews.com/news/articleView.html?idxno=8950)

로봇 개발의 초기부터 너무 모든 것을 고려할 필요는 없습니다. 작은 사이즈의 테스트 베드를 구축하여 기능과 경험을 충분히 쌓은 뒤, 점차 하드웨어를 개선해 나가면서 업그레이드를 진행합니다.

![lec9_4.png](/kr/ros2_basic_foxy/images9/lec9_4.png?height=300px)

> 함께하는 개발은 언제나 즐겁습니다.

- 개발을 진행하고 하드웨어를 업그레이드해가면서, CAD 파일도 함께 관리합니다. 비록 SRC는 금속 가공이 들어간 것은 아니라 비교적 간단하지만, 규모 있는 로봇 프로젝트에서는 반드시 거쳐야 하는 과정입니다.

![lec9_5.png](/kr/ros2_basic_foxy/images9/lec9_5.png?height=300px)

- CAD 파일로부터 urdf를 추출하고, 움직이는 joint들에는 gazebo controller plugin을, 센서 joint에는 gazebo sensor plugin을 적용하여 시뮬레이션을 제작합니다.

![lec9_6.png](/kr/ros2_basic_foxy/images9/lec9_6.png?height=300px)

시뮬레이션을 통해 제어기와 Odometry Driver, 자율 주행 로직의 검증이 완료되면, 실제 로봇에 이를 적용합니다. 적용과 검증을 반복하면서 로봇을 계속해서 개선해나갑니다.

![src_outdoor.gif](/kr/ros2_basic_foxy/images9/src_outdoor.gif?height=250px)

![src_indoor.gif](/kr/ros2_basic_foxy/images9/src_indoor.gif?height=250px)

작은 팁을 이야기하자면, **발생하는 모든 기록들을 문서화**하는 것을 추천드립니다. 하드웨어를 다루다보면, 방금까지 되던 기능이 갑자기 말썽을 일으켜서 하루를 날리는 날도 있습니다. 항상 모든 것을 기록합시다.

![lec9_7.png](/kr/ros2_basic_foxy/images9/lec9_7.png?height=300px)

- image from : [에누리닷컷](http://www.enuri.com/knowcom/detail.jsp?kbno=1551585)

> 강의의 마지막으로, ROS / ROS 2에 대한 프로젝트들, 센서들, 도전해볼 수 있는 것들을 제시해보고자 합니다.

**Tutorial & Courses**

- **awesome-ros2** : [https://github.com/fkromer/awesome-ros2](https://github.com/fkromer/awesome-ros2)
- **ROS2 Packages on NVIDIA Jetson** : [https://nvidia-ai-iot.github.io/ros2_jetson/ros2-packages/](https://nvidia-ai-iot.github.io/ros2_jetson/ros2-packages/)
- **Stereo labs** : [Getting Started with ROS2 and ZED](https://www.stereolabs.com/docs/ros2/)
- **Micro ROS Tutorial** : [https://micro.ros.org/docs/tutorials/core/overview/](https://micro.ros.org/docs/tutorials/core/overview/)
- **Self-Driving Cars with ROS 2 & Autoware** : [https://www.youtube.com/playlist?list=PLL57Sz4fhxLpCXgN0lvCF7aHAlRA5FoFr](https://www.youtube.com/playlist?list=PLL57Sz4fhxLpCXgN0lvCF7aHAlRA5FoFr)

**Certification**

- **Google Source of Code** : [https://summerofcode.withgoogle.com/](https://summerofcode.withgoogle.com/)
- **Jetson AI Ambassador** : [Jetson AI Courses and Certification - NVIDIA Developer](https://developer.nvidia.com/embedded/learn/jetson-ai-certification-programs)

**Open Source** **Contribution**

- **Nav2** : [https://navigation.ros.org/contribute/index.html](https://navigation.ros.org/contribute/index.html)
- **ROS 2 Control** : [https://control.ros.org/master/doc/project_ideas.html](https://control.ros.org/master/doc/project_ideas.html)
- **MoveIt** : [https://moveit.ros.org/documentation/contributing/](https://moveit.ros.org/documentation/contributing/)
- **Gazebo** : [https://gazebosim.org/docs/all/contributing](https://gazebosim.org/docs/all/contributing)

---

#### 지금까지 부족한 강의를 함께해주셔서 감사드리며, 재미있는 로봇 개발을 프로젝트를 시작하시거나, 기획하고 계신 분, 커피 한잔 하며 로봇 이야기를 나누고 싶으신 분, 언제나 환영합니다.

**김수영 / Kim Soo Young**

- **Tel : 010-8689-0259**
- **Email : [tge1375@hanyang.ac.kr](mailto:tge1375@hanyang.ac.kr) / [mr.swimmingkim@gmail.com](mailto:mr.swimmingkim@gmail.com)**

![rb_logo.png](/kr/ros2_basic_foxy/images9/rb_logo.png?height=200px)
