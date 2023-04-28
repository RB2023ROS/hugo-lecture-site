---
title: "Lecture2. Dev Env Setup"
date: 2023-04-28T10:51:29+09:00
draft: false
---

![Untitled.png](/kr/ros2_foxy/images2/Untitled.png?height=400px)

### Develpoment Environment Setup

프로젝트 기반의 강의인 만큼, 개발 환경이 구축되지 않으면 앞으로의 실습을 진행할 수 없습니다.

따라서 이번 파트에 많은 시간을 배정하였고, 힘들 수 있지만 끝까지 따라와주시면 감사합니다.

준비한 환경은 다음과 같습니다.

1. **Ubuntu Linux & ROS 2 Setup**
2. **Windows 11 & WSL 2 Setup**
3. **Windows 10 & WSL 2 Setup**

1,2,3 순서로 추천하는 설정입니다. 강의 컨텐츠를 따라오면서 선호하는 설정에 맞추어 작업하시다가 도저히 해결할 수 없는 문제가 발생한 경우 github issue를 발행하시거나 현장 강의의 경우 저를 불러 주시면 됩니다.

## **Ubuntu Linux & ROS 2 Setup**

### 해당 메뉴얼은 우분투 리눅스 20.04버전을 사용하는 PC를 위한 ROS 2 설치 튜토리얼입니다.

기존 Windows를 사용하고 있었다면 **듀얼 부팅**이라는 것을 통해 Ubuntu + Windows를 모두 사용 가능합니다. 듀얼 부팅에 방법에 대해선 잘 설명한 영상들이 많으므로 링크로 대체하겠습니다.

=> [https://www.itechguides.com/dual-boot-ubuntu-windows-10/](https://www.itechguides.com/dual-boot-ubuntu-windows-10/)

![Untitled1.png](/kr/ros2_foxy/images2/Untitled1.png?height=400px)

{{% notice note %}}
Dual Booting 관련 자료 링크

- [How to Dual Boot Ubuntu 20.04 LTS and Windows 10](https://www.youtube.com/watch?v=-iSAyiicyQY&t=492s)
- [How to Dual Boot Ubuntu and Windows 11](https://www.youtube.com/watch?v=Z0T3uiCoEpA)

{{% /notice %}}

> ROS 2를 설치하기 전, 유용한 유틸리티 프로그램들을 몇가지 소개해드리겠습니다.

- ternimator 설치 - 다중 터미널을 지원하는 훌륭한 터미널 프로그램입니다.

```python
sudo apt update
sudo apt install terminator -y
```

terminator 화면 분할 예시

1. ctrl + shift + e : 가로 분할
2. ctrl + shift + o : 세로 분할
3. ctrl + shift + w : 창 닫기
4. alt + 화살표 : 창 간 이동

- peek - gif 제작 시 유용한 프로그램입니다. 저는 Gazebo와 같은 시뮬레이션 상황의 디버깅 시 사용하고 있습니다.

```jsx
sudo add-apt-repository ppa:peek-developers/stable
sudo apt-get update
sudo apt install peek
```

- 다음으로, 제가 미리 준비해둔 스크립트를 통해 ROS / ROS 2의 한줄 설치를 진행합니다. 설치 여부를 묻는 질문에는 y를 입력하시면 되며, OpenVINO를 설치하는 질문에서는 n를 입력하시면 됩니다.

```bash
cd ~/
sh -c "$(curl -fsSL https://raw.githubusercontent.com/kimsooyoung/ros_menu/main/scripts/setup.sh)"
…
Do you want to install ROS automatically? (y/N): y
```

- 설치 후, 여러분들의 PC는 아래와 같이 실행 전, ROS 사용 여부를 선택하실 수 있습니다.

```bash
********* RoadBalance Startup Menu for ROS ***********
* Usage: Please select the following options to load *
*        ROS environment automatically.              *
******************************************************
0) Do nothing
1) ROS 1 noetic
2) ROS 2 foxy
3) ROS 2 galactic
4) ROS2/ROS1_bridge
h) Help
Please choose an option: 2
------------------------------------------------------
* ROS_DOMAIN_ID = 30
------------------------------------------------------
...
```

{{% notice note %}}
해당 setup은 ADLINK의 오픈소스를 참고하였음을 밝힙니다. ADLINK는 DDS Vendor 중 하나로 ROS 2 보급에도 힘쓰고 있답니다. 😃
{{% /notice %}}

- Gazebo와 Rviz2의 설치 여부를 함께 확인해 봅시다.

```jsx
# 1. gazebo install check
gazebo

# 2. rviz2 install check
rviz2
```

> 그림와 같이 Gazebo의 화면이 어둡거나, 그림자가 보이지 않는다면 호환되는 그래픽 드라이버를 설치해야 하는 상황입니다. 당장 크게 불편하지 않다면 상관은 없습니다.

![Untitled2.png](/kr/ros2_foxy/images2/Untitled2.png?height=300px)

- 혹시나 Gazebo가 설치되지 않았다면, 아래의 커멘드 라인을 통해 설치해줍니다.

```bash
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" /etc/apt/sources.list.d/gazebo-stable.list'
wget https://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
sudo apt update
sudo apt install gazebo11 libgazebo11-dev -y
sudo apt install ros-foxy-gazebo-ros-pkgs -y
sudo apt install ros-noetic-gazebo-ros-pkgs -y
```

## **Windows 11 & WSL 2 Setup**

> 해당 메뉴얼은 Windows11을 사용하는 PC를 위한 ROS 2 설치 튜토리얼입니다.

- 설치 전, “Windows Key ⇒ 업데이트”를 통해 최신 업데이트를 반영합니다. (상황에 따라 재부팅이 필요할 수 있습니다.)

![Untitled3.png](/kr/ros2_foxy/images2/Untitled3.png?height=300px)

- 더불어, Ctrl + Alt + Delete ⇒ 작업관리자 ⇒ 왼쪽 성능 탭을 통해 가상화 기능이 Enable 되어 있는지 확인합니다.

![Untitled4.png](/kr/ros2_foxy/images2/Untitled4.png?height=350px)

{{% notice note %}}
이 시점에서 문제가 있다면, 링크를 통해 문제를 해결해봅시다.
{{% /notice %}}

⇒ [https://www.manualfactory.net/15704](https://www.manualfactory.net/15704)

- 다음으로, 터미널 프로그램을 관리자 권한으로 실행합니다. 이는 “Windows Key + X” ⇒ 터미널(관리자)를 통해 실행 가능합니다.

![Untitled5.png](/kr/ros2_foxy/images2/Untitled5.png?height=400px)

- 관리자 권한 Powershell에서 아래와 같은 커멘드 라인을 입력합니다. 사진과 같이 가상 머신 플렛폼 / WSL / Ubuntu가 모두 한번에 설치됩니다. - 여기에서 설치되는 Ubuntu는 22.04버전입니다.

```jsx
wsl --install
```

![Untitled6.png](/kr/ros2_foxy/images2/Untitled6.png?height=350px)

- 다음으로 동일 터미널에서 아래의 두 커멘드 라인을 입력합니다.

```jsx
# 1. WSL Enable
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 2. VirtualMachinePlatform Enable
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

> 아래와 같이 모든 작업이 성공적으로 완료되어야 합니다!

```jsx
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

- WSL 2 설정이 완료되었는지 확인을 해볼까요?

```jsx
PS C:\Users\tge13> wsl -l -v
  NAME            STATE           VERSION
* Ubuntu          Stopped         2
```

- 설치된 Ubuntu는 22.04 버전으로, 강의에서 사용되는 20.04버전은 MS Store에서 다시 다운로드받아야 합니다.

![Untitled7.png](/kr/ros2_foxy/images2/Untitled7.png?height=400px)

- 설치 이후, “열기” 버튼을 눌러 초기 실행을 하면 아래과 같은 터미널을 확인할 수 있으며, username, password를 설정합니다. - password는 보이지 않음에 유의합니다.

![Untitled8.png](/kr/ros2_foxy/images2/Untitled8.png?height=200px)

- 다음으로, 제가 미리 준비해둔 스크립트를 통해 ROS / ROS 2의 한줄 설치를 진행합니다. 설치 여부를 묻는 질문에는 y를 입력하시면 되며, OpenVINO를 설치하는 질문에서는 n를 입력하시면 됩니다.

```bash
cd ~/
sh -c "$(curl -fsSL https://raw.githubusercontent.com/kimsooyoung/ros_menu/main/scripts/setup.sh)"
…
Do you want to install ROS automatically? (y/N): y
```

- 설치 후, 여러분들의 PC는 아래와 같이 실행 전, ROS 사용 여부를 선택하실 수 있습니다.

```jsx
********* RoadBalance Startup Menu for ROS ***********
* Usage: Please select the following options to load *
*        ROS environment automatically.              *
******************************************************
0) Do nothing
1) ROS 1 noetic
2) ROS 2 foxy
3) ROS 2 galactic
4) ROS2/ROS1_bridge
h) Help
Please choose an option: 2
------------------------------------------------------
* ROS_DOMAIN_ID = 30
------------------------------------------------------
...
```

## GUI 인터페이스 설정

> Gazebo, Rviz2, Rqt와 같은 시각화 프로그램을 사용하기 위해 Windows용 X Server인 VcXsrv를 사용하고자 합니다.

1. 링크를 통해 VcXsrv를 설치합니다. ⇒ [https://sourceforge.net/projects/vcxsrv/](https://sourceforge.net/projects/vcxsrv/)
2. 설치 이후, “Windows Key” ⇒ XLaunch를 검색하여 실행하고, 사진과 같은 설정을 진행합니다.

![Untitled9.png](/kr/ros2_foxy/images2/Untitled9.png?height=200px)

| 첫번째 페이지 | 하단 Display Number를 -1에서 0으로 변경               |
| ------------- | ----------------------------------------------------- |
| 두번째 페이지 | 통과                                                  |
| 세번째 페이지 | Native opengl 체크해제 후 Disable access control 체크 |
| 네번째 페이지 | 통과                                                  |

{{% notice tip %}}
앞으로 재부팅 시마다 이 설정을 반복해줘야 하며, 종종 Gazebo의 메모리 누수로 인해 화면이 종료되지 않는 경우가 있는데, 이런 경우 XLaunch를 강제 종료해주면 됩니다.
{{% /notice %}}

![Untitled10.png](/kr/ros2_foxy/images2/Untitled10.png?height=400px)

- 설정이 잘 되었는지 Gazebo를 실행해봅시다. 터미널 상의 아래의 4줄 커멘드 라인을 입력합니다.

```bash
export GAZEBO_IP=127.0.0.1
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0
export LIBGL_ALWAYS_INDIRECT=0

# 설정 완료 후 Gazebo 실행
gazebo
```

![Untitled11.png](/kr/ros2_foxy/images2/Untitled11.png?height=400px)

- 마지막으로, 앞선 설정 커멘드 라인을 매번 수고스럽게 입력하지 않도록 bashrc 설정 파일을 수정합니다.

```bash
echo 'export GAZEBO_IP=127.0.0.1' >> ~/.bashrc
echo "export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0 " >> ~/.bashrc
echo 'export LIBGL_ALWAYS_INDIRECT=0' >> ~/.bashrc
```

---

## 오류 FAQ

- Ubuntu 실행 시 0x80370114 에러

```jsx
Installing, this may take a few minutes...
WslRegisterDistribution failed with error: 0x80370114
Error: 0x80370114 ??? ??? ???? ?? ?? ??? ??? ??? ? ????.

Press any key to continue...
```

⇒ 관리자 권한 Powershell에서 아래의 커멘드 라인을 입력해주세요

```jsx
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

## **Windows 10 & WSL 2 Setup**

해당 메뉴얼은 Windows 10을 사용하는 PC를 위한 ROS 2 설치 튜토리얼입니다.

- 설치 전, “Windows Key ⇒ 업데이트”를 통해 최신 업데이트를 반영합니다. (상황에 따라 재부팅이 필요할 수 있습니다.)

![Untitled12.png](/kr/ros2_foxy/images2/Untitled12.png?height=300px)

- powershell을 관리자 권한으로 실행한 뒤, 설치 되어있는 WSL 2를 활성화시킵니다. 이는 “Windows Key + X” ⇒ 터미널(관리자)를 통해 실행 가능합니다.

![Untitled13.png](/kr/ros2_foxy/images2/Untitled13.png?height=300px)

- 관리자 권한 Powershell에서 아래와 같은 커멘드 라인을 입력합니다.

```jsx
# 1. WSL Enable
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 2. VirtualMachinePlatform Enable
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

> 아래와 같이 모든 작업이 성공적으로 완료되어야 합니다!

```jsx
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

- “Windows Key” ⇒ MS Store로 이동하여 windows terminal을 설치합니다.

![Untitled14.png](/kr/ros2_foxy/images2/Untitled14.png?height=400px)

windows terminal 화면 분할 예시

- alt + shift + - : 가로 분할
- alt + shift + + : 세로 분할
- ctrl + shift + w : 창 닫기
- alt + 화살표 : 창 간 이동

[다음 링크](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)를 통해 WSL 업데이트 프로그램을 다운받고 WSL 2 Linux 커널 업데이트를 설치/진행 후 재부팅합니다.

![Untitled15.png](/kr/ros2_foxy/images2/Untitled15.png?height=400px)

- 커널 업데이트를 마친 다음, powershell에 아래 커맨드 라인을 입력하여 WSL 2를 기본 사용하도록 설정합니다.

```jsx
> wsl --set-default-version 2

WSL 2와의 주요 차이점에 대한 자세한 내용은 https://aka.ms/wsl2를 참조하세요
작업을 완료했습니다.
```

- 지금까지 진행한 작업들이 제대로 설정 되어있는지 확인해봅시다.

```jsx
> wsl -l -v
  NAME            STATE            VERSION
* Ubuntu-20.04    Running         2
```

- MS Store 진입 후, Ubuntu 20.04 버전을 설치합니다.

![Untitled16.png](/kr/ros2_foxy/images2/Untitled16.png?height=400px)

- 설치 이후, **열기** 버튼을 눌러 초기 실행을 하고 username, password를 설정합니다.

![Untitled17.png](/kr/ros2_foxy/images2/Untitled17.png?height=200px)

- 설치 이후에는 windows terminal에서 **Ubuntu Terminal**을 선택하여 실행 및 진입이 가능합니다. 설정 탭을 통해 Ubuntu 20.04를 기본 터미널로 설정 후 저장합시다.

![Untitled18.png](/kr/ros2_foxy/images2/Untitled18.png?height=400px)

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

![Untitled19.png](/kr/ros2_foxy/images2/Untitled19.png?height=400px)

### GUI 인터페이스 설정

Gazebo, Rviz2, Rqt와 같은 시각화 프로그램을 사용하기 위해 Windows용 X Server인 VcXsrv를 사용하고자 합니다.

1. 링크를 통해 VcXsrv를 설치합니다. ⇒ [https://sourceforge.net/projects/vcxsrv/](https://sourceforge.net/projects/vcxsrv/)
2. 설치 이후, “Windows Key” ⇒ XLaunch를 검색하여 실행하고, 사진과 같은 설정을 진행합니다.

![Untitled9.png](/kr/ros2_foxy/images2/Untitled9.png?height=200px)

| 첫번째 페이지 | 하단 Display Number를 -1에서 0으로 변경               |
| ------------- | ----------------------------------------------------- |
| 두번째 페이지 | 통과                                                  |
| 세번째 페이지 | Native opengl 체크해제 후 Disable access control 체크 |
| 네번째 페이지 | 통과                                                  |

{{% notice tip %}}
앞으로 재부팅 시마다 이 설정을 반복해줘야 하며, 종종 Gazebo의 메모리 누수로 인해 화면이 종료되지 않는 경우가 있는데, 이런 경우 XLaunch를 강제 종료해주면 됩니다.
{{% /notice %}}

- 설정이 잘 되었는지 Gazebo를 실행해봅시다. 터미널 상의 아래의 4줄 커멘드 라인을 입력합니다.

```bash
export GAZEBO_IP=127.0.0.1
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0
export LIBGL_ALWAYS_INDIRECT=0

# 설정 완료 후 Gazebo 실행
gazebo
```

![Untitled11.png](/kr/ros2_foxy/images2/Untitled11.png?height=400px)

- 마지막으로, 앞선 설정 커멘드 라인을 매번 수고스럽게 입력하지 않도록 bashrc 설정 파일을 수정합니다.

```bash
echo 'export GAZEBO_IP=127.0.0.1' >> ~/.bashrc
echo "export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0 " >> ~/.bashrc
echo 'export LIBGL_ALWAYS_INDIRECT=0' >> ~/.bashrc
```

---

## 오류 FAQ

- `wsl --set-default-version 2` 에서 작업 완료 메세지가 등장하지 않는 경우

⇒ Linux용 Windows 하위 시스템 설정 여부를 확인해보신 뒤, 제일 처음 명령어부터 제가 보여드린 예시와 동일한 결과를 얻었는지 확인해보세요

![Untitled20.png](/kr/ros2_foxy/images2/Untitled20.png?height=400px)

- Ubuntu 설치 후 **‘The WSL Optional Component is not Enabled. Please Enable it and Try again’** 오류가 발생하는 경우

⇒ 관리자 권한 Powershell에서 아래의 커멘드 라인을 입력합니다.

```jsx
> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

- Ubuntu 설치 후 **0x80370102** 에러가 발생한 경우
  부팅 BIOS에서 가상화 기능을 활성화해줍니다.

- Ubuntu 설치 후 **0xc03a001a** 에러가 발생한 경우
  [다음 블로그 포스팅](https://goaloflife.tistory.com/193)을 참고합니다.

- Gazebo 실행 시 symbol lookup error가 발생하는 경우
  `sudo apt upgrade -y libignition-math2` 을 통해 누락된 패키지를 설치합니다.

```bash
$ gazebo
gazebo: symbol lookup error: /usr/lib/x86_64-linux-gnu/libgazebo_common.so.9: undefined symbol: _ZN8ignition10fuel_tools12ClientConfig12SetUserAgentERKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
```

- Gazebo 자체가 실행되지 않는 경우
  PC 재시작 후 백신 프로그램을 종료시킵니다.
  제어판 ⇒ 시스템 및 보안 ⇒ Windows 방화벽 ⇒ 고급 설정 => 인바운드 규칙 ⇒ VcXsrv 사용 허용을 적용합니다.

![Untitled21.png](/kr/ros2_foxy/images2/Untitled21.png?height=200px)

- Gazebo는 실행되지만, Grid (실선)이 등장하지 않는 경우
  안타깝지만 그래픽 드라이버 문제일 가능성이 큽니다. Window ⇒ 업데이트 확인 진입 후 가장 최신 버전으로 모두 업그레이드를 실행해줍니다

## 기타 개발 환경

강의 중 사용할 코드 환경은 다음과 같습니다.

- VSCode + Github Copilot

강의 중 실행한 한 줄 설치에는 제가 기본적으로 설정한 여러 단축 커멘드들이 있습니다. 몇가지만 실습하고 넘어가보겠습니다.

- eb : edit bashrc의 약자로 ~/.bashrc 파일을 gedit을 통해 수정할 수 있습니다.
- sb : source bashrc의 약자로 수정된 ~/.bashrc 파일을 반영시킵니다.
- killg : 실행중인 Gazebo관련 모든 프로그램을 종료시킵니다.
