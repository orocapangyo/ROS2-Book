# 6장 ROS1 과 ROS2의 차이점으로 알아보는 ROS2 특징

---

## 6.1. ROS 2 개발의 필요성
- ROS 2는 멀티로봇 확장과 임베디드 연동을 고려한 장기 확장성 측면에서 ROS 1의 한계를 보완하기 위한 필수 진화라는 결론이다.

### 개발 제한사한
- 단일로봇
- 워크스테이션급 컴퓨터
- 리눅스 환경
- 실시간 제어 지원하지 않음
- 안정된 네트워크 환경이 요구됨
- 주로 대학이나 연구소와 같은 아키데믹 연구 용도

### 개발 요구사항
- 두 대 이상의 로봇
- 임베디드 시스템에서의 ROS 사용
- 실시간 제어
- 불안정한 네트워크 환경에서도 동작할 수 있는 유연함
- 멀티 플랫폼(리눅스, 윈도우, macOS)
- 최신 기술 지원(Zeroconf, Protocol Buffers, ZeroMQ, WebSockets, DDS 등)
---
## 6.2. ROS 1과 ROS 2의 차이점

| Features | ROS 1 | ROS 2 |
|---|---|---|
| Platforms | Linux, macOS | Linux, macOS, Windows |
| Real-time | External frameworks like OROCOS | Real-time nodes when using a proper RTOS with carefully written user code |
| Security | SROS | SROS2, DDS-Security, Robotic Systems Threat Model |
| Communication | XMLRPC + TCPROS | DDS (RTPS) |
| Middleware interface | - | RMW |
| Node manager (discovery) | ROS Master | No, use DDS’s dynamic discovery |
| Languages | C++03, Python 2.7 | C++14 (C++17), Python 3.5+ |
| Client library | roscpp, rospy, rosjava, rosnodejs, and more | rclcpp, rclpy, rcljava, rcljs, and more |
| Build system | rosbuild → catkin (CMake) | Ament (CMake), Python setuptools (Full support) |
| Build tools | catkin_make, catkin_tools | Colcon |
| Build options | - | Multiple workspace, No non-isolated build, No devel space |
| Version control system | rosws → wstool, rosinstall<br>(`*.rosinstall`) | vcstool<br>(`*.repos`) |
| Life cycle | - | Node life cycle |
| Multiple nodes | One node in a process | Multiple nodes in a process |
| Threading model | Single-threaded execution or multi-threaded execution | Custom executors |
| Messages (topic, service, action) | `*.msg`, `*.srv`, `*.action` | `*.msg`, `*.srv`, `*.action`, `*.idl` |
| Command Line Interface | rosrun, roslaunch, rostopic … | ros2 run, ros2 launch, ros2 topic … |
| roslaunch | XML | Python, XML, YAML |
| Graph API | Remapping at startup time only | Remapping at runtime |
| Embedded Systems | rosserial, mROS | microROS, XEL Network, ros2arduino, Renesas DDS-XRCE (Micro-XRCE-DDS), AWS ARCLM |

---

## 6.3. ROS 2의 특징
- 20가지 이유 정리 

### 1) Platforms
- ROS 1은 Linux, macOS 중심 지원이라는 특징이다.
- ROS 2는 Linux, macOS, Windows까지 지원 범위를 확대하는 특징이다.
- 산업 환경(개발·운영 OS 다양화)에서 적용 가능성이 커지는 특징이다.

### 2) Real-time
- ROS 1은 실시간성을 위해 OROCOS 등 외부 프레임워크 의존이 큰 구조이다.
- ROS 2는 RTOS와 신중한 사용자 코드 설계 시 실시간 노드 구성이 가능한 구조이다.
- 제어 주기 안정성(지터 감소)을 설계 목표로 올려놓은 특징이다.

### 3) Security
- ROS 1은 SROS가 존재하지만 기본 통신에서 보안 내장이 약한 구조이다.
- ROS 2는 SROS2, DDS-Security, Threat Model 기반으로 인증·암호화·접근제어 적용이 가능한 구조이다.
- 원격 운영/OTA/클라우드 연동에서 “기본 보안 전제”를 갖는 특징이다.

### 4) Communication
- ROS 1은 XMLRPC + TCPROS 중심의 통신 구조이다.
- ROS 2는 DDS(RTPS) 기반 통신 구조이다.
- 분산 시스템 확장성과 전송 정책(QoS) 제어에 유리한 특징이다.

### 5) Middleware interface
- ROS 1은 통신 계층 추상화가 제한적인 구조이다.
- ROS 2는 RMW(Robot Middleware Interface)로 미들웨어 계층을 추상화한 구조이다.
- DDS 구현체 교체/선택이 가능해지는 유연성이 특징이다.

### 6) Node manager (Discovery)
- ROS 1은 ROS Master가 디스커버리(노드 발견)의 핵심 구성요소인 구조이다.
- ROS 2는 DDS의 동적 디스커버리를 사용해 Master 없이 동작 가능한 구조이다.
- 단일 장애점(SPOF) 감소 및 멀티로봇/멀티네트워크에 유리한 특징이다.

### 7) Languages
- ROS 1은 C++03, Python 2.7 중심 생태계가 강한 구조이다.
- ROS 2는 C++14(C++17), Python 3.5+ 중심으로 현대 언어/런타임을 채택한 구조이다.
- 장기 유지보수와 라이브러리 호환성 측면에서 유리한 특징이다.

### 8) Build system
- ROS 1은 rosbuild → catkin(CMake) 흐름의 빌드 시스템이다.
- ROS 2는 ament(CMake) + Python setuptools를 공식 지원하는 빌드 시스템이다.
- CMake/파이썬 패키징의 표준 흐름을 강화한 특징이다.

### 9) Build tools
- ROS 1은 catkin_make, catkin_tools 중심 도구 체인이다.
- ROS 2는 colcon 중심 도구 체인이다.
- 대규모 워크스페이스 빌드/테스트/확장에 유리한 특징이다.

### 10) Build options
- ROS 1은 워크스페이스/빌드 옵션이 비교적 단순한 편인 구조이다.
- ROS 2는 다중 워크스페이스, 격리 빌드 지향, devel space 미사용 등 운영형 빌드 구조를 채택한 특징이다.
- 배포/CI 파이프라인에 맞춘 구조화가 특징이다.

### 11) Version control system
- ROS 1은 rosws → wstool, rosinstall(`*.rosinstall`) 기반 관리가 흔한 구조이다.
- ROS 2는 vcstool(`*.repos`) 기반으로 저장소 집합을 관리하는 구조이다.
- 멀티 저장소 프로젝트의 재현성/동기화가 향상되는 특징이다.

### 12) Client library
- ROS 1은 roscpp, rospy, rosjava, rosnodejs 등 클라이언트 라이브러리 구조이다.
- ROS 2는 rclcpp, rclpy, rcljava, rcljs 등 rcl 기반으로 재정렬된 구조이다.
- 공통 코어(rcl) 위에서 언어별 일관성이 높아지는 특징이다.

### 13) Lifecycle
- ROS 1은 노드 상태 전이를 표준화하기 어려운 구조이다.
- ROS 2는 Node Lifecycle로 구성/활성/비활성/종료 상태를 명시적으로 관리하는 구조이다.
- 현장 운영에서 부분 복구·재시작 설계가 쉬워지는 특징이다.

### 14) Multiple nodes
- ROS 1은 일반적으로 프로세스당 노드 1개 운용이 흔한 구조이다.
- ROS 2는 프로세스 내 다중 노드 구성이 가능한 구조이다.
- 성능(지연/오버헤드)과 장애 격리를 설계로 선택할 수 있는 특징이다.

### 15) Threading model
- ROS 1은 싱글/멀티 스레드 실행 모델이 존재하나 구조적 확장 여지가 제한되는 편이다.
- ROS 2는 커스텀 Executor 등 실행 모델 설계 자유도가 높은 구조이다.
- 고주기 파이프라인의 지연 변동(지터) 제어에 유리한 특징이다.

### 16) Messages
- ROS 1은 `*.msg`, `*.srv`, `*.action` 중심 인터페이스 정의이다.
- ROS 2는 `*.msg`, `*.srv`, `*.action`에 더해 `*.idl` 기반 정의가 가능한 구조이다.
- DDS 기반 상호운용성과 인터페이스 표준화에 유리한 특징이다.

### 17) Command Line Interface
- ROS 1은 `rosrun`, `roslaunch`, `rostopic` 등 분산된 CLI 도구 체인이다.
- ROS 2는 `ros2 run`, `ros2 launch`, `ros2 topic` 등 통합된 CLI 구조이다.
- 도구 사용성, 자동화, 학습 비용 측면에서 개선된 특징이다.

### 18) Launch
- ROS 1은 launch 파일이 XML 중심인 구조이다.
- ROS 2는 Python 기반 launch를 중심으로 XML/YAML도 지원하는 구조이다.
- 조건 분기·재사용·코드화가 쉬워 운영 시나리오 표현력이 커지는 특징이다.

### 19) Graph API
- ROS 1은 주로 시작 시점(remapping at startup) 중심 제어가 일반적인 구조이다.
- ROS 2는 런타임(remapping at runtime)까지 고려되는 방향의 구조이다.
- 운영 중 재구성/동적 관리에 유리한 특징이다.

### 20) Embedded Systems
- ROS 1은 rosserial, mROS 등 임베디드 연계가 가능하나 프로젝트별 설계 의존이 큰 편이다.
- ROS 2는 micro-ROS, DDS-XRCE(Micro-XRCE-DDS) 등 임베디드 연계 생태계가 확장된 구조이다.
- Jetson/PC 상위제어와 MCU 저수준제어를 표준 인터페이스로 계층화하기 쉬운 특징이다.
