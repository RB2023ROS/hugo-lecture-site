---
title: "Lecture6 - ROS 2 Service and Examples"
date: 2023-01-02T12:39:03+09:00
draft: false
---

### ROS 2 Service

> ROS Service의 개념을 다시 복습해봅시다.

![service1.gif](/kr/ros2_basic_foxy/images6/service1.gif?height=300px)

- image from : [docs.ros.org](https://docs.ros.org/en/foxy/Tutorials/Services/Understanding-ROS2-Services.html)

Service 개념 정리

- Service를 요청하는 주체를 **Service Client**라고 하며, Service 요청 자체를 **Request, 혹은 Call**이라고 합니다.
- Service를 요청받는 주체를 **Service Server**라고 하며, Service Server는 Request에 대한 응답, 즉 Service **Response**를 다시 Service Client에게 회답합니다.
- Request와 Response를 위해 사용되는 데이터 타입은 **srv**라고 하며 Request와 Response로 나뉩니다.

Service의 중요한 특징 한 가지 추가하자면, 하나의 Service Server에 여러 Client가 request 할 수 있지만, **Server는 동시에 여러 request를 처리하지 못합니다.**

![service2.gif](/kr/ros2_basic_foxy/images6/service2.gif?height=300px)

- image from : [docs.ros.org](https://docs.ros.org/en/foxy/Tutorials/Services/Understanding-ROS2-Services.html)

### ROS Service 커멘드 라인

> 지난 topic 예시와 같이 ROS 1에서 ROS 2로 넘어오면서 변경된 커멘드 라인들을 살펴보겠습니다.

- rosservice list ⇒ ros2 service list
- ros2 service info는 없으며, ros2 service type으로 변경되었습니다.
- srv는 ros2 interface show로 조회 가능합니다.

```python
$ ros2 interface show turtlesim/srv/Spawn
float32 x
float32 y
float32 theta
string name # Optional.  A unique name will be created and returned if this is empty
---
string name
```

{{% notice note %}}
서비스 타입 중간에 보이는 **- - -** 부분은 request와 response의 **구분자**라고 생각하시면 됩니다.
{{% /notice %}}

### Service Example - 📸 KimChi Service

> 이번에 보여드릴 ROS 2 service 예시는 **사진을 찍는 service server**입니다.

- 예시 실행 - rqt를 통해 service call이 성공하면 현재 시간에 해당하는 파일이름으로 로봇이 바라보는 시야의 카메라 이미지가 PNG 형식으로 저장됩니다.

```python
# Terminal 1 - 여러분들만의 world를 사용하시면 더욱 좋습니다.
$ ros2 launch src_gazebo empty_world.launch.py

# Terminal 2
$ colcon build --packages-select py_service_tutorial
$ source install/local_setup.bash
$ ros2 run py_service_tutorial take_picture_server
[INFO] [turtle_circle_server]: Picture Taking Node Started
[INFO] [turtle_circle_server]: KimChi~
[INFO] [turtle_circle_server]: Image saved in 1672569224.png

# Terminal 3 - rqt 실행 후 service caller 실행 (plugins => services => service caller)
$ rqt
```

- Service Caller의 사용법과 결과는 다음과 같습니다.

![picture_srv.gif](/kr/ros2_basic_foxy/images6/picture_srv.gif?height=300px)

코드를 살펴보기 전에, 이를 어떻게 구현할 수 있을지 같이 생각해봅시다.

- 사진 촬영을 포함하는 **Service Server** 필요
- 로봇 전방 image data의 **Subscriber** 필요
- ROS의 **sensor_msgs/msg/Image**를 저장 가능한 포멧으로 변경 필요

따라서, Node의 생성자는 다음과 같이 작성합니다.

```python
class PictureNode(Node):

    def __init__(self):
        super().__init__('turtle_circle_server')

        self.server = self.create_service(
            SetBool, 'take_picture', self.take_picture_callback
        )

        self.subscriber = self.create_subscription(
            Image, 'logi_camera_sensor/image_raw', self.sub_callback, 10
        )

        self.br = CvBridge()
        self.is_request = False
```

- 이번에 사용하는 srv는 **example_interfaces/srv/SetBool**이며, Bool Type의 request와 String response를 갖고 있습니다.

```bash
$ ros2 interface show example_interfaces/srv/SetBool
# This is an example of a service to set a boolean value.
# This can be used for testing but a semantically meaningful
# one should be created to be built upon.

bool data # e.g. for hardware enabling / disabling
---
bool success   # indicate successful run of triggered service
string message # informational, e.g. for error messages
```

- service call에 대한 callback입니다. **is_request**를 바꿔주기만 하는데, 이것이 하는 역할이 무엇일지 생각해보세요.

```python
    def take_picture_callback(self, request, response):

        if request.data is True:
            self.get_logger().info('KimChi~')
            self.is_request = True

        response.success = True
        response.message = "Successfully image written"

        return response
```

- 정답은 subscription callback에서 찾을 수 있습니다. subscribe된 이미지 데이터는 is_request가 True인 순간에만 사용됩니다. CV Bridge를 통해 ROS topic을 OpenCV 포맷으로 바꿀 수 있으며, **imwrite**를 통해 이미지를 저장할 수 있습니다.

```python
def sub_callback(self, data):

    if self.is_request:
        current_frame = self.br.imgmsg_to_cv2(data, "bgr8")

        file_name = str(self.get_clock().now().to_msg().sec) + '.png'
        cv2.imwrite(file_name, current_frame)
        self.get_logger().info(f'Image saved in {file_name}')

        self.is_request = False
```

> Gazebo에 다양한 물체를 배치시킨 뒤 사진을 찍어보는 것도 좋은 실습이 될 것입니다. 제가 준비한 dataset을 사용하여 여러분들만의 실습도 해보세요.

📁 [3DGEMS.zip](https://drive.google.com/file/d/14P83HSgLmN5iC4Yx6TMl7XvbIKuFr7as/view?usp=sharing)

- 제공되는 3DGEMS 폴더를 압축해제한 뒤, **~/.gazebo/models** 폴더에 위치시킵니다.

{{% notice note %}}
해당 모델들의 출처는 다음과 같습니다. (["The Effect of Color Space Selection on Detectability and Discriminability of Colored Objects." arXiv preprint arXiv:1702.05421 (2017).](https://arxiv.org/abs/1702.05421))
{{% /notice %}}

{{% notice tip %}}
WSL2를 사용하시는 분들께서는 터미널에서 **explorer.exe .** 를 입력하면 윈도우 파일 탐색기를 실행 가능합니다.
{{% /notice %}}

- 다시 한 번 Gazebo를 실행시킨 뒤 새로 추가된 모델들을 사용해봅시다.

![lec6_0.png](/kr/ros2_basic_foxy/images6/lec6_0.png?height=300px)

### Custom Interface와 코딩 과제 - Turtle Jail

> ROS 2에서 custom interface를 만들기 위해서는 **C++ Package**에서 작업이 이루어져야 합니다. C++ package는 build type ament_cmake를 사용하는 package였습니다.

```bash
$ ros2 pkg create --build-type ament_cmake <package-name>
$ ros2 pkg create --build-type ament_cmake custom_interfaces
```

- 해당 패키지 내 action, msg, srv 라는 폴더를 만들고 해당 폴더 안에 나만의 인터페이스를 작성합니다.

![lec6_1.png](/kr/ros2_basic_foxy/images6/lec6_1.png?height=300px)

{{% notice note %}}
사용할 수 있는 기본 데이터 형식들은 [이 링크](https://design.ros2.org/articles/interface_definition.html)를 참고합니다.
{{% /notice %}}

이번에 만들어볼 custom interface는 다음 과제와 연결됩니다. 우선 저를 따라와 주세요.

- srv라는 폴더를 만들고, **TurtleJail.srv**라는 파일을 생성하여 아래와 같은 내용을 작성합니다.

```cpp
float32 width
float32 height
---
bool success
```

해당 interface를 rclpy, rclcpp에서 사용 가능하도록 해봅시다. ROS 2는 DDS의 IDL(Interface Description Language)를 사용하여 다양한 언어에서 사용 가능한 데이터 타입을 만들 수 있습니다.

- CMakeLists.txt 수정

```cpp
find_package(rosidl_default_generators REQUIRED)
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Num.msg"
  "srv/AddThreeInts.srv"
  "srv/TurningControl.srv"
  "srv/TurtleJail.srv"
  "action/Fibonacci.action"
  "action/Maze.action"
)
```

- package.xml 수정

```xml
<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

- 패키지 빌드

```bash
$ colcon build --packages-select custom_interfaces && rosfoxy
Starting >>> custom_interfaces
Finished <<< custom_interfaces [0.62s]

Summary: 1 package finished [0.84s]
```

- 이제 ROS 2에서 해당 인터페이스를 사용할 수 있습니다 (단, 해당 workspace를 바라보게 해야 합니다.)

```bash
$ ros2 interface show custom_interfaces/srv/TurtleJail
float32 width
float32 height
---
bool success
```

custom interface의 사용 시 파이썬 패키지에서는 별도 작업 없이 사용 가능하지만 C++ 패키지는 코딩 시 CMakeLists.txt의 수정이 필요합니다. 이는 다소 난이도가 있어 강의에서 살피지는 않고 링크를 남겨두겠습니다.

- [https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Custom-ROS2-Interfaces.html#id10](https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Custom-ROS2-Interfaces.html#id10)

### Assignment - Turtle Jail

> topic과 service에 대해서 모두 살펴본 지금 상황에서 여러분들께 **코딩 과제**를 제시해보고자 합니다. 이번 코딩 과제에서 구현해야 하는 최종 결과는 다음과 같습니다.

![turtle_jail.gif](/kr/ros2_basic_foxy/images6/turtle_jail.gif?height=300px)

rqt를 통해 turtle_jail_size service call을 하며, 감옥의 사이즈를 설정합니다.

거북이는 감옥을 벗어날 수 없으며, 감옥을 벗어나는 순간 원점으로 순간이동합니다.

- 터미널에서 실행하는 절차는 다음과 같습니다.

```bash
# Terminal 1 – turtlesim실행
ros2 run turtlesim turtlesim_node

# Terminal 2 - turtle_teleop 실행
ros2 run turtlesim turtle_teleop_key

# Terminal 3 - 과제 프로그램 실행
ros2 run py_service_tutorial turtle_jail
[INFO] [turtle_jail_node]: === [Service Client : Ready to Call Service Request] ===
[INFO] [turtle_jail_node]: ==== [Service Server : Ready to receive Service Request] ====

# Terminal 4 - rqt의 service caller 실행 후 /turtle_jail_size에게 service call
cd ~/ros2_ws
source install/local_setup.bash
rqt
```

{{% notice note %}}
이번 예시는 custom interface를 사용하므로 local_setup.bash를 꼭 실행해주세요!
{{% /notice %}}

이 예시에서는 topic과 service를 모두 사용해야 합니다. 지금까지 학습한 내용들을 확인해볼 수 있는 좋은 기회가 될 것입니다.

- 힌트1 : turtlesim의 좌표계

![lec6_2.png](/kr/ros2_basic_foxy/images6/lec6_2.png?height=300px)

- 힌트 2 : 거북이를 순간이동시키기 위해 **/turtle1/teleport_absolute** service를 사용하세요.
