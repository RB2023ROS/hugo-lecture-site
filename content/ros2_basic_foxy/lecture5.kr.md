---
title: "Lecture5 - ROS 2 Topic and Examples"
date: 2023-01-02T12:39:05+09:00
draft: false
---

> Topic에 대해 배워보기 전에, 실습 시스템을 구축해봅시다.

### SRC Gazebo

> 이번 ROS 2 강의의 실습들은 Road Balance의 차량형 로봇 SRC를 통해 진행하고자 합니다. 실습을 진행하기 전 Gazebo 환경을 함께 세팅해보겠습니다.

- src_gazebo 실행 세팅 - 다소 번거로운 작업이므로 gif를 보시면서 차근차근 잘 따라와주세요!

```bash
cd ~/ros2_ws/src/du2023-ros2
./setup_scripts.sh

# CMakeLists.txt 수정 (주석)
colcon build --packages-select src_gazebo
source install/local_setup.bash

# CMakeLists.txt 수정 (주석 해제)
colcon build --packages-select src_gazebo

colcon build --packages-select src_controller
source install/local_setup.bash
```

![src_urdf.gif](/kr/ros2_basic_foxy/images5/src_urdf.gif?height=500px)

- empty world 실행

```python
ros2 launch src_gazebo empty_world.launch.py
```

- 빈 gazebo 환경에 차량형 로봇이 등장할 것입니다. 함께 등장하는 **rqt_robot_steering**을 통해 로봇을 움직여보세요.

![src_turn.gif](/kr/ros2_basic_foxy/images5/src_turn.gif?height=400px)

- 실행 중 발생할 수 있는 오류와 해결방법을 제시드립니다. 아래와 같은 에러 발생 시 `sudo apt dist-upgrade`를 입력하면 됩니다.

![lec5_0.png](/kr/ros2_basic_foxy/images5/lec5_0.png?height=100px)

> 로봇에 부착되어 있는 센서 데이터들과 제어 데이터를 **ros2 topic list**로 확인해봅시다.

- **/cmd_vel** : Twist msg를 통해 로봇을 제어하는 topic
- **/imu/data** : imu 데이터
- **/scan** : 2D lidar
- **/logi_camera_sensor/image_raw** : 전방 카메라 데이터

### ROS 2 Topic

> 강의를 열심히 따라오면서 ROS 개념을 잊어버렸을 수도 있으므로 간단히 개념 복습을 해보겠습니다.

- Topic은 Node들 사이에 데이터(Message)가 오가는 길(Bus)의 이름이며, 적절한 이름으로 데이터를 송수신하지 않으면 원하는 동작을 수행할 수 없습니다.

![topic.gif](/kr/ros2_basic_foxy/images5/topic.gif?height=300px)

image from : [docs.ros.org](https://docs.ros.org/en/foxy/Tutorials/Topics/Understanding-ROS2-Topics.html)

- Topic의 중요한 특징으로 Topic은 여러 Node들로부터 데이터를 받을 수 있고, 전송 시에도 여러 Node들에게 전송이 가능한 방식입니다.

![topic2.gif](/kr/ros2_basic_foxy/images5/topic2.gif?height=300px)

- image from : [docs.ros.org](https://docs.ros.org/en/foxy/Tutorials/Topics/Understanding-ROS2-Topics.html)

> topic을 통해 데이터가 전달되는 과정은 다음과 같습니다.

- 데이터를 보내는 주체인 **publisher**는 데이터를 받는 주체, **subscriber**를 지정한 뒤, topic을 통해 원하는 정보를 전달합니다. 이것을 **topic publish**라고 부르지요.
- 반대로, 데이터를 받는 주체인 **subscriber** 입장에선 topic을 통해 데이터를 받게 되며 이것은 **topic subscribe**라고 불립니다.

### ROS 2 Topic msg

> 로봇 프로그래밍 시 센서, 제어 데이터를 비롯하여 다양한 유형의 데이터들이 Topic을 통해 오고 갑니다. ROS에서는 이러한 데이터 형식을 **msg - Message**라는 형태로 제공하며, 원하는 데이터에 적합한 topic msg를 사용하면 더욱 효율적인 로봇 프로그래밍이 가능해집니다.

![lec5_1.png](/kr/ros2_basic_foxy/images5/lec5_1.png?height=300px)

- image from : [ethz.ch](https://ethz.ch/content/dam/ethz/special-interest/mavt/robotics-n-intelligent-systems/rsl-dam/ROS2021/lec4/ROS%20Course%20Slides%20Course%204.pdf)

geometry_msgs/Twist와 같은 message는 ROS 2에서도 동일하게 사용할 수 있습니다. 다만, ROS 2에서는 msg/srv/action 과 같이 필드의 구분이 하나 더 추가됩니다.

- **ROS 1 : geometry_msgs/Twist**
- **ROS 2 : geometry_msgs/msg/Twist**

![lec5_2.png](/kr/ros2_basic_foxy/images5/lec5_2.png?height=300px)

- 출처 : [http://docs.ros.org/](http://docs.ros.org/en/melodic/api/geometry_msgs/html/msg/Twist.html)

특정 message가 어떻게 구성되어 있는지 알고 싶을 때는 다음과 같은 커멘드 라인을 사용합니다.

```bash
$ ros2 interface show geometry_msgs/msg/Twist
# This expresses velocity in free space broken into its linear and angular parts.

Vector3  linear
Vector3  angular
```

{{% notice note %}}
혹은 앞서 제가 살펴본 것 처럼 검색을 해 보아도 됩니다.
{{% /notice %}}

ROS 2로 넘어오면서, topic 커멘드들은 다음과 같이 살짝 바뀌었습니다.

- **rostopic list ⇒ ros2 topic list**
- **rostopic info ⇒ ros2 topic info**
- **rostopic echo ⇒ ros2 topic echo**

### ROS 2 Topic Tools

> ROS 2에서도 GUI툴 RQT를 사용할 수 있습니다. Topic과 관련된 rqt tools들을 사용해봅시다.

{{% notice note %}}
ROS 1과 ROS 2의 rqt는 다른 프로그램이며 ROS 1의 plugin과 ROS 2의 plugin이 완전 동일하지는 않습니다.
{{% /notice %}}

- 실습을 위해 gazebo와 rqt를 실행합니다.

```bash
# Terminal 1
$ ros2 launch src_gazebo empty_world.launch.py
# Terminal 2
$ rqt
```

- rqt 상단의 메뉴바에서 **Plugins ⇒ Topics ⇒ Topic Monitor**를 클릭합니다.
  ![lec5_3.png](/kr/ros2_basic_foxy/images5/lec5_3.png?height=300px)

- topic monitor를 통해 현재 오가고 있는 topic들을 한눈에 볼 수 있습니다.
  ![lec5_4.png](/kr/ros2_basic_foxy/images5/lec5_4.png?height=300px)

- 이번에는 rqt 상단의 메뉴바에서 **Plugins ⇒ Topics ⇒ Message Publisher**를 클릭하고, **+ 버튼**을 눌러 publish를 원하는 topic을 추가합니다.

![lec5_5.png](/kr/ros2_basic_foxy/images5/lec5_5.png?height=300px)

- **Topic Publisher**는 별도의 작업 없이 msg에 원하는 값을 채워 topic publish가 가능하게 해주는 툴입니다. 최종 publish를 위해 체크박스를 클릭하고, 원하는 값을 채워넣습니다.

### ROS Bridge

> 강의노트를 작성하고 있는 2023년 지금도 ROS 1으로 개발된 시스템에 종속성을 가진 기업들과 프로젝트들이 많습니다. ROS 1과 ROS 2를 같이 사용할 수는 없을까요? **ros1 bridge**를 실습해봅시다.

- 일전, ROS 1에서 사용하던 smb gazebo를 오랜만에 실행시켜봅시다.

```bash
roslaunch smb_gazebo smb_gazebo.launch
```

![lec5_6.png](/kr/ros2_basic_foxy/images5/lec5_6.png?height=300px)

- **ros1 bridge**를 통해 ROS 2 시스템에서 로봇을 제어해보겠습니다. 새로운 터미널을 열고, 초기 옵션 선택 시 3번을 선택하면 ros1 bridge가 실행됩니다. (이 동작은 사실 dynamic_bridge를 실행시키는 것입니다.)

```bash
sudo apt install ros-foxy-ros1-bridge -y

# Terminal 1 - ros bridge
ros2 run ros1_bridge dynamic_bridge

# Terminal 2 - ros2 foxy
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

- ROS 2로 설정된 터미널에서 ROS 1으로 구동되고 있는 로봇을 조종할 수 있습니다.

![ros_bridge.gif](/kr/ros2_basic_foxy/images5/ros_bridge.gif?height=400px)

- ros bridge가 있으니 ROS 2 개발을 굳이 하지 않아도 된다고 생각할 수 있습니다. 하지만 보안, 지연과 같은 ROS 1의 고질적인 문제들은 여전히 남아있게 되므로 프로젝트를 시작하는 단계라면, ROS 2로 모든 개발을 진행하시길 추천드립니다.

![lec5_7.png](/kr/ros2_basic_foxy/images5/lec5_7.png?height=200px)

- image from : [swri.org](https://www.swri.org/industry/industrial-robotics-automation/blog/the-ros-1-vs-ros-2-transition)

{{% notice tip %}}
참고로, `/opt/ros/<ros-version>/setup.bash`는 ROS 시스템을 사용하기 위해서 필요한 설정이 담긴 파일입니다. 혹여나 galactic, humble과 같이 최신 버전을 사용하고 싶을 때 참고하시기 바랍니다.
{{% /notice %}}

> ROS 1과 ROS 2를 모두 관리하기에는 많은 노력이 필요하고, 원하는 기능이 모두 동작하지 않을 수 있습니다. 이전 버전 legacy가 있는 경우, 잘 판단하여 시스템을 유지/보수합시다.

### My Gazebo World

> Topic의 응용으로 Gazebo를 사용하여 나만의 장애물을 만들고, 충돌을 회피하는 코드를 작성해보고자 합니다. 이를 위해 Gazebo의 Building Editor 사용법을 알려드리겠습니다.

- gazebo를 실행시킨 뒤, 상단 Edit 옵션에서 Building Editor를 실행합니다.

![gz_building_editor.gif](/kr/ros2_basic_foxy/images5/gz_building_editor.gif?height=400px)

- 상단 모눈종이에 스케치를 통해 건물 벽을 생성할 수 있으며, 하단 view를 통해 실시간으로 업데이트되는 건물을 확인할 수 있습니다. 혹여 실수를 했거나, 보다 정확한 수치를 입력하고 싶은 경우, 모눈종이 위의 검은 선을 더블 클릭하면 사진과 같이 구체적인 설정을 변경할 수 있는 탭이 등장합니다.

![lec5_8.png](/kr/ros2_basic_foxy/images5/lec5_8.png?height=300px)

- 수정이 완료되었다면, 왼쪽 탭에서 door, stairs도 사용해보고, Texture도 입혀봅시다.

![lec5_9.png](/kr/ros2_basic_foxy/images5/lec5_9.png?height=300px)

- 모든 작업을 마친 뒤, File 탭에서 Save As를 선택하여 완성한 물체를 저장합니다. (저는 wall 이라는 이름으로 저장해보겠습니다.)

![gz_save_model.gif](/kr/ros2_basic_foxy/images5/gz_save_model.gif?height=400px)

- 저장된 폴더 내부는 다음과 같은 구조를 갖게 됩니다.

```python
├─  wall
   ├── model.config
   └── model.sdf
```

- **model.config** : 해당 model의 이름, 작성자 등 기본적인 정보들이 기입됩니다.
- **model.sdf** : 실직적인 sdf 형식의 모델이 위치합니다.

작성한 물체를 Gazebo에서 두고두고 사용하는 몇가지 방법들이 있습니다.

1. GAZEBO_MODEL_PATH에 추가하기 ⇒ **~/.gazebo/gui.ini** 파일 내 model_paths에 building 폴더에 절대 경로를 추가합니다.

```python
$ gedit ~/.gazebo/gui.ini

[geometry]
x=0
y=0
[model_paths]
filenames=/home/kimsooyoung/<your-folder-location>
```

2. launch file에서 GAZEBO_MODEL_PATH 수정하기 ⇒ 대신 이렇게 하면 해당 launch file 사용 시에만 GAZEBO_MODEL_PATH가 반영됩니다.

```python
if 'GAZEBO_MODEL_PATH' in os.environ:
    os.environ['GAZEBO_MODEL_PATH'] += ":" + gazebo_model_path
else:
    os.environ['GAZEBO_MODEL_PATH'] = gazebo_model_path
```

- 이제 Gazebo를 실행시키면, 사진과 같이 Insert 부분에 building이 추가되어있음을 확인 가능합니다. 이를 클릭 후 gazebo 환경으로 커서를 옮기면 원하는 위치에 wall을 위치시킬 수 있습니다.
  ![lec5_10.png](/kr/ros2_basic_foxy/images5/lec5_10.png?height=300px)

- wall이 추가된 world는 별도로 저장해줘야 합니다. **File ⇒ Save World As**를 클릭하면 완성된 world 파일을 저장합시다.
  ![lec5_11.png](/kr/ros2_basic_foxy/images5/lec5_11.png?height=200px)

> 생성한 world 파일을 launch 파일로 실행해봅시다.

- 저장해두었던 Gazebo world 파일을 src_gazebo의 worlds 폴더에 위치시킵니다.
  ![lec5_12.png](/kr/ros2_basic_foxy/images5/lec5_12.png?height=300px)

- launch 파일을 수정하여 새로운 world 파일의 위치를 전달합니다.

```python
def generate_launch_description():
		...

    # gazebo
    pkg_gazebo_ros = FindPackageShare(package='gazebo_ros').find('gazebo_ros')
    pkg_path = os.path.join(get_package_share_directory('src_gazebo'))
    world_path = os.path.join(pkg_path, 'worlds', '여러분의-world-file.world')
		...

		# Start Gazebo server
		start_gazebo_server_cmd = IncludeLaunchDescription(
		    PythonLaunchDescriptionSource(os.path.join(pkg_gazebo_ros, 'launch', 'gzserver.launch.py')),
		    launch_arguments={'world': world_path}.items()
		)
```

- package를 빌드하고, gazebo를 다시 실행해봅시다.

```python
colcon build --packages-select src_gazebo
source install/local_setup.bash

ros2 launch src_gazebo empty_world.launch.py
```

- 제작한 world와 로봇이 함께 등장하는 것을 확인할 수 있습니다.

![lec5_13.png](/kr/ros2_basic_foxy/images5/lec5_13.png?height=300px)

{{% notice note %}}
최대한 여러분들만의 장애물을 만들어보시고, 혹시나 이 과정에서 도저히 모르겠는 오류가 발생했거나, 시간이 없는 경우, 우선 제가 제작한 world와 launch file을 사용하도록 합니다. ⇒ `ros2 launch src_gazebo wall_world.launch.py`
{{% /notice %}}

## ROS 2 Topic 프로그래밍

### example1 - cmd_vel publish

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

- 로봇이 원을 그리며 움직이기 시작합니다. ROS 1 강의를 완료하였다면 어떻게 구현하였을지 아시겠지요?

![circle_src.gif](/kr/ros2_basic_foxy/images5/circle_src.gif?height=400px)

- 코드를 살펴보겠습니다 - 시작으로, 필요한 파이썬 패키지들을 import 합니다. ROS 2에서 파이썬 프로그래밍을 하기 위해서는 rclpy를 import 해야 합니다. 더불어, **Twist**를 import하는 문법도 눈여겨봅시다.

```python
import random

from geometry_msgs.msg import Twist
import rclpy
from rclpy.node import Node
```

- 핵심이 되는 **TwistPubNode** 클래스의 생성자부터 살펴보겠습니다.

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

**create_publisher**는 topic publisher를 생성하는 함수로 3개의 매개변수를 받으며 각각에 대한 설명은 다음과 같습니다.

- **message type** : Topic 통신에 사용될 msg Type 입니다.
- **topic name** : 사용할 Topic 이름을 지정합니다. (이 이름을 잘못 설정하면 존재하지 않는 topic에 Publish하는 오류 상황이 발생하니 주의합니다.)
- **queue_size** : 대기열의 크기라고 이해하시면 좋습니다.
- **timer callback**에서 publish가 이루어집니다. 생성한 publisher에서 publish 함수를 사용하며 매개변수로 message가 전달됩니다.

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

### example2 - scan subscription

- 이번 예시는 라이다 데이터를 다뤄보고자 합니다. 명령어를 실행해봅시다.

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

예시를 실행시킨 상황에서 Gazebo 상의 로봇 주변으로 물체를 등장시켜봅시다.

![lec5_14.png](/kr/ros2_basic_foxy/images5/lec5_14.png?height=300px)

scan data의 ranges는 사진과 같은 거리 데이터를 담고 있답니다.

![lec5_15.png](/kr/ros2_basic_foxy/images5/lec5_15.png?height=300px)

- 코드를 살펴봅시다. 이번에는 [sensor_msgs/msg/LaserScan](http://docs.ros.org/en/melodic/api/sensor_msgs/html/msg/LaserScan.html)이 사용되었습니다.

```python
import rclpy
from rclpy.node import Node

from sensor_msgs.msg import LaserScan
```

- ScanSubNode 클래스의 생성자에서는 scan topic의 subscriber가 생성됩니다.

```python
class ScanSubNode(Node):

    def __init__(self):

        super().__init__('scan_sub_node')

        queue_size = 10  # Queue Size

        self.pose_subscriber = self.create_subscription(
            LaserScan, 'scan', self.sub_callback, queue_size
        )
```

**create_subscription**은 4개의 매개변수를 받으며 각각에 대한 설명은 다음과 같습니다.

- **Message Type** : Topic 통신에 사용될 msg Type 입니다.
- **topic name** : 데이터를 Subscribe할 Topic의 이름을 지정합니다. (이 이름을 잘못 설정하면 아무것도 publish하고 있지 않는 topic을 마냥 기다리고 있게 되는 상황이 발생할 것입니다.)
- **subscribe callback** : 데이터가 전달될 때마다 실행되는 callback 함수로, 전달된 데이터를 통해 실행할 작업이 이 함수 안에 구현됩니다.
- **queue_size** : create_publisher때와 마찬가지로, 대기열의 크기라고 이해하시면 좋습니다.

sub_callback의 첫번쨰 매개변수는 항상 topic message data가 됩니다. 해당 message에서 원하는 데이터만 추출한 다음, logger를 통해 콘솔 출력을 진행합니다.

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

### example3 - parking

- 이번 예시를 실행하기 전에, empty_world.launch.py를 수정해야 합니다.

```bash
    # gazebo
    pkg_gazebo_ros = FindPackageShare(package='gazebo_ros').find('gazebo_ros')
    pkg_path = os.path.join(get_package_share_directory('src_gazebo'))
    # world_path = os.path.join(pkg_path, 'worlds', 'empty_world.world')
    world_path = os.path.join(pkg_path, 'worlds', 'wall_world.world')
```

- 물체와 충돌하기 전까지 로봇을 움직이다가 일정 거리 내 벽이 검출되면 자동으로 멈추는 예시입니다.

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

![parking.gif](/kr/ros2_basic_foxy/images5/parking.gif?height=400px)

이 기능을 구현하기 위해서는 Topic Publish와 Subscribe가 모두 필요합니다.

- **/cmd_vel topic publish**
- **/scan topic subscribe**

⇒ 따라서, Node의 생성자에서도 Publisher와 Subscriber를 모두 생성합니다.

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

- **sub_callback**에서는 전방 (msg.ranges[60]) 물체와의 거리를 탐지하고, 이 거리가 0.5m 이하가 되면 정지 topic을 publish 합니다.

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

### CV 2 and ROS 2

> Topic과 ROS 2 Bag을 사용한 예시를 실행해보고자 합니다.

이번 실습을 위해 제가 미리 준비한 데이터셋을 제공드리니, 기억하기 쉬운 위치에 다운받아 주세요.

📁 [quadrupped_train.zip](https://drive.google.com/file/d/1Ue_mDFaPoKW2LA8mD9eGFAeJD5IVc3K-/view?usp=sharing)

{{% notice tip %}}
Windows + WSL2 유저의 경우 `explorer.exe` 명령어를 통해 파일 탐색기를 열 수 있습니다.
{{% /notice %}}

- 준비된 예시를 실행해봅시다.

```bash
# 예제 실행 준비
sudo apt install ros-foxy-rosbag2
cd ~/ros2_ws
colcon build --packages-select py_cv_tutorial
source install/local_setup.bash

# 터미널 1 – 데이터 위치로 이동하여 ros2 bag 실행
cd <데이터를 저장한 위치>
ros2 bag play quadrupped_train.bag_0.db3

# 터미널 2 – image_view 실행
ros2 run rqt_image_view rqt_image_view
```

- 사진과 같이 기차 선로 모습이 보인다면 성공입니다.

![lec5_16.png](/kr/ros2_basic_foxy/images5/lec5_16.png?height=300px)

해당 데이터셋은 **기차 선로에서 주행하는 4족 보행 로봇**으로부터 취득한 것입니다. 기차 선로는 주기적으로 관리가 필요하며, 길고 긴 선로를 로봇이 자동으로 검사해준다면 아주 유용할 것입니다. 이러한 상상을 해보면서 실습에 임해봅시다.

![lec5_17.png](/kr/ros2_basic_foxy/images5/lec5_17.png?height=300px)

### Example1 - CV Bridge

> 이미지를 다루기 위한 오픈소스 라이브러리인 **OpenCV**를 사용해보려 합니다. 준비된 예시와, ros2 bag 파일을 실행하면 cv2.imshow를 통해 연속된 이미지를 확인할 수 있습니다.

```bash
# 터미널 1
$ cd <bag 파일 위치>
$ ros2 bag play quadruped_train/quadruped_train.bag_0.db3
[INFO] [1667077909.362223100] [rosbag2_storage]: Opened database 'quadrupped_train/quadrupped_train.bag_0.db3' for READ_ONLY.

# 터미널 2
$ ros2 run py_cv_tutorial img_sub
[INFO] [1667077909.613586300] [image_subscriber]: Receiving video frame
[INFO] [1667077910.934147000] [image_subscriber]: Receiving video frame
...
```

- OpenCV에서는 CV::Mat 이라는 특정 타입을 사용합니다. 이를 ROS 2의 Topic Message로 변환하기 위해서 **CV Bridge**라는 것을 사용합니다.

```python
from cv_bridge import CvBridge # Package to convert between ROS and OpenCV Images
import cv2 # OpenCV library

class ImageSubscriber(Node):

    def __init__(self):

        # Used to convert between ROS and OpenCV images
        self.br = CvBridge()
				...

		def listener_callback(self, data):
				...
        # Convert ROS Image message to OpenCV image
        current_frame = self.br.imgmsg_to_cv2(data, "bgr8")
        edge_frame = self.hough_transform(current_frame)
```

- imgmsg_to_cv2를 통해 CV::Mat으로 변환된 이미지 데이터를 OpenCV의 다양한 기능들과 함께 사용할 수 있습니다. 코드의 imshow 부분을 바꿔서 이미지 처리를 적용해 봅시다. 직선을 검출하는 알고리즘이며, 매개변수의 최적화를 통해 선로를 인지할 수 있습니다.

![lec5_18.png](/kr/ros2_basic_foxy/images5/lec5_18.png?height=300px)

직선 검출을 위해 사용된 OpenCV 기능들은 아래와 같습니다. 로직을 업그레이드하여 여러분만의 선로 검출 알고리즘을 만들어 보세요!

- [cv2.GaussianBlur](https://opencv-python.readthedocs.io/en/latest/doc/11.imageSmoothing/imageSmoothing.html)
- [cv2.fillPoly](https://www.geeksforgeeks.org/draw-a-filled-polygon-using-the-opencv-function-fillpoly/) / [cv2.bitwise_and](https://opencv-python.readthedocs.io/en/latest/doc/07.imageArithmetic/imageArithmetic.html) ROI 설정
- [cv2.HoughLinesP](https://opencv-python.readthedocs.io/en/latest/doc/25.imageHoughLineTransform/imageHoughLineTransform.html)

### rosbag to img

> 컴퓨터 비전만으로 완벽한 선로 인식을 구현하기는 너무나 힘듭니다. 딥러닝을 사용해서 이를 극복할 수 있을 것이며, 데이터셋의 제작을 위해 rosbag 데이터에서 이미지를 추출하는 예시를 준비하였습니다.

```bash
$ ros2 run py_cv_tutorial rosbag2_to_timedimg
/home/kimsooyoung/djhrd_ws/quadrupped_train/quadrupped_train.bag_0.db3
saved: color_1666796292992515592.png
saved: color_1666796293035428479.png
saved: color_1666796293079139778.png
saved: color_1666796293120768031.png
```

- 예제 실행 전 main 함수에서 bag 파일의 위치를 여러분의 것으로 수정해야 합니다.

```python
def main(args=None):

    # Change below roots to your ros2 bag locations
    ROOT_DIR = "/home/kimsooyoung/djhrd_ws/quadrupped_train"
    FILENAME = "/quadrupped_train.bag_0.db3"
    DESCRIPTION = "color_"
```

- 예제 실행 후 사진과 같이 기차 생성된 이미지들이 보인다면 성공입니다.

![lec5_19.png](/kr/ros2_basic_foxy/images5/lec5_19.png?height=300px)
