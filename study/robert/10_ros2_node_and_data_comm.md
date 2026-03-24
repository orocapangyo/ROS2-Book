# 10장 ROS 2 노드와 데이터 통신

> **참고**: [ROS 2 Jazzy 공식 튜토리얼 - Understanding nodes](https://docs.ros.org/en/jazzy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html) 기반으로 작성

## 10.1 개요

ROS 2에서 로봇 소프트웨어는 여러 개의 **노드(Node)**로 구성되며,  
이 노드들이 서로 **데이터를 주고받는 구조**가 ROS 2 시스템의 핵심이다.  
이 장에서는 노드의 구조와 노드 간 데이터 통신의 원리를 상세히 살펴본다.

---

## 10.2 ROS 2 그래프 (ROS 2 Graph)

ROS 2 그래프는 **동시에 데이터를 처리하는 ROS 2 요소들의 네트워크**를 의미한다.  
모든 실행 파일과 그 사이의 연결을 포함하며, 이를 시각화하면 노드 간 관계를 파악할 수 있다.

![노드 간 토픽과 서비스 통신](images/nodes_topic_and_service.gif)

위 다이어그램은 ROS 2에서 노드가 토픽과 서비스를 통해 통신하는 구조를 보여준다.

---

## 10.3 노드(Node)의 구조

### 10.3.1 노드란?

노드는 ROS 2에서 **하나의 독립적인 실행 단위**이다.  
각 노드는 단일 모듈 목적을 담당해야 한다. 예를 들어:

- 바퀴 모터 제어
- 라이다 센서 데이터 퍼블리시
- SLAM(지도 생성 및 위치 추정)
- 경로 계획

> 📌 **공식 문서**: "Each node in ROS should be responsible for a single, modular purpose, e.g. controlling the wheel motors or publishing the sensor data from a laser range-finder."

**ROS 2에서는 하나의 실행 파일(C++ 또는 Python 프로그램)에 하나 이상의 노드를 포함할 수 있다.**

```
[카메라 노드] → (이미지 토픽) → [영상처리 노드] → (인식 결과 토픽) → [제어 노드]
```

### 10.3.2 노드의 생명주기

노드가 실행되면 다음과 같은 과정을 거친다.

1. **초기화 (Initialization)**: `rclpy.init()` / `rclcpp::init()` — ROS 2 통신 인프라 초기화
2. **노드 생성**: 퍼블리셔, 서브스크라이버, 서비스, 타이머 등 등록
3. **스핀 (Spin)**: `rclpy.spin()` — 콜백 함수를 반복 실행하며 메시지 처리
4. **종료 (Shutdown)**: `rclpy.shutdown()` — 노드 정리 및 ROS 2 종료

### 10.3.3 Python 노드 기본 구조

```python
import rclpy
from rclpy.node import Node

class MyNode(Node):
    def __init__(self):
        super().__init__('my_node')  # 노드 이름 지정
        self.get_logger().info('노드가 시작되었습니다!')

def main(args=None):
    rclpy.init(args=args)           # ROS 2 초기화
    node = MyNode()                 # 노드 생성
    rclpy.spin(node)                # 콜백 대기 루프
    node.destroy_node()             # 노드 정리
    rclpy.shutdown()                # ROS 2 종료

if __name__ == '__main__':
    main()
```

### 10.3.4 C++ 노드 기본 구조

```cpp
#include "rclcpp/rclcpp.hpp"

class MyNode : public rclcpp::Node {
public:
    MyNode() : Node("my_node") {
        RCLCPP_INFO(this->get_logger(), "노드가 시작되었습니다!");
    }
};

int main(int argc, char *argv[]) {
    rclcpp::init(argc, argv);           // ROS 2 초기화
    auto node = std::make_shared<MyNode>();  // 노드 생성
    rclcpp::spin(node);                 // 콜백 대기 루프
    rclcpp::shutdown();                 // ROS 2 종료
    return 0;
}
```

---

## 10.4 데이터 통신의 세 가지 방식

ROS 2는 노드 간 통신을 위해 세 가지 방식을 제공한다.

| 통신 방식 | 패턴 | 특징 | 사용 예 |
|-----------|------|------|---------|
| **토픽 (Topic)** | Pub/Sub | 비동기, 단방향, N:N | 센서 데이터, 제어 명령 |
| **서비스 (Service)** | Request/Response | 동기, 양방향, 1:N | 설정 변경, 상태 조회 |
| **액션 (Action)** | Goal/Feedback/Result | 비동기, 양방향, 장기 작업 | 내비게이션, 경로 추종 |

### 통신 방식 선택 기준

```
데이터가 지속적으로 흐르는가?
├── 예 → Topic (비동기 스트리밍)
└── 아니오 → 즉각적인 응답이 필요한가?
    ├── 예 → Service (동기 요청-응답)
    └── 아니오 → 오래 걸리는 작업인가?
        ├── 예 → Action (비동기 + 피드백)
        └── 아니오 → Service
```

> 📌 **공식 문서**: "Topics are one of the main ways in which data is moved between nodes and therefore between different parts of the system." / "Services are based on a call-and-response model versus the publisher-subscriber model of topics."

---

## 10.5 메시지 (Message)

메시지는 노드 간 주고받는 **데이터의 구조**를 정의한다.

### 10.5.1 메시지 타입 확인

```bash
# 사용 가능한 메시지 타입 목록
ros2 interface list

# 특정 메시지 타입의 구조 확인
ros2 interface show geometry_msgs/msg/Twist
```

**출력:**
```
Vector3 linear
  float64 x
  float64 y
  float64 z
Vector3 angular
  float64 x
  float64 y
  float64 z
```

### 10.5.2 주요 표준 메시지 타입

| 메시지 타입 | 설명 | 사용 예 |
|------------|------|---------|
| `std_msgs/msg/String` | 문자열 | 텍스트 로그, 상태 메시지 |
| `std_msgs/msg/Int32` | 정수 | 카운터, ID |
| `geometry_msgs/msg/Twist` | 선속도 + 각속도 | 로봇 이동 명령 (`/cmd_vel`) |
| `sensor_msgs/msg/LaserScan` | 라이다 스캔 | 장애물 감지 |
| `sensor_msgs/msg/Image` | 이미지 데이터 | 카메라 영상 |
| `nav_msgs/msg/Odometry` | 오도메트리 | 로봇 위치 추정 |
| `turtlesim/msg/Pose` | 2D 위치(x, y, theta, 속도) | turtlesim 거북이 위치 |

### 10.5.3 커스텀 메시지 정의

프로젝트에 맞는 메시지를 직접 정의할 수 있다.

**메시지 파일 (msg/RobotStatus.msg)**
```
string robot_name
float64 battery_level
bool is_moving
int32 error_code
```

---

## 10.6 콜백 (Callback) 함수

ROS 2의 통신은 **콜백(Callback) 기반**으로 동작한다.  
메시지가 도착하면 미리 등록된 콜백 함수가 자동으로 호출된다.

### 콜백 동작 원리

```
[퍼블리셔] --메시지 발행--> [토픽] --메시지 도착--> [서브스크라이버의 콜백 함수 실행]
```

### Python 콜백 예시

```python
from rclpy.node import Node
from std_msgs.msg import String

class Listener(Node):
    def __init__(self):
        super().__init__('listener')
        self.subscription = self.create_subscription(
            String,
            '/chatter',
            self.listener_callback,  # 콜백 함수 등록
            10                       # QoS depth
        )

    def listener_callback(self, msg):
        self.get_logger().info(f'수신: {msg.data}')
```

---

## 10.7 Executor — 콜백 실행 관리

Executor는 **콜백 함수의 실행을 관리하는 메커니즘**이다.  
`rclpy.spin()`을 호출하면 내부적으로 Executor가 콜백을 처리한다.

### Executor의 종류

| Executor | 설명 | 적합한 상황 |
|----------|------|------------|
| `SingleThreadedExecutor` | 콜백을 하나씩 순차 실행 | 단순한 노드, 기본값 |
| `MultiThreadedExecutor` | 여러 콜백을 동시에 병렬 실행 | 처리량이 많은 노드 |

### MultiThreadedExecutor 사용 예시

```python
import rclpy
from rclpy.executors import MultiThreadedExecutor

rclpy.init()

node1 = CameraNode()
node2 = LidarNode()

executor = MultiThreadedExecutor()
executor.add_node(node1)
executor.add_node(node2)

executor.spin()  # 두 노드의 콜백을 병렬 처리

rclpy.shutdown()
```

---

## 10.8 Timer — 주기적 실행

타이머를 사용하면 **일정 주기마다** 콜백 함수를 실행할 수 있다.  
센서 데이터 발행이나 주기적 상태 확인에 주로 사용한다.

```python
class TimerNode(Node):
    def __init__(self):
        super().__init__('timer_node')
        self.timer = self.create_timer(
            0.5,                    # 0.5초(2Hz) 주기
            self.timer_callback     # 콜백 함수
        )
        self.count = 0

    def timer_callback(self):
        self.count += 1
        self.get_logger().info(f'타이머 콜백: {self.count}회 실행')
```

---

## 10.9 ros2doctor — 문제 진단 도구

`ros2doctor`는 ROS 2 설치 및 실행 환경의 문제를 자동으로 진단해주는 도구이다.

```bash
# 시스템 전체 진단
ros2 doctor

# 상세 보고서 출력
ros2 doctor --report

# 토픽 통신 상태 확인
ros2 doctor --report | grep -A 5 "topic"
```

> 💡 `ros2 doctor`는 네트워크 설정, QoS 불일치, 미응답 노드 등의 문제를 자동으로 감지한다.

---

## 10.10 노드 간 통신 흐름 요약

전체적인 노드 간 데이터 통신의 흐름을 정리하면 다음과 같다.

```
[노드 A]                                   [노드 B]
  |                                            |
  |-- 퍼블리셔 생성 (토픽 이름, 메시지 타입) --|
  |-- 타이머로 주기적 메시지 발행 ------------>|
  |                                            |-- 서브스크라이버 생성 (토픽, QoS)
  |                                            |-- 콜백 함수에서 메시지 처리
  |                                            |
  |<-------- 서비스 요청 ----------------------|
  |-- 서비스 서버에서 응답 반환 -------------->|
```

---

## 10.11 정리

ROS 2 노드와 데이터 통신의 핵심 내용을 요약하면 다음과 같다.

| 개념 | 설명 |
|------|------|
| ROS 2 그래프 | 동시에 데이터를 처리하는 ROS 2 요소들의 네트워크 |
| 노드 | ROS 2의 독립적 실행 단위, 단일 모듈 목적을 담당 |
| 메시지 | 노드 간 전달하는 데이터 구조 (`.msg` 파일) |
| 토픽 | 비동기 Pub/Sub 패턴의 데이터 채널 |
| 서비스 | 동기 Request/Response 통신 |
| 액션 | 장기 작업을 위한 Goal/Feedback/Result 통신 |
| 콜백 | 메시지 수신 시 자동 호출되는 함수 |
| Executor | 콜백 실행을 관리하는 메커니즘 (Single/Multi Threaded) |
| Timer | 주기적으로 콜백을 실행하는 기능 |
| ros2doctor | 환경 진단 및 문제 감지 도구 |

노드와 통신 구조에 대한 이해는 ROS 2 개발의 근간이다.  
이 개념들을 바탕으로, 다음 장에서는 토픽 통신을 직접 구현하는 방법을 실습한다.
