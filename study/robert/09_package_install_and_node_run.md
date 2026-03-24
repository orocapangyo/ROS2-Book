# 9장 패키지 설치와 노드 실행

> **참고**: [ROS 2 Jazzy 공식 튜토리얼](https://docs.ros.org/en/jazzy/Tutorials.html) 기반으로 작성

## 9.1 개요

ROS 2에서 로봇 소프트웨어를 사용하려면 먼저 **패키지를 설치**하고, 그 안에 포함된 **노드를 실행**해야 한다.  
이 장에서는 패키지를 설치하는 방법과 노드를 실행하는 다양한 방법, 그리고 turtlesim을 활용한 실습을 다룬다.

---

## 9.2 ROS 2 환경 설정

노드를 실행하기 전에 **매 터미널마다** ROS 2 환경을 소싱해야 한다.

```bash
# ROS 2 Jazzy 환경 소싱
source /opt/ros/jazzy/setup.bash

# .bashrc에 추가하면 매번 소싱할 필요 없음
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
```

### 주요 환경 변수

| 환경 변수 | 설명 | 예시 |
|-----------|------|------|
| `ROS_DISTRO` | ROS 2 배포판 이름 | `jazzy` |
| `ROS_DOMAIN_ID` | DDS 도메인 분리 (0~232) | `export ROS_DOMAIN_ID=30` |
| `ROS_LOCALHOST_ONLY` | 로컬 통신만 허용 | `export ROS_LOCALHOST_ONLY=1` |

> 💡 **Tip**: 같은 네트워크에서 여러 사람이 ROS 2를 사용할 때, `ROS_DOMAIN_ID`를 서로 다르게 설정하면 메시지 충돌을 방지할 수 있다.

---

## 9.3 패키지 설치 방법

ROS 2 패키지를 설치하는 방법은 크게 두 가지가 있다.

### 9.3.1 바이너리 설치 (apt)

미리 빌드된 패키지를 시스템 패키지 관리자를 통해 설치하는 방법이다.

```bash
# 패키지 목록 업데이트
sudo apt update

# ROS 2 패키지 설치 (예: turtlesim)
sudo apt install ros-jazzy-turtlesim

# 설치 확인 — 패키지의 실행 파일 목록 출력
ros2 pkg executables turtlesim
```

**실행 결과:**
```
turtlesim draw_square
turtlesim mimic
turtlesim turtle_teleop_key
turtlesim turtlesim_node
```

**장점**
- 설치가 빠르고 간편하다
- 의존성이 자동으로 해결된다
- 안정적인 릴리즈 버전을 사용할 수 있다

**단점**
- 소스 코드를 수정할 수 없다
- 최신 개발 버전을 사용하기 어렵다

---

### 9.3.2 소스 빌드 설치

소스 코드를 직접 다운로드하여 워크스페이스에서 빌드하는 방법이다.

```bash
# 워크스페이스 생성
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

# 소스 코드 클론
git clone https://github.com/ros/ros_tutorials.git -b jazzy

# 의존성 설치
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y

# 빌드
colcon build

# 환경 설정 적용 (오버레이)
source install/setup.bash
```

**장점**
- 소스 코드를 직접 수정할 수 있다
- 최신 개발 버전을 사용할 수 있다
- 커스텀 패키지를 추가할 수 있다

**단점**
- 빌드 시간이 소요된다
- 의존성 문제가 발생할 수 있다

### 언더레이(Underlay)와 오버레이(Overlay)

```
[언더레이: /opt/ros/jazzy]  ← ROS 2 기본 설치 (apt로 설치된 패키지)
         ↓ source
[오버레이: ~/ros2_ws]       ← 사용자 워크스페이스 (소스 빌드한 패키지)
```

- **언더레이**: 기본 ROS 2 설치 경로. 시스템 전체에서 공유
- **오버레이**: 사용자가 작업하는 워크스페이스. 언더레이 위에 덮어씀

> ⚠️ 오버레이에 같은 이름의 패키지가 있으면, 언더레이의 패키지보다 우선된다.

---

## 9.4 rosdep — 의존성 관리 도구

`rosdep`은 패키지의 의존성을 자동으로 확인하고 설치해주는 도구이다.

```bash
# rosdep 초기화 (최초 1회)
sudo rosdep init
rosdep update

# 워크스페이스 의존성 일괄 설치
rosdep install --from-paths src --ignore-src -r -y
```

### rosdep 동작 과정

1. `package.xml` 파일에서 `<depend>`, `<build_depend>`, `<exec_depend>` 항목을 읽는다
2. 시스템에 설치 가능한 패키지로 매핑한다 (OS별 매핑 규칙 사용)
3. `apt` 등 시스템 패키지 관리자를 통해 설치한다

---

## 9.5 노드 실행 — ros2 run

`ros2 run` 명령어는 특정 패키지의 실행 파일(노드)을 직접 실행하는 명령이다.

### 기본 문법

```bash
ros2 run <패키지명> <실행파일명>
```

### 실행 예시: turtlesim

```bash
# turtlesim 노드 실행
ros2 run turtlesim turtlesim_node
```

**실행하면 아래와 같은 시뮬레이터 창이 나타난다:**

![turtlesim 실행 화면](images/turtlesim_window.png)

**터미널 출력:**
```
[INFO] [turtlesim]: Starting turtlesim with node name /turtlesim
[INFO] [turtlesim]: Spawning turtle [turtle1] at x=[5.544445], y=[5.544445], theta=[0.000000]
```

```bash
# 키보드 조종 노드 실행 (다른 터미널)
ros2 run turtlesim turtle_teleop_key
```

> 💡 **Tip**: `turtle_teleop_key` 노드가 활성화된 터미널에서 **방향키**를 누르면 거북이가 움직인다. 한 번 누르면 짧은 거리만 이동하고 멈추는데, 이는 실제 로봇에서 연결이 끊어졌을 때 계속 이동하는 것을 방지하기 위한 설계이다.

### 파라미터와 함께 실행

```bash
# 파라미터를 지정하여 노드 실행
ros2 run turtlesim turtlesim_node --ros-args -p background_r:=255

# 여러 파라미터 지정
ros2 run my_package my_node --ros-args -p param1:=value1 -p param2:=value2
```

---

## 9.6 노드 이름 변경 — 리매핑 (Remapping)

리매핑은 노드 이름, 토픽 이름, 서비스 이름 등을 **실행 시점에** 변경하는 기능이다.  
코드를 수정하지 않고도 같은 노드를 다른 이름으로 실행할 수 있다.

### 노드 이름 리매핑

```bash
# 노드 이름을 my_turtle로 변경
ros2 run turtlesim turtlesim_node --ros-args -r __node:=my_turtle
```

`ros2 node list`로 확인하면 `/my_turtle`이라는 이름으로 실행된 것을 볼 수 있다.

### 토픽 이름 리매핑

```bash
# /cmd_vel 토픽을 /my_cmd_vel로 변경
ros2 run turtlesim turtlesim_node --ros-args -r /turtle1/cmd_vel:=/my_cmd_vel
```

### 리매핑 활용 시나리오

| 상황 | 리매핑 대상 | 예시 |
|------|------------|------|
| 같은 노드를 여러 개 실행 | 노드 이름 | `__node:=camera_left`, `__node:=camera_right` |
| 다른 토픽에 연결 | 토픽 이름 | `/input:=/sensor_data` |
| 서비스 이름 변경 | 서비스 이름 | `/get_state:=/robot1/get_state` |

---

## 9.7 네임스페이스 (Namespace)

네임스페이스는 노드, 토픽, 서비스 등의 이름 앞에 **접두사(prefix)**를 붙여 그룹화하는 기능이다.  
멀티 로봇 환경에서 각 로봇의 리소스를 구분할 때 유용하다.

### 네임스페이스 적용

```bash
# 네임스페이스를 지정하여 노드 실행
ros2 run turtlesim turtlesim_node --ros-args -r __ns:=/robot1
```

### 네임스페이스 적용 전후 비교

| 구분 | 네임스페이스 없음 | 네임스페이스 `/robot1` |
|------|-------------------|----------------------|
| 노드 이름 | `/turtlesim` | `/robot1/turtlesim` |
| 토픽 이름 | `/turtle1/cmd_vel` | `/robot1/turtle1/cmd_vel` |
| 서비스 이름 | `/turtle1/set_pen` | `/robot1/turtle1/set_pen` |

### 멀티 로봇 시나리오

```bash
# 로봇1
ros2 run turtlesim turtlesim_node --ros-args -r __ns:=/robot1

# 로봇2 (다른 터미널)
ros2 run turtlesim turtlesim_node --ros-args -r __ns:=/robot2
```

이렇게 하면 두 개의 turtlesim 노드가 충돌 없이 독립적으로 동작한다.

---

## 9.8 런치 파일을 이용한 노드 실행 — ros2 launch

`ros2 launch` 명령어는 런치 파일을 통해 여러 노드를 한 번에 실행한다.

### 기본 문법

```bash
ros2 launch <패키지명> <런치파일명>
```

### 실행 예시

```bash
# 런치 파일로 여러 노드 동시 실행
ros2 launch turtlesim multisim.launch.py
```

### 런치 파일 예시 (Python)

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='turtlesim',
            executable='turtlesim_node',
            name='turtle1',
            namespace='robot1',
            parameters=[{'background_r': 200}],
        ),
        Node(
            package='turtlesim',
            executable='turtlesim_node',
            name='turtle2',
            namespace='robot2',
            parameters=[{'background_r': 100}],
        ),
    ])
```

### ros2 run vs ros2 launch 비교

| 항목 | `ros2 run` | `ros2 launch` |
|------|-----------|--------------|
| 실행 단위 | 노드 1개 | 노드 여러 개 |
| 설정 관리 | 명령줄 인자 | 런치 파일에 일괄 정의 |
| 재현성 | 매번 명령어 입력 필요 | 파일로 관리되어 버전 관리 가능 |
| 적합한 상황 | 테스트, 디버깅 | 실제 시스템 운용, 배포 |

---

## 9.9 실행 중인 노드 확인

노드를 실행한 후 현재 상태를 확인하는 명령어들을 알아본다.

### 주요 확인 명령어

```bash
# 실행 중인 노드 목록 확인
ros2 node list

# 특정 노드의 상세 정보 확인
ros2 node info /turtlesim

# 실행 중인 토픽 목록 확인
ros2 topic list

# 실행 중인 서비스 목록 확인
ros2 service list

# 실행 중인 액션 목록 확인
ros2 action list

# 설치된 패키지 목록 확인
ros2 pkg list

# 특정 패키지의 실행 파일 목록 확인
ros2 pkg executables turtlesim
```

### ros2 node info 출력 예시

```
/turtlesim
  Subscribers:
    /turtle1/cmd_vel: geometry_msgs/msg/Twist
  Publishers:
    /turtle1/color_sensor: turtlesim/msg/Color
    /turtle1/pose: turtlesim/msg/Pose
  Service Servers:
    /clear: std_srvs/srv/Empty
    /spawn: turtlesim/srv/Spawn
    /kill: turtlesim/srv/Kill
    /turtle1/set_pen: turtlesim/srv/SetPen
    /turtle1/teleport_absolute: turtlesim/srv/TeleportAbsolute
    /turtle1/teleport_relative: turtlesim/srv/TeleportRelative
  Action Servers:
    /turtle1/rotate_absolute: turtlesim/action/RotateAbsolute
```

위 출력을 통해 `/turtlesim` 노드가 사용하는 **토픽, 서비스, 액션**을 한눈에 파악할 수 있다.

---

## 9.10 rqt — GUI 기반 도구

`rqt`는 ROS 2의 다양한 GUI 플러그인을 제공하는 도구 모음이다.

```bash
# rqt 설치 (Jazzy)
sudo apt install 'ros-jazzy-rqt*'

# rqt 실행
rqt
```

**주요 플러그인:**

| 플러그인 | 메뉴 경로 | 기능 |
|----------|----------|------|
| rqt_graph | Plugins > Introspection > Node Graph | 노드-토픽 연결 관계 그래프 |
| rqt_console | Plugins > Logging > Console | 로그 메시지 실시간 확인 |
| rqt_service_caller | Plugins > Services > Service Caller | GUI에서 서비스 호출 |
| rqt_topic | Plugins > Topics > Topic Monitor | 토픽 데이터 실시간 모니터링 |

### rqt에서 서비스 호출하기

1. rqt 실행 → **Plugins > Services > Service Caller** 선택
2. Service 드롭다운에서 `/spawn` 선택
3. `x`, `y`, `theta`, `name` 값 입력
4. **Call** 버튼 클릭 → 새 거북이 생성

---

## 9.11 정리

패키지 설치와 노드 실행에 관한 핵심 내용을 요약하면 다음과 같다.

| 항목 | 설명 |
|------|------|
| 환경 소싱 | 매 터미널마다 `source /opt/ros/jazzy/setup.bash` 필요 |
| 바이너리 설치 | `apt`로 빠르게 설치, 소스 수정 불가 |
| 소스 빌드 | `colcon build`로 직접 빌드, 커스터마이징 가능 |
| `rosdep` | 의존성 자동 설치 도구 |
| `ros2 run` | 단일 노드 실행 |
| `ros2 launch` | 런치 파일을 통한 다중 노드 실행 |
| 리매핑 | 실행 시 이름 변경 (`--ros-args -r`) |
| 네임스페이스 | 리소스 그룹화 (`__ns:=`) |
| 노드 확인 | `ros2 node list`, `ros2 node info` |
| rqt | GUI 기반 도구 모음 (서비스 호출, 그래프 시각화 등) |

패키지 설치와 노드 실행은 ROS 2 개발의 가장 기본적인 작업이다.  
이 과정을 정확히 이해하고 있어야 이후의 토픽, 서비스, 액션 실습을 원활하게 진행할 수 있다.
