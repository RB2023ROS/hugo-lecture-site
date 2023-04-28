---
title: "Lecture1. What is ROS?"
date: 2023-04-28T10:51:29+09:00
draft: false
---

## What is ROS?

> 강의를 제작하고 있는 현 시점, 세상은 chatgpt로 인해 큰 격변을 맞이하고 있는데요, chatgpt에게 ROS가 무엇인지 한 번 물어보았습니다. 😊

![Untitled.png](/kr/ros2_foxy/images1/Untitled.png?height=400px)

{{% notice tip %}}
💡 앞으로 프로그래밍과 실습 시 chatgpt를 적극 활용하여 강의를 이어나가도록 하겠습니다. 😉
{{% /notice %}}

#### chatgpt가 잘 정리해주었지만, 제가 조금 더 설명을 이어나가보도록 하겠습니다.

- ROS란, 로봇 소프트웨어 개발에 사용되는 일종의 프레임워크입니다.
- 프레임워크라는 말은, ROS 나름대로의 실행 시나리오를 갖고 있다는 뜻입니다.
- 사용자인 우리들은, 이 시나리오를 사용하여 로봇을 다루는 우리만의 Application을 만들게 됩니다.

![Untitled1.png](/kr/ros2_foxy/images1/Untitled1.png?height=150px)

> image from : [wikimedia](https://commons.wikimedia.org/wiki/File:Ros_logo.svg)

### 그런데 왜 OS라는 이름이 붙게 되었을까요?

> 로봇을 실행하기 위해서, 수많은 프로그램들이 실행되며, ROS는 이들 사이의 우선순위와, 프로그램 사이의 데이터 흐름을 책임집니다. 이 작업은 **스케쥴링**이라고 불리며, 이러한 동작을 수행하는 시스템을 Operating System이라고 부르기 때문에 ROS라는 이름을 갖게 되었습니다.

![Untitled2.png](/kr/ros2_foxy/images1/Untitled2.png?height=300px)

- image from : [tutorialspoint](https://www.tutorialspoint.com/operating_system/os_process_scheduling.htm)

**그럼, 로봇을 개발하기 위해서 어떠한 프로그램들이 필요할까요?**
=> 로봇이 수행하는 임무들을 크게 3가지로 분류하면 인지, 판단, 제어의 3가지로 나뉩니다.

![Untitled3.png](/kr/ros2_foxy/images1/Untitled3.png?height=300px)

- **인지**란, 센서들을 통한 물체 인지, 자기 자신의 위치와 방향 인지, 상황 인지 등 로봇에게 있어 환경과 상호작용하는 과정에 해당합니다.
- **판단**이란, 앞/뒤로 움직일지, 로봇 팔을 뻗을지와 같이 인지를 기반으로 얻은 데이터를 통해 결정을 내리는 작업들이 해당할 것입니다.
- **제어** 또한 빼놓을 수 없는 영역으로, 로봇은 역학이라는 법칙이 존재하는 실제 세상에서 움직이기 때문에, 이 시스템을 분석하여 적절한 움직임을 취하도록 해야합니다.

> 이렇게 로봇 시스템은 무척 복잡하며, 여러 작업을 수행하는 프로그램들 사이 수많은 데이터가 오가고, 데이터의 신뢰성과 속도 모두 충축시켜야 하며, 또 사람과 상호 작용해야 합니다.

![Untitled4.png](/kr/ros2_foxy/images1/Untitled4.png?height=300px)

- image from : [ROS Industrial](https://rosindustrial.org/developmentprocess)

### ROS는 이러한 작업을 손쉽게 해주는 **시스템**입니다.

> 예를 들면, 로봇의 센서, 구동부, 알고리즘이 모두 준비되어 있는 상황에서, 이들을 하나의 시스템으로 엮어주는 역할을 하는 것이 바로 ROS입니다.

![Untitled5.png](/kr/ros2_foxy/images1/Untitled5.png?height=300px)

- image from : [ros wiki](http://wiki.ros.org/rqt_graph)

#### 그렇다면, ROS와 ROS 2는 어떤 점에서 다를까요?

- ROS의 시작은 연구실이었습니다. 그리고 PR2라는 단일 로봇을 위해 만들어진 시스템이었기에 연구용, 간단한 프로젝트용으로는 사용될 수 있지만, 상용화를 위한 여러 조건들은 고려하지 못하고 있었습니다.

![Untitled6.png](/kr/ros2_foxy/images1/Untitled6.png?height=300px)

image from : [https://robots.ieee.org/robots/pr2/](https://robots.ieee.org/robots/pr2/)

- **보안**
- **안정성**
- **실시간성**
- **개발 용이성**
- **기술 지원**
- **하드웨어 연동**
- **etc…**

> 연구용으로 설계된 기존 ROS를 통해서는 이러한 모든 조건을 충족할 수 없다는 결론을 내렸고, Open Robotics는 **ROS 2**를 새롭게 선보이게 됩니다.

![Untitled7.png](/kr/ros2_foxy/images1/Untitled7.png?height=500px)

- image from : [maker.pro](https://maker.pro/ros/tutorial/robot-operating-system-2-ros-2-introduction-and-getting-started)

{{% notice note %}}
💡 기존 언급되었던 ROS 1의 모든 문제들이 ROS 2에서 해결되었다고 할 수는 없지만 적어도 해결을 고려하여 설계되었다고 말하고 싶습니다.
{{% /notice %}}

- 상용화를 고려한 가장 큰 변화 중 하나는 패킷 통신에 TCPROS/UPDROS가 아닌 DDS를 도입하였다는 것입니다.

![Untitled8.png](/kr/ros2_foxy/images1/Untitled8.png?height=300px)

- image from : [omg.org](https://www.omg.org/dds-directory/)

> DDS는 Data Distribution Service의 약자이며, 다양한 프로그램이 동시에 동작하는 시스템을 위한 일종의 약속입니다. 기존의 ROS가 지양하던 방식이 DDS와 일치하는 부분이 많았고, 산업 표준에 널리 사용되고 있었기 때문에 채택하지 않았나 싶습니다. (자세한 내용은 강의를 통해 함께 살펴보겠습니다.)

---

# About this lecture

> 앞서 이야기한 바와 같이 ROS는 특정 알고리즘을 위한 라이브러리가 아닌 **시스템**입니다.

따라서, 이 시스템을 이해하고, 익숙해지고, 실습하고, 코딩하고, 기존의 코드를 연동하는 작업들을 통해 여러분들의 코드와 알고리즘을 ROS로 감쌀 수 있게 됩니다.

- ROS Wrapper Google 검색 결과
  ![Untitled9.png](/kr/ros2_foxy/images1/Untitled9.png?height=300px)

이번 강의를 기획하는 시점에서 세상에 큰 변화가 있었습니다. 바로 ChatGPT인데요. C++ 코드를 Python 코드로 바꾼다거나, 내가 원하는 ROS 2 코드를 말만 하면 순식간에 짜주는 세상이 되었습니다.

- ChatGPT를 통해 생성한 코드 예시

![Untitled10.png](/kr/ros2_foxy/images1/Untitled10.png?height=400px)

뿐만 아니라 코드 작성 시에도 Github Copilot, Code GPT 등 자동 완성 기능들이 등장하여 개발자들의 편의를 높여주고 있습니다.

- copliot을 통해 생성한 rclcpp 예시

```cpp
#include "rclcpp/rclcpp.hpp"
// rclcpp node example
int main(int argc, char *argv[]) {
  rclcpp::init(argc, argv);
  auto node = rclcpp::Node::make_shared("test_node");
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}
```

> 이러한 이유로 개발자들은 더이상 코드 API를 외우고, 예제를 뒤적이고 할 필요가 없어졌다고 생각합니다. 이는 ROS 2 개발시에도 물론 적용될 것입니다.

따라서, 이번 강의에서는 AI가 알려줄 수 없는 것들을 위주로 진행하고자 합니다.

- ROS 2를 어떻게 손쉽게 설치하고,
- 어떠한 명령으로 다룰 수 있는지,
- 또 어떠한 개념을 갖고 있으며,
- 내가 원하는 기능을 위해 어떤 코드 구현이 필요한지
- 이 강의에서 모두 다루어 보겠습니다. 😉

---

### 강의를 수강하기 위해 필요한 선수지식

> 본 강의는 최대한 많은 분들이 끝까지 이해할 수 있도록 설계되었습니다. 따라서, 최대한 쉽고, 프로그래밍 실력이 출중하지 않아도 모두 완강할 수 있게 진행합니다. 프로그래밍 지식은 아래 코드를 이해할 수 있을 정도면 괜찮습니다.

- python basic

```python
class OOPNode:

    def __init__(self):
        self.counter_ = 0

    def hello_du(self, event=None):
        hello_du = f"hello du {rospy.get_time()}, counter: {self.counter_}"
        print(hello_du)
        self.counter_ += 1

def my_first_oop_node():
    oop_node = OOPNode()

if __name__ == '__main__':
    try:
        my_first_oop_node()
    except Exception as e:
        print(e)
```

C++의 경우 이야기가 조금 다른데요. C++11 이후, 통칭 Modern C++에 대한 지식이 필요합니다.

- type deduction
- smart pointer
- lambda, bind
- etc…

{{% notice note %}}
더불어 빌드 시스템에 대한 이해 (make, CMake)도 어느 정도 필요합니다.
{{% /notice %}}

> 파이썬으로 개념을 익히고, C++ 언어 공부는 언제든 추가로 하면 되니 강의에서 이해에 어려움이 있다면 추가로 리서치하면 되시겠습니다.

---

### 강의 코드와 강의 노트 사용법

> 강의 도중 사용되는 코드들은 Github Repository를 통해 배포되어 있습니다.

- [https://github.com/RB2023ROS/du2023-ros2](https://github.com/RB2023ROS/du2023-ros2)

![Untitled11.png](/kr/ros2_foxy/images1/Untitled11.png?height=400px)

> 강의 노트의 주소는 [https://rb2023ros.github.io/kr/](https://rb2023ros.github.io/kr/) 입니다. 코드와 명령어 등 필요한 리소스를 모두 담고 있으므로 복사/붙여넣기를 활용하여 강의 청취 시간을 절약하시기 바랍니다.

{{% notice tip %}}
해당 노트는 **hugo**를 사용하여 제작된 웹 페이지임을 밝힙니다.
{{% /notice %}}
