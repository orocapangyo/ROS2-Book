# 8장 DDS의 QoS (Quality of Service)

## 8.1 QoS란?

QoS는 **데이터 통신의 옵션 설정**이다.
ROS 2는 DDS 도입으로 TCP 방식(신뢰성 우선)과 UDP 방식(속도 우선)을 선택적으로 사용할 수 있으며,
퍼블리셔/서브스크라이버 선언 시 QoS를 매개변수로 지정한다.

DDS 사양상 설정 가능한 QoS 항목은 **22가지**이며, ROS 2에서는 그 중 **6가지**를 주로 사용한다.

---

## 8.2 ROS 2 주요 QoS 6가지

### History — 데이터 보관 개수

| 값 | 설명 |
|---|---|
| `KEEP_LAST` | `depth` 크기만큼만 보관 |
| `KEEP_ALL` | 모든 데이터 보관 (벤더마다 한도 다름) |

```python
# rclpy
QoSProfile(history=QoSHistoryPolicy.KEEP_LAST, depth=10)
```

---

### Reliability — 신뢰성 vs 속도

| 값 | 설명 |
|---|---|
| `RELIABLE` | 손실 시 재전송 → 신뢰성 우선 (TCP 방식) |
| `BEST_EFFORT` | 손실 허용 → 속도 우선 (UDP 방식) |

**RxO (Pub ↓ / Sub →)**

| | BEST_EFFORT | RELIABLE |
|---|---|---|
| BEST_EFFORT | 가능 | 불가 |
| RELIABLE | BEST_EFFORT | 가능 |

```python
QoSProfile(reliability=QoSReliabilityPolicy.BEST_EFFORT)
```

---

### Durability — 구독자 생성 전 데이터 처리

| 값 | 설명 |
|---|---|
| `TRANSIENT_LOCAL` | 구독자 생성 전 데이터도 보관 (Publisher에만 적용) |
| `VOLATILE` | 구독자 생성 전 데이터 폐기 |

**RxO (Pub ↓ / Sub →)**

| | TRANSIENT_LOCAL | VOLATILE |
|---|---|---|
| TRANSIENT_LOCAL | 가능 | 가능 |
| VOLATILE | 불가 | 가능 |

```python
QoSProfile(durability=QoSDurabilityPolicy.TRANSIENT_LOCAL)
```

---

### Deadline — 주기 내 미수신 시 이벤트 발생

정해진 주기 안에 데이터가 발신/수신되지 않으면 EventCallback 실행.

**RxO (Pub ↓ / Sub →, 단위: ms)**

| | 1000ms | 2000ms |
|---|---|---|
| 1000ms | 가능 | 가능 |
| 2000ms | 불가 | 가능 |

```python
QoSProfile(depth=10, deadline=Duration(0.1))
```

---

### Lifespan — 유효 기간 초과 데이터 삭제

정해진 주기 안에 수신된 데이터만 유효 처리, 초과 데이터는 삭제.

```python
QoSProfile(lifespan=Duration(0.01))
```

---

### Liveliness — 노드/토픽 생존 확인

정해진 주기 안에서 노드 또는 토픽의 생존 여부를 확인.

| 값 | 설명 |
|---|---|
| `AUTOMATIC` | DDS 미들웨어가 자동으로 생존성 관리 |
| `MANUAL_BY_NODE` | 사용자가 노드 단위로 수동 관리 |
| `MANUAL_BY_TOPIC` | 사용자가 토픽 단위로 수동 관리 |

**RxO (Pub ↓ / Sub →)**

| | AUTOMATIC | MANUAL_BY_NODE | MANUAL_BY_TOPIC |
|---|---|---|---|
| AUTOMATIC | 가능 | 불가 | 불가 |
| MANUAL_BY_NODE | 가능 | 가능 | 불가 |
| MANUAL_BY_TOPIC | 가능 | 가능 | 가능 |

```python
QoSProfile(liveliness=AUTOMATIC, liveliness_lease_duration=Duration(1.0))
```

---

## 8.3 RMW QoS 프로파일 (사전 정의)

자주 사용하는 QoS 조합을 세트로 미리 정의해 둔 것.

| 프로파일 | Reliability | History | Depth | Durability |
|---|---|---|---|---|
| Default | RELIABLE | KEEP_LAST | 10 | VOLATILE |
| Sensor Data | BEST_EFFORT | KEEP_LAST | 5 | VOLATILE |
| Service | RELIABLE | KEEP_LAST | 10 | VOLATILE |
| Action Status | RELIABLE | KEEP_LAST | 1 | TRANSIENT_LOCAL |
| Parameters | RELIABLE | KEEP_LAST | 1,000 | VOLATILE |
| Parameter Events | RELIABLE | KEEP_LAST | 1,000 | VOLATILE |

```python
# Sensor Data 프로파일 사용 예
from rclpy.qos import qos_profile_sensor_data

self.pub = self.create_publisher(Int8MultiArray, 'sensor', qos_profile_sensor_data)
```

---

## 8.4 유저 QoS 프로파일 (커스텀)

직접 옵션을 조합해 프로파일을 정의할 수 있다.

```python
from rclpy.qos import QoSDurabilityPolicy, QoSHistoryPolicy, QoSProfile, QoSReliabilityPolicy

QOS_RKL10V = QoSProfile(
    reliability=QoSReliabilityPolicy.RELIABLE,
    history=QoSHistoryPolicy.KEEP_LAST,
    depth=10,
    durability=QoSDurabilityPolicy.VOLATILE)

self.pub = self.create_publisher(Int8MultiArray, 'sensor', QOS_RKL10V)
```

> 유저 QoS 프로파일 방식이 자유도가 높아 실제 개발에서 더 많이 사용된다.

---

# 정리

| QoS | 핵심 역할 | 주요 선택지 |
|---|---|---|
| History | 보관 데이터 개수 | KEEP_LAST / KEEP_ALL |
| Reliability | 신뢰성 vs 속도 | RELIABLE / BEST_EFFORT |
| Durability | 구독 전 데이터 처리 | TRANSIENT_LOCAL / VOLATILE |
| Deadline | 주기 내 미전송 감지 | 주기(duration) 설정 |
| Lifespan | 유효 기간 초과 데이터 삭제 | 주기(duration) 설정 |
| Liveliness | 노드/토픽 생존 확인 | AUTOMATIC / MANUAL_BY_NODE / MANUAL_BY_TOPIC |

QoS는 통신 환경과 데이터 특성에 맞게 설정하는 것이 핵심이다.
센서처럼 속도가 중요한 데이터는 `BEST_EFFORT`, 명령/서비스처럼 신뢰성이 중요한 데이터는 `RELIABLE`을 사용한다.
