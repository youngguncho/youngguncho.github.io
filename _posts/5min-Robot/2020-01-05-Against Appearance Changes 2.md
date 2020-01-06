---
layout: post
title:  "[5분 로봇] Long-term Autonomy - Against appearance changes2"
date:   2020-01-05 02:07:58 +0900
tags: [5min-robot]
description: >
  장기적 자율주행과 SLAM
---


# [5분 로봇] Against Appearance Changes 2

[https://www.youtube.com/watch?v=s8XV1Y6opig&t=24s](https://www.youtube.com/watch?v=s8XV1Y6opig&t=24s)

_Oxford의 Paul Newman 교수님 연구실의 GAN을 이용항 appearance change에 대처하는 연구_

<img align="middle" src="/image/posts/5min-Robot/2020-01-05/2020-01-05.png" width="90%">

_예전 ICRA 2018에서 발표할때 사용했던 슬라이드_

앞의 포스팅과 마찬가지로 계절이나 날시 변화와 같은 appearance change에 대응하는 가장 직관적인 방법은 바로 Image-to-image translation을 이용해서 입력 영상을 원하는 appearance를 가지도록 변화시키는 것이다.

로봇 쪽 학회에서는 2018년 ICRA에서 관련 연구들이 많이 소개되었는데, 위 그림, 또는 아래의 링크와 같이 Image-to-image translation을 통해서 visual localization 성능을 높이는 연구들을 진행하였다.

각 연구별로 방법은 조금씩 다르나 기본적인 흐름은 Deep Generative Model (GAN이나 VAE 와 같은)을 이용하여 영상을 변화시키는 것이다. 즉, 기존에 획득한 데이터가 낮!에 획득하였고 현재 자동차가 밤에 주행하면서 이전 데이터 상에 위치 인식을 시키고 싶을때 현재 데이터를 낮 영상과 같이 변화 시켜서 위치 인식 성능을 높이는 것이다.

- ICRA 2018에 소개된 관련논문

  - [Addressing Challenging Place Recognition Tasks using Generative Adversarial Networks](https://arxiv.org/abs/1709.08810)

  - [Adversarial Training for Adverse Conditions: Robust Metric Localisation using Appearance Transfer](https://arxiv.org/abs/1803.03341)

  - [How to Train a CAT: Learning Canonical Appearance Transformations for Direct Visual Localization Under Illumination Change](https://arxiv.org/abs/1709.03009)

  - [DejavuGAN: Multi-temporal Image Translation toward Long-term Robot Autonomy](https://irap.kaist.ac.kr/index.php/Main/Publication?action=bibentry&bibfile=ref.bib&bibref=ycho-2018-icraws)

- CVPR에 소개된 관련논문

  - [Night-to-Day Image Translation for Retrieval-based Localization](https://arxiv.org/abs/1809.09767)
