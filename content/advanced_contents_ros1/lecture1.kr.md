---
title: "Lecture1 - ROSCPP"
date: 2022-12-23T03:00:51+09:00
draft: false
---

### roscpp Programming

ros는 다양한 언어를 지원하고 있습니다. 지금까지 살펴보았던 rospy는 가장 쉽고 빠르게 배울 수 있어서 사용하였지만, UDPROS, Nodelet, Plugin과 같은 Advanced ROS 개발을 위해서는 C++ 프로그래밍을 통한 Node 개발이 필요합니다.

![cpp.png](/kr/advanced_contents_ros1/images1/cpp.png?height=300px)

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

|                  | rospy                           | roscpp                       |
| ---------------- | ------------------------------- | ---------------------------- |
| Client Library   | import rospy                    | #include <ros/ros.h>         |
| initialization   | rospy.init_node                 | ros::init                    |
| interface import | from std_msgs.msg import String | #include <std_msgs/String.h> |
| logging          | rospy.loginfo()                 | ROS_INFO()                   |
| spin             | rospy.spin()                    | ros::spin()                  |
| rate             | rospy.Rate()                    | ros::Rate()                  |

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

![lf.png](/kr/advanced_contents_ros1/images1/lf.png?height=300px)

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
# Example 1
rosrun cpp_topic_pkg cmd_vel_pub
# Example 2
rosrun cpp_topic_pkg laser_scan_sub
```

- **cmd_vel_pub code**

{{< tabs >}}
{{% tab name="python" %}}

```python
#!/usr/bin/env python3

import rospy
from geometry_msgs.msg import Twist

class CmdVelPubNode:

    def __init__(self):
        # Publisher requires 3 paramters
        #  1. topic name
        #  2. topic msg type
        #  3. topic queue size
        self.cmd_vel_pub_ = rospy.Publisher("cmd_vel", Twist, queue_size=10)
        self.timer_ = rospy.Timer(rospy.Duration(1.0/10.0), self.pub_msg)
        self.twist_ = Twist()

    def pub_msg(self, event=None):
        # geometry_msgs.Twist
        # ref: http://docs.ros.org/en/melodic/api/geometry_msgs/html/msg/Twist.html
        self.twist_.linear.x = 0.5
        self.twist_.angular.z = 1.0

        self.cmd_vel_pub_.publish(self.twist_)

def cmd_vel_node():
    rospy.init_node('cmd_vel_node', anonymous=True)
    cmd_vel_pub_node = CmdVelPubNode()
    rospy.spin()

if __name__ == '__main__':
    try:
        cmd_vel_node()
    except rospy.ROSInterruptException:
        pass
```

{{% /tab %}}
{{% tab name="c++" %}}

```cpp
#include <ros/ros.h>
#include <geometry_msgs/Twist.h>

class CmdVelPubNode
{
private:
    ros::Publisher cmd_vel_pub_;
    ros::Timer timer_;

    geometry_msgs::Twist twist_msg_;

public:
    CmdVelPubNode(ros::NodeHandle *nh) {
        ROS_INFO("Publisher and Subscriber initialized");
        timer_ = nh->createTimer(ros::Duration(0.1), &CmdVelPubNode::timerCallback, this);
        cmd_vel_pub_ = nh->advertise<geometry_msgs::Twist>("cmd_vel", 10);
    }

    void timerCallback(const ros::TimerEvent& event){

        twist_msg_.linear.x = 0.5;
        twist_msg_.angular.z = 0.5;

        cmd_vel_pub_.publish(twist_msg_);
    }
};

int main(int argv, char** argc) {

    ros::init(argv, argc, "cmd_vel_node");
    // ros::NodeHandle nh("my_namespace");
    ros::NodeHandle nh;

    CmdVelPubNode cmd_pub_node(&nh);

    ros::spin();

    return 0;
}
```

{{% /tab %}}
{{< /tabs >}}

- **laser_scan_sub code**

{{< tabs >}}
{{% tab name="python" %}}

```python
#!/usr/bin/env python3

import rospy
from sensor_msgs.msg import LaserScan

class LaserSubNode:

    def __init__(self):
        # Publisher requires 3 paramters
        #  1. topic name
        #  2. topic msg type
        #  3. sub callback method
        self.laser_sub_ = rospy.Subscriber("scan", LaserScan, self.laser_cb)

    # first param of callback method is always topic msg
    def laser_cb(self, data):
        rospy.loginfo( len(data.ranges))

        print(f"""
        data.ranges[0]: {data.ranges[0]}
        data.ranges[90]: {data.ranges[90]}
        data.ranges[179]: {data.ranges[179]}
        data.ranges[270]: {data.ranges[270]}
        data.ranges[360]: {data.ranges[360]}
        """)

def laser_sub_node():
    rospy.init_node('laser_sub_node', anonymous=True)
    laser_sub_node = LaserSubNode()
    rospy.spin()

if __name__ == '__main__':
    try:
        laser_sub_node()
    except rospy.ROSInterruptException:
        pass
```

{{% /tab %}}
{{% tab name="c++" %}}

```cpp
#include <ros/ros.h>
#include <geometry_msgs/Twist.h>
#include <sensor_msgs/LaserScan.h>

class LaserSubNode
{
private:
    ros::Subscriber laser_sub_;
    ros::Timer timer_;

public:
    LaserSubNode(ros::NodeHandle *nh) {
        ROS_INFO("Publisher and Subscriber initialized");
        // TCPROS
        // laser_sub_ = nh->subscribe("scan", 10, &LaserSubNode::laserSubCallback, this);
        // UDPROS
        laser_sub_ = nh->subscribe("scan", 10, &LaserSubNode::laserSubCallback, this,
                ros::TransportHints()
                    .unreliable()
                    .reliable()
                    .maxDatagramSize(1000)
                    .tcpNoDelay()
            );
    }

    void laserSubCallback(const sensor_msgs::LaserScanConstPtr data){

        ROS_INFO_STREAM("data.ranges[0]: " << data->ranges[0]);
        ROS_INFO_STREAM("data.ranges[90]: " << data->ranges[90]);
        ROS_INFO_STREAM("data.ranges[179]: " << data->ranges[179]);
        ROS_INFO_STREAM("data.ranges[270]: " << data->ranges[270]);
        ROS_INFO_STREAM("data.ranges[360]: " << data->ranges[360]);
        // std::cout << std::to_string(data.ranges[0]) << std::endl;

        ROS_INFO("Publisher and Subscriber initialized");

    }
};

int main(int argv, char** argc) {

    ros::init(argv, argc, "laser_sub_node");
    // ros::NodeHandle nh("my_namespace");
    ros::NodeHandle nh;

    LaserSubNode laser_sub_node(&nh);

    ros::spin();

    return 0;
}
```

{{% /tab %}}
{{< /tabs >}}

|            | rospy                                                                                                                                    | roscpp                                                                                                                        |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Publisher  | [rospy.Publisher(topic_name, Message-Type, queue_size)](http://docs.ros.org/en/melodic/api/rospy/html/rospy.topics.Publisher-class.html) | [advertise<Message-Type>(topic_name, queue_size)](http://docs.ros.org/en/kinetic/api/roscpp/html/classros_1_1NodeHandle.html) |
| Subscriber | [rospy.Subscriber(topic_name, Message-Type, callback)](http://docs.ros.org/en/melodic/api/rospy/html/rospy.topics.Subscriber-class.html) | [subscribe(topic_name, queue_size, callback)](http://docs.ros.org/en/kinetic/api/roscpp/html/classros_1_1NodeHandle.html)     |

- C++로 OOP 코드 작성 시, boost를 통한 binding이 필요함에 유의합니다.

```cpp
CmdVelPubNode(ros::NodeHandle *nh) {
    ROS_INFO("Publisher and Subscriber initialized");
    timer_ = nh->createTimer(ros::Duration(0.1), &CmdVelPubNode::timerCallback, this);
    cmd_vel_pub_ = nh->advertise<geometry_msgs::Twist>("cmd_vel", 10);
}
```

- 파이썬과 마찬가지로 sub callback의 첫번째 매개변수는 topic msg data이며, **std::shared_ptr**가 사용됩니다.

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

> ROS Topic 통신을 위해 Publisher와 Subscriber간의 negotiation이 이루어지며, 이 시점에서 Subscriber에 의해 TCP/UDPROS 중 어떠한 통신이 사용될 지 결정됩니다. Subscriber 코드를 수정하여 UDPROS를 사용해봅시다.

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

위 코드에서, unreliable은 UDPROS를, reliable은 TCPROS를 뜻합니다. UDPROS를 사용하는 경우, maxDatagramSize를 지정할 수 있으며, TCPROS를 사용하는 경우 tcpNoDelay를 사용 가능합니다. 위 코드는 UDPROS 통신을 먼저 시도한 뒤, 응답이 없다면 TCPROS를 사용하게 됩니다.

> 이 같은 설정에 대한 상세 내용은 링크를 확인합시다. ⇒ 참고링크 : [ros::TransportHints Class Reference](https://docs.ros.org/en/api/roscpp/html/classros_1_1TransportHints.html#a897ab86f1b9d7e145cdecc6a5f86ed28)

- 소스코드 빌드 후 다시 실행

```cpp
catkin build cpp_topic_pkg
source devel/setup.bash
rosrun cpp_topic_pkg laser_scan_sub
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

### roscpp Service

- service 예시들도 rospy와 동일한 기능을 갖습니다.

```bash
# Terminal 1
roslaunch smb_gazebo smb_gazebo.launch
# Example 1
rosrun cpp_service_pkg emergency_stop
# Example 2
rosrun cpp_service_pkg spawn_model_client
```

- **emergency_stop code**

{{< tabs >}}
{{% tab name="python" %}}

```python
#! /usr/bin/env python3

import rospy
from roslaunch.pmon import start_process_monitor

from geometry_msgs.msg import Twist
from std_srvs.srv import SetBool, SetBoolResponse

class EmergencyStopNode(object):

    def __init__(self):
        self.cmd_vel_pub_ = rospy.Publisher("cmd_vel", Twist, queue_size=10)
        self.stop_server_ = rospy.Service("emergency_stop", SetBool, self.stop_cb)

        self.pm_ = start_process_monitor()
        self.twist_msg_   = Twist()
        self.response_    = SetBoolResponse()

        rospy.loginfo("E Stop Server Started")

        self.twist_pub()
        rospy.sleep(0.1)

    def twist_pub(self):
        self.twist_msg_.linear.x  = 0.5
        self.twist_msg_.angular.z = 1.0

        self.cmd_vel_pub_.publish(self.twist_msg_)

    def stop_cb(self, request):

        if request.data is True:
            self.twist_msg_.linear.x = 0.0
            self.twist_msg_.angular.z = 0.0
            self.cmd_vel_pub_.publish(self.twist_msg_)

            self.response_.success = True
            self.response_.message = "Successfully Stopped"
        else:
            self.response_.success = False
            self.response_.message = "Stop Failed"

        return self.response_

def main():
    rospy.init_node("emergency_stop_node")

    e_stop_node = EmergencyStopNode()
    rospy.sleep(1.0)
    e_stop_node.twist_pub()

    rospy.spin()

if __name__ == '__main__':
    try:
        main()
    except rospy.ROSInterruptException:
        pass
```

{{% /tab %}}
{{% tab name="c++" %}}

```c++
#include <ros/ros.h>
#include <std_srvs/SetBool.h>
#include <geometry_msgs/Twist.h>

using SetBool = std_srvs::SetBool;

class EmergencyStopNode {
private:
    ros::ServiceServer service_;
    ros::Publisher cmd_vel_pub_;

    geometry_msgs::Twist twist_msg_;
public:
    EmergencyStopNode(ros::NodeHandle *nh){
        service_ = nh->advertiseService("emergency_stop", &EmergencyStopNode::eStopCallback, this);
        cmd_vel_pub_ = nh->advertise<geometry_msgs::Twist>("cmd_vel", 10);

        ROS_INFO_STREAM("EmergencyStopNode Started");
    }

    void moveRobot(){
        twist_msg_.linear.x = 0.5;
        twist_msg_.angular.z = 1.0;

        cmd_vel_pub_.publish(twist_msg_);
    }

    bool eStopCallback(SetBool::Request &req, SetBool::Response &res){

        if(req.data == true){
            twist_msg_.linear.x = 0.0;
            twist_msg_.angular.z = 0.0;
            cmd_vel_pub_.publish(twist_msg_);

            res.success = true;
            res.message = "Successfully Stopped";

            return true;
        } else {
            res.success = false;
            res.message = "Stop Failed";

            return false;
        }
    }
};

int main(int argc, char **argv)
{
    ros::init(argc, argv, "emergency_stop_node");
    ros::NodeHandle nh;
    EmergencyStopNode e_stop_service(&nh);

    auto start_time = ros::Time::now();
    auto cur_time   = ros::Time::now();

    while( (cur_time - start_time) < ros::Duration(3.0)){
        e_stop_service.moveRobot();
        cur_time = ros::Time::now();
    }

    ros::spin();
    ros::shutdown();

    return 0;
}
```

{{% /tab %}}
{{< /tabs >}}

- **spawn_model_client code**

{{< tabs >}}
{{% tab name="python" %}}

```python
#! /usr/bin/env python3

"""
referenced from programcreek
url : https://www.programcreek.com/python/example/93572/rospkg.RosPack
"""

import math
import rospy
import rospkg
from geometry_msgs.msg import Pose
from gazebo_msgs.srv import SpawnModel

def spawn_helix():
    rospy.init_node("gazebo_spawn_model")

    # model_name
    model_name = "box"

    # model_xml
    rospack = rospkg.RosPack()
    model_path = rospack.get_path("py_service_pkg") + "/urdf/"

    with open(model_path + model_name + ".urdf", "r") as xml_file:
        model_xml = xml_file.read().replace("\n", "")

    # robot_namespace
    robot_namespace = ""

    # initial_pose
    initial_pose = Pose()
    initial_pose.position.x = 0.0
    initial_pose.position.y = -1
    initial_pose.position.z = 0.2

    # z rotation -pi/2 to Quaternion
    initial_pose.orientation.z = -0.707
    initial_pose.orientation.w = 0.707

    # reference_frame
    reference_frame = "world"
    theta = 0.0

    spawn_model_prox = rospy.ServiceProxy("gazebo/spawn_urdf_model", SpawnModel)

    for i in range(100):
        # service call
        initial_pose.position.x = theta * math.cos(theta)
        initial_pose.position.y = theta * math.sin(theta)
        theta += 0.2
        entity_name = model_name + str(i)

        result = spawn_model_prox(
            entity_name, model_xml, robot_namespace, initial_pose, reference_frame
        )

        """ result fromat
        bool success
        string status_message
        """
        rospy.loginfo(result)

if __name__ == '__main__':
    try:
        spawn_helix()
    except rospy.ROSInterruptException:
        pass
```

{{% /tab %}}
{{% tab name="c++" %}}

```cpp
#include <fstream> // ros.h doesn't contain this lib
#include <ros/ros.h>
#include <ros/package.h>
#include <geometry_msgs/Pose.h>
#include <gazebo_msgs/SpawnModel.h>

void addXml(gazebo_msgs::SpawnModel& model_in, const std::string& file_path ){
    std::ifstream file(file_path);
    std::string line;

    while (!file.eof()){
        std::getline(file, line);
        model_in.request.model_xml += line;
    }
    file.close();
}

class SpawnModelClient {
private:
    ros::ServiceClient spawn_model_prox;

    int model_num_ = 0;
    double theta_  = 0.0;
public:
    SpawnModelClient(ros::NodeHandle *nh){
        spawn_model_prox = nh->serviceClient<gazebo_msgs::SpawnModel>("gazebo/spawn_urdf_model");

        for(auto i = 0; i < 100; i++){
            this->serviceCall();
        }
    }

    void serviceCall(){

        gazebo_msgs::SpawnModel model;

        // add roslib in find_package()
        auto file_path = ros::package::getPath("cpp_service_pkg") +  "/urdf/box.urdf";

        addXml(model, file_path);

        model.request.model_name = "box" + std::to_string(model_num_++);
        model.request.reference_frame = "world";

        model.request.initial_pose = getPose();

        // ServiceClient.call() => return bool type
        if (spawn_model_prox.call(model)){
            auto response = model.response;
            ROS_INFO("%s", response.status_message.c_str()); // Print the result given by the service called
        }
        else {
            ROS_ERROR("Failed to call service /trajectory_by_name");
            ros::shutdown();
        }

        model_num_++;
    }

    geometry_msgs::Pose getPose(){
        geometry_msgs::Pose initial_pose;

        initial_pose.position.x = theta_ * cos(theta_);
        initial_pose.position.y = theta_ * sin(theta_);
        theta_ += 0.2;
        initial_pose.position.z = 0.2;

        initial_pose.orientation.z = -0.707;
        initial_pose.orientation.w = 0.707;

        return initial_pose;
    }
};

int main(int argc, char** argv){

    ros::init(argc, argv, "gazebo_spawn_model");
    ros::NodeHandle nh;
    SpawnModelClient spawn_model_client(&nh);

    ros::shutdown();

    return 0;
}
```

{{% /tab %}}
{{< /tabs >}}

- Service Server와 Client를 생성하는 코드 API 차이를 비교해봅시다.

|         | rospy                                                                                                                                        | roscpp                                                                                                                           |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Server  | [rospy.Service(service_name, srv_type, callback)](http://docs.ros.org/en/jade/api/rospy/html/rospy.service-module.html)                      | [advertiseService(service_name, srv_type, callback)](http://docs.ros.org/en/kinetic/api/roscpp/html/classros_1_1NodeHandle.html) |
| Client  | [rospy.ServiceProxy(server_name, srv_type)](http://docs.ros.org/en/melodic/api/rospy/html/rospy.impl.tcpros_service.ServiceProxy-class.html) | [serviceClient<srv_type>(service_name)](http://docs.ros.org/en/kinetic/api/roscpp/html/classros_1_1NodeHandle.html)              |
| Time    | rospy.Time.now()                                                                                                                             | ros::Time::now()                                                                                                                 |
| rospack | rospkg.RosPack().get_path()                                                                                                                  | ros::package::getPath()                                                                                                          |

> 실제 로봇 프로그래밍시에는 python보다 c++가 우세하게 사용됩니다. 하지만 우리는 이미 rospy를 통해 통신 메커니즘에 대해 이해하였기 때문에 간단히 짚고 넘어갔으며, 관련된 추가 개발은 강의 노트를 통해 지속 업데이트할 예정입니다.

**참고자료**

- [http://wiki.ros.org/roscpp/Overview/Publishers and Subscribers](http://wiki.ros.org/roscpp/Overview/Publishers%20and%20Subscribers)
- [http://docs.ros.org/](http://docs.ros.org/)
