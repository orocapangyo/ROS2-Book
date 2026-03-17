# 8장. DDS의 QoS

## 8.1 DDS의 서비스 품질(QoS, Quality of Service)란?

**QoS(Quality of Service)** 는 쉽게 말하면 **“데이터 통신 옵션 세트”** 입니다.  
ROS2에서는 DDS의 QoS 개념을 그대로 도입하여, 퍼블리셔/서브스크라이버/서비스/액션 등을 선언할 때  
원하는 QoS를 지정할 수 있습니다.

QoS를 통해 조절할 수 있는 대표적인 요소는 다음과 같습니다.

- 데이터 전송의 **실시간성**
- 네트워크 **대역폭 사용 방식**
- 메시지의 **지속성(보관 여부)**
- 시스템의 **중복성/신뢰성** 등

즉, QoS는 “어떻게 통신할 것인가?”에 대한 **동작 정책**을 정해 주는 역할을 합니다.

---

## 8.2 ROS2에서 사용하는 주요 QoS 옵션

DDS 사양서에는 22가지 이상의 QoS 정책이 정의되어 있지만,  
ROS2에서 주로 사용하는 핵심 옵션은 다음과 같습니다.

- `Reliability`
- `History`
- `Durability`
- `Deadline`
- `Lifespan`
- `Liveliness`

아래에서 하나씩 살펴보겠습니다.

### 8.2.1 History – 얼마나 오래/많이 보관할까?

`History` 는 **데이터를 몇 개나 보관할지** 결정하는 옵션입니다.

1. **KEEP_LAST**
   - 정해진 메시지 큐 크기(`depth`) 만큼의 데이터만 보관합니다.
   - 가장 흔히 사용하는 설정입니다.

2. **KEEP_ALL**
   - 가능한 모든 데이터를 보관합니다.
   - 메시지 큐 크기는 DDS 벤더 구현에 따라 달라질 수 있습니다.

### 8.2.2 Reliability – 속도 vs 신뢰성

`Reliability` 는 데이터 전송에서 **속도와 신뢰성의 우선순위**를 정하는 옵션입니다.  
주의할 점은, Pub/Sub 간 설정 조합에 제약이 있다는 것입니다  
(예: Pub가 `BEST_EFFORT`, Sub가 `RELIABLE` 인 조합은 사용할 수 없음).

1. **BEST_EFFORT**
   - 데이터 송신에 집중합니다.
   - 전송 속도를 중시하며, 일부 유실이 발생할 수 있습니다.

2. **RELIABLE**
   - 데이터 수신 보장을 우선합니다.
   - 유실 시 재전송을 통해 데이터 수신을 최대한 보장하지만,  
     재전송으로 인해 지연·끊김이 증가할 수 있습니다.

### 8.2.3 Durability – 나중에 붙은 구독자에게도 줄까?

`Durability` 는 **구독자가 생성되기 전의 데이터를 어떻게 처리할지**를 결정하는 QoS 옵션입니다.  
마찬가지로 Pub/Sub 조합에 제약이 있습니다  
(예: Pub이 `VOLATILE`, Sub가 `TRANSIENT_LOCAL` 인 조합은 허용되지 않음).

1. **TRANSIENT_LOCAL**
   - 퍼블리셔가 데이터를 **로컬에 보관**합니다.
   - 나중에 구독을 시작한 서브스크라이버도, 이전의 데이터를 전달받을 수 있습니다.

2. **VOLATILE**
   - 구독자가 생성되기 전의 데이터는 유효하지 않습니다.
   - **실시간 현재값만 중요**한 센서 데이터 등에 자주 사용됩니다.

### 8.2.4 Deadline – 제때 안 오면 이벤트 발생

`Deadline` 은 정해진 주기 안에 데이터가 발신·수신되지 않을 경우  
**이벤트 콜백(EventCallback)을 발생시키는 옵션**입니다.

- `deadline_duration` 값으로 허용 주기를 설정합니다.
- 단, 퍼블리셔의 주기가 구독자보다 **더 느리면**(더 큰 값이면) 설정할 수 없습니다.  
  - 예: Pub: 2000ms, Sub: 1000ms 는 불가

이 옵션을 사용하면,

- “이 센서가 **기대했던 주기보다 느려지면** 알림을 받고 싶다”  
와 같은 요구사항을 QoS로 표현할 수 있습니다.

### 8.2.5 Lifespan – 유통기한이 지난 데이터는 버리기

`Lifespan` 은 **정해진 시간 안에 수신되지 않은 데이터는 무효**로 처리하는 옵션입니다.

- `lifespan_duration` 값으로 유효 시간을 설정합니다.
- 예를 들어, 100ms 안에 도착하지 않은 데이터는 더 이상 의미가 없다고 판단하고 삭제할 수 있습니다.

실시간성이 중요한 센서 데이터(레이저 스캔, 카메라 등)에서 유용하게 사용할 수 있습니다.

### 8.2.6 Liveliness – 노드/토픽이 살아 있는지 감시

`Liveliness` 는 **노드 혹은 토픽이 여전히 살아 있는지**를 일정 주기로 확인하는 옵션입니다.

- `liveliness` 옵션으로 확인 방식을 지정합니다.
  - `AUTOMATIC`
  - `MANUAL_BY_NODE`
  - `MANUAL_BY_TOPIC`
- `lease_duration` 값으로 확인 주기를 설정합니다.

설정에 따라 동작 주체가 달라집니다.

- `AUTOMATIC`
  - 생존성 관리는 DDS 미들웨어가 담당합니다.
- `MANUAL_*`
  - 생존성 관리를 사용자가 직접 호출해 주어야 합니다.

주의할 점:

- Pub가 `AUTOMATIC` 인 경우에는 **수동(manual) 모드**를 사용할 수 없습니다.
- Pub가 `MANUAL_BY_NODE` 인 경우에는 `MANUAL_BY_TOPIC` 조합을 사용할 수 없습니다.

---

## 8.3 `rmw_qos_profile` 와 사용자 정의 QoS 프로파일

ROS2에서는 QoS를 보다 쉽게 사용하기 위해, **자주 쓰는 QoS 조합을 미리 세트로 정의**해 두었습니다.

### 8.3.1 `rmw_qos_profile` – 미리 정의된 프로파일

RMW 계층에서 다음과 같은 기본 QoS 프로파일을 제공합니다.

- `rmw_qos_profile_default`
- `rmw_qos_profile_sensor_data`
- `rmw_qos_profile_services_default`
- `rmw_qos_profile_parameter_events`
- `rmw_qos_profile_parameters`
- `rmw_qos_profile_action_status`

각 프로파일은 주로 다음 항목을 설정합니다.

- `Reliability`
- `History`
- `Depth`
- `Durability`

예를 들어, 센서 데이터에는 `rmw_qos_profile_sensor_data` 를 사용하는 식으로  
상황에 맞는 기본 프로파일을 골라 쓰면 됩니다.

### 8.3.2 사용자 정의 QoS 프로파일

보다 세밀한 제어가 필요하다면, 사용자가 **직접 QoS 프로파일을 정의**할 수 있습니다.

Python 예시:

```python
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy

qos = QoSProfile(
    reliability=ReliabilityPolicy.BEST_EFFORT,
    history=HistoryPolicy.KEEP_LAST,
    depth=10,
)
```

이렇게 만든 `QoSProfile` 객체를 퍼블리셔/서브스크라이버 선언 시 인자로 넘기면 됩니다.

```python
node.create_publisher(MsgType, 'topic_name', qos)
```

이 장에서 설명한 옵션 의미를 이해하고 나면,  
실제 프로젝트에서 **“통신이 원하는 대로 동작하지 않을 때”**  
어디를 어떻게 조정해야 할지 훨씬 명확해집니다.

