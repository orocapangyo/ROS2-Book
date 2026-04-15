# ROS2 action

## 13.1 개요

Node간에 Action Gold(service), Feedback(topic), result(service) 처럼 비동기식, 동기식 양방향 메세지 송수신 방식으로 액션 목표를 지정하고, 수행하면서 중간 결과 값을 전송하며 최종 결과값을 전송하는 방식이다.

액션의 구현 방식을 살펴보면, 토픽과 서비스의 혼합니다.

Action Client는 Service Client 3개와 Topic Subscriber 2개로 구성되어 있으며, Action Server는 Service Server 3개와 Topic Publisher 2개로 구성된다.
액션 목표, 피드백, 결과 데이터는 msg, srv 인터페이스의 변형인 action 인터페이스를 사용한다.

ROS1과 ROS2의 차이
ROS1에서는 Topic으로만 구성되어 있었다. 비동기 방식을 이용하다보면, 원하는 타임에 적절한 액션을 수행하기 어려운데 ROS2에서는 이를 해결하기 위해 목표 성태(Goal_state)를 추가했다.

액션의 상태로는 ACCEPTED, EXECUTING, CANCELING, SUCCEEDED, ABORTED, CANCELED가 있습니다.

## 13.2 Action Server and Action Client

노드의 액션 정보를 확인하기 위해서는 ros2 node info를 사용해 액션 부분을 확인 가능하다.
Action Servers: Action Clients: 에서 확인이 가능하다.
