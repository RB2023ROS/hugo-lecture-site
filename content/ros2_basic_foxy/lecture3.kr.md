---
title: "Lecture3 - ROS 2 Node Programming, ROS 2 parameter"
date: 2023-01-02T12:39:11+09:00
draft: true
---

> Python을 사용하여 Node Programming을 실습해봅시다. 강의 마지막에는 간단히 C++ 코딩에 대해서도 다뤄보겠습니다.

### example 1 - Hello ROS 2

강의를 위해 준비된 예제 코드 패키지를 실습하고 분석하겠습니다.

```bash
$ colcon build --packages-select cbp py_node_tutorial
$ source install/local_setup.bash

$ ros2 run py_node_tutorial example_node_1
[INFO] [1672463872.778216198] [example_node_1]:
==== Hello ROS 2 ====
```

- 모든 예제 코드는 아래 링크에서 확인할 수 있습니다.

[https://github.com/RB2023ROS/du2023-ros2/tree/main/py_node_tutorial/py_node_tutorial](https://github.com/RB2023ROS/du2023-ros2/tree/main/py_node_tutorial/py_node_tutorial)

> 코드 분석을 차근차근 함께 해보겠습니다.

**rcl**은 **ROS Client Libraries**의 약자로 ROS 2에서는 **rclc, rclcpp, rclpy, rcljs**와 같은 다양한 언어를 지원하고 있습니다. 파이썬에서 ROS 2 개발을 하기 위해서는 필수적으로 rclpy의 import가 필요하며 Node의 사용을 위해서는 Node class를 import 해야 합니다.

```python
# !/usr/bin/env python3

import rclpy
from rclpy.node import Node
```

- rclpy 코딩 규칙

ROS 2에서 파이썬 파일을 조회하고 실행하는 과정이 있어 아래와 같이 main()부분을 항상 따로 분리하여 작성하도록 합니다.

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

1. **rclpy.init**을 통해 initialization, 즉 초기화를 하고 있습니다.
2. **node = Node('node_name')** : Node를 생성하는 부분으로 앞서 import한 Node class를 사용하고 있습니다. 매개변수로 node의 이름이 들어갑니다.
3. **node.destroy_node()** : Node를 생성하고 원하는 작업을 모두 수행했다면, 이제 사용했던 Node를 제거해야 할 것입니다. 그래야 불필요한 자원의 낭비를 줄일 수 있겠지요.
4. **rclpy.shutdown()** : 이번 예제의 제일 첫 부분에 rclpy.init을 통하여 초기화를 해주었습니다. 이제 rclpy를 통한 작업이 모두 끝났으므로 안전하게 종료시켜줍니다.

위 과정이 Python에서 rclpy를 통해 Node를 다루는 기본 절차입니다.

{{% notice note %}}
1과 4, 2와 3이 짝꿍처럼 보이지요?
{{% /notice %}}

### setup.py 수정

> 파이썬 파일을 `ros2 run` 으로 실행하기 위해서 패키지 내 **setup.py** 파일에 **entry_points**를 추가해 주어야 합니다.

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

작성하는 방법은 다음과 같습니다. ⇒ `실행 시 사용될 이름 = <패키지 이름>.<파일 이름>.main`

### example 2 - timer

- 로봇은 실행된 이후 계속해서 작업을 수행해야 하기에 주기적으로 무언가를 실행하는 일이 잦습니다. 이를 구현하는 **Timer**를 살펴봅시다.

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

timer를 생성하기 위해서 **crros2 run basic_topic_pkg example_node_3eate_timer** 함수가 사용됩니다.

- **timer_period_sec** : 실행 주기 (초)
- **callback** : 해당 주기마다 실행될 함수

![lec3_0.png](/kr/ros2_basic_foxy/images3/lec3_0.png?height=250px)

image from : [docs.ros2.org](https://docs.ros2.org/latest/api/rclpy/api/node.html)

### example 3 - spin_once, spin

> Node의 상태를 살피면서 반복 실행시키는 **spin** 함수에 대해 좀 더 자세하게 살펴봅니다.

- 예제 실행

```python
$ ros2 run py_node_tutorial example_node_3
==== Hello ROS 2 : 1====
==== Hello ROS 2 : 2====
==== Hello ROS 2 : 3====
...
```

- 주요 코드를 분석해 보겠습니다.

Node는 상태를 지속 유지하면서 변경된 내용에 따라 지정된 동작을 수행해야 합니다. 이는 로봇 프로그램에서 매우 보편적인 작업으로, ROS 2에서는 **spin()**이라는 이름의 함수로 기능을 제공하고 있습니다.

```python
def main(args=None):
    """Do enter into this main function first."""
    rclpy.init(args=args)

    node = Node('node_name')
    node.create_timer(0.2, timer_callback)

    while True:
        rclpy.spin_once(node, timeout_sec=10)

    node.destroy_node()

    rclpy.shutdown()
```

spin을 비롯하여 **spin_once**, **spin_until_future_complete**와 같이 프로그램의 실행을 관리하기 위한 다양한 추가 함수들이 존재합니다.

- timer_callback과 OOP의 필요성

```python
def timer_callback():
    """Timer will run this function periodically."""
    global count
    count += 1
    print(f'==== Hello ROS 2 : {count}====')
    # How can I use logger without globalization ?
    # node.get_logger().info('\n==== Hello ROS 2 ====')
```

callback 함수가 사용되면 필연적으로 두 함수 간 공유되는 count와 같은 자원이 생기며, 이 count를 다루면서 예기치 못한 실수가 발생할 수 있습니다.

지금은 모두 전역 변수로 작업하고 있었는데, 이것을 어떻게 효율적으로 처리할 수 있을까요?

### example 4 - OOP Node

- 예제 실행의 결과는 이전과 같습니다. 하지만 구현에서 차이를 갖습니다.

```python
$ ros2 run py_node_tutorial example_node_5
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

ROS 1과 달리, ROS 2의 OOP 구현은 **Node**를 상속받습니다. (때문에 생성 시, Node이름을 `super().__init__()`안에 넣어주어야 합니다.)

{{% notice tip %}}
이렇게 객체지향을 사용하면 Node의 기능들을 적극 활용하여 더욱 쉽고 강력한 ROS 2 개발이 가능해집니다. 앞으로의 예시에서는 모두 객체 지향을 사용하겠습니다.
{{% /notice %}}

- **rclpy logger**

```python
    super().__init__('node_name')
		...
    node.get_logger().info('\n==== Hello ROS 2 ====')
```

rospy.loginfo()와 같이 rclpy에서도 **get_logger**라는 logging API를 제공합니다. 다만, rclpy의 logger는 Node에 종속되는 개념입니다. (ROS 2에서는 여러 Node가 하나의 프로세스 안에서 실행될 수 있기 때문입니다.)

get_logger()를 사용하면 일반적인 print 콘솔 출력과는 달리, 실행중인 Node이름, 시간, 위험성 등을 디버깅할 수 있어 이후 복잡한 시스템에서 큰 도움이 됩니다.

### **example 5 - Logger Level**

- 기본 node 프로그래밍의 마지막 예시입니다.

```python
$ ros2 run py_node_tutorial example_node_5
[INFO] [1657348108.163389800] [node_name]: ==== Hello ROS 2 : 1====
[WARN] [1657348108.163810900] [node_name]: ==== Hello ROS 2 : 1====
[ERROR] [1657348108.164126200] [node_name]: ==== Hello ROS 2 : 1====
[FATAL] [1657348108.164514300] [node_name]: ==== Hello ROS 2 : 1====
...
```

![py_node_logger.gif](/kr/ros2_basic_foxy/images3/py_node_logger.gif?height=300px)

ROS 1에서와 유사하게 ROS 2에서도 위험도에 따라서 다른 logger level을 적용할 수 있습니다.

info를 기준으로 아래로 갈수록 높은 레벨의 log이며, 제일 심각한 error와 fatal의 경우, 콘솔 출력시에도 빨간 글씨로 보이는 것을 확인할 수 있습니다.

![lec3_1.png](/kr/ros2_basic_foxy/images3/lec3_1.png?height=200px)

- image from : [51CTO](https://blog.51cto.com/u_7784550/5675313)

debug의 경우 실제 콘솔 출력으로는 나타나지 않으며, 효과적인 Tracking을 위해 ROS 1 강의에서 배운 **rqt console** 사용을 권장합니다.

![lec3_2.png](/kr/ros2_basic_foxy/images3/lec3_2.png?height=300px)

### ROS 2 Parameter

> ROS 1에서와 마찬가지로, ROS 2에서도 각종 매개변수를 다룰 수 있는 커멘드 명령어와 코드 API를 제공합니다.

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

param_ex_node에서 5종류의 매개변수가 선언되었습니다. 이들을 확인하는 커멘드 라인을 배워봅시다.

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

- ros2 param get/set - node 이름과 매개변수 이름을 모두 필요로 함에 주의합니다.

```bash
$ ros2 param get /param_ex_node arr_param
Integer values are: array('q', [1, 2, 3])

$ ros2 param set /param_ex_node arr_param '[1,2,3,4]'
Set parameter successful

$ ros2 param get /param_ex_node arr_param
Integer values are: array('q', [1, 2, 3, 4])
```

이제, 파이썬 코드를 분석해봅시다.

- parameter의 생성은 node 내에서 이루어지며, **declare_parameter**를 통해 생성합니다. 함수의 두번째 인자는 기본값입니다.

**nested_param**과 같이, parameter는 계층 구조를 가질 수 있으며 . 을 통해 구분할 수 있습니다. string_param이라는 이름을 가진 parameter가 두 종류 존재하지만 서로 소속된 계층이 달라 공존할 수 있는 것입니다.

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

- 선언된 매개변수의 값은 **get_parameter**를 통해 확인 가능합니다. get_parameter 자체는 Object이고, value 속성이 실제 값을 갖고 있습니다.

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

- parameter는 launch file에서도 설정할 수 있습니다. - Hello, 112로 변경된 값을 확인해봅시다.

```bash
$ ros2 launch py_param_tutorial launch_with_param.launch.py
...
[param_example-1] [INFO] [1672387864.135213913] [param_example]:
[param_example-1] string_param: Hello
[param_example-1] int_param: 112
[param_example-1] float_param: 3.1415
[param_example-1] arr_param: [1, 2, 3]
```

- launch file의 parameters 옵션을 사용하여 이러한 작업이 가능합니다.

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

- parameter가 매우 많은 경우에는 ROS 1에서와 같이 yaml 파일을 사용해 관리할 수 있습니다. launch file의 주석된 부분을 해제하고 다시 실행해봅시다.

```python
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
```

모든 매개변수들이 변경된 것을 확인 가능합니다.

```python
$ ros2 launch py_param_tutorial launch_with_param.launch.py
...
[param_example-1] [INFO] [1672391557.995024614] [param_example]:
[param_example-1] string_param: Yaml Yaml
[param_example-1] int_param: 5
[param_example-1] float_param: 3.14
[param_example-1] arr_param: ['I', 'love', 'ROS 2']
[param_example-1] nested_param.string_param: Ooh Wee
```

- yaml 파일은 config/params.yaml에 위치하고 있습니다. parameter 관리 용도로 사용하기 위해서 yaml 파일은 일정한 규칙을 갖춰야 합니다.

```yaml
param_example:
  ros__parameters:
    string_param: "Yaml Yaml"
    int_param: 5
    float_param: 3.14
    arr_param: ['I', 'love', 'ROS 2']
    nested_param:
      string_param: "Ooh Wee"

<node-name>:
	ros__parameters:
		<param-name>: <param-value>
		...
		<nested-layer-name>:
			<param-name>: <param-value>
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

```python
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
```

> 아래 예시에서 보이듯이 로봇의 초기 속도, 최대/최소 값들, 하드웨어와 관련된 튜닝값 등 수많은 매개변수들이 사용되며 모두 지금 배운 parameter를 사용하게 됩니다.

[https://github.com/ros-planning/navigation2/blob/main/nav2_bt_navigator/src/bt_navigator.cpp](https://github.com/ros-planning/navigation2/blob/main/nav2_bt_navigator/src/bt_navigator.cpp)

---
