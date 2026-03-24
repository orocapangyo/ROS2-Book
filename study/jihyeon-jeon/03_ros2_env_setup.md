# 3장 ROS 2 개발환경 구축

## 3.1 개발 환경 사양

이 강좌는 Linux Mint 20.x와 ROS 2 Foxy Fitzroy를 기반으로 한다.

| 구분 | 추천 | 선택 사항 |
|---|---|---|
| 기본 운영 체제 | Linux Mint 20.x | Ubuntu 20.04.x LTS (Focal Fossa) |
| 로봇 운영 체제 | ROS 2 Foxy Fitzroy | ROS 2 Rolling Ridley |
| 컴퓨터 아키텍처 | amd64 | amd64, arm64 |
| IDE | Visual Studio Code | QtCreator |
| 프로그래밍 언어 | Python 3 (3.8.0), C++ 14 | 최신 Python/C++ |
| 시뮬레이터 | Gazebo 11.x | Ignition Citadel |
| DDS | Fast DDS | Cyclone DDS |
| 기타 | CMake 3.16.3, Qt 5.12.5, OpenCV 4.2.0 | - |

---

## 3.2 기본 운영 체제 설치

Linux Mint 20.x - Cinnamon (64-bit)를 설치한다.
Linux Mint 20.x는 Ubuntu 20.04 LTS 기반의 배포판으로 2025년까지 기술 지원을 제공한다.

> **주의**: 리눅스 설치 시 기본 언어를 반드시 `English`로 설정한다.
> 한글로 설치하면 터미널 및 빌드 오류 메시지가 한글로 출력되어 구글링이 어려워진다.

> **참고**: 가상머신(VM)이나 Docker는 리소스 및 디바이스 연결 측면에서 권장하지 않는다.
> 처음 학습 시 리눅스 네이티브 환경을 사용하는 것이 가장 안정적이다.

---

## 3.3 ROS 2 설치

데비안 패키지를 이용한 설치 방법을 권장한다.

### 1) 지역 설정

```bash
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

### 2) 소스 설정

```bash
sudo apt update && sudo apt install curl gnupg2 lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu focal main" \
  | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

> **참고**: Linux Mint 20.x는 `lsb_release -cs`가 `ulyssa`를 반환하지만,
> Ubuntu 20.04 코드명인 `focal`을 명시적으로 사용해야 한다.

### 3) ROS 2 패키지 설치

```bash
sudo apt update
sudo apt install ros-foxy-desktop ros-foxy-rmw-fastrtps* ros-foxy-rmw-cyclonedds*
```

### 4) 설치 확인

두 터미널에서 talker/listener 노드를 실행하여 통신을 확인한다.

```bash
# 터미널 1
source /opt/ros/foxy/setup.bash
ros2 run demo_nodes_cpp talker

# 터미널 2
source /opt/ros/foxy/setup.bash
ros2 run demo_nodes_py listener
```

---

## 3.4 ROS 개발 툴 설치

```bash
sudo apt update && sudo apt install -y \
  build-essential cmake git libbullet-dev \
  python3-colcon-common-extensions python3-flake8 \
  python3-pip python3-pytest-cov python3-rosdep \
  python3-setuptools python3-vcstool wget

python3 -m pip install -U \
  argcomplete flake8-blind-except flake8-builtins \
  flake8-class-newline flake8-comprehensions flake8-deprecated \
  flake8-docstrings flake8-import-order flake8-quotes \
  pytest-repeat pytest-rerunfailures pytest

sudo apt install --no-install-recommends -y \
  libasio-dev libtinyxml2-dev libcunit1-dev
```

---

## 3.5 워크스페이스 빌드 테스트

```bash
source /opt/ros/foxy/setup.bash
mkdir -p ~/robot_ws/src
cd ~/robot_ws/
colcon build --symlink-install
```

빌드 완료 후 `src/` 외에 `build/`, `install/`, `log/` 폴더가 생성되면 정상이다.

---

## 3.6 bashrc 환경 변수 설정

`~/.bashrc` 파일 하단에 아래 설정을 추가하면 터미널을 열 때마다 자동으로 환경이 적용된다.

```bash
source /opt/ros/foxy/setup.bash
source ~/robot_ws/install/local_setup.bash

source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash
source /usr/share/vcstool-completion/vcs.bash
source /usr/share/colcon_cd/function/colcon_cd.sh
export _colcon_cd_root=~/robot_ws

export ROS_DOMAIN_ID=7
export ROS_NAMESPACE=robot1

export RMW_IMPLEMENTATION=rmw_fastrtps_cpp

export RCUTILS_CONSOLE_OUTPUT_FORMAT='[{severity}]: {message}'
export RCUTILS_COLORIZED_OUTPUT=1
export RCUTILS_LOGGING_USE_STDOUT=0
export RCUTILS_LOGGING_BUFFERED_STREAM=1

alias cw='cd ~/robot_ws'
alias cs='cd ~/robot_ws/src'
alias ccd='colcon_cd'
alias cb='cd ~/robot_ws && colcon build --symlink-install'
alias cbs='colcon build --symlink-install'
alias cbp='colcon build --symlink-install --packages-select'
alias cbu='colcon build --symlink-install --packages-up-to'
alias ct='colcon test'
alias ctp='colcon test --packages-select'
alias ctr='colcon test-result'
alias rt='ros2 topic list'
alias re='ros2 topic echo'
alias rn='ros2 node list'
alias killgazebo='killall -9 gazebo & killall -9 gzserver & killall -9 gzclient'
alias af='ament_flake8'
alias ac='ament_cpplint'
alias testpub='ros2 run demo_nodes_cpp talker'
alias testsub='ros2 run demo_nodes_cpp listener'
alias testpubimg='ros2 run image_tools cam2image'
alias testsubimg='ros2 run image_tools showimage'
```

---

## 3.7 통합 개발 환경 (IDE)

### Visual Studio Code (권장)

크로스 플랫폼 에디터로 amd64/arm64를 모두 지원하며, 확장 기능을 통해 ROS 2 개발에 최적화할 수 있다.

#### 필수 확장 목록

| 분류 | 이름 | 코드명 |
|---|---|---|
| C/C++ | C/C++ | ms-vscode.cpptools |
| C/C++ | CMake Tools | ms-vscode.cmake-tools |
| Python | Python | ms-python.python |
| ROS | ROS | ms-iot.vscode-ros |
| ROS | URDF | smilerobotics.urdf |
| ROS | Colcon Tasks | deitry.colcon-helper |
| 포맷 | XML Tools | dotjoshjohnson.xml |
| 포맷 | YAML | redhat.vscode-yaml |
| 포맷 | Markdown All in One | yzhang.markdown-all-in-one |

#### 주요 설정 파일

| 파일 | 경로 | 역할 |
|---|---|---|
| settings.json | ~/.config/Code/User/settings.json | 전역 에디터 설정, ROS distro 지정 |
| c_cpp_properties.json | ~/robot_ws/.vscode/ | include 경로, C++ 표준 설정 |
| tasks.json | ~/robot_ws/.vscode/ | colcon build/test/clean 단축키 등록 |
| launch.json | ~/robot_ws/.vscode/ | Python(debugpy), C++(GDB) 디버깅 설정 |

- 빌드: `Ctrl + Shift + b`
- 디버깅: `Ctrl + Shift + d` → 언어(rclcpp/rclpy)에 맞는 설정 선택 후 `F5`

### QtCreator (보조)

RQt Plugin 개발 시 UI 작성에 편리하다. `sudo apt install qtcreator`로 설치한다.
ROS-Industrial 컨소시엄의 ROS Qt Creator Plug-in(`qtcreator-ros`)을 사용하면 ROS 개발 환경 설정이 더 편리하다.

---

## 3.8 ROS 2 삭제

```bash
sudo apt remove ros-foxy-* && sudo apt autoremove
```

---

# 정리

ROS 2 개발환경 구축은 다음 단계로 이루어진다.

1. Linux Mint 20.x (또는 Ubuntu 20.04) 설치 — **English** 언어 설정 필수
2. 데비안 패키지로 ROS 2 Foxy 설치
3. 개발 도구 설치 (colcon, rosdep, vcstool 등)
4. 워크스페이스(`robot_ws`) 생성 및 빌드 테스트
5. `~/.bashrc`에 환경 변수 및 alias 등록
6. VSCode 및 확장 설치, `.vscode/` 설정 파일 구성

환경이 올바르게 갖춰지면 `testpub` / `testsub` alias로 즉시 통신 테스트가 가능하다.
