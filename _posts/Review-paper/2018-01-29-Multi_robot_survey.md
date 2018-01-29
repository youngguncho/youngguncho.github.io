---
layout: post
title:  "Multiple-Robot Simultaneous Localization and Mapping: A Review"
date:   2018-01-29 02:07:58 +0900
tags: [Self-study]
description: >
  2016년 Journal of field robotics (JFR)에 개제되었던 Survey논문인 "Multiple-Robot Simultaneous Localization and Mapping, A Review"에 대한 리뷰 포스트 이다. [작성중]
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

<img align="middle" src="/image/posts/Review-paper/2018-01-29-Multi_robot_survey/multi_prob_state.png" width="60%">

## SLAM의 3가지 이슈
우선 SLAM을 구성하는 3가지 요소를 살펴보면 1) sensors 2) data processing 3) map representation으로 나타낼 수 있다. 용어에 맞게 사용하는 센서, 데이터 처리 방법 (알고리즘), 그리고 맵의 표현 방법 이렇게 3가지 이슈들이 정리가 되어야 하나의 SLAM method를 완성할 수 있다. 저자가 논문에서 정리한 표를 보면 다음과 같다.

<img align="middle" src="/image/posts/Review-paper/2018-01-29-Multi_robot_survey/slam_3_issues.png" width="60%">

여기서 센서는 환경에 따라서 달라지기도 하고, 플랫폼이나 용도에 따라서 달라지기 때문에 일관되게 표현할 수 없지만 acoustic sensor와 같이 수중에서만 되는 센서도 있을 수 있으며 LiDAR와 같이 지상에서 좀 더 많이 사용되는 센서가 있을 수도 있다. Data processing 같은 경우는 크게 Filtering / Smoothing / AI 로 나눌 수가 있다. 여기서 재밌는 부분은 바로 map representation에 대한 부분이다. 사실 위와 같이 센서나 데이터 처리에 대한 내용은 다른 논문에서도 자주 언급되기 때문에 이미 파악하고 있을 수도 있는 내용인데 map representation같은 경우는 이 논문처럼 분류해놓은 것을 자주 보진 못한 것 같다. 논문에는 자세하게 표현되어 있지만 결과적으로 정리된 표를 보면 아래와 같다.

<img align="middle" src="/image/posts/Review-paper/2018-01-29-Multi_robot_survey/map_representation.png" width="60%">

논문에 표현된 map representation에 대한 예시들을 보면 다음과 같다. Topological map을 그림상의 점들과 연결된 링크들 처럼 보통 metric 정보 없이 (metric은 다시 embedding이 가능하니까) 노드들 간의 연결성을 보는 관점이고, semantic map은 map에 semantic 정보 (object, region labeling과 같은)를 추가해서 표현한 맵이다. 그리고 appearance map을 일반적으로 그래프로 표현되며 각 노드가 geometric 정보가 아닌 그림처럼 사진이나 또는 압축된 appearance 정보등으로 표현된 맵을 의미한다.

[작성중]
