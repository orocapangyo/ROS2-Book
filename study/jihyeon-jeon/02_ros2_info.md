# 2장 ROS 2 기반 로봇 개발에 필요한 정보

## 2.1 커뮤니티 중심의 개발 문화

ROS는 Open Robotics를 중심으로 글로벌 커뮤니티가 함께 개발하는 오픈소스 플랫폼이다.
코드, 문서, 이슈 모두 공개되며 누구나 기여할 수 있다.

| 리소스 | 링크 |
|---|---|
| ROS 2 소스코드 | https://github.com/ros2 |
| Gazebo 소스코드 | https://github.com/osrf/gazebo |
| ROS 2 공식 문서 | https://index.ros.org/doc/ros2/ |
| ROS 1 위키 | http://wiki.ros.org/ |

처음 접하면 흩어진 코드와 문서로 인해 혼란스러울 수 있다.
이 장에서 소개하는 링크들을 기반으로 스스로 탐색하는 습관을 기르자.

---

## 2.2 자주 찾는 링크 BEST 10

| # | 링크 | 설명 |
|---|---|---|
| 01 | https://discourse.ros.org/ | ROS 커뮤니티 게시판 |
| 02 | https://index.ros.org/doc/ros2/ | ROS 2 공식 문서 |
| 03 | http://design.ros2.org/ | ROS 2 디자인 문서 |
| 04 | http://wiki.ros.org/ | ROS 1 위키 (참고용) |
| 05 | https://roscon.ros.org/ | ROSCon 발표 자료/영상 |
| 06 | https://github.com/ros2 | ROS 2 소스코드 저장소 |
| 07 | https://ros.org/reps/rep-0000.html | REPs (ROS Enhancement Proposals) |
| 08 | https://github.com/ros/rosdistro | ROS 배포판 저장소 |
| 09 | http://repo.ros2.org/status_page/ | ROS 2 패키지 배포 현황 |
| 10 | https://answers.ros.org/questions/ | ROS 질의응답 페이지 |

---

## 2.3 ROS 커뮤니티 게시판 (ROS Discourse)

> https://discourse.ros.org/

모든 ROS 정보의 근원지. 다음 내용이 매일 올라온다.

- 버전별 정규 릴리즈 소식
- TSC(Technical Steering Committee) 회의 내용
- 워킹 그룹 회의록
- 신규 패키지 및 프로젝트 소식
- 각국 ROS 이벤트 및 구인/구직 정보

**알림/구독 기능**을 활성화하면 메일로 최신 소식을 받을 수 있다.
ROS 2 개발을 시작한다면 가입과 구독은 필수다.

---

## 2.4 ROS 공식 문서

| 링크 | 설명 |
|---|---|
| https://index.ros.org/doc/ros2/ | ROS 2 공식 문서 모음 (핵심) |
| http://design.ros2.org/ | ROS 2 디자인 문서 — 개발 목적과 방향 이해에 필수 |
| http://wiki.ros.org/ | ROS 1 위키 — ROS 2 부족한 부분 보완용 |
| https://roscon.ros.org/ | ROSCon 2012~ 발표 자료 및 영상 (2014년 이후 ROS 2 내용 포함) |
| https://fkromer.github.io/awesome-ros2/ | Awesome ROS2 비공식 링크 모음 |

> ROS 2 디자인 문서([03])는 ROS 2 학습 전 반드시 읽을 것을 권장한다.
> ROS 2 공식 문서([02]) 안의 `ROSCon Content` 항목에서 ROS 2 관련 발표만 따로 찾아볼 수 있다.

---

## 2.5 ROS 2 소스코드 및 REP

### 소스코드 저장소

> https://github.com/ros2

ROS 2의 공개 소스코드. 단순 실행에 그치지 않고 소스를 직접 읽고 수정하는 습관이 실력 향상에 도움이 된다.
소스 수정이 필요하다면 바이너리 설치 대신 **소스 빌드 설치**를 권장한다.

### API 문서

| 언어 | 링크 |
|---|---|
| rclcpp (C++) | http://docs.ros2.org/foxy/api/rclcpp/index.html |
| rclpy (Python) | http://docs.ros2.org/foxy/api/rclpy/index.html |

### REPs (ROS Enhancement Proposals)

> https://ros.org/reps/rep-0000.html

ROS의 모든 기능 개발 제안이 담긴 문서. Python의 PEP와 유사한 역할이다.
"왜 이렇게 설계됐지?" 라는 의문이 생길 때 찾아보자.

| REP | 내용 |
|---|---|
| REP 3 | ROS 1 배포판의 타겟 OS 및 라이브러리 버전 |
| REP 103 | 표준 단위 및 좌표 변환 규칙 |
| REP 149 | package.xml 포맷 규정 |
| REP 2000 | ROS 2 배포판의 타겟 OS, DDS, 언어 버전 |
| REP 2005 | ROS 2 기본 패키지 리스트 |

---

## 2.6 ROS 2 패키지 배포 현황

| 링크 | 설명 |
|---|---|
| https://github.com/ros/rosdistro | ROS 배포판 저장소 (rosdep 포함) |
| http://repo.ros2.org/status_page/ | ROS 2 전체 패키지 배포 현황 |
| http://repo.ros2.org/status_page/ros_foxy_default.html | Foxy 패키지 배포 현황 |
| http://repositories.ros.org/status_page/ | ROS 1 패키지 배포 현황 |

- `rosdistro`의 `rosdep` 섹션은 제3의 의존성 패키지를 정리한 것으로, 프로젝트 의존성 문제 해결 시 반드시 확인해야 한다.
- 제품 개발 시에는 관련 패키지의 배포 현황과 버전을 주기적으로 체크한다.

---

## 2.7 ROS 질의응답

> https://answers.ros.org/questions/

54,000개 이상의 질문/답변이 쌓여 있다. 먼저 여기서 키워드 검색을 하면 대부분의 문제를 해결할 수 있다.
그래도 해결되지 않으면 Stack Overflow와 구글링을 활용한다.

---

## 2.8 기타 유용한 링크

| 링크 | 설명 |
|---|---|
| https://metrics.ros.org/ | ROS 통계 자료 (유저 수, 인기 패키지, 논문 인용 수 등) |
| https://planet.ros.org/ | ROS 커뮤니티 주요 소식 모음 |

---

# 정리

ROS 2를 잘 사용하기 위해서는 단순히 명령어를 외우는 것보다, **정보를 스스로 찾는 능력**이 더 중요하다.
아래 순서로 습관을 들이자.

1. **ROS Discourse** 구독 → 최신 동향 파악
2. **ROS 2 디자인 문서** 정독 → 설계 의도 이해
3. **ROS 2 공식 문서** 탐독 → 실제 개발 지식 확보
4. **GitHub 소스코드** 탐색 → 동작 원리 파악
5. **REPs** 참고 → "왜?" 에 대한 답
6. **answers.ros.org** 검색 → 문제 해결

다음 장에서는 이 강좌의 개발 환경인 ROS 2 Foxy와 Linux Mint 20.x 기반으로 실제 개발환경 구축 방법을 다룬다.
