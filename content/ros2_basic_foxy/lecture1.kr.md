---
title: "Lecture1 - Introduction to ROS 2"
date: 2023-01-02T12:39:15+09:00
draft: false
---

## About ROS 2

![lec1_0.png](/kr/ros2_basic_foxy/images1/lec1_0.png?height=150px)

- image from : [foxglove.dev](https://foxglove.dev/blog/the-building-blocks-of-ros2)

> **ROS의 시작은 연구실**이었습니다. 구글의 핵심 개발자였던 **Scott Hassan**은 2006년 **Willow Garage**를 설립하였고, 여기서 처음으로 ROS가 탄생하였습니다. 이후 Willow Garage는 여러 Spin-off를 거친 뒤 OSRF와 OSRC등으로 나뉘게 되었고, 현재 Gazebo와 ROS는 이들에 의해 관리되고 있습니다.

2022년 12월 기준 구글의 자회사인 Alphabet의 Intrinsic이 OSRF를 품게 되었으며, Ubuntu Linux를 관리하는 Canonical에서 ROS의 공식 서포트를 약속하고, Jetson 보드와 Issac Sim들을 개발하고 있는 Nvidia에서도 ROS를 공식 지원하고 있습니다.

이렇게 많은 기업들이 ROS에 거는 기대가 큰 만큼 로봇 시장의 성장성도 기대가 됩니다. 하지만, 로봇이 상용화되기 위해서는 여러 난관이 있습니다.

- **보안**
- **안정성**
- **실시간성**
- **개발 용이성**
- **기술 지원**
- **하드웨어 연동**
- **etc…**

연구용으로 설계된 기존 ROS를 통해서는 이러한 모든 조건을 충족할 수 없다는 결론을 내렸고, Open Robotics는 **ROS 2**를 새롭게 선보이게 됩니다.

### ROS 1 vs ROS 2

기존 언급되었던 ROS 1의 모든 문제들이 ROS 2에서 해결되었다고 할 수는 없지만 적어도 해결을 고려하여 설계되었다고 말하고 싶습니다.

ROS 1과 비교하여 ROS 2의 장점들을 간단히 살펴봅시다.

![lec1_1.png](/kr/ros2_basic_foxy/images1/lec1_1.png?height=400px)

- image from : [maker.pro](https://maker.pro/ros/tutorial/robot-operating-system-2-ros-2-introduction-and-getting-started)

상용화를 고려한 가장 큰 변화로 패킷 통신에 TCPROS/UPDROS가 아닌 DDS(Data Distribution Service)를 도입하였다는 것입니다.

![lec1_2.png](/kr/ros2_basic_foxy/images1/lec1_2.png?height=200px)

- image from : [omg.org](https://www.omg.org/dds-directory/)

뿐만 아니라, 임베디드 시스템을 위한 micro-ros와 같은 멋진 프로젝트들도 ROS 2에서 새롭게 등장하였습니다.

![lec1_3.png](/kr/ros2_basic_foxy/images1/lec1_3.png?height=300px)

- image from : [freertos](https://www.freertos.org/2020/09/micro-ros-on-freertos.html)

이렇게 멋진 ROS 2를 지금부터 함께 배워보겠습니다.

## About this lecture

ROS 1 강의에서는 **ROS Noetic** 버전을 사용하였으며, 동일한 Ubuntu 20.04 버전에서 구동되는 **ROS 2 Foxy**를 사용할 예정입니다.

ROS 설정을 잘 따라오셨다면 별도의 설치 과정은 필요 없으며, 리눅스 커멘드 사용법, 패키지 설치 등 기본적인 내용은 알고 있다는 가정 하에 강의를 진행해보겠습니다.

![lec1_4.png](/kr/ros2_basic_foxy/images1/lec1_4.png?height=300px)

- image from : [docs.ros.org](https://docs.ros.org/en/foxy/index.html)

ROS 1강의를 통해 탄탄하게 다진 기본기를 바탕으로 아래와 같은 내용들을 다뤄봅니다.

- **ROS 2 기본 커멘드**
- **C++ ROS 프로그래밍**
- **Nav 2를 사용한 자율주행**
- **실제 로봇 데이터 활용해보기**
- **과제를 통한 ROS 2 응용 프로그래밍**

더불어, 이번 강의를 통해 실제 로봇이 어떻게 개발되는지 바닥부터 살펴보고자 합니다.

- **로봇 설계**
- **시뮬레이션 제작과 실습**
- **실제 로봇을 만들기 위해 필요한 요소들**
- **유지 보수와 로봇 제품 개발**

Road Balance의 차량형 교육 로봇, **SRC**를 통해 실습해보겠습니다.

![lec1_5.png](/kr/ros2_basic_foxy/images1/lec1_5.png?height=300px)

### 강의를 수강하기 위해 필요한 선수지식

ROS 2는 Python 3와 C++ 11이상의 버전을 지원합니다. 따라서 최소한의 프로그래밍 지식이 있다는 가정 하에 강의가 진행되며, 환경 설정, ROS 개념 등 **ROS 1 강의를 모두 수강했다**는 전제 하에 진행됩니다.

![lec1_6.png](/kr/ros2_basic_foxy/images1/lec1_6.png?height=350px)

- image from : [docs.ros.org](https://docs.ros.org/en/foxy/Concepts/About-Internal-Interfaces.html)

이번 강의에서는 좀 더 실질적인 ROS 2 개발을 맛보고자 C++를 활용한 프로그래밍도 준비해 보았으니 이번 기회에 C++를 공부해 보는 것도 좋은 기회가 되실 겁니다.

### ROS 2 Workspace 생성

ROS 1에서 catkin build system을 사용한 것과 유사하게, ROS 2에서는 **colcon**이라는 빌드 시스템을 사용하고 있습니다. colcon을 사용하기 위해서 Workspace가 필요하며, ROS 1과 ROS 2를 모두 사용하는 현재 시스템 같은 경우, 혼란스럽지 않도록 이름을 달리 설정하겠습니다.

```bash
cd ~/
mkdir -p ros2_ws/src
cd ros2_ws
colcon build
```

- 아래와 같은 폴더 구조가 생성되었을 것입니다.

![lec1_7.png](/kr/ros2_basic_foxy/images1/lec1_7.png?height=150px)

- **build** : 컴파일 된 C++ 프로그램, custom interface 등이 위치하게 됩니다.
- **install** : ros2 launch와 ros2 run 등의 명령어는 프로그램의 실행 시 이 install 폴더 내 파일들을 조회합니다. 일종의 바로가기들의 모임이라고 생각하면 됩니다.
- **log** : colcon build 시 발생하는 로드들이 위치하게 됩니다.
- **src** : 모든 소스 코드들이 위치하게 됩니다.

{{% notice note %}}
Package를 지우고 싶은 경우 ⇒ build와 install 폴더에서 해당 package에 해당하는 내용들을 삭제합니다.
{{% /notice %}}

colcon을 사용하여 package를 빌드하는 방법들을 간단히 소개합니다.

- **colcon build** : src 폴더 내부에 존재하는 모든 package들을 빌드합니다.
- **colcon build --packages-up-to <pkg-name>** : 해당 package의 종속성이 존재할 시, 이들을 먼저 빌드하고 pkg-name을 빌드합니다.
- **colcon build --packages-select <pkg-name>** : 해당 package만을 빌드합니다.

새로운 Package를 빌드한 다음 ROS 2에서도 setup.bash를 source 해주어야 합니다. 이 작업에는 크게 두가지가 존재합니다.

- `source install/setup.bash` ⇒ workspace를 source하고 ROS 2시스템 전체를 갱신합니다.
- `source install/local_setup.bash` ⇒ workspace만을 sources합니다. (여러 ROS 2 workspace가 있는 경우 local_setup.bash를 사용합시다.)

```python
# source chained prefixes
# setting COLCON_CURRENT_PREFIX avoids determining the prefix in the sourced script
COLCON_CURRENT_PREFIX="/opt/ros/foxy"
_colcon_prefix_chain_bash_source_script "$COLCON_CURRENT_PREFIX/local_setup.bash"

# source this prefix
# setting COLCON_CURRENT_PREFIX avoids determining the prefix in the sourced script
COLCON_CURRENT_PREFIX="$(builtin cd "`dirname "${BASH_SOURCE[0]}"`" > /dev/null && pwd)"
_colcon_prefix_chain_bash_source_script "$COLCON_CURRENT_PREFIX/local_setup.bash"

unset COLCON_CURRENT_PREFIX
unset _colcon_prefix_chain_bash_source_script
```

마지막으로, 이번 실습에 필요한 소스 코드를 clone 하고, apt 패키지를 설치하면서, 강의를 마치겠습니다.

```python
cd ~/ros2_ws/src
git clone https://github.com/RB2023ROS/du2023-ros2.git

cd du2023-ros2
./setup_scripts.sh
```

**참고자료**

- [https://www.theconstructsim.com/infographic-ros-1-vs-ros-2-one-better-2/](https://www.theconstructsim.com/infographic-ros-1-vs-ros-2-one-better-2/)
- [https://rsl.ethz.ch/education-students/lectures/ros.html](https://rsl.ethz.ch/education-students/lectures/ros.html)
- [https://velog.io/@hwang-chaewon/ROS2006](https://velog.io/@hwang-chaewon/ROS2006)
