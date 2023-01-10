---
title: "Lecture11 - ROSCPP"
date: 2022-12-23T03:00:51+09:00
draft: false
---

### roscpp Programming

ros는 다양한 언어를 지원하고 있습니다. 지금까지 살펴보았던 rospy는 가장 쉽고 빠르게 배울 수 있어서 사용하였지만, UDPROS, Nodelet, Plugin과 같은 Advanced ROS 개발을 위해서는 C++ 프로그래밍을 통한 Node 개발이 필요합니다.

![cpp.png](/kr/ros_basic_noetic/images11/cpp.png?height=300px)

- image from : [wikipedia](https://ko.m.wikipedia.org/wiki/%ED%8C%8C%EC%9D%BC:ISO_C%2B%2B_Logo.svg)


{{% notice note %}}
rospy를 통해 개념을 모두 익혔기 때문에 이번 강의에서는 개발 API를 위주로 roscpp을 배워보겠습니다.
{{% /notice %}}


- 가장 기초가 되는 Node 프로그래밍부터 차이점을 살펴봅시다.


{{< tabs >}}
{{% tab name="rospy" %}}

```python
#!/usr/bin/env python3

import rospy
from std_msgs.msg import String

def my_first_node():
    # ROS nodes require initialization
    # It contains master registration, uploading parameters
    rospy.init_node('my_first_node', anonymous=True)
    
    # ROS safe timer
    rate = rospy.Rate(10) # 10hz
    
    # Loop control Example
    while not rospy.is_shutdown():
        hello_du = "hello du %s" % rospy.get_time()
        rospy.loginfo(hello_du)
        # Below line calls sleep method in Python internally.
        rate.sleep()

if __name__ == '__main__':
    try:
        my_first_node()
    except rospy.ROSInterruptException:
        pass
```

{{% /tab %}}
{{% tab name="roscpp" %}}

```c++
#include <ros/ros.h>
#include <std_msgs/String.h>

int main(int argc, char** argv){

    ros::init(argc, argv, "basic_node");
    ros::NodeHandle nh;
    ros::Rate r(5);

    while ( ros::ok() ) {
        ROS_INFO("This is Basic Node");
        // ros::spinOnce();
        r.sleep();
    }

    return 0;
}
```

{{% /tab %}}
{{< /tabs >}}


> 간단히 표를 통해 비교해보면 아래와 같이 상당 부분 반복되는 점들을 확인할 수 있습니다. 

|  | rospy | roscpp |
| --- | --- | --- |
| Client Library | import rospy | #include <ros/ros.h> |
| initialization | rospy.init_node | ros::init |
| interface import  | from std_msgs.msg import String | #include <std_msgs/String.h> |
| logging | rospy.loginfo() | ROS_INFO() |
| spin | rospy.spin() | ros::spin() |
| rate | rospy.Rate() | ros::Rate() |

---

### rospy와의 차이점으로, roscpp은 NodeHandle이라는 클래스를 사용합니다.

roscpp은 NodeHandle을 통해 parameter, publisher, subscriber, serviceServer들을 생성하며, 매개변수로 **namespace**를 받습니다. 


{{% notice note %}}
cmd_vel_pub 예시코드에 namespace를 설정한 뒤 topic값의 변화를 살펴봅시다.
{{% /notice %}}


```c++
ros::NodeHandle nh("my_namespace");
```

더불어, NodeHandle은 ROS Node lifecycle 중 “start”의 트리거가 됩니다. Node의 Lifecycle에 대한 자세한 설명은 링크로 대체하겠습니다.

![lf.png](/kr/ros_basic_noetic/images11/lf.png?height=300px)

- image from : [Initialization and Shutdown](http://wiki.ros.org/roscpp/Overview/Initialization%20and%20Shutdown)

{{% notice tip %}}
NodeHandle은 반드시 ros::init 보다 뒤에 생성되어야 합니다
{{% /notice %}}

### ROS C++ Package Build


C++로 작성된 코드는 빌드가 필요하며 caktin 시스템에서는 **CMake**가 빌드를 도와줍니다.

빌드 시에 필요한 공유 라이브러리, 외부 라이브러리들, 빌드 속성과 같은 상세 내용들이 CMakeLists.txt에 위치합니다. 

본 강의에서는 CMake에 대해서는 자세히 다루지 않고, 프로그래밍 시 알아야 하는 부분만 살펴보겠습니다.

- **find_package** - ROS에서 기본 제공하는 패키지들을 손쉽게 추가할 수 있습니다.

```python
## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  roscpp
  geometry_msgs
  sensor_msgs
)
```

- 컴파일 옵션 - add_executable과 add_dependencies를 적용하고, 빌드 결과가 위치하는 지점을 catkin workspace의 build 폴더로 지정해야 합니다.

```python
## Declare a C++ executable
## With catkin_make all packages are built within a single CMake context
## The recommended prefix ensures that target names across packages don't collide
add_executable(cmd_vel_pub_node src/cmdvel_pub.cpp)
add_executable(laser_sub_node src/laser_sub.cpp)
...
add_dependencies(cmd_vel_pub_node ${cmd_vel_pub_node_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
add_dependencies(laser_sub_node ${laser_sub_node_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
...
target_link_libraries(cmd_vel_pub_node
  ${catkin_LIBRARIES}
)
target_link_libraries(laser_sub_node
  ${catkin_LIBRARIES}
)
```

- 이후 catkin build와 실행은 rospy와 동일합니다. (executable name에 주의합니다.)

```bash
catkin build <pkg-name>
source devel/setup.bash
rosrun <pkg-name> <executable-name>
```

## roscpp Topic

- roscpp의 예시들은 rospy와 동일한 기능을 하도록 작성하였습니다. 코드의 API에 집중하여 비교, 분석해보겠습니다.

```bash
# Terminal 1
roslaunch smb_gazebo smb_gazebo.launch
# Terminal 2
rosrun cpp_topic_pkg cmd_vel_pub
# Terminal 3
rosrun cpp_topic_pkg laser_scan_sub
```

|  | rospy | roscpp |
| --- | --- | --- |
| Publisher | http://docs.ros.org/en/melodic/api/rospy/html/rospy.topics.Publisher-class.html | http://docs.ros.org/en/kinetic/api/roscpp/html/classros_1_1NodeHandle.html |
| Subscriber | http://docs.ros.org/en/melodic/api/rospy/html/rospy.topics.Subscriber-class.html | http://docs.ros.org/en/kinetic/api/roscpp/html/classros_1_1NodeHandle.html |

- sub callback의 첫번째 매개변수는 topic msg data이며, std::shared_ptr 형태가 사용됩니다.

```cpp
void sub_callback(const sensor_msgs::LaserScanConstPtr &data ){
    ROS_INFO_STREAM("data.ranges[0]: " << data->ranges[0]);
}
```

- roscpp에서는 Logger 사용 시 c_str 포멧으로 변환해줘야 한다는 단점이 있습니다. 이를 간편하게 하기 위해 저는 **ROS_INFO_STREAM**를 애용합니다.

```cpp
ROS_INFO_STREAM("data.ranges[0]: " << data->ranges[0]);
```

### ROSUDP

- subscribe 예시를 다음과 같이 수정하고 빌드 후 다시 실행시켜봅시다.

```cpp
laser_sub_ = nh->subscribe("scan", 10, &LaserSubNode::laserSubCallback, this,
                ros::TransportHints()
                    .unreliable()
                    .reliable()
                    .maxDatagramSize(1000)
                    .tcpNoDelay()
            );
```

- rosnode info를 통해 transport 상태를 확인합니다. ⇒ UDPROS를 사용하고 있음을 알 수 있습니다.

```bash
$ rosnode info /laser_sub_node
--------------------------------------------------------------------------------
...
 * topic: /scan
    * to: /pointcloud_to_laserscan (http://192.168.55.236:39875/)
    * direction: inbound
    * transport: UDPROS
```

이러한 UDP/TCP 통신의 선택은 Subscriber에서 이루어집니다. Publisher가 Subscriber와 통신을 주고받기 전, Negotiation이라는 과정이 진행됩니다. 이 시점에 Subscriber가 unreliable, UDPROS를 요청하면 Publisher도 UDPROS를 사용해서 Topic Publish를 하는 것입니다.

같은 Publisher라도 Subscriber의 속성에 따라 TCP/UDP를 선책하여 사용할 수 있으며, 예시 코드와 같이  MTU, no delay option을 적용할 수 있습니다.

### roscpp Service

- service 예시들도 rospy와 동일한 기능을 갖습니다.

```bash
# Terminal 1
roslaunch smb_gazebo smb_gazebo.launch
# Terminal 2
rosrun cpp_service_pkg emergency_stop
# Terminal 3
rosrun cpp_service_pkg spawn_model_client
```

- 코드 API의 차이를 비교해봅시다.

|  | rospy | roscpp |
| --- | --- | --- |
| Server | http://docs.ros.org/en/jade/api/rospy/html/rospy.service-module.html | http://docs.ros.org/en/kinetic/api/roscpp/html/classros_1_1NodeHandle.html |
| Client | http://docs.ros.org/en/melodic/api/rospy/html/rospy.impl.tcpros_service.ServiceProxy-class.html | http://docs.ros.org/en/kinetic/api/roscpp/html/classros_1_1NodeHandle.html |


> 실제 로봇 프로그래밍시에는 python보다 c++가 우세하게 사용됩니다. 하지만 우리는 이미 rospy를 통해 통신 메커니즘에 대해 이해하였기 때문에 간단히 짚고 넘어갔으며, 관련된 추가 개발은 강의 노트를 통해 지속 업데이트할 예정입니다.

**참고자료**

- [http://wiki.ros.org/roscpp/Overview/Publishers and Subscribers](http://wiki.ros.org/roscpp/Overview/Publishers%20and%20Subscribers)
- [http://docs.ros.org/](http://docs.ros.org/)