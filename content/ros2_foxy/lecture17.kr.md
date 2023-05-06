---
title: "Lecture17. Real Hardware Examples"
date: 2023-04-28T10:51:09+09:00
draft: false
---

> 이번 시간에는 실제 센서 하드웨어를 실행하는 ROS 2 package들을 실행해보고, 코드 분석을 진행해보려합니다. 준비된 예시들은 다음과 같습니다.

- Intel Realsense2 D455 Camera
- Velodyne VLP16 3D lidar
- Orbbec Astra+ Camera

## Intel realsense2 camera ROS 2

real hardware 강의의 두번째 시간으로 Intel의 realsense2 카메라 패키지를 실행하고, 분석해 보겠습니다.

![Untitled.png](/kr/ros2_foxy/images17/Untitled.png?height=300px)

- 예제 실행을 위해 필요한 종속성들을 설치하고 realsense sdk인 librealsense를 설치합니다.

```bash
sudo apt-get update && sudo apt-get upgrade

# Register the server's public key
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
> gpg: Total number processed: 1

# Add the server to the list of repositories
sudo add-apt-repository "deb https://librealsense.intel.com/Debian/apt-repo $(lsb_release -cs) main" -u

# Install the libraries (see section below if upgrading packages)
sudo apt-get install librealsense2-dkms -y
sudo apt-get install librealsense2-utils -y
```

- 실제 카메라 연결을 통해 realsense2 패키지 설치를 검증해봅시다.

```bash
# Reconnect the Intel RealSense depth camera and run
realsense-viewer

modinfo uvcvideo | grep "version:"
# should include realsense string
```

![Untitled1.png](/kr/ros2_foxy/images17/Untitled1.png?height=350px)

⇒ 위와 같은 이미지 출력을 얻었다면 성공입니다.

- 사실, realsense camera는 intel이 개발한 만큼 ubuntu 배포판 패키지로 ros2를 지원하고 있습니다. 사용만 하고 싶다면 아래와 같이 손쉽게 설치가 가능합니다.

```bash
# Install Dependencies
sudo apt-get install ros-foxy-cv-bridge ros-foxy-librealsense2 ros-foxy-message-filters ros-foxy-image-transport -y
sudo apt install ros-foxy-diagnostic-updater -y
sudo apt-get install -y libssl-dev libusb-1.0-0-dev pkg-config libgtk-3-dev
sudo apt-get install -y libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev
```

⇒ 하지만 우리는 패키지 분석이 목적이므로 직접 소스코드 빌드와 분석을 해보겠습니다. 아래 링크를 통해 release version을 다운받고 ros2 workspace에 압축을 해제합니다.

[https://github.com/IntelRealSense/realsense-ros/releases](https://github.com/IntelRealSense/realsense-ros/releases)

![Untitled2.png](/kr/ros2_foxy/images17/Untitled2.png?height=200px)

- 폴더 이동 후 빌드는 아래 순서대로 해주세요!

```bash
cbp realsense2_camera_msgs && source install/local_setup.bash
cbp realsense2_description && source install/local_setup.bash
cbp realsense2_camera && source install/local_setup.bash
```

- 우선 빌드가 잘 되었는지 예시부터 실행해봅시다.

```bash
ros2 launch realsense2_camera rs_launch.py
```

- topic list, rviz2를 통해 정상 동작을 확인합니다.

```bash
$ ros2 topic list
/camera/color/camera_info
/camera/color/image_raw
/camera/color/metadata
/camera/depth/camera_info
/camera/depth/image_rect_raw
/camera/depth/metadata
/camera/extrinsics/depth_to_color
/camera/extrinsics/depth_to_depth
/camera/imu
/parameter_events
/rosout
/tf_static
```

![Untitled3.png](/kr/ros2_foxy/images17/Untitled3.png?height=300px)

- point cloud topic을 활성화하기 위해선 아래와 같은 launch가 필요합니다.

```bash
ros2 launch realsense2_camera rs_launch.py pointcloud.enable:=true
```

- rviz2를 통해 pointcloud를 시각화해봅시다. (메가커피에서 강의를 제작해서 손흥민 선수 얼굴이 보이네요 ㅎㅎ)

![Untitled4.png](/kr/ros2_foxy/images17/Untitled4.png?height=350px)

- rviz2에서 tf2 data도 확인할 수 있습니다. fixed frame은 camera_link로 두시면 됩니다.

![Untitled5.png](/kr/ros2_foxy/images17/Untitled5.png?height=300px)

- tf2 tree도 확인해봅시다.

![Untitled6.png](/kr/ros2_foxy/images17/Untitled6.png?height=300px)

### Composition launch

> realsense ros2 package는 Intra-communication을 사용하여 메모리 최적화를 적용한 composition launch도 지원하고 있습니다. 이를 사용하기 위해서는 패키지 빌드를 다시 해야 하며, 이러한 이유로 저는 realsense_ws라는 별도의 workspace를 만들었습니다.

```bash
cd ~/
mkdir -p realsense_ws/src
cd realsense_ws

# locate packages & build again
colcon build --cmake-args '-DBUILD_TOOLS=ON'
```

- Composition 예시를 실행해봅시다!

```bash
$ ros2 launch realsense2_camera rs_intra_process_demo_launch.py
...
[component_container-1] [INFO] [1682508794.621382739] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0597926 [sec]
[component_container-1] [INFO] [1682508794.689360200] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0610729 [sec]
[component_container-1] [INFO] [1682508794.755558725] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0605604 [sec]
[component_container-1] [INFO] [1682508794.822757399] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0610629 [sec]
[component_container-1] [INFO] [1682508794.889473916] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0610784 [sec]
[component_container-1] [INFO] [1682508794.955772739] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0606767 [sec]
[component_container-1] [INFO] [1682508795.022029595] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0602304 [sec]
[component_container-1] [INFO] [1682508795.088818104] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0603308 [sec]
[component_container-1] [INFO] [1682508795.156870084] [frame_latency]: Got msg with address 0x7fe51c6cb570 with latency of 0.0616579 [sec]
[component_container-1] [INFO] [1682508795.223072425] [frame_latency]:
```

- rqt_image_view를 통해 이미지도 확인해 보겠습니다.

```bash
$ ros2 run rqt_image_view rqt_image_view
```

![Untitled7.png](/kr/ros2_foxy/images17/Untitled7.png?height=300px)

- 실제 일반 node 실행과 composition 사용 시 점유하는 메모리를 비교해 보았습니다.

```bash
PID   USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 2530 kimsooy+  20   0  897252  82312  50300 R  12.3   0.5   1:09.80 x-terminal-emul
 1247 kimsooy+  20   0 6284836 344712 109988 S  11.3   2.1   7:17.14 gnome-shell
41892 kimsooy+  20   0 1183960  85788  57140 S   5.3   0.5   0:03.28 component_conta
```

```bash
PID   USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 1247 kimsooy+  20   0 6333092 344816 109988 S  11.6   2.1   8:00.07 gnome-shell
 2530 kimsooy+  20   0  897252  82664  50300 S  10.9   0.5   1:33.14 x-terminal-emul
42240 kimsooy+  20   0 1457552  92104  57096 S   8.9   0.6   0:03.88 realsense2_came
```

{{% notice note %}}
노트북 환경이어 겉보기에 큰 차이는 없지만, 실제 edge device에서 실행 + 다중 Node들이 결합되는 경우 Composition 사용 여부가 큰 차이를 가질 것입니다.
{{% /notice %}}

### realsense2 Launch file 분석

> launch file과 소스코드 일부 분석을 통해 센서 패키지 개발의 흐름을 이해해봅시다.

- rs_launch.py의 시작은 수많은 launch configuration들로 시작됩니다. (`DeclareLaunchArgument`, `LaunchConfiguration을` 별도의 함수로 구현해 두고 재사용하고 있습니다.)

```python
configurable_parameters = [{'name': 'camera_name',                  'default': 'camera', 'description': 'camera unique name'},
                           {'name': 'serial_no',                    'default': "''", 'description': 'choose device by serial number'},
                           {'name': 'usb_port_id',                  'default': "''", 'description': 'choose device by usb port id'},
                           {'name': 'device_type',                  'default': "''", 'description': 'choose device by type'},
...

def declare_configurable_parameters(parameters):
    return [DeclareLaunchArgument(param['name'], default_value=param['default'], description=param['description']) for param in parameters]

def set_configurable_parameters(parameters):
    return dict([(param['name'], LaunchConfiguration(param['name'])) for param in parameters])
```

- ROS 2 dashing, eloquent, foxy / 별도 user config file이 있는지 등 각종 조건에 따른 조건 분리도 설정해 두었습니다.

```python
def generate_launch_description():
    log_level = 'info'
    if (os.getenv('ROS_DISTRO') == "dashing") or (os.getenv('ROS_DISTRO') == "eloquent"):

...

else:
    return LaunchDescription(declare_configurable_parameters(configurable_parameters) + [
        # Realsense
        launch_ros.actions.Node(
            condition=IfCondition(PythonExpression([LaunchConfiguration('config_file'), " == ''"])),

...
        launch_ros.actions.Node(
            condition=IfCondition(PythonExpression([LaunchConfiguration('config_file'), " != ''"])),
```

- 실제 실행되는 것은 realsense2_camera_node이며, 다음으로 이 node에 대해서 파헤쳐보겠습니다.

```python
launch_ros.actions.Node(
	package='realsense2_camera',
	namespace=LaunchConfiguration("camera_name"),
	name=LaunchConfiguration("camera_name"),
	executable='realsense2_camera_node',
	parameters=[set_configurable_parameters(configurable_parameters)
	            , PythonExpression([LaunchConfiguration("config_file")])
	            ],
	output='screen',
	arguments=['--ros-args', '--log-level', LaunchConfiguration('log_level')],
	emulate_tty=True,
)
```

### 코드 분석

- `CMakeLists.txt`를 살펴보면 realsense2_camera_node이 어떻게 빌드된 결과물인지 확인할 수 있습니다.

```python
rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "realsense2_camera::RealSenseNodeFactory"
  EXECUTABLE realsense2_camera_node
)
```

{{% notice tip %}}
그런데, 일반적으로 executable을 빌드하는 방식이 아닌 rclcpp_components_register_node라는 옵션을 사용하고 있습니다. rclcpp_components_register_node은 라이브러리, 실행 파일을 모두 생성할 수 있는 rclcpp의 CMake tools입니다.
[https://docs.ros2.org/latest/api/rclcpp_components/](https://docs.ros2.org/latest/api/rclcpp_components/)
{{% /notice %}}

{{% notice note %}}
realsense2 ROS 2 package는 ROS 1 때부터의 레거시가 잔존하고 구조상으로 깔끔하다고 말하기는 어려운 코드입니다. 분석은 하겠지만 이러한 형태가 일반적이라고 할 수 없음을 인지하시기 바랍니다! (저라면 싹다 처음부터 짤 것 같습니다.)
{{% /notice %}}

- realsense2_camera_node를 구성하는 composition인 RealSenseNodeFactory는 realsense_node_factory.h에 구현되어 있습니다. 디자인 패턴을 따랐다고 말할 수는 없지만, 일종의 팩토리이며 \_realSenseNode가 Node에 해당합니다.

```cpp
namespace realsense2_camera
{
    class RealSenseNodeFactory : public rclcpp::Node
    {
    public:
        explicit RealSenseNodeFactory(const rclcpp::NodeOptions & node_options = rclcpp::NodeOptions());
        RealSenseNodeFactory(
            const std::string & node_name, const std::string & ns,
            const rclcpp::NodeOptions & node_options = rclcpp::NodeOptions());
        virtual ~RealSenseNodeFactory();

    private:
        void init();
        void closeDevice();
        void startDevice();
        void changeDeviceCallback(rs2::event_information& info);
        void getDevice(rs2::device_list list);
        void tryGetLogSeverity(rs2_log_severity& severity) const;
        static std::string parseUsbPort(std::string line);

        rclcpp::Node::SharedPtr _node;
        rs2::device _device;
        std::unique_ptr<BaseRealSenseNode> _realSenseNode;
        rs2::context _ctx;
        std::string _serial_no;
        std::string _usb_port_id;
        std::string _device_type;
        double _wait_for_device_timeout;
        double _reconnect_timeout;
        bool _initial_reset;
        std::thread _query_thread;
        bool _is_alive;
        rclcpp::Logger _logger;
        std::shared_ptr<Parameters> _parameters;
    };
}//end namespace
```

> 우선 여기서 몇가지 주요 함수들을 분석하고 더 깊이 들어가보겠습니다.

- realsense_node_factory.cpp - startDevice()

```cpp
void RealSenseNodeFactory::startDevice()
{
    if (_realSenseNode) _realSenseNode.reset();
    std::string pid_str(_device.get_info(RS2_CAMERA_INFO_PRODUCT_ID));
    uint16_t pid = std::stoi(pid_str, 0, 16);
    try
    {
        switch(pid)
        {
        case SR300_PID:
        case SR300v2_PID:
        case RS400_PID:
        case RS405_PID:
        case RS410_PID:
        case RS460_PID:
        case RS415_PID:
        case RS420_PID:
        case RS420_MM_PID:
        case RS430_PID:
        case RS430_MM_PID:
        case RS430_MM_RGB_PID:
        case RS435_RGB_PID:
        case RS435i_RGB_PID:
        case RS455_PID:
        case RS465_PID:
        case RS_USB2_PID:
        case RS_L515_PID_PRE_PRQ:
        case RS_L515_PID:
        case RS_L535_PID:
            _realSenseNode = std::unique_ptr<BaseRealSenseNode>(new BaseRealSenseNode(*this, _device, _parameters, this->get_node_options().use_intra_process_comms()));
            break;
        case RS_T265_PID:
            _realSenseNode = std::unique_ptr<T265RealsenseNode>(new T265RealsenseNode(*this, _device, _parameters, this->get_node_options().use_intra_process_comms()));
            break;
        default:
            ROS_FATAL_STREAM("Unsupported device!" << " Product ID: 0x" << pid_str);
            rclcpp::shutdown();
            exit(1);
        }
        _realSenseNode->publishTopics();

    }
    catch(const rs2::backend_error& e)
    {
        std::cerr << "Failed to start device: " << e.what() << '\n';
        _device.hardware_reset();
        _device = rs2::device();
    }
}
```

> main Node인 \_realSenseNode를 정의하고 있으며, 이는 사용하는 카메라 모델에 따라 차이를 갖습니다.

- \_realSenseNode를 생성한 이후, publishTopics 메소드가 호출됩니다. `base_realsense_node.cpp - publishTopics()`

```cpp
void BaseRealSenseNode::publishTopics()
{
    getParameters();
    setup();
    ROS_INFO_STREAM("RealSense Node Is Up!");
}
```

⇒ getParameters()에서는 말 그대로 각종 ROS 2 매개변수들의 파싱이 이루어집니다.

- setup() 메소드에서 주요 기능들이 실행되며 여기로 한단계 더 진입해보겠습니다. `BaseRealSenseNode::setup()`

```cpp
void BaseRealSenseNode::setup()
{
    setDynamicParams();
    startDiagnosticsUpdater();
    setAvailableSensors();
    SetBaseStream();
    setupFilters();
    setupFiltersPublishers();
    setCallbackFunctions();
    monitoringProfileChanges();
    updateSensors();
    publishServices();
}
```

{{% notice note %}}
이렇게 복잡한 구조를 갖게 된 까닭은 유추하건데 ROS 1의 레거시에서 시작했기 때문일겁니다. ⇒ ROS 1 코드에서 주요 기능들을 모두 모듈화하고 ROS 2에서는 setup()이라는 이름으로 모조리 때려넣은 것이지요. 다시 말하지만, 구조적으로 잘 구현된 코드는 아닙니다.
{{% /notice %}}

> 위 함수들 대부분은 이름을 통해 기능을 유추할 수 있으며 중요한 함수들만 몇가지 파헤쳐보겠습니다.

- setAvailableSensors

```cpp
std::function<void(rs2::frame)> frame_callback_function = [this](rs2::frame frame){
    bool is_filter(_filters.end() != find_if(_filters.begin(), _filters.end(), [](std::shared_ptr<NamedFilter> f){return (f->is_enabled()); }));
    if (_sync_frames || is_filter)
        this->_asyncer.invoke(frame);
    else
        frame_callback(frame);
};

std::function<void(rs2::frame)> imu_callback_function = [this](rs2::frame frame){
    imu_callback(frame);
    if (_imu_sync_method != imu_sync_method::NONE)
        imu_callback_sync(frame);
};

std::function<void(rs2::frame)> multiple_message_callback_function = [this](rs2::frame frame){multiple_message_callback(frame, _imu_sync_method);};

std::function<void()> update_sensor_func = [this](){
    {
        std::lock_guard<std::mutex> lock_guard(_profile_changes_mutex);
        _is_profile_changed = true;
    }
    _cv_mpc.notify_one();
};
```

> 각종 callback들이 구현됩니다. 현재 callback안에서 다시 callback이 실행되기도 하고, lock guard가 호출되고도 있지만, 결국 해당 callback들이 subscriber, service server의 callback으로 사용됩니다.

- 더불어, setAvailableSensors에서 librealsense와, ROS 2의 연동이 동작합니다. 이를 RosSensor라는 패턴으로 구현하여 앞서 구현한 callback들과 binding합니다.

```cpp
for(auto&& sensor : _dev_sensors)
{
    const std::string module_name(sensor.get_info(RS2_CAMERA_INFO_NAME));
    std::unique_ptr<RosSensor> rosSensor;
    if (sensor.is<rs2::depth_sensor>() ||
        sensor.is<rs2::color_sensor>() ||
        sensor.is<rs2::fisheye_sensor>())
    {
        ROS_DEBUG_STREAM("Set " << module_name << " as VideoSensor.");
        rosSensor = std::make_unique<RosSensor>(sensor, _parameters, frame_callback_function, update_sensor_func, hardware_reset_func, _diagnostics_updater, _logger, _use_intra_process, _dev.is<playback>());
    }
    else if (sensor.is<rs2::motion_sensor>())
    {
        ROS_DEBUG_STREAM("Set " << module_name << " as ImuSensor.");
        rosSensor = std::make_unique<RosSensor>(sensor, _parameters, imu_callback_function, update_sensor_func, hardware_reset_func, _diagnostics_updater, _logger, false, _dev.is<playback>());
    }
    else if (sensor.is<rs2::pose_sensor>())
    {
        ROS_DEBUG_STREAM("Set " << module_name << " as PoseSensor.");
        rosSensor = std::make_unique<RosSensor>(sensor, _parameters, multiple_message_callback_function, update_sensor_func, hardware_reset_func, _diagnostics_updater, _logger, false, _dev.is<playback>());
    }
    else
    {
        ROS_ERROR_STREAM("Module Name \"" << module_name << "\" does not define a callback.");
        throw("Error: Module not supported");
    }
    _available_ros_sensors.push_back(std::move(rosSensor));
}
```

- 생성된 rosSensor들은 \_available_ros_sensors에 push_back된 후, updateSensors에서 소비됩니다.

```cpp
void BaseRealSenseNode::updateSensors()
{
  std::lock_guard<std::mutex> lock_guard(_update_sensor_mutex);
  try{
    for(auto&& sensor : _available_ros_sensors)
    {

      if(is_profile_changed)
      {
        // Start/stop sensors only if profile was changed
        // No need to start/stop sensors if align_depth was changed
        ROS_INFO_STREAM("Stopping Sensor: " << module_name);
        sensor->stop();
      }
      stopPublishers(active_profiles);

      if (!wanted_profiles.empty())
      {
          if(is_profile_changed)
          {
              ROS_INFO_STREAM("Starting Sensor: " << module_name);
              sensor->start(wanted_profiles);
          }
       ...
```

- RosSensor의 start 메소드에서 드디어 realsense sdk와의 연동이 동작하며, `\_frame_callback`이라는 메모리 전달이 구현되어 있습니다.

```cpp
// base_realsense_node.h
std::vector<std::unique_ptr<RosSensor>> _available_ros_sensors;

bool RosSensor::start(const std::vector<stream_profile>& profiles)
{
    if (get_active_streams().size() > 0)
        return false;
    setupErrorCallback();
    rs2::sensor::open(profiles);

    for (auto& profile : profiles)
    ROS_INFO_STREAM("Open profile: " << ProfilesManager::profile_string(profile));

    rs2::sensor::start(_frame_callback);
    ...
    return true;
}

...

_frame_callback = [this](rs2::frame frame)
{
  runFirstFrameInitialization();
  auto stream_type = frame.get_profile().stream_type();
  auto stream_index = frame.get_profile().stream_index();
  stream_index_pair sip{stream_type, stream_index};
  try
  {
      _origin_frame_callback(frame);
      if (_frequency_diagnostics.find(sip) != _frequency_diagnostics.end())
          _frequency_diagnostics.at(sip).Tick();
  }
  catch(const std::exception& ex)
  ...
};
```

### Publish 분석

> 실제 image topic이 publish 되는 부분을 추가로 분석해 보겠습니다.

- startPublishers 내부에서 `image_rcl_publisher` 타입의 클래스 포인터가 할당됩니다.

```cpp
void BaseRealSenseNode::startPublishers(const std::vector<stream_profile>& profiles, const RosSensor& sensor)
{
    const std::string module_name(create_graph_resource_name(rs2_to_ros(sensor.get_info(RS2_CAMERA_INFO_NAME))));
    for (auto& profile : profiles)
    {
       ...
        // We can use 2 types of publishers:
        // Native RCL publisher that support intra-process zero-copy comunication
        // image-transport package publisher that adds a commpressed image topic if package is found installed
        if (_use_intra_process)
        {
            _image_publishers[sip] = std::make_shared<image_rcl_publisher>(_node, image_raw.str(), qos);
        }
        else
        {
            _image_publishers[sip] = std::make_shared<image_transport_publisher>(_node, image_raw.str(), qos);
            ROS_DEBUG_STREAM("image transport publisher was created for topic" << image_raw.str());
        }
        ...
        if (_use_intra_process)
        {
            _depth_aligned_image_publishers[sip] = std::make_shared<image_rcl_publisher>(_node, aligned_image_raw.str(), qos);
        }
        else
        {
            _depth_aligned_image_publishers[sip] = std::make_shared<image_transport_publisher>(_node, aligned_image_raw.str(), qos);
            ROS_DEBUG_STREAM("image transport publisher was created for topic" << image_raw.str());
        }
        _depth_aligned_info_publisher[sip] = _node.create_publisher<sensor_msgs::msg::CameraInfo>(aligned_camera_info.str(),
                                              rclcpp::QoS(rclcpp::QoSInitialization::from_rmw(info_qos), info_qos));
        }
```

- 참고로, \_image_publishers는 아래와 같이 `std::map` 타입의 클래스 변수이며 index_pair와 image_publisher 포인터를 담고 있습니다.

```cpp
std::map<stream_index_pair, std::shared_ptr<image_publisher>> _image_publishers;
```

- `image_publisher.cpp - image_rcl_publisher`
  image_rcl_publisher의 publish 메소드에서 드디어 실질적인 publish가 이루어집니다. 더불어 `std::move`를 통해 소유권을 이전하고 있습니다. (intra-processing에 대비한 것으로 보입니다.)

```cpp
image_rcl_publisher::image_rcl_publisher( rclcpp::Node & node,
                                          const std::string & topic_name,
                                          const rmw_qos_profile_t & qos )
{
    image_publisher_impl = node.create_publisher< sensor_msgs::msg::Image >(
        topic_name,
        rclcpp::QoS( rclcpp::QoSInitialization::from_rmw( qos ), qos ) );
}

void image_rcl_publisher::publish( sensor_msgs::msg::Image::UniquePtr image_ptr )
{
    image_publisher_impl->publish( std::move( image_ptr ) );
}

...

image_transport_publisher::image_transport_publisher( rclcpp::Node & node,
                                                      const std::string & topic_name,
                                                      const rmw_qos_profile_t & qos )
{
    image_publisher_impl = std::make_shared< image_transport::Publisher >(
        image_transport::create_publisher( &node, topic_name, qos ) );
}
void image_transport_publisher::publish( sensor_msgs::msg::Image::UniquePtr image_ptr )
{
    image_publisher_impl->publish( *image_ptr );
}
```

- `image_rcl_publisher`가 사용되는 부분을 추적해 보겠습니다. BaseRealSenseNode의 frame_callback에서 다시 publishFrame callback을 호출하여 \_image_publishers를 전달하고 있습니다.

```cpp
void BaseRealSenseNode::frame_callback(rs2::frame frame)
{
...

	publishFrame(f, t, sip,
	                _image,
	                _info_publisher,
	                _image_publishers);
	...
  publishFrame(frame_to_send, t,
              DEPTH,
              _image,
              _info_publisher,
              _image_publishers);

```

- 전달된 \_image_publishers는 std::map type이기 때문에 rgb, depth, infra camera 중에서 indexing하여 최종 publish가 진행됩니다.

```cpp
void BaseRealSenseNode::publishFrame(rs2::frame f, const rclcpp::Time& t,
                                     const stream_index_pair& stream,
                                     std::map<stream_index_pair, cv::Mat>& images,
                                     const std::map<stream_index_pair, rclcpp::Publisher<sensor_msgs::msg::CameraInfo>::SharedPtr>& info_publishers,
                                     const std::map<stream_index_pair, std::shared_ptr<image_publisher>>& image_publishers,
                                     const bool is_publishMetadata){

auto& image_publisher = image_publishers.at(stream);
...
image_publisher->publish(std::move(img));
```

{{% notice note %}}
다시 말하지만, realsense2 ros2 코드가 완성도가 높은 것은 아닙니다. 하지만, 적어도 ROS 2시스템 차원에서 코드를 분석할 수 있는 예시가 되기에 같이 파헤쳐 보았습니다.
{{% /notice %}}

### rs_intra_process_demo_launch.py

- Composition 예시에서 살펴본 바와 같이 launch file에서 `ComposableNodeContainer`를 통해 composition container를 실행하고, RealSenseNodeFactory/FrameLatencyNode composition을 load하고 있습니다.

```python
rs_node_class=  'RealSenseNodeFactory'
rs_latency_tool_class = 'FrameLatencyNode'
...

def generate_launch_description():
  return LaunchDescription(declare_configurable_parameters(configurable_parameters) + [
    ComposableNodeContainer(
      name='my_container',
      namespace='',
      package='rclcpp_components',
      executable='component_container',
      composable_node_descriptions=[
        ComposableNode(
          package='realsense2_camera',
          namespace='',
          plugin='realsense2_camera::' + rs_node_class,
          name="camera",
          parameters=[set_configurable_parameters(configurable_parameters)],
          extra_arguments=[{'use_intra_process_comms': LaunchConfiguration("intra_process_comms")}]) ,
        ComposableNode(
          package='realsense2_camera',
          namespace='',
          plugin='rs2_ros::tools::frame_latency::' + rs_latency_tool_class,
          name='frame_latency',
          parameters=[set_configurable_parameters(configurable_parameters)],
          extra_arguments=[{'use_intra_process_comms': LaunchConfiguration("intra_process_comms")}]) ,
        ],
      output='screen',
      emulate_tty=True, # needed for display of logs
      arguments=['--ros-args', '--log-level', LaunchConfiguration('log_level')],
  )])
```

- `RealSenseNodeFactory`는 애초에 composition 형태로 개발되었으므로 실제 코드를 살펴보아도 로그를 남기는 것외에 추가 작업은 없습니다.

```cpp
BaseRealSenseNode::BaseRealSenseNode(rclcpp::Node& node,
                                     rs2::device dev,
                                     std::shared_ptr<Parameters> parameters,
                                     bool use_intra_process) :
    ...
    _use_intra_process(use_intra_process),
    ...
    {
    if ( use_intra_process )
    {
        ROS_INFO("Intra-Process communication enabled");
    }
```

- FrameLatencyNode는 tools ⇒ frame_latency에 구현되어 있으며, 단순히 topic publish의 지연 시간을 계산하여 콘솔 출력합니다.

```cpp
FrameLatencyNode::FrameLatencyNode( const rclcpp::NodeOptions & node_options )
    : Node( "frame_latency", "/", node_options )
    , _logger( this->get_logger() )
{
    ROS_INFO_STREAM( "frame_latency node is UP!" );
    ROS_INFO_STREAM( "Intra-Process is "
                     << ( this->get_node_options().use_intra_process_comms() ? "ON" : "OFF" ) );
    // Create a subscription on the input topic.
    _sub = this->create_subscription< sensor_msgs::msg::Image >(
        "/color/image_raw",  // TODO Currently color only, we can declare and accept the required
                             // streams as ros parameters
        rclcpp::QoS( rclcpp::QoSInitialization::from_rmw( rmw_qos_profile_default ),
                     rmw_qos_profile_default ),
        [&, this]( const sensor_msgs::msg::Image::SharedPtr msg ) {
            rclcpp::Time curr_time = this->get_clock()->now();
            auto latency = ( curr_time - msg->header.stamp ).seconds();
            ROS_INFO_STREAM( "Got msg with address 0x"
                             << std::hex << reinterpret_cast< std::uintptr_t >( msg.get() )
                             << std::dec << " with latency of " << latency << " [sec]" );
        } );
}
```

### rs_multi_camera_launch.py

- rs_multi_camera_launch에서는 namespace가 지정된 rs_launch.py 2개가 실행되며, rviz2 시각화를 위해 별도로 static_transform_publisher가 실행됩니다.

```python
local_parameters = [{'name': 'camera_name1', 'default': 'camera1', 'description': 'camera unique name'},
                    {'name': 'camera_name2', 'default': 'camera2', 'description': 'camera unique name'},
                   ]
...
def generate_launch_description():
  params1 = duplicate_params(rs_launch.configurable_parameters, '1')
  params2 = duplicate_params(rs_launch.configurable_parameters, '2')
  return LaunchDescription(
    rs_launch.declare_configurable_parameters(local_parameters) +
    rs_launch.declare_configurable_parameters(params1) +
    rs_launch.declare_configurable_parameters(params2) +
    [
    IncludeLaunchDescription(
      PythonLaunchDescriptionSource([ThisLaunchFileDir(), '/rs_launch.py']),
      launch_arguments=set_configurable_parameters(params1).items(),
    ),
    IncludeLaunchDescription(
      PythonLaunchDescriptionSource([ThisLaunchFileDir(), '/rs_launch.py']),
      launch_arguments=set_configurable_parameters(params2).items(),
    ),
    # dummy static transformation from camera1 to camera2
    launch_ros.actions.Node(
      package = "tf2_ros",
      executable = "static_transform_publisher",
      arguments = ["0", "0", "0", "0", "0", "0", "camera1_link", "camera2_link"]
    ),
  ])
```

- rs_multi_camera_launch.py에서 선언된 camera_name launch argument는 rs_launch.py로 전달되어 namespace 옵션에 추가됨과 더불어 Composotion에게 전달됩니다.

```python
return LaunchDescription(declare_configurable_parameters(configurable_parameters) + [
  # Realsense
  launch_ros.actions.Node(
    condition=IfCondition(PythonExpression([LaunchConfiguration('config_file'), " == ''"])),
    package='realsense2_camera',
    namespace=LaunchConfiguration("camera_name"),
    name=LaunchConfiguration("camera_name"),
    executable='realsense2_camera_node',
    parameters=[set_configurable_parameters(configurable_parameters)],
    output='screen',
    arguments=['--ros-args', '--log-level', LaunchConfiguration('log_level')],
    emulate_tty=True,
  ),
```

- parameter를 모두 관리하는 parameters.cpp에서 camera_name의 파싱을 확인할 수 있습니다.

```cpp
void BaseRealSenseNode::getParameters()
{
    ROS_INFO("getParameters...");

    std::string param_name;
    param_name = std::string("camera_name");
    _camera_name = _parameters->setParam<std::string>(param_name, "camera");
    _parameters_names.push_back(param_name);
```

## realsense2 Description

> description이라는 이름을 가진 패키지는 CAD, URDF 파일과 같은 물성치, 외관에 대한 데이터를 담고 있습니다. realsense의 description package를 분석해봅시다.

- 기본 launch file을 실행해 보겠으며, d455 옵션으로 실행해보겠습니다.

```cpp
ros2 launch realsense2_description view_model.launch.py model:=<sth>
ros2 launch realsense2_description view_model.launch.py model:=test_d455_camera.urdf.xacro
```

![Untitled8.png](/kr/ros2_foxy/images17/Untitled8.png?height=300px)

⇒ 센서 외관을 비롯하여 다양한 내장 센서들에 대한 tf2도 확인할 수 있습니다.

- tf2 tree를 통해 이에 대한 자세한 구조를 파악할 수 있으며, d455는 카메라 뿐만 아니라 가속도 센서, 자이로 센서를 포함하고 있어 아래와 같이 복잡한 구조를 갖게 됩니다.

![Untitled9.png](/kr/ros2_foxy/images17/Untitled9.png?height=400px)

> 지금 tf2 tree에서는 base_link가 최상위 Node인데요. 실제 로봇 시스템에 장착 시 로봇의 base_link가 이미 존재한다면 의도치 않은 tf2 error가 발생할 수 있습니다.

- 실제 test_d455_camera.urdf.xacro 파일에서도 이를 손쉽게 제거할 수 있도록 아래와 같이 base_link를 별도 분리해둔 모습을 볼 수 있습니다.

```xml
<?xml version="1.0"?>
<robot name="realsense2_camera" xmlns:xacro="http://ros.org/wiki/xacro">
  <xacro:arg name="use_nominal_extrinsics" default="false"/>
  <xacro:include filename="$(find realsense2_description)/urdf/_d455.urdf.xacro" />

  <link name="base_link" />
  <xacro:sensor_d455 parent="base_link" use_nominal_extrinsics="$(arg use_nominal_extrinsics)">
    <origin xyz="0 0 0" rpy="0 0 0"/>
  </xacro:sensor_d455>
</robot>
```

### Description Launch file 분석

- view_model.launch.py - 실제 실행되는 Node는 robot_state_publisher와 rviz2 입니다. (센서는 움직이는 파츠가 없으니 joint_state_publisher는 굳이 필요 없겠지요?)

```python
rviz_node = Node(
    package='rviz2',
    executable='rviz2',
    node_name='rviz2',
    output = 'screen',
    arguments=['-d', rviz_config_dir],
    parameters=[{'use_sim_time': False}]
    )
model_node = Node(
    node_name='model_node',
    package='robot_state_publisher',
    executable='robot_state_publisher',
    namespace='',
    output='screen',
    arguments = [urdf]
    )
return launch.LaunchDescription([rviz_node, model_node])
```

{{% notice note %}}
robot_state_publisher의 argument로 urdf 파일들이 전달되며, 사용자로부터 mode:= 값을 받아 해당 urdf file이 실행되는 형식입니다. (저라면 Launch Argument를 사용했을 것 같습니다.)
{{% /notice %}}

- urdf 폴더를 살펴보면, 모든 모델에 대한 기본 urdf file과, description launch를 위해 base_link가 결합된 test urdf들이 자동생성되어있는 모습을 확인할 수 있습니다.

![Untitled10.png](/kr/ros2_foxy/images17/Untitled10.png?height=300px)

- 마지막으로 카메라 센서의 tf2 사용 시 주의할 점이 있는데요. 일반적으로 컴퓨터 비전 도메인에서 사용하는 좌표계와, ROS 2에서 사용하는 좌표계는 아래와 같이 차이를 갖습니다. gazebo plugin 시간에 살펴보았던 것처럼 rviz2를 통해 검증을 한 뒤 사용하기를 추천드립니다.

![Untitled11.png](/kr/ros2_foxy/images17/Untitled11.png?height=300px)

### Custom Message

- 대부분 센서 패키지들은 sdk에서 정의된 자료구조와 ROS 2의 연동을 위해 custom interface를 구현합니다. realsense2에서도 IMUInfo를 비롯한 Custom Interface들을 제공하고 있습니다.

```python
$ ros2 interface show realsense2_camera_msgs/msg/IMUInfo
# header.frame_id is either set to "imu_accel" or "imu_gyro"
# to distinguish between "accel" and "gyro" info.
std_msgs/Header header
float64[12] data
float64[3] noise_variances
float64[3] bias_variances
```

{{% notice note %}}
⇒ 이렇게 custom interface가 있다면, 다른 패키지의 종속성이 될 확률이 높이 때문에 먼저 빌드해줘야 합니다.

{{% /notice %}}

{{% notice note %}}
⇒ 더불어 custom interface는 해당 workspace에만 적용되기 때문에 workspace가 바뀌면 적용되지 않습니다.

{{% /notice %}}

- CMakeLists.txt를 통해 빌드되는 모든 IDL들을 확인할 수 있습니다.

```cpp
set(msg_files
  "msg/IMUInfo.msg"
  "msg/Extrinsics.msg"
  "msg/Metadata.msg"
)
rosidl_generate_interfaces(${PROJECT_NAME}
  ${msg_files}
  "srv/DeviceInfo.srv"
  DEPENDENCIES builtin_interfaces std_msgs
  ADD_LINTER_TESTS
)
```

- 일전 이야기한 것과 같이 다른 소스코드에서 custom interface를 사용하고 있어 선행 빌드 후 소싱이 꼭 필요합니다.

```cpp
#pragma once

#include <librealsense2/rs.hpp>
#include <librealsense2/rsutil.h>
#include "constants.h"
#include <cv_bridge/cv_bridge.h>

#include <diagnostic_updater/diagnostic_updater.hpp>
#include <diagnostic_updater/publisher.hpp>
#include "realsense2_camera_msgs/msg/imu_info.hpp"
#include "realsense2_camera_msgs/msg/extrinsics.hpp"
#include "realsense2_camera_msgs/msg/metadata.hpp"
#include "realsense2_camera_msgs/srv/device_info.hpp"
```

## velodyne Lidar ROS 2 실행

이번에는 point cloud 데이터를 센싱할 수 있는 3D Lidar 중 가장 보편적으로 사용되는 VLP-16의 ROS 2 패키지를 분석해보겠습니다.

![Untitled12.png](/kr/ros2_foxy/images17/Untitled12.png?height=300px)

- velodyne lidar 또한 ubuntu package를 제공하고 있으며 자체적인 Gazebo package와 driver 패키지를 모두 제공합니다.

```cpp
$ sudo apt install ros-foxy-velodyne-
ros-foxy-velodyne-description            ros-foxy-velodyne-laserscan-dbgsym
ros-foxy-velodyne-driver                 ros-foxy-velodyne-msgs
ros-foxy-velodyne-driver-dbgsym          ros-foxy-velodyne-msgs-dbgsym
ros-foxy-velodyne-gazebo-plugins         ros-foxy-velodyne-pointcloud
ros-foxy-velodyne-gazebo-plugins-dbgsym  ros-foxy-velodyne-pointcloud-dbgsym
ros-foxy-velodyne-laserscan              ros-foxy-velodyne-simulator
```

- 이번에도 우리는 학습을 위해 소스코드 빌드를 진행하겠습니다.

```bash
cd ~/ros2_ws/src

# package clone
git clone -b foxy-devel https://github.com/ros-drivers/velodyne.git

# dependencies install
sudo apt-get update && sudo apt-get install libpcap-dev

# source code build
cbp velodyne_msgs && source install/local_setup.bash
cbp velodyne_laserscan && source install/local_setup.bash
cbp velodyne_pointcloud && source install/local_setup.bash
cbp velodyne_driver && source install/local_setup.bash
cbp velodyne && source install/local_setup.bash
```

- velodyne lidar는 Ethernet 인터페이스를 제공합니다. 따라서 Ubuntu의 네트워크 설정에서 고정 IP 설정을 해주며, VLP-16의 기본 IP는 **192.168.1.100**입니다.

![Untitled13.png](/kr/ros2_foxy/images17/Untitled13.png?height=300px)

- 설정 이후 launch는 다음과 같습니다.

```cpp
ros2 launch velodyne velodyne-all-nodes-VLP16-launch.py
```

- launch rviz2를 통한 시각화를 해봅시다. fixed frame을 velodyne으로 설정해야 합니다.

![velo1.png](/kr/ros2_foxy/images17/velo1.png?height=300px)

- tf2 tree를 살펴보면 아무것도 조회되지 않습니다. velodyne tf2만 broadcast되고 있기 때문에 확인할 수 없는 것입니다.

{{% notice tip %}}
저였다면 stasic transform broadcaster를 통해 world > velodyne tf2를 하나 생성했을 것 같습니다.
{{% /notice %}}

![Untitled14.png](/kr/ros2_foxy/images17/Untitled14.png?height=300px)

- velodyne lidar 또한 composed-launch라는 comspoition 기반 실행을 지원합니다.

```bash
$ ros2 launch velodyne velodyne-all-nodes-VLP16-composed-launch.py
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/velodyne_driver_node' in container '/velodyne_container'
[component_container-1] [INFO] [1682566114.537928654] [velodyne_container]: Load Library: /home/kimsooyoung/ros2_ws/install/velodyne_pointcloud/lib/libconvert.so
[component_container-1] [INFO] [1682566114.540579179] [velodyne_container]: Found class: rclcpp_components::NodeFactoryTemplate<velodyne_pointcloud::Convert>
[component_container-1] [INFO] [1682566114.540601962] [velodyne_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<velodyne_pointcloud::Convert>
[component_container-1] [INFO] [1682566114.542606795] [velodyne_convert_node]: correction angles: /home/kimsooyoung/ros2_ws/install/velodyne_pointcloud/share/velodyne_pointcloud/params/VLP16db.yaml
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/velodyne_convert_node' in container '/velodyne_container'
[component_container-1] [INFO] [1682566114.548422161] [velodyne_container]: Load Library: /home/kimsooyoung/ros2_ws/install/velodyne_laserscan/lib/libvelodyne_laserscan.so
[component_container-1] [INFO] [1682566114.549147973] [velodyne_container]: Found class: rclcpp_components::NodeFactoryTemplate<velodyne_laserscan::VelodyneLaserScan>
[component_container-1] [INFO] [1682566114.549171818] [velodyne_container]: Instantiate class: rclcpp_components::NodeFactoryTemplate<velodyne_laserscan::VelodyneLaserScan>
[INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/velodyne_laserscan_node' in container '/velodyne_container'
[component_container-1] [INFO] [1682566114.597621115] [velodyne_laserscan_node]: Latched ring count of 16
[component_container-1] [WARN] [1682566114.597755427] [velodyne_laserscan_node]: PointCloud2 fields in unexpected order. Using slower generic method.
```

- composed-launch와 일반 launch 시 차이점을 rqt_graph로 확인해봅시다.

![velo_rqt.png](/kr/ros2_foxy/images17/velo_rqt.png?height=300px)

![veloc_rqt.png](/kr/ros2_foxy/images17/veloc_rqt.png?height=300px)

### Launch file 분석

- `velodyne-all-nodes-VLP16-launch.py`를 분석해봅시다. velodyne_driver_node, velodyne_convert_node, velodyne_laserscan_node이 실행되며 velodyne_driver_node의 종료 시 launch 자체가 종료되도록 Event를 걸어주었습니다.

```python
return launch.LaunchDescription([
    velodyne_driver_node,
    velodyne_convert_node,
    velodyne_laserscan_node,

    launch.actions.RegisterEventHandler(
        event_handler=launch.event_handlers.OnProcessExit(
            target_action=velodyne_driver_node,
            on_exit=[launch.actions.EmitEvent(
                event=launch.events.Shutdown())],
        )),
    ])
```

- `velodyne-all-nodes-VLP16-composed-launch.py`에서는 component_container와 velodyne_driver::VelodyneDriver, velodyne_pointcloud::Convert, velodyne_laserscan::VelodyneLaserScan 3개의 composition이 동작합니다.

```python
	container = ComposableNodeContainer(
	  name='velodyne_container',
	  namespace='',
	  package='rclcpp_components',
	  executable='component_container',
	  composable_node_descriptions=[
	    ComposableNode(
	      package='velodyne_driver',
	      plugin='velodyne_driver::VelodyneDriver',
	      name='velodyne_driver_node',
	      parameters=[driver_params]),
	    ComposableNode(
	      package='velodyne_pointcloud',
	      plugin='velodyne_pointcloud::Convert',
	      name='velodyne_convert_node',
	      parameters=[convert_params]),
	    ComposableNode(
	      package='velodyne_laserscan',
	      plugin='velodyne_laserscan::VelodyneLaserScan',
	      name='velodyne_laserscan_node',
	      parameters=[laserscan_params]),
	  ],
	  output='both',
	)

	return LaunchDescription([container])
```

> 이제 방금 전의 실행 코드들(velodyne_driver_node, velodyne_convert_node, velodyne_laserscan_node)을 분석해 봅시다.

### velodyne_driver_node

- velodyne package의 `CMakeLists.txt`에서 velodyne_driver_node의 코드를 확인할 수 있습니다.

```python
add_executable(velodyne_driver_node src/driver/velodyne_node.cpp)
ament_target_dependencies(velodyne_driver_node rclcpp)
target_link_libraries(velodyne_driver_node velodyne_driver)
install(TARGETS velodyne_driver_node DESTINATION lib/${PROJECT_NAME})
```

- velodyne_node.cpp 자체는 일반적인 rclcpp node programming이 구현되어 있습니다. (대신 NodeOptions을 매개변수로 전달하는데 이는 VelodyneDriver가 Composition형태로 구현되어 있기 때문입니다.)

```cpp
#include <rclcpp/rclcpp.hpp>
#include <memory>
#include "velodyne_driver/driver.hpp"

int main(int argc, char ** argv)
{
  // Force flush of the stdout buffer.
  setvbuf(stdout, nullptr, _IONBF, BUFSIZ);

  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<velodyne_driver::VelodyneDriver>(rclcpp::NodeOptions()));
  rclcpp::shutdown();

  return 0;
}
```

- `VelodyneDriver::VelodyneDriver`에서는 velodyne sdk => ROS 2로의 형태 변환이 이루어집니다. Composition을 고려한 형태로 Node 프로그래밍 되어있습니다.

```cpp
namespace velodyne_driver
{
class VelodyneDriver final : public rclcpp::Node
{
public:
  explicit VelodyneDriver(const rclcpp::NodeOptions & options);
  ~VelodyneDriver() override;
  ...

private:
  bool poll();
  void pollThread();

  // configuration parameters
  struct config_;
  ...
  std::unique_ptr<Input> input_;
  rclcpp::Publisher<velodyne_msgs::msg::VelodyneScan>::SharedPtr output_;
  int last_azimuth_;
  ...
};
}  // namespace velodyne_driver

#endif  // VELODYNE_DRIVER__DRIVER_HPP_
```

- velodyne sdk => ROS 2로의 데이터 형태 변환은 `std::thread` 통해 구현되어 있습니다. ROS 2와의 충돌을 막기 위해 소멸자에서 join하는 방식을 사용하였습니다.

{{% notice note %}}
사실 이렇게 하면 이벤트 발생 시 다시 deadlock 문제가 발생합니다. 저라면 일전의 callback group & executor를 사용했을 것 같습니다.
{{% /notice %}}

```cpp
std::thread poll_thread_;

// node 생성자에서 thread 시작
VelodyneDriver::VelodyneDriver(const rclcpp::NodeOptions & options)
: rclcpp::Node("velodyne_driver_node", options),
  diagnostics_(this, 0.2)
{
  poll_thread_ = std::thread(&VelodyneDriver::pollThread, this);
}

// thread 함수에서 무한 poll 호출
void VelodyneDriver::pollThread()
{
  std::future_status status;

  do {
    poll();
    status = future_.wait_for(std::chrono::seconds(0));
  } while (status == std::future_status::timeout);
}

// poll 함수가 마치 ROS 2 callback 처럼 동작
bool VelodyneDriver::poll()
```

- velodyne_driver => ROS 2로의 데이터 변환을 정리해보았습니다. (poll 함수의 내용입니다.)

1. velodyne driver 데이터를 std::unique_ptr<Input> input으로 파싱
2. input에서 config으로 데이터 변환
3. config\_에서 std::unique_ptr<velodyne_msgs::msg::VelodyneScan> scan으로 데이터 변환
4. output\_->publish(std::move(scan));

{{% notice tip %}}
⇒ 상당히 메모리 형변환이 잦은데 이전 코드에서의 레거시가 있는 것 같습니다.
{{% /notice %}}

- diagnostic_updater ⇒ 중간중간 보이는 diagnostic 관련 코드들은 ROS 2의 diagnostic_updater API로 device drivers의 여러 상태를 topic 형태로 publish할 수 있는 기능입니다.

```cpp
diagnostic_updater::Updater diagnostics_;
std::unique_ptr<diagnostic_updater::TopicDiagnostic> diag_topic_;
```

### velodyne_convert_node

> rviz2에서 보았던 것처럼 제대로 된 Pointcloud topic을 위해서 velodyne_driver에서의 raw data들을 누적하고, 다시 sensor_msgs/msg/PointCloud2로 변환하는 작업이 필요합니다. velodyne_convert_node에서 이 내용이 구현되어 있습니다.

![velo1.png](/kr/ros2_foxy/images17/velo1.png?height=300px)

- velodyne_pointcloud package의 CMakeLists.txt에서 velodyne_convert_node의 코드를 확인할 수 있습니다.

```python
add_executable(velodyne_convert_node src/conversions/convert_node.cpp)
ament_target_dependencies(velodyne_convert_node rclcpp)
target_link_libraries(velodyne_convert_node convert)
install(TARGETS velodyne_convert_node
  DESTINATION lib/${PROJECT_NAME}
)
```

- convert_node.cpp 자체는 단순 Node 생성 후 spin을 담고 있습니다.

```cpp
int main(int argc, char ** argv)
{
  // Force flush of the stdout buffer.
  setvbuf(stdout, nullptr, _IONBF, BUFSIZ);

  rclcpp::init(argc, argv);

  // handle callbacks until shut down
  rclcpp::spin(
    std::make_shared<velodyne_pointcloud::Convert>(
      rclcpp::NodeOptions()));

  rclcpp::shutdown();

  return 0;
}
```

- Convert 클래스는 velodyne_msgs::msg::VelodyneScan topic을 subscribe하여 누적하고, sensor_msgs::msg::PointCloud2 topic으로 다시 publish합니다.

```cpp
namespace velodyne_pointcloud
{
class Convert final
  : public rclcpp::Node
{
public:
  explicit Convert(const rclcpp::NodeOptions & options);
...

private:
  void processScan(const velodyne_msgs::msg::VelodyneScan::SharedPtr scanMsg);

  std::unique_ptr<velodyne_rawdata::RawData> data_;
  rclcpp::Subscription<velodyne_msgs::msg::VelodyneScan>::SharedPtr velodyne_scan_;
  rclcpp::Publisher<sensor_msgs::msg::PointCloud2>::SharedPtr output_;
  tf2_ros::Buffer tf_buffer_;
  std::unique_ptr<velodyne_rawdata::DataContainerBase> container_ptr_;
...

};
}  // namespace velodyne_pointcloud
```

- velodyne_packets의 subscribe callback으로 processScan이 바인딩되어있으며, processScan에서는 일전 언급한 데이터 누적과 publish가 이루어집니다.

```cpp
// subscribe to VelodyneScan packets
velodyne_scan_ =
  this->create_subscription<velodyne_msgs::msg::VelodyneScan>(
  "velodyne_packets", rclcpp::QoS(10),
  std::bind(&Convert::processScan, this, std::placeholders::_1));

void Convert::processScan(const velodyne_msgs::msg::VelodyneScan::SharedPtr scanMsg)
{
  ...
  // allocate a point cloud with same time and frame ID as raw data
  container_ptr_->setup(scanMsg);

  // process each packet provided by the driver
  for (size_t i = 0; i < scanMsg->packets.size(); ++i) {
    data_->unpack(scanMsg->packets[i], *container_ptr_);
  }

  // publish the accumulated cloud message
  diag_topic_->tick(scanMsg->header.stamp);
  output_->publish(container_ptr_->finishCloud());
}
```

- velodyne_convert_node에서 tf2 데이터를 다루는 부분이 구현되어 있습니다. 하지만 편의를 위해 별도의 컨테이너를 만들고 구현은 추상화해둔 모습을 확인할 수 있습니다.

```cpp
tf2_ros::Buffer tf_buffer_;

if (organize_cloud) {
  container_ptr_ = std::make_unique<OrganizedCloudXYZIR>(
    min_range, max_range, target_frame, fixed_frame,
    data_->numLasers(), data_->scansPerPacket(), tf_buffer_);
} else {
  container_ptr_ = std::make_unique<PointcloudXYZIR>(
    min_range, max_range, target_frame, fixed_frame,
    data_->scansPerPacket(), tf_buffer_);
}
```

- DataContainerBase 클래스에서는 tf2 lookupTransform을 통해 scan data의 좌표를 미리 계산해둡니다.

```cpp
std::unique_ptr<velodyne_rawdata::DataContainerBase> container_ptr_;

class DataContainerBase
{
public:
  ...
	void computeTransformation(const rclcpp::Time & time)
	  {
	    geometry_msgs::msg::TransformStamped transform;
	    try {
	      const std::chrono::nanoseconds dur(time.nanoseconds());
	      std::chrono::time_point<std::chrono::system_clock, std::chrono::nanoseconds> time(dur);
	      transform = tf_buffer_.lookupTransform(config_.target_frame, cloud.header.frame_id, time);
	    } catch (tf2::LookupException & e) {
	      return;
	    } catch (tf2::ExtrapolationException & e) {
	      return;
	    }

	    tf2::Quaternion quaternion(
	      transform.transform.rotation.x,
	      transform.transform.rotation.y,
	      transform.transform.rotation.z,
	      transform.transform.rotation.w);
	    Eigen::Quaternionf rotation(quaternion.w(), quaternion.x(), quaternion.y(), quaternion.z());

	    Eigen::Vector3f eigen_origin;
	    tf2::Vector3 origin(
	      transform.transform.translation.x,
	      transform.transform.translation.y,
	      transform.transform.translation.z);
	    vectorTfToEigen(origin, eigen_origin);
	    Eigen::Translation3f translation(eigen_origin);
	    transformation = translation * rotation;
	  }
```

### velodyne_laserscan_node

> velodyne_laserscan_node는 sensor_msgs::msg::PointCloud2 data를 sensor_msgs::msg::LaserScan로 변환 후 /scan topic으로 publish 하는 node입니다. 3D pointcloud만 사용한다면 일반적으로 필요 없는 기능입니다.

- 클래스 header file을 통해 간단히 기능만 리뷰해봅시다.

```cpp
namespace velodyne_laserscan
{

class VelodyneLaserScan final
  : public rclcpp::Node
{
public:
  explicit VelodyneLaserScan(const rclcpp::NodeOptions & options);
  ...
private:
  void recvCallback(const sensor_msgs::msg::PointCloud2::SharedPtr msg);

  rclcpp::Subscription<sensor_msgs::msg::PointCloud2>::SharedPtr sub_;
  rclcpp::Publisher<sensor_msgs::msg::LaserScan>::SharedPtr pub_;

  uint16_t ring_count_{0};
  int ring_;
  double resolution_;
};

}  // namespace velodyne_laserscan
```

1. PointCloud2 topic을 subscribe 받고
2. 이 데이터를 LaserScan으로 변환합니다. (subscribe callback)
3. 마지막으로 변환된 데이터를 scan topic으로 다시 publish합니다.

- 코드를 분석하면서 보시다시피 모든 Node 구현은 Composition을 고려하여 만들어졌습니다. 따라서 CMakeLists.txt를 보면 아래와 같이 rclcpp_components_register_nodes 키워드들을 통해 같은 코드로 Composition을 생성할 수 있습니다.

```cpp
target_link_libraries(velodyne_driver velodyne_input)

# install runtime and library files
install(TARGETS velodyne_driver
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

rclcpp_components_register_nodes(velodyne_driver "velodyne_driver::VelodyneDriver")
```

### velodyne_msgs

> velodyne_msgs 패키지에는 VelodynePacket, VelodyneScan라는 두 종류의 custom message를 정의하고 있습니다. 이는 주로 velodyne 자체의 프로토콜 패킷을 다루기 위한 것으로 ROS 2 데이터 변환 시 중간 데이터 구조로 사용됩니다.

```bash
> VelodynePacket.msg
# Raw Velodyne LIDAR packet.

builtin_interfaces/Time stamp              # packet timestamp
uint8[1206] data        # packet contents

> VelodyneScan.msg
# Velodyne LIDAR scan packets.

std_msgs/Header header         # standard ROS message header
VelodynePacket[] packets        # vector of raw packets
```

## orbbec astra camera ROS 2 패키지 분석

![Untitled15.png](/kr/ros2_foxy/images17/Untitled15.png?height=300px)

이번 시간에는 Orbbec Astra+ Development Kit의 ROS 2 실행과 분석을 진행해보겠습니다. 실제 사용될 카메라와 코드이므로 좀 더 자세하게 분석해보려 합니다. 이전 센서 패키지들과 비교하면서 따라와 주세요!

### 개발환경 설정

- 필요한 apt package들과 ROS 2 package들을 설치합니다.

```bash
sudo apt install libgflags-dev nlohmann-json3-dev
sudo apt install ros-foxy-image-transport ros-foxy-image-publisher
```

- 빌드 종속성인 glog을 설치합니다.

```bash
wget -c https://github.com/google/glog/archive/refs/tags/v0.6.0.tar.gz  -O glog-0.6.0.tar.gz
tar -xzvf glog-0.6.0.tar.gz
cd glog-0.6.0
mkdir build && cd build
cmake .. && make -j4
sudo make install
sudo ldconfig  # Refreshing the link library
```

- 다음으로, 빌드 종속성인 magic_enum을 설치합니다.

```bash
wget -c https://github.com/Neargye/magic_enum/archive/refs/tags/v0.8.0.tar.gz -O  magic_enum-0.8.0.tar.gz

tar -xzvf magic_enum-0.8.0.tar.gz
cd magic_enum-0.8.0
mkdir build && cd build
cmake .. && make -j4
sudo make install
sudo ldconfig # Refreshing the link library
```

이번 실습을 위해 사용된 코드는 2022-07-15 버전 Orbbec SDK Beta for ROS입니다. ( 이후 업데이트 시 코드 변경이 있을 수 있어 명확히 명시하겠습니다. ⇒ [download link](https://orbbec3d.com/index/download.html))

![Untitled16.png](/kr/ros2_foxy/images17/Untitled16.png?height=300px)

- 다운로드 받은 source code를 원하는 workspace에 위치시킨 뒤, 개발문서에서 요구하는 usb rule 추가 작업을 진행합니다.

```bash
cd orbbec_camera/scripts
sudo bash install.sh
sudo udevadm control --reload-rules && sudo udevadm trigger
```

- 해당 작업 후 혹시 모를 오류를 방지하기 위해 재부팅을 권장드리며, 카메라를 연결한 뒤 장치 인식이 되는지 확인해봅시다.

```bash
$ ls /dev | grep Astra
Astra+
Astra+_rgb
```

- 소스코드를 빌드하며, 공식 문서에서는 Release build를 제시하고 있어 그대로 진행해 보겠습니다.

```bash
cd ~/ros2_ws/src
colcon build --event-handlers  console_direct+  --cmake-args  -DCMAKE_BUILD_TYPE=Release
```

{{% notice note %}}
빌드 중 오류가 발생했다면 패키지를 하나하나씩 빌드하며 원인을 분석해봅니다.
{{% /notice %}}

```bash
colcon build --packages-select orbbec_camera_msgs --event-handlers  console_direct+  --cmake-args  -DCMAKE_BUILD_TYPE=Release
source install/local_setup.bash
colcon build --packages-select orbbec_camera --event-handlers  console_direct+  --cmake-args  -DCMAKE_BUILD_TYPE=Release
source install/local_setup.bash
```

### Getting started

> 빌드가 완료되었다면 기본 예시를 사용해봅시다.

- 제공되는 초기 코드 parameter가 아닌, default parameter를 사용하겠습니다. launch file에서 아래와 같이 parameter를 설정을 제거하고 실행합시다.

```python
from launch import LaunchDescription
from launch_ros.actions import Node
from ament_index_python import get_package_share_directory

def generate_launch_description():
    ob_params_file = (
        get_package_share_directory("orbbec_camera") + "/params/astra_plus_params.yaml"
    )
    return LaunchDescription(
        [
            Node(
                package="orbbec_camera",
                namespace="camera",
                name="camera",
                executable="orbbec_camera_node",
                output="screen",
                # parameters=[ob_params_file],
            ),
        ]
    )
```

- astra_plus.launch 예제를 실행해봅시다.

```python
cd <my_ws>
source install/local_setup.bash
ros2 launch orbbec_camera astra_plus.launch.py
...
[orbbec_camera_node-1] [WARN] [1683197732.405664220] [camera.camera]: Using default profile instead.
[orbbec_camera_node-1] [WARN] [1683197732.405680938] [camera.camera]: default FPS 15
[orbbec_camera_node-1] [INFO] [1683197732.413525012] [camera.camera]:  stream color is enabled - width: 2048, height: 1536, fps: 30, Format: OB_FORMAT_MJPG
[orbbec_camera_node-1] [WARN] [1683197732.427355527] [camera.camera]: Publishing dynamic camera transforms (/tf) at 10 Hz
```

{{% notice note %}}
custom interface들을 사용하기 때문에 workspace sourcing이 반드시 필요합니다.
{{% /notice %}}

- publish되고 있는 topic들은 다음과 같습니다.

```bash
$ ros2 topic list
/camera/color/camera_info
/camera/color/image_raw
/camera/depth/camera_info
/camera/depth/color/points
/camera/depth/image_raw
/camera/depth/points
/camera/extrinsic/depth_to_color
/camera/ir/camera_info
/camera/ir/image_raw
/clicked_point
/goal_pose
/initialpose
/parameter_events
/rosout
/tf
/tf_static
```

- 특이한 점으로, 카메라 사이 좌표계 값을 depth_to_color라는 이름의 topic으로 publish하고 있습니다.

```python
$ ros2 topic echo --qos-durability=transient_local /camera/extrinsic/depth_to_color  --qos-profile=services_default
header:
  stamp:
    sec: 0
    nanosec: 0
  frame_id: depth_to_color_extrinsics
rotation:
- 0.9999811053276062
- -0.006122155115008354
- -0.0005165112670511007
- 0.006124507170170546
- 0.9999702572822571
- 0.004682439845055342
- 0.0004878292966168374
- -0.004685514606535435
- 0.9999889135360718
translation:
- -25.19551658630371
- -0.30450138449668884
- 1.0939441919326782
---
```

- rviz2를 통해 데이터를 시각화해봅시다.

![Untitled17.png](/kr/ros2_foxy/images17/Untitled17.png?height=300px)

{{% notice note %}}
주의해야 할 점으로, pointcloud2 topic의 DDS QoS를 아래와 같이 잘 맞춰주어야 합니다.
{{% /notice %}}

![Untitled18.png](/kr/ros2_foxy/images17/Untitled18.png?height=200px)

{{% notice note %}}
더불어, color pointcloud2 topic의 색상을 확인하기 위해서는 rviz2 옵션을 아래와 같이 맞춰주어야 합니다.
{{% /notice %}}

![Untitled19.png](/kr/ros2_foxy/images17/Untitled19.png?height=200px)

> 혹은, 제가 제공드리는 rviz파일을 사용하여 configuration 하셔도 좋습니다. ⇒ [astra rviz download](https://drive.google.com/file/d/1heFUJDr7Q2N9jWhP_jYCV9EG_KTUjiQq/view?usp=sharing)

- 카메라 센서인 만큼 tf2를 다룰 시 조심해야 할 부분이 있습니다. tf2 tree는 다음과 같습니다.

![Untitled20.png](/kr/ros2_foxy/images17/Untitled20.png?height=300px)

- rviz2를 통해 확인해보면, optical frame들은 회전된 camera 좌표 체계를 갖는 것을 확인 가능합니다.

![Untitled21.png](/kr/ros2_foxy/images17/Untitled21.png?height=300px)

- 마지막으로, astra node의 parameter들을 조회해봅시다.

```bash
$ ros2 param list
/camera/camera:
  camera_link_frame_id
  color_format
  color_fps
  color_frame_id
  color_height
  color_optical_frame_id
  color_width
  d2c_mode
  ...

$ ros2 param get /camera/camera color_format
String value is: YUYV
```

> 해당 parameter들에 따라 코드 실행에 어떤 변화가 있는지 분석을 통해 살펴보겠습니다.

### 코드 분석

- launch file을 통해 확인했던 orbbec_camera_node가 어떻게 생성되었는지 추적해봅시다.

```python
Node(
    package="orbbec_camera",
    namespace="camera",
    name="camera",
    executable="orbbec_camera_node",
    output="screen",
    # parameters=[ob_params_file],
),
```

- CMakeLists.txt를 살펴보면 rclcpp_components_register_node를 통해 생성되는 orbbec_camera_node를 확인할 수 있습니다. (component와 executable을 모두 빌드하기 위함입니다.)

```bash
ament_target_dependencies(${PROJECT_NAME}
  ${dependencies}
  )
rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "orbbec_camera::OBCameraNodeFactory"
  EXECUTABLE orbbec_camera_node
)
```

- orbbec_camera_node을 구성하는 코드들은 다음과 같습니다.

```bash
add_library(${PROJECT_NAME} SHARED
  src/dynamic_params.cpp
  src/ob_camera_node_factory.cpp
  src/ob_camera_node.cpp
  src/ros_param_backend.cpp
  src/ros_service.cpp
  src/utils.cpp
)
```

- File별 기능을 분석하면 다음과 같습니다.

| Code                   | Description                                                                                                                                                                               |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ros_param_backend      | parameter 변경 시 특정 callback이 실행되도록 binding 해주는 decorator입니다.                                                                                                              |
| dynamic_params         | ROS 2에서 제공되는 기본적인 set_parameter와 유사한 역할을 수행하는 각종 setParameter API들을 구현한 부분입니다. (아마도 별도의 parameter 변경 reject 로직을 구현하고 싶었던 것 같습니다.) |
| ob_camera_node         | 일반적인 ROS 2 Node의 로직과, orbbec sdk의 pipeline을 융합하여 구현한 코드입니다. 실질적인 topic publish, service server들이 구현되어 있습니다.                                           |
| ros_service            | service server들에 대한 cpp 구현을 별도로 분리한 코드입니다.                                                                                                                              |
| ob_camera_node_factory | ob_camera_node에서 구현한 OBCameraNode node를 포인터로 갖는 별도의 Component를 구현하였습니다. (이 부분은 ROS 2 구현 측면에서 다소 미숙한 부분이라고 볼 수 있습니다.)                     |

> 주요 구현은 ob_camera_node에 작성되어 있습니다. 해당 파일 내 메소드들을 호출 계층 구조에 따라 정리하면 아래와 같습니다.

- ob_camera_node

  - Constructor
    - setupTopics
      - getParameters
      - setupDevices
      - setupProfiles
      - setupDefaultStreamCalibData
        - findDefaultCameraParam
        - updateStreamCalibData
      - setupCameraCtrlServices
      - setupPublishers
      - publishStaticTransforms
    - startPipeline
      - ob::Pipeline 실행
      - frameSetCallback
        - publishColorFrame
        - publishDepthFrame
        - publishIRFrame
        - publishPointCloud
          - publishColorPointCloud
          - publishDepthPointCloud

- setupTopics에서 호출되는 함수들은 다음과 같은 역할을 갖습니다.

| Function                    | Description                                                                                                                                                                                                                                                   |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| getParameters               | dynamic_params에 구현된 setParam를 사용하여 각종 매개변수들을 파싱하고, 세팅합니다.                                                                                                                                                                           |
| setupDevices                | orbbec sdk를 사용하여 camera profile, stream data등에 대한 정보를 조회하고 저장합니다.                                                                                                                                                                        |
| setupProfiles               | getParameters와 setupDevices에서 얻은 정보를 바탕으로 orbbec 카메라를 셋업합니다. d2c_mode등 orbbec만의 고유 설정들을 셋업하는 코드가 구현되어 있습니다.                                                                                                      |
| setupDefaultStreamCalibData | 내부적으로 findDefaultCameraParam / updateStreamCalibData라는 메소드를 호출하며, 카메라 calibration data를 받아 셋업하는 부분입니다.                                                                                                                          |
| setupCameraCtrlServices     | topic이 아닌 service server들에 대한 선언과 callback mapping들이 이루어집니다.                                                                                                                                                                                |
| setupPublishers             | depth/color/points, depth/points, camera_info, extrinsic/depth_to_color, image_raw 등의 topic publisher들이 선언되고 세팅됩니다.                                                                                                                              |
| publishStaticTransforms     | dynamic*tf_broadcaster*와 static*tf_broadcaster*라는 두개의 tf2 broadcast가 구현되어 있습니다. dynamic*tf_broadcaster*의 경우 publishDynamicTransforms라는 메소드를 별도 thread로 실행합니다. (하지만 실제 카메라 내부에서 동적으로 변화하는 tf2는 없습니다.) |

- 대부분 설정과 관련된 구현이지만, ROS 2 구현 측면에서 setupPublishers를 살펴보겠습니다.

QoS profile을 적용한 topic publisher들과 tf broadcaster가 생성되고 있습니다.

```cpp
void OBCameraNode::setupPublishers() {
  static_tf_broadcaster_ = std::make_shared<tf2_ros::StaticTransformBroadcaster>(node_);
  using PointCloud2 = sensor_msgs::msg::PointCloud2;
  using CameraInfo = sensor_msgs::msg::CameraInfo;
  point_cloud_publisher_ = node_->create_publisher<PointCloud2>(
      "depth/color/points", rclcpp::QoS{1}.best_effort().keep_last(1));
  depth_point_cloud_publisher_ = node_->create_publisher<PointCloud2>(
      "depth/points", rclcpp::QoS{1}.best_effort().keep_last(1));
  for (const auto& stream_index : IMAGE_STREAMS) {
    std::string name = stream_name_[stream_index.first];
    std::string topic = name + "/image_raw";
    image_publishers_[stream_index] = image_transport::create_publisher(node_, topic);
    topic = name + "/camera_info";
    camera_info_publishers_[stream_index] =
        node_->create_publisher<CameraInfo>(topic, rclcpp::QoS{1}.best_effort());
  }
  extrinsics_publisher_ = node_->create_publisher<orbbec_camera_msgs::msg::Extrinsics>(
      "extrinsic/depth_to_color", rclcpp::QoS{1}.transient_local());
}
```

- publishStaticTransforms 메소드는 다음과 같습니다. dynamic transform이라는 구현이 보이는데, 사실 카메라 내부적으로는 움직이는 부분이 없습니다.

```cpp
void OBCameraNode::publishStaticTransforms() {
  calcAndPublishStaticTransform();
  if (tf_publish_rate_ > 0) {
    tf_thread_ = std::make_shared<std::thread>([this]() { publishDynamicTransforms(); });
  } else {
    static_tf_broadcaster_->sendTransform(static_tf_msgs_);
  }
}
```

- 코드 확인 결과, transform의 의미보다 시간 동기화를 위해 구현해둔 것으로 보입니다.

```cpp
void OBCameraNode::publishDynamicTransforms() {
  RCLCPP_WARN(logger_, "Publishing dynamic camera transforms (/tf) at %g Hz", tf_publish_rate_);
  std::mutex mu;
  std::unique_lock<std::mutex> lock(mu);
  while (rclcpp::ok() && is_running_) {
    tf_cv_.wait_for(lock, std::chrono::milliseconds((int)(1000.0 / tf_publish_rate_)),
                    [this] { return (!(is_running_)); });
    {
      rclcpp::Time t = node_->now();
      for (auto& msg : static_tf_msgs_) {
        msg.header.stamp = t;
      }
      dynamic_tf_broadcaster_->sendTransform(static_tf_msgs_);
    }
  }
}
```

- startPipeline에서 호출되는 함수들은 다음과 같은 역할을 갖습니다.

| Function          | Description                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------- |
| ob::Pipeline 실행 | orbbec sdk로부터 pipeline을 다루는 핸들러를 받아 image stream을 시작합니다.                       |
| frameSetCallback  | setupPublishers에서 설정한 각종 topic publisher들이 실질적인 topic publish를 수행하는 부분입니다. |

- 코드를 확인해보면, orbbec SDK의 pipeline을 실행한 뒤, frame callback과 binding하고 있는 모습을 확인 가능합니다.

```cpp
void OBCameraNode::startPipeline() {
  if (pipeline_ != nullptr) {
    pipeline_.reset();
  }
  pipeline_ = std::make_unique<ob::Pipeline>(device_);
  pipeline_->start(config_, [this](std::shared_ptr<ob::FrameSet> frame_set) {
    frameSetCallback(std::move(frame_set));
  });
}
```

- frame이 갱신될 떄마다 실행되는 frameSetCallback에서는 갱신된 이미지 데이터를 사용하여 각종 topic publish를 진행합니다.

```cpp
void OBCameraNode::frameSetCallback(std::shared_ptr<ob::FrameSet> frame_set) {
  auto color_frame = frame_set->colorFrame();
  auto depth_frame = frame_set->depthFrame();
  auto ir_frame = frame_set->irFrame();
  if (color_frame && enable_[COLOR]) {
    publishColorFrame(color_frame);
  }
  if (depth_frame && enable_[DEPTH]) {
    publishDepthFrame(depth_frame);
  }
  if (ir_frame && enable_[INFRA0]) {
    publishIRFrame(ir_frame);
  }
  publishPointCloud(frame_set);
}
```

- publishColorFrame / publishDepthFrame / publishIRFrame에서는 cv::Mat 형식의 데이터를 ROS 2 Image msg로 변환하여 publish하고 있습니다.

```cpp
void OBCameraNode::publishColorFrame(std::shared_ptr<ob::ColorFrame> frame) {
  ...
  sensor_msgs::msg::Image::SharedPtr img;
  img = cv_bridge::CvImage(std_msgs::msg::Header(), encoding_.at(stream), image).toImageMsg();

  img->width = width;
  img->height = height;
  img->is_bigendian = false;
  img->step = width * unit_step_size_[stream];
  img->header.frame_id = optical_frame_id_[COLOR];
  img->header.stamp = timestamp;
  auto& image_publisher = image_publishers_.at(stream);
  image_publisher.publish(img);
}
```

- publishPointCloud에서는 특정 조건에 따라 color pointcloud, depth point cloud를 준비시킵니다. 이 부분 때문에 parameter설정이 중요합니다.

```cpp
void OBCameraNode::publishPointCloud(std::shared_ptr<ob::FrameSet> frame_set) {
  try {
    if (align_depth_ && (format_[COLOR] == OB_FORMAT_YUYV || format_[COLOR] == OB_FORMAT_I420)) {
      if (frame_set->depthFrame() != nullptr && frame_set->colorFrame() != nullptr) {
        publishColorPointCloud(frame_set);
      }
    }
    if (frame_set->depthFrame() != nullptr) {
      publishDepthPointCloud(frame_set);
    }
```

- pointcloud를 publish하는 로직을 살펴보면 공통적으로 아래와 같은 iterator 구현을 확인할 수 있습니다. 이는 ROS 2의 PointCloud2 type에서 제공하는 API로 손쉽게 각 point data를 append 할 수 있습니다.

```cpp
  ...
  sensor_msgs::PointCloud2Iterator<float> iter_x(point_cloud_msg_, "x");
  sensor_msgs::PointCloud2Iterator<float> iter_y(point_cloud_msg_, "y");
  sensor_msgs::PointCloud2Iterator<float> iter_z(point_cloud_msg_, "z");
  size_t valid_count = 0;
  for (size_t point_idx = 0; point_idx < point_size; point_idx++, points++) {
    bool valid_pixel(points->z > 0);
    if (valid_pixel) {
      *iter_x = static_cast<float>(points->x / 1000.0);
      *iter_y = -static_cast<float>(points->y / 1000.0);
      *iter_z = static_cast<float>(points->z / 1000.0);
      ++iter_x;
      ++iter_y;
      ++iter_z;
      ++valid_count;
    }
  }
```

- 마지막으로 ros_service.cpp 코드에는 각종 설정을 service로 제어할 수 있는 로직이 구현되어 있습니다. 자세한 내용을 살펴보지는 않겠지만, service server를 설정하고 binding되는 callback을 구현하는 부분이 반복됩니다.

```cpp
service_name = "set_" + stream_name + "_exposure";
set_exposure_srv_[stream_index] = node_->create_service<SetInt32>(
    service_name,
    [this, stream_index = stream_index](const std::shared_ptr<SetInt32::Request> request,
                                        std::shared_ptr<SetInt32::Response> response) {
      setExposureCallback(request, response, stream_index);
    });
...

void OBCameraNode::setExposureCallback(const std::shared_ptr<SetInt32::Request>& request,
                                       std::shared_ptr<SetInt32::Response>& response,
                                       const stream_index_pair& stream_index) {
  auto stream = stream_index.first;
  ...
```

- OBCameraNodeFactory는 OBCameraNode, ob::Device, ob::DeviceInfo 인스턴스를 스마트 포인터로 포함하고 있습니다. 대부분의 구현들도 해당 스마트 포인터 내부의 메소드를 호출하는 식으로 구성됩니다.

```cpp
void init();
void startDevice();
void getDevice(const std::shared_ptr<ob::DeviceList>& list);
void updateDeviceInfo();
void deviceConnectCallback(const std::shared_ptr<ob::DeviceList>& device_list);
void deviceDisconnectCallback(const std::shared_ptr<ob::DeviceList>& device_list);
void printDeviceInfo(const std::shared_ptr<ob::DeviceInfo>& device_info);

std::unique_ptr<OBCameraNode> ob_camera_node_;
std::shared_ptr<ob::Device> device_;
std::shared_ptr<ob::DeviceInfo> device_info_;
```

{{% notice note %}}
이 부분은 구조적으로 다소 비효율적이라고 생각합니다. orbbec 관련 인스턴스를 클래스 변수로 갖는 것은 이해가 되지만, 저라면 Node를 스마트 포인터로 갖지 않고, Composition 형태로 바로 구현했을 것 같습니다.
{{% /notice %}}

> orbbec astra plus model의 ROS 2 패키지 분석을 끝으로 모든 예시 분석을 마쳤습니다. 실제 ROS 2 package 개발에 대한 안목이 생기셨으리라 생각합니다.
