# Slide 1: 타이틀

# 08. DDS의 서비스 품질 (QoS)

### 요구조건에 딱 맞는 데이터 통신 설계하기

---

# Slide 2: QoS(Quality of Service)란?

### "그냥 보내기"가 아닌 "맞춤형 전달"

* **정의**: 데이터 통신을 요구조건에 맞게 전달 특성을 설계하는 기능임.
* **핵심 역할**: 유실 허용 여부, 지연과 안정성의 트레이드오프, 과거 데이터 제공 여부 등을 선택할 수 있는 구조를 제공함.
* **주요 기능**: 통신 품질 설계(전달 보장, 생존 감지 등)를 통해 시스템의 안정성을 확보함.

---

# Slide 3: 핵심 QoS 종류 (1) 신뢰성과 히스토리

### 데이터를 어떻게 믿고 보관할 것인가?

* **Reliability (신뢰성)**:
* **BEST_EFFORT**: 빠르지만 데이터 유실을 일부 허용함.
* **RELIABLE**: 전달 보장을 최우선으로 하며, 필요시 재전송을 수행함.


* **History (히스토리)**:
* **KEEP_LAST**: 최신 N개 데이터만 유지함.
* **KEEP_ALL**: 가능한 한 모든 데이터를 유지함 (메모리 사용 증가 주의).



---

# Slide 4: 핵심 QoS 종류 (2) 깊이와 지속성

### 버퍼의 크기와 과거 데이터의 가치

* **Depth (큐 깊이)**: `KEEP_LAST` 방식에서 최근 몇 개의 데이터를 보관할지 결정하는 값임 (예: Depth=10).
* **Durability (지속성)**:
* **VOLATILE**: 현재 연결된 구독자에게만 실시간으로 전달함.
* **TRANSIENT_LOCAL**: 나중에 참여한 구독자에게도 가장 최근 데이터를 제공함.



---

# Slide 5: 고급 QoS 정책들

### 기한, 생존성, 그리고 유효 기간

* **Deadline (기한)**: 일정 주기 안에 데이터가 도착해야 한다는 시간 제약을 정의함.
* **Liveliness (생존성)**: 노드가 살아있는지 확인하고 통신 끊김을 감지함.
* **Lifespan (수명)**: 데이터가 유효한 만료 시간을 설정하여 오래된 데이터의 전달을 막음.
* **Latency Budget**: 시스템에 허용 가능한 지연 시간 힌트를 제공함.

---

# Slide 6: ROS 2 QoS 정책 구현

### 코드에 적용하는 통신 옵션

* **History & Depth 설정**:
* C++: `rclcpp::QoS(rclcpp::KeepLast(10));`
* Python: `QoSProfile(history=KEEP_LAST, depth=10)`


* **Reliability 설정**:
* 빠르고 유실 허용 시 `.best_effort()` 또는 `RELIABILITY_BEST_EFFORT` 사용.


* **Durability 설정**:
* 과거 데이터 필요 시 `.transient_local()` 또는 `DURABILITY_TRANSIENT_LOCAL` 사용.



---

# Slide 7: rmw_qos_profile 프리셋 비교

### 상황별 최적의 QoS 조합

| 항목 | Default | Sensor Data | Service | Action Status |
| --- | --- | --- | --- | --- |
| **Reliability** | RELIABLE | **BEST_EFFORT** | RELIABLE | RELIABLE |
| **History** | KEEP_LAST | KEEP_LAST | KEEP_LAST | KEEP_LAST |
| **Depth** | 10 | **5** | 10 | 1 |
| **Durability** | VOLATILE | VOLATILE | VOLATILE | **TRANSIENT_LOCAL** |

---

# Slide 8: 주요 프리셋 상세 분석

### Sensor Data와 Default의 차이

* **Default Profile**: 일반적인 토픽 통신용으로, 안정적으로 데이터를 받고 최근 10개를 버퍼링함.
* **Sensor Data Profile**:
* 카메라, LiDAR 등 고주기 센서 스트림에 최적화됨.
* 지연을 줄이기 위해 일부 유실을 허용하고 최신 데이터(Depth 5) 위주로 처리함.


* **Action/Parameter**: 파라미터는 큰 버퍼(1,000)를, 액션 상태는 늦게 온 구독자도 알 수 있게 `TRANSIENT_LOCAL`을 사용함.

---

# Slide 9: 유저 커스텀 QoS 프로파일

### 나만의 통신 정책 만들기

* **정의**: 애플리케이션에서 직접 정의하는 "사용자 친화적 QoS"로, 내부적으로 DDS 통신 구조로 변환됨.
* **작성 예시 (Python)**:
```python
QoS_RK10V = QoSProfile(
    reliability=QoSReliabilityPolicy.RELIABLE,
    history=QoSHistoryPolicy.KEEP_LAST,
    depth=10,
    durability=QoSDurabilityPolicy.VOLATILE
)

```


* **적용**: `create_publisher` 또는 `create_subscription` 호출 시 해당 프로파일을 매개변수로 전달함.

---

# Slide 10: QoS 테스트와 마무리

### 의도한 대로 동작하는지 검증하기

* **테스트 목적**: 단순 통신 여부가 아니라 유실, 지연, 과거 데이터 제공 등이 요구사항에 맞는지 확인하는 것임.
* **주요 점검 사항**: 퍼블리셔와 서브스크라이버의 QoS가 맞지 않아 발생하는 통신 단절(특히 Reliability 불일치)을 조기에 발견함.
* **결론**: QoS는 안정적인 로봇 시스템 운영을 위한 필수 설계 옵션임.

---
