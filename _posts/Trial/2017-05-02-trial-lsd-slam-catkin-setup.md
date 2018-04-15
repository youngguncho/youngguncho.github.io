---
layout: post
title:  "LSD-SLAM with ROS kinetic + Ubuntu 16.04"
date:   2017-05-02 22:07:58 +0900
tags: [trial]
description: >
  lsd slam을 ROS kinetic + Ubuntu 16.04에서 cakin으로 빌드하기 위한 구성
---

# lsd slam setup

Direct slam의 대표적인 논문 중 하나인 [LSD-SLAM](https://github.com/tum-vision/lsd_slam/tree/master)을 ROS kinetic + Ubuntu 16.04에서 사용하기 위한 설정에 대해 설명한다. LSD-SLAM의 기본 설치방법을 보면 ubuntu 12.04, 14.04 그리고 ros fuerte, indigo를 공식 지원하며 catkin make 아닌 ros의 이전 빌드 시스템인 rosmake를 사용한다. 이번포스트에서는 현재 환경에서 사용하기 쉽게 catkin build system으로 warping된 LSD-SLAM과 kinetic에서 사용하기 위한 추가적인 구성에 대해 살펴보자. (빌드 성공까지는 확인)

## LSD-SLAM catkin version download
[LSD-SLAM catkin version](https://github.com/tum-vision/lsd_slam/tree/catkin)은 LSD-SLAM을 catkin_ws에서 사용하기 위해 warping된 git branch이다. catkin_ws의 src에서 LSD-SLAM을 clone한다.

```
git clone https://github.com/tum-vision/lsd_slam.git
cd lsd_slam
git checkout catkin
```

## Qglviewer install
보통 qglviewer에서 많이 에러가 나는데 qt4의 경우 qt5와 충돌이 나지 않으니 apt-get install로 setup해주어도 충분하다. 만약 libqglviewer-dev이 먼저 설치되어 있다면 libqglviewer-dev-qt4를 새로 설치하자.
```
sudo apt remove libqglviewer-dev
sudo apt install qt4-default
sudo apt install libqglviewer-dev-qt4
cd /usr/lib/x86_64-linux-gnu
sudo ln -s libQGLViewer-qt4.so libQGLViewer.so
```

## CMakeLists.txt and package.xml setup
처음 LSD-SLAM을 clone하고 catkin_make를 해보면 아마 FindEigen을 찾을 수 없다는 에러 메세지가 나올 것이다. 아마 다른 에러 메세지도 더 나올 수 있는데 아래의 설정을 추가하면 문제를 해결할 수 있다.

1. lsd_slam_core와 lsd_slam_viewer의 package.xml 에 cmake_modules 추가
    ```
    <build_depend>cmake_modules</build_depend>
    <run_depend>cmake_modules</run_depend>
    ```

2. lsd_slam_core와 lsd_slam_viewer의 CMakeLists.txt 에 cmake_modules 추가
    ```
    find_package(catkin REQUIRED COMPONENTS
      ...
      cmake_modules
    )
    ```

3. CMakeLists의 find_package(catkin ~~~)에 추가되어 있는데 package.xml에서 빠진 항목이 있으면 package.xml에 추가 (예를들어 message_generation 또는 image_transport)

4. lsd_slam_core의 CMakeLists.txt에서 lsmslam의 target link library에 X11 추가
    ```
    target_link_libraries(lsdslam ${FABMAP_LIB} ${G2O_LIBRARIES} ${catkin_LIBRARIES} csparse cxsparse X11)
    ```
5. lsd_slam_core의 CMakeLists.txt에서 catkin_INCLUDE_DIRS 추가
    ```
    include_directory {
      ...
      ${catkin_INCLUDE_DIRS}
      ...
    }
    ```
6. lsd_slam_viewer에서 PointCloudViewer.cpp (326), PointCloudViewer.h (98, 135) 소스파일 수정
    ```
    float x, y, z -> qreal x, y, z;
    ```

## Catkin make
빌드 순서에 따라서 에러가 날 수도 있기 때문에 catkin_make를 두 번 정도 해본다.
  ```
  cd catkin_ws
  catkin_make clean
  catkin_make
  catkin_make
  source devel/setup.bash
  rosrun lsd_slam_viewer viewer
  ```
