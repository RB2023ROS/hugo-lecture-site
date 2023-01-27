---
title: "Lecture12 - SROS2"
date: 2022-12-21T04:41:42+09:00
draft: false
---

## sros2

DDS는 기본적으로 보안이 적용되어 있지는 않습니다. 대신, 아래와 같이 5가지의 보안 기능이 표준 정의에 포함되어 있습니다. 사용자는 이들을 Plugin 형태로 추가하여 사용하게 되며, 주로 Vendor단에서 패치 형태로 제공합니다.

1. **Authentication**: 같은 네트워크를 사용하고, 특정 domain내에 존재하는 participant 사이의 신원을 검사합니다. (x.509 인증서와 특정 Public Key Infrastructure - PKI를 사용합니다. )
2. **Access control**: participant의 동작, 혹은 리소스를 제한합니다. (apparmor나 cgroup이 아니라 특정 participant를 특정 DDS 도메인으로 제한하거나, participant가 특정 DDS 항목을 읽거나 쓸 수 있도록 하는 형태입니다.)
3. **Cryptography**: encryption/decryption/signing/hashing/digital signatures와 같은 암호화 관련 작업을 담당합니다. (Authentication과 Access control plugin에서 모두 이 기능을 사용하게 되고, DDS의 Topic Message를 암호화합니다.)
4. **Logging**: DDS의 보안 관련 이벤트를 로깅합니다.
5. **Data tagging**: data에 추가 label을 달수 있습니다. 이를 통해 데이터의 보안 측면 분류를 할 수 있으며, 더욱 개선된 access control이 가능합니다.

{{% notice note %}}
아래 2종류는 필수 표준이 아니기 때문에 대부분의 Vendor들은 제공하고 있지 않습니다.
{{% /notice %}}

> [ROS 2 Security Working Group (SWG)](https://github.com/ros-security/community)은 DDS와 ROS 2의 보안과 관련된 오픈 그룹입니다. 해당 그룹에서는 DDS의 보안 취약점을 분석하였는데요, 그 결과 총 13종류의 보안 취약점을 발견하였습니다.

- 이러한 13가지 취약성은 CVE(Common Vulnerabilities and Exposure)로 등록되었고, 이 중 7개는 [CVSS(Common Vulnerability Scoring System)](https://www.first.org/cvss/specification-document) v3를 기준으로 'High' 등급을 판정받았으며, 나머지 5개 또한 'Medium' 등급을 판정받았습니다.

- 이러한 분석 결과로 US Cybersecurity and Infrastructure Security Agency (CISA)는 DDS 시스템에 보안 권고 경고 ICS Advisory [(ICSA-21-315-02)](https://www.cisa.gov/uscert/ics/advisories/icsa-21-315-02)를 내렸고, 그 결과 각 DDS 벤더들은 현재 보안에 계속해서 신경쓰고 있는 상황입니다.

=> 해당 13종의 보안 취약점에 대해서 간단히 리뷰를 해보겠습니다.

#### Network-based vulnerabilities

- Network Layer에서 고의적인 RTPS packet 전송을 통해 DoS 공격이 가능한 점
- 불충분한 IP Check (기본 Domain Multicast IP 등)
  관련 DDS 표준 - CVE-2021-38425, CVE-2021-38429, CVE-2021-38487, CVE-2021-43547
  관련 위험 코드 - [Common Weakness CWE-406](https://cwe.mitre.org/data/definitions/406.html)

#### Configuration-based vulnerabilities

- DDS의 셋업 파일은 XML, JSON, YAML과 같은 보편적인 형태를 사용하므로 위험하다고 판단
- ex1) DDS 시스템 중 보안이 취약한 XML 라이브러리를 파고들어 액세스 권한을 탈취할 수 있음
- ex2) 특정 XML Pasrse 내 부적절한 값을 전달하여 buffer overflow를 발생시킬 수 있음

⇒ 이러한 이유로 현재 대부분의 DDS Vendor들은 보안 Patch를 배포한 상황입니다.

| Fast DDS | https://github.com/eProsima/Fast-DDS      |
| -------- | ----------------------------------------- |
| Open DDS | https://opendds.org/                      |
| rti DDS  | https://www.rti.com/products/dds-standard |

> 아래 첨부 파일에는 Ubuntu를 관리하는 Canonical 재단에서 배포한 ROS 사용자를 위한 보안 강화 전략들이 담겨 있습니다. 라즈베리파이와 TurtleBot 로봇을 통해 각종 보안 테스트를 한 결과로 얻게 된 인사이트가 담겨 있습니다.

- Disabling USB
- Remove default users such as “ubuntu”
- SSH hardening
- Disabling Internet Protocol v6
- Unattended upgrades

{{% attachments title="Example Packages" /%}}

## SROS2

> ROS 1의 sros와 마찬가지로 sros2는 ROS 2의 보안을 위한 도구들의 집합입니다.

- CA 인증서 자동 생성, x.509에 기반한 keypair 생성
- keypairs, governance and permissions files등이 담기는 keystore 생성
- DDS traffic을 암호화하는 governance file 제공

SROS2 공식 레퍼런스에서 제공하는 기본 예시들을 함께 실행해보고, 패킷 분석도 진행해보겠습니다.

- sros2 setup

```bash
sudo apt update && sudo apt install libssl-dev
sudo apt install ros-foxy-demo-nodes-cpp -y
sudo apt install ros-foxy-demo-nodes-py -y

mkdir ~/sros2_demo
cd ~/sros2_demo
```

- 각종 인증 관련 파일들이 위치하게 될 keystore를 생성합니다.

```bash
$ ros2 security create_keystore demo_keystore
creating keystore: demo_keystore
creating new CA key/cert pair
creating governance file: demo_keystore/enclaves/governance.xml
creating signed governance file: demo_keystore/enclaves/governance.p7s
all done! enjoy your keystore in demo_keystore
cheers!
```

- 생성된 keystore 폴더는 다음과 같은 폴더 형태를 취하고 있습니다.

```bash
├── enclaves
│   ├── governance.p7s
│   └── governance.xml
├── private
│   ├── ca.key.pem
│   ├── identity_ca.key.pem -> ca.key.pem
│   └── permissions_ca.key.pem -> ca.key.pem
└── public
    ├── ca.cert.pem
    ├── identity_ca.cert.pem -> ca.cert.pem
    └── permissions_ca.cert.pem -> ca.cert.pem
```

아래 예시와 같이 각종 인증키가 생성되는 것을 확인 가능합니다.

- ca.key.pem

```bash
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQgiXuds0wpQA1F7Cnv
+533UuaPWKnZBYscqvHoyo00XOqhRANCAATI2CRyvxYccuxaWGTKDDe5b2dmdqH5
pBBbeHXXD1QRt7khzpX11InLr9jjpmaotgNIl6oap5d5IF1cIQUXuF1Y
-----END PRIVATE KEY-----
```

### Authentication Example

> 첫번째 예시에서는 인증 keypair를 가진 topic publisher와 subscriber를 생성해보려 합니다. 더불어 wireshark를 통해 암호화된 topic message도 확인해보겠습니다.

- sros2 키워드를 사용하여 두 세트의 인증서를 생성합니다.

```bash
ros2 security create_key demo_keystore /talker_listener/talker
ros2 security create_key demo_keystore /talker_listener/listener
```

- keystore 폴더 내부에 인증서와 permission이라는 config 파일이 추가되었습니다.

```bash
├── enclaves
│   ├── governance.p7s
│   ├── governance.xml
│   └── talker_listener
│       ├── listener
│       │   ├── cert.pem
│       │   ├── governance.p7s -> ../../governance.p7s
│       │   ├── identity_ca.cert.pem -> ../../../public/identity_ca.cert.pem
│       │   ├── key.pem
│       │   ├── permissions_ca.cert.pem -> ../../../public/permissions_ca.cert.pem
│       │   ├── permissions.p7s
│       │   └── permissions.xml
│       └── talker
│           ├── cert.pem
│           ├── governance.p7s -> ../../governance.p7s
│           ├── identity_ca.cert.pem -> ../../../public/identity_ca.cert.pem
│           ├── key.pem
│           ├── permissions_ca.cert.pem -> ../../../public/permissions_ca.cert.pem
│           ├── permissions.p7s
│           └── permissions.xml
├── private
│   ├── ca.key.pem
│   ├── identity_ca.key.pem -> ca.key.pem
│   └── permissions_ca.key.pem -> ca.key.pem
└── public
    ├── ca.cert.pem
    ├── identity_ca.cert.pem -> ca.cert.pem
    └── permissions_ca.cert.pem -> ca.cert.pem
```

예제를 실행하기 전, 일반적인 Topic 통신 시 String data가 패킷에 노출되는 모습을 살펴보겠습니다. wireshark를 실행시킨 뒤, 커멘드 라인을 실행시키고 패킷을 수집합니다.

```bash
# Terminal 1
ros2 run demo_nodes_cpp talker
# Terminal 2
ros2 run demo_nodes_cpp listener
```

![sros0.png](/kr/advanced_contents_ros2/images2/sros0.png?height=300px)

⇒ RTPS Heartbeat 패킷을 살펴보면, Hello World 라는 데이터가 고스란히 노출되어 있는 모습을 확인 가능합니다.

![sros1.png](/kr/advanced_contents_ros2/images2/sros1.png?height=500px)

- 이제 보안 옵션을 적용해보겠으며, sros2에서는 다양한 환경 변수를 통해 보완 관련 옵션을 변경할 수 있도록 하고 있습니다. security를 enable 시켜보겠습니다.

```bash
export ROS_SECURITY_KEYSTORE=~/sros2_demo/demo_keystore
export ROS_SECURITY_ENABLE=true
export ROS_SECURITY_STRATEGY=Enforce
```

- 다음으로, 사용한 RMW DDS를 변경해줍니다. 안타깝게도 CycloneDDS의 Debian Package는 보안 plugin을 제공하지 않기 때문에 보안 옵션 적용 시 아래와 같은 에러가 발생합니다. (소스 코드 빌드를 하면 해결됩니다.)

```bash
$ ros2 run demo_nodes_cpp talker --ros-args --enclave /talker_listener/talker
[INFO] [1674150011.742615669] [rcl]: Found security directory: /home/kimsooyoung/sros2_demo/demo_keystore/enclaves/talker_listener/talker
1674150011.756184 [19]     talker: Could not load Authentication library: dds_security_auth: cannot open shared object file: No such file or directory
1674150011.756216 [19]     talker: Could not load Authentication plugin.
1674150011.756221 [19]     talker: Could not load security
[ERROR] [1674150011.757250268] [rmw_cyclonedds_cpp]: rmw_create_node: failed to create DDS participant

>>> [rcutils|error_handling.c:108] rcutils_set_error_state()
This error state is being overwritten:
...
```

- 따라서, 이번 예시를 위해 Fast DDS로 RMW를 변경하겠습니다.

```bash
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

{{% notice note %}}
서로 다른 Vendor 끼리는 보안 plugin이 호환되지 않습니다. (보안은 표준이 아닙니다.)
{{% /notice %}}

{{% notice note %}}
rmw_connextdds ( rit DDS )는 유료 사용 시 보안 옵션을 사용 가능하며 30일 무료판을 사용하셔도 됩니다. ⇒ [https://www.rti.com/free-trial](https://www.rti.com/free-trial)

{{% /notice %}}

- 이제 pub-sub 예시를 실행해보겠습니다.

```bash
# Terminal 1
$ cd ~/sros2_demo/
$ export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
$ ros2 run demo_nodes_cpp talker --ros-args --enclave /talker_listener/talker
[INFO] [1674150302.868609218] [rcl]: Found security directory: /home/kimsooyoung/sros2_demo/demo_keystore/enclaves/talker_listener/talker
[INFO] [1674150303.901477001] [talker]: Publishing: 'Hello World: 1'
[INFO] [1674150304.901496377] [talker]: Publishing: 'Hello World: 2'

# Terminal 2
$ cd ~/sros2_demo/
$ export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
$ ros2 run demo_nodes_cpp listener
???
$ ros2 run demo_nodes_cpp listener --ros-args --enclave /talker_listener/listener
```

> 보안 옵션을 적용하지 않았기 때문에 Publisher가 동작하지 않습니다.

- 현 상황에서 다시금 wireshark를 실행하여 패킷을 수집해보겠습니다.

![sros2.png](/kr/advanced_contents_ros2/images2/sros2.png?height=300px)

- 사진과 같이 이제는 RTPS가 아닌, TLSv1.2 프로토콜을 사용하여 데이터가 오가게 됩니다. 해당 패킷을 열어보면 아래와 같이 암호화가 되어있습니다.

![sros3.png](/kr/advanced_contents_ros2/images2/sros3.png?height=500px)

- 더불어, RTPS 프로토콜 패킷을 살펴보면, enclave 옵션이 적용된 것도 확인 가능합니다.

![sros4.png](/kr/advanced_contents_ros2/images2/sros4.png?height=500px)

{{% notice note %}}
만약, 서로 다른 머신끼리 통신을 하고 싶다면, `demo_keystore`를 두 머신 모두 동일하게 소유하고 있어야 합니다. 옆자리에 다른 PC가 있다면 직접 실습해보세요
{{% /notice %}}

```bash
mkdir -p ~/sros2_demo/demo_keystore
scp -r talker USERNAME@oldschool.local:~/sros2_demo/demo_keystore
```

### **Access Control**

> 시스템의 보안 수준을 높이기 위해 각 노드가 수행할 수 있는 작업을 제한하는 Access Control을 정의할 수 있습니다. 이번 예시에서는 특정 topic만 사용 가능하도록 profile을 지정해보겠습니다.

- ROS 2에서 기본 제공하는 policy 예시를 복사합니다.

```bash
sudo apt update && sudo apt install subversion
cd ~/sros2_demo
svn checkout https://github.com/ros2/sros2/trunk/sros2/test/policies
```

- clone한 policy를 바탕으로 node의 action을 제한하는 description을 생성합니다.

```bash
cd ~/sros2_demo
ros2 security create_permission demo_keystore /talker_listener/talker policies/sample.policy.xml
ros2 security create_permission demo_keystore /talker_listener/listener policies/sample.policy.xml
```

{{% notice note %}}
위 작업은 talker / listener의 cert.pem이 있어야 실행 가능합니다.
{{% /notice %}}

- 일전 예시를 다시 실행시켜보겠습니다. 얼핏 보기에는 달라진 점이 전혀 없어 보입니다.

```bash
# Terminal 1
ros2 run demo_nodes_cpp talker --ros-args -e /talker_listener/talker
# Terminal 2
ros2 run demo_nodes_py listener --ros-args -e /talker_listener/listener
```

- 하지만, argument를 통해 사용하는 topic을 다른 것으로 바꾸는 순간, **not found in allow rule**라는 명령어와 함께 Node가 생성되지 않습니다.

```bash
# Terminal 1
ros2 run demo_nodes_cpp talker --ros-args -e /talker_listener/talker
# Terminal 2
ros2 run demo_nodes_py listener --ros-args -r chatter:=not_chatter -e /talker_listener/listener
[INFO] [1674152415.259964110] [rcl]: Found security directory: /home/kimsooyoung/sros2_demo/demo_keystore/enclaves/talker_listener/listener
2023-01-20 03:20:15.303 [SECURITY Error] rt/not_chatter topic not found in allow rule. (/tmp/binarydeb/ros-foxy-fastrtps-2.1.2/src/cpp/security/accesscontrol/Permissions.cpp:1271) -> Function check_create_datareader
2023-01-20 03:20:15.303 [SECURITY Error] Error checking creation of local reader bb.dd.19.9c.75.22.60.e0.55.c1.c0.3a|0.0.11.4 (rt/not_chatter topic not found in allow rule. (/tmp/binarydeb/ros-foxy-fastrtps-2.1.2/src/cpp/security/accesscontrol/Permissions.cpp:1271))
 -> Function register_local_reader
2023-01-20 03:20:15.303 [PARTICIPANT Error] Problem creating associated Reader -> Function createSubscriber
```

- 이번 예시에서 사용한 policy file입니다. - **policies/sample.policy.xml** topic publisher와 subscriber는 오직 /chatter topic만을 사용할 수 있게 access limit 걸리게 됩니다.

```bash
<?xml version="1.0" encoding="UTF-8"?>
<policy version="0.2.0"
  xmlns:xi="http://www.w3.org/2001/XInclude">
  <enclaves>
    <xi:include href="talker_listener.policy.xml"
      xpointer="xpointer(/policy/enclaves/*)"/>
    <xi:include href="add_two_ints.policy.xml"
      xpointer="xpointer(/policy/enclaves/*)"/>
    <xi:include href="minimal_action.policy.xml"
      xpointer="xpointer(/policy/enclaves/*)"/>
    <enclave path="/sample_policy/admin">
      <profiles>
        <profile ns="/" node="admin">
          <xi:include href="common/node.xml"
            xpointer="xpointer(/profile/*)"/>
          <actions call="ALLOW" execute="ALLOW">
            <action>fibonacci</action>
          </actions>
          <services reply="ALLOW" request="ALLOW">
            <service>add_two_ints</service>
          </services>
          <topics publish="ALLOW" subscribe="ALLOW">
            <topic>chatter</topic>
          </topics>
        </profile>
      </profiles>
    </enclave>
  </enclaves>
</policy>
```

---

**참고자료**

- [https://canonical.com/blog/security-vulnerabilities-on-the-data-distribution-service-dds](https://canonical.com/blog/security-vulnerabilities-on-the-data-distribution-service-dds)
- [https://github.com/ros2/sros2](https://github.com/ros2/sros2)
- [https://canonical.com/blog/what-is-sros-2](https://canonical.com/blog/what-is-sros-2)
