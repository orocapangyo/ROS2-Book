# 8장 DDS의 QoS

## 8.1 개요

QoS(Quality of Service)는 DDS 통신에서 **데이터 전달 방식과 품질을 제어하는 정책**이다.  
ROS 2는 DDS의 QoS를 지원함으로써, 단순한 메시지 전달을 넘어  
네트워크 환경과 응용 요구에 맞는 통신 전략을 선택할 수 있다.

---

## 8.2 QoS가 필요한 이유

모든 통신에 같은 전달 방식을 사용하면 비효율적이거나 위험할 수 있다.

### 예시 비교

| 상황                       | 적합한 통신 방식                          |
|----------------------------|-------------------------------------------|
| 카메라 이미지 스트리밍     | 빠른 전달이 중요, 일부 손실 허용         |
| 비상 정지 명령 전달        | 반드시 전달되어야 하고 손실은 절대 불허  |
| 맵 데이터 전달             | 늦게 구독해도 최신 맵을 받아야 함        |
| 센서 데이터 실시간 처리    | 오래된 데이터는 버리는 것이 나을 수 있음 |

이처럼 응용마다 최적의 통신 방식이 다르며, QoS는 이를 세밀하게 제어할 수 있게 한다.

---

## 8.3 주요 QoS 정책

### 8.3.1 Reliability (신뢰성)

메시지의 전달 보장 여부를 설정한다.

| 값            | 설명                                               |
|---------------|----------------------------------------------------|
| `RELIABLE`    | 메시지가 반드시 전달됨을 보장. 재전송 수행         |
| `BEST_EFFORT` | 최선을 다하지만, 전달 보장 없음. 빠른 전달 우선    |

**사용 예시**
- `RELIABLE`: 명령 전달, 상태 보고, 중요한 제어 데이터
- `BEST_EFFORT`: 카메라 이미지, 라이다 스트림 등 지속적 고빈도 데이터

---

### 8.3.2 Durability (내구성)

늦게 생긴 구독자에게 이전 데이터를 제공할지 여부를 설정한다.

| 값                    | 설명                                              |
|-----------------------|---------------------------------------------------|
| `VOLATILE`            | 구독 시작 이후의 메시지만 수신                    |
| `TRANSIENT_LOCAL`     | 구독자가 늦게 생겼을 때 마지막 메시지를 전달      |

**사용 예시**
- `TRANSIENT_LOCAL`: 맵 데이터, 로봇 초기 상태 정보 등 한 번 발행 후 계속 유효한 데이터
- `VOLATILE`: 센서 스트림처럼 항상 최신 데이터만 필요한 경우

---

### 8.3.3 History (히스토리)

메시지를 어떻게 보관할지 설정한다.

| 값          | 설명                                   |
|-------------|----------------------------------------|
| `KEEP_LAST` | 최근 N개의 메시지만 보관 (Depth 설정)  |
| `KEEP_ALL`  | 모든 메시지 보관 (메모리가 허용하는 한) |

**Depth**: `KEEP_LAST` 설정 시 보관할 메시지의 개수를 지정한다.

---

### 8.3.4 Deadline (데드라인)

메시지가 주기적으로 발행되어야 하는 경우, 최대 허용 주기를 설정한다.  
지정된 시간 내에 메시지가 발행되지 않으면 이벤트가 발생한다.

**사용 예시**
- 센서 데이터가 일정 주기 이상 오지 않으면 경고

---

### 8.3.5 Lifespan (생존 시간)

발행된 메시지의 유효 기간을 설정한다.  
지정된 시간이 지난 메시지는 구독자에게 전달되지 않는다.

**사용 예시**
- 오래된 장애물 위치 데이터는 무효화
- 최신 데이터만 처리해야 하는 상황

---

### 8.3.6 Liveliness (활성 상태)

퍼블리셔가 살아있음을 구독자가 확인할 수 있는 메커니즘이다.

| 값                     | 설명                                     |
|------------------------|------------------------------------------|
| `AUTOMATIC`            | DDS가 자동으로 퍼블리셔 생존을 감지      |
| `MANUAL_BY_TOPIC`      | 퍼블리셔가 직접 생존 신호를 보내야 함    |

---

### 8.3.7 Lease Duration (임대 기간)

Liveliness 정책과 함께 사용하며, 이 시간 내에 생존 신호가 없으면 비활성으로 간주한다.

---

## 8.4 QoS 호환성 (Compatibility)

퍼블리셔와 서브스크라이버의 QoS 설정이 호환되어야 통신이 연결된다.  
호환 규칙은 "요청(서브스크라이버) is offered by (퍼블리셔)" 원칙으로 동작한다.

### Reliability 호환 규칙

| 퍼블리셔        | 서브스크라이버   | 연결 가능 여부 |
|-----------------|------------------|----------------|
| RELIABLE        | RELIABLE         | ✅              |
| RELIABLE        | BEST_EFFORT      | ✅              |
| BEST_EFFORT     | RELIABLE         | ❌              |
| BEST_EFFORT     | BEST_EFFORT      | ✅              |

**핵심 원칙**: 서브스크라이버가 요구하는 수준을 퍼블리셔가 충족해야 연결된다.

### Durability 호환 규칙

| 퍼블리셔        | 서브스크라이버      | 연결 가능 여부 |
|-----------------|---------------------|----------------|
| TRANSIENT_LOCAL | TRANSIENT_LOCAL     | ✅              |
| TRANSIENT_LOCAL | VOLATILE            | ✅              |
| VOLATILE        | TRANSIENT_LOCAL     | ❌              |
| VOLATILE        | VOLATILE            | ✅              |

---

## 8.5 ROS 2에서 QoS 설정 방법

### C++에서 QoS 설정

```cpp
#include "rclcpp/rclcpp.hpp"

// 기본 QoS: Depth 10
auto qos = rclcpp::QoS(10);

// 신뢰성 설정
qos.reliability(rclcpp::ReliabilityPolicy::Reliable);

// 내구성 설정
qos.durability(rclcpp::DurabilityPolicy::TransientLocal);

// 퍼블리셔 생성
auto publisher = node->create_publisher<std_msgs::msg::String>("/topic", qos);
```

### Python에서 QoS 설정

```python
from rclpy.node import Node
from rclpy.qos import QoSProfile, ReliabilityPolicy, DurabilityPolicy

qos_profile = QoSProfile(
    depth=10,
    reliability=ReliabilityPolicy.RELIABLE,
    durability=DurabilityPolicy.TRANSIENT_LOCAL
)

publisher = self.create_publisher(String, '/topic', qos_profile)
```

---

## 8.6 사전 정의된 QoS 프로파일

ROS 2는 자주 사용되는 QoS 조합을 미리 정의해두었다.

| 프로파일                    | 설명                                                     |
|-----------------------------|----------------------------------------------------------|
| `qos_profile_sensor_data`   | BEST_EFFORT, VOLATILE, Depth 5 — 센서 스트림에 적합       |
| `qos_profile_parameters`    | RELIABLE, VOLATILE — 파라미터 통신용                     |
| `qos_profile_services_default` | RELIABLE — 서비스 통신 기본값                        |
| `qos_profile_system_default` | 시스템 기본값 사용                                      |

### C++에서 프로파일 사용

```cpp
#include "rmw/qos_profiles.h"

// 센서 데이터 프로파일 사용
auto qos = rclcpp::QoS(
    rclcpp::QoSInitialization::from_rmw(rmw_qos_profile_sensor_data)
);
```

---

## 8.7 QoS 활용 시나리오

### 시나리오 1: 카메라 이미지 스트리밍

카메라 이미지는 고빈도로 발행되며, 최신 이미지가 중요하다.  
오래된 이미지를 기다리는 것보다 최신 이미지를 빠르게 받는 것이 낫다.

```python
# 권장 설정
QoSProfile(
    depth=1,
    reliability=ReliabilityPolicy.BEST_EFFORT,
    durability=DurabilityPolicy.VOLATILE,
)
```

### 시나리오 2: 비상 정지 명령

비상 정지 명령은 반드시 전달되어야 한다.

```python
# 권장 설정
QoSProfile(
    depth=10,
    reliability=ReliabilityPolicy.RELIABLE,
    durability=DurabilityPolicy.VOLATILE,
)
```

### 시나리오 3: 지도(Map) 데이터 전달

맵 데이터는 한 번 생성되면 새로운 구독자에게도 제공되어야 한다.

```python
# 권장 설정
QoSProfile(
    depth=1,
    reliability=ReliabilityPolicy.RELIABLE,
    durability=DurabilityPolicy.TRANSIENT_LOCAL,
)
```

---

## 8.8 ros2 topic 명령어로 QoS 확인하기

실행 중인 토픽의 QoS 설정을 다음 명령으로 확인할 수 있다.

```bash
# 토픽 정보 확인 (QoS 포함)
ros2 topic info /topic_name --verbose

# 출력 예시
Type: std_msgs/msg/String
Publisher count: 1
Subscription count: 1

Node name: my_node
Reliability: RELIABLE
Durability: VOLATILE
Lifespan: infinite
Deadline: infinite
Liveliness: AUTOMATIC
```

---

## 8.9 정리

QoS 정책별 요약을 정리하면 다음과 같다.

| QoS 항목     | 주요 선택지                   | 적용 목적                             |
|--------------|-------------------------------|---------------------------------------|
| Reliability  | RELIABLE / BEST_EFFORT        | 메시지 전달 보장 여부                 |
| Durability   | VOLATILE / TRANSIENT_LOCAL    | 늦게 생긴 구독자에게 과거 데이터 제공 |
| History      | KEEP_LAST / KEEP_ALL          | 메시지 보관 방식                      |
| Deadline     | Duration 설정                 | 메시지 발행 주기 감시                 |
| Lifespan     | Duration 설정                 | 메시지 유효 기간                      |
| Liveliness   | AUTOMATIC / MANUAL_BY_TOPIC   | 퍼블리셔 생존 감지                    |

ROS 2의 QoS 기능은 단순한 통신 이상의 것을 가능하게 한다.  
네트워크 상태, 데이터 특성, 시스템 요구에 따라 통신 방식을 정밀하게 조절함으로써,  
**더 안정적이고, 더 효율적이며, 더 신뢰할 수 있는 로봇 시스템**을 구성할 수 있다.
