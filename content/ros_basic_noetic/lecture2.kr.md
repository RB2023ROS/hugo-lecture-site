---
title: "Lecture2 - Dev Env Setup"
date: 2022-12-25T13:36:33+09:00
draft: false
---

### Develpoment Environment Setup

> 프로젝트 기반의 강의인 만큼, 개발 환경이 구축되지 않으면 앞으로의 실습을 진행할 수 없습니다. 따라서 이번 파트에 많은 시간을 배정하였고, 힘들 수 있지만 끝까지 따라와주시면 감사합니다.

준비한 환경은 다음과 같습니다.

1. **Ubuntu Linux 설치와 ROS 설정**
2. **wsl2를 기반으로 한 Windows 내 ROS 설정**
3. **Docker 기반의 ROS 설정**

1,2,3 순서로 추천하는 설정입니다. 1번을 진행하시다가 도저히 못하겠으면 2번으로, 그래도 안되면 3번으로 진행해 주시면 됩니다.

---

### **Ubuntu Linux 설치와 ROS 설정**

**Linux**라고 함은 사실 OS 자체라기보다 OS의 기초가 되는 코어 소프트웨어에 가깝습니다. 이 코어를 사용하여 여러 버전의 OS가 개발되었으며 이를 **Liunx 배포판** 이라고 지칭합니다.

Ubuntu Linux는 리눅스 배포판 중 가장 널리 알려진 배포판으로, 영국의 Software회사인 Canonical과 우분투 재단에서 개발, 유지보수 및 배포를 하고 있습니다.

![lec2_0.png](/kr/ros_basic_noetic/images2/lec2_0.png?height=300px)

- image from : [omdriod.com](https://www.omdroid.com/104)

기존의 Windows 노트북을 사용하고 있었다면 **듀얼 부팅**이라는 것을 통해 Ubuntu + Windows를 모두 사용 가능합니다. 듀얼 부팅에 방법에 대해선 잘 설명한 영상들이 많으므로 링크로 대체하겠습니다.

- [How to Dual Boot Ubuntu 20.04 LTS and Windows 10](https://www.youtube.com/watch?v=-iSAyiicyQY&t=492s)
- [How to Dual Boot Ubuntu and Windows 11](https://www.youtube.com/watch?v=Z0T3uiCoEpA)

Ubuntu 20.04 설치를 확인 하셨다면 터미널을 실행한 뒤 아래의 명령어들을 실행합니다.

- terminator 설치

```python
sudo apt update
sudo apt install terminator -y
```

terminator 화면 분할 예시

- ctrl + alt + t : 터미널 실행
- ctrl + shift + e : 세로 분할
- ctrl + shift + o : 가로 분할
- ctrl + shift + w : 창 닫기
- alt + 화살표 : 창 간 이동

##### ROS / ROS 2 한줄 설치

```bash
cd ~/
sh -c "$(curl -fsSL https://raw.githubusercontent.com/kimsooyoung/ros_menu/main/scripts/setup.sh)"
…
Do you want to install ROS automatically? (y/N): y
```

다음으로, 시뮬레이션 프로그램인 Gazebo를 설치합니다. Gazebo는 ROS를 관리하는 조직인 OSRF에서 공식 지원하는 로봇 시뮬레이터입니다.

```bash
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" /etc/apt/sources.list.d/gazebo-stable.list'
wget https://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
sudo apt update
sudo apt install gazebo11 libgazebo11-dev -y
sudo apt install ros-foxy-gazebo-ros-pkgs -y
```

그림와 같이 Gazebo의 화면이 어둡고, 그림자가 보이지 않는다면 호환되는 그래픽 드라이버를 설치해야 합니다.

![pic.png](/kr/ros_basic_noetic/images2/pic.png?height=400px)

```bash
sudo ubuntu-drivers autoinstall
sudo reboot
# 장착된 장치 확인
ubuntu-drivers devices
```

마지막으로, catkin build system을 사용하기 위해 아래 커멘드 명령어를 실행합니다.

```bash
sudo apt update
sudo apt install python3-catkin-tools
```

### **Windows + WSL2 설정**

**WSL(Windows Subsystem for Linux)** 이란, 리눅스용 윈도우 하위 시스템의 약자로, 윈도우 10에서 네이티브로 리눅스 실행 파일(ELF)을 실행하기 위한 호환성 계층입니다.

Windows10부터 WSL을 지원하며, Windows 2004(20H1) version부터 WLS2를 지원하고 있습니다.

- 최신 Windows 업데이트 적용 ("Windows Key ⇒ 업데이트"로 이동하여 최신 업데이트를 적용합니다.)

![lec2_1.png](/kr/ros_basic_noetic/images2/lec2_1.png?height=300px)

- powershell을 관리자 권한으로 실행한 뒤, 설치 되어있는 WSL 2를 활성화시킵니다.
  ![lec2_2.png](/kr/ros_basic_noetic/images2/lec2_2.png?height=300px)

```powershell
> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

배포 이미지 서비스 및 관리 도구
버전: 10.0.19041.844

이미지 버전: 10.0.19043.1348

기능을 사용하도록 설정하는 중 [==========================100.0%==========================]
작업을 완료했습니다.

> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
배포 이미지 서비스 및 관리 도구
버전: 10.0.19041.844

이미지 버전: 10.0.19043.1348

기능을 사용하도록 설정하는 중 [==========================100.0%==========================]
작업을 완료했습니다.
```

- MS Store로 이동하여 windows terminal을 설치합니다.
  ![lec2_3.png](/kr/ros_basic_noetic/images2/lec2_3.png?height=300px)

##### windows terminal 화면 분할 예시

- alt + shift + - : 가로 분할
- alt + shift + + : 세로 분할
- ctrl + shift + w : 창 닫기
- alt + 화살표 : 창 간 이동

[다음 링크](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)를 통해 프로그램을 다운받고 WSL 2 Linux 커널 업데이트를 설치/진행 후 재부팅합니다.

커널 업데이트를 마친 다음, powershell에 아래 커맨드 라인을 입력하여 WSL 2를 기본 사용하도록 설정합니다.

![lec2_4.png](/kr/ros_basic_noetic/images2/lec2_4.png?height=300px)

```powershell
> wsl --set-default-version 2

WSL 2와의 주요 차이점에 대한 자세한 내용은 https://aka.ms/wsl2를 참조하세요
작업을 완료했습니다.
```

> 이 시점에서 발생할 수 있는 문제들을 한차례 살펴보고 가겠습니다.

- `wsl --set-default-version 2` 에서 작업 완료 메세지가 등장하지 않는 경우
  ⇒ 제일 처음 명령어부터 제가 보여드린 예시와 동일한 결과를 얻었는지 확인해보세요

- Linux용 Windows 하위 시스템 설정 여부도 확인합니다.

![lec2_5.png](/kr/ros_basic_noetic/images2/lec2_5.png?height=350px)

##### 지금까지 진행한 작업들이 제대로 설정 되어있는지 확인해봅시다.

```powershell
> wsl -l -v
  NAME                 STATE            VERSION
* Ubuntu-20.04    Running         2
```

- 문제가 없다면 MS Store 진입 후, Ubuntu 20.04 버전을 설치합니다.

![lec2_6.png](/kr/ros_basic_noetic/images2/lec2_6.png?height=300px)

설치 이후, **열기** 버튼을 눌러 초기 실행을 하고 **username**, **password**를 설정합니다.

> 이 시점에서 발생할 수 있는 문제들도 다시 한차례 살펴보고 가겠습니다.

- ‘The WSL Optional Component is not Enabled. Please Enable it and Try again’ 오류가 발생하는 경우

```powershell
> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

- 0x80370102 에러가 발생한 경우
  ⇒ 부팅 BIOS에서 가상화 기능을 활성화해줍니다.

- 0xc03a001a 에러가 발생한 경우
  ⇒ [다음 블로그 포스팅](https://goaloflife.tistory.com/193)을 참고합니다.

설치 이후에는 windows terminal에서 **Ubuntu Terminal**을 선택하여 실행 및 진입이 가능합니다. 설정 탭을 통해 Ubuntu 20.04를 기본 터미널로 설정 후 저장합시다.

![lec2_7.png](/kr/ros_basic_noetic/images2/lec2_7.png?height=350px)

- ROS / ROS 2를 한줄 설치합니다.

```bash
$ cd ~/
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/kimsooyoung/ros_menu/main/scripts/setup.sh)"
Installing Neuron Startup Menu...
Cloning into '/home/kimsooyoung/ros_menu'...
remote: Enumerating objects: 583, done.
remote: Counting objects: 100% (290/290), done.
remote: Compressing objects: 100% (193/193), done.
remote: Total 583 (delta 179), reused 173 (delta 93), pack-reused 293
Receiving objects: 100% (583/583), 154.50 KiB | 3.22 MiB/s, done.
Resolving deltas: 100% (340/340), done.
Do you want to install ROS automatically? (y/N): y
```

- 설치가 완료되었다면, 앞으로 터미널을 새로 등장시킬 때마다 다음과 같이 사용할 ROS 버전을 묻게 됩니다.

![lec2_8.png](/kr/ros_basic_noetic/images2/lec2_8.png?height=300px)

### GUI 인터페이스 설정

- 링크를 통해 VcXsrv를 설치합니다. ⇒ [https://sourceforge.net/projects/vcxsrv/](https://sourceforge.net/projects/vcxsrv/)
- 설치 이후, XLaunch를 실행하고, 사진과 같은 설정을 진행합니다.

![lec2_9.png](/kr/ros_basic_noetic/images2/lec2_9.png?height=200px)

앞으로 재부팅 시마다 이 설정을 반복해줘야 하며, 종종 Gazebo의 Memory Leak로 인해 화면이 종료되지 않는 경우가 있는데, 이런 경우 XLaunch를 강제 종료해주면 됩니다.

- 설정이 잘 되었는지 Gazebo를 실행해봅시다.

![lec2_10.png](/kr/ros_basic_noetic/images2/lec2_10.png?height=300px)

```bash
export GAZEBO_IP=127.0.0.1
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0
export LIBGL_ALWAYS_INDIRECT=0

gazebo
```

{{% notice note %}}
Gazebo의 실행 시 그래픽 카드 인식 문제가 발생하면 아래와 같이 어두운 화면이 등장합니다.
{{% /notice %}}


![gazebo_black.png](/kr/ros_basic_noetic/images2/gazebo_black.png?height=300px)

* 이러한 경우,  World 탭 ⇒ Scene ⇒ Shadows 옵션을 토글해주시면 됩니다.

![scene.png](/kr/ros_basic_noetic/images2/scene.png?height=300px)

- 마지막입니다! `~/.bashrc`를 수정합니다.

```bash
echo 'export GAZEBO_IP=127.0.0.1' >> ~/.bashrc
echo "export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0 " >> ~/.bashrc
echo 'export LIBGL_ALWAYS_INDIRECT=0' >> ~/.bashrc
```

> 발생할 수 있는 오류 상황들을 살펴보고 가겠습니다.

- gazebo 실행 시 symbol error

```bash
$ gazebo
gazebo: symbol lookup error: /usr/lib/x86_64-linux-gnu/libgazebo_common.so.9: undefined symbol: _ZN8ignition10fuel_tools12ClientConfig12SetUserAgentERKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
```

해결 방법 ⇒ `sudo apt upgrade -y libignition-math2`

- Gazebo 자체가 실행되지 않는 경우

  PC 재시작 후 백신 프로그램을 종료시킵니다.
  제어판 ⇒ 시스템 및 보안 ⇒ Windows 방화벽 ⇒ 고급 설정 ⇒ 인바운드 규칙 ⇒ VcXsrv 사용 허용

![lec2_11.png](/kr/ros_basic_noetic/images2/lec2_11.png?height=200px)

- Gazebo는 실행되지만, Grid (실선)이 등장하지 않는 경우

  안타깝지만 그래픽 드라이버 문제일 가능성이 큽니다. (Window ⇒ 업데이트 확인 진입 후 가장 최신 버전으로 모두 업그레이드를 실행해줍니다.)

---

### **Docker 기반의 ROS 설정**

> 이전 설치 모두 실패하신 경우 사용할 수 있는 최후의 방법이자. MacOS 사용자들을 위한 설정입니다.

- docker desktop for windows를 설치합니다. ⇒ [https://docs.docker.com/desktop/windows/install/](https://docs.docker.com/desktop/windows/install/)

```bash
# 설치 확인
> docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

- 설치 확인 이후 동일한 명령 프롬프트에서 다음 커멘드 라인을 입력하여 도커 이미지를 다운받습니다. (해당 docker image는 강의에 필요한 작업을 제가 미리 준비해둔 것입니다.)

```bash
docker pull tge1375/du_ros_noetic:0.0.0
```

- docker 이미지를 최초 실행 시, 방금의 명령 프롬프트에서 아래와 같은 커멘드 라인을 입력해 주세요.

```bash
docker run -it -p 6080:80 --name ros_noetic tge1375/du_ros_noetic:0.0.0
```

- 입력 후 링크를 통해 로컬 VNC 서버로 접속이 가능합니다. 첫 실행 시 방화벽 보안 경고가 등장하며, 이때 반드시 엑세스를 허용해줍니다. ⇒ ([http://127.0.0.1:6080/](http://127.0.0.1:6080/))

![lec2_12.png](/kr/ros_basic_noetic/images2/lec2_12.png?height=300px)

> 위와 같은 화면이 등장했다면 noVNC에 접속이 성공한 것입니다.

- 최초 명령 프롬프트에서 실행한 이후 다시 실행시키고 싶은 경우, 혹은 중지시키고 싶은 경우 docker desktop을 사용하면 매우 편리하게 작업이 가능합니다.

![lec2_13.png](/kr/ros_basic_noetic/images2/lec2_13.png?height=400px)

### VScode 셋업

> VNC내에서의 개발은 매우 불편하기 때문에, VScode를 통해 코드 개발을 하고자 합니다. 다음과 같은 extension들을 설치합시다.

- Docker
- Docker Explorer
- Remote Development

설치 이후, `cmd + p`를 통해 Remote-Containers를 입력하면, 현재 실행 중인 컨테이너의 목록을 조회할 수 있습니다.

이들 중 원하는 컨테이터를 선택하면, 새로운 vscode가 새로이 실행되며 작업 폴더를 설정하게 됩니다.

![lec2_14.png](/kr/ros_basic_noetic/images2/lec2_14.png?height=200px)

### Linux Commands

셋팅한 환경은 모두 리눅스를 기반으로 합니다.

때문에 최소한의 리눅스 커멘드 지식이 필요하며, 짚고 넘어가겠습니다.

**리눅스 필수 명령어**

- **cd <디렉토리>** : 해당 디렉토리로 이동하기 (절대 경로와 상대 경로, ../ 와 ~/)
- **pwd** : 현재 위치한 절대 경로
- **ls** : 현재 디렉토리에 위치하는 모든 파일들 (ls -al 옵션)
- **touch <파일 이름>** : 해당 이름을 가진 파일 생성
- **source** : 쉘 스크립트 실행하기 (source ~/.bashrc)

**강의를 위해 알아야 할 추가 명령어들**

- **sudo** : root 권한으로 실행하기
- **apt install / remove / purge** : 리눅스 배포 패키지 설치
- **apt update** : apt의 상태를 최신화, 설치된 패키지들을 최신화하는 과정을 포함함

**그 외 알아두면 좋은 것들**

- **top / htop** : 리눅스의 작업 관리자
- **ps aux** : 실행 중인 프로세스 출력 (ps aux | grep ros)
- **tee <파일 이름>** : 터미널 로그를 기록하기
- **xdg-open .** : 현재 위치에서 파일 탐색기 열기

**강의 중 설정된 Alias 설명**

- **eb** : edit bashrc의 약자로 ~/.bashrc 파일을 gedit을 통해 수정할 수 있습니다.
- **sb** : source bashrc의 약자로 수정된 ~/.bashrc 파일을 반영시킵니다.
- **killg** : 실행중인 Gazebo관련 모든 프로그램을 종료시킵니다.

**유용한 프로그램들 설치하기**

```bash
sudo apt install gedit
sudo apt install terminator -y
```

**코드 에디터와 IDE 추천**

- **VSCode** - [https://code.visualstudio.com/](https://code.visualstudio.com/)
- **PyCharm** - [https://www.jetbrains.com/ko-kr/pycharm/download](https://www.jetbrains.com/ko-kr/pycharm/download/#section=mac)
- **CLion** - [https://www.jetbrains.com/ko-kr/clion/](https://www.jetbrains.com/ko-kr/clion/)

---

**참고자료**

- [https://www.youtube.com/watch?v=DW7l9LHdK5c](https://www.youtube.com/watch?v=DW7l9LHdK5c)
- [https://www.lainyzine.com/ko/article/how-to-install-wsl2-and-use-linux-on-windows-10/](https://www.lainyzine.com/ko/article/how-to-install-wsl2-and-use-linux-on-windows-10/)
- [https://github.com/Tiryoh/docker-ros2-desktop-vnc](https://github.com/Tiryoh/docker-ros2-desktop-vnc)
- [https://89douner.tistory.com/123](https://89douner.tistory.com/123)
