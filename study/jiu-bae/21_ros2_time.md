# 21. ROS2의 시간

## 21.1 시계(Clock)와 시간(Time)

다수의 노드로 구성된 로봇의 소프트웨어 동작에서 시작은 매우 중요합니다.
시간에 따른 각 센서 값의 변화량과 그 센서들 간의 시간 동기화가 매우 중요.
여러 노드들이 서로 통신할 때 해당 데이터가 퍼블리시된 정확한 시간이 필수적. 또한 각 센서들은 주기가 다른 경우가 있음.

ROS2에서는 토픽에 데이터 뿐만 아니라 시간 함께 포함. stamp, frame id를 포함하는 std_msgs/msg/header 데이터 타입은 ROS2에서 제공하는 표준 메세지 타입.

ROS 프레임워크는 시간과 관련된 라이브러리를 제공하고 있으므로 적극적으로 활용합니다.
ROS2의 기본 시계는 System Clock(UTC, Coordinated Universal Time으로 표시),

- rclcpp: std::chrono
- rclpy: time

## 21.2 시간의 추상화(Time Abstractions)

ROS2에서는 시간을 추상화하여 쉽게 사용할 수 있는 환경을 제공합니다. 기본적인 시계 뿐만 아니라 타임 머신처럼 동작하는 시계도 사용합니다. 과거로 돌아가거나 시간을 더 빠르게 흘러가게 하거나 멈출 수 있음.

과거에 기록한 데이터를 다룰 때 ros2bag, gazebo, ignition에서 사용 가능.

시간을 추상화하여 3가지 형태로 제공합니다. 시간 소스를 통해 선언하여 사용 가능

typedef enum rcl_clock_type_e
{
/// Clock uninitialized
RCL_CLOCK_UNINITIALIZED = 0,
/// Use ROS time
RCL_ROS_TIME,
/// Use system time
RCL_SYSTEM_TIME,
/// Use a steady clock time
RCL_STEADY_TIME
} rcl_clock_type_t;

### 21.2.1 System Time

Host 시스템의 시간을 사용하는 것을 의미합니다. 예외적으로, 호스트와 네트워크 시간이 동기화되면 시스템 시간이 반대로 진행될 수 있습니다. 네트워크 상에서 특정 서버의 시간을 기준으로 호스트 머신들의 시간을 동기화하는 방법은 다음 명령어를 사용하는 것입니다.
sudo ntpdate ntp.ubuntu.com

### 21.2.2 ROS Time

ROS Time 은 시뮬레이션 환경에서 시간을 다루기 위해 사용됩니다. use_sim_time 라는 파라미터를 사용 합니다. 이 값이 True인 경우 설정된 노드가 clock 토픽을 수신할 때까지 시간을 0으로 초기화합니다.

### 21.2.3 Steady Time

하드웨어 타임아웃을 사용하는 시간을 의미하며, 단조 증가하는 특성을 가집니다.

## 21.3 Time API

ROS2에서 제공하는 시간 API에는 time, duration, rate가 있습니다.
rclcpp, rclpy 예제

```

#include <memory>
#include <utility>

#include "rclcpp/rclcpp.hpp"
#include "rclcpp/time_source.hpp"

#include "std_msgs/msg/header.hpp"


int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);

  auto node = rclcpp::Node::make_shared("time_example_node");
  auto time_publisher = node->create_publisher<std_msgs::msg::Header>("time", 10);
  std_msgs::msg::Header msg;

  rclcpp::WallRate loop_rate(1.0);
  rclcpp::Duration duration(1, 0);

  while (rclcpp::ok()) {
    static rclcpp::Time past = node->now();

    rclcpp::Time now = node->now();
    RCLCPP_INFO(node->get_logger(), "sec %lf nsec %ld", now.seconds(), now.nanoseconds());

    if ((now - past).nanoseconds() * 1e-9 > 5) {
      RCLCPP_INFO(node->get_logger(), "Over 5 seconds!");
      past = node->now();
    }

    msg.stamp = now + duration;
    time_publisher->publish(msg);

    rclcpp::spin_some(node);
    loop_rate.sleep();
  }

  rclcpp::shutdown();

  return 0;
}
```

```

import rclpy
from rclpy.duration import Duration
from std_msgs.msg import Header


def main(args=None):
    rclpy.init(args=args)

    node = rclpy.create_node('time_example_node')
    time_publisher = node.create_publisher(Header, 'time', 10)
    msg = Header()

    rate = node.create_rate(1.0)
    duration = Duration(seconds=1, nanoseconds=0)
    past = node.get_clock().now()

    try:
        while rclpy.ok():
            now = node.get_clock().now()
            seconds, nanoseconds = now.seconds_nanoseconds()
            node.get_logger().info('sec {0} nsec {1}'.format(seconds, nanoseconds))

            if ((now - past).nanoseconds * 1e-9) > 5:
                node.get_logger().info('Over 5 seconds!')
                past = node.get_clock().now()

            msg.stamp = (now + duration).to_msg()
            time_publisher.publish(msg)

            rclpy.spin_once(node)
            rate.sleep()
    except KeyboardInterrupt:
        node.get_logger().info('Keyboard Interrupt (SIGINT)')
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 21.3.1 Time

Time 클래스는 시간을 다룰 수 있는 연산자를 제공합니다.
그 결과를 seconds(double), nanoseconds(unisigned int, 64비트) 단위로 반환.
결과를 다양한 시간 형식으로 변환할 수 있으며, now 함수를 통해 현재 시간을 얻을 수 있습니다.

### 21.3.2 Duration

Duration 클래스는 시간의 기간에 대한 연산자를 제공합니다. 그 결과를 seconds(double), nanoseconds(unisigned int, 64비트) 단위로 반환. 이전 시간은 음수로 표기됩니다.

### 21.3.3 Rate

Rate 클래스는 반복문에서 특정 주기를 유지시켜주는 API를 제공합니다.
예시로 WallRate 클래스의 생성자에 헤르츠 단위로 주기 설정 후 반복문 가장 아래 줄에 sleep함수로 주기를 맞춰줄 수 있다.
ROS2에서는 Timer API 사용 추천

### 21.3.4 실행 결과

rclcpp개발된 노드 실행 시 UTC 형식으로 1초마다 로그가 나오고, 3초가 지날 때마다 로그가 나오는 것 확인 가능
use_sim_time:=True를 하는 경우 0으로 초기화된다. (/clock토픽이 없기 때문) /time토픽을 확인해보면 Duration을 이용한 시간 수정으로 인해 1초 느리게 나오는 것을 확인할 수 있다.

rclpy로 개발된 노드를 실행하고 확인 가능.
