---
layout: post
title:  "[5분 로봇] Long-term Autonomy - Against appearance changes"
date:   2020-01-02 02:07:58 +0900
tags: [5min-robot]
description: >
  장기적 자율주행과 SLAM
---

# [5분 로봇] Long-term Autonomy - Against appearance changes

[https://www.youtube.com/watch?v=LznbIiK__Zo&t=153s](https://www.youtube.com/watch?v=LznbIiK__Zo&t=153s)

GN-Net, 딥러닝을 이용한 Relocalization 방법

GN-Net을 보면 딥러닝 학습에 기존 Gauss-Newton 방법을 integrate했다는 장점도 있지만, 이 논문에서 가지는 특징은 바로 appearance change (계절 및 날씨변화)에 강건한 relocalization 방법을 상징한다는 것이다.

GN-Net에서는 학습을 통해 중간 mid-level trained feature를 사용하는데, 즉 네트워크에서 나오는 feature map를 relocalization image로서 활용하는 것이다.

위 영상의 아래 그림처럼 서로 다른 appearance에 대해서도 유사한 feature map이  나오는 것을 볼 수 있다.

Visual SLAM을 다양한 환경에서 사용할 때, 가장 큰 문제가 되는 것이 바로 외부 환경의 변화에 따른 appearance change이다. 이는 조명이나 날씨, 그리고 장기적으로는 계절에 의해서 발생할 수 있는데 이전에도 관련해서 다양한 연구들이 진행되었다.

이렇듯 강건성을 목표로 하는 연구 중에 몇가지를 소개해 보자면

- How to train a CAT : 저 조명 상태의 영상이 들어오더라도 Canonical image (가령 좋은 조명 상태의 이미지)를 만들어서 위치 인식 성능을 높이는 연구
- Illumination invariant imaging: RGB 카메라 특성을 이용해서 single channel invariant image를 만들어서 (Normalized RGB와 같은..) 위치 인식 성능을 높이는 연구

위와 같은 연구들이 있으며 이외에도 geometric한 성질만 이용하든가 하는 방향으로 문제를 해결하는 노력들도 보이고 있다.
