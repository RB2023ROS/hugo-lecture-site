---
title: "Lecture4 - ROS 2 Node C++ Programming"
date: 2023-01-02T12:39:10+09:00
draft: false
---

> 실제 로봇 개발에서 Python만 사용되지는 않습니다. tensorflow, pytorch와 같은 딥러닝 프레임워크들이 모두 Python을 사용하고 있고, 상대적으로 개발 과정이 쉬워 많은 사람들이 Python을 사용하고 있지만, 제어, 센싱을 위해서는 C++를 대부분 사용한다고 보시면 됩니다.

![lec4_0.png](/kr/ros2_basic_foxy/images4/lec4_0.png?height=300px)

- image from : [wikipedia](https://ko.m.wikipedia.org/wiki/%ED%8C%8C%EC%9D%BC:ISO_C%2B%2B_Logo.svg)

따라서 앞선 node 프로그래밍 예시들을 모두 C++로 구현해 볼 예정입니다. 아래와 같은 C++ 지식이 필요하지만 필수는 아닙니다.

- **클래스 생성 (OOP)**
- **스마트 포인터**
- **헤더 분리**
- **람다 함수**
- **std::bind와 placeholders**
- **C++ 빌드 시스템 (CMake)**

코드 설명에 앞서 ROS 2 C++ 패키지를 개발하는 절차에 대해 빠르게 훑어보겠습니다.

- 패키지 생성 후 src & include 폴더에서 코드 작성

```python
ros2 pkg create --build-type ament_cmake <package_name>
```

- CMakeLists.txt & package.xml 수정

```python
add_executable(example_node_1 src/node_example_1.cpp)
ament_target_dependencies(example_node_1 rclcpp)

install(
  TARGETS
    example_node_1
  DESTINATION
    lib/${PROJECT_NAME}
)
```

- 코드 빌드와 디버깅

```bash
colcon build --packages-select <package_name>
source install/local_setup.bash
```

- 코드 실행

```bash
ros2 run <pkg-name> <executable-name>
```

개발 과정 자체는 파이썬 package와 크게 다르지 않습니다.

하지만, C++ 개발은 코드를 빌드하는 과정에서 각종 **컴파일 에러와 링크 에러, 런타임 에러**까지 발생하기 때문에 개발에 여러움이 생길 수 있습니다.

⇒ 이를 해결하기 위해서, 학생 라이센스로 무료 사용 가능한 IDE를 소개하고, 함께 셋업해보고자 합니다.

### CLion 설치와 ROS 2 개발환경 설정

> CLion은 IDE의 명가 Jetbrains에서 만든 C/C++용 통합 개발 환경입니다. C++ 개발자를 힘들게 하는 각종 에러들의 디버깅을 편리하게 해줄 뿐더러 ROS 2 개발을 위한 솔루션도 제공하고 있습니다.

![lec4_1.png](/kr/ros2_basic_foxy/images4/lec4_1.png?height=300px)

- image from : [wikipedia](https://ru.wikipedia.org/wiki/CLion)

CLion은 오픈소스가 아닌 판매되고 있는 소프트웨어입니다.

하지만 학생에게는 무료 라이센스를 제공하고 있습니다. 아래 링크를 통해서 회원가입과 학생인증을 진행한 다음, CLion을 설치합시다.

- [https://www.jetbrains.com/clion/](https://www.jetbrains.com/clion/)

- [https://www.jetbrains.com/community/education/#students](https://www.jetbrains.com/community/education/#students)

{{% notice note %}}
이메일 인증을 거치면 어렵지 않게 학생인증이 완료됩니다. 이 과정은 생략하겠습니다.
{{% /notice %}}

![lec4_2.png](/kr/ros2_basic_foxy/images4/lec4_2.png?height=300px)

- 학생 라이센스를 통해 CLion을 비롯하여 [IntelliJ](https://www.jetbrains.com/ko-kr/idea/download/), [PyCharm](https://www.jetbrains.com/ko-kr/pycharm/), [WebStorm](https://www.jetbrains.com/ko-kr/webstorm/)과 같은 다양한 언어의 IDE를 무료로 사용 가능합니다.

![lec4_3.png](/kr/ros2_basic_foxy/images4/lec4_3.png?height=300px)

- snapcraft를 통해 CLion을 설치하고 실행 해봅시다.

```bash
$ sudo snap install clion --classic
[sudo] password for kimsooyoung:
clion 2022.3.1 from jetbrains✓ installed

$ clion
```

- 약관 동의 및 로그인을 통한 학생 인증을 거치면 모든 설치 절차가 완료됩니다.

![lec4_4.png](/kr/ros2_basic_foxy/images4/lec4_4.png?height=300px)

- colcon build를 통해 빌드했던 **cpp_node_tutorial** **package**를 clion 프로젝트로 열어보겠습니다. clion에서 **File ⇒ Open**을 실행한 뒤 cpp_node_tutorial의 **CMakeLists.txt**를 선택합니다.

![lec4_5.png](/kr/ros2_basic_foxy/images4/lec4_5.png?height=300px)

- build target를 지정하고 실행하면 CLion상에서 ROS 2 프로그램을 실행시킬 수 있습니다.

![clion.gif](/kr/ros2_basic_foxy/images4/clion.gif?height=300px)

- CLion에서의 장점을 직접 살펴보기 위해 프로그램 개발을 함께 해보겠습니다.

{{% notice note %}}

**라이브 코딩 Time!!**

{{% /notice %}}

모든 설정이 완료되었다면, 이제 본격적으로 C++ 예제 코드를 살펴봅시다.

### example 1 - **Hello ROS 2**

> 파이썬에서 한차례 살펴본 바 있기에 자세한 로직들은 생략합니다.

- 예제 실행

```bash
$ ros2 run cpp_node_tutorial example_node_1
[INFO] [1666431282.586519100] [example_node_1]: ==== Hello ROS 2 ====
```

- 코드 분석

```cpp
#include <rclcpp/rclcpp.hpp>
int main(int argc, char **argv) {
  rclcpp::init(argc, argv);

  auto node = rclcpp::Node::make_shared("example_node_1");
  RCLCPP_INFO(node->get_logger(), "==== Hello ROS 2 ====");

  rclcpp::shutdown();
  return 0;
}
```

**rclcpp**은 ROS 2를 C++에서 사용하기 위해 필요한 코드의 집합입니다.

- Node 생성 시 주의하셔야 할 점은 “모든 Node가 **포인터**의 형식을 갖는다”는 것입니다. **shared_ptr**를 사용하여 메모리 누수를 막기 위해 파생된 모든 데이터들을 추적하고 있습니다.
- rclcpp에서 로그의 실행은 **RCLCPP_INFO와 get_logger()** 메소드를 통해 실행할 수 있으며, 로그는 Node에서 실행된다는 점을 기억합시다.

- CMakeLists.txt 수정

```python
# find_package를 통해 종속성들을 추가합니다.
find_package(<depends> REQUIRED)

# 실행 프로그램 빌드 설정
add_executable(<program_name> include/<header>.hpp src/<code>.cpp ...)
ament_target_dependencies(<program_name> <dependency1> <dependency2> ...)

install(
  TARGETS
    <program_name>
  DESTINATION
    lib/${PROJECT_NAME}
)
```

- 패키지 빌드 & 실행

```bash
colcon build --packages-select cpp_node_tutorial
source install/local_setup.bash

ros2 run cpp_node_tutorial example_node_1
```

{{% notice note %}}
예제 소스 코드를 조금이라도 수정하여 여러분들만의 코드를 작성해보고 CLion을 통한 빌드까지 스스로 한 번 해봅시다.
{{% /notice %}}

### example 2 - Timer

- 예제 실행

```bash
$ ros2 run cpp_node_tutorial example_node_2
==== Hello ROS 2 : 0 ====
==== Hello ROS 2 : 1 ====
==== Hello ROS 2 : 2 ====
==== Hello ROS 2 : 3 ====
==== Hello ROS 2 : 4 ====
...
```

- rclcpp에서 timer는 `create_wall_timer`를 사용합니다.

```cpp
#include <iostream>
#include <memory>
#include "rclcpp/rclcpp.hpp"

static int count = 0;

void timer_callback(){
  std::cout << "==== Hello ROS 2 : " << count << " ====" << std::endl;
  count++;
}

int main(int argc, char **argv) {
  rclcpp::init(argc, argv);
  auto node = rclcpp::Node::make_shared("example_node_2");
  auto timer = node->create_wall_timer(std::chrono::milliseconds(200), timer_callback);

  rclcpp::spin(node);

  rclcpp::shutdown();
  return 0;
}
```

### example 3 - OOP Node

- 기능 자체는 큰 의미가 없으므로 코드를 위주로 설명하겠습니다.

```cpp
#include <memory>
#include "rclcpp/rclcpp.hpp"

class NodeClass: public rclcpp::Node {
public:
  NodeClass(): Node("example_node_4") {}
};

int main(int argc, char **argv) {
  rclcpp::init(argc, argv);

  auto node = std::make_shared<NodeClass>();
  RCLCPP_INFO(node->get_logger(), "==== Hello ROS 2 ====");

  rclcpp::shutdown();
  return 0;
}
```

모든 Node는 **rclcpp::Node**로부터 상속을 받습니다. 상속 후 생성자에서 Node의 이름을 지정해줘야 하며, topic pub/sub, logger와 같은 기능들은 모두 rclcpp::Node에 구현되어 있습니다.

더불어, Node 자체는 포인터로 취급된다는 점도 다시 한 번 강조드립니다.

### example 4 - OOP Timer Node

- callback 함수를 binding하는 부분에 집중하세요. timer_callback는 클래스 메소드이기 때문에 binding 시 **NodeClass::timer_callback**와 같이 명시해주어야 합니다.

```cpp
#include <memory>
#include "rclcpp/rclcpp.hpp"

class NodeClass: public rclcpp::Node {
private:
  size_t count;
  rclcpp::TimerBase::SharedPtr timer;

  void timer_callback() {
    RCLCPP_INFO(this->get_logger(), "==== Hello ROS 2 : %d", count);
    count++;
  }

public:
  NodeClass() : Node("example_node_5") {
    timer = this->create_wall_timer(
      std::chrono::milliseconds(200),
      std::bind(&NodeClass::timer_callback, this)
    );
  }
};

int main(int argc, char **argv) {
  rclcpp::init(argc, argv);

  auto node = std::make_shared<NodeClass>();
  rclcpp::spin(node);

  rclcpp::shutdown();
  return 0;
}
```

- 마지막으로 **rclcpp logger level**을 간단히 살펴보고 마무리짓겠습니다.

```cpp
void timer_callback() {
  RCLCPP_DEBUG(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
  RCLCPP_INFO(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
  RCLCPP_WARN(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
  RCLCPP_ERROR(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
  RCLCPP_FATAL(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
  count++;
}
```

---

**참고자료**

- [https://roboticsbackend.com/ros2-yaml-params/](https://roboticsbackend.com/ros2-yaml-params/)
- [https://roboticsbackend.com/rclpy-params-tutorial-get-set-ros2-params-with-python/](https://roboticsbackend.com/rclpy-params-tutorial-get-set-ros2-params-with-python/)
- [https://www.jetbrains.com/help/clion/ros2-tutorial.html#change-prj-root](https://www.jetbrains.com/help/clion/ros2-tutorial.html#change-prj-root)
