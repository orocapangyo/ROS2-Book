# 5장 ROS 2의 중요 컨셉과 특징

## 5.1 Why ROS 2 — 10가지 이유

2019년 5월 Open Robotics가 공개한 문서. ROS 2 TSC 창립 멤버들의 의견 수렴을 거쳐 작성된 공식 메시지다.

| # | 항목 | 핵심 내용 |
|---|---|---|
| 01 | Shorten time to market | 공통 도구 재사용으로 본질적인 로봇 개발에 집중 가능 |
| 02 | Designed for production | 산업 이해관계자 요구 기반 설계, 높은 신뢰성·안전성 |
| 03 | Multi-platform | Linux, Windows, macOS, 임베디드 OS 지원 |
| 04 | Multi-domain | 가정, 자동차, 산업, 수중, 우주 등 전 분야 적용 가능 |
| 05 | No vendor lock-in | 통신 추상화로 오픈소스/독점 솔루션 자유 선택 |
| 06 | Built on open standards | DDS, IDL, DDS-I RTPS 등 산업 표준 채택 |
| 07 | Permissive open source license | Apache 2.0 라이선스, 폭넓은 상업 활용 가능 |
| 08 | Global community | 수십만 개발자/사용자 생태계, 커뮤니티 주도 개발 |
| 09 | Industry support | ROS 2 TSC 기업들의 직접 기여 및 오픈소스 투자 |
| 10 | Interoperability with ROS 1 | ROS 1↔2 브리지 제공, 점진적 마이그레이션 가능 |

---

# 정리

### ROS 1과의 차이

| 항목 | ROS 1 | ROS 2 |
|---|---|---|
| 통신 구조 | ROS Master 중앙 집중 | DDS 기반 분산, Master 불필요 |
| 설계 목적 | 연구·교육 중심 | 생산 환경까지 고려 |
| 플랫폼 | Linux 중심 | Linux / Windows / macOS |
| 보안 | 기본 지원 부족 | 보안 기능 강화 |
| 표준 | 자체 통신 방식 | IDL, DDS 등 산업 표준 |
| 라이선스 | 패키지별 상이 | Apache 2.0 기본 |

ROS 2는 ROS 1의 단순 업그레이드가 아니라 **연구에서 생산까지** 전 과정을 다루는 현대적 플랫폼이다.
ROS 1을 사용 중이라면 브리지를 통해 전환할 수 있고, 처음 시작한다면 ROS 2로 바로 시작하면 된다.
