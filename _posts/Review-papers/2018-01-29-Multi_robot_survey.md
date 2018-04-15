---
layout: post
title:  "Multiple-Robot Simultaneous Localization and Mapping: A Review"
date:   2018-01-29 02:07:58 +0900
tags: [review-papers]
description: >
  2016년 Journal of field robotics (JFR)에 개제되었던 Survey논문인 "Multiple-Robot Simultaneous Localization and Mapping, A Review"에 대한 리뷰 포스트 이다.
---

이번 글은 최근 읽었던 서베이 논문인 [Multiple-Robot Simultaneous Localization and Mapping, A Review](http://onlinelibrary.wiley.com/doi/10.1002/rob.21620/abstract)에 대한 리뷰 글이다. Multiple-Robot SLAM에 대한 궁금증에 무심코 읽은 논문인데 생각보다 정리가 잘 되어있어서 글로 남기고 싶어서 글을 적게 되었다.

## Introduction
이 논문은 우선 가장 기본적인 SLAM의 problem statement 정의에서 부터 시작한다. Single/Multiple에 상관 없이 SLAM 문제에서 정의될 수 있는 부분을 먼저 구분하여, 또는 Categorize 하여 정의하고, Multiple-Robot 로봇에 특정하여 발생할 수 있는 문제점들을 짚어가며 설명을 이어간다. 각 문제점들에 대한 정의가 끝나면 방법론적으로 접근하여 최근 Multiple-Robot SLAM을 문제를 푸는 방법에 따라(EKF, GraphSLAM 등의) 분류하였다. 그리고 마지막으로는 현재 Multiple-Robot SLAM 연구들에서 직면한 문제들, 앞으로의 방향에 대해서 언급하며 논문이 마무리 된다.

이 논문에서 도움이 될 만한 부분은 위에 표현된 것 처럼 기존 연구들이 잘 분류화 되어있다는 것이다. 물론 최근 서베이 논문인 [Past, Present, and Future of Simultaneous Localization and Mapping: Toward the Robust-Perception Age](http://rpg.ifi.uzh.ch/docs/TRO16_cadena.pdf)에서 더 방대한 부분을 다루고 있긴 하지만 여기서는 Mapping에 따른 분류, 그리고 Multiple-Robot을 다루며서 발생할 수 있는 문제점들을 좀 더 자세히 짚고 넘어갔다는 것이 아주 재밌는 부분이다.

그렇다면 여기서 Multiple-Robot SLAM이 필요한 이유는 뭘까?? 논문에서는 다음 application들이 Multiple-Robot의 활용이 필요한 부분이라고 하고 있다.

- fire fighting in forested and urban areas,
- rescue operations in natural disasters,
- cleaning operations like removing marine oil spills,
- underwater and space exploration,
- security and surveillance, and
- maintenance investigations.

즉, "로봇"의 활용이 필요한 극한환경에서는 대부분 Multiple-Robot의 도입이 임무 수행에 도움이 될 것이라고 예상하고 있다. 그도 그럴것이 자율주행차와 같이 사용자와 로봇이 1:1 매칭이 되는 경우가 아니라면 여러 대의 로봇이 서로 정보를 공유하면서 문제를 해결하는 것이 임무 수행 효울 + 안전성을 높이는데 큰 기여를 할 것이다.

## Problem Statement
우선 메인 내용에 들어가기에 앞서 문제 정의를 먼저 살펴본다. 상세 부분은 논문을 참고하고, 먼저 일반적인 SLAM의 문제정의를 보면 다음과 같다.

$$ p(m, x_{1:t}|z_{1:t},u_{1:t},x_0)$$

SLAM이라고 함은 위의 식과 같은 사후확률분포 (Poseterior)를 구하는 것인데, 1~t까지의 observation (z), control input 또는 odometry (u), 초기 로봇의 위치 (x_0)가 주어졌을 때 map (m)과 로봇의 경로(trajectory)를 구하는 것이다. (사실 EKF같은 filtering 기반 방법은 map + 현재의 state만 구하는 확률 모델로 볼 수 있다.)

이러한 SLAM의 확률적 정의는 Multiple-Robot에 대해서 확장해서 살펴볼 수 있다. 우선 두 로봇에 a, b에 대해서만 정의하면 다음과 같다.

$$ p(m, x^a_{1:t}, x^b_{1:t}|z^a_{1:t}, z^b_{1:t},u^a_{1:t}, u^b_{1:t}, x^a_0, x^b_0)$$

공유하는 global map (m)을 제외하고 나머지 state는 모두 a, b에 대해 확장하게 된다. 즉, 확률 문제가 두 로봇에 대해서만 정의되어도 매우 복잡해지게 되는데 이러한 정의를 위해서는 각 로봇의 observation과 global measurement 들간의 관계 (data association)도 정의가 되어있어야 한다는 것이다. 그림으로 그려보면 다음과 같다.

<img align="middle" src="/image/posts/Review-paper/2018-01-29-Multi_robot_survey/multi_prob_state.png" width="70%">

## SLAM의 3가지 이슈
우선 SLAM을 구성하는 3가지 요소를 살펴보면 1) sensors 2) data processing 3) map representation으로 나타낼 수 있다. 용어에 맞게 사용하는 센서, 데이터 처리 방법 (알고리즘), 그리고 맵의 표현 방법 이렇게 3가지 이슈들이 정리가 되어야 하나의 SLAM method를 완성할 수 있다. 저자가 논문에서 정리한 표를 보면 다음과 같다.

<img align="middle" src="/image/posts/Review-paper/2018-01-29-Multi_robot_survey/slam_3_issues.png" width="70%">

여기서 센서는 환경에 따라서 달라지기도 하고, 플랫폼이나 용도에 따라서 달라지기 때문에 일관되게 표현할 수 없지만 acoustic sensor와 같이 수중에서만 되는 센서도 있을 수 있으며 LiDAR와 같이 지상에서 좀 더 많이 사용되는 센서가 있을 수도 있다. Data processing 같은 경우는 크게 Filtering / Smoothing / AI 로 나눌 수가 있다.

#### Map representation에 따른 구분
여기서 재밌는 부분은 바로 map representation에 대한 부분이다. 사실 위와 같이 센서나 데이터 처리에 대한 내용은 다른 논문에서도 자주 언급되기 때문에 이미 파악하고 있을 수도 있는 내용인데 map representation같은 경우는 이 논문처럼 분류해놓은 것을 자주 보진 못한 것 같다. 논문에는 자세하게 표현되어 있지만 결과적으로 정리된 표를 보면 아래와 같다.

<img align="middle" src="/image/posts/Review-paper/2018-01-29-Multi_robot_survey/map_representation.png" width="60%">

논문에 표현된 map representation에 대한 예시들을 보면 다음과 같다. Topological map을 그림상의 점들과 연결된 링크들 처럼 보통 metric 정보 없이 (metric은 다시 embedding이 가능하니까) 노드들 간의 연결성을 보는 관점이고, semantic map은 map에 semantic 정보 (object, region labeling과 같은)를 추가해서 표현한 맵이다. 그리고 appearance map을 일반적으로 그래프로 표현되며 각 노드가 geometric 정보가 아닌 그림처럼 사진이나 또는 압축된 appearance 정보등으로 표현된 맵을 의미한다.

<img align="middle" src="/image/posts/Review-paper/2018-01-29-Multi_robot_survey/map_representation_ex.png" width="90%">

#### Data processing에 따른 구분
Map을 어떻게 표현할 것인가에 대해서 위에서 구분지어 살펴봤으면, 이번에는 Data processing, 즉 SLAM 문제를 풀기위한 방법론에서 구분지어 살펴볼 수 있다. 위의 표에서 Data processing에는 **Filtering**, **Smoothing**, **AI** 이렇게 3가지 방법으로 구별할 수 있다. Filtering 기반 방법에서 대표되는 방법은 우리가 흔히 아는 EKF (Extended Kalman Filter) SLAM, EIF (Extended Information Filter) SLAM, PF (Particle Filter) SLAM을 예시로 들 수 있다. 기본적으로 Filtering 기반 방법은 **로봇의 현재 위치와 맵**을 구하기 위한 목표로 삼는다. 반면 Smoothing은 기본적으로 **로봇의 전체 경로**를 구하는 목적으로 하고 (Pose graph SLAM의 경우에는) 추가적으로 맵을 함께 업데이트 하는 경우가 많다. AI 기반 방법은 이와는 별개로 학습을 통해서 Localization이나 Place recognition을 하는 연구가 많다. 이러한 연구들도 표로 정리해 보면 다음과 같이 관련 연구의 장, 단점과 함께 살펴볼 수 있다.

<img align="middle" src="/image/posts/Review-paper/2018-01-29-Multi_robot_survey/slam_comparison.png" width="90%">


## Multiple-Robot SLAM
Multiple-Robot의 SLAM에 대한 설명이 나오기 까지 설명이 꽤나 길었다. 하지만 Multiple-Robot도 기본적인 문제는 위에서 언급한 내용들의 확장이므로 꼭 필요한 부분이긴 하다. 거두절미하고, 이번 챕터에서는 Multiple-Robot의 Data 처리에 따른 구분과 Multiple-Robot SLAM에서 문제가 되는 10가지 문제점들을 한번 살펴보자.

### Multiple-Robot SLAM에 고려해야 할 4가지 질문들
Multiple-Robot SLAM에 있어서 다음 4가지 질문들이 필수적으로 고려되어야 한다.
1. 어떤 타입의 데이터를 주고 받을 것인가 (Data Sharing)
2. 어떻게 통신할 것인가 (Data Communication)
3. 어디에서 데이터를 처리할 것인가 (Data Distribution)
4. 어떻게 데이터를 처리할 것인가 (Data Processing)

이 4가지 질문은 내가 원하는 플랫폼에 따라서, 처리해야하는 문제에 따라서 다르게 답변할 수 있다. 각 문제에 대해서 예를 들어보면 1) 데이터는 일반적인 raw data를 주고 받을수도 있고, map이나 pose같은 처리된 데이터를 받을 수도 있고 2) 통신 방법과 통신의 안정성, 허용가능한 bandwidth에 대한 고민이 필요하고 3) 각 로봇에서 주변 로봇들의 데이터를 확보하고 각각 SLAM문제를 처리할지, 또는 한 agent에서 전체 로봇들의 데이터를 종합해서 처리할지 4) Filtering 기반으로 문제를 풀지, Graph로 문제를 풀지.. 등등의 고려사항들이 나올 수 있다.

이러한 문제들에 대해서 표로 나타내면 다음과 같다.

<img align="middle" src="/image/posts/Review-paper/2018-01-29-Multi_robot_survey/multi_four_data.png" width="70%">

대부분의 용어는 직관적으로 파악할 수 있는데 Data distribution에는 centralized, decentralized, distributed, nondistributed는 어떤 방법인지 쉽게 파악이 안된다. 이 문제 들에 대해 저자의 설명을 보면 다음과 같이 나눌 수 있다.

- Centralized: 임무 수행을 위한 연산이 사전에 지정된 로봇이나 외부 에이전트에 의해 이뤄지는 구조 (서버에서 문제를 해결)
- Decentralized: 각각의 로봇이 연산을 독립적으로 수행하는 구조, 이 경우 각 로봇 모두 문제를 풀기위한 충분한 computation power가 요구 (예를 들면, 여러대의 로봇이 서로 인접한 또는 전체 데이터를 주고 받고 각각 연산을 해서 독립적으로 임무를 수행할 수 있는 구조)
- Distributed: 임무 수행을 위한 연산을 여러 에이전트에서 나누어서 수행하는 구조 (예를 들면, 전체 맵에서 각 부분별로 나누어 문제를 해결하는 구조)

### Multiple-Robot SLAM이 어려운 10가지 이유
이제 Multiple-Robot에 대한 마지막 챕터이다. Multiple-Robot SLAM을 하면서 문제가 되는 10가지를 먼저 리스트업 해보면 다음과 같다.

1. **Relative Poses of Robots**: 각 로봇은 각자 자신이 기준되는 coordinate system을 가지고 있을텐데, 데이터를 종합하려면 로봇들 서로의 relative 정보를 알아야 한다.
2. **Uncertainty of the Relative Poses**: 위와 마찬가지로 각 로봇이 가지는 uncertainty는 각각이 지난 경로나 주변 measurement에 따라 다를텐데 이에 대한 uncertainty diff가 고려되어야 한다.
3. **Updating Maps and Poses**: 우선 로봇의 상대적 위치를 찾더라도 맵 정보가 업데이트 디ㅗ면서 로봇의 위치도 함께 보정이 되는데 이를 다시 로봇들이 반영하는 것이 고려되어야 한다.
4. **Line-of-sight Observations**: 여러 대의 로봇이 동시에 임무 수행을 할 때 로봇이 같은 시점에 함께 관측할 수 있는 observation을 파악해야 Multiple-Robot 문제를 좀 더 쉽게 할 수 있다.
5. **Closing Loops**: 각 로봇이 만든 서로 다른 맵 상에서 Loop closing을 할 수 있는 방법이 고려되어야 한다.
6. **Complexity**: 일반적으로 로봇에서는 real-time 성능이 필요하기 때문에 데이터 처리 속도를 고려해야 한다.
7. **Communications**: 어떤 방법으로 통신할 지, 어떤 데이터를 얼마만큼의 전송량을 가지고 주고 받을 지 고려해야 한다.
8. **Heterogeneous Vehicles and Sensors**: Platform이나 센서 정보가 다른 경우 서로 다른 정보들을 어떻게 association 할지 고려해야 한다.
9. **Synchronization**: 각각 로봇은 하드웨어 적으로 분리되어 있기 때문에 각 플랫폼의 system time에 따라 sync가 floating되어 있는데 각 로봇들의 sync를 얼마나 잘 맞출 수 있는지가 중요하다.
10. **Performance Measure**: Performance를 측정할 수 있는 measure를 찾는 것이 중요하다. (여러대의 실제 로봇 trajectory를 구하기가 쉽지 않기 때문)

논문에도 잘 설명이 되어있지만 각 부분을 간략히 보면 위와 같이 요약해서 볼 수 있다. 저자는 이 부분도 역시 표로 정리하고, 위 문제들을 고려한 Multiple-Robot SLAM 논문들도 함께 리스트를 제공하였다.

<img align="middle" src="/image/posts/Review-paper/2018-01-29-Multi_robot_survey/prob_multi_robot.png" width="90%">


## Conclusion
제목은 Multiple-Robot의 SLAM이지만 SLAM에 대해서 전반적인 내용도 알차게 들어있던 논문이었다. 본문이 워낙 길어서 _무려 46페이지에 달하는_ 분량이기 때문에 전체를 모두 요약하지는 못했지만 내가 보았을 때 한번쯤 키워드를 기억해두면 좋을 것 같은 부분을 요약하였다. 이외에도 method 부분을 좀 더 상세히 보고 그 중 재미있던 아이디어들만 살펴보면 다음과 같다.

- Duplicate landmark 검사를 위한 sequential nearest neighbor test
- Cooperative Positioning System
    - Parent-child robot이 이동-정지를 하며 위치 보정
- Manifold representation을 이용한 map 표현
- Reinforcement Learning등을 통해 언제 map merging을 할 것인지 학습
- Map 들간에 중복영역이나 유사성 판단을 위해 space transform
    - Hough transform등을 이용해서 map merging candidate selection
- Topological approach 에서 probabilistic generalized voronoi diagram

다음 Survey 페이퍼는 조금 더 시간을 들여서 가장 최근 SLAM survey 논문이자 2016년 TRO에 개제되었던
[Past, Present, and Future of Simultaneous Localization and Mapping: Toward the Robust-Perception Age](http://rpg.ifi.uzh.ch/docs/TRO16_cadena.pdf)를 정리해보도록 하자.
