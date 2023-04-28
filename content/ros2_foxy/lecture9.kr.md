---
title: "Lecture9. ROS 2 Action Programming"
date: 2023-04-28T10:51:28+09:00
draft: false
---

## ROS 2 Action Programming

> 강의를 열심히 따라오면서 개념을 잊어버렸을 수 있으므로 Action에 대해서 간단히 개념 복습을 해보겠습니다.

![action.gif](/kr/ros2_foxy/images9/action.gif?height=300px)

- image from : [https://docs.ros.org/en/foxy/Tutorials/Understanding-ROS2-Actions.html](https://docs.ros.org/en/foxy/Tutorials/Understanding-ROS2-Actions.html)

사진과 같이 Action Client와 Server가 주고받는 내용은 크게 5가지가 있습니다.

1. Client ⇒ Server, Goal Request (service request와 유사합니다.)
2. server ⇒ client, Goal Response
3. client ⇒ server, Result Request
4. server ⇒ client, Feedback (topic과 유사합니다.)
5. server ⇒ client, Result Response

{{% notice tip %}}
만약 4번 도중 cancel이 발생하면 Action은 종료됩니다.
{{% /notice %}}

> 이렇게 Action은 Topic, Service의 특징을 모두 갖고 있으며 Cancel이라는 추가 기능까지 갖추고 있는 복잡한 통신 메커니즘이었습니다. 이번 예시에서는 ActionServer와 ActionClient, 그리고 과제로 별도 ActionServer까지 제시해보겠습니다.

### ROS 2 Action Programming - python

#### example 1 - Action Server (Parking Master)

![Untitled.png](/kr/ros2_foxy/images9/Untitled.png?height=400px)

image from : [기호일보](https://www.kihoilbo.co.kr/news/articleView.html?idxno=1006073)

> Action을 사용하는 재미있는 예시를 준비해 보았습니다. 아래 명령어를 입력해 주세요

```python
# Terminal 1
ros2 launch src_gazebo wall_world.launch.py

# Terminal 2
ros2 run py_action_tutorial parking_action_server

# Terminal 3
ros2 action send_goal /src_parking custom_interfaces/action/Parking "start_flag: true" --feedback
```

{{% notice note %}}
현재 custom interface를 사용하고 있으므로 workspace를 sourcing 해야 합니다!
{{% /notice %}}

- 예시 실행 시 벽에 인접하여 흰색 상자들이 주차 공간을 배정해줄 것입니다

⇒ rqt_robot_steering으로 로봇을 잘 제어하여 주어진 주차 공간에 알맞게 주차를 해보세요!

![Untitled1.png](/kr/ros2_foxy/images9/Untitled1.png?height=400px)

- 터미널 상의 Feedback을 통해 벽과의 거리를 확인할 수 있으며, 이 거리가 **0.5m 이내**가 되면 주차가 완료됩니다. 완료 시 좌우 공간이 얼마가 균형이 맞는지에 따라 다른 Result를 얻게 됩니다.

```cpp
Feedback:
    distance: 0.7399989366531372

Feedback:
    distance: 0.630066454410553

Result:
    message: '[Success!] Oh... Teach me how you did :0'
```

![src_parking.gif](/kr/ros2_foxy/images9/src_parking.gif?height=300px)

프로그래밍을 시작하기 전, 필요한 통신 메커니즘들을 살펴봅시다.

1. Action Server : Goal을 받으면, 정면 벽과의 거리를 feedback으로 전달합니다. 최종 Result는 String으로 성공 여부를 알려줍니다.
2. LaserScan Sub : 주변 물체와의 스캔된 거리를 알 수 있습니다.

   ⇒ Feedback Callback과 Subscription Callback 두 Callback 함수도 구현해야 할 것입니다.

- 이번 예시를 위해 custom interface를 만들어 보았습니다. (custom interface를 생성하는 방법에 대해 복습을 해볼 좋은 기회입니다. ⇒ 7강. ROS 2 Topic and Programming)

```xml
$ ros2 interface show custom_interfaces/action/Parking
#goal definition
bool start_flag
---
#result definition
string message
---
#feedback definition
float32 distance
```

### 코드 분석

- rclpy에서 Action Server를 사용하기 위해서는 ActionServer라는 별도의 패키지를 import 해야 합니다.

```python
import rclpy
from rclpy.node import Node
from rclpy.action import ActionServer
```

- 코드 분석 전 살펴본 바와 같이 두 종류의 핸들러가 필요합니다.

```python
class ParkingActionServer(Node):

    def __init__(self):
        super().__init__('parking_action_server')

        self.laser_sub = self.create_subscription(
            LaserScan, 'scan', self.sub_callback, 10
        )

        self.action_server = ActionServer(
            self, Parking, 'src_parking',
            goal_callback=self.goal_callback,
            cancel_callback=self.cancel_callback,
            execute_callback=self.execute_callback,
        )
```

- 이제 Action의 callback 지옥을 시작해보려 하는데요, 다시 한 번 개념을 다잡고 갑시다.

|                  | Description                                      |
| ---------------- | ------------------------------------------------ |
| goal_callback    | client로부터 첫 goal request 발생 시 실행되는 cb |
| cancel_callback  | feedback 도중 cancel request 발생 시 실행되는 cb |
| execute_callback | feedback 중 실행되는 cb                          |

1. goal_callback은 goal message를 매개변수로 가지며, **ACCEPT** 혹은 **Reject** 값을 리턴합니다.

```python
from rclpy.action import GoalResponse
...

def goal_callback(self, goal_request):
    self.get_logger().info('Received goal request.')
    self.goal = goal_request
    return GoalResponse.ACCEPT
```

2. **cancel_callback**은 goal_handle이라는 action handler object를 매개변수로 받습니다. 이 goal_handle 내부에는 여러 API들이 구현되어 있어 직접 확인해보고 넘어가고자 합니다.

```python
from rclpy.action import CancelResponse
...
def cancel_callback(self, goal_handle):
    self.is_sub = False
    self.is_done = True
		# print(dir(goal_handle))
    goal_handle.canceled()

    self.get_logger().info('Received cancel request')
    return CancelResponse.ACCEPT
```

3. **execute_callback**에서 주로 실제 로직이 구현되며, 업데이트되는 환경에 따라 client에게 feedback을 전달하고, 최종 result가 발동되는 조건도 구현합니다.

```python
def execute_callback(self, goal_handle):
    ...
    feedback_msg = Parking.Feedback()

    while self.f_obs_distance > 0.5 and self.is_done == False:
        goal_handle.publish_feedback(feedback_msg)
        ...

    ...

    return result
```

> 이번 예시의 로직을 간단하게 설명해보겠습니다. 로봇이 벽에 인접하게 되어 while loop를 벗어나면 **goal succed**를 수행하고, Result를 리턴합니다. 완료 시 좌우 물체와의 거리가 균일할 때 성공으로 판정짓습니다.

```python
    goal_handle.succeed()

    result = Parking.Result()
    lr_diff = abs(self.r_obs_distance - self.l_obs_distance)
    if lr_diff < 0.15:
        result.message = "[Success!] Oh... Teach me how you did :0"
    else:
        result.message = "[Fail] Be careful, Poor Driver! "

    self.is_done = True

    return result
```

- 이번 예시의 main문에서 특별한 점을 찾아볼 수 있습니다. **MultiThreadedExecutor**를 사용하고 있는데요. 이것이 하는 역할이 무엇일지 지금은 간단하게 실습을 통해 살펴봅시다.
  ⇒ main 문의 주석을 토글하고, 다시 예제를 실행 시켜봅니다. 어떠한 결과를 얻으셨나요?

```python
def main(args=None):

    rclpy.init(args=args)

    # parking_action_server = ParkingActionServer()
    # rclpy.spin(parking_action_server)
    # parking_action_server.destroy_node()
    # rclpy.shutdown()ㅁ
    ...
```

- 현 예시는 여러 종류의 callback을 갖고 있습니다. 문제는 **execute_callback**이 실행되면서 while loop로 진입하면, 자원을 점유하여 sub_callback이 동작할 수 없는 구조가 됩니다. 이를 해결하기 위해 ROS 2에서는 Executor라는 개념을 도입합니다. (자세한 내용은 이후 살펴보겠습니다.)

![Untitled2.png](/kr/ros2_foxy/images9/Untitled2.png?height=300px)

- image from : [docs.ros.org](https://docs.ros.org/en/foxy/Concepts/About-Executors.html)

> 이후 여러가지 구현을 하다 보면 지금처럼 다중 Subscribe를 해야하는 경우, 혹은 하나의 프로세스에서 여러개의 Node를 실행시켜야 하는 경우가 발생합니다. 이때, **Node Composition**과 **MultiThreadedExecutor**를 적극 사용해보세요!

### example 2 - Action Client (Parking Master)

Action 자체도 쉽지 않은 개념인데 MultiThreadedExecutor등 새로운 개념으로 많은 혼란이 있을 것 같습니다. 따라서 Action Client는 일전 Parking Example의 Client만 구현해보도록 하겠습니다.

- 이전 예시에서는 콘솔 창을 통해 직접 Action Client를 실행하였습니다.
  ⇒ ros2 action send_goal <action-name> <interface-name> <data field> <feedback option>

```python
ros2 action send_goal /src_parking custom_interfaces/action/Parking "start_flag: true" --feedback
```

- Action Client 예시는 바로 이 커멘드 라인을 코드로 호출하는 것입니다.

```python
ros2 run cpp_action_tutorial parking_status_client
```

### 코드 분석

- parking_status_client.py를 참고하시면 됩니다. 파일의 처음으로 ActionClient와 action data type을 import하고 있습니다.

```python
import rclpy
from rclpy.node import Node
from rclpy.action import ActionClient

from custom_interfaces.action import Parking
```

- 이번 예시에서도 Callback이 많아 정리부터 하고 넘어가겠습니다.

|                        | Description                        |
| ---------------------- | ---------------------------------- |
| feedback_callback      | Action Server로부터의 feedback cb  |
| goal_response_callback | Action Server로부터의 Response cb  |
| get_result_callback    | Action Server로부터의 Result cb    |
| cancel_goal            | Action Server로 전달하는 Cancel cb |
| goal_canceled_callback | Action Server로부터의 Cancel cb    |

- 생성자에서는 Action Client 생성 외에 큰 작업은 없습니다.

```python
class ParkingActionClient(Node):

    def __init__(self):
        super().__init__('parking_action_client')
        self._action_client = ActionClient(self, Parking, 'src_parking')
```

|               | Description      |
| ------------- | ---------------- |
| Parking       | Action data type |
| 'src_parking’ | Action name      |

- send_goal()은 main에서 직접 호출되는 함수로, feedback_callback과 goal_response_callback을 바인딩합니다.

```python
def send_goal(self):
    ...
    self._send_goal_future = self._action_client.send_goal_async(goal_msg, feedback_callback=self.feedback_callback)
    self._send_goal_future.add_done_callback(self.goal_response_callback)

def main(args=None):
    ...
    action_client.send_goal()
```

- goal_response_callback 완료 시, get_result_callback이 실행되며, result를 받았다는 것은 action이 완료되었다는 뜻이므로 종료 로직이 포함됩니다.

```python
def goal_response_callback(self, future):
    self.goal_handle = future.result()
    if not self.goal_handle.accepted:
        self.get_logger().info('Goal rejected :(')
        return

    self.get_logger().info('Goal accepted :)')

    self._get_result_future = self.goal_handle.get_result_async()
    self._get_result_future.add_done_callback(self.get_result_callback)

def get_result_callback(self, future):
    result = future.result().result
    self.get_logger().info(f'Result: {result.message}')
    rclpy.shutdown()
```

- cancel 로직은 feedback_callback안에 존재합니다. 피드백을 통해 받은 전방 물체와의 거리가 너무 멀어져버리면 오류로 인지하고 cancel이 이루어지는 것입니다.

```python
def feedback_callback(self, feedback_msg):
    ...
    if feedback.distance > 6.0:
        self.cancel_goal()

def cancel_goal(self):
    self.get_logger().info('Canceling goal')
    future = self.goal_handle.cancel_goal_async()
    future.add_done_callback(self.goal_canceled_callback)

def goal_canceled_callback(self, future):
    cancel_response = future.result()

    if len(cancel_response.goals_canceling) > 0:
        self.get_logger().info('Cancelling of goal complete')
    else:
        self.get_logger().warning('Goal failed to cancel')
```

⇒ 따라서 예시 실행 시 의도적으로 뒤로 가버리면 Action이 종료됩니다.

- main문에서 send_goal을 호출하는 것을 잊지 마세요.

```python
def main(args=None):
    rclpy.init(args=args)

    action_client = ParkingActionClient()
    action_client.send_goal()
    rclpy.spin(action_client)
```

### example 3 - Action Server (Dataset Generation)

> 이제 Topic, Service, Action을 모두 배워보았습니다. 때문에 이를 모두 사용하는 과제를 내드리고자 합니다.

- Dataset Generation

```python
# Terminal1 - gazebo env
ros2 launch rgbd_world depth_world.launch.py

# Terminal2 - Assignment Node
ros2 run py_action_tutorial dataset_generation_server

# Terminal3 - Action Call
ros2 action send_goal /dataset_generation custom_interfaces/action/DataGen "{x: 1, z: 1, model_name: beer}"
```

- 선택한 물체와 위치에 따라 총 6번 spawn, capture, deletion이 반복됩니다.

![action_datasetgen.gif](/kr/ros2_foxy/images9/action_datasetgen.gif?height=300px)

⇒ 모든 작업이 완료된 후 workspace에는 다음과 같이 사진 dataset이 생성됩니다.

![Untitled3.png](/kr/ros2_foxy/images9/Untitled3.png?height=300px)

### 💪 이 Node를 개발해보는 것이 이번 시간의 과제입니다.

- **Hint 1. 필요한 통신 메커니즘**

|                     | Description                                                                                                     |
| ------------------- | --------------------------------------------------------------------------------------------------------------- |
| create_subscription | image topic을 받아 저장해야 합니다.                                                                             |
| create_client       | 일전 Service Client 때처럼 물성치를 정지시키고, 원하는 위치에 물체를 등장시키고, 또 삭제하는 작업이 필요합니다. |
| ActionServer        | Action을 통해 잘 진행되고 있는지 로직을 구현하고, Service들도 핸들링합니다.                                     |

- **Hint 2. action data type**

이번 예시를 위해 사용할 데이터 타입을 미리 준비해두었습니다. 하지만, 직접 여러분만의 interface를 만들어보고, 사용해보기를 추천드립니다.

```python
#goal definition
string model_name
float32 x
float32 y
float32 z
---
#result definition
string message
---
#feedback definition
string message
```

- **Hint 3. sleep**

사진이 찍히기까지 시간 여유를 주기 위해 create_rate 함수 사용을 권장합니다. (단위는 Hz)

```python
self.rate = self.create_rate(0.3)
self.rate.sleep()
```

⇒ 해설은 다음 시간에 진행하도록 하겠습니다!!

## ROS 2 Action Programming - C++

### example 1 - Action Server (Parking Master)

- 바로 예시 실행은 다음과 같습니다.

```python
# Terminal 1
ros2 launch src_gazebo wall_world.launch.py

# Terminal 2
ros2 run cpp_action_tutorial parking_status_server

# Terminal 3
ros2 action send_goal /src_parking custom_interfaces/action/Parking "start_flag: true" --feedback
```

### 코드 분석

- 프로그래밍 중 타이핑을 최소화하기위해 ServerGoalHandle에 대한 키워드를 재정의하였습니다.

```python
#include "rclcpp/rclcpp.hpp"

#include "sensor_msgs/msg/laser_scan.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "custom_interfaces/action/parking.hpp"

using LaserScan = sensor_msgs::msg::LaserScan;
using Parking = custom_interfaces::action::Parking;
using GoalHandleParking = rclcpp_action::ServerGoalHandle<Parking>;
```

- 파이썬 때와 동일하게 Action Server와 Topic Subscriber가 필요합니다.

```cpp
public:
  ParkingActionServer() : Node("parking_action_server") {
    using namespace std::placeholders;
    m_action_server = rclcpp_action::create_server<Parking>(
      this, "src_parking",
      std::bind(&ParkingActionServer::handle_goal, this, _1, _2),
      std::bind(&ParkingActionServer::handle_cancel, this, _1),
      std::bind(&ParkingActionServer::handle_accepted, this, _1)
    );

    m_laser_sub = this->create_subscription<LaserScan>(
      "scan", 10, std::bind(&ParkingActionServer::laser_callback, this, _1)
    );

    RCLCPP_INFO(get_logger(), "Action Ready...");
  }
```

- rclcpp Action Server의 callback들은 다음과 같습니다. 파이썬과 달리 accept에 대한 callback을 갖습니다.

|                   | Description                                      |
| ----------------- | ------------------------------------------------ |
| handle_goal       | client로부터 첫 goal request 발생 시 실행되는 cb |
| handle_cancel     | feedback 도중 cancel request 발생 시 실행되는 cb |
| handle_accepted   | goal accept 시 실행되는 cb                       |
| template argument | action data type                                 |
| "src_parking"     | action name                                      |

1. **handle_goal**은 goal id, goal 자체를 매개변수로 가지며, REJECT, ACCEPT_AND_EXECUTE, ACCEPT_AND_DEFER을 리턴으로 가질 수 있습니다.

```cpp
rclcpp_action::GoalResponse handle_goal(const rclcpp_action::GoalUUID &uuid,
            std::shared_ptr<const Parking::Goal> goal) {
  RCLCPP_INFO(get_logger(), "Got goal request with order %s", goal->start_flag ? "true" : "false");
  (void)uuid;
  return rclcpp_action::GoalResponse::ACCEPT_AND_EXECUTE;
}
```

2. **handle_cancel**은 action goal handler object를 매개변수로 받습니다. 파이썬과 동일하게 이 goal_handle 내부에는 여러 API들이 구현되어 있습니다.

⇒ CancelResponse는 ACCEPT, REJECT 두종류가 가능합니다.

```cpp
rclcpp_action::CancelResponse handle_cancel(const std::shared_ptr<GoalHandleParking> goal_handle) {
  RCLCPP_WARN(get_logger(), "Got request to cancel goal");
  (void)goal_handle;
  return rclcpp_action::CancelResponse::ACCEPT;
}
```

3. **handle_accepted**에서 feedback을 구현하는 execute로 연결이 됩니다. 이 부분에서 파이썬과 차이를 갖는데, 복습을 해볼 겸 비교를 해보겠습니다.

```cpp
void handle_accepted(const std::shared_ptr<GoalHandleParking> goal_handle) {
  // this needs to return quickly to avoid blocking the executor, so spin up a
  // new thread
  using namespace std::placeholders;
  std::thread{std::bind(&ParkingActionServer::execute, this, _1), goal_handle}.detach();
}
```

```python
class ParkingActionServer(Node):

  def __init__(self):
    super().__init__('parking_action_server')

    self.laser_sub = self.create_subscription(
      LaserScan, 'scan', self.sub_callback, 10
    )

    self.action_server = ActionServer(
      self, Parking, 'src_parking',
      goal_callback=self.goal_callback,
      cancel_callback=self.cancel_callback,
      execute_callback=self.execute_callback,
    )
```

- execute의 로직은 살펴볼 바 있어 간단히 넘어가겠습니다. 다만, cancel 로직이 삽입되는 부분과 구현에 집중하시기 바랍니다.

```cpp
while (f_obs_distance > 0.5 && is_done == false) {
  if (goal_handle->is_canceling()) {
    result->message = "Canceled";
    goal_handle->canceled(result);
    is_done = true;
    RCLCPP_WARN(get_logger(), "Goal Canceled");
    return;
  }

  feedback->distance = f_obs_distance;
  goal_handle->publish_feedback(feedback);
  RCLCPP_INFO(get_logger(), "Distance from forward obstacle %f", f_obs_distance);

  loop_rate.sleep();
}
```

- rclcpp에서 **MultiThreadedExecutor**를 사용하는 방법은 아래와 같습니다. add_node에 여러 node를 추가할 수 있는 것이지요.

```python
int main(int argc, char * argv[]){
  rclcpp::init(argc, argv);

  rclcpp::executors::MultiThreadedExecutor executor;
  auto server_node = std::make_shared<ParkingActionServer>();

  executor.add_node(server_node);
  executor.spin();

  rclcpp::shutdown();
  return 0;
}
```

### example 2 - Action Client (Parking Master)

> cancel을 포함한 rclcpp Action Client 작성법을 알아봅시다.

```python
ros2 run cpp_action_tutorial parking_status_client
```

### 코드 분석

> Action은 자체가 쉽지 않은 개념이어 코딩 시 개념이 잡혀있지 않으면 무척 힘들어집니다. 코드 분석 전, 개요를 먼저 살펴보고 넘어갑시다.

- Action Client Logic

1. 생성자에서 Action Client 생성
2. main에서 send_goal 실행
3. send_goal에서 여러 callback이 정의되어 있으며 상황에 따라 적절한 callback이 실행됨
   1. goal_response_callback
   2. feedback_callback
   3. result_callback

- action server 때와 마찬가지로 rclcpp_action의 include와 편의를 위해 ClientGoalHandle 키워드를 다시 정의하고 있습니다.

```cpp
#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "custom_interfaces/action/parking.hpp"

using namespace std::placeholders;
using Parking = custom_interfaces::action::Parking;
using GoalHandleParking = rclcpp_action::ClientGoalHandle<Parking>;
```

- rclcpp action client는 다음과 같이 생성합니다. (실제 callback들은 send_goal 시 Handler에 의해 바인딩됩니다.)

```cpp
class ParkingActionClient : public rclcpp::Node {
private:
  rclcpp_action::Client<Parking>::SharedPtr m_action_client;

public:
  ParkingActionClient() : Node("parking_action_client"){
    m_action_client = rclcpp_action::create_client<Parking>(this, "src_parking");
  }
```

|               | Description      |
| ------------- | ---------------- |
| Parking       | Action data type |
| 'src_parking’ | Action name      |

- 다시금 이번 예시에서의 callback과 주요 함수를 정리해보았습니다.

|                        | Description                       |
| ---------------------- | --------------------------------- |
| feedback_callback      | Action Server로부터의 feedback cb |
| goal_response_callback | Action Server로부터의 Response cb |
| result_callback        | Action Server로부터의 Result cb   |
| async_cancel_all_goals | Action Server로 Cancel call       |

- **goal_response_callback**은 GoalHandler shared_pointer의 shared_future를 매개변수로 갖습니다. 다소 복잡한데, GoalHandler 자체가 아닌 future이기 때문에 get() 함수를 통해 값을 따로 받아 처리해야 합니다.

```cpp
void goal_response_callback(std::shared_future<GoalHandleParking::SharedPtr> future){
  auto goal_handle = future.get();
  if (!goal_handle) {
    RCLCPP_ERROR(this->get_logger(), "Goal was rejected by server");
  } else {
    RCLCPP_INFO(this->get_logger(), "Goal accepted by server, waiting for result");
  }
}
```

- **feedback_callback** 내부 cancel 로직에서 이전과 새로운 점을 볼 수 있습니다. 구현의 편의를 위해 async_cancel_all_goals을 사 용하였는데요, 현재 사용중인 action이 하나뿐이라 문제가 없지만 다수의 action이 존재하는 경우 async_cancel_goal 사용을 권장합니다.

```cpp
void feedback_callback(GoalHandleParking::SharedPtr,
  const std::shared_ptr<const Parking::Feedback> feedback){
  RCLCPP_INFO(this->get_logger(), "Received feedback: %f", feedback->distance);

  if(feedback->distance > 6.0)
    m_action_client->async_cancel_all_goals();
}
```

|                                 | Description                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------ |
| async_cancel_goal(goal_handler) | async_cancel_goal (typename GoalHandle::SharedPtr goal_handle, CancelCallback cancel_callback=nullptr) |
| async_cancel_all_goals()        | async_cancel_all_goals (CancelCallback cancel_callback=nullptr)                                        |

- **result_callback**은 WrappedResult라는 1개의 매개변수를 갖습니다. goal result를 다루기 위한 별도의 객체이며, ResultCode라는 enum type을 갖고 있습니다.

```cpp
void result_callback(const GoalHandleParking::WrappedResult & result){
  switch (result.code) {
    case rclcpp_action::ResultCode::SUCCEEDED:
      break;
    case rclcpp_action::ResultCode::ABORTED:
      RCLCPP_ERROR(this->get_logger(), "Goal was aborted");
      return;
    case rclcpp_action::ResultCode::CANCELED:
      RCLCPP_ERROR(this->get_logger(), "Goal was canceled");
      return;
    default:
      RCLCPP_ERROR(this->get_logger(), "Unknown result code");
      return;
  }

  RCLCPP_INFO(this->get_logger(), "Result received: %s", result.result->message.c_str());
  rclcpp::shutdown();
}
```

| Enumerator |
| ---------- |
| UNKNOWN    |
| SUCCEEDED  |
| CANCELED   |
| ABORTED    |

{{% notice note %}}
이번 예시에서도 main문에서 send_goal을 호출하는 것을 잊지 마세요.
{{% /notice %}}
