---
layout: post
title:  "Visual Inertial Odometry Review 2"
date:   2020-01-08 02:07:58 +0900
tags: [review-papers]
description: >
  본 글은 ETH-RPG의 Davide Scaramuzza 교수님의 논문을 보고 정리한 글이며, 앞으로 Visual + Inertial 관련 연구를 진행할 계획이기 때문에 Survey paper로 정리를 시작한다.
---

# Visual Inertial Odometry 2

지난번 글에 이어서  ETH-RPG의 Davide Scaramuzza 교수님의 Survey paper를 정리한다.

## Camera-IMU Calibration

카메라와 IMU를 함께 사용하기 위해서는 첫번째로 두 센서의 Calibration 정보가 전제되어야 한다. Calibartion은 spatial transformation (두 센서간 위치 차이)와 time offset 정보 유추해야한다. 특히, 하드웨어적으로 싱크가 맞춰져있는 경우가 아니라, 사용자가 임의로 카메라와 IMU를 이용해서 센서 모듈을 만든다면 카메라와 IMU 간의 time offset을 아는 것이 매우 중요하다.

결론적으로는 Open Calibration tool box인 [Kalibr](https://github.com/ethz-asl/kalibr)를 이용하여 두 센서간의 캘리브레이션 정보를 획득할 수 있다.

## Examples of Applications

최근에 발표된 Open-source Visual Inertial Odometry 중 대표적인 몇가지 연구에 대한 소개이다.

- MSCKF (Multi-state contraint Kalman Filter)
    - 구글 ARCore나 Tango에서 베이스로 사용되는 기술 (최근에는 original MSCKF에서 많은 부분이 업데이트 되었다.)
    - Filter에서 Measurement 모델이 co-visibility feature를 가지고 있는 camera pose간의 geometric contraint로 표현되며, 3차원 feature가 아니라 2D 상에서 feature를 가지고 있다. (논문 확인 필요)
    - 최근에는 Event camera에 해당 방법을 확장한 연구를 진행
    - 코드: [https://github.com/daniilidis-group/msckf_mono](https://github.com/daniilidis-group/msckf_mono)
- OKVIS (Open Keyframe-based Visual-Inertial SLAM)
    - Keyframe 기반의 Sliding window상에서 non-linear optimization 방법을 이용한 연구
    - Cost function이 visual landmark의 weighted reprojection error와 weight inertial error로 이루어져있다.
    - Multi-scale Harris corner와 BRISK descriptor를 이용해 feature를 정의
    - Sliding window에서 오래된 keyframe은 marginalize해서 drop시킴
    - 코드: [https://github.com/ethz-asl/okvis](https://github.com/ethz-asl/okvis)
- ROVIO (Robust Visual Inertial Odometry)
    - EKF를 이용한 Visual State Estimator
    - Feature detection은 FAST corner feature를 사용하는데 landmark를 표현하는 데서 contribution을 가짐
    - Feature들은 Robot-centric bearing vector와 거리 (로봇 포즈 기준으로 range-bearing 모델로 표현)로 표현하며 multi-level patch를 추출해서 description에 적용한다.
    - Filter에서 IMU motion을 이용해서 initial tracking을 하고, photometric error를 update step에서의 innovation term으로서 사용한다.
- Vins-Mono
    - Sliding window기반의 non-linear optimization 방법이다.
    - 기본적으로 corner feature를 기반으로 하는데, 논문에서 추가적으로 새로운 corner feature 들을 소개 (논문 확인 필요)
    - 특이한 점은 Initialization (Bootstrapping)에서는 loosely coupled 방법을 사용하고, relocalization시에는 imu preintegration을 이용한 tightly coupled fusion을 사용한다.
    - Loop closure에 대해서는 4DoF pose graph optimzation을 제안하였다.
    - 코드: [https://github.com/HKUST-Aerial-Robotics/VINS-Mono](https://github.com/HKUST-Aerial-Robotics/VINS-Mono)
- SVO+MSF (Semi-Direct Visual Odometry + Multi-sensor Fusion)
    - Visual Odometry인 SVO와 sensor fusion 방법인 MSF를 이용한 방법
    - 각 센서별 모션을 독립적으로 추출한 후 fusion하는 loosely-coupled 방법을 사용한다.
    - 예상컨데, IMU motion으로 state-trasition을 하고 Camera motion으로 update를 수행할 듯
    - 코드: [https://github.com/uzh-rpg/rpg_svo_example](https://github.com/uzh-rpg/rpg_svo_example) / [https://github.com/ethz-asl/ethzasl_msf](https://github.com/ethz-asl/ethzasl_msf)
- SVO+GTSAM
    - IMU를 사용하는 방법중 최근에 많이 소개되는 On-manifold IMU preintegration을 이용한 방법.
    - IMU measurement를 keyframe pose간에 preintegration하여 구성하고, iSAM2를 이용한 online  factor graph optimization을 이용한다.
    - 대표적인 full smoothing 방법
    - 코드: [https://github.com/uzh-rpg/rpg_svo_example](https://github.com/uzh-rpg/rpg_svo_example) / [https://bitbucket.org/gtborg/gtsam/src/develop/](https://bitbucket.org/gtborg/gtsam/src/develop/)

### 결론

실제 로봇 (특히 드론)을 구동하는 환경에서 vins는 매우 합당한 선택이다. 최근까지 다양한 연구들이 진행되어왔으나, 안정적인 활용을 위해서는 타겟 환경에 맞는 최적화가 필수적이다.

이벤트 카메라와 같이 다른 특성을 가지는 센서를 이용하는 것도 하나의 방법이며, semantic이나 contextual 정보를 이용하는 것도 올바른 연구 방향일 것이다.
