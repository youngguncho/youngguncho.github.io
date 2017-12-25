---
layout: post
title:  "Stereo Visual Odometry C++ implementation!"
date:   2017-12-25 02:07:58 +0900
tags: [Self-study]
description: >
  Stereo Visual Odometry (C++ version) [작성중]
---

이번 글은 지난 포스트에 이어서 Stereo Visual Odometry에 대한 포스트이다. 이번 글에서는 Stereo Visual Odometry의 C++ implementation에 대해서 간략하게 설명한다. 코드는 [ORB-SLAM2](https://github.com/raulmur/ORB_SLAM2)에서 사용한 ORB feature 기반의 Stereo matching 기법과 [GTSAM](https://bitbucket.org/gtborg/gtsam)의 projection BA (Bundle adjustment)를 이용해서 작성하였다. Odometry dataset은 public dataset인 [Kitti Odometry Benchmark](http://www.cvlibs.net/datasets/kitti/eval_odometry.php)를 사용하였다.

## Stereo Matching
지난 Stereo Odometry의 Matlab 기반 포스트와는 달리 이번 글에서는 ORB feature를 이용해서 포인트 기반으로 빠르고 안정적으로 stereo matching을 할 수 있는 ORB SLAM2의 방법을 차용하여 구현하였다. ORB SLAM2에서는 왼쪽, 오른쪽 이미지에서 각각 ORB feature/descriptor를 추출한 후 왼쪽 이미지와 가장 matching이 잘 되는 오른쪽 이미지의 descriptor를 matching 포인트로 이용하였다. 엄밀하게 매칭 포인트를 구하기 위해서 feature description의 각 Octave level에서 matching을 해서 disparity를 구한다. 아래의 이미지가 Stereo Matching에서 구한 feature를 3차원 포인트로 복원한 모습으로 지면과 양옆의 건물이 올바르게 나온 것을 확인할 수 있다. (추가중))

## Motion estimation
이전 프레임의 왼쪽 이미지 $$I^{t-1}_l$$과 현재 프레임의 왼쪽 이미지 $$I^{t}_l$$가 모션을 구하기 위한 포인트로 사용된다. ORB descriptor가 Binary descriptor이기 때문에 Opencv에서 Brute-Force Hamming Distance를 Matching Distance로 사용하고 Nearest Neighbor Distance Ratio (NNDR)을 이용해서 두 프레임간의 매칭 포인트를 구한다. 아래의 과정은 두 프레임간 매칭포인트를 구한 후 Outlier를 제거하고 Inlier와 대응되는 3차원 포인트 (스테레오 카메라이므로)를 이용해서 모션을 구하는 방법이다.

### Compute Inliers
모션을 구하기 위해 Bundle adjustment를 하기 전 해야하는 과정은 바로 Feature matching과 Outlier를 최대한 제거하는 과정이다. 물론 Bundle adjustment (BA)를 하기위해서 error model을 세울 때 robust kernel을 이용해서 (Huber weight와 같은 [M-estimator](https://en.wikipedia.org/wiki/M-estimator)) Outlier의 영향을 줄일 수 있지만 기본적으로 모든 포인트의 에러를 줄이는 방향으로 최적화가 되기 때문에 Outlier가 모션 추정에 영향을 주는 것이 사실이다.
이러한 문제를 방지하기 위해 많이 사용하는 방법이 Motion Model을 기반으로 한 (Homography나 Fundamental Matrix) RANSAC 알고리즘을 이용해서 Inlier를 추리는 방법이다. [RANSAC](http://darkpgmr.tistory.com/61)은 어떤 모델의 파라미터를 구하기 위한 포인트 N개를 랜덤하게 뽑고 모델에 적용했을 때의 포인트들의 에러를 구해서 일정 에러 이내의 포인트를 추리고, 다시 랜덤하게 뽑고 하는 과정을 통해서 모델을 구하는 방법이며 링크된 블로그에 자세한 설명이 되어있다. C++ 코드를 작성하는 경우에는 opencv를 이용하면 매우 간편하게 homography나 fundamental matrix기반의 RANSAC을 적용할 수 있다. 아래의 코드에서 points1, points2가 의미하는게 연속된 두 프레임에서 매칭된 ORB feature들이며 status가 각 포인트가 Inlier인지 Outlier인지 나타내는 Index가 CV::Mat의 type으로 저장되어있다.

```c
// Fundamental matrix
H = cv::findFundamentalMat(points1, points2, cv::FM_RANSAC, error, 0.99, status);
// Homography
H = cv::findHomography(points1, points2, cv::RANSAC, error, status);
```

### Two-view BA
위 과정에서 Inlier point를 추리고 나면 이제 연속된 프레임에서 어느정도 신뢰할 만한 매칭 포인트를 가지고 있다고 말할 수 있다. 이제 이 포인트들을 이용해서 두 카메라간의 모션을 구하는 방법에 대해서 살펴본다. 이 방법은 [ORB-SLAM2](https://github.com/raulmur/ORB_SLAM2)에서 Appendix의 Bundle adjustment와 같은 방법으로 multi-frame에 대해서 적용할 수 있지만 이번 구현에서는 연속된 영상의 two-view BA에 적용하였다. 우선 방법에 대해서 간단히 말로 풀어쓰면 다음과 같다. 기준 프레임 (이전 프레임)에서 포인트들의 초기 3차원 위치를 알고 있을 때 이 포인트들을 임의의 모션을 이용해 타켁 프레임으로 transform하고 이미지 좌표계 상으로 projection 했을 때의 포인트 위치 (u, v)는 feature matching에서 구한 대응되는 feature들의 위치와 같아야 한다. 우선 $$i$$번째 포인트의 이미지 좌표상의 위치와 $$\mathbf{x} = (u, v)$$와 3차원 위치 $$X = (x, y, z)$$를 식과 같이 표현하며 아래의 subscription은 해당하는 프레임을 의미한다. 프레임간의 모션은 $$\mathbf{T}_{t,t-1}$$로 나타냈으며 의미는 t-1에서 t로의 모션을 뜻한다. 아래의 식은 포인트 i에 대한 에러를 나타내며 $$\pi$$는 3차원 포인트를 이미지 상으로 projection하는 projection function을 의미한다.

$$
\mathbf{e}^i = \mathbf{x}^i_{t}-\pi^i(\mathbf{T}_{t,t-1}, X^i_{t-1})
$$

그러므로 Cost function은 위 에러의 합이 최소화 되는 방향으로 최적화 되며 아래의 식과 같이 표현할 수 있다. 아래의 식에서 $$\Omega$$는 2x2 Covariance matrix이며 $$\rho$$는 robust kernel인 M-estimator를 의미한다. 본 구현에서는 Huber cost function을 적용하였다.

$$
C=\sum_{i}\rho(\mathbf{e}^i\Omega^{-1}\mathbf{e}^i)
$$

## Result and Conclusion (작성중)
