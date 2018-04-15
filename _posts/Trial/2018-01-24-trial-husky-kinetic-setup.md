---
layout: post
title:  "Husky setup for ROS kinetic"
date:   2018-01-24 02:07:58 +0900
tags: [trial]
description: >
  lsd slam을 ROS kinetic + Ubuntu 16.04에서 cakin으로 빌드하기 위한 구성 (작성중, 그림추가 필요)
---

## Husky + ROS kinetic
Husky의 [ROS 공식 패키지](http://wiki.ros.org/Robots/Husky)에 들어가면 [github repository](https://github.com/husky/husky)에 가서 source를 받고 인스톨 후에 어떻게 셋업하는지에 대한 설명이 나와있다. 하지만 github repo에서 아무 생각없이 바로 소스를 받게 되면 저자? 회사? 가 말하는 현재 가장 stable version인 indigo version을 받게 된다. 물론 Husky만을 사용한다면 ROS를 indigo로 낮춰서 사용할 수도 있겠지만.. 다른 센서 셋업과도 함께 사용하기 때문에 Kinetic에서 잘 setup하는 방법을 살펴보자. 참고로 본 글의 내용은 [issue](https://github.com/husky/husky/issues/66)와 개인적인 경험을 토대로 작성하였다.

## Dependencies
github repo에서 메인 소스를 받기 전에 Husky를 셋업할 때 필요한 다른 패키지들을 설치해준다. 다음 패키지 들은 husky driver를 설치할때, 또는 설치하고 실행할 때 나오는 요구사항들을 종합해놓은 것이다. Husky의 control과 simulation에 필요한 패키지로 종합할 수 있다.

```
sudo apt-get install ros-kinetic-lms1xx
sudo apt-get install ros-kinetic-interactive-marker-twist-server
sudo apt-get install ros-kinetic-twist-mux
sudo apt-get install ros-kinetic-imu-tools
sudo apt-get install ros-kinetic-controller-manager
sudo apt-get install ros-kinetic-robot-localization
```
## Main source build
다음은 Husky의 메인 소스를 받을 차례이다. [Github page](https://github.com/husky/husky)를 들어가보면 branches에 kinetic-devel이 있는 것을 확인할 수 있다. 우리는 ROS 버전에 맞게 kinetic-devel의 노드를 기반으로 셋업한다.

```
git clone https://github.com/husky/husky.git
git checkout kinetic-devel
cd <catkin_ws_dir>
catkin_make
```

## Test installation (Simulating Husky)
Husky 설치가 완료되었으면 Husky simulator를 통해서 제대로 설치되었는지 확인할 수 있다. 우선 시스템이 husky repo를 읽을 수 있도록, 그리고 simulation을 위한 husky model의 위치를 환경변수로 지정한다.

```
export source/devel/setup.bash
export HUSKY_GAZEBO_DESCRIPTION=$(rospack find husky_gazebo)/urdf/description.gazebo.xacro
```

그리고 다음 husky launch를 실행하면 Gazebo와 husky simulation을 확인할 수 있다. 그리고, Gazebo 처음 실행할 때 매우 느리니 조금만 참고 기다려 본다.

```
roslaunch husky_gazebo husky_empty_world.launch
```
