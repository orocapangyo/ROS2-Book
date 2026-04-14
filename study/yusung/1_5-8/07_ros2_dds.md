# 7장 ROS2 와 DDS

---
## 7.1. ROS2 와 DDS
![alt text](p089-Ros1vsRos2.png)
- ROS 1은 Master 중심의 발견 구조, ROS 2는 DDS로 분산 Discovery + QoS 기반 통신 설계가 가능해지며, 이 변화가 멀티로봇·제품화·운영 안정성에 직접적인 이점을 만든 구조이다.
- ROS 2는 DDS를 사용해 Master 없이 노드들이 자동으로 서로를 찾는(Discovery) 분산 구조이다.
- 통신은 DDS가 제공하는 데이터 버스(DataBus) 개념 위에서 이루어지며,
QoS로 신뢰성/지연/버퍼/지속성 같은 전달 특성을 설계할 수 있는 구조이다
- ROS 2는 로봇 앱 구조(노드/메시지)를 제공하고, DDS는 그 메시지를 안정적으로 전달하는 **통신 엔진** 역할이다.
---
## 7.2. DDS란?
![alt text](p091-ROS2미들웨어.png)
- DDS(Data Distribution Service) 는 로봇/산업용 분산 시스템에서 많이 쓰는 데이터 중심(Pub/Sub) 통신 미들웨어 표준이다.

- Discovery(동적 검색): Master 없이도 노드가 서로 자동 발견함
- QoS(서비스 품질): 신뢰성/지연/버퍼/지속성 같은 전달 특성을 상황에 맞게 설계함
- (옵션) 보안(DDS-Security): 인증/암호화/접근 제어 지원 가능
- ROS 2 구조에서는 rclcpp/rclpy → rcl → rmw → DDS 구현체(Cyclone/Fast/Connext…) 형태로 연결되어, DDS 구현체를 교체(RMW)할 수 있는 구조이다.
---
## 7.3. DDS의 특징
### 1) Industry Standards  
- DDS는 산업 현장에서 검증된 **표준 통신 규격(OMG 표준)**이라는 뜻이다.
### 2) OS Independent  
- 특정 운영체제에 묶이지 않아 **Linux/Windows/macOS/RTOS 등에서 동일한 방식으로** 쓸 수 있다는 뜻이다.
### 3) Language Independent  
- 특정 언어에 종속되지 않고 **여러 언어(C/C++/Python 등)에서 동일한 데이터 구조로** 통신할 수 있다는 뜻이다.
![alt text](p93-RMW독립성.png)
### 4) Transport on UDP/IP  
- 보통 **UDP/IP(주로 RTPS over UDP)** 기반으로 빠르게 데이터를 전달한다는 뜻이다.
### 5) Data Centricity  
- “패킷”이 아니라 **데이터(토픽/상태)** 중심으로 설계되어, 데이터를 어떻게 공유할지에 초점이 있다는 뜻이다.
![alt text](p95-데이터중심성의도식화.png)
### 6) Dynamic Discovery  
- 중앙 서버 없이도 **노드들이 네트워크에서 자동으로 서로를 찾는** 기능이 있다는 뜻이다.
### 7) Scalable Architecture  
- 노드/토픽/장치가 늘어나도 **확장 가능하도록** 설계된 구조라는 뜻이다.
### 8) Interoperability  
- 표준 기반이라 서로 다른 벤더 구현체라도 **같이 동작(상호운용)할 수 있게** 만든다는 뜻이다.
### 9) Quality of Service  
- 신뢰성/지연/버퍼/지속성 등을 **QoS로 조절**해 통신 품질을 설계할 수 있다는 뜻이다.
### 10) Security  
- 인증/암호화/접근제어 같은 **보안 기능(DDS-Security)**을 적용할 수 있다는 뜻이다.
---
## 7.4. ROS에서의 사용법
### 1) 기본 퍼블리셔/서브스크라이버 노드 실행
- 데모 노드(talker/listener) 실행이다
- 터미널 A(퍼블리셔)이다.
```bash
ros2 run demo_nodes_cpp talker
```
- 터미널 B(서브스크라이버)이다.
```bash
ros2 run demo_nodes_cpp listener
```
![alt text](p98-Topic.png)
- 토픽 확인
```bash
ros2 topic list
ros2 topic echo /chatter
```
```bash
rqt_graph
```
---
### 2) RMW 변경 방법
- 2.2 RMW 설정(세션 적용) 과 Cyclone DDS로 설정 예시
```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
```
- **터미널을 새로 열거나, 같은 터미널에서 export 후 노드를 재실행**하는 방식이다.
![alt text](p100-rmw변경방법.png)

---
### 3) RMW 상호 운용성 테스트(Interoperability Test)
- ROS 2는 RMW 계층을 통해 DDS 구현체를 교체하는 구조이다. 상호운용성 테스트는 **서로 다른 RMW 조합이 통신되는지** 또는 **동일 RMW로 일관되게 통신되는지** 를 확인하는 테스트이다. 통신이 되면 상호운용이 되는 조합이다.
- 통신이 안 되면 “RMW/DDS 구현체 간 상호운용 제한” 또는 “네트워크/방화벽/설정” 문제를 의심하는 흐름이다.
- 실제 상호운용성은 DDS 구현체/버전/설정에 따라 달라질 수 있어 프로젝트별 검증이 필요하다.
```bash
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
ros2 run demo_nodes_cpp listener
```
```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
ros2 run demo_nodes_cpp talker
```
![alt text](p100-rmw상호운영성.png)

---
### 4) Domain 변경 방법
- `ROS_DOMAIN_ID`는 DDS Domain을 설정해 통신 그룹을 논리적으로 분리하는 기능이다.
- 서로 다른 Domain ID는 같은 네트워크여도 서로를 발견하지 못하는 것이 정상이라는 전제이다.
- Linux/macOS 설정 예시
```bash
export ROS_DOMAIN_ID=10
```
- Windows 설정 예시
```bat
setx ROS_DOMAIN_ID 10
```
![alt text](p101-Domain변경.png)

---
### 5) QoS 테스트(기본)
- 통신 검증은 **(talker/listener 실행) → (Domain/RMW 확인) → (RMW 조합 테스트) → (QoS 확인/실험)** 순서로 진행하면 대부분의 이슈를 빠르게 분리할 수 있는 구조이다.
- **Reliability, RELIABLE vs BEST_EFFORT**
- RELIBLE : 모든 메시지가 반드시 전달되도록 보장함. 손실된 메시지는 재전송됨
- 터미널 1
- 불안정한 네트워크 환경을 위해 로컬 네트워크 인터페이스에 10% 패킷 손실을 인위적으로 추가
```bash
$ sudo tc qdisc delete dev lo root netem loss 10%
$ ros2 run demo_nodes_cpp talker
```
- 터미널 2
```bash
$  ros2 run demo_nodes_cpp listener_best_effor
```
![alt text](p102-Reliable.png)

- BEST_EFFORT : 메시지 전달을 보장하지 않지만, 더 낮은 지연시간 제공
![alt text](p103-Effort.png)

---