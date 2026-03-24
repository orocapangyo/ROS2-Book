# 7장 ROS 2와 DDS

## 7.1 DDS 도입 배경

ROS 1은 자체 개발한 **TCPROS** 통신 라이브러리를 사용했다.
ROS 2는 OMG(Object Management Group)가 표준화한 **DDS(Data Distribution Service)** 와 그 프로토콜인 **DDSI-RTPS** 를 미들웨어로 채택했다.

DDS 도입으로 얻은 핵심 효과:
- ROS Master 제거 → 동적 검색(Dynamic Discovery)으로 대체
- QoS 설정으로 신뢰성/속도 조절 가능
- DDS-Security 적용으로 보안 강화
- 상업용 제품에 사용 가능한 산업 표준 기반 확보

---

## 7.2 DDS란?

> DDS는 데이터 분산 서비스(Data Distribution Service)의 약자로, OMG가 표준화한 미들웨어 프로토콜 및 API이다.
> 저지연 데이터 연결, 높은 신뢰성, 확장 가능한 아키텍처를 제공한다.

- OSI 7계층 중 **4~7계층(호스트 계층)** 에 해당
- 운영체제와 사용자 애플리케이션 사이에 위치하는 소프트웨어 계층

---

## 7.3 DDS의 10가지 특징

| # | 특징 | 설명 |
|---|---|---|
| 1 | 산업 표준 | OMG 관리, OpenFMB·AUTOSAR·ROS 2 등에 사용되는 국제 표준 |
| 2 | OS 독립 | Linux, Windows, macOS, Android, VxWorks 등 지원 |
| 3 | 언어 독립 | RMW 추상화로 C++, Python, Java 등 다양한 언어 지원 |
| 4 | UDP 기반 전송 | UDP 멀티캐스트 기반, `ROS_DOMAIN_ID`로 도메인 그룹 분리 |
| 5 | 데이터 중심 (DCPS) | 어떤 데이터를, 어떤 형식으로, 어떻게 보낼지를 미들웨어가 관리 |
| 6 | 동적 검색 | IP/포트 사전 설정 없이 노드 자동 탐색 및 연결 |
| 7 | 확장 가능한 아키텍처 | IoT 소형 디바이스부터 국방·항공·우주 대형 시스템까지 확장 가능 |
| 8 | 상호 운용성 | 서로 다른 DDS 벤더 제품 간 혼용 통신 가능 |
| 9 | QoS | 통신 품질을 22가지 항목으로 세밀하게 설정 가능 |
| 10 | 보안 | DDS-Security 사양 + SROS 2 툴킷 제공 |

### 지원 DDS 벤더 (ROS 2 기준)

| 제품명 | 벤더 | 비고 |
|---|---|---|
| Fast DDS | Eprosima | 오픈소스 |
| Cyclone DDS | Eclipse Foundation | 오픈소스 |
| Connext DDS | RTI | 상용 |
| OpenSplice | ADLINK | 상용 |
| Gurum DDS | Gurum Network | 상용, 국산 |

---

## 7.4 QoS 주요 설정 항목

| 항목 | 설명 |
|---|---|
| Reliability | `RELIABLE` (TCP처럼 손실 방지) / `BEST_EFFORT` (UDP처럼 속도 우선) |
| History | 지정 크기만큼 데이터 보관 |
| Durability | 수신자 생성 전 데이터를 유지할지 폐기할지 설정 |
| Deadline | 지정 주기 내 데이터 미수신 시 이벤트 함수 실행 |
| Lifespan | 지정 주기 내 수신된 데이터만 유효 처리 |
| Liveliness | 지정 주기 내 노드/토픽 생존 여부 확인 |

---

## 7.5 실습 예제

### 기본 퍼블리셔/서브스크라이버 실행

```bash
# 터미널 1
ros2 run demo_nodes_cpp listener

# 터미널 2
ros2 run demo_nodes_cpp talker

# 노드/토픽 관계 시각화
rqt_graph
```

### RMW 변경

```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
ros2 run demo_nodes_cpp talker
```

사용 가능한 RMW: `rmw_fastrtps_cpp`, `rmw_cyclonedds_cpp`, `rmw_connext_cpp`, `rmw_gurumdds_cpp`, `rmw_opensplice_cpp`

### 상호 운용성 테스트 (서로 다른 RMW)

```bash
# 터미널 1 — Cyclone DDS
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
ros2 run demo_nodes_cpp listener

# 터미널 2 — Fast DDS
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
ros2 run demo_nodes_cpp talker
# → 서로 다른 RMW 간에도 통신 성공
```

### Domain 변경 (네트워크 분리)

```bash
export ROS_DOMAIN_ID=11
ros2 run demo_nodes_cpp talker

export ROS_DOMAIN_ID=12
ros2 run demo_nodes_cpp listener   # 수신 없음 (다른 도메인)

export ROS_DOMAIN_ID=11
ros2 run demo_nodes_cpp listener   # 수신 성공 (같은 도메인)
```

### QoS Reliability 테스트

```bash
# 10% 패킷 손실 설정
sudo tc qdisc add dev lo root netem loss 10%

# RELIABLE (기본): 손실 있어도 재전송하여 누락 없음
ros2 run demo_nodes_cpp listener

# BEST_EFFORT: 손실된 메시지는 그냥 누락됨
ros2 run demo_nodes_cpp listener_best_effort

# 테스트 후 반드시 해제
sudo tc qdisc delete dev lo root netem loss 10%
```

---

# 정리

| 항목 | ROS 1 | ROS 2 |
|---|---|---|
| 통신 방식 | TCPROS (자체 개발) | DDS / DDSI-RTPS (산업 표준) |
| 노드 탐색 | ROS Master | DDS Dynamic Discovery |
| 통신 품질 제어 | 불가 | QoS 22가지 설정 |
| 보안 | SROS (취약) | DDS-Security + SROS 2 |
| 벤더 선택 | 없음 | RMW로 5개 벤더 중 선택 가능 |

DDS 채택은 ROS 2의 가장 큰 변화로, 상업용 로봇 개발의 발판을 마련했다는 평가가 지배적이다.
