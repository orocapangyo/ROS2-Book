1페이지. 제목

슬라이드 제목
ROS 2의 시간, 파일 시스템, 빌드, 패키지

슬라이드에 넣을 내용

분산 로봇 시스템에서 꼭 알아야 하는 4가지 기반 개념
시간 동기화 → 코드 구조 → 빌드 → 배포/재사용
예시 환경: Humble 기반 워크스페이스

발표 대본
“오늘 발표에서는 ROS 2를 처음 공부할 때 따로따로 보이던 네 가지 주제를 하나의 흐름으로 묶어서 보겠습니다. 시간은 데이터의 신뢰성과 연결되고, 파일 시스템은 코드가 어디에 있어야 하는지를 정해주고, 빌드는 의존성을 풀어 실행 가능한 형태로 만드는 과정이며, 패키지 파일은 이 모든 정보를 도구가 이해하도록 정리한 메타데이터입니다. 결국 이 네 가지를 같이 이해해야 ROS 2 프로젝트를 제대로 다룰 수 있습니다.”

2페이지. 전체 흐름

슬라이드 제목
왜 이 4개를 한 번에 이해해야 하는가

슬라이드에 넣을 내용

시간: 메시지가 “언제” 생성되었는가
파일 시스템: 패키지가 “어디” 있는가
빌드: 의존성을 “어떻게” 해석하는가
패키지 파일: 도구가 “무엇을” 알아야 하는가

발표 대본
“ROS 2는 여러 노드가 동시에 돌아가는 생태계입니다. 그래서 어떤 데이터가 언제 만들어졌는지 알아야 하고, 그 노드와 패키지가 파일 시스템 어디에 있는지도 알아야 하며, 여러 패키지의 의존성을 어떤 순서로 빌드할지도 정해야 합니다. 마지막으로 package.xml, CMakeLists.txt, setup.py 같은 파일이 있어야 이런 작업을 도구가 자동으로 처리할 수 있습니다.”

3페이지. ROS 2에서 시간이 중요한 이유

슬라이드 제목
왜 시간(timestamp)이 그렇게 중요한가

슬라이드에 넣을 내용

센서 데이터는 시간 순서가 맞아야 의미가 있음
여러 노드 간 통신에서 publish 시각이 중요
Header의 stamp, frame_id가 자주 함께 사용됨
예시: 카메라 30Hz, LiDAR 10Hz, IMU 200Hz

발표 대본
“로봇은 카메라, LiDAR, IMU처럼 주기가 다른 센서를 동시에 사용합니다. 이때 단순히 값만 맞는다고 되는 게 아니라, 그 값이 정확히 언제 측정된 것인지가 중요합니다. 예를 들어 IMU는 아주 빠르게 들어오고 LiDAR는 상대적으로 느리기 때문에, timestamp가 어긋나면 EKF나 sensor fusion 결과가 틀어질 수 있습니다. 그래서 ROS 2에서는 메시지 내용뿐 아니라 시간 정보도 매우 중요한 메타데이터로 다룹니다.”

4페이지. Clock, Time, Time Source

슬라이드 제목
Clock와 Time은 무엇이 다른가

슬라이드에 넣을 내용

Clock: 시간 소스에 접근하는 창구
Time: 특정 시점
Duration: 두 시점의 차이
now()는 현재 Clock 기준 시간 반환

발표 대본
“여기서 먼저 구분해야 할 것이 Clock과 Time입니다. Clock은 시간의 원천, 즉 어떤 시간을 기준으로 볼지 정하는 인터페이스이고, Time은 그 Clock에서 읽어온 특정 시점을 의미합니다. 그리고 Duration은 두 Time의 차이입니다. 발표에서는 ‘Clock은 시계, Time은 그 시계가 가리키는 한 순간’ 정도로 설명하면 듣는 분들이 이해하기 쉽습니다.”

5페이지. 시간의 추상화 3종

슬라이드 제목
System Time, ROS Time, Steady Time

슬라이드에 넣을 내용

System Time: 운영체제 시스템 시간
ROS Time: /clock 기반 시뮬레이션/재생 시간
Steady Time: 단조 증가, 타임아웃에 적합

발표 대본
“ROS 2는 시간을 하나로 고정하지 않고 세 가지로 추상화합니다. System Time은 우리가 평소 생각하는 운영체제 시간이고, ROS Time은 시뮬레이터나 rosbag 재생이 퍼블리시하는 /clock을 따라갑니다. Steady Time은 뒤로 가지 않는 단조 증가 시간이라서, 하드웨어 타임아웃이나 재시도 간격처럼 ‘절대 멈추면 안 되는 내부 시간’에 적합합니다. 발표에서는 ‘센서 메시지 timestamp는 ROS Time, 장치 timeout은 Steady Time’처럼 역할을 나눠주면 좋습니다.”

6페이지. use_sim_time과 /clock

슬라이드 제목
시뮬레이션 시간은 어떻게 동작하는가

슬라이드에 넣을 내용

use_sim_time=true이면 ROS Time 활성화
/clock이 publish되면 시스템 시간 대신 사용
/clock을 못 받으면 시간 0 가능
시간 점프/일시정지/빠른 재생 가능

발표 대본
“이 슬라이드는 실제 발표에서 꼭 강조해 주세요. use_sim_time를 켠 노드는 ROS Time을 사용하고, 그때 실제 시간의 기준은 /clock 토픽이 됩니다. 그래서 Gazebo나 bag playback을 멈추면 노드의 시간도 같이 멈출 수 있습니다. 반대로 /clock이 아직 오지 않은 상태라면 시간이 0으로 보일 수 있기 때문에, 시뮬레이션에서 ‘시간이 왜 안 가죠?’라는 질문이 자주 나옵니다.”

7페이지. Time API 개요

슬라이드 제목
ROS 2의 Time API: Time, Duration, Rate, Timer

슬라이드에 넣을 내용

Time: 현재 시각, 비교, 덧셈/뺄셈
Duration: 경과 시간, timeout 계산
Rate: 반복 주기 유지
Timer: ROS 2 스타일의 주기 콜백

발표 대본
“개발자가 가장 많이 쓰는 API는 Time, Duration, Rate입니다. 예를 들어 now - past는 경과 시간 계산이고, now + duration은 미래 시점 계산입니다. Rate는 while 루프를 일정 주기로 돌릴 때 직관적이고, 실제 ROS 2 노드에서는 Timer 콜백 구조를 쓰는 경우가 많습니다. 그래서 발표에서는 ‘Time은 시점, Duration은 간격, Timer는 실행 주기’로 정리하면 좋습니다.”

8페이지. 시간 예제 코드 해설

슬라이드 제목
예제 코드로 보는 시간 연산

슬라이드에 넣을 내용

rclcpp::Time now = node->now();
if ((now - past).nanoseconds() \* 1e-9 > 5) { ... }
msg.stamp = now + duration;
now - past: 경과 시간 계산
now + duration: 미래 timestamp 생성
발표 시 정정: “1초 느린 값”이 아니라 “1초 앞선 stamp”

발표 대본
“이 예제의 핵심은 세 가지입니다. 첫째, node->now() 또는 get_clock().now()로 현재 시간을 읽습니다. 둘째, now - past로 얼마가 지났는지 계산합니다. 셋째, msg.stamp = now + duration은 현재 시각보다 1초 뒤의 timestamp를 메시지에 넣는 것입니다. 따라서 원문처럼 ‘1초 느리다’라고 설명하기보다 ‘미래 시점을 stamp로 넣는 예제’라고 말하는 편이 정확합니다.”

9페이지. ROS 2 파일 시스템 개요

슬라이드 제목
ROS 2 파일 시스템은 왜 규칙적이어야 하는가

슬라이드에 넣을 내용

패키지 검색, 실행 파일 검색, 설정 파일 검색을 자동화
패키지 루트는 package.xml로 식별
한 워크스페이스에 여러 패키지 공존 가능
일반적으로 src/ 아래에 패키지 생성

발표 대본
“ROS 2 파일 시스템의 목적은 사람이 보기 좋으라고만 있는 것이 아닙니다. ros2 run, colcon, rosdep 같은 도구가 패키지를 찾고, 의존성을 읽고, 실행 파일을 찾기 위해 구조가 통일되어 있어야 합니다. 그래서 package.xml이 패키지의 루트를 표시하고, 패키지는 보통 워크스페이스의 src 아래에 둡니다. 또 한 워크스페이스에는 여러 패키지를 넣을 수 있지만, 패키지 안에 또 패키지를 중첩해서 넣지는 않습니다.”

10페이지. 패키지와 메타패키지

슬라이드 제목
패키지, 메타패키지, variant를 어떻게 설명할까

슬라이드에 넣을 내용

패키지: ROS 2 코드의 기본 단위
여러 관련 패키지를 묶는 묶음 개념 존재
ROS 2는 ROS 1처럼 “특수 metapackage 타입”이 핵심은 아님
실무에서는 관련 패키지를 exec_depend로 묶는 방식 사용

발표 대본
“원래 노트에서는 메타패키지를 하나의 집합 단위로 설명하고 있는데, ROS 2에서는 이걸 조금 더 정확히 말해주는 게 좋습니다. ROS 2에선 ROS 1처럼 특별한 메타패키지 타입에 기대기보다, 관련 패키지를 실행 의존성으로 묶은 일반 패키지나 variant 개념으로 이해하는 편이 실무와 가깝습니다. 예를 들어 ‘내 로봇의 navigation 묶음’ 패키지는 직접 기능을 수행하지 않고, localization·planner·controller 패키지를 함께 설치하게 만드는 역할을 할 수 있습니다.”

11페이지. 바이너리 설치와 소스 빌드 설치

슬라이드 제목
패키지는 설치해서 쓸까, 소스로 빌드할까

슬라이드에 넣을 내용

바이너리 설치: 빠르게 사용
소스 빌드: 수정, 디버깅, 내부 구조 확인
예시
sudo apt install ros-humble-teleop-twist-joy
git clone ...
colcon build --packages-select ...

발표 대본
“ROS 2 패키지를 쓰는 방식은 크게 두 가지입니다. 첫 번째는 apt로 설치하는 바이너리 방식이고, 이 경우 빠르게 실행해볼 수 있습니다. 두 번째는 Git으로 소스를 받아 직접 빌드하는 방식인데, 코드를 읽거나 수정하거나 브레이크포인트를 걸어야 할 때 훨씬 유리합니다. 발표에서는 ‘써보기만 할 때는 바이너리, 개발에 들어가면 소스 빌드’라고 구분해주면 이해가 빠릅니다.”

12페이지. 기본 설치 폴더와 사용자 워크스페이스

슬라이드 제목
/opt/ros와 ros2_ws는 어떤 차이인가

슬라이드에 넣을 내용

Underlay: /opt/ros/<distro>
Overlay: 사용자 워크스페이스
빌드 후 생성 폴더: build / install / log / src
install/local_setup.bash와 setup.bash의 차이

발표 대본
“보통 배포판으로 설치된 ROS 2는 /opt/ros/배포판명 아래에 있고, 이것을 underlay라고 생각하면 됩니다. 그 위에 내가 만든 ros2_ws를 overlay처럼 얹어서 쓰는 구조가 일반적입니다. 워크스페이스를 빌드하면 build, install, log, src 네 폴더가 보이게 되는데, 여기서 install 아래 setup 파일을 source해서 내 패키지를 환경에 추가합니다. 특히 local_setup은 overlay만 더하고, setup은 underlay까지 포함한다는 차이를 짚어주면 좋습니다.”

13페이지. 빌드 시스템 전체 그림

슬라이드 제목
ROS 2 빌드는 한 층이 아니다

슬라이드에 넣을 내용

개별 패키지 build tool: CMake / setuptools
build helpers: ament
meta-build tool: colcon
dependency graph의 기준은 package.xml

발표 대본
“이 부분은 발표에서 오해를 가장 줄여주는 슬라이드입니다. 흔히 ‘ROS 2는 ament로 빌드하고 colcon을 쓴다’고 말하지만, 공식 문서 기준으로 보면 C++ 패키지는 CMake, Python 패키지는 setuptools가 실제 빌드를 수행합니다. 그 위에서 ament가 ROS 2에 필요한 helper를 제공하고, colcon이 여러 패키지를 의존성 순서대로 묶어 빌드합니다. 즉, 하나의 단어로 끝나는 구조가 아니라 층이 나뉘어 있다고 설명하는 것이 정확합니다.”

14페이지. ament와 패키지 타입

슬라이드 제목
ament_cmake와 ament_python은 무엇이 다른가

슬라이드에 넣을 내용

ament package: package.xml을 가진 ROS 패키지
ament_cmake: C++ 중심 패키지
ament_python: Python 중심 패키지
혼합 패키지는 ament_cmake_python도 고려

발표 대본
“ament는 단순히 CMake를 대체하는 이름이 아니라, ROS 2 패키징 규칙과 helper 집합을 뜻합니다. 그래서 package.xml을 갖고 그 규칙을 따르는 패키지를 넓게 ament package라고 부를 수 있습니다. C++ 위주라면 ament_cmake, Python 위주라면 ament_python을 쓰고, 둘을 섞는 경우에는 ament_cmake_python 계열을 고려할 수 있습니다. 발표에서는 ‘언어에 따라 빌드 방식이 다르지만, 패키지로 포장되는 규칙은 ROS 2가 공통으로 가져간다’고 설명하면 좋습니다.”

15페이지. ros2 pkg create와 colcon build

슬라이드 제목
패키지 생성과 빌드 실전 명령어

슬라이드에 넣을 내용

cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake --node-name my_node my_package

cd ~/ros2_ws
colcon build --packages-up-to my_package --symlink-install
source install/local_setup.bash
--packages-select: 지정 패키지만
--packages-up-to: 대상 + 의존성까지
--symlink-install: 복사 대신 심볼릭 링크

발표 대본
“이 슬라이드는 실습과 가장 직접 연결됩니다. 먼저 src 안에서 ros2 pkg create로 기본 골격을 만들고, 워크스페이스 루트로 나와 colcon build를 실행합니다. 이때 --packages-up-to는 필요한 의존성까지 같이 올려주기 때문에 부분 빌드에 편하고, --symlink-install은 Python 코드나 설정 파일을 수정할 때 매번 다시 복사하지 않아도 되어서 개발 속도를 높여줍니다. 빌드 뒤에는 반드시 install/local_setup.bash를 source해야 새 패키지를 ROS 2가 인식합니다.”

16페이지. vcstool, rosdep, bloom

슬라이드 제목
협업에서 꼭 나오는 3가지 보조 도구

슬라이드에 넣을 내용

vcs export --exact > my.repos
vcs import < my.repos

sudo rosdep init
rosdep update
rosdep install --from-paths src -y --ignore-src
vcstool: 여러 저장소 버전 맞추기
rosdep: package.xml 기반 의존성 설치
bloom: 배포판용 패키지 릴리스 자동화

발표 대본
“실무 협업에서 세 도구의 역할은 명확히 다릅니다. vcstool은 ‘어떤 저장소들을 어떤 버전으로 받았는지’를 .repos 파일로 맞추는 도구입니다. rosdep은 package.xml을 읽고 OS 패키지 의존성을 설치해줍니다. bloom은 개발이 끝난 패키지를 공개 빌드팜과 rosdistro 흐름에 맞춰 릴리스해서, 나중에 apt install로 설치할 수 있게 만드는 역할입니다. 순서로 보면 ‘저장소 가져오기 → 의존성 설치 → 빌드 → 릴리스’입니다.”

17페이지. 패키지 파일 전체 지도

슬라이드 제목
패키지 안에는 어떤 파일이 들어가는가

슬라이드에 넣을 내용

공통 핵심: package.xml
C++ 패키지 핵심: CMakeLists.txt
Python 패키지 핵심: setup.py, setup.cfg, resource/패키지명
부가 문서: plugin.xml, CHANGELOG.rst, LICENSE, README.md

발표 대본
“여기부터는 패키지 폴더 내부를 보겠습니다. 모든 패키지의 중심은 package.xml입니다. C++ 패키지는 CMakeLists.txt가 빌드를 설명하고, Python 패키지는 setup.py, setup.cfg, 그리고 resource marker 파일이 중요합니다. 그 밖에 플러그인 선언, 변경 이력, 라이선스, 사용 문서 같은 파일이 붙으면서 패키지가 단순 코드 묶음이 아니라 재사용 가능한 소프트웨어 단위가 됩니다.”

18페이지. package.xml 깊게 보기

슬라이드 제목
package.xml은 도구가 읽는 패키지 설명서다

슬라이드에 넣을 내용

<package format="3">
  <name>sensor_fusion_pkg</name>
  <version>0.1.0</version>
  <description>Sensor fusion example</description>
  <maintainer email="dev@example.com">Dev</maintainer>
  <license>Apache-2.0</license>
  <buildtool_depend>ament_cmake</buildtool_depend>
  <depend>rclcpp</depend>
  <exec_depend>launch_ros</exec_depend>
  <test_depend>ament_lint_auto</test_depend>
</package>
이름, 버전, 설명, 관리자, 라이선스
빌드/실행/테스트 의존성 선언
depend는 build+build_export+exec를 한 번에 표현 가능

발표 대본
“package.xml은 사람이 읽는 README가 아니라, 도구가 해석하는 manifest입니다. 이 파일이 있어야 colcon이 패키지를 찾고, rosdep이 의존성을 설치하고, 릴리스 도구가 패키지 정보를 이해할 수 있습니다. 특히 depend 태그는 build, build export, exec 의존성을 한 번에 표현하는 축약형이라는 점을 같이 설명하면 좋습니다. 그리고 발표에서는 ‘ROS 2는 format 3 예제가 흔하지만, 개념적으로는 format 2 이상을 지원한다’는 점까지 한 줄로 정리하면 더 정확합니다.”

19페이지. CMakeLists.txt 깊게 보기

슬라이드 제목
C++ 패키지는 CMakeLists.txt에서 어떻게 설명되는가

슬라이드에 넣을 내용

find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)

add_executable(listener src/listener.cpp)
ament_target_dependencies(listener rclcpp std_msgs)

install(TARGETS listener
DESTINATION lib/${PROJECT_NAME})

ament_package()
의존 패키지 찾기
실행 파일 만들기
링크/설치 경로 지정
마지막에 ament_package()

발표 대본
“C++ 패키지에서는 CMakeLists.txt가 실제 빌드 로직의 중심입니다. find_package로 의존성을 찾고, add_executable로 실행 파일을 만들고, ament_target_dependencies로 ROS 2 패키지 의존성을 연결합니다. 그리고 install 규칙을 써야 ros2 run이 실행 파일을 찾을 수 있는 위치에 들어갑니다. 발표에서는 이 파일을 ‘컴파일 규칙 + 설치 규칙 + 의존성 연결표’라고 소개하면 잘 전달됩니다.”

20페이지. setup.py와 setup.cfg 깊게 보기

슬라이드 제목
Python 패키지는 왜 setup.py와 setup.cfg가 필요한가

슬라이드에 넣을 내용

data_files=[
('share/ament_index/resource_index/packages',
['resource/' + package_name]),
('share/' + package_name, ['package.xml']),
],
entry_points={
'console_scripts': [
'talker = my_pkg.talker:main',
],
}
[develop]
script-dir=$base/lib/my_pkg
[install]
install-scripts=$base/lib/my_pkg
resource/<package_name> marker 필요
entry_points로 실행 명령 등록
setup.cfg가 ros2 run이 찾을 위치를 맞춰줌

발표 대본
“Python 패키지는 CMake 대신 setuptools 기반으로 설치됩니다. 그래서 setup.py에 package.xml과 resource marker를 함께 설치하도록 적어주고, entry_points에 콘솔 명령을 등록합니다. 여기에 setup.cfg가 더해져서 실행 스크립트가 lib/<package_name> 아래 설치되도록 맞춰주는데, ROS 2의 ros2 run이 바로 그 위치를 기준으로 실행 파일을 찾습니다. 즉 Python 패키지에서는 setup.py와 setup.cfg가 합쳐져 CMakeLists.txt 비슷한 역할을 한다고 설명하면 됩니다.”

21페이지. plugin.xml, CHANGELOG, LICENSE, README

슬라이드 제목
실무에서는 코드보다 문서/메타데이터가 더 중요할 때가 있다

슬라이드에 넣을 내용

플러그인 XML: 어떤 플러그인을 제공하는지 선언
CHANGELOG.rst: 버전별 변경 내역
LICENSE: 법적 사용 조건
README.md: 설치법, 실행법, 예제, 주의사항

발표 대본
“패키지가 잘 실행되는 것만으로는 좋은 패키지라고 보기 어렵습니다. 플러그인 패키지라면 어떤 클래스를 어떤 이름으로 노출하는지 XML로 선언해야 하고, 사용자는 README를 통해 설치와 실행 방법을 이해해야 합니다. CHANGELOG는 릴리스 이력을 추적하게 해주고, LICENSE는 재사용의 법적 근거가 됩니다. 특히 bloom 같은 릴리스 흐름까지 생각하면, 이런 파일들은 선택이 아니라 품질의 일부라고 말해주는 편이 좋습니다.”

22페이지. 마무리

슬라이드 제목
ROS 2 프로젝트를 보는 한 줄 흐름

슬라이드에 넣을 내용

src에 패키지를 만든다
package.xml에 의존성을 적는다
CMakeLists.txt 또는 setup.py/setup.cfg를 작성한다
rosdep으로 의존성을 설치한다
colcon build로 빌드한다
source install/local_setup.bash로 overlay를 적용한다
필요하면 vcs로 저장소 묶음을 공유하고 bloom으로 배포한다

발표 대본
“이제 전체 흐름을 한 줄로 정리할 수 있습니다. ROS 2 개발은 패키지를 만들고, 그 패키지가 어떤 의존성을 갖는지 manifest에 적고, 언어에 맞는 빌드 파일을 준비한 뒤, rosdep과 colcon으로 실제 빌드하는 과정입니다. 그리고 실행 환경에 overlay를 올려서 내가 만든 패키지를 시스템에 연결합니다. 마지막으로 팀 협업에서는 vcstool이 저장소 묶음을 재현해주고, 공개 배포 단계에서는 bloom이 apt 설치 가능한 형태까지 연결해줍니다. 결국 오늘 본 시간, 파일 시스템, 빌드, 패키지 파일은 모두 하나의 개발 파이프라인입니다.”
