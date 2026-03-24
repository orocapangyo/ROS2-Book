# 6장 ROS 1과 ROS 2의 차이점

## 6.1 ROS 2 개발 배경

ROS 1은 Willow Garage의 PR2 개발 환경을 기반으로 설계되어 다음과 같은 제약이 있었다.

| ROS 1 가정 | ROS 2 요구 |
|---|---|
| 단일 로봇 | 복수 로봇 |
| 워크스테이션급 PC | 임베디드 시스템 포함 |
| Linux 전용 | Linux, Windows, macOS |
| 실시간 제어 미지원 | 실시간 제어 지원 |
| 안정된 네트워크 필요 | 불안정한 네트워크에서도 동작 |
| 아카데믹 용도 | 상업용 제품 지원 |

ROS 1과의 호환성 유지가 어렵고 대규모 API 변경이 필요해 **ROS 2를 별도로 개발**하게 되었다.
ROS 1 사용자는 그대로 ROS 1을 사용하거나 `ros1_bridge`를 통해 두 버전을 함께 사용할 수 있다.

---

## 6.2 ROS 1 vs ROS 2 비교표 (20개 항목)

| 항목 | ROS 1 | ROS 2 |
|---|---|---|
| **플랫폼** | Linux, macOS | Linux, macOS, Windows |
| **실시간** | OROCOS 등 외부 프레임워크 사용 | RTOS + 잘 짜인 코드 조합으로 실시간 노드 지원 |
| **보안** | SROS | SROS 2, DDS-Security, Robotic Systems Threat Model |
| **통신** | XMLRPC + TCPROS | DDS (RTPS) |
| **미들웨어 인터페이스** | - | rmw (추상화 인터페이스) |
| **노드 관리 (탐색)** | ROS Master 필수 | roscore 없음, DDS Dynamic Discovery 사용 |
| **프로그래밍 언어** | C++03, Python 2.7 | C++14 (C++17), Python 3.5+ |
| **클라이언트 라이브러리** | roscpp, rospy, rosjava, rosnodejs 등 | rclcpp, rclpy, rcljava, rcljs 등 (RCL) |
| **빌드 시스템** | rosbuild → catkin (CMake) | ament (CMake), Python setuptools 완전 지원 |
| **빌드 도구** | catkin_make, catkin_tools | colcon |
| **빌드 옵션** | - | 복수 워크스페이스, 격리 빌드, devel 공간 없음 |
| **버전 관리 도구** | rosws → wstool, rosinstall (*.rosinstall) | vcstool (*.repos) |
| **라이프사이클** | - | 노드 라이프사이클 내장 |
| **복수 노드** | 프로세스당 노드 1개 | 프로세스당 복수 노드 실행 가능 |
| **스레딩 모델** | 단일/다중 스레드 선택 | 사용자 정의 Executor 지원 |
| **메시지 (토픽/서비스/액션)** | *.msg, *.srv, *.action | *.msg, *.srv, *.action, *.idl 추가 |
| **CLI** | rosrun, roslaunch, ros topic ... | ros2 run, ros2 launch, ros2 topic ... |
| **roslaunch** | XML 형식 전용 | Python, XML, YAML 지원 |
| **그래프 API** | 시작 시에만 리매핑 | 런타임 리매핑 지원 |
| **임베디드 시스템** | rosserial, mROS | microROS, XEL Network, ros2arduino, Renesas, DDS-XRCE (Micro-XRCE-DDS), AWS ARCLM |

---

## 6.3 핵심 변경사항 요약

### 통신: DDS 채택
ROS 2의 가장 큰 변화. DDS는 노드 자동 검색(Dynamic Discovery), QoS 정책, 보안(DDS-Security)을 제공한다.
지원 DDS 벤더: Fast DDS (Eprosima), Cyclone DDS (Eclipse), Connext DDS (RTI), Gurum DDS (한국), OpenSplice (ADLINK)

### roscore 제거
ROS Master의 단일 장애 지점 문제 해결. DDS Participant 개념으로 노드 간 직접 검색/연결.

### 빌드 시스템 개선
- `ament`: Python 패키지 완전 독립 지원
- `colcon`: 복수 워크스페이스, 격리 빌드, `--symlink-install` 옵션

### Lifecycle 내장
노드 상태 모니터링 및 상태 천이 제어를 클라이언트 라이브러리 수준에서 지원.

### 임베디드 지원 강화
micro-ROS, DDS-XRCE 등을 통해 MCU에서 직접 ROS 노드 실행 가능.

---

# 정리

ROS 2는 기본 개념(노드, 토픽, 서비스, 액션)은 ROS 1과 동일하지만 내부 구현은 완전히 새로 작성된 플랫폼이다.
`ros1_bridge`를 통해 ROS 1를 유지하면서 점진적으로 전환이 가능하다.
