# 4장 왜? ROS 2로 가야하는가?

## 4.1 ROS 1에서 ROS 2로

ROS 1을 잘 쓰고 있는 기존 사용자라면 ROS 2로의 전환을 망설일 수 있다.
하지만 ROS 2는 2014년 개발 시작 이후 7년간의 개발과 6개의 공식 배포판을 거쳐 이미 주류가 되고 있다.

**ROS 2 배포 이력**

| 연도 | 내용 |
|---|---|
| 2014 | 첫 개발 시작 (ROSCon 2014 공개 발표) |
| 2015 | 알파 버전 릴리즈 (총 8회) |
| 2016~2017 | 베타 버전 작업 |
| 2017.12 | 첫 공식 배포판 Ardent Apalone |
| 2020.05 | Foxy Fitzroy (6번째 공식 배포판) |

> ROS 1을 사용 중이라면 지금 ROS 2로 전환해야 한다.
> 처음 시작하는 경우라면 ROS 2로 바로 시작하면 된다.

---

## 4.2 대세는 이제 ROS 2

### ROS 1 마지막 버전 선언

2020년 5월 23일, ROS 1의 13번째 버전인 **Noetic Ninjemys**가 릴리즈되었고,
이와 함께 Open Robotics와 ROS 커뮤니티는 이것이 **마지막 ROS 1 배포판**임을 공식 선언했다.

- EOL: 2025년
- Kinetic 이후 ROS 1 핵심 코드 개발은 이미 중단된 상태 — 이후는 단순 유지 보수

### 개발자 사용률 (ROSDISTRO 커밋 기준)

2018년 이후 ROS 2 관련 커밋 수가 지속적으로 증가하는 추세.
ROS 개발자 사이에서 ROS 2의 비중이 빠르게 높아지고 있다.

- 참고: https://metrics.ros.org/rosdistro_rosdistro.html

### 사용자 사용률 (패키지 다운로드 기준)

2019년부터 사용이 본격화되어 약 20%대까지 성장.
ROS 1의 Kinetic Kame와 비슷한 수준에 도달했다.

- 참고: https://metrics.ros.org/packages_rosdistro.html

---

## 4.3 ROS 2 관심도

### 커뮤니티 (ROSCon)

ROSCon은 매년 600여 명의 ROS 개발자가 모이는 연례 컨퍼런스다.

| ROSCon | ROS 2 관련 발표 수 |
|---|---|
| 2014 | 2건 |
| 2019 | 17건 (전체 발표의 50% 이상) |

> 앞으로의 ROSCon에서 ROS 2 비중은 계속 증가할 것으로 보인다.

- ROSCon 자료: https://index.ros.org/doc/ros2/ROSCon-Content/

### 기업 (ROS 2 TSC)

ROS 2는 기업들이 단순 관심을 넘어 직접 개발에 참여하는 수준으로 발전했다.
**ROS 2 TSC (Technical Steering Committee)**에는 현재 17개 기업 및 단체가 참여하고 있다.

- 참고: https://index.ros.org/doc/ros2/Governance/

---

# 정리

"왜 ROS 2인가?"에 대한 답은 단순히 최신 기술이기 때문이 아니다.

- ROS 1은 공식적으로 마지막 버전이 선언되었다.
- 개발자, 사용자, 기업 모두 ROS 2 중심으로 빠르게 이동하고 있다.
- ROS 2는 실제 산업 및 서비스 환경에 필요한 구조적 요구를 충족한다.
- 처음 시작하는 경우라면 고민 없이 ROS 2로 시작하면 된다.