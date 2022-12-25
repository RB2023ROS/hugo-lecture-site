---
title: "Lecture2"
date: 2022-12-25T13:36:33+09:00
draft: false
---

> 지난 시간 개발환경 세팅을 잘 진행하였는지 확인을 해보면서 강의를 시작해보겠습니다.

터미널 프로그램을 실행한 뒤, gazebo를 실행해 봅시다.

```bash
gazebo
```

![img0](/kr/ros_basic_noetic/images2/lec2_0.png?height=400px)

위 사진과 같은 화면이 나오지 않았다면 설치가 제대로 되지 않은 것입니다.

다음으로, ROS 설치는 잘 되었는지도 확인해봅시다.

```bash
# Terminal 1
roscore
# Terminal 2
rosrun rospy_tutorials talker
```

![talker.gif](/kr/ros_basic_noetic/images2/talker.gif?height=400px)

모든 확인이 끝났다면, 예제 프로그램을 실행시켜보겠습니다.

---

## Husky Gazebo

- 예제 패키지 설치

```
sudo apt-get update
sudo apt-get install ros-noetic-husky-desktop
sudo apt-get install ros-noetic-husky-simulator
```

- 예제 프로그램 실행

```bash
# Terminal 1
roslaunch husky_gazebo husky_empty_world.launch
# Terminal 2
roslaunch husky_viz view_robot.launch
# Terminal 3
rosrun teleop_twist_keyboard teleop_twist_keyboard.py cmd_vel:=/husky_velocity_controller/cmd_vel
```

Terminal 1에서 발생하는 아래 오류는 무시해도 좋습니다.

![lec2_1.png](/kr/ros_basic_noetic/images2/lec2_1.png?height=300px)

- 모든 실행이 정상 동작하였다면, Terminal 3에서 키보드를 통해 Husky를 제어할 수 있습니다.

![husky_move.gif](/kr/ros_basic_noetic/images2/husky_move.gif?height=400px)

- 로봇이 움직임에 따라 두번째 터미널 결과였던 Rviz에 아래와 같은 변화가 생깁니다.

![lec2_2.png](/kr/ros_basic_noetic/images2/lec2_2.png?height=400px)

- 다음으로, 새로운 터미널에서 아래 커멘드 라인을 실행시켜 봅시다.

```bash
rosrun rqt_graph rqt_graph
```

![rosgraph.png](/kr/ros_basic_noetic/images2/rosgraph.png?height=150px)

위 그림은 방금 전 실행한 예제 내부적으로 어떠한 동작들이 이루어지고 있었는지를 보여주는 것으로, 강의를 마칠 때면 여러분들은 위 그림이 어떠한 의미를 갖는지 모두 이해하실 수 있을 것입니다.

다음으로, 터미널을 새로 실행시켜 `rosnode list`와 `rostopic list`를 실행시켜 봅시다.

- **rosnode list**

```bash
$ rosnode list
/base_controller_spawner
/ekf_localization
/gazebo
/gazebo_gui
/joy_teleop/joy_node
/joy_teleop/teleop_twist_joy
/robot_state_publisher
/rosout
/teleop_twist_keyboard
/twist_marker_server
/twist_mux
```

- **rostopic list**

```bash
$ rostopic list
/clock
/cmd_vel
/diagnostics
/e_stop
/gazebo/link_states
/gazebo/model_states
/gazebo/parameter_descriptions
/gazebo/parameter_updates
/gazebo/performance_metrics
/gazebo/set_link_state
/gazebo/set_model_state
...
```

> 앞으로의 강의들에서, 위 명령어들이 어떠한 의미를 갖는지 하나하나씩 함께 살펴보겠습니다.

---
