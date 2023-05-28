---
title: "Lecture19. Orbbec ROS 2 Package - Final"
date: 2023-04-28T10:40:29+09:00
draft: false
---

> 강의 도중 (2022년 5월) Orbbec ROS 2 SDK가 업데이트됨에 따라 새롭게 분석을 진행해보았습니다.

{{% notice note %}}
강의에서 사용되는 패키지 버전은 아래 사진과 같이 2023-05-15 버전입니다.
{{% /notice %}}

![Untitled.png](/kr/ros2_foxy/images19/Untitled.png?height=200px)

> 새롭게 분석한다는 마음으로 launch file부터 차근차근 분석해나가겠습니다. 😂

- `gemini2.launch.xml` - xml 문법으로 구성된 launch file로 실질적으로 실행시키는 것은 orbbec_camera_node라는 이름의 단일 Node입니다.

```bash
<node name="camera" pkg="orbbec_camera" exec="orbbec_camera_node" output="screen">
```

- orbbec_camera_node가 생성되는 코드를 분석하기 위해 `CMakeLists.txt`를 살펴봅시다. main.cpp를 사용하고 있음이 확인 가능합니다.

```bash
add_executable(${PROJECT_NAME}_node
  src/main.cpp
  )
```

- `main.cpp`에서는 SingleThreaded Executor와 OBCameraNodeDriver라는 Node를 spin하고 있습니다.

```cpp
#include <rclcpp/rclcpp.hpp>
#include <orbbec_camera/ob_camera_node_driver.h>

int main(int argc, char** argv) {
  rclcpp::init(argc, argv);
  rclcpp::NodeOptions options;
  using namespace orbbec_camera;
  auto node = std::make_shared<OBCameraNodeDriver>(options);
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}
```

- OBCameraNodeDriver를 분석해보기 위해 해당 구현을 담고 있는 `ob_camera_node_driver.cpp` 파일을 살펴봅시다.

config file을 Node내에서 직접 불러오는 모습과, `init()` 메소드를 호출하고 있음이 확인됩니다.

```cpp
namespace orbbec_camera {
OBCameraNodeDriver::OBCameraNodeDriver(const rclcpp::NodeOptions &node_options)
    : Node("orbbec_camera_node", "/", node_options),
      config_path_(ament_index_cpp::get_package_share_directory("orbbec_camera") +
                   "/config/OrbbecSDKConfig_v1.0.xml"),
      ctx_(std::make_unique<ob::Context>(config_path_.c_str())),
      logger_(this->get_logger()) {
  init();
}
```

- `OrbbecSDKConfig_v1.0.xml`은 중국어로 주석이 되어있어 정확히 분석할 수는 없었지만, 각종 카메라들의 기본 설정값을 담고 있는 파일로 추측됩니다.

```xml
<Gemini>
    <Depth>
        <!--默认分辨率的宽，int类型-->
        <Width>640</Width>
        <!--默认分辨率的高，int类型-->
        <Height>400</Height>
        <!--默认帧率，int类型-->
        <FPS>30</FPS>
        <!--默认帧格式-->
        <Format>Y11</Format>
        <!--开流失败后是否需要进行重试，0-不重试，>=1-重试并重试多少次-->
        <StreamFailedRetry>0</StreamFailedRetry>
    </Depth>
    <Color>
        <!--默认分辨率的宽，int类型-->
        <Width>640</Width>
        <!--默认分辨率的高，int类型-->
        <Height>480</Height>
        <!--默认帧率，int类型-->
        <FPS>30</FPS>
        <!--默认帧格式-->
        <Format>MJPG</Format>
        <!--开流失败后是否需要进行重试，0-不重试，>=1-重试并重试多少次-->
        <StreamFailedRetry>0</StreamFailedRetry>
    </Color>
    <IR>
        <!--默认分辨率的宽，int类型-->
        <Width>640</Width>
        <!--默认分辨率的高，int类型-->
        <Height>400</Height>
        <!--默认帧率，int类型-->
        <FPS>30</FPS>
        <!--默认帧格式-->
        <Format>Y10</Format>
        <!--开流失败后是否需要进行重试，0-不重试，>=1-重试并重试多少次-->
        <StreamFailedRetry>0</StreamFailedRetry>
    </IR>
</Gemini>
```

> init()으로 부터 시작되는 OBCameraNodeDriver의 로직은 아래와 같습니다.

### OBCameraNodeDriver

- init()

  - setDeviceChangedCallback - orbbec 디바이스의 연결, 해제 시 실행되는 callback을 binding합니다.
  - 디바이스의 연결을 지속 탐지하는 check*connect_timer*를 생성하고 checkConnectTimer callback을 binding합니다.

  다음으로, 디바이스의 연결 확인 시 실행되는 두개의 callback을 추가로 실행하는데요. 이는 ROS 2가 아닌 thread를 사용하고 있습니다.

  - queryDevice thread - thead를 통해 디바이스 연결 상태와 ROS 2 실행 상태를 지속 확인합니다.
    - 디바이스 연결 시 onDeviceConnected 메소드가 실행되며 내부에서는 try-catch로 예외처리를 구현하였습니다.
      - startDevice - 어떠한 디바이스가 연결되었는지, 인식한 숫자와 일치하는 디바이스가 존재하는지를 검사한 뒤, 디바이스를 초기화합니다.
        - initializeDevice - 각종 디바이스의 정보를 조회하고 사용자에게 콘솔 출력으로 알려줌과 동시에 OBCameraNode를 생성합니다. ⇒ OBCameraNode는 다음 분석할 파일인 ob_camera_node에서 살펴봅시다.
  - deviceCountUpdate thread
    - updateConnectedDeviceCount - 연결되어 있는 디바이스의 개수를 갱신하고 이 값은 일전 분석했던 예외처리에서 사용됩니다.

> init() 메소드부터 상세하게 살펴봅시다.

```cpp
std::unique_ptr<ob::Context> ctx_ = nullptr;
...
ctx_->setDeviceChangedCallback([this](std::shared_ptr<ob::DeviceList> removed_list,
                                      std::shared_ptr<ob::DeviceList> added_list) {
  (void)added_list;
  onDeviceDisconnected(removed_list);
});
check_connect_timer_ =
    this->create_wall_timer(std::chrono::milliseconds(1000), [this]() { checkConnectTimer(); });
CHECK_NOTNULL(check_connect_timer_);
query_thread_ = std::make_shared<std::thread>([this]() { queryDevice(); });
device_count_update_thread_ = std::make_shared<std::thread>([this]() { deviceCountUpdate(); });
CHECK_NOTNULL(device_count_update_thread_);
```

- checkConnectTimer

```cpp
void OBCameraNodeDriver::checkConnectTimer() {
  if (!device_connected_.load()) {
    RCLCPP_ERROR_STREAM(logger_,
                        "checkConnectTimer: device " << serial_number_ << " not connected");
    return;
  } else if (!ob_camera_node_) {
    device_connected_.store(false);
  }
}
```

- queryDevice

```cpp
void OBCameraNodeDriver::queryDevice() {
  while (is_alive_ && rclcpp::ok()) {
    if (!device_connected_.load()) {
      RCLCPP_INFO_STREAM_THROTTLE(logger_, *get_clock(), 1000, "Waiting for device connection...");
      auto device_list = ctx_->queryDeviceList();
      if (device_list->deviceCount() == 0) {
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
        continue;
      }
      onDeviceConnected(device_list);
    } else {
      std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    }
  }
}
```

- onDeviceConnected

```cpp
void OBCameraNodeDriver::onDeviceConnected(const std::shared_ptr<ob::DeviceList> &device_list) {
  CHECK_NOTNULL(device_list);
  if (device_list->deviceCount() == 0) {
    return;
  }
  RCLCPP_INFO_STREAM_THROTTLE(logger_, *get_clock(), 1000, "onDeviceConnected");
  if (!device_) {
    try {
      startDevice(device_list);
    } catch (ob::Error &e) {
      RCLCPP_ERROR_STREAM(logger_, "startDevice failed: " << e.getMessage());
    } catch (const std::exception &e) {
      RCLCPP_ERROR_STREAM(logger_, "startDevice failed: " << e.what());
    } catch (...) {
      RCLCPP_ERROR_STREAM(logger_, "startDevice failed");
    }
  }
}
```

- startDevice

```cpp
void OBCameraNodeDriver::startDevice(const std::shared_ptr<ob::DeviceList> &list) {
  std::scoped_lock<decltype(device_lock_)> lock(device_lock_);
  if (device_connected_) {
    return;
  }
  if (list->deviceCount() == 0) {
    RCLCPP_WARN(logger_, "No device found");
    return;
  }
  if (device_) {
    device_.reset();
  }
  auto device = selectDevice(list);
  if (device == nullptr) {
    RCLCPP_WARN_THROTTLE(logger_, *get_clock(), 1000, "Device with serial number %s not found",
                         serial_number_.c_str());
    device_connected_ = false;
    return;
  }
  initializeDevice(device);
}
```

- initializeDevice

```cpp
void OBCameraNodeDriver::initializeDevice(const std::shared_ptr<ob::Device> &device) {
  device_ = device;
  CHECK_NOTNULL(device_);
  CHECK_NOTNULL(device_.get());
  if (ob_camera_node_) {
    ob_camera_node_.reset();
  }
  ob_camera_node_ = std::make_unique<OBCameraNode>(this, device_, parameters_);
  device_connected_ = true;
  device_info_ = device_->getDeviceInfo();
  CHECK_NOTNULL(device_info_.get());
  device_unique_id_ = device_info_->uid();
  RCLCPP_INFO_STREAM(logger_, "Device " << device_info_->name() << " connected");
  RCLCPP_INFO_STREAM(logger_, "Serial number: " << device_info_->serialNumber());
  RCLCPP_INFO_STREAM(logger_, "Firmware version: " << device_info_->firmwareVersion());
  RCLCPP_INFO_STREAM(logger_, "Hardware version: " << device_info_->hardwareVersion());
  RCLCPP_INFO_STREAM(logger_, "device type: " << ObDeviceTypeToString(device_info_->deviceType()));
  RCLCPP_INFO_STREAM(logger_, "device unique id: " << device_unique_id_);
}
```

> 다음으로, initializeDevice에서 생성되었던 OBCameraNode에 대해 분석해보았습니다.

### OBCameraNode

- OBCameraNode

  - setupDefaultImageFormat
  - setupTopics
    - getParameters - 각종 매개변수들이 잔뜩 선언, 초기화되고 있습니다.
    - setupDevices - `getParameters`에서 설정한 parameter에 따라 Orbbec device를 셋업하면서 다시금 오류 상황에 대한 처리를 진행합니다.
    - setupProfiles - data stream의 구체적인 profile을 조회하면서 올바른 설정을 확인합니다.
    - setupCameraCtrlServices - `ros_service.cpp` 파일에 각종 service들이 구현되어 있는데요. 이에 대한 초기화를 담당합니다.
    - setupPublishers - color image, depth image, imu, ir 등 각종 publisher들에 대한 초기화를 진행합니다.
    - publishStaticTransforms - `static_tf_broadcaster_`와 `dynamic_tf_broadcaster_` 설정합니다.
      - calcAndPublishStaticTransform - imu, camera, optical frame 등 orbbec camera 내부의 센서들간 위치 offset를 고려하여 각종 tf frame을 계산합니다.
        - publishStaticTF - `std::vector` 타입의 클래스 변수인 `static_tf_msgs_`에 tf data들을 push_back 합니다.
      - publishDynamicTransforms - 실제 `sendTransform`이 동작하는 부분인데요. 그렇지만 사실상 static broadcast가 실행됩니다.
  - startStreams - `ob::Pipeline` 타입의 클래스 변수를 초기화한 뒤, pipeline을 시작시키고 `onNewFrameSetCallback`를 callback으로 binding 합니다.
    - setupPipelineConfig - 사전 정의된 parameter들에 따라 orbbec SDK에 구현된 `setAlignMode`, `enableStream` 등의 메소드를 호출합니다.
    - onNewFrameSetCallback - pipeline으로부터 새로운 data가 유입될 때마다 실행되는 callback입니다.
      - publishPointCloud - depth_frame과 color_frame data를 받아 ob::PointCloudFilter를 통해 필터링하고 상황에 맞는 topic data를 준비한 뒤, 최종 publish합니다.
        - publishColoredPointCloud
        - publishDepthPointCloud
      - onNewFrameCallback - pointcloud가 아닌 rgb, depth, ir 카메라 publisher들에 대해 ROS 2 Conversion을 적용한 뒤 publish합니다.
    - startIMU
      - onNewIMUFrameCallback
        - setDefaultIMUMessage
        - publish

> OBCameraNode 생성자부터 상세 분석을 해봅시다.

이 부분에서 사용된 `is_running_`과 같은 클래스 변수는 atomic type인데요. 아마 Thread간 상태 변환 로직에 사용되는 것 같습니다. 저였다면 Thread가 아닌, Composition을 사용하여 여러 코드로 나누고, 이런 복잡한 구현을 피하려고 하지 않았을까 생각해봅니다.

```cpp
OBCameraNode::OBCameraNode(rclcpp::Node* node, std::shared_ptr<ob::Device> device,
                           std::shared_ptr<Parameters> parameters)
    : node_(node),
      device_(std::move(device)),
      parameters_(std::move(parameters)),
      logger_(node->get_logger()) {
  is_running_.store(true);
  stream_name_[COLOR] = "color";
  stream_name_[DEPTH] = "depth";
  stream_name_[INFRA0] = "ir";
  stream_name_[INFRA1] = "ir2";
  stream_name_[ACCEL] = "accel";
  stream_name_[GYRO] = "gyro";

  compression_params_.push_back(cv::IMWRITE_PNG_COMPRESSION);
  compression_params_.push_back(0);
  compression_params_.push_back(cv::IMWRITE_PNG_STRATEGY);
  compression_params_.push_back(cv::IMWRITE_PNG_STRATEGY_DEFAULT);
  setupDefaultImageFormat();
  setupTopics();
  startStreams();
  if (enable_d2c_viewer_) {
    auto rgb_qos = getRMWQosProfileFromString(image_qos_[COLOR]);
    auto depth_qos = getRMWQosProfileFromString(image_qos_[DEPTH]);
    d2c_viewer_ = std::make_unique<D2CViewer>(node_, rgb_qos, depth_qos);
  }
}
```

- OBCameraNode에서 정의된 ROS 2 메커니즘들은 다음과 같습니다.

```cpp
// Topic publisher
std::map<stream_index_pair, image_transport::Publisher> image_publishers_;
std::map<stream_index_pair, rclcpp::Publisher<sensor_msgs::msg::CameraInfo>::SharedPtr>
    camera_info_publishers_;
...
rclcpp::Publisher<sensor_msgs::msg::PointCloud2>::SharedPtr depth_registration_cloud_pub_;
rclcpp::Publisher<sensor_msgs::msg::PointCloud2>::SharedPtr depth_cloud_pub_;
...
rclcpp::Publisher<Extrinsics>::SharedPtr extrinsics_publisher_;
...
std::map<stream_index_pair, rclcpp::Publisher<sensor_msgs::msg::Imu>::SharedPtr> imu_publishers_;
...
// TF broadcaster
std::shared_ptr<tf2_ros::StaticTransformBroadcaster> static_tf_broadcaster_ = nullptr;
std::shared_ptr<tf2_ros::TransformBroadcaster> dynamic_tf_broadcaster_ = nullptr;
```

- setupDefaultImageFormat

```
void OBCameraNode::setupDefaultImageFormat() {
  format_[DEPTH] = OB_FORMAT_Y16;
  format_str_[DEPTH] = "Y16";
  image_format_[DEPTH] = CV_16UC1;
  encoding_[DEPTH] = sensor_msgs::image_encodings::TYPE_16UC1;
  unit_step_size_[DEPTH] = sizeof(uint16_t);
  format_[INFRA0] = OB_FORMAT_Y16;
  format_str_[INFRA0] = "Y16";
  image_format_[INFRA0] = CV_16UC1;
  encoding_[INFRA0] = sensor_msgs::image_encodings::MONO16;
  unit_step_size_[INFRA0] = sizeof(uint8_t);

  image_format_[COLOR] = CV_8UC3;
  encoding_[COLOR] = sensor_msgs::image_encodings::RGB8;
  unit_step_size_[COLOR] = 3 * sizeof(uint8_t);
}
```

- setupTopics

```cpp
void OBCameraNode::setupTopics() {
  getParameters();
  setupDevices();
  setupProfiles();
  setupCameraCtrlServices();
  setupPublishers();
  publishStaticTransforms();
}
```

- getParameters

```cpp
void OBCameraNode::getParameters() {
  setAndGetNodeParameter<std::string>(camera_name_, "camera_name", "camera");
  for (auto stream_index : IMAGE_STREAMS) {
    std::string param_name = stream_name_[stream_index] + "_width";
    setAndGetNodeParameter(width_[stream_index], param_name, IMAGE_WIDTH);
    param_name = stream_name_[stream_index] + "_height";
    setAndGetNodeParameter(height_[stream_index], param_name, IMAGE_HEIGHT);
    param_name = stream_name_[stream_index] + "_fps";
    ...

	setAndGetNodeParameter(publish_tf_, "publish_tf", true);
  setAndGetNodeParameter(tf_publish_rate_, "tf_publish_rate", 10.0);
  setAndGetNodeParameter(depth_registration_, "depth_registration", false);
  setAndGetNodeParameter(enable_point_cloud_, "enable_point_cloud", true);
  setAndGetNodeParameter<std::string>(ir_info_url_, "ir_info_url", "");
  setAndGetNodeParameter<std::string>(color_info_url_, "color_info_url", "");
  ...
```

- setupDevices

```cpp
void OBCameraNode::setupDevices() {
  auto sensor_list = device_->getSensorList();
  for (size_t i = 0; i < sensor_list->count(); i++) {
    auto sensor = sensor_list->getSensor(i);
    auto profiles = sensor->getStreamProfileList();
    for (size_t j = 0; j < profiles->count(); j++) {
      auto profile = profiles->getProfile(j);
      stream_index_pair sip{profile->type(), 0};
      if (sensors_.find(sip) != sensors_.end()) {
        continue;
      }
      sensors_[sip] = sensor;
    }
  }
  ...
  device_->setBoolProperty(OB_PROP_DEPTH_SOFT_FILTER_BOOL, enable_soft_filter_);
  device_->setBoolProperty(OB_PROP_COLOR_AUTO_EXPOSURE_BOOL, enable_color_auto_exposure_);
  device_->setBoolProperty(OB_PROP_IR_AUTO_EXPOSURE_BOOL, enable_ir_auto_exposure_);
  auto default_soft_filter_max_diff = device_->getIntProperty(OB_PROP_DEPTH_MAX_DIFF_INT);
  if (soft_filter_max_diff_ != -1 && default_soft_filter_max_diff != soft_filter_max_diff_) {
    device_->setIntProperty(OB_PROP_DEPTH_MAX_DIFF_INT, soft_filter_max_diff_);
  }
  auto default_soft_filter_speckle_size =
      device_->getIntProperty(OB_PROP_DEPTH_MAX_SPECKLE_SIZE_INT);
  if (soft_filter_speckle_size_ != -1 &&
      default_soft_filter_speckle_size != soft_filter_speckle_size_) {
    device_->setIntProperty(OB_PROP_DEPTH_MAX_SPECKLE_SIZE_INT, soft_filter_speckle_size_);
  }
} catch (const ob::Error& e) {
  RCLCPP_ERROR_STREAM(logger_, "Failed to setup devices: " << e.getMessage());
} catch (const std::exception& e) {
  RCLCPP_ERROR_STREAM(logger_, "Failed to setup devices: " << e.what());
}

```

- setupProfiles

```
void OBCameraNode::setupProfiles() {
  for (const auto& elem : IMAGE_STREAMS) {
    if (enable_stream_[elem]) {
      const auto& sensor = sensors_[elem];
      CHECK_NOTNULL(sensor.get());
      auto profiles = sensor->getStreamProfileList();
      CHECK_NOTNULL(profiles.get());
      CHECK(profiles->count() > 0);
      for (size_t i = 0; i < profiles->count(); i++) {
        auto profile = profiles->getProfile(i)->as<ob::VideoStreamProfile>();
        RCLCPP_DEBUG_STREAM(
            logger_, "Sensor profile: "
                         << "stream_type: " << magic_enum::enum_name(profile->type())
                         << "Format: " << profile->format() << ", Width: " << profile->width()
                         << ", Height: " << profile->height() << ", FPS: " << profile->fps());
        supported_profiles_[elem].emplace_back(profile);
      }
```

- setupPublisher

```cpp
void OBCameraNode::setupPublishers() {
  using PointCloud2 = sensor_msgs::msg::PointCloud2;
  using CameraInfo = sensor_msgs::msg::CameraInfo;
  auto point_cloud_qos_profile = getRMWQosProfileFromString(point_cloud_qos_);
  if (enable_colored_point_cloud_) {
    depth_registration_cloud_pub_ = node_->create_publisher<PointCloud2>(
        "depth/color/points",
        rclcpp::QoS(rclcpp::QoSInitialization::from_rmw(point_cloud_qos_profile),
                    point_cloud_qos_profile));
  }
  if (enable_point_cloud_) {
    depth_cloud_pub_ = node_->create_publisher<PointCloud2>(
        "depth/points", rclcpp::QoS(rclcpp::QoSInitialization::from_rmw(point_cloud_qos_profile),
                                    point_cloud_qos_profile));
  }
  for (const auto& stream_index : IMAGE_STREAMS) {
    if (!enable_stream_[stream_index]) {
      continue;
    }
    std::string name = stream_name_[stream_index];
    std::string topic = name + "/image_raw";
    auto image_qos = image_qos_[stream_index];
    auto image_qos_profile = getRMWQosProfileFromString(image_qos);
    image_publishers_[stream_index] =
        image_transport::create_publisher(node_, topic, image_qos_profile);
    topic = name + "/camera_info";
    auto camera_info_qos = camera_info_qos_[stream_index];
    auto camera_info_qos_profile = getRMWQosProfileFromString(camera_info_qos);
    camera_info_publishers_[stream_index] = node_->create_publisher<CameraInfo>(
        topic, rclcpp::QoS(rclcpp::QoSInitialization::from_rmw(camera_info_qos_profile),
                           camera_info_qos_profile));
  }
  if (enable_publish_extrinsic_) {
    extrinsics_publisher_ = node_->create_publisher<orbbec_camera_msgs::msg::Extrinsics>(
        "extrinsic/depth_to_color", rclcpp::QoS{1}.transient_local());
  }
}
```

- publishStaticTransforms

```cpp
void OBCameraNode::publishStaticTransforms() {
  if (!publish_tf_) {
    return;
  }
  static_tf_broadcaster_ = std::make_shared<tf2_ros::StaticTransformBroadcaster>(node_);
  dynamic_tf_broadcaster_ = std::make_shared<tf2_ros::TransformBroadcaster>(node_);
  calcAndPublishStaticTransform();
  if (tf_publish_rate_ > 0) {
    tf_thread_ = std::make_shared<std::thread>([this]() { publishDynamicTransforms(); });
  } else {
    static_tf_broadcaster_->sendTransform(static_tf_msgs_);
  }
}
```

- calcAndPublishStaticTransform

```cpp
void OBCameraNode::calcAndPublishStaticTransform() {
  tf2::Quaternion quaternion_optical, zero_rot, Q;
  std::vector<float> trans(3, 0);
  zero_rot.setRPY(0.0, 0.0, 0.0);
  quaternion_optical.setRPY(-M_PI / 2, 0.0, -M_PI / 2);
  std::vector<float> zero_trans = {0, 0, 0};
  auto camera_param = findDefaultCameraParam();
  if (enable_publish_extrinsic_ && extrinsics_publisher_ && camera_param.has_value()) {
    auto ex = camera_param->transform;
    Q = rotationMatrixToQuaternion(ex.rot);
    Q = quaternion_optical * Q * quaternion_optical.inverse();
    extrinsics_publisher_->publish(obExtrinsicsToMsg(ex, "depth_to_color_extrinsics"));
  } else {
    Q.setRPY(0, 0, 0);
  }
  rclcpp::Time tf_timestamp = node_->now();

  publishStaticTF(tf_timestamp, trans, Q, frame_id_[DEPTH], frame_id_[COLOR]);
  publishStaticTF(tf_timestamp, trans, Q, camera_link_frame_id_, frame_id_[COLOR]);
  publishStaticTF(tf_timestamp, zero_trans, quaternion_optical, frame_id_[COLOR],
                  optical_frame_id_[COLOR]);
  publishStaticTF(tf_timestamp, zero_trans, quaternion_optical, frame_id_[DEPTH],
                  optical_frame_id_[DEPTH]);
  publishStaticTF(tf_timestamp, zero_trans, zero_rot, camera_link_frame_id_, frame_id_[DEPTH]);
  publishStaticTF(tf_timestamp, zero_trans, zero_rot, camera_link_frame_id_, frame_id_[ACCEL]);
  publishStaticTF(tf_timestamp, zero_trans, zero_rot, camera_link_frame_id_, frame_id_[GYRO]);
}
```

- publishStaticTF

```cpp
void OBCameraNode::publishStaticTF(const rclcpp::Time& t, const std::vector<float>& trans,
                                   const tf2::Quaternion& q, const std::string& from,
                                   const std::string& to) {
  CHECK_EQ(trans.size(), 3u);
  geometry_msgs::msg::TransformStamped msg;
  msg.header.stamp = t;
  msg.header.frame_id = from;
  msg.child_frame_id = to;
  msg.transform.translation.x = trans.at(2) / 1000.0;
  msg.transform.translation.y = -trans.at(0) / 1000.0;
  msg.transform.translation.z = -trans.at(1) / 1000.0;
  msg.transform.rotation.x = q.getX();
  msg.transform.rotation.y = q.getY();
  msg.transform.rotation.z = q.getZ();
  msg.transform.rotation.w = q.getW();
  static_tf_msgs_.push_back(msg);
}
```

- publishDynamicTransforms

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

- startStreams

```cpp
void OBCameraNode::startStreams() {
  if (pipeline_ != nullptr) {
    pipeline_.reset();
  }
  pipeline_ = std::make_unique<ob::Pipeline>(device_);
  try {
    setupPipelineConfig();
    pipeline_->start(pipeline_config_, [this](std::shared_ptr<ob::FrameSet> frame_set) {
      onNewFrameSetCallback(frame_set);
    });
  } catch (const ob::Error& e) {
    RCLCPP_ERROR_STREAM(logger_, "Failed to start pipeline: " << e.getMessage());
    RCLCPP_INFO_STREAM(logger_, "try to disable ir stream and try again");
    enable_stream_[INFRA0] = false;
    setupPipelineConfig();
    pipeline_->start(pipeline_config_, [this](std::shared_ptr<ob::FrameSet> frame_set) {
      onNewFrameSetCallback(frame_set);
    });
  }
  pipeline_started_.store(true);
  // startIMU();
}
```

- setupPipelineConfig

```cpp
void OBCameraNode::setupPipelineConfig() {
  if (pipeline_config_) {
    pipeline_config_.reset();
  }
  pipeline_config_ = std::make_shared<ob::Config>();
  if (depth_registration_ && enable_stream_[COLOR] && enable_stream_[DEPTH]) {
    pipeline_config_->setAlignMode(ALIGN_D2C_HW_MODE);
  }
  for (const auto& stream_index : IMAGE_STREAMS) {
    if (enable_stream_[stream_index]) {
      RCLCPP_INFO_STREAM(logger_, "Enable " << stream_name_[stream_index] << " stream");
      RCLCPP_INFO_STREAM(
          logger_, "Stream " << stream_name_[stream_index] << " width: " << width_[stream_index]
                             << " height: " << height_[stream_index] << " fps: "
                             << fps_[stream_index] << " format: " << format_str_[stream_index]);
      pipeline_config_->enableStream(stream_profile_[stream_index]);
    }
  }
}
```

- onNewFrameSetCallback

```cpp
void OBCameraNode::onNewFrameSetCallback(const std::shared_ptr<ob::FrameSet>& frame_set) {
  if (frame_set == nullptr) {
    return;
  }
  try {
    publishPointCloud(frame_set);
    auto color_frame = std::dynamic_pointer_cast<ob::Frame>(frame_set->colorFrame());
    auto depth_frame = std::dynamic_pointer_cast<ob::Frame>(frame_set->depthFrame());
    auto ir_frame = std::dynamic_pointer_cast<ob::Frame>(frame_set->irFrame());
    onNewFrameCallback(color_frame, COLOR);
    onNewFrameCallback(depth_frame, DEPTH);
    onNewFrameCallback(ir_frame, INFRA0);
  } catch (const ob::Error& e) {
    RCLCPP_ERROR_STREAM(logger_, "onNewFrameSetCallback error: " << e.getMessage());
  } catch (const std::exception& e) {
    RCLCPP_ERROR_STREAM(logger_, "onNewFrameSetCallback error: " << e.what());
  } catch (...) {
    RCLCPP_ERROR_STREAM(logger_, "onNewFrameSetCallback error: unknown error");
  }
}
```

- startIMU

```cpp
void OBCameraNode::startIMU() {
  for (const auto& stream_index : HID_STREAMS) {
    if (enable_stream_[stream_index]) {
      CHECK(sensors_.count(stream_index));
      auto profile_list = sensors_[stream_index]->getStreamProfileList();
      for (size_t i = 0; i < profile_list->count(); i++) {
        auto item = profile_list->getProfile(i);
        if (stream_index == ACCEL) {
          auto profile = item->as<ob::AccelStreamProfile>();
          auto accel_rate = sampleRateFromString(imu_rate_[stream_index]);
          auto accel_range = fullAccelScaleRangeFromString(imu_range_[stream_index]);
          if (profile->fullScaleRange() == accel_range && profile->sampleRate() == accel_rate) {
            sensors_[stream_index]->start(profile,
                                          [this, stream_index](std::shared_ptr<ob::Frame> frame) {
                                            onNewIMUFrameCallback(frame, stream_index);
                                          });
            imu_started_[stream_index] = true;
            RCLCPP_INFO_STREAM(logger_, "start accel stream with "
                                            << magic_enum::enum_name(accel_range) << " range and "
                                            << magic_enum::enum_name(accel_rate) << " rate");
          }
```

### IMU Test?!

> 소스 코드 확인 시, 주석되어 있는 IMU publisher와 관련된 부분들을 확인할 수 있는데요. 과연 IMU data를 사용할 수 있는지 확인해보겠습니다.

- 주석 해제 - `ob_camera_node.cpp`

```cpp
void OBCameraNode::startStreams() {
  ...
  // 주석 해제
  pipeline_started_.store(true);
  startIMU();
}

void OBCameraNode::setupPublishers() {
  ...
  // 주석 해제
  for (const auto& stream_index : HID_STREAMS) {
    if (!enable_stream_[stream_index]) {
      continue;
    }
    std::string data_topic_name = stream_name_[stream_index] + "/sample";
    auto data_qos = getRMWQosProfileFromString(imu_qos_[stream_index]);
    imu_publishers_[stream_index] = node_->create_publisher<sensor_msgs::msg::Imu>(
      data_topic_name, rclcpp::QoS(rclcpp::QoSInitialization::from_rmw(data_qos), data_qos));
    RCLCPP_INFO_STREAM(logger_, "IMU Publisher created");
  }
  if (enable_publish_extrinsic_) {
    extrinsics_publisher_ = node_->create_publisher<orbbec_camera_msgs::msg::Extrinsics>(
      "extrinsic/depth_to_color", rclcpp::QoS{1}.transient_local());
  }
}
```

- 주석 해제 - `gemini2.launch.xml`

```xml
	<!-- imu -->
	<arg name="enable_accel" default="true"/>
	<arg name="accel_rate" default="100hz"/>
	<arg name="accel_range" default="4g"/>
	<arg name="enable_gyro" default="true"/>
	<arg name="gyro_rate" default="100hz"/>
	<arg name="gyro_range" default="1000dps"/>
	<arg name="liner_accel_cov" default="0.01"/>
	<arg name="angular_vel_cov" default="0.01"/>


  <group>
    <push-ros-namespace namespace="$(var camera_name)"/>
	    <node name="camera" pkg="orbbec_camera" exec="orbbec_camera_node" output="screen">
          ...
	        <!-- imu -->
	        <param name="enable_accel" value="$(var enable_accel)" />
	        <param name="accel_rate" value="$(var accel_rate)" />
	        <param name="accel_range" value="$(var accel_range)" />
	        <param name="enable_gyro" value="$(var enable_gyro)" />
	        <param name="gyro_rate" value="$(var gyro_rate)" />
	        <param name="gyro_range" value="$(var gyro_range)" />
	        <param name="liner_accel_cov" value="$(var liner_accel_cov)"/>
	        <param name="angular_vel_cov" value="$(var angular_vel_cov)"/>
```

- 마지막으로 imu data의 시각화를 위해 `rviz2 plugin`을 설치합니다.

```cpp
sudo apt install ros-foxy-rviz-imu-plugin -y
```

- launch file을 다시 실행시킨 뒤 rviz2를 확인해보면 다음과 같이 /gyro/sample/imu라는 데이터를 확인할 수 있습니다.

![Untitled1.png](/kr/ros2_foxy/images19/Untitled1.png?height=400px)

- 하지만 Topic 시각화를 추가하면 아래와 같이 tf2관련 에러들이 확인됩니다. **topic은 만들어두고, tf2는 broadcast하고 있지 않기 때문입니다.**

```xml
[INFO] [1685265445.144163057] [rviz]: Message Filter dropping message: frame 'camera_gyro_optical_frame' at time 1685265444.622 for reason 'Unknown'
[INFO] [1685265445.179582316] [rviz]: Message Filter dropping message: frame 'camera_gyro_optical_frame' at time 1685265444.652 for reason 'Unknown'
[INFO] [1685265445.214360819] [rviz]: Message Filter dropping message: frame 'camera_gyro_optical_frame' at time 1685265444.712 for reason 'Unknown'
[INFO] [1685265445.244405298] [rviz]: Message Filter dropping message: frame 'camera_gyro_optical_frame' at time 1685265444.751 for reason 'Unknown'
```

- 실제 topic data를 살펴보아도 frame*id는 camera_gyro_optical_frame로 되어있는 반면, tf2 tree에서 해당 tf2는 확인되지 않습니다. 더불어 angular_velocity만 갱신될 뿐 제일 중요한 orientation data를 얻을 수 없습니다. *(orbbec측의 빠른 업데이트를 기대해봅시다.)\_

```xml
---
header:
  stamp:
    sec: 1685265428
    nanosec: 750000128
  frame_id: camera_gyro_optical_frame
orientation:
  x: 0.0
  y: 0.0
  z: 0.0
  w: 0.0
orientation_covariance:
- -1.0
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0
- 0.0
angular_velocity:
  x: 0.002128450432792306
  y: -0.006385351065546274
  z: 0.0026605629827827215
angular_velocity_covariance:
```
