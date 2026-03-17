# 7장 ROS 2와 DDS

## 7.1 개요

ROS 2는 통신 미들웨어로 **DDS(Data Distribution Service)** 를 채택했다.  
DDS는 분산 시스템에서의 실시간 데이터 통신을 위한 산업 표준 프로토콜이다.  
이 장에서는 DDS가 무엇인지, 왜 ROS 2가 DDS를 선택했는지, 그리고 ROS 2와 DDS의 관계를 살펴본다.

---

## 7.2 DDS란 무엇인가?

### 정의

DDS(Data Distribution Service)는 OMG(Object Management Group)에서 정의한  
**분산 실시간 통신 미들웨어 표준**이다.

네트워크에 연결된 여러 시스템 간에 데이터를 안정적이고 효율적으로 교환할 수 있도록 설계되었다.

### DDS의 주요 특징

| 특징              | 설명                                               |
|-------------------|----------------------------------------------------|
| 발행-구독 모델    | 퍼블리셔/서브스크라이버 방식으로 데이터 교환       |
| 분산 발견         | 중앙 브로커 없이 참여자 자동 발견                 |
| QoS 지원          | 다양한 품질 정책으로 통신 방식 조절                |
| 실시간성          | 예측 가능한 지연 시간으로 실시간 시스템에 적합     |
| 다양한 구현체     | 여러 벤더가 표준에 맞는 구현체를 제공             |

### DDS의 적용 분야

DDS는 ROS 2 이전에도 다음과 같은 분야에서 널리 사용되었다.

- 자율주행 차량
- 항공 및 방위 시스템
- 산업 자동화 및 SCADA
- 의료 장비
- 금융 데이터 분산 시스템

---

## 7.3 DDS의 핵심 구조

### 7.3.1 도메인(Domain)

DDS에서 도메인은 독립적인 통신 영역이다.  
같은 도메인 내의 참여자(Participant)끼리만 데이터를 주고받을 수 있다.  
ROS 2에서 `ROS_DOMAIN_ID` 환경변수가 이 도메인 ID를 설정한다.

```bash
export ROS_DOMAIN_ID=0   # 기본값
export ROS_DOMAIN_ID=10  # 다른 로봇 그룹 예시
```

### 7.3.2 도메인 참여자(Domain Participant)

도메인에 참여하는 엔티티이다.  
ROS 2에서는 각 노드가 내부적으로 도메인 참여자를 가진다.

### 7.3.3 퍼블리셔와 서브스크라이버(Publisher / Subscriber)

DDS에서 데이터를 전송하는 측을 Publisher, 수신하는 측을 Subscriber라고 한다.  
ROS 2의 토픽 퍼블리셔와 서브스크라이버가 DDS의 이 개념에 직접 대응한다.

### 7.3.4 토픽(Topic)

DDS에서 토픽은 데이터를 주고받는 채널이다.  
이름과 데이터 타입으로 구성된다.  
ROS 2의 토픽 이름이 DDS 토픽 이름으로 매핑된다.

### 7.3.5 데이터라이터와 데이터리더(DataWriter / DataReader)

퍼블리셔 내부의 실제 데이터 전송 객체를 DataWriter,  
서브스크라이버 내부의 실제 데이터 수신 객체를 DataReader라고 한다.

### 전체 구조 도식

```
Domain
├── Participant A (Node: camera_node)
│   └── Publisher
│       └── DataWriter → Topic: /camera/image_raw
│
└── Participant B (Node: vision_node)
    └── Subscriber
        └── DataReader ← Topic: /camera/image_raw
```

---

## 7.4 왜 ROS 2는 DDS를 선택했는가?

ROS 2 설계 초기에 통신 미들웨어 후보로 여러 기술이 검토되었다.  
최종적으로 DDS가 선택된 이유는 다음과 같다.

### 7.4.1 이미 검증된 산업 표준

DDS는 자율주행, 방위, 항공 등 신뢰성이 중요한 산업 분야에서 오랫동안 사용되어 왔다.  
새로운 통신 프로토콜을 직접 개발하지 않고도 신뢰성 있는 통신 기반을 확보할 수 있었다.

### 7.4.2 QoS 지원

DDS는 처음부터 QoS를 지원하도록 설계되었다.  
이를 통해 ROS 2도 별도 구현 없이 다양한 통신 품질 정책을 활용할 수 있게 되었다.

### 7.4.3 분산 발견 (Discovery)

DDS는 중앙 서버 없이 네트워크에서 참여자를 자동으로 발견하는 기능을 기본으로 제공한다.  
ROS 1의 rosmaster를 없애고 분산 구조를 실현하기 위한 핵심 기능이었다.

### 7.4.4 벤더 독립성

DDS는 표준이기 때문에, 여러 벤더의 구현체를 선택적으로 사용할 수 있다.  
즉, ROS 2 사용자는 자신의 환경에 맞는 DDS 구현체를 선택할 수 있다.

---

## 7.5 주요 DDS 구현체

ROS 2에서 사용 가능한 주요 DDS 구현체는 다음과 같다.

| 구현체              | 개발사           | 특징                                      |
|---------------------|------------------|-------------------------------------------|
| Fast DDS (eProsima) | eProsima         | ROS 2 기본 구현체, 오픈소스               |
| CycloneDDS          | Eclipse Foundation | 높은 성능, ROS 2에서 널리 사용           |
| OpenDDS             | Object Computing | 오래된 역사, 항공/방위 분야에서 사용      |
| RTI Connext DDS     | RTI              | 상용 제품, 고성능·고신뢰성 환경에 적합    |

### DDS 구현체 변경 방법

```bash
# CycloneDDS 사용 설정
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

# Fast DDS 사용 설정
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

---

## 7.6 RMW (ROS Middleware)

ROS 2는 DDS를 직접 호출하지 않고, **RMW(ROS Middleware Interface)** 라는 추상화 레이어를 통해 DDS와 통신한다.

### 계층 구조

```
ROS 2 응용 코드 (Node, Topic, Service 등)
        ↕
   rclcpp / rclpy  (ROS Client Library)
        ↕
   RMW Interface   (추상화 레이어)
        ↕
DDS 구현체 (Fast DDS, CycloneDDS, OpenDDS 등)
        ↕
   실제 네트워크 통신
```

### RMW의 역할

- ROS 2 코드가 특정 DDS 구현체에 의존하지 않도록 추상화
- DDS 구현체를 교체해도 ROS 2 소스 코드를 수정하지 않아도 된다
- 다양한 환경과 요구사항에 맞는 DDS 구현체를 유연하게 선택할 수 있다

---

## 7.7 RTPS (Real-Time Publish-Subscribe Protocol)

DDS는 내부적으로 **RTPS(Real-Time Publish-Subscribe)** 프로토콜 위에서 동작한다.  
RTPS는 OMG 표준으로, UDP 기반의 신뢰성 있는 데이터 전송을 지원한다.

### RTPS의 특징

- UDP 기반으로 낮은 오버헤드
- 메시지 신뢰성 보장 메커니즘 내장
- 멀티캐스트 지원으로 네트워크 효율성 향상
- 참여자 발견(Discovery) 프로토콜 포함

---

## 7.8 DDS와 네트워크 발견 과정

ROS 2 노드가 시작되면 DDS의 발견 과정을 통해 다른 노드를 찾는다.

### 발견 과정 단계

```
① 노드 시작
② DDS Participant 생성
③ Simple Participant Discovery Protocol (SPDP) 브로드캐스트
④ 다른 참여자 응답 수신
⑤ Simple Endpoint Discovery Protocol (SEDP) 로 토픽/서비스 정보 교환
⑥ 같은 토픽을 가진 Publisher-Subscriber 간 통신 연결 수립
⑦ 데이터 교환 시작
```

이 과정이 자동으로 이루어지기 때문에 ROS 1에서 필요했던 `rosmaster` 없이도 노드들이 서로를 발견하고 통신할 수 있다.

---

## 7.9 DDS 사용 시 고려사항

DDS를 기반으로 한 ROS 2 사용 시 주의해야 할 사항들이 있다.

### 네트워크 구성

- 같은 서브넷 내에서는 멀티캐스트로 자동 발견
- 다른 서브넷 간에는 Discovery Server 또는 Static Discovery 설정 필요

### 방화벽 설정

- DDS는 UDP 포트를 사용하므로, 방화벽에서 해당 포트를 허용해야 한다
- 기본적으로 7400 포트 대역을 사용

### 도메인 ID 선택

- 0~232 사이의 값 사용 가능
- 같은 네트워크에서 다른 그룹의 로봇과 간섭이 없도록 도메인 ID 분리 권장

---

## 7.10 정리

ROS 2와 DDS의 관계를 요약하면 다음과 같다.

| 항목             | 설명                                                       |
|------------------|------------------------------------------------------------|
| DDS 역할         | ROS 2의 통신 미들웨어 (메시지 전달, 발견, QoS 처리)        |
| RMW 역할         | ROS 2와 DDS 구현체 사이의 추상화 레이어                   |
| 선택 이유        | 산업 표준, QoS 지원, 분산 발견, 벤더 독립성               |
| 구현체 선택      | Fast DDS, CycloneDDS 등 환경에 따라 선택 가능              |

DDS의 도입은 ROS 2가 연구용 프레임워크를 넘어서,  
실제 산업 현장에서 사용할 수 있는 안정적이고 유연한 로봇 미들웨어로 성장하는 데 핵심적인 역할을 했다.
