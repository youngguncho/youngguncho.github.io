---
layout: post
title:  "ORB Slam 2 ROS setup"
date:   2017-03-02 22:07:58 +0900
tags: [Trial]
description: >
  ROS에서 사용하기 위한 ORB-SLAM2 setup
---

# ORB-SLAM2 Setup

우선 [ORB-SLAM2](https://github.com/raulmur/ORB_SLAM2) 빌드가 성공적으로 된 상태에서 ROS사용을 위한 추가적인 빌드 구성에 대해서 설명한다. 셜명은 ORB-SLAM2 github의 설명에서 추가적으로 필요한 부분을

## ROS package path 설정
ORB-SLAM2은 catkin build system이 아니라 일반적인 cmake project로 만들었기 때문에 catkin_ws가 아닌 다른 곳에서 build 한다. 그렇기 때문에 ROS에서 사용하기 위해서는 ORB-SLAM2 경로를 ROS 환경변수에 추가하고 build 해야 한다.

1. 터미널 창을 열고 다음과 같이 입력한며 <ORB-SLAM2-Root>는 ORB-SLAM2의 메인 폴더를 의미한다.
```
export ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:<ORB-SLAM2-Root>/Examples/ROS/ORB_SLAM2
```

2. 같은 터미널 창에서 (export 명령은 같은 터미널 창에서만 유효하다) 다음과 같이 입력하여 build 한다.
```
chmod +x build_ros.sh
./build_ros.sh
```
## Running ORB-SLAM2
1. 빌드한 ORB-SLAM2 binary 링크
ROS를 사용하면 catkin_ws에서 빌드한 프로젝트들은 "source devel/setup.bash" 명령을 통해 catkin_ws 작업공간을 ROS 환경에 오버레이 시킨다. 즉, catkin_ws에 새롭게 작성한 프로젝트를 "실행"하기 위해서 필요한 과정이다.
이와 마찬가지로 ORB-SLAM2에서도 똑같은 setup을 해야한다. Package path로 잡았던  <ORB-SLAM2-Root>/Examples/ROS/ORB_SLAM2 폴더를 보면 빌드된 바이너리들 Mono, Stereo, RGBD 실행파일들이 생성된 것을 확인할 수 있다. 즉, 이 작업공간을 ROS에서 확인(접근)할 수 있게 링크를 해주어야 하며 <ORB-SLAM2-Root>/Examples/ROS/ORB_SLAM2/build/devel/setup.bash에 관련 명령이 들어있다.
위에서 사용한 터미널 창에서 아래와 같이 입력하자.
```
source <ORB-SLAM2-Root>/Examples/ROS/ORB_SLAM2/build/devel/setup.bash
```

2. ORB-SLAM2 실행
위의 setup을 정상적으로 수행했다면 이제 rosrun을 통해 ORB-SLAM2를 실행할 수 있다.
PATH_TO_VOCABULARY는 Vocabulary/ORBvoc.txt를 의미하며 Setting_file은 각 example에 들어있는 .yaml 파일을 의미한다.
```
rosrun ORB_SLAM2 Mono PATH_TO_VOCABULARY PATH_TO_SETTINGS_FILE
```
