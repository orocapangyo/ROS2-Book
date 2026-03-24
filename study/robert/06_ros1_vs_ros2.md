# 6장 ROS 1과 2의 차이점으로 알아보는 ROS 2의 특징

## 6.1 개요

ROS 2는 ROS 1의 단순한 버전 업이 아니다.  
설계 철학부터 내부 통신 구조, 지원 플랫폼, 보안 모델까지 근본적으로 재설계된 프레임워크이다.  
이 장에서는 ROS 1과 ROS 2를 비교하면서 ROS 2가 어떤 방향으로 발전했는지를 살펴본다.

---

## 6.2 전체 비교 요약

| 항목               | ROS 1                        | ROS 2                              |
|--------------------|------------------------------|------------------------------------|
| 통신 미들웨어      | 자체 구현 (TCPROS/UDPROS)   | DDS (Data Distribution Service)    |
| 마스터 노드        | 필요 (rosmaster)             | 불필요 (분산 발견 방식)             |
| 실시간성 지원      | 제한적                       | 지원 (rclc 등 활용)                |
| 운영체제 지원      | Linux 중심                   | Linux, Windows, macOS 지원         |
| 보안 기능          | 없음                         | DDS-Security 기반 보안 지원        |
| 빌드 시스템        | catkin                       | ament (ament_cmake, ament_python)  |
| Python 버전        | Python 2 (구버전)            | Python 3                           |
| 멀티 로봇 지원     | 불편 (네임스페이스 설정 필요) | 더 유연하게 지원                   |
| QoS 지원           | 없음                         | 지원 (신뢰성, 내구성 등)           |
| 런치 파일 형식     | XML                          | Python                             |
| 생명주기(Lifecycle) | 없음                        | 지원 (Managed Node Life Cycle)     |

---

## 6.3 마스터 노드의 제거

### ROS 1: 중앙 마스터 필요

ROS 1에서는 `roscore`를 실행하여 ROS Master를 구동시켜야 했다.  
모든 노드는 Master에 등록되고, Master를 통해 다른 노드를 찾아 통신을 연결했다.

```
[문제점]
- Master가 꺼지면 전체 시스템이 중단된다
- 분산 환경에서 네트워크 설정이 복잡하다
- 단일 장애 지점(Single Point of Failure)이 된다
```

### ROS 2: 마스터 없는 분산 발견

ROS 2는 DDS의 **Discovery 프로토콜**을 활용한다.  
노드들은 네트워크에 브로드캐스트하여 서로를 자동으로 발견하고 직접 연결한다.

```
[장점]
- 중앙 서버 없이도 통신 가능
- 일부 노드가 꺼져도 나머지 시스템은 정상 동작
- 멀티 로봇 환경에서 확장성이 높다
```

---

## 6.4 통신 미들웨어: DDS 도입

### ROS 1: 자체 구현 통신

ROS 1은 TCPROS, UDPROS라는 자체 구현 통신 레이어를 사용했다.  
이는 연구 목적에는 충분했지만, 산업 환경의 기준을 충족하기에는 한계가 있었다.

### ROS 2: DDS 기반 표준 통신

ROS 2는 산업 표준인 **DDS(Data Distribution Service)** 를 통신 미들웨어로 채택했다.  
DDS는 분산 실시간 시스템에서 널리 검증된 프로토콜이다.

```
[DDS 채택의 효과]
- 네트워크 자동 발견
- QoS 정책 지원
- 보안 통신 (DTLS/TLS 기반)
- 다양한 DDS 구현체 교체 가능 (eProsima Fast DDS, CycloneDDS 등)
```

---

## 6.5 QoS (Quality of Service) 지원

ROS 1은 메시지 전달 방식에 대한 세밀한 제어 수단이 없었다.  
ROS 2에서는 토픽, 서비스, 액션 통신에 QoS 정책을 설정할 수 있다.

### 주요 QoS 항목

| QoS 항목     | 설명                                      |
|--------------|-------------------------------------------|
| Reliability  | 메시지의 전달 보장 여부 (RELIABLE / BEST_EFFORT) |
| Durability   | 구독자가 늦게 생겼을 때 과거 메시지 제공 여부 |
| History      | 메시지 보관 방식 (KEEP_LAST / KEEP_ALL)   |
| Depth        | 보관할 메시지 수                          |
| Lifespan     | 메시지의 유효 기간                        |

이를 통해 네트워크 상태나 센서 특성에 따라 통신 전략을 유연하게 설정할 수 있다.

---

## 6.6 실시간성 지원

### ROS 1: 실시간성 미지원

ROS 1은 기본적으로 실시간(real-time) 처리를 지원하지 않았다.  
정밀한 타이밍이 필요한 시스템에 적용하기 어려웠다.

### ROS 2: 실시간 처리 가능

ROS 2는 실시간 운영체제(RTOS)와 함께 사용할 수 있는 구조를 가지고 있다.  
특히 `rclc`(ROS 2 Client Library for C)는 마이크로컨트롤러 수준에서의 실시간 처리를 지원한다.

```
[적용 분야]
- 정밀 모터 제어
- 마이크로 ROS (micro-ROS)를 통한 임베디드 시스템 연동
- 안전이 중요한 산업 로봇 시스템
```

---

## 6.7 운영체제 지원 확대

### ROS 1: Linux 전용

ROS 1은 Ubuntu Linux 중심으로 개발되었다.  
Windows나 macOS에서는 사용이 매우 제한적이었다.

### ROS 2: 멀티 플랫폼 지원

ROS 2는 공식적으로 다음 플랫폼을 지원한다.

| 플랫폼  | 지원 여부 |
|---------|-----------|
| Ubuntu  | ✅ (주요 플랫폼) |
| Windows | ✅ 공식 지원 |
| macOS   | ✅ 공식 지원 |
| RTOS    | ✅ (micro-ROS) |

이는 팀 협업 환경에서 OS 선택의 유연성을 높여준다.

---

## 6.8 보안 (Security) 강화

### ROS 1: 보안 없음

ROS 1은 보안 개념이 거의 없었다.  
누구나 네트워크에 접근하면 노드의 메시지를 가로채거나 제어 명령을 전송할 수 있었다.

### ROS 2: DDS-Security 기반 보안

ROS 2는 **SROS2(Secure ROS 2)** 를 통해 보안 통신을 지원한다.

```
[지원 기능]
- 노드 인증 (Authentication)
- 메시지 암호화 (Encryption)
- 접근 제어 (Access Control)
- 로그 서명 및 무결성 검증
```

---

## 6.9 빌드 시스템 변화

### ROS 1: catkin

ROS 1은 catkin 빌드 시스템을 사용했다.  
CMake 기반이었으나 Python 2와 강하게 결합되어 있었다.

### ROS 2: ament + colcon

ROS 2는 `ament` 빌드 시스템과 `colcon` 빌드 도구를 사용한다.

| 구분              | ROS 1        | ROS 2               |
|-------------------|--------------|---------------------|
| 빌드 시스템       | catkin       | ament_cmake / ament_python |
| 빌드 도구         | catkin_make  | colcon build        |
| Python 버전       | Python 2     | Python 3            |

---

## 6.10 런치 파일 형식 변화

### ROS 1: XML 기반

```xml
<launch>
  <node pkg="my_pkg" type="my_node" name="node1" />
</launch>
```

### ROS 2: Python 기반

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(package='my_pkg', executable='my_node', name='node1'),
    ])
```

Python 기반 런치 파일은 조건문, 반복문, 함수 호출 등 프로그래밍 언어의 기능을 그대로 활용할 수 있어 훨씬 유연하다.

---

## 6.11 노드 생명주기 (Lifecycle Node)

ROS 1에는 노드의 상태 관리 개념이 없었다.  
ROS 2에서는 **Managed Node Lifecycle** 을 통해 노드의 상태를 명시적으로 관리할 수 있다.

### 생명주기 상태 도식

```
Unconfigured
    ↓ configure
Inactive
    ↓ activate
Active
    ↓ deactivate
Inactive
    ↓ cleanup
Unconfigured
    ↓ shutdown
Finalized
```

이 구조를 통해 노드의 초기화, 활성화, 비활성화, 종료를 명확하게 제어할 수 있어 안정적인 시스템 운영이 가능하다.

---

## 6.12 멀티 로봇 환경 지원

### ROS 1: 복잡한 네임스페이스 설정 필요

ROS 1은 멀티 로봇 환경을 구성하려면 복잡한 네임스페이스 및 네트워크 설정이 필요했다.

### ROS 2: 도메인 ID와 네임스페이스 활용

ROS 2는 **ROS_DOMAIN_ID** 환경변수를 통해 로봇 그룹을 쉽게 분리할 수 있다.  
또한 네임스페이스를 활용하여 멀티 로봇 시스템을 명확하게 구성할 수 있다.

```bash
# 로봇 A
export ROS_DOMAIN_ID=1

# 로봇 B
export ROS_DOMAIN_ID=2
```

두 로봇은 서로 다른 도메인으로 분리되어 토픽 충돌 없이 독립적으로 동작한다.

---

## 6.13 정리

ROS 1과 ROS 2의 차이점을 통해 ROS 2가 지향하는 방향을 정리하면 다음과 같다.

| 방향성        | 설명                                                   |
|---------------|--------------------------------------------------------|
| 분산 설계     | 중앙 마스터 없이 노드 간 직접 발견 및 통신             |
| 표준 기반     | 자체 구현 대신 DDS 산업 표준 채택                     |
| 실용성 강화   | 실시간성, 보안, 멀티 플랫폼, 생명주기 관리 등 현장 요구 반영 |
| 확장성 확보   | 멀티 로봇, 대규모 시스템, 임베디드 환경까지 지원       |

ROS 2는 연구 목적에 머물렀던 ROS 1을 넘어서, 실제 산업 현장에서도 사용할 수 있는 수준의 로봇 미들웨어로 설계된 것이다.
