---
title: "Lecture7 - Useful ROS 2 Example, Navigation2"
date: 2023-01-02T12:39:02+09:00
draft: false
---

> 코딩을 시작하기 전에 구현해야 하는 기능들과 Topic, Service 기준 input, output을 정리해봅시다.

- **Part 1 - 감옥 구현**

거북이가 일정 범위를 벗어나게 되면 다시 원위치로 돌아오게 한다.

1. 거북이의 위치는 **/turtle1/pose topic subscribe**을 통해 얻을 수 있습니다.
2. 거북이가 일정 위치를 벗어나는 순간, 원위치로 돌아오게 하는 **/turtle1/teleport_absolute** service call을 해야 합니다. 이를 위한 **service client**가 필요합니다.

- **Part 2 - 감옥 크기 변경**

사용자로부터 감옥의 크기를 변경해달라는 service request가 오면 기존 감옥의 크기를 변경하는 작업이 필요하다.

1. **/turtle_jail_size service server**를 구현해야 합니다.
2. 감옥의 크기가 변경되면 topic 로직에도 반영되어야 하므로 **OOP**의 형태로 구현이 필요할 듯 합니다.

코드를 구현하기 위해 필요한 topic, service interface들을 조회합시다.

```bash
$ ros2 interface show custom_interfaces/srv/TurtleJail
float32 width
float32 height
---
bool success

$ ros2 interface show turtlesim/
srv/TeleportAbsolute
float32 x
float32 y
float32 theta
---

$ ros2 interface show turtlesim/msg/Pose
float32 x
float32 y
float32 theta

float32 linear_velocity
float32 angular_velocity
```

모든 필요조건들을 알게 되었습니다. 이제 프로그래밍을 해보겠습니다.

- 필요 파이썬 패키지 import

```python
import rclpy
from rclpy.node import Node
from turtlesim.msg import Pose
from turtlesim.srv import TeleportAbsolute

from custom_interfaces.srv import TurtleJail
```

- **Service Server, Service Client, Topic Subscriber** 생성

```python
# Create Turtle teleport client
self.client = self.create_client(TeleportAbsolute, 'turtle1/teleport_absolute')
while not self.client.wait_for_service(timeout_sec=1.0):
    self.get_logger().info('Service not available, Waiting again...')

self.request_srv = TeleportAbsolute.Request()
self.get_logger().info('=== [Service Client : Ready to Call Service Request] ===')

# Create Subscriber for turtle1/pose
queue_size = 10  # Queue Size
self.pose_subscriber = self.create_subscription(
    Pose, 'turtle1/pose', self.sub_callback, queue_size
)

# Create Service Server for User Interfaces
self.srv = self.create_service(
    TurtleJail, 'turtle_jail_size', self.turtle_jail_callback
)
self.get_logger().info('==== [Service Server : Ready to receive Service Request] ====')
```

- 클래스 변수 선언 (감옥 사이즈, 벽에 도달하기 전 거북이의 각도)

```python
# Preserve its rotation before teleport
self.cur_theta = 0.0

# jail size in rectangular form
self.jail_width = 6.0
self.jail_height = 6.0
```

> topic subscribe와 service server에 대한 **callback이 두 개 필요**합니다.

- 거북이의 Pose Topic Subscribe Callback

```python
def sub_callback(self, msg):
    """Turtle Pose Subscriber Callback"""

    if abs(msg.x - 6.0) > self.jail_width or abs(msg.y - 6.0) > self.jail_height:
        self.cur_theta = msg.theta
        self.get_logger().warn("You can't go out Turtle! :(")
        self.send_request()
```

- 감옥 사이즈 변경 요청 시 발생하는 Service Server Callback

```python
def turtle_jail_callback(self, request, response):
    """Service Server for jail resizing client request"""
    self.jail_width = request.width
    self.jail_height = request.height

    self.get_logger().info(f"""Jail Size Update to {self.jail_width}/{self.jail_height}""")

    response.success = True

    return response
```

- 마지막으로, 거북이를 원점으로 이동시켜달라는 service request를 구현합니다.

```python
def send_request(self):
    """Service Clinet request fuction"""
    self.request_srv.x = 6.0
    self.request_srv.y = 6.0
    self.request_srv.theta = self.cur_theta

    self.future = self.client.call_async(self.request_srv)

    return self.future
```

![turtle_jail.gif](/kr/ros2_basic_foxy/images7/turtle_jail.gif?height=300px)

> 바로 코딩을 시작하지 말고, 무엇을 구현해야 하는지 적어보는 것, 조금씩 조금씩 구현한 뒤, 확인하면서 개발하는 것, ROS 뿐만 아니라 앞으로의 프로그래밍에서 반드시 잊지 마시기 바랍니다.

## Navigation 2

> 이번 시간에는 유용한 ROS 2 프로젝트를 소개하고, 관련된 내용을 함께 살펴보고자 합니다. ROS 2의 자율 주행 메타페키지인 Navigation 2, **Nav2** 입니다.

![lec7_0.png](/kr/ros2_basic_foxy/images7/lec7_0.png?height=300px)

image from : [https://navigation.ros.org](https://navigation.ros.org/)

Nav 2는 삼성 리서치 아메리카의 **Steven Macenski**의 주도 하에 개발되고 있으며, 아래와 같은 자율주행을 위해 필요한 거의 모든 기능들을 집합해둔 프로젝트입니다.

- 지도 저장, 전송, 관리 **(Map Server)**
- 지도 상에서 로봇의 위치 파악하기 **(AMCL)**
- A to B 주행을 위한 경로 생성 **(Nav2 Planner)**
- 경로를 따라 로봇을 이동시키기 위한 컨트롤 시스템 **(Nav2 Controller)**
- 주행 경로의 최적화 **(Nav2 Smoother)**
- Costmap 생성과 센서 데이터 반영 **(Nav2 Costmap 2D)**
- Behavior Tree를 통한 복잡한 주행 시나리오 핸들링 **(Nav2 Behavior Trees and BT Navigator)**
- 주행 실패를 방지하기 위한 **recovery behaviors (Nav2 Recoveries)**
- Waypoints 주행 **(Nav2 Waypoint Follower)**
- Nav2 Node들의 모니터링과 관리 **(Nav2 Lifecycle Manager)**
- 커스텀 알고리즘 개발을 위한 Plugins **(Nav2 Core)**

일반 개발자나 기업이 자율주행을 구현하기 위해서는 이렇게 많은 것들이 필요합니다. 하지만, ROS 2를 사용한다면 Nav 2를 통해 많은 시간을 단축할 수 있지요!

이 강의에서 모든 내용을 다루고 싶지만, 시간적 한계가 있기 때문에 예시를 통해 로봇 자율주행에 대한 개념을 체감해보도록 하겠습니다.

![lec7_1.png](/kr/ros2_basic_foxy/images7/lec7_1.png?height=300px)

### Nav 2 Example 1 - SLAM_ToolBox

> 처음 가보는 환경을 맞닥뜨리면, 우리는 주변 상황을 탐색하고 내 위치를 파악하려 할 것입니다. 이것을 로보틱스에서는 **SLAM - Simultaneous localization and mapping**이라고 이야기합니다.

- **SLAM Toolbox**는 Steven Macenski에 의해 제작된 ROS 2 패키지이며 backend와 frontend SLAM 알고리즘 뿐만 아니라, ROS 2와 호환되는 관리 도구, 최적화까지 담고 있습니다.

![lec7_2.png](/kr/ros2_basic_foxy/images7/lec7_2.png?height=300px)

- 예시를 통해 SRC 로봇으로 SLAM을 실습해봅시다. 로봇을 이동시키면서 rviz에 갱신되는 지도를 확인합니다.

```bash
colcon build --packages-select src_slam
colcon build --packages-select src_odometry
source install/local_setup.bash

# Terminal 1
ros2 launch src_gazebo racecourse.launch.py use_rviz:=false
# Terminal 2
ros2 launch src_slam src_slam_gazebo_slam_toolbox.launch.py
```

![slam.gif](/kr/ros2_basic_foxy/images7/slam.gif?height=300px)

**지도를 저장하는 두 가지 방법이 있습니다.**

- rviz plugin 사용 - RViz상의 공란에 지도를 저장할 위치를 포함한 절대경로를 기입하고 왼쪽 **“Save Map”** 버튼을 눌러 최종 지도를 추출합니다.

![lec7_3.png](/kr/ros2_basic_foxy/images7/lec7_3.png?height=300px)

- 커멘드 라인 사용 - 저장 위치가 존재하는지 확인 후 실행하세요.

```bash
ros2 run nav2_map_server map_saver_cli -f <map_dir>/<map_name>
```

- 저장 결과로 두 가지 파일이 생성되며, 이러한 지도 데이터를 관리해주는 것이 Nav 2의 Map Server 입니다.

1. 지도 이미지 파일
2. 지도 정보를 담은 yaml 파일

![lec7_4.png](/kr/ros2_basic_foxy/images7/lec7_4.png?height=200px)

### Localization

> 로봇의 현재 위치와 방향을 파악하는 것을 **localization**이라고 부릅니다.

Nav2의 **AMCL** 패키지는 주어진 맵을 기반으로 Adaptive Monte Carlo Localization 방식을 사용하여 로봇의 위치를 파악합니다.

![Particle_filters.gif](/kr/ros2_basic_foxy/images7/Particle_filters.gif?height=300px)

image from : [wikipedia](https://commons.wikimedia.org/wiki/File:Particle_filters.gif)

이 Adaptive Monte Carlo Localization은 입자(Particle)를 사용하여 로봇의 위치를 파악하는 방식이며, 이 입자들은 실제 로봇처럼 주어진 가중치와 함께 그들만의 좌표와 방향 값을 가지고 있게 됩니다.

로봇이 주어진 환경에서 이동하여 새 센서 데이터를 제공할 때마다 입자가 다시 샘플링되며, 각 샘플링에서 가중치가 작은 입자는 소멸되고 가중치가 큰 입자는 더 커지면서 생존합니다.

AMCL 알고리즘을 여러 번 반복하게 되면 입자들의 위치가 수렴하여 로봇 포즈의 근사치를 평가할 수 있고, 결국 이 로봇의 최종 방향과 위치를 추정하게 됩니다.

### Nav2 Lifecycle Manager

> Map Server, AMCL과 같은 Nav2의 stack은 **Nav2 Lifecycle Manager**에 의해 관리됩니다.

- Lifecycle이라는 것은 생성과 실행, 일시정지와 최종 종료와 같이 Node의 상태를 지칭합니다.

![lec7_5.png](/kr/ros2_basic_foxy/images7/lec7_5.png?height=300px)

image from : [roscon2019](https://roscon.ros.org/2019/talks/roscon2019_navigation2_overview_final.pdf)

- 기존 ROS 2도 lifecycle을 갖고 있지만, Nav 2는 전용 lifecycle_manager를 사용함으로 다른 node과 별도로 자율주행과 관련된 최적화된 시스템 설계가 가능해졌습니다.

![lec7_6.png](/kr/ros2_basic_foxy/images7/lec7_6.png?height=300px)

image from : [navigation2](https://github.com/ros-planning/navigation2)

{{% notice note %}}
하지만, ROS 2의 lifecycle이 아닌 독자적인 방식을 사용하게 됨으로 디버깅 툴 사용이 불가하다는 점이 문제점으로 대두됩니다. 이러한 이유로, nav2 개발 시에는 **rqt console**의 사용을 권장합니다.
{{% /notice %}}

![lec7_7.png](/kr/ros2_basic_foxy/images7/lec7_7.png?height=300px)

### Costmap 2D

> 자율주행의 경로를 계산할 시, 일반적으로 구역을 나누고 장애물 여부, 충돌 가능 여부등을 점수화하여 구역별 **costmap**을 만듭니다.

Nav2에서도 로봇의 크기와 사용자의 정의에 따라 inflation을 적용한 costmap을 생성합니다.

![lec7_8.png](/kr/ros2_basic_foxy/images7/lec7_8.png?height=300px)

Nav2의 costmap에서 각 셀의 cost는 unknown, free, occupied, inflated로 정의되고, controller, planner, recovery plugin등이 이를 사용하여 안전하고 효율적인 경로를 계산하게 됩니다. 이 costmap에는 두가지 종류가 있는데

1. **Global Costmap**은 SLAM을 통해 얻은 정적 지도를 기반으로 생성되며, 벽, 가구와 같이 SLAM 당시 존재했던 장애물들에 대한 costmap입니다.
2. **Local Costmap**은 로봇 주위 일정 범위에 로봇에 부착된 센서들을 기반으로 생성되며, 동적 장애물을 감지할 수 있습니다. 로봇이 이동함에 따라 매번 갱신되는 데이터입니다.

아래 사진에서 왼쪽은 Global costmap을, 오른쪽은 Local costmap을 보이고 있습니다.

![lec7_9.png](/kr/ros2_basic_foxy/images7/lec7_9.png?height=300px)

inflation의 변경에 따른 costmap의 차이는 아래와 같이 rviz 화면에서 확인할 수 있습니다. inflation을 줄이게 되면 경로 상 갈 수 있는 영역이 많아졌지만, 충돌 측면에서는 좀 더 위험해졌다고 말할 수 있습니다. 따라서, 로봇의 크기와 최대 속도, 제동거리를 고려하여 적절한 inflation radius를 설정해야 합니다.

![lec7_10.png](/kr/ros2_basic_foxy/images7/lec7_10.png?height=300px)

### Local Path와 Global Path

> 센서 데이터에서 장애물이 검출되었다면, local planner는 이를 회피하기 위해 기존 global planner와는 다른 별도의 경로를 따르도록 로봇에게 지시할 것입니다.

Nav2에서 물체 회피를 위해 수행되는 동작들은 다음과 같습니다.

- **global planner**가 지도를 기반으로 거시적인 경로를 계산하면 이 경로의 일부가 local planner에게 전달됩니다.
- **local planner**는 기본적으로 global Planner의 파편이지만, 현재 주행중인 환경과 센서 데이터를 입력으로 받아, 경로를 최신화합니다.

global plan과 local plan은 엄연히 다른 알고리즘이 사용되는 분야입니다.

![lec7_11.png](/kr/ros2_basic_foxy/images7/lec7_11.png?height=300px)

image from : **[Autonomous Wheeled Mobile Robot Control](https://www.researchgate.net/publication/320744124_Autonomous_Wheeled_Mobile_Robot_Control)**

### Controller

> local Path가 계산되었다면, 이제 로봇이 실제로 해당 경로를 주행할 차례입니다.

로봇의 최대 선속도, 각속도와 후진 여부 등 매개변수에 따라 경로를 최적화해야 합니다. Nav2에서는 **Controller**가 이를 담당하고 있습니다.

Controller는 그 이름과 같이 주어진 경로를 따르기 위해 로봇에게 실제 제어 신호를 전달하게 되며, 따라서 사용하는 로봇의 모델과 밀접한 관련이 있습니다. (제자리 회전, y축 이동 등)

![lec7_12.png](/kr/ros2_basic_foxy/images7/lec7_12.png?height=200px)

image from : [dwa_local_planner](http://library.isr.ist.utl.pt/docs/roswiki/dwa_local_planner.html)

### Nav2 Behavior Tree

> 로봇의 일련의 행동들은 Tree 자료 구조로 표현될 수 있으며, Tree의 Node에 해당하는 작업을 구현해두고 이들을 조합하여 시나리오를 구성할 수 있습니다.

- ex) A 위치에 도착한 뒤, 물건을 싣고, 다시 B 위치로 이동하여 새로운 물건을 싣고, 최종적으로 C지점으로 돌아와라. 만약 C 지점에 물건이 가득 차있다면 D 지점으로 이동하여 물건을 최종 이송해라...

![lec7_13.png](/kr/ros2_basic_foxy/images7/lec7_13.png?height=200px)

image from : [wikipedia](<https://en.wikipedia.org/wiki/Behavior_tree_(artificial_intelligence,_robotics_and_control)>)

- Nav2는 **[BehaviorTree.CPP](https://www.behaviortree.dev/)**를 사용하여 로봇의 자율주행 시나리오 기능을 제공합니다. Behavior Tree는 Nav2와는 별도의 오픈소스 프로그램인데, 시각화를 포함한 높은 완성도를 갖추고 있어 게임 및 로봇 도메인에서 많이 사용됩니다.

![lec7_14.png](/kr/ros2_basic_foxy/images7/lec7_14.png?height=300px)

[https://robohub.org/introduction-to-behavior-trees/](https://robohub.org/introduction-to-behavior-trees/)

### Nav2 Core

> nav2_core 패키지는 custom 알고리즘을 적용한 플러그인을 구현할 수 있도록 인터페이스를 제공합니다. 연구실이나 기업에서는 자신만의 로직을 Plugin으로 구현해서 Nav2 시스템에 적용할 수 있으며, 이러한 방식으로 구현의 시간을 단축시킬 수 있습니다.

- **Global Planner** – global_planner.hpp
- **Local Planner** – local_planner.hpp
- **Recovery behaviors** – recovery.hpp
- **Goal checker** – goal_checker.hpp
- **Exceptions** – exceptions.hpp

여러분만의 plugin을 만들어보고 싶다면, 아래 링크를 참고하세요.

- [https://navigation.ros.org/plugin_tutorials/index.html](https://navigation.ros.org/plugin_tutorials/index.html)

### Nav2 실행 예시

- 지루한 이론을 듣느라 고생하셨습니다. 이제 src를 통해 자율 주행 예시를 함께 실행해보겠습니다.

```bash
colcon build --packages-select src_nav
source install/local_setup.bash

# Terminal 1
ros2 launch src_gazebo racecourse.launch.py use_rviz:=false
# Terminal 2
ros2 launch src_nav bringup_launch.py
```

![src_nav.gif](/kr/ros2_basic_foxy/images7/src_nav.gif?height=300px)

- **rqt_graph**를 확인해봅시다.

![rosgraph.png](/kr/ros2_basic_foxy/images7/rosgraph.png?height=300px)

> 방금 우리가 실행한 launch file을 같이 살펴볼까요?

- LaunchDescription부터 살펴봅시다. 대부분 launch arguement의 선언이고 결국 실행과 관련된 것은 **bringup_cmd_group** 뿐입니다.

```python
		# Set environment variables
    ld.add_action(stdout_linebuf_envvar)

    # Declare the launch options
    ld.add_action(declare_namespace_cmd)
    ld.add_action(declare_use_namespace_cmd)
    ld.add_action(declare_map_yaml_cmd)
    ld.add_action(declare_use_sim_time_cmd)
    ld.add_action(declare_slam_cmd)
    ld.add_action(declare_params_file_cmd)
    ld.add_action(declare_autostart_cmd)
    ld.add_action(declare_bt_xml_cmd)
    ld.add_action(declare_open_rviz_cmd)

    # Add the actions to launch all of the navigation nodes
    ld.add_action(bringup_cmd_group)

    return ld
```

- Launch file의 실행 시 GroupAction을 통해 관련된 실행 프로그램들을 한데 묶을 수 있습니다. 이렇게 하는 이유는, 여러 로봇의 실행을 대비한 것입니다.
- IncludeLaunchDescription을 통해 4개의 launch file들을 다시 추가하고 있습니다.

```bash
    # Specify the actions
    bringup_cmd_group = GroupAction([
        PushRosNamespace(
            condition=IfCondition(use_namespace),
            namespace=namespace),

        IncludeLaunchDescription(
            PythonLaunchDescriptionSource(os.path.join(my_launch_dir, 'slam_launch.py')),
                   ...

        IncludeLaunchDescription(
            # Run Localization only when we don't use SLAM
            PythonLaunchDescriptionSource(os.path.join(my_launch_dir, 'localization_launch.py')),
                   ...

        IncludeLaunchDescription(
            PythonLaunchDescriptionSource(os.path.join(my_launch_dir, 'navigation_launch.py')),
						       ...

        IncludeLaunchDescription(
            PythonLaunchDescriptionSource(os.path.join(my_launch_dir, 'rviz_view_foxy_launch.py')),
                   ...
        ])
```

- 각각의 하위 launch file은 개별 실행될 수 있도록 구성되었습니다. **nav2_lifecycle_manager**가 위치하고 있는 점도 확인 가능하고 node를 전달하는 방법도 알 수 있습니다.

```python
        Node(
            package='nav2_lifecycle_manager',
            executable='lifecycle_manager',
            name='lifecycle_manager_localization',
            output='screen',
            parameters=[{'use_sim_time': use_sim_time},
                        {'autostart': autostart},
                        {'node_names': lifecycle_nodes}])
...
lifecycle_nodes = [nav2, related, nodes]
```

- Nav2의 최적화를 위해서는 수많은 매개변수들과 씨름해야 합니다. **yaml** 파일을 통해 관리되며 지금까지 살펴본 내용들이 모두 녹아들어 있습니다.

```yaml
amcl:
  ros__parameters:

bt_navigator:
  ros__parameters:

controller_server:
  ros__parameters:

controller_server_rclcpp_node:
  ros__parameters:
    use_sim_time: False

local_costmap:
  local_costmap:

global_costmap:
  global_costmap:

map_server:
  ros__parameters:
    use_sim_time: False
    yaml_filename: "turtlebot3_world.yaml"

map_saver:
  ros__parameters:

planner_server:
  ros__parameters:

planner_server_rclcpp_node:
  ros__parameters:
    use_sim_time: False

recoveries_server:
  ros__parameters:

robot_state_publisher:
  ros__parameters:
    use_sim_time: False
```

> 자율 주행 시 발생하는 상황들에 대처하기 위해 기본적으로 Behavior Tree 예시를 제공합니다. 구체적인 내용보다 Node를 작성하고 시나리오를 구성할 수 있다는 점에 집중합시다.
