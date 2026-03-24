# 3장. ROS2 개발환경 구축

## 1. ROS2 개발 환경 개요

ROS2 프로그래밍을 위해서는 **검증된 로봇 소프트웨어 개발 스택**을 준비하는 것이 중요합니다.  
운영체제, ROS2 배포판, 빌드 도구, 에디터/IDE 등이 조합되어 하나의 개발 환경을 이룹니다.

일반적으로 다음 조합이 많이 사용됩니다.

- **운영체제**: Ubuntu LTS (예: 22.04, 24.04)
- **ROS2 배포판**: Humble, Iron, Jazzy 등
- **언어**: C++, Python
- **도구**: `colcon`, `git`, `VS Code` 또는 `CLion`, `rviz`, `Gazebo` 등

각각의 ROS2 버전이 어떤 Ubuntu 버전을 공식 지원하는지는 **공식 문서**에서 확인할 수 있습니다.

- 참고:  
  - `https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html`
  - 개인 블로그 정리 예시: `https://swbee.tistory.com/27`

ROS2를 처음 설치할 때는, **현재 사용하는 Ubuntu 버전에 맞는 ROS2 배포판**을 선택하는 것이 가장 중요합니다.

---

## 2. ROS2 빌드 시스템

ROS2는 ROS1의 `catkin`을 대체하여, 새로운 표준 빌드 시스템과 통합 빌드 도구를 사용합니다.

### 1) 빌드 도구 개요

ROS2에서 주로 사용하는 빌드 관련 도구는 다음과 같습니다.

#### ament

- CMake 기반의 C++ 빌드와 Python `setuptools`를 모두 지원하는 **메타 빌드 시스템**
- ROS2 패키지에서 공통적으로 사용하는 빌드 규칙과 헬퍼 스크립트를 제공
- `ament_cmake`, `ament_python` 형태로 언어별 빌드 타입을 구분

#### colcon

- 여러 패키지를 한 번에 병렬로 빌드하고 테스트할 수 있는 **범용 CLI 빌드 도구**
- ROS2 공식 튜토리얼에서도 기본 빌드 도구로 사용
- 주요 기능
  - 패키지 간 의존성 분석 후 **올바른 순서로 빌드**
  - 병렬 빌드를 통한 빌드 시간 단축
  - 빌드, 테스트, 설치 디렉토리를 자동으로 구조화

예: 워크스페이스에서 전체 패키지 빌드

```bash
colcon build --symlink-install
```

---

## 3. ROS2 워크스페이스 구조

ROS2는 효율적인 관리를 위해 **정형화된 디렉토리 구조**를 사용하는 것을 권장합니다.  
일반적인 워크스페이스 예시는 다음과 같습니다.

```text
robot_ws/        # Root workspace
  ├── src/       # 사용자 소스 코드 및 패키지
  ├── build/     # 중간 빌드 결과물
  ├── install/   # 최종 빌드 결과 (실행 파일, 라이브러리, 설정)
  └── log/       # 빌드 및 실행 관련 로그
```

### 디렉토리별 역할

- **`robot_ws/`**
  - ROS2 개발을 위한 작업 공간(Workspace)의 루트 디렉토리
  - 이름은 자유롭게 정할 수 있지만, 관례적으로 `*_ws`를 많이 사용 (`ros2_ws`, `dev_ws` 등)

- **`src/`**
  - 직접 만든 ROS2 패키지와 외부에서 가져온 패키지를 배치하는 곳
  - Git 서브모듈이나 `vcs`를 사용해 여러 패키지를 한 번에 관리하기도 함

- **`build/`**
  - `colcon build` 수행 시 생성되는 **중간 빌드 파일**들이 저장되는 디렉토리
  - CMake 캐시, 오브젝트 파일 등 사용자 입장에서는 직접 건드릴 일이 거의 없음

- **`install/`**
  - 실제로 실행 가능한 바이너리, 라이브러리, 설정 파일 등이 설치되는 위치
  - 워크스페이스를 사용하기 위해서는

    ```bash
    source install/setup.bash
    ```

    처럼 `setup` 스크립트를 소스(sourcing)해야 함

- **`log/`**
  - 빌드 로그, 테스트 로그, 실행 중 출력 로그 등 기록
  - 빌드 실패 원인 분석이나 런타임 에러 분석 시 유용

---

## 4. 개발 편의 설정(run commands)

실제 개발할 때는 매번 터미널을 열고 아래 명령을 반복하게 됩니다.

```bash
source /opt/ros/<배포판이름>/setup.bash
source ~/robot_ws/install/setup.bash
```

이 과정을 단축하기 위해, 쉘 설정 파일(`~/.bashrc`, `~/.zshrc` 등)에 **run commands**를 미리 등록해 두는 것을 권장합니다.

예시 (zsh 기준):

```bash
# ROS2 기본 환경 설정
source /opt/ros/jazzy/setup.zsh  # 배포판에 맞게 수정

# 내 워크스페이스 환경 설정
alias src_robot_ws='source ~/robot_ws/install/setup.zsh'
```

이렇게 설정해 두면, 터미널에서

```bash
src_robot_ws
```

와 같이 짧은 명령만으로 워크스페이스 환경을 준비할 수 있어,  
**개발 속도가 크게 향상되고 실수도 줄어듭니다.**

