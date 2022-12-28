---
title: "Lecture1 - Introduction to ROS"
date: 2022-12-25T13:36:35+09:00
draft: false
---

## What is ROS?

> ROS란, 로봇 소프트웨어 개발에 사용되는 일종의 프레임워크입니다.

프레임워크라는 말은, ROS 나름대로의 실행 시나리오를 갖고 있다는 뜻입니다.

사용자인 우리들은, 이 시나리오를 사용하여 로봇을 다루는 우리만의 Application을 만들게 됩니다.

![lec1_0.png](/kr/ros_basic_noetic/images1/lec1_0.png?height=150px)

- image from : [wikimedia](https://commons.wikimedia.org/wiki/File:Ros_logo.svg)

> 그런데 왜 OS라는 이름이 붙게 되었을까요?

로봇을 실행하기 위해서, 수많은 프로그램들이 실행되며, ROS는 이들 사이의 우선순위와, 프로그램 사이의 데이터 흐름을 책임집니다. 이 작업은 스케쥴링이라고 불리며, 이러한 동작을 수행하는 시스템을 Operating System이라고 부르기 때문에 ROS라는 이름을 갖게 되었습니다.

![lec1_1.png](/kr/ros_basic_noetic/images1/lec1_1.png?height=300px)

- image from : [tutorialspoint](https://www.tutorialspoint.com/operating_system/os_process_scheduling.htm)

> 로봇을 개발하기 위해서 어떠한 프로그램들이 필요할까요?

로봇이 수행하는 임무들을 크게 3가지로 분류하면 인지, 판단, 제어의 3가지로 나뉩니다.

![lec1_2.png](/kr/ros_basic_noetic/images1/lec1_2.png?height=300px)

- **인지**란, 센서들을 통한 물체 인지, 자기 자신의 위치와 방향 인지, 상황 인지 등 로봇에게 있어 환경과 상호작용하는 과정에 해당합니다.
- **판단**이란, 앞/뒤로 움직일지, 로봇 팔을 뻗을지와 같이 인지를 기반으로 얻은 데이터를 통해 결정을 내리는 작업들이 해당할 것입니다.
- **제어**는 로봇에서 빼놓을 수 없는 영역으로, 로봇은 실제 세상에서 움직이기 때문에, 얼마나 움직일지, 어느정도의 속도로 힘은 얼마나 강하게 줄지 등 물리적인 임무를 포함합니다.

이렇게 로봇 시스템은 무척 복잡하며, 이뿐만 아니라 회로, 설계, 재료, 에너지 등을 고려해야 하는 완성품 로봇은 현대 공학의 집합 그 자체라고 말할 수 있습니다.

---

## About this lecture

이 강의에서 다루고자 하는 부분을 정확히하자면, 인지도 판단도 제어도 아닌, **시스템**입니다.

로봇의 센서, 구동부, 알고리즘이 모두 준비되어 있는 상황에서, 이들을 하나의 시스템으로 엮어주는 역할을 하는 것이 바로 ROS입니다.

![lec1_3.png](/kr/ros_basic_noetic/images1/lec1_3.png?height=300px)

- image from : [ROS Industrial](https://rosindustrial.org/developmentprocess)

ROS라는 시스템의 특성상 정해진 코드와 방법으로 소프트웨어를 개발해야 하며, 대부분 ROS를 다루는 강의라고 하면 이를 지칭합니다. 우리가 배우고자 하는 주된 내용도 바로 이 부분이라고 말할 수 있습니다.

- **ROS 개념**
- **ROS 커멘드 다루기**
- **ROS 프로그래밍 - Topic, Service, Action**
- etc…

하지만, 여기서 그치지 않고, 저는 좀 더 실질적인 로봇 개발을 이야기하고자 합니다.

- **리눅스 시스템**
- **Docker 사용하기**
- **로봇 시뮬레이션**
- **라이브 코딩과 에러 디버깅**

![lec1_4.png](/kr/ros_basic_noetic/images1/lec1_4.png?height=400px)

> 학교를 다니다 보면 아무리 많은 이론을 공부하고 머릿속에 집어넣어도, 시험을 치고 나면 모두 사라지곤 합니다. 머리 속에 남는 공부를 위해서는 직접 코딩을 해보고, 프로젝트를 진행해봐야 합니다.

---

### 강의를 수강하기 위해 필요한 선수지식

- 본 강의는 최대한 많은 분들이 끝까지 이해할 수 있도록 설계되었습니다.
- 따라서, 최대한 쉽고, 프로그래밍 실력이 출중하지 않아도 모두 완강할 수 있게 진행합니다.
- 하지만 그럼에도, 아래 코드를 이해할 수 있을 정도의 배경지식은 필요합니다.

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

- 파이썬 클래스, 메소드와 인스턴스
- for, while, if/else 등 기본 문법

---

### 강의 코드와 강의 노트 사용법

강의 도중 사용되는 코드들은 Github Repository를 통해 배포되어 있습니다. 코드 강의 중 지속해서 링크를 해드리며, 강의 시작 전 미리 살펴보시면 더욱 좋습니다.

- [https://github.com/RB2023ROS/du2023-ros1](https://github.com/RB2023ROS/du2023-ros1)

강의 노트의 주소는 [https://rb2023ros.github.io/kr/](https://rb2023ros.github.io/kr/) 입니다. 코드와 명령어 등 필요한 리소스를 모두 담고 있으므로 복사/붙여넣기를 활용하여 강의 청취 시간을 절약하시기 바랍니다.

{{% notice note %}}
참고로 해당 노트는 **hugo**를 사용하여 제작된 웹 페이지임을 밝힙니다.
{{% /notice %}}
