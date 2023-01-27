---
title: "Lecture3 - SROS"
date: 2022-12-21T03:02:41+09:00
draft: false
---

> 이전 데모를 통해 ROS의 보안 취약성에 대해 살펴보았습니다. 이러한 취약점들을 개선하기 위한 방법으로 SROS에 대해 배워보겠습니다.

{{% notice note %}}
sros는 마지막 commit 2018년 이후 개발이 사실상  멈춘 프로젝트입니다. ROS 개발자들이 보안을 위해 어떠한 노력을 들였는지 정도만 살펴보고, 데모와 자세한 내용은 ROS 2 강의에서 이어나가도록 하겠습니다.
{{% /notice %}}

![sros.png](/kr/advanced_contents_ros1/images3/sros.png?height=200px)

* UDPROS가 roscpp만 지원했던 것처럼, sros도 rospy와 사용할 시 제한이 있습니다.

|  | TCPROS | UDPROS  |
| --- | --- | --- |
| rospy | X | O |
| roscpp | O | O |
| rosjava | O | O |

---


* **sros**는 ROS의 client 라이브러리 소스 코드에 보안을 적용하여 새롭게 개발한 ROS Client Library입니다. sros가 사용되는 절차는 다음과 같습니다.

1. sros 빌드 (기존 ROS 코드가 아닌)
2. sroskeyserver 실행
3. sroscore 실행
4. sroslaunch를 통해 응용 프로그램 실행

sros에서 제공되는 기능들은 다음과 같습니다.

- 소켓 전송에 대한 Native TLS 지원
- ROS 노드단에서 자동 생성된 node key pairs 관리
- 리눅스 커널 단에서의 보안 강화 ([AppArmor](http://wiki.ros.org/AppArmor) profile library)
- x.509 인증서 사용 지원
- etc…

**SROS**는 ROS 내의 보안 강화를 위한 방법들의 집합을 지칭하며, 크게 3 계층의 보안을 제공합니다. 각 계층들에 대한 간단한 설명을 해보겠습니다.

1. **Transport Security**
2. **Access Control**
3. **Process Profile**


### Transport Security and ROS


SROS는 TLS를 활용하여 ROS 관련 트래픽을 보호합니다. 이를 통해 악의적인 행위자의 redirection, 메시지를 탈취하여 중간에 수정한다거나 Spoofing을 막을 수 있습니다. 또한 시스템/물리적 수준 액세스를 통해 로봇이 손상되더라도 TLS에서 제공하는 Forward Secrecy 덕분에 이전에 기록된 트래픽도 보호할 수 있습니다. 

SROS가 TLS를 적용한 방법은, 네트워크 스택과 ROS 클라이언트 라이브러리 사이에 TLS를 끼워 넣는 것입니다. 이를 통해 SROS는 모든 Socket 수준의 ROS 통신을 Wrapping할 수 있으며, 사용자는 기존 ROS 프로그램 코드를 수정하지 않고도 SROS의 모든 이점을 적용할 수 있습니다.

![tls.png](/kr/advanced_contents_ros1/images3/tls.png?height=300px)

* image from : [hpbn.co](https://hpbn.co/transport-layer-security-tls/)
#### SROS Keyserver

**Keyserver**는 ROS node에게 인증된 key를 분배하고 관리하는 역할을 합니다. Keyserver는 ROS와 독립적으로 실행되며, 다른 기기에서 실행되어도 무관합니다. 만약 자신만의 PKI 방식을 갖고 있다면 기본 제공되는 keyserver를 대체할 수 있습니다. 

SROS는 기본적으로 보안에 익숙치 않고, TLS 시스템을 구축하는 개념에 서투른 사용자를 대상으로 만들어졌습니다.

![ca.png](/kr/advanced_contents_ros1/images3/ca.png?height=350px)

{{% notice note %}}
Keyserver를 통한 Node configuration과 실행은 아래 링크를 참고합니다. **[How is a Keyserver used in SROS](http://wiki.ros.org/SROS/Tutorials/KeyserverAndSROS)**
{{% /notice %}}


#### Running Keyserver

> SROS의 Node들은 실행 전 Keyserver를 통한 key 생성과 CA를 통한 인증이 필요합니다. 이 과정 없이 생성된 node들은 rosmaster와의 연결이 거부됩니다.

- keyserver 실행

```bash
$ sroskeyserver 
Starting an XML-RPC server to bootstrap SSL key distribution...
Certificate generated: root
Certificate generated: master
sleeping until keyserver has generated the initial keyring...
Horray, the keyserver is now open for business. 
```

- SROS는 ~/.ros 폴더 내 자신만의 configuration을 구축합니다. 이 내부에는 인증에 필요한 각종 파일들이 위치하게 됩니다.

```powershell
$ tree ~/.ros/sros
  /root/.ros/sros
  ├── config
  │   ├── keyserver_config.yaml
  │   └── policy_config.yaml
  └── keystore
      ├── ca
      │   ├── master
      │   │   ├── master.cert
      │   │   └── master.pem
      │   └── root
      │       ├── root.cert
      │       └── root.pem
      ├── capath
      │   ├── d11d170d.0 -> /root/.ros/sros/keystore/ca/root/root.cert
      │   └── f4ad5f10.0 -> /root/.ros/sros/keystore/ca/master/master.cert
      └── utils
          └── keyserver
              ├── keyserver.cert
              └── keyserver.pem

  8 directories, 10 files
```

1. 초기 실행 시 로딩된 configuration 파일은 keyserver에 CA를 로드하도록 지시하고, keystore에 존재하지 않는 경우 방법을 지시합니다. 
2. root와 중간 Master CA가 초기화되면 keyserver는 자체 transport 인증서와 keypair 로딩을 요청받습니다.
3. keyserver는 자체 key를 사용하여 자신에게 연결을 시도하고, keyserver의 API 끝이 온라인 상태이며 작동 중인지 확인합니다. 
4. 이 작업이 모두 완료되면 keyserver가 연결 Node의 요청을 받을 준비가 됩니다.


## Access Control

> Access Control을 통해 ROS에서 사용하는 리소스와 Action을 제어할 수 있습니다. SROS 튜토리얼에서는 아래와 같은 ROS API들에 대하여 Access Control 설정이 가능하다고 이야기하고 있습니다.

- topics
    - publish
    - subscribe
- parameters
    - read
    - write
- services
    - advertise
    - call
- ROS API
    - master calls
    - slave calls
    

Access Control의 실제 구현 방식에 있어 SROS에서 제안하는 두가지 방법을 간략히 정리해보았습니다.

#### Certificate Embedding

* SROS는 Node간 보안 네트워크 소켓 연결(TLS 핸드셰이크) 시 [X.509](https://en.wikipedia.org/wiki/X.509) 인증서를 사용합니다. 
* 검증된 방식을 사용하고 계산 효율이 좋다는 장점이 있지만, 이러한 방식은 Policy data가 public 이기 때문에 node, topic 이름이 노출될 염려가 있습니다.

#### Online Arbiter

* 중앙 집중식으로 데이터 통신을 중재할 수 있습니다. global policy에 대한 정보를 알고 있는 중앙 통제 프로세스가 요청된 Node와 접촉하고 허용여부를 판단합니다.
* 이는 ROS Master Node가 사용하는 기존 DNS 방식과 유사하지만 네임스페이스 대신 access control restriction을 사용한다는 차이점이 있습니다.

## Linux Security Modules

> AppArmor(애플리케이션 Armor)는 프로그램 프로파일로 프로그램의 기능을 제한할 수 있도록 하는 리눅스 커널 보안 모듈입니다. SROS 튜토리얼에서는 AppArmor의 ROS 호환 프로파일 예시를 제공하고 있습니다.


![apparmor.png](/kr/advanced_contents_ros1/images3/ca.png?height=300px)

- image from : [https://wiki.apparmor.net/](https://wiki.apparmor.net/)

AppArmor를 활용하여 ROS 프로세스를 격리하고 보호할 수 있으며, 이를 통해 악의적이거나 오작동하는 Node가 Host 시스템과 다른 Node에 미칠 수 있는 영향을 제한할 수 있습니다.

예를 들면, 특정 IMU Node에 대해 전용 Bus를 예약하고 노드가 해당 Serial Port를 사용하지 못하도록 할 수 있습니다. 이와 같이 특정 노드에 대한 프로파일을 지정하고 예약된 장치에만 읽고 쓸 수 있는 권한을 지정할 수 있습니다.

- AppArmor 설치

```powershell
sudo apt-get install apparmor apparmor-utils
```

- AppArmor를 사용하기 위해 프로파일 configuration file을 작성해야 합니다. github를 통해 ROS를 위한 프로파일 예시를 제공하고 있으며, 이를 AppArmor의 설정 폴더로 복사합니다.

```powershell
git clone https://github.com/ros-infrastructure/apparmor_profiles
sudo cp --recursive apparmor_profiles/profiles /etc/apparmor.d
```

- 각 프로파일들에 대한 설명은 아래와 같습니다.

```powershell
$ tree apparmor_profiles/profiles/
apparmor_profiles/profiles/
├── ros # root profile library folder
│   ├── base # base networking and signal abstractions for ROS
│   ├── node # node abstractions executables needed for ros nodes
│   ├── nodes # additional node specific abstractions
│   │   └── roslaunch # additional file and signal abstractions for roslaunch
│   └── python # node abstractions needed for python nodes
└── tunables # root tunables folder for AppArmor profile variables
    ├── ros # path abstractions executables needed for ros nodes
    └── ros.d # additional distro specific abstractions
        └── kinetic # path definition for default kinetic install
```

- 프로파일을 변경하게 되면, AppArmor를 새롭게 재시작해야 합니다. 이는 아래 키워드를 사용합니다.

```powershell
sudo service apparmor restart
```

SROS Tutorial에서는 Topic을 사용하는 talker - listener에 대한 프로파일과 실행 예시를 제공하고 있습니다. Custom 프로파일의 작성법과 실행 방법은 링크로 대체합니다.

- [Customizing AppArmor Profiles for ROS](http://wiki.ros.org/SROS/Tutorials/CustomizingAppArmorProfilesForROS)

---

**참고자료**

- [http://wiki.ros.org/SROS/CommandLineTools](http://wiki.ros.org/SROS/CommandLineTools)
- [http://wiki.ros.org/SROS/Concepts/PolicyDissemination](http://wiki.ros.org/SROS/Concepts/PolicyDissemination)
- [http://wiki.ros.org/SROS/Tutorials/TrasportSecurityAndROS](http://wiki.ros.org/SROS/Tutorials/TrasportSecurityAndROS)