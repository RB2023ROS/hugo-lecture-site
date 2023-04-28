---
title: "Lecture4. ROS 2 Node and Parameter Programming"
date: 2023-04-28T10:51:29+09:00
draft: false
---

지난 시간 살펴본 바와 같이 ROS 2에서 프로그램이 실행되는 단위는 Node라는 이름으로 관리됩니다.

여러분들께서 원하는 기능을 수행하는 Node를 생성하고, 이를 담은 패키지를 작성하고, 관련된 패키지들을 묶어 메타패키지를 구성하는 식으로 ROS 2 개발이 진행됩니다.

> 오늘은 그 시작이 되는 Node의 기초 프로그래밍을 실습해보겠습니다.

![Untitled.png](/kr/ros2_foxy/images4/Untitled.png?height=300px)

첫 코딩 강의의 시작이므로, 제가 사용하는 환경과 셋업도 공유드릴 예정입니다.

- Python 개발 : VSCode와 Copilot
- C++ 개발 : Clion과 Copilot

{{% notice tip %}}

ROS 2 프로그래밍의 경우 일반적인 프로그래밍과 더불어 tf, 통신 메커니즘, 멀티 프로세싱 등 실제 동작시켜보지 않으면 알 수 없는 기능들의 구현이 잦습니다. 잦은 확인과 Command and Conquer 방식으로 접근해보겠습니다. 😊
{{% /notice %}}

## ROS 2 Node Programming - Python

- 제가 미리 준비해 둔 패키지를 빌드해봅시다.

```python
cd ~/ros2_ws/src
git clone https://github.com/RB2023ROS/du2023-ros2.git

# 빌드는 항상 WS의 최상단 디렉토리에서 진행합니다!!!
cd ~/ros2_ws
cbp py_node_tutorial
# or
colcon build --packages-select py_node_tutorial

# 빌드 후 항상 소싱!!!
cd ~/ros2_ws
rosfoxy
# or
source install/local_setup.bash
```

#### example 1 - Hello ROS 2

- 앞서 빌드된 예시를 실행시켜보겠습니다.

```python
$ ros2 run py_node_tutorial example_node_1
[INFO] [1680692820.096342214] [example_node_1]:
==== Hello ROS 2 ====
```

{{% notice tip %}}
모든 예제 코드는 [Github 링크](https://github.com/RB2023ROS/du2023-ros2/tree/main/py_node_tutorial/py_node_tutorial)에서도 확인할 수 있습니다.
{{% /notice %}}

### 코드 분석

> **rcl**은 ROS Client Libraries의 약자로 ROS 2에서는 rclc, rclcpp, rclpy, rcljs와 같은 다양한 언어를 지원하고 있습니다. 파이썬에서 ROS 2 개발을 하기 위해서는 필수적으로 rclpy의 import가 필요하며 Node의 사용을 위해서 Node class를 import 해야 합니다.

```python
# !/usr/bin/env python3

import rclpy
from rclpy.node import Node
```

- rclpy 코딩 규칙

ROS 2에서 파이썬 파일을 조회하고 실행하는 일련의 정해진 과정이 있어 아래와 같이 main()부분을 항상 따로 분리하여 작성하도록 합니다.

```python
if __name__ == '__main__':
    """main function"""
    main()
```

- Node 생성

```python
def main(args=None):
    """Do enter into this main function first."""
    rclpy.init(args=args)

    node = Node('node_name')
    node.get_logger().info('\n==== Hello ROS 2 ====')
    node.destroy_node()

    rclpy.shutdown()
```

실제 동작을 수행하는 main 함수를 살펴보면 다음과 같은 과정을 거치고 있습니다.

| Function                 | Description                                                                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| rclpy.init               | initialization, 즉 초기화를 하고 있습니다.                                                                                                  |
| node = Node('node_name') | Node를 생성하는 부분으로 앞서 import한 Node class를 사용하고 있습니다. 매개변수로 node의 이름이 들어갑니다.                                 |
| node.destroy_node()      | Node를 생성하고 원하는 작업을 모두 수행했다면, 이제 사용했던 Node를 제거해야 할 것입니다. (그래야 불필요한 자원의 낭비를 줄일 수 있겠지요.) |
| rclpy.shutdown()         | 이번 예시 첫 부분, rclpy.init()을 통하여 초기화를 해주었습니다. 이제 rclpy를 통한 작업이 모두 끝났으므로 안전하게 종료시켜줍니다.           |

⇒ 위 과정이 Python에서 rclpy를 통해 Node를 다루는 기본 절차입니다. (get_logger 부분은 이후에 계속해서 살펴보겠습니다.)

#### setup.py 수정

- 파이썬 파일을 `ros2 run` 으로 실행하기 위해서 패키지 내 **setup.py** 파일에 **entry_points**를 추가해 주어야 합니다.

```python
		entry_points={
        'console_scripts': [
            'example_node_1 = py_node_tutorial.node_example_1:main',
            'example_node_2 = py_node_tutorial.node_example_2:main',
            'example_node_3 = py_node_tutorial.node_example_3:main',
            'example_node_4 = py_node_tutorial.node_example_4:main',
            'example_node_5 = py_node_tutorial.node_example_5:main',
        ],
    },
```

- 작성하는 방법은 다음과 같습니다. ⇒ `실행 시 사용될 이름 = <패키지 이름>.<파일 이름>.main`

| 실행 시 이름   | 패키지 이름      | 파일 이름      |
| -------------- | ---------------- | -------------- |
| example_node_1 | py_node_tutorial | node_example_1 |

{{% notice tip %}}
진입 지점(**entry_points**)를 설정하지 않으면 파이썬 파일이 있어도 **ros2 run** 과 같은 커멘드를 사용할 수 없게 됩니다.
{{% /notice %}}

{{% notice note %}}
[과제] 나만의 Node를 구현하는 파이썬 스크립트를 생성하여 ros2 run으로 실행할 수 있도록 해보세요!
{{% /notice %}}

#### example 2 - timer

- 계속해서 Timer 예시입니다. 로봇은 일반적으로 실행된 이후 계속해서 작업을 수행해야 하기에 **주기적으로 무언가를 실행**하는 일이 잦습니다. 이를 구현하는 Timer를 살펴봅시다.

```bash
$ ros2 run py_node_tutorial example_node_2
==== Hello ROS 2 : 1====
==== Hello ROS 2 : 2====
==== Hello ROS 2 : 3====
==== Hello ROS 2 : 4====
==== Hello ROS 2 : 5====
```

- main문에 추가된 **create_timer**와 timer_callback 함수를 확인할 수 있습니다.

```python
def timer_callback():
    """Timer will run this function periodically."""
    global count
    count += 1
    print(f'==== Hello ROS 2 : {count}====')

def main(args=None):
    """Do enter into this main function first."""
    rclpy.init(args=args)

    node = Node('node_name')
    node.create_timer(0.2, timer_callback)
    rclpy.spin(node)

    node.destroy_node()

    rclpy.shutdown()
```

timer 생성 시 **create_timer**라는 함수가 사용됩니다.

| Code             | Description               |
| ---------------- | ------------------------- |
| timer_period_sec | 실행 주기 (초)            |
| callback         | 해당 주기마다 실행될 함수 |

{{% notice note %}}
동기/비동기의 측면에서 timer와 Node의 분석은 이후 강의에서 이어집니다. 지금은 기본적인 코딩에만 집중하겠습니다!
{{% /notice %}}

#### example 3 - spin_once, spin

- Node의 상태를 살피면서 반복 실행시키는 **spin** 함수에 대해 좀 더 자세하게 살펴봅니다.

```python
$ ros2 run py_node_tutorial example_node_3
==== Hello ROS 2 : 1====
==== Hello ROS 2 : 2====
==== Hello ROS 2 : 3====
...
```

- Node는 상태를 지속 유지하면서 변경된 내용에 따라 지정된 동작을 수행해야 합니다. 이는 로봇 프로그램에서 매우 보편적인 작업으로, ROS 2에서는 **spin()**이라는 이름의 함수로 기능을 제공하고 있습니다.

```python
def main(args=None):
    """Do enter into this main function first."""
    rclpy.init(args=args)

    node = Node('node_name')
    node.create_timer(0.2, timer_callback)

    # 1. spin_once() will run node only once.
    # timeout_sec: Seconds to wait.
    # Block forever if ``None`` or negative. Don't wait if 0.
    rclpy.spin_once(node, timeout_sec=10)

    # # 2. spin() will run node until Ctrl+C.
    rclpy.spin(node)

    # # 3. spin_once() with while loop will run node periodically.
    while True:
        rclpy.spin_once(node, timeout_sec=10)

    node.destroy_node()

    rclpy.shutdown()
```

spin을 비롯하여 **spin_once**, **spin_until_future_complete**와 같이 프로그램의 실행을 관리하기 위한 다양한 추가 함수들이 존재하며, 예시에서는 3가지 자주 사용되는 패턴을 제시하고 있습니다.

| Code                | Description                                     |
| ------------------- | ----------------------------------------------- |
| spin_once()         | Node를 단 한번만 실행                           |
| spin()              | Ctrl+C 입력 전까지 계속해서 실행                |
| while + spin_once() | spin과 더불어 다른 무언가를 추가할 수 있는 패턴 |

#### example 4 - OOP Node

- 예제 실행의 결과는 이전과 같습니다. 하지만 구현에서 OOP를 사용한다는 차이를 갖는 예시입니다.

```python
$ ros2 run py_node_tutorial example_node_4
[INFO] [1657348011.971419700] [composition_example_node]: ==== Hello ROS 2 : 1====
[INFO] [1657348012.163466100] [composition_example_node]: ==== Hello ROS 2 : 2====
[INFO] [1657348012.363590700] [composition_example_node]: ==== Hello ROS 2 : 3====
```

- class를 사용하여 Node를 구현한 모습을 확인할 수 있습니다.

```python
class NodeClass(Node):
    """Second Node Class.
    Just print log periodically.
    """

    def __init__(self):
        """Node Initialization.
        You must type name of the node in inheritanced initializer.
        """
        super().__init__('composition_example_node')
        self.create_timer(0.2, self.timer_callback)

        self._count = 1

    def timer_callback(self):
        """Timer will run this function periodically."""
        self.get_logger().info(f'==== Hello ROS 2 : {self._count}====')
        self._count += 1
```

ROS 1과 달리, ROS 2의 OOP 구현은 **Node**를 상속받습니다. (때문에 생성 시, Node이름을 `super().__init__()`안에 넣어주어야 합니다.) 이렇게 객체지향을 사용하면 Node의 기능들을 적극 활용하여 더욱 쉽고 강력한 ROS 2 개발이 가능해집니다.

- rclpy logger

```python
		super().__init__('node_name')
		...
    node.get_logger().info('\n==== Hello ROS 2 ====')
```

ROS 1의 rospy.loginfo()와 같이 rclpy에서도 get_logger()라는 logging API를 제공합니다. 다만, rclpy의 logger는 Node에 종속되는 개념입니다. (ROS 2에서는 여러 Node가 하나의 프로세스 안에서 실행될 수 있기 때문입니다.)

get_logger()를 사용하면 일반적인 print 콘솔 출력과는 달리, 실행중인 Node이름, 시간, 위험성 등을 디버깅할 수 있어 이후 복잡한 시스템에서 큰 도움이 됩니다.

#### example 5 - Logger Level

- 기본 node 프로그래밍의 마지막 예시입니다.

```python
$ ros2 run py_node_tutorial example_node_5
[INFO] [1657348108.163389800] [node_name]: ==== Hello ROS 2 : 1====
[WARN] [1657348108.163810900] [node_name]: ==== Hello ROS 2 : 1====
[ERROR] [1657348108.164126200] [node_name]: ==== Hello ROS 2 : 1====
[FATAL] [1657348108.164514300] [node_name]: ==== Hello ROS 2 : 1====
...
```

![py_node_logger.gif](/kr/ros2_foxy/images4/py_node_logger.gif?height=300px)

ROS 2에서는 위험도에 따라서 다른 logger level을 적용할 수 있습니다. info를 기준으로 아래로 갈수록 높은 레벨의 log이며, 제일 심각한 error와 fatal의 경우, 콘솔 출력시에도 빨간 글씨로 보이는 것을 확인할 수 있습니다.

- debug의 경우 실제 콘솔 출력으로는 나타나지 않으며, 효과적인 Tracking을 위해 이후 학습할 **rqt console** 사용을 권장합니다.

![Untitled1.png](/kr/ros2_foxy/images4/Untitled1.png?height=300px)

## ROS 2 Parameter Programming - Python

모터의 게인값, 카메라의 Focal Lenght, Intrinsic, Extrinsic Parameter, Point Cloud의 resolution과 min/max range 등 로봇 프로그래밍 시 수많은 매개변수들이 사용됩니다.

ROS 2에서는 각종 매개변수를 다룰 수 있는 별도의 Parameter Server를 제공하며, Node들은 모두 개별 Parameter Server를 갖고 있습니다.

- 예제 Package 빌드와 실행

```bash
$ colcon build --packages-select py_param_tutorial
$ source install/local_setup.bash
$ ros2 run py_param_tutorial param_example
[INFO] [1672390971.030532687] [param_ex_node]:
string_param: world
int_param: 119
float_param: 3.1415
arr_param: [1, 2, 3]
nested_param.string_param: Wee Woo
```

⇒ param_ex_node에서 5종류의 매개변수가 선언되었습니다. 이들을 확인하는 커멘드 라인을 배워봅시다.

- ros2 param list

```bash
$ ros2 param list
/param_ex_node:
  arr_param
  float_param
  int_param
  nested_param.string_param
  string_param
  use_sim_time
```

⇒ param_ex_node의 Parameter Server는 현재 6개의 값을 갖고 있습니다.

- 개별 Parameter들의 값을 다루고 싶은 경우 ros2 param get/set 을 사용합니다.

```bash
$ ros2 param get /param_ex_node arr_param
Integer values are: array('q', [1, 2, 3])

$ ros2 param set /param_ex_node arr_param '[1,2,3,4]'
Set parameter successful

$ ros2 param get /param_ex_node arr_param
Integer values are: array('q', [1, 2, 3, 4])
```

⇒ node 이름과 매개변수 이름을 모두 필요로 함에 주의합니다.

### 코드 분석

- parameter의 생성은 node 내에서 이루어지며, **declare_parameter**를 통해 생성합니다. (함수의 두번째 인자는 기본값입니다.)

```python
class ParamExNode(rclpy.node.Node):
    def __init__(self):
        super().__init__('param_ex_node')

        self.declare_parameter('string_param', 'world')
        self.declare_parameter('int_param', 119)
        self.declare_parameter('float_param', 3.1415)
        self.declare_parameter('arr_param', [1,2,3])
        self.declare_parameter('nested_param.string_param', 'Wee Woo')
```

⇒ **nested_param**과 같이 parameter는 **계층 구조**를 가질 수 있으며 온점을 통해 구분할 수 있습니다. ⇒ 현재 string_param이라는 이름을 가진 parameter가 두 종류 존재하지만 서로 소속된 계층이 달라 공존할 수 있는 것입니다.

- 선언된 매개변수의 값은 **get_parameter**를 통해 확인 가능합니다. get_parameter 자체는 Object이고, value 속성이 실제 값을 갖고 있음에 주의합니다.

```python
		string_param = self.get_parameter('string_param')
    int_param = self.get_parameter('int_param')
    float_param = self.get_parameter('float_param')
    arr_param = self.get_parameter('arr_param')
    nested_param = self.get_parameter('nested_param.string_param')

    self.get_logger().info(f"\nstring_param: {string_param.value} \
        \nint_param: {int_param.value} \
        \nfloat_param: {float_param.value} \
        \narr_param: {arr_param.value} \
        \nnested_param.string_param: {nested_param.value}"
    )
```

- parameter는 launch file에서도 설정할 수 있는데요. 예시를 우선 실행해봅시다. - 변경된 Parameter값도 확인해봅시다.

```bash
$ ros2 launch py_param_tutorial launch_with_param.launch.py
...
[param_example-1] [INFO] [1672387864.135213913] [param_example]:
[param_example-1] string_param: Hello
[param_example-1] int_param: 112
[param_example-1] float_param: 3.1415
[param_example-1] arr_param: [1, 2, 3]

$ ros2 launch py_param_tutorial launch_with_yaml.launch.py
...
[param_example-1] [INFO] [1680857428.211772092] [param_example]:
[param_example-1] string_param: Yaml Yaml
[param_example-1] int_param: 5
[param_example-1] float_param: 3.14
[param_example-1] arr_param: ['I', 'love', 'ROS 2']
[param_example-1] nested_param.string_param: Ooh Wee
```

- Node를 실행하는 launch file을 기억하시지요? Node의 parameters 옵션을 사용하면 코드를 직접 바꾸지 않고 parameter의 변경이 가능합니다.

```python
def generate_launch_description():

    param_ex_node = Node(
        package='py_param_tutorial',
        executable='param_example',
        name='param_example',
        output='screen',
        parameters=[
            {'string_param': 'Hello'},
            {'int_param': 112},
        ],
    )
```

- parameter가 매우 많은 경우에는 yaml 형식의 파일을 사용해 관리할 수 있는데요, yaml 파일을 사용하는 launch 파일의 일부를 함께 살펴봅시다.

```python
def generate_launch_description():

    config = os.path.join(
        get_package_share_directory('py_param_tutorial'), 'config', 'params.yaml'
    )

    param_ex_node = Node(
        package = 'py_param_tutorial',
        executable = 'param_example',
        name = 'param_example',
        output='screen',
        parameters = [config]
    )

    return LaunchDescription([
        param_ex_node
    ])
```

{{% notice tip %}}
추가 설명 - os.path.join, get_package_share_directory를 사용하는 이유!
{{% /notice %}}

- yaml 파일은 config/params.yaml에 위치하고 있습니다. parameter 관리 용도로 사용하기 위해서 yaml 파일은 일정한 규칙을 갖춰야 합니다.

```yaml
<node-name>:
	ros__parameters:
		<param-name>: <param-value>
		...
		<nested-layer-name>:
			<param-name>: <param-value>

param_example:
  ros__parameters:
    string_param: "Yaml Yaml"
    int_param: 5
    float_param: 3.14
    arr_param: ['I', 'love', 'ROS 2']
    nested_param:
      string_param: "Ooh Wee"
```

- 이렇게 새로운 폴더와 파일을 추가한 경우, python 패키지의 setup.py를 수정해주어야 하며 패키지 빌드도 새로 해주어야 합니다.

```python
import os
from glob import glob
from setuptools import setup

package_name = 'py_param_tutorial'

setup(
    name=package_name,
    version='0.0.0',
    packages=[package_name],
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        (os.path.join('share', package_name, 'config'), glob('config/*.yaml')),
        (os.path.join('share', package_name, 'launch'), glob('launch/*.launch.py')),
    ],
```

- launch file에 추가된 내용을 다시 살펴보면, 방금 전의 yaml 파일을 불러와서 node의 실행 option에 전달하고 있습니다.

### Namespace Parameter

아직 이른 감이 있지만, 다수의 로봇을 다루는 경우를 생각해봅시다. 종류는 같지만 별도의 Parameter가 필요할 것입니다.

![Untitled2.png](/kr/ros2_foxy/images4/Untitled2.png?height=300px)

⇒ 이러한 경우를 대비해서 ROS 2에서는 **namespace**라는 기능을 지원합니다. topic과 node 앞에 별도의 접미사를 추가하여 서로 다른 객체임을 구별하는 방식입니다.

{{% notice note %}}
이를 지금 언급하는 이유는, Parameter의 namespace를 지정하는 일이 매우 빈번하고 그 사용 방식이 node나 topic과는 조금 다르기 때문입니다.
{{% /notice %}}

- namespace를 지정하는 가장 일반적인 방법은 launch 파일에서 옵션을 지정하는 것입니다.

```python
		param_ex_node = Node(
        package = 'py_param_tutorial',
				# namespace 지정!
        namespace = 'robot1',
        executable = 'param_example',
        name = 'param_example',
        output='screen',
        parameters = [config]
    )
```

- 예시를 다시 실행시킨 다음 어떤 결과를 얻게 되는지 살펴봅시다.

```python
$ ros2 launch py_param_tutorial launch_with_yaml.launch.py
...
[param_example-1] [INFO] [1680861992.330212575] [robot1.param_example]:
[param_example-1] string_param: world
[param_example-1] int_param: 119
[param_example-1] float_param: 3.1415
[param_example-1] arr_param: [1, 2, 3]
[param_example-1] nested_param.string_param: Wee Woo
```

⇒ yaml 파일이 반영되지 않은 결과를 얻었습니다!

- yaml 파일에도 namespace를 반영해주어야 하기 때문이며 예를 들어 robot1이라는 namespace를 반영하고 싶다면 yaml 파일을 아래와 같이 변경해야 합니다.

```python
robot1/param_example:
  ros__parameters:
    string_param: "Yaml Yaml"
    int_param: 5
    float_param: 3.14
    arr_param: ['I', 'love', 'ROS 2']
    nested_param:
      string_param: "Ooh Wee"
```

- 하지만 namespace를 바꿀 때마다 새로운 yaml파일을 만들 수는 없습니다. 이를 대비하여 ROS 2에서는 다음과 같은 yaml 문법을 허용합니다.

```python
/**:
  ros__parameters:
    string_param: "Namespaced Param"
    int_param: 10
    float_param: 2.718
    arr_param: ['I', 'absolutely', 'love', 'ROS 2']
    nested_param:
      string_param: "Yeah Hee"
```

- 새롭게 작성한 yaml 파일을 반영한 새로운 예시를 실행해봅시다.

```python
$ ros2 launch py_param_tutorial launch_with_namespace.launch.py
...
[param_example-1] [INFO] [1680862245.965886805] [robot1.param_example]:
[param_example-1] string_param: Namespaced Param
[param_example-1] int_param: 10
[param_example-1] float_param: 2.718
[param_example-1] arr_param: ['I', 'absolutely', 'love', 'ROS 2']
[param_example-1] nested_param.string_param: Yeah Hee
```

> 다수의 센서를 사용하거나, 다수의 로봇을 사용하는 경우 지금 학습한 namespace를 꼭 기억해주세요!!

## ROS 2 Node Programming - C++

- 앞선 Node 프로그래밍 예시들을 C++에서 구현하는 방법을 배워봅시다. 제가 미리 준비해 둔 패키지를 빌드하며 시작하겠습니다.

```python
# 빌드는 항상 WS의 최상단 디렉토리에서 진행합니다!!!
cd ~/ros2_ws
cbp cpp_node_tutorial
# or
colcon build --packages-select cpp_node_tutorial

# 빌드 후 항상 소싱!!!
cd ~/ros2_ws
rosfoxy
# or
source install/local_setup.bash
```

#### example 1 - Hello ROS 2

- 파이썬 때와 마찬가지로, 앞서 빌드된 예시를 실행시켜보겠습니다.

```python
$ ros2 run cpp_node_tutorial example_node_1
[INFO] [1680692820.096342214] [example_node_1]:
==== Hello ROS 2 ====
```

### ROS 2 C++ 코드 빌드와 실행

ROS 2에서는 C++ 코드의 빌드를 위해 **CMake**를 사용하고 있습니다. 코드 분석 이전에 새로운 코드를 생성하고, 빌드하는 과정을 간단히 설명해보고자 합니다.

1. 새로운 코드 생성 -C++ 코드는 패키지 내 **src** 폴더에 생성하는 것이 관례입니다. 단, 작업 전 해당 패키지가 C++용으로 만들어져 있는지 꼭 확인하셔야 합니다. (ament_cmake를 사용했던 점을 복기해봅시다.)

2. 코드 작성- 우선, 패키지 내 존재하는 예시 혹은 copilot을 사용하여 가장 간단한 node code를 생성하겠습니다.

3. CMakeLists.txt 수정

```yaml
find_package(rclcpp REQUIRED)

add_executable(test_node src/test.cpp)
ament_target_dependencies(test_node rclcpp)

install(
TARGETS
test_node
DESTINATION
lib/${PROJECT_NAME}
)
```

| Function                  | Description                                                                                                |
| ------------------------- | ---------------------------------------------------------------------------------------------------------- |
| find_package              | 필요한 종속성들을 추가합니다. ROS 2 종속성 뿐만 아니라, 3rd party 종속성도 여기에서 추가됩니다.            |
| add_executable            | Build 후 실행 파일을 생성합니다. (일반적인 CMake 과정과 동일합니다.)                                       |
| ament_target_dependencies | CMake의 target_link_libaries와 동일한 역할을 하지만, ROS 2에서 사용을 위해 편의성이 추가된 Function입니다. |
| install                   | 빌드 결과물을 특정 디렉토리에 위치시킵니다.                                                                |

{{% notice note %}}
모든 CMake function들은 ament_package() 상단에 위치해야 합니다!
{{% /notice %}}

- 다음으로, 새로운 코드의 빌드 작업을 함께 해봅시다.

```python
# 빌드는 항상 WS의 최상단 디렉토리에서 진행합니다!!!
cd ~/ros2_ws
cbp cpp_node_tutorial
# or
colcon build --packages-select cpp_node_tutorial

# 빌드 후 항상 소싱!!!
cd ~/ros2_ws
rosfoxy
# or
source install/local_setup.bash

# 새롭게 빌드한 예시 실행
$ ros2 run cpp_node_tutorial test_node
[INFO] [1681299676.469101345] [example_node_1]: ==== Hello ROS 2 ====
```

### 코드 분석

- python 코드 작성 시 rclpy를 import 했던 것처럼, cpp 코딩 시 rclcpp을 import 해야 합니다. (해당 header안에는 memory, vector, string 등 범용적으로 사용되는 웬만한 라이브러리가 포함되어 있습니다.)

```cpp
#include <rclcpp/rclcpp.hpp>
```

- Node 생성

```cpp
int main(int argc, char **argv) {
  // Initialize ROS 2
  rclcpp::init(argc, argv);

  // Create a node
  auto node = rclcpp::Node::make_shared("example_node_1");

  // Log a message
  RCLCPP_INFO(node->get_logger(), "==== Hello ROS 2 ====");

  // Shutdown ROS 2
  rclcpp::shutdown();
  return 0;
}
```

- python 코드 분석 시 대부분의 함수를 살펴보았으므로 간단하게 짚고 넘어가겠습니다.

| Function                    | Description                                                                                                                       |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| rclcpp::init(argc, argv)    | initialization, 즉 초기화를 하고 있습니다.                                                                                        |
| rclcpp::Node::make_shared() | Node를 생성하는 부분으로 앞서 import한 Node class를 사용하고 있습니다. 매개변수로 node의 이름이 들어갑니다.                       |
| rclcpp::shutdown()          | 이번 예시 첫 부분, rclpy.init()을 통하여 초기화를 해주었습니다. 이제 rclpy를 통한 작업이 모두 끝났으므로 안전하게 종료시켜줍니다. |
|                             | 스마트 포인터를 사용하기 때문에 destroy_node라는 것은 별도로 호출하지 않습니다.                                                   |

### example 2 - timer

- 계속해서 Timer 예시입니다.

```cpp
#include "rclcpp/rclcpp.hpp"

using namespace std::chrono_literals;

static int count = 0;

void timer_callback(){
  std::cout << "==== Hello ROS 2 : " << count << " ====" << std::endl;
  count++;
}

int main(int argc, char **argv) {
  rclcpp::init(argc, argv);
  auto node = rclcpp::Node::make_shared("example_node_2");
  auto timer = node->create_wall_timer(200ms, timer_callback);

  rclcpp::spin(node);

  rclcpp::shutdown();
  return 0;
}
```

| Code              | Description                                             |
| ----------------- | ------------------------------------------------------- |
| create_wall_timer | Timer 생성, 실행 주기와 callback을 매개변수로 갖습니다. |
| timer_callback    | 특정 해당 주기마다 실행될 함수                          |

- Timer, Node와 같은 인터페이스들은 모두 스마트 포인터를 사용하고 있습니다. 따라서 함수의 인자로 전달하거나, 내부 함수를 호출할 시 포인터라는 점을 기억하시어 코딩하셔야 합니다.

```python
auto node => std::shared_ptr<rclcpp::Node>
auto timer => std::shared_ptr<rclcpp::Timer>
```

### example 3 - OOP Node

- OOP를 사용한 Timer Node 구현입니다.

```python
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

- 모든 rclcpp Node들은 반드시 rclcpp::Node를 **public** 상속받아야 합니다.

```python
class NodeClass: public rclcpp::Node {
public:
  NodeClass(): Node("example_node_4") {}
};
```

상속을 받은 만큼 구현 시 주의해야 하는 부분들이 있으며, 생성자는 반드시 Access Modifier public을 가져야 합니다.

- rclcpp logger

```python
RCLCPP_INFO(node->get_logger(), "==== Hello ROS 2 ====");
```

python의 get_logger()와 동일하지만, RCLCPP_INFO로 wrapping이 되어있다는 차이점을 기억하시기 바랍니다.

- OOP 구현 시 새로운 Node 생성을 위해 template을 사용합니다.

```cpp
auto node = std::make_shared<NodeClass>();
```

### example 4 - spin_once, spin

- Node의 상태를 살피면서 반복 실행시키는 spin 함수가 추가되며 OOP 구현이 보입니다.

```cpp
class NodeClass: public rclcpp::Node {
private:
  size_t count;
  rclcpp::TimerBase::SharedPtr timer;

  void timer_callback() {
    RCLCPP_INFO(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
    count++;
  }

public:
  NodeClass() : Node("example_node_5") {
    timer = this->create_wall_timer(
      std::chrono::milliseconds(200),
      // timer_callback,
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

- rclpy와 마찬가지로, rclcpp에서도 spin, spin_some, spin_until_future_complete등 다양한 기능을 제공하는데요. spin을 이해하기 위해서 많은 배경지식이 필요하니 구체적인 내용은 이후에 살펴보고 지금은 기능 구현에 집중하겠습니다.

```python
rclcpp::spin(node);
```

- Timer를 비롯한 인터페이스들은 모두 shared_ptr 타입을 갖습니다. SharedPtr은 ROS 2 단에서 편의를 위해 wrapping한 것입니다. - [참고 링크](https://docs.ros2.org/latest/api/rclcpp/classrclcpp_1_1TimerBase.html)

```cpp
rclcpp::TimerBase::SharedPtr timer;
```

### example 5 - Logger Level

- 기본 node 프로그래밍의 마지막 예시였습니다. 실행 후 간단하게 살펴보고 넘어가겠습니다.

```cpp
class NodeClass: public rclcpp::Node {
private:
  size_t count;
  rclcpp::TimerBase::SharedPtr timer;

  void timer_callback() {
    RCLCPP_DEBUG(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
    RCLCPP_INFO(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
    RCLCPP_WARN(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
    RCLCPP_ERROR(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
    RCLCPP_FATAL(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
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

- 코드 중 logger level을 지정하는 부분을 확인 가능합니다.

```python
RCLCPP_DEBUG(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
RCLCPP_INFO(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
RCLCPP_WARN(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
RCLCPP_ERROR(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
RCLCPP_FATAL(this->get_logger(), "==== Hello ROS 2 : %d ====", count);
```

## ROS 2 Parameter Programming - C++

- 예제 Package 빌드와 실행

```bash
$ colcon build --packages-select cpp_param_tutorial
$ source install/local_setup.bash

$ ros2 run cpp_param_tutorial cpp_param_example
[INFO] [1681300487.037502527] [param_ex_node]:
str: world
int: 119
double: 3.141500
arr: [1, 2, 3]
nested: Wee Woo
```

### **코드 분석**

- rclcpp 코드에서 parameter의 생성과 사용은 다음과 같습니다.

| Code                                   | Description                                                         |
| -------------------------------------- | ------------------------------------------------------------------- |
| declare_parameter(name, default_value) | parameter의 생성                                                    |
| get_parameter()                        | parameter object를 받아옵니다.                                      |
| as\_<type>()                           | get_parameter 결과는 object로 실제 값을 파싱하기 위해서 필요합니다. |

- 코드를 통해서도 확인이 가능하며, 최소 3 줄의 라인이 필요하기 때문에 get_parameter와 as_type을 한번에 사용하기도 합니다.

```cpp
class ParamExNode : public rclcpp::Node
{
public:
    ParamExNode() : Node("param_ex_node") {

        this->declare_parameter("string_param", "world");
				...
        rclcpp::Parameter str_param = this->get_parameter("string_param");
        ...
        std::string my_str = str_param.as_string();
```

- 간소화 버전

```cpp
this->declare_parameter("string_param", "world");
auto my_str = this->get_parameter("string_param").as_string();
```

- cpp로 구현된 node의 parameter도 ros2 launch와 yaml 파일을 통해 손쉽게 수정이 가능합니다.

```bash
$ ros2 launch cpp_param_tutorial launch_with_yaml.launch.py
...
[cpp_param_example-1] [INFO] [1681301614.005450889] [param_example]:
[cpp_param_example-1] str: Yaml Yaml
[cpp_param_example-1] int: 5
[cpp_param_example-1] double: 3.141500
[cpp_param_example-1] arr: [1, 2, 3]
[cpp_param_example-1] nested: Ooh Wee
```
