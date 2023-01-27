---
title: "Lecture10 - About DDS"
date: 2023-01-01T14:08:02+09:00
draft: false
---

## About DDS

> ROS 1과 대두되는 가장 큰 차이점으로, ROS 2는 DDS를 미들웨어로 기반하여 개편되었습니다. 따라서 DDS에 대한 개요와 기능을 이해하는 것은 ROS 2의 성질을 파악하는 데 많은 도움이 됩니다.

### What is DDS?

**DDS - Data Distribution Service**는 OMG에서 정의한 Publish-Subscribe 방식의 실시간 데이터 분배 서비스 표준입니다.

Pub-Sub이라는 용어는 ROS 개발자에게 무척 익숙한 단어이지요? DDS의 기본 통신 개념은 ROS의 Topic과 매우 유사합니다. 이러한 결론을 머리속에 잘 담아두고 계속해서 진행해 보겠습니다.

{{% notice note %}}
OMG(Object Management Group)는 분산 객체 컴퓨팅(Distributed Object Computing) 영역의 표준을 제정하는 국제 비영리 컨소시엄으로 영향력 있는 각종 세계 표준을 정의하고 있습니다.
{{% /notice %}}

![main0.png](/kr/ros2_basic_foxy/images10/main0.png?height=300px)

- image from : [분산이동컴퓨팅 연구실](http://strauss.cnu.ac.kr/research_echodds.php)

OMG DDS는 통신 프로토콜이 아닌 기능적 표준으로, 위 그림과 같이 TCP/UDP 같은 통신 프로토콜 위에서 정의되는 객체입니다.

그림의 아래에서부터 OMG DDS의 구조를 간단히 살펴보겠습니다.

### RTPS(Real-Time Publish-Subscribe Wire Protocol) Layer

- **RTPS Layer**는 네트워크 내, 참여자 정보를 유지하고, 네트워크 참여자에 대한 정보를 기반으로 자동 검색을 해주며, (같은 네트워크를 사용하는 ROS 2 시스템은 서로 통신이 가능합니다.) 더불어, 참여자의 동적 추가와 이탈에 대응하는 기능을 합니다.

- RTPS는 OMG에 의해 표준화된, 데이터 분산 시스템을 위한 프로토콜로써 Pub-Sub 구조의 통신 모델을 지원합니다. UDP와 같이 신뢰성 없는 계층 위에서도 동작 가능하도록 설계되었으며, 4가지 모듈로 구성되어 있습니다.

1. **Structure Module** : 데이터 교환 시, 통신에 참여하게 되는 개체들에 대해 정의합니다.
2. **Message Module** : Writer와 Reader간 정보 교환을 위해 사용되는 메시지에 대해 정의합니다.
3. **Behavior Module** : Writer와 Reader간 상태, 시간 조건에 따라 수행되어야 할 메시지 전송 절차에 대해 정의합니다.
4. **Discovery Module** : 같은 도메인 상에 존재하는 개체에 대한 정보를 탐색하는 기능을 수행합니다. Discovery Module은 다음과 같이 두 가지의 RTPS 프로토콜을 사용합니다.
   1. **PDP**(Participant Discovery Protocol) : 서로 다른 네트워크 상에서의 Participant탐색을 위한 프로토콜
   2. **EDP**(Endpoint Discovery Protocol) : Writer, Reader와 같이 서로 다른 종단점 간의 탐색 정보 교환에 사용되는 프로토콜

![main1.png](/kr/ros2_basic_foxy/images10/main1.png?height=300px)

- image from : [OMG DDS(Data Distrubution Service) 기술 개요](https://blog.naver.com/neos_rtos/30173805276)

### DDS Discovery

> ROS 2를 사용하는 두 디바이스는 같은 네트워크를 사용하고 있다면 자동으로 서로를 인식할 수 있습니다. 어떻게 이러한 동작이 가능하며, 이를 위한 필요조건은 무엇인지 살펴보겠습니다.

**Discovery를 위해 필요한 정보들** - DDS 미들웨어 통신을 위해 아래 4가지 정보를 사전에 교환해야 합니다.

- **Who** : Topic Publish 혹은 Subscribe 대상
- **Where** : DDS 표준에서 생성한 Locator 정보
- **What** : Publish하거나 Subscribe하려는 Topic
- **How** : Topic Publisher인지 Subscriber인지에 대한 정보

{{% notice note %}}
DDS RTPS도 결국 UDP나 TCP 네트워크 레이어를 사용하며, 이를 위해서 실질적으로 IP와 PORT등의 정보가 필요합니다. 하지만, DDS는 사용자가 주소를 지정하지 않아도 “Topic”만으로 통신이 가능합니다. 이를 가능하게 해주는 것이 **Locator**이며, 이는 Middle Ware단에서 능동적으로 통신에 필요한 정보를 포함하는 새로운 개념을 별도로 생성하는 것입니다.
{{% /notice %}}

DDS 표준에서는 Discovery를 수행하는 절차를 **PDP** 와 **EDP**로 구분하고 있습니다.

- **PDP(Participant Discovery Protocol)** : 같은 도메인에 존재하는 서로 다른 Participant를 검색하는 절차로, Participant에 포함되어 있는 endpoint 의 정보를 검색합니다.
- **EDP(Endpoint Discovery Protocol)** : PDP를 통해 Participant가 서로를 탐지하고 나면, EDP가 수행됩니다. 표준에서는 각 프로토콜이 지원해야할 최소한의 기능을 만족하는 SPDP와 SEDP에 대해 설명하고 있습니다.

{{% notice note %}}
vendor에 따라 다양한 PDP와 EDP를 지원할 수 있지만, 호환성을 위해 모든 RTPS는 SPDP(Simple Participant Discovery Protocol)와 SEDP(Simple Endpoint Discovery Protocol)를 필수로 지원해야합니다. 서로 다른 vendor를 가진 머신 사이에 통신이 가능한 이유이기도 합니다.
{{% /notice %}}

> 그림을 통해 SPDP와 SEDP를 활용한 Discovery 절차에 대해 알아봅시다.

- 같은 Domain을 사용하는 Participant는 생성 시, Discovery를 수행하는 3쌍의 builtin Writer와 builtin Reader, 그리고 pre-defined topic를 포함하게 됩니다. DDS가 구동되면, Participant는 Multicast로 PDP(Participant Discovery Protocol) 메시지를 주기적으로 전송하기 시작합니다.

![main2.png](/kr/ros2_basic_foxy/images10/main2.png?height=250px)

- 각 Participant의 builtinParticipantWriter와 builtinPariticpantReader 사이 SPDP 교환이 이루어집니다. 전달되는 정보에는 Participant에 포함된 builtinPublicationWriter와 builtinPublicationReader의 Locator 정보가 담겨 있습니다.

![main3.png](/kr/ros2_basic_foxy/images10/main3.png?height=250px)

- SPDP 교환 이후, 상대방 Participant의 정보를 통해 양측 Participant들은 서로 연결됩니다.

![main4.png](/kr/ros2_basic_foxy/images10/main4.png?height=350px)

- 이후, Topic 수행을 위해 각 Participant에 Endpoint에 해당하는 Publisher와 Datawriter가 하나씩 생성됩니다.

![main5.png](/kr/ros2_basic_foxy/images10/main5.png?height=300px)

- SPDP에서 교환했던 Locator정보를 바탕으로 builtinPublicationWriter는 builtiinPublicationReader에게, builtinSubscriptionWriter는 SubscriptionReader에게 Unicast를 통해 정보를 전달합니다.(SEDP를 교환하는 것입니다.) 전달되는 정보에는 topic, data type, Qos등의 정보가 포함되어 있습니다.

![main6.png](/kr/ros2_basic_foxy/images10/main6.png?height=300px)

- SEDP의 결과로 Participant에 포함된 Endpoint들이 연결되고, 연결된 두 Endpoint는 동일 Topic으로 통신을 시작합니다.

![main7.png](/kr/ros2_basic_foxy/images10/main7.png?height=300px)

> Discovery 과정을 한번에 정리한 그림입니다. 다시 한 번 의미를 이해하면서 과정을 복기해봅시다.

![main8.png](/kr/ros2_basic_foxy/images10/main8.png?height=400px)

### DCPS(Data-Centric Publish-Subscribe) Layer

> DCPS Layer는 RTPS Layer와 Application 사이의 인터페이스 역할을 합니다. 이후 다뤄지는 Participant, Domain, Topic 등을 정의하고, Publish-Subscribe를 수행합니다. 더불어, DDS의 중요한 기능 중 하나인 QoS(Quality of Service)도 담당하고 있습니다.

구현 측면에서, DCPS는 Data read/write API를 제공함으로 응용 프로그램들 사이 데이터를 교환할 상대에 대한 인지 없이 원하는 데이터의 송수신이 가능하게 합니다.

![main9.png](/kr/ros2_basic_foxy/images10/main9.png?height=300px)

- image from : [OMG DDS(Data Distrubution Service) 기술 개요](https://blog.naver.com/neos_rtos/30173805276)

#### DataWriter / DataReader

> DataWriter는 송신자, DataReader는 수신자의 역할을 합니다. DataWriter는 Sample(실제 데이터)을 생성하고 전송하는 역할을 하며, DataReader는 Sample을 수신하는 기능을 합니다. 단일 DataWriter와 DataReader는 단일 Topic만을 보유할 수 있습니다.

- **Publisher**

데이터 Publish를 담당하는 객체로, 하나의 Publisher는 다수의 DataWriter를 가질 수 있습니다. 때문에, 각 Publisher들은 여러 Topic 데이터를 Publish 할 수 있습니다. Publish하고자 하는 데이터 객체의 값이 Publisher에게 전해지면 Publisher는 자신에게 설정된 QoS값에 따라 연관된 Subscriber에게 전달합니다.

- **Subscriber**

데이터 Subscribe를 담당하는 객체로, Publisher와 유사하게 다수의 DataReader를 가질 수 있으며 여러 Topic 데이터를 수신할 수 있습니다. Subscriber가 데이터를 수신한 이후, 응용 프로그램은 DataReader를 통해 실제 데이터로 접근하는 방식입니다.

### DDS Pub-Sub의 기본 구성 요소

![main10.png](/kr/ros2_basic_foxy/images10/main10.png?height=300px)

- image from : [MDS 테크](https://blog.naver.com/PostView.naver?blogId=neos_rtos&logNo=220346180839&parentCategoryNo=&categoryNo=20&viewDate=&isShowPopularPosts=false&from=postList)

**Topic**

Topic은 Publish-Subscribe의 관계를 정의하는 Key입니다. topic은 데이터 타입과 해당 데이터의 **QoS**에 대한 정보를 담고 있습니다. QoS를 Topic 데이터에 추가하고, Topic과 연관된 DataWriter, Datawriter에 연관된 QoS를 설정하는 식으로 제어됩니다. (Subscriber 측도 동일합니다.)

{{% notice note %}}
Topic의 이름은 Domain 내에서 유일해야 합니다.
{{% /notice %}}

**Participant**

Participant는 DDS Publisher와 Subscriber를 담게 되는 객체입니다. ROS 2의 Node안에 다수의 Publisher와 Subscriber와 공존할 수 있는 것처럼, Participant도 다수의 DataWriter와 DataReader를 가질 수 있습니다.

![main11.png](/kr/ros2_basic_foxy/images10/main11.png?height=300px)

**Domain**

Domain은 Participant들이 활동하는 영역으로, 같은 Domain을 갖는 Participant내에서만 정보전달을 할 수 있습니다. ROS 2 터미널의 실행 시 **ROS_DOMAIN_ID**라는 콘솔 출력이 나왔던 것을 기억하시나요?

```bash
***************  Startup Menu for ROS  ***************
* Usage: To set ROS env to be auto-loaded, please    *
*        assign ros_option in ros_menu/config.yaml   *
******************************************************
0) Do nothing
1) ROS 1 noetic
2) ROS 2 foxy
3) ROS2/ROS1_bridge
Please choose an option: 2
------------------------------------------------------
* ROS_DOMAIN_ID = 30
------------------------------------------------------
```

> 이렇게 DDS의 통신 구조에 대해서 알아보았습니다. 위 구조에서 실제 사용자는 Topic, Publisher, Subscriber만 구현하면 되며, DataWriter, DataReader와 RTPS, DCPS는 DDS가 알아서 처리하게 됩니다. 이러한 DDS의 장점에 기반하여 ROS 2 시스템이 올라가게 되는 것이지요.

## DDS QoS

> TCPROS/UDPROS라는 자체 프로토콜을 사용했던 기존 ROS 1과는 달리, ROS 2에서는 QoS 옵션을 통해 원하는 기능을 선택적으로 사용할 수 있습니다. 이는 DDS의 QoS를 도입하였기 때문으로, Publisher, Subscriber를 선언할 때, QoS를 매개변수형태로 지정하는 방식이 사용됩니다.

상황에 따라 데이터의 신뢰성이 중요할 수도 있고, 버퍼의 크기를 크게 하여 Topic의 안정성을 높여야 할 수도 있습니다. 이러한 통신의 품질을 옵션화하기 위해서, DDS에서는 아래와 같은 22가지의 QoS를 정의하고 있습니다.

![qos0.png](/kr/ros2_basic_foxy/images10/qos0.png?height=300px)

각각의 QoS 옵션은 연관된 조합으로 Topic, DataWriter, DataReader, Publisher, Subscriber, Domain, Participant에 적용되며, 본 세션에서는 주로 사용되는 QoS를 위주로 살펴보고자 합니다.

- **RELIABILITY**: 데이터 통신의 신뢰성 레벨(재전송 여부)을 결정하며 두 가지 속성을 설정 할 수 있습니다.
  - **RELIABLE** : DataWriter History에 있는 모든 샘플들이 DataReader에 전달되는 것을 보장합니다.
  - **BEST_EFFORT** : 통신 시 손실된 데이터 샘플을 재전송 하지 않고, 전달된 데이터의 순서는 유지 합니다.

![qos1.png](/kr/ros2_basic_foxy/images10/qos1.png?height=100px)

{{% notice note %}}
알파벳은 다음과 같은 의미를 갖습니다. **T** : TOPIC, **DR** : DataReader, **DW** : DataWriter, **P** : Publisher, **S** : Subscriber
{{% /notice %}}

{{% notice note %}}
RxO는 Requested/Offered의 약자로 QoS가 적용되는 대상을 Publisher(발간/송신) 측과 Subscriber(구독/수신) 측으로 구분해서 서로의 상관관계를 표현하는 방법입니다.
{{% /notice %}}

- **‘YES’** : Publisher 측과 Subscriber 측 모두 QoS가 적용되어야 하고, 설정된 QoS 값이 호환되어야 한다.
- **‘NO’** : Publisher 측과 Subscriber 측 모두 QoS 가 적용되어야 하지만, 설정된 QoS 값은 서로 독립적이다.
- **‘N/A’** : Publisher 측과 Subscriber 측 중 한 측에만 적용되어야 한다.

{{% notice note %}}
Changeable이 ‘YES’ 일 경우에는, 동작하면서도 QoS 의 값이 변경될 수 있지만, ‘NO’일 경우에는 처음 생성된 이후에는 변경할 수 없습니다.
{{% /notice %}}

- **HISTORY** : 데이터의 재전송을 위해 HistoryCache내 데이터 보관 방법을 결정하며 두 가지 속성을 설정 할 수 있습니다.
  - **KEEP_LAST** : Depth(History Cache안에 유지하고 있을 데이터 개수) 크기 만큼 최신 데이터를 유지 합니다.
  - **KEEP_ALL** : DataReader에게 인스턴스의 모든 값들을 유지하고 전송 합니다.

![qos2.png](/kr/ros2_basic_foxy/images10/qos2.png?height=100px)

- **DURABILITY** : 나중에 참여한 DataReader에게 이전 데이터를 전송할지 여부를 결정 하며 네 가지 속성을 설정 할 수 있습니다.
  - **VOLATILE** : 연결이 설정 된 이후의 데이터만 제공 합니다.
  - **TRANSIENT** **LOCAL** : DataWriter 생명주기와 일치 일치합니다.
  - **TRANSIENT** : DataReader에 과거 데이터를 제공 합니다.
  - **PERSISTENT** : Permanent-Storage에 과거 데이터를 저장하며, 데이터의 유효성은 System 보다 오래 지속 됩니다.

![qos3.png](/kr/ros2_basic_foxy/images10/qos3.png?height=100px)

- **OWNERSHIP** : 다수의 DataWriter가 동일한 인스턴스를 갱신하게 허용할지를 결정합니다.
  - **SHARED** : 다수의 DataWriter들이 동일한 데이터 인스턴스 업데이트가 가능합니다.
  - **EXCLUSIVE** : 데이터 객체의 각 인스턴스는 하나의 DataWriter에 의해서만 수정 가능합니다.

![qos4.png](/kr/ros2_basic_foxy/images10/qos4.png?height=100px)

- **PARTITION** : Domain 내부에서 Node들을 분리하는 방법을 결정 합니다. (Domain 내에서 별도의 논리적인 통신 채널 형성) - Partition Name이 같은 DataWriter/DataReader간 데이터 배포 가 가능합니다.

{{% notice note %}}
**Partition과 Domain의 차이** - 다른 Domain들에 속하는 개체들은 완전히 서로 독립 된 상태입니다. Entity는 다수의 Partition에 있을 수 있지만 하나의 Domain에만 속할 수 있습니다.

{{% /notice %}}

![qos5.png](/kr/ros2_basic_foxy/images10/qos5.png?height=100px)

- **DEADLINE** : 데이터 샘플 사이의 최대 도착 시간을 정의할 수 있습니다.
  - **DataWriter**는 DEADLINE(period:Duration_t)이 설정된 시간 안에 적어도 한번 이상의 데이터를 전송합니다.
  - **DataReader**는 DEADLINE(period:Duration_t) 시간 안에 DataWriter로부터 데이터를 받지 못하면 DDS로부터 위반 통보를 받습니다.

![qos6.png](/kr/ros2_basic_foxy/images10/qos6.png?height=100px)

- **TIME_BASED_FILTER** : DataReader 가 데이터를 필터링 하는 시간을 결정 합니다.
  - Minimum_separation: Duation_t 값을 설정하여 DataReader는 이 값 이내에 수신한 데이터는 삭제 합니다.

![qos7.png](/kr/ros2_basic_foxy/images10/qos7.png?height=100px)

> ROS 2에서는 자주 사용되는 QoS 조합을 묶어 RMW QoS Profile이라는 이름으로 제공하고 있습니다. 지금까지 우리가 queue_size만을 전달하고 있었지만, 사실은 아래 사진의 Default에서 Depth만 바꿔주고 있었던 것입니다.

![qos8.png](/kr/ros2_basic_foxy/images10/qos8.png?height=300px)

- image from : [오로카](https://cafe.naver.com/openrt/24319)

> ros2에서 QoS 설정하는 방법은 매우 간단합니다. Publisher, Subscriber 클래스를 생성하면서 QoS 옵션을 매개변수로 전달하면 됩니다.

- python example - RMW QoS Profile 사용 시

```python
from rclpy.qos import qos_profile_sensor_data

self.string_publisher = self.create_publisher(String, 'qos_test_topic', qos_profile_sensor_data)
```

- python example - 직접 Profile 설정 시

```python
from rclpy.qos import QoSDurabilityPolicy
from rclpy.qos import QoSHistoryPolicy
from rclpy.qos import QoSProfile
from rclpy.qos import QoSReliabilityPolicy

my_profile = QoSProfile(
    reliability=QoSReliabilityPolicy.BEST_EFFORT,
    history=QoSHistoryPolicy.KEEP_LAST,
    depth=10,
    durability=QoSDurabilityPolicy.VOLATILE
)

self.string_publisher = self.create_publisher(String, 'qos_test_topic', my_profile)
```

=> 참고 링크 : [rclpy/qos.py](https://github.com/ros2/rclpy/blob/4e1654eca2758c42e7c22f3d6c518d6ec26ad9de/rclpy/rclpy/qos.py#L390)

- topic 통신이 이루어지고 있는 상황에서, QoS 옵션을 보고 싶다면 Topic 커멘드에 verbose 옵션을 추가하면 됩니다.

```bash
# Terminal 1 - publisher
$ ros2 run py_topic_tutorial qos_example_publisher
# Terminal 2 - subscriber
$ ros2 run py_topic_tutorial qos_example_subscriber
# Terminal 3 - topic info
$ ros2 topic info /qos_test_topic --verbose
Type: std_msgs/msg/String

Publisher count: 1

Node name: twist_pub_node
Node namespace: /
Topic type: std_msgs/msg/String
Endpoint type: PUBLISHER
GID: 60.77.10.01.f3.67.78.3d.6b.0c.cc.7b.00.00.14.03.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: RMW_QOS_POLICY_RELIABILITY_BEST_EFFORT
  Durability: RMW_QOS_POLICY_DURABILITY_VOLATILE
  Lifespan: 9223372036854775807 nanoseconds
  Deadline: 9223372036854775807 nanoseconds
  Liveliness: RMW_QOS_POLICY_LIVELINESS_AUTOMATIC
  Liveliness lease duration: 9223372036854775807 nanoseconds

Subscription count: 1

Node name: string_sub_node
Node namespace: /
Topic type: std_msgs/msg/String
Endpoint type: SUBSCRIPTION
GID: d2.03.10.01.d5.6b.69.e0.be.3f.b5.a2.00.00.14.04.00.00.00.00.00.00.00.00
QoS profile:
  Reliability: RMW_QOS_POLICY_RELIABILITY_BEST_EFFORT
  Durability: RMW_QOS_POLICY_DURABILITY_VOLATILE
  Lifespan: 9223372036854775807 nanoseconds
  Deadline: 9223372036854775807 nanoseconds
  Liveliness: RMW_QOS_POLICY_LIVELINESS_AUTOMATIC
  Liveliness lease duration: 9223372036854775807 nanoseconds
```

> DDS QoS는 **RxO - requested by offered** 특성을 갖고 있어 서로 양립할 수 없는 조합이 존재합니다. 예를 들어, Publish의 Reliability가 BEST_EFFORT일때, Subscriber의 Reliability가 RELIABLE라면, Topic 통신이 발생할 수 없습니다.

- 코드를 수정하여 직접 실습해봅시다.

{{< tabs >}}
{{% tab name="qos_example_publisher.py" %}}

```python
my_profile = QoSProfile(
    reliability=QoSReliabilityPolicy.BEST_EFFORT,
    history=QoSHistoryPolicy.KEEP_LAST,
    depth=10,
    durability=QoSDurabilityPolicy.VOLATILE
)
...
self.string_publisher = self.create_publisher(String, 'qos_test_topic', qos_profile_sensor_data)
```

{{% /tab %}}
{{% tab name="qos_example_subscriber.py" %}}

```python
my_profile = QoSProfile(
    reliability=QoSReliabilityPolicy.RELIABLE,
    history=QoSHistoryPolicy.KEEP_LAST,
    depth=10,
    durability=QoSDurabilityPolicy.VOLATILE
)
...
self.pose_subscriber = self.create_subscription(
    String, 'qos_test_topic', self.sub_callback, my_profile
)
```

{{% /tab %}}
{{< /tabs >}}

- 코드 변경 후 실행 - Warning과 함께 Subscriber가 동작하지 않음을 알 수 있습니다.

```python
$ ros2 run py_topic_tutorial qos_example_publisher
[WARN] [1673945160.449603592] [twist_pub_node]: New subscription discovered on this topic, requesting incompatible QoS. No messages will be sent to it. Last incompatible policy: RELIABILITY_QOS_POLICY
$ ros2 run py_topic_tutorial qos_example_subscriber
[WARN] [1673945160.455319911] [string_sub_node]: New publisher discovered on this topic, offering incompatible QoS. No messages will be received from it. Last incompatible policy: RELIABILITY_QOS_POLICY
```

> QoS를 통해 원하는 도메인에서의 성능을 끌어올리고 안정성을 갖춘 시스템을 구축합시다.

⇒ 다음 페이지에서 계속됩니다.
