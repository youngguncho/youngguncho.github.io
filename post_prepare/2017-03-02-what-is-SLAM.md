---
layout: post
title:  "What is SLAM?"
date:  March 2, 2017 12:38 AM
tags: [SLAM]
description: >
  SLAM의 정의와 배경을 먼저 살펴본다.
---

(Start on March 2, 2017 12:38 AM, wip)

## What is SLAM?
SLAM (Simultaneous Localization and Mapping)
지금은 University of Bonn에서 [Cyrill Stachniss](http://www.ipb.uni-bonn.de/) 교수님의 강의였던 [Robot mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 에서는 SLAM에 대해서 다음과 같이 표현하였다. 
**로봇의 pose와(localization) 주변 환경의 map을(mapping) 동시에 추정하는 것**

풀어서 설명하면 localization은 주변 환경 정보 (landmark 등의 주변정보)를 알고 있을 때 로봇의 위치를 정확히 추정하는 것을 의미하며 Mapping은 로봇의 위치를 정확히 알고 있을 때 landmark나 장애물의 위치와 같은 주변 정보를 유추하는 것을 의미한다. 
또다른 참고자료인 Survey 논문([Past, Present, and Future of SLAM](https://arxiv.org/pdf/1606.05830.pdf))을 참고하면 맵의 필요성에 대해 다음과 같이 언급하였다. <br/>
1) Path planning을 위한 정보 제공이나 주변 환경의 시각화를 통한 정보 제공하며 <br/> 
2) Landmark등을 이용하여 로봇의 위치추정 오류를 제한한다. <br/>
만약 주어진 맵이 없는 경우 wheel odometry나 motion sensor를 통한 추측항법(dead-reckoning)은 계속해서 에러를 누적하게 된다. 즉, 로봇의 포즈에 drift가 쌓여 위치 오차가 점점 커지게 된다. 그리고 이러한 경우에 로봇은 기존에 주행했던 영역을 다시 방문함 (Loop closure) 으로써 로봇의 위치를 보정한다. <br/>
  위의 경우를 종합해서 표현하면 로봇은 모션 정보를 이용해서 주행하게 되는데 기존에 주어진 맵이 있는 경우 맵 정보를 이용해서 위치 보정을 수행하지만 만약 맵이 주어지지 않는 경우에도 Loop closure를 통해 로봇의 위치를 보정할 수 있다. 또한 로봇의 위치가 보정되면 새롭게 추가되는 또는 로봇이 인식하는 맵 정보도 함께 보정할 수 있다. <br/>
이렇듯 SLAM을 이용하면 기존에 주어진 맵이 없는 다양한 시나리오에서도 주행 및 주변환경 인식이 가능해진다. 

## Definition of SLAM
다시 [Robot mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/)으로 돌아와서 SLAM의 문제를 정의하면 다음과 같다.
* Given (주어진 정보)
	* Robot's control (보통 로봇의 odometry로 표현된다)
		$$U = u_{1:T} = \lbrace u_1, u_2, \dots, u_T \rbrace$$
    * Observations (Laser scan, feature point등의 주변 환경정보)
    	$$Z = z_{1:T} = \lbrace z_1, z_2, \dots, z_T \rbrace $$
* Wanted (Unknown variables)
	* Map of the environment (Position of Landmark)
		$$m$$
    * Path of the robot
		$$X = x_{0:T} = \lbrace x_1, x_2, \dots, x_T \rbrace $$

수식으로 풀어서 정리할 때 각 논문별로 표현이 조금씩 다르지만 (주어진 정보를 observation으로 하나로 표현하거나, map과 robot pose를 하나로 표현하는 등) 결국 SLAM은 아래와 같이 주어진 정보를 이용하여 원하는 unknown variable을 구하는 estimation 문제인 것이다. 확률 모델과 그래프 모델로 나타내면 다음과 같다. 

* Probabilistic Model
    $$p(x_{0:T}, m | z_{1:T}, u_{1:T})$$
    
* Graphical Model
	<img align="middle" src="/image/posts/SLAM/what-is-slam/slam_graph_model.png" width="100%">

