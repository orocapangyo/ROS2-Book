# 12장 ROS 2 서비스

> **참고**: [ROS 2 Jazzy 공식 튜토리얼 - Understanding services](https://docs.ros.org/en/jazzy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Services/Understanding-ROS2-Services.html) 기반으로 작성

## 12.1 개요

서비스(Service)는 ROS 2에서 **요청(Request)과 응답(Response)** 패턴의 통신 방식이다.  
토픽이 지속적인 데이터 스트리밍에 적합하다면, 서비스는 **특정 동작을 요청하고 결과를 받는** 일회성 통신에 적합하다.

> 📌 **공식 문서**: "Services are another method of communication for nodes in the ROS graph. Services are based on a call-and-response model versus the publisher-subscriber model of topics. While topics allow nodes to subscribe to data streams and get continual updates, services only provide data when they are specifically called by a client."

---

## 12.2 서비스 통신의 구조

### 12.2.1 서비스 클라이언트와 서버

**단일 서비스 클라이언트:**

![서비스: 단일 클라이언트와 서버](images/service_single.gif)

**다중 서비스 클라이언트:**

![서비스: 다중 클라이언트](images/service_multiple.gif)

- **서비스 서버 (Service Server)**: 요청을 받아 처리하고 응답을 반환
- **서비스 클라이언트 (Service Client)**: 서버에 요청을 보내고 응답을 기다림
- **서비스 이름**: 서버와 클라이언트를 연결하는 식별자

### 12.2.2 토픽과 서비스 비교

| 특성 | 토픽 (Topic) | 서비스 (Service) |
|------|-------------|-----------------|
| 통신 패턴 | Pub/Sub (단방향) | Request/Response (양방향) |
| 동기/비동기 | 비동기 | 동기 (응답 대기) |
| 관계 | N:N (다대다) | 1:N (서버 1개 : 클라이언트 다수) |
| 데이터 흐름 | 지속적 스트리밍 | 필요 시 1회 호출 |
| 적합한 상황 | 센서 데이터, 연속 명령 | 설정 변경, 상태 조회, 단발 작업 |

---

## 12.3 서비스 타입 (.srv)

서비스 타입은 요청과 응답의 데이터 구조를 정의한다.  
`.srv` 파일로 정의하며, `---` 구분자로 요청과 응답을 나눈다.

### 서비스 타입 확인

```bash
# 사용 가능한 서비스 타입 목록
ros2 interface list | grep srv

# 특정 서비스 타입 구조 확인
ros2 interface show example_interfaces/srv/AddTwoInts
```

**출력:**
```
int64 a
int64 b
---
int64 sum
```

> 💡 `---` 위쪽이 **요청(Request)**, 아래쪽이 **응답(Response)** 구조이다.

```bash
# turtlesim/srv/Spawn 서비스 타입 구조 확인
ros2 interface show turtlesim/srv/Spawn
```

**출력:**
```
float32 x
float32 y
float32 theta
string name  # Optional. A unique name will be created and returned if this is empty
---
string name
```

### 주요 표준 서비스 타입

| 서비스 타입 | 요청 | 응답 | 사용 예 |
|------------|------|------|---------|
| `std_srvs/srv/Empty` | (없음) | (없음) | 리셋, 초기화 |
| `std_srvs/srv/SetBool` | `bool data` | `bool success`, `string message` | 기능 ON/OFF |
| `std_srvs/srv/Trigger` | (없음) | `bool success`, `string message` | 동작 트리거 |
| `example_interfaces/srv/AddTwoInts` | `int64 a, b` | `int64 sum` | 덧셈 연산 |

### 커스텀 서비스 타입 정의

**서비스 파일 (srv/GetBatteryStatus.srv)**
```
string robot_name       # 요청: 로봇 이름
---
float64 battery_level   # 응답: 배터리 잔량 (%)
bool is_charging        # 응답: 충전 중 여부
string status_message   # 응답: 상태 메시지
```

---

## 12.4 서비스 서버 구현 (Python)

### 기본 서비스 서버

```python
import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts

class MinimalServiceServer(Node):
    def __init__(self):
        super().__init__('minimal_service_server')
        # 서비스 서버 생성: 서비스 타입, 서비스 이름, 콜백 함수
        self.srv = self.create_service(
            AddTwoInts,
            '/add_two_ints',
            self.add_two_ints_callback
        )
        self.get_logger().info('서비스 서버가 준비되었습니다.')

    def add_two_ints_callback(self, request, response):
        response.sum = request.a + request.b
        self.get_logger().info(f'요청: {request.a} + {request.b} = {response.sum}')
        return response

def main(args=None):
    rclpy.init(args=args)
    node = MinimalServiceServer()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 12.5 서비스 클라이언트 구현 (Python)

### 기본 서비스 클라이언트

```python
import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts

class MinimalServiceClient(Node):
    def __init__(self):
        super().__init__('minimal_service_client')
        # 서비스 클라이언트 생성
        self.cli = self.create_client(AddTwoInts, '/add_two_ints')
        # 서비스 서버가 준비될 때까지 대기
        while not self.cli.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('서비스 대기 중...')
        self.req = AddTwoInts.Request()

    def send_request(self, a, b):
        self.req.a = a
        self.req.b = b
        # 비동기 호출 — 블로킹 없이 요청 전송
        self.future = self.cli.call_async(self.req)

def main(args=None):
    rclpy.init(args=args)
    node = MinimalServiceClient()
    node.send_request(3, 5)

    while rclpy.ok():
        rclpy.spin_once(node)
        if node.future.done():
            response = node.future.result()
            node.get_logger().info(f'결과: {response.sum}')
            break

    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 12.6 서비스 서버 구현 (C++)

### 기본 서비스 서버

```cpp
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

class MinimalServiceServer : public rclcpp::Node {
public:
    MinimalServiceServer() : Node("minimal_service_server") {
        service_ = this->create_service<example_interfaces::srv::AddTwoInts>(
            "/add_two_ints",
            std::bind(&MinimalServiceServer::add_callback, this,
                      std::placeholders::_1, std::placeholders::_2)
        );
        RCLCPP_INFO(this->get_logger(), "서비스 서버가 준비되었습니다.");
    }

private:
    void add_callback(
        const std::shared_ptr<example_interfaces::srv::AddTwoInts::Request> request,
        std::shared_ptr<example_interfaces::srv::AddTwoInts::Response> response)
    {
        response->sum = request->a + request->b;
        RCLCPP_INFO(this->get_logger(), "요청: %ld + %ld = %ld",
                    request->a, request->b, response->sum);
    }
    rclcpp::Service<example_interfaces::srv::AddTwoInts>::SharedPtr service_;
};

int main(int argc, char *argv[]) {
    rclcpp::init(argc, argv);
    rclcpp::spin(std::make_shared<MinimalServiceServer>());
    rclcpp::shutdown();
    return 0;
}
```

---

## 12.7 서비스 클라이언트 구현 (C++)

### 기본 서비스 클라이언트

```cpp
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

class MinimalServiceClient : public rclcpp::Node {
public:
    MinimalServiceClient() : Node("minimal_service_client") {
        client_ = this->create_client<example_interfaces::srv::AddTwoInts>(
            "/add_two_ints"
        );
        // 서비스 서버 대기
        while (!client_->wait_for_service(std::chrono::seconds(1))) {
            RCLCPP_INFO(this->get_logger(), "서비스 대기 중...");
        }
    }

    auto send_request(int64_t a, int64_t b) {
        auto request = std::make_shared<example_interfaces::srv::AddTwoInts::Request>();
        request->a = a;
        request->b = b;
        return client_->async_send_request(request);
    }

private:
    rclcpp::Client<example_interfaces::srv::AddTwoInts>::SharedPtr client_;
};

int main(int argc, char *argv[]) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<MinimalServiceClient>();
    auto result = node->send_request(3, 5);

    if (rclcpp::spin_until_future_complete(node, result) ==
        rclcpp::FutureReturnCode::SUCCESS)
    {
        RCLCPP_INFO(node->get_logger(), "결과: %ld", result.get()->sum);
    }

    rclcpp::shutdown();
    return 0;
}
```

---

## 12.8 ros2 service 명령어

서비스를 확인하고 테스트하기 위한 명령어들이다.

### 주요 명령어

```bash
# 현재 활성화된 서비스 목록
ros2 service list
```

**turtlesim 실행 시 출력 예시:**
```
/clear
/kill
/reset
/spawn
/teleop_turtle/describe_parameters
/teleop_turtle/get_parameter_types
/teleop_turtle/get_parameters
/teleop_turtle/list_parameters
/teleop_turtle/set_parameters
/teleop_turtle/set_parameters_atomically
/turtle1/set_pen
/turtle1/teleport_absolute
/turtle1/teleport_relative
/turtlesim/describe_parameters
/turtlesim/get_parameter_types
/turtlesim/get_parameters
/turtlesim/list_parameters
/turtlesim/set_parameters
/turtlesim/set_parameters_atomically
```

> 💡 `parameters`가 포함된 서비스는 ROS 2의 **파라미터 인프라**가 자동으로 제공하는 것이다. 거의 모든 노드에 존재한다.

```bash
# 서비스 목록 + 서비스 타입 함께 표시
ros2 service list -t

# 특정 서비스의 타입 확인
ros2 service type /spawn
# 출력: turtlesim/srv/Spawn

# 서비스 상세 정보 (클라이언트/서버 수)
ros2 service info /spawn
# 출력:
# Type: turtlesim/srv/Spawn
# Clients count: 0
# Services count: 1

# 특정 타입의 서비스 찾기
ros2 service find std_srvs/srv/Empty
# 출력: /clear, /reset

# 서비스 타입의 구조 확인
ros2 interface show turtlesim/srv/Spawn

# 명령줄에서 서비스 호출
ros2 service call /spawn turtlesim/srv/Spawn "{x: 2, y: 2, theta: 0.2, name: ''}"
```

**서비스 호출 결과:**
```
requester: making request: turtlesim.srv.Spawn_Request(x=2.0, y=2.0, theta=0.2, name='')
response: turtlesim.srv.Spawn_Response(name='turtle2')
```

### 서비스 인트로스펙션 (Jazzy 신기능)

Jazzy 버전에서는 `ros2 service echo` 명령으로 서비스 통신을 모니터링할 수 있다.

```bash
# 서비스 인트로스펙션 활성화 후 사용
ros2 service echo /add_two_ints
```

**출력 예시:**
```
info:
  event_type: REQUEST_SENT
  stamp: {sec: 1709408301, nanosec: 423227292}
  sequence_number: 618
request: [{a: 2, b: 3}]
response: []
---
info:
  event_type: RESPONSE_RECEIVED
  stamp: {sec: 1709408301, nanosec: 424153133}
  sequence_number: 618
request: []
response: [{sum: 5}]
```

> 💡 `ros2 service echo`는 서비스 인트로스펙션이 활성화되어 있어야 동작한다. 기본적으로는 비활성화 상태이다.

### 명령어 정리

| 명령어 | 기능 |
|--------|------|
| `ros2 service list` | 서비스 목록 조회 |
| `ros2 service list -t` | 서비스 목록 + 타입 |
| `ros2 service type` | 서비스 타입 확인 |
| `ros2 service info` | 서비스 상세 정보 (클라이언트/서버 수) |
| `ros2 service find` | 특정 타입의 서비스 검색 |
| `ros2 service call` | 명령줄에서 서비스 호출 |
| `ros2 service echo` | 서비스 통신 모니터링 (Jazzy) |
| `ros2 interface show` | 서비스 타입 구조 확인 |

---

## 12.9 실습: turtlesim과 서비스

turtlesim을 활용한 서비스 통신 실습이다.

### 실습 순서

```bash
# 1. turtlesim 노드 실행
ros2 run turtlesim turtlesim_node

# 2. 서비스 목록 확인
ros2 service list

# 3. /spawn 서비스 타입 확인
ros2 service type /spawn
# → turtlesim/srv/Spawn

# 4. 서비스 구조 확인
ros2 interface show turtlesim/srv/Spawn
# → x, y, theta, name (요청) / name (응답)

# 5. 새 거북이 생성
ros2 service call /spawn turtlesim/srv/Spawn "{x: 2, y: 2, theta: 0.2, name: ''}"

# 6. 거북이를 특정 위치로 순간이동
ros2 service call /turtle1/teleport_absolute \
  turtlesim/srv/TeleportAbsolute "{x: 5.0, y: 5.0, theta: 0.0}"

# 7. 거북이 펜 색상 변경 (빨간색, 두께 5)
ros2 service call /turtle1/set_pen \
  turtlesim/srv/SetPen "{r: 255, g: 0, b: 0, width: 5, off: 0}"

# 8. 화면 초기화 (Empty 타입 — 인자 없음)
ros2 service call /clear std_srvs/srv/Empty

# 9. 거북이 삭제
ros2 service call /kill turtlesim/srv/Kill "{name: 'turtle2'}"
```

### turtlesim 주요 서비스

| 서비스 이름 | 서비스 타입 | 설명 |
|------------|------------|------|
| `/spawn` | `turtlesim/srv/Spawn` | 새 거북이 생성 |
| `/kill` | `turtlesim/srv/Kill` | 거북이 제거 |
| `/clear` | `std_srvs/srv/Empty` | 배경 궤적 초기화 |
| `/reset` | `std_srvs/srv/Empty` | 시뮬레이터 전체 초기화 |
| `/turtle1/set_pen` | `turtlesim/srv/SetPen` | 펜 색상/두께/ON/OFF |
| `/turtle1/teleport_absolute` | `turtlesim/srv/TeleportAbsolute` | 절대 좌표 이동 |
| `/turtle1/teleport_relative` | `turtlesim/srv/TeleportRelative` | 상대 좌표 이동 |

---

## 12.10 서비스 사용 시 주의사항

### 12.10.1 동기 호출의 위험성

서비스 콜백 내에서 다른 서비스를 동기 호출하면 **데드락(Deadlock)**이 발생할 수 있다.

```python
# ❌ 잘못된 예: 콜백 안에서 동기 서비스 호출
def service_callback(self, request, response):
    result = self.other_client.call(other_request)  # 데드락 위험!
    return response

# ✅ 올바른 예: 비동기 호출 사용
def service_callback(self, request, response):
    future = self.other_client.call_async(other_request)
    return response
```

### 12.10.2 서비스 서버는 하나만 가능

하나의 서비스 이름에 대해 서버는 **단 하나만** 등록할 수 있다.  
같은 이름의 서비스 서버를 두 개 이상 등록하면 후자가 전자를 덮어쓴다.

### 12.10.3 서비스 vs 토픽 — 선택 기준

| 기준 | 토픽 사용 | 서비스 사용 |
|------|----------|------------|
| 데이터 흐름 | 지속적, 반복적 | 일회성, 필요 시 |
| 응답 필요 여부 | 불필요 (fire-and-forget) | 필수 (결과 확인) |
| 예시 | 센서 읽기, 모터 명령 | 설정 변경, 모드 전환, 맵 저장 |
| 실패 처리 | 개별 메시지 손실 허용 | 성공/실패 명확히 구분 |
| 서버 수 | 퍼블리셔 N개 가능 | 서버 1개만 가능 |

### 12.10.4 파라미터 서비스

모든 ROS 2 노드는 자동으로 **파라미터 관련 서비스**를 제공한다.  
`describe_parameters`, `get_parameters`, `set_parameters` 등이 있으며,  
`ros2 param` 명령어로도 접근할 수 있다.

```bash
# 파라미터 조회
ros2 param get /turtlesim background_r

# 파라미터 변경
ros2 param set /turtlesim background_r 200
```

---

## 12.11 정리

ROS 2 서비스의 핵심 내용을 요약하면 다음과 같다.

| 항목 | 설명 |
|------|------|
| 서비스 | Request/Response 패턴의 동기 통신 |
| 서비스 서버 | 요청을 받아 처리하고 응답 반환 |
| 서비스 클라이언트 | 서버에 요청을 보내고 응답 수신 |
| 서비스 타입 | `.srv` 파일로 요청/응답 구조 정의 (`---` 구분) |
| `ros2 service` | 서비스 조회, 호출을 위한 CLI 도구 |
| `ros2 service info` | 클라이언트/서버 수 확인 |
| `ros2 service echo` | 서비스 통신 모니터링 (Jazzy) |
| `ros2 interface show` | `.srv` 타입 구조 확인 |
| 동기 호출 주의 | 콜백 내 동기 호출은 데드락 위험 |
| 서버 단일 등록 | 같은 이름의 서비스 서버는 하나만 가능 |
| 파라미터 서비스 | 모든 노드가 자동 제공하는 파라미터 관련 서비스 |

서비스는 토픽과 함께 ROS 2의 핵심 통신 방식이다.  
토픽이 데이터의 흐름(stream)을 담당한다면, 서비스는 동작의 요청(command)을 담당한다.  
두 통신 방식을 적절히 조합하여 사용하는 것이 효과적인 로봇 소프트웨어 설계의 핵심이다.
