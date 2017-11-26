---
layout: post
title:  "Stereo Visual Odometry Scratch!"
date:   2017-11-26 02:07:58 +0900
tags: [Self-study]
description: >
  Stereo Visual Odometry
---

이번 글은 Visual odometry에 대한 대략적인 설명은 담고있다. SLAM을 접하면 가장 기본적으로 보게 되는 용어중 하나가 'Odometry'인데 이 글을 통해 기본적인 Visual Odometry에 대한 개념을 이해할 수 있기를 기대한다. 글은 기본적으로 [Avi Shingh의 영문포스트](https://avisingh599.github.io/vision/visual-odometry-full/)를 번역+수정하여 작성하였다. Visaul Odometry 샘플 알고리즘은 Avi Shingh과 같이 [Real-Time Stereo Visual Odometry for Autonomous Ground Vehicles(Howard2008)](https://www-robotics.jpl.nasa.gov/publications/Andrew_Howard/howard_iros08_visodom.pdf)을 기반으로 하였다. 코드는 원문 포스트의 깃허브를 Folk해서 부분적으로 수정하였다[]


## What is Odometry?
Odometry 또는 오도메트리라고 불리는 용어는 무엇을 표현할까? 자동차 계기판을 보면 차량이 간 거리를 표현하는 '주행거리' 표시가 있는데 이를 영어로 [Odometer](https://en.wikipedia.org/wiki/Odometer)라고 표현한다. 예상컨데 차량의 바퀴 회전수를 체크해서 (엔코더와 같이) 차량의 진행 거리를 측정하여 나타낼 것이다. Robotics에서의 Odometry를 좀 더 일반적인 표현을 사용하는데 단순한 이동거리가 아니라 로봇이 움직인 전체 경로를 표현하기도 한다. 이러한 경로를 구할 때 사용한 센 서에 따라서 Wheel Odometry (엔코더), Visual Odometry (카메라), Visual Inertial Odometry (카메라 + IMU) 등으로 표현한다. Odometry에서 나타내는 경로는 로봇의 Pose들로 구성되어 있고 일반적으로 시간 $$t$$일때의 포즈는 $$X^t = [x^t, y^t, z^t, \phi^t, \theta^t, \psi^t]$$로 나타낸다. 여기서 $$[\phi^t, \theta^t, \psi^t]$$는 [Euler angles](http://mathworld.wolfram.com/EulerAngles.html)을 표현하며 $$[x^t, y^t, z^t]$$는  [Cartesian coordinate](https://en.wikipedia.org/wiki/Cartesian_coordinate_system)에서 나타낸다. 결국 Visual Odometry라 함은 카메라 (이미지)를 이용해서 구한 카메라의 포즈 또는 로봇의 포즈를 의미한다.

## Stereo? Monocular?
Visual Odometry라는 키워드로 검색해보면 주로 접할 수 있는 용어가 Monocular 또는 Stereo Visual Odometry이다. 두 방법의 차이는 용어에서 확인할 수 있듯이 한 대의 카메라를 사용해서 포즈를 구하면 Monocular Visual Odometry, 그리고 두 대의 카메라를 사용해서 포즈를 구하면 Stereo Visual Odometry이다.

## Sample Stereo Visual Odometry

### Formulation of the Problem
#### Input
알고리즘의 입력은 Stereo 카메라에서 오는 image pair (left and right)의 stream이 사용된다. 간단하게 표현하게 위해서 시간 (프레임) $$t$$와 $$t+1$$일때의 스테레오 카메라의 왼쪽, 오른쪽 이미지를 수식을 표현하면 $$I^t_l, I^t_r, I^{t+1}_l, I^{t+1}_r$$이다. 여기서 위첨자는 시간, 아래첨자는 left, right를 의미한다. 그리고 스테레오 카메라는 캘리브레이션 (Calibration)이 되어있다고 전제한다. 알고리즘에서 사용되는 사전정보 (Prior)는 캘리브레이션 결과인 두 카메라의 [Intrinsic과 extrinsinc parameter](https://kr.mathworks.com/help/vision/stereo-camera-calibration.html)이다.

#### Output
알고리즘의 출력은 연속된 stereo image pair 간의 상대적인 포즈 정보이며 rotation matrix $$R$$와 translation vector $$t$$로 구성되어 있다.

### The algorithm
다양한 Stereo Odometry 알고리즘이 존재하지만 본 포스팅에서 참고할 논문은 2008년 IROS에 발표되었던 [Real-Time Stereo Visual Odometry for Autonomous Ground Vehicles(Howard2008)](https://www-robotics.jpl.nasa.gov/publications/Andrew_Howard/howard_iros08_visodom.pdf)이다. 샘플 데이터로는 Open dataset인 [KITTI benchmark dataset](http://www.cvlibs.net/datasets/kitti/eval_odometry.php)을 사용한다.

Outline:
1. 카메라로 부터 연속된 스테레오 영상 획득 $$I^t_l, I^t_r, I^{t+1}_l, I^{t+1}_r$$
2. 스테레오 영상 전처리: Undistortion & Rectification
3. Disparity map 계산: $$D^t \gets I^t_l, I^t_r$$, $$D^{t+1} \gets I^{t+1}_l, I^{t+1}_r$$
4. FAST 알고리즘으로 두 왼쪽 이미지 ($$I^t_l, I^{t+1}_l$$)에서 feature 검출 및 매칭
5. 순서 3에서 구한 Disparity map을 이용해서 순서 4에서 구한 feature들의 3D position 계산, $$I^t_l, I^{t+1}_l$$에서의 Point Clouds $$W^t_l, W^{t+1}_l$$ 획득
6. 전체 feature points들 중에서 모션 검출에 사용하기 적합한 포인트들 추리기 (Inliers)
7. 순서 6에서 구한 Inliers을 이용해서 Optimization, Frame $$t$$에서 왼쪽 카메라에 대한 Frame $$t+1$$의 왼쪽 카메라의 상대적인 위치를 구할 수 있다.

### Stereo Image Preprocessing
먼저 메일 알고리즘에 들어가기 전에 스테레오 영상을 전처리 하는 과정이 필요하다.
1. [Undistortion](https://en.wikipedia.org/wiki/Distortion_(optics)): 카메라의 렌즈나 센서의 위치 등에 의한 왜곡을 보정하는 단계
2. [Rectification](https://en.wikipedia.org/wiki/Image_rectification): 스테레오카메라에서 왼쪽, 오른쪽 카메라의 오차를 보정하는 단계이다. 이 단계를 거치고 나면 두 카메라의 epipolar line이 수평축과 평행하게 되므로 disparity를 계산할 때 한 축으로만 (수평축) 계산하면 된다. 아래의 그림을 보면 feature mathcing 결과(yellow line)가 수평축과 평행한 것을 확인할 수 있다. 즉, rectification을 통해서 아래의 그림과 같은 결과를 얻을 수 있도록 보정하는 것이다.

    <img align="middle" src="/image/posts/Self-study/2017-11-26-Stereo-visual-odometry/epi.jpg" width="80%">

### Disparity Map computation
스테레오 카메라에서 image pair를 획득하면 disparity map을 구할 수 있다. (카메라가 보고있는) 3차원 환경에서 어떤 점이 왼쪽 카메라에서 $$(x,y)$$ 위치에 있었다면 오른쪽 카메라에서 동일한 점은 $$(x+d, y)$$에 위치하게 된다. 여기서 $$d$$는 disparity를 의미하며, 반대로 두 카메라에서 disparity를 구하는 방법은 $$d = x_l-x_r$$이다.

Disparity map을 구할 때 사용하는 알고리즘은 Block 기반, Feature 기반 등 여러 방법이 있는데 본 포스팅에서는 (매틀랩에 내장되었는) 가장 기본적인 방법인 Block-Matching 알고리즘을 사용한다. 왼쪽 이미지에서 15x15 크기의 블록을 만들고 sliding-window 방법으로 오른쪽 이미지에서 해당 블록과 가장 유사한 부분을 찾아서 disparity를 구하는 방법이다. 수평축으로 검색을 하며 Sum-of-Absolute Differences (SAD)가 최소화하는 위치를 찾는다. 매틀랩에서는 이의 상위 버전인  [Semi-Global Block Matching algorithm](http://zone.ni.com/reference/en-XX/help/372916M-01/nivisionconceptsdita/guid-53310181-e4af-4093-bba1-f80b8c5da2f4/)이 구현되어 있으며 아래와 같이 사용한다.

```matlab
disparityMap1 = disparity(I1_l,I1_r, 'DistanceThreshold', 5);
```

### Feature Detection
여기서 사용한 Feature는 [FAST corner](http://www.edwardrosten.com/work/fast.html)이다. FAST corner는 중심점을 기준으로 주변 16 픽셀을 원 모양으로 탐색해서 빠르게 코너를 찾는 알고리즘으로 [ORB-SLAM](http://webdiis.unizar.es/~raulmur/orbslam/)등의 알고리즘에서도 사용한다. 물론 FAST corner가 아니라 SIFT나 SURF등의 feature를 사용할 수도 있으나 속도 등의 문제로 인해서 FAST를 사용한다.

그리고 Odometry 검출을 강인하기 위해서 detection 단계에서 들어간 하나의 트릭은 바로 ''Bucketing''이다. 이 과정은 입력 영상을 그리드로 나누고 FAST 특징점을 그리드의 각각 셀에서 독립적으로 구하는 과정을 의미한다. 예를들면, 약 600x400 이미지를 100x100 으로 그리드를 나누면 6x4 크기의 그리드가 된다. 각 셀마다 뽑은 FAST corner중에서 상위 20개만 후보군으로 사용된다. 이러한 방법을 사용하는 이유는 영상 전체를 대상으로 상위 feature를 추리게 되면 영상의 한 부분의 feature만 선택되믄 문제가 발생하게 된다. 이러한 bias는 모션 검출에서도 악영향을 끼칠 가능성이 높기 때문에 영상 전체 영역에서 전반적으로 feature가 검출되도록 하는 것이다. 관련 코드는 아래와 같다.

```matlab
points1_l = bucketFeatures(I1_l, h, b, h_break, b_break, numCorners);
```
그리고 function 내부는 다음과 같다.
``` matlab
function points = bucketFeatures(I, h, b, h_break, b_break, numCorners)
% input image I should be grayscale

y = floor(linspace(1, h - h/h_break, h_break));
x = floor(linspace(1, b - b/b_break, b_break));

final_points = [];
for i=1:length(y)
    for j=1:length(x)
    roi =   [x(j),y(i),floor(b/b_break),floor(h/h_break)];
    corners = detectFASTFeatures(I, 'MinQuality', 0.00, 'MinContrast', 0.1, 'ROI',roi );
    corners = corners.selectStrongest(numCorners);
    final_points = vertcat(final_points, corners.Location);
    end
end
points = cornerPoints(final_points);
```

### Feature Matching
Feature를 추출하고 나면 다음단계로는 Feature description과 matching 단계이다. 일반적으로는 Mathcing을 위해서 feature descriptor를 정의하고 NNDR (Nearest Neighbor Distance Ratio)등을 이용해서 matching을 하는데 본 포스팅에서는 [KLT tracker](https://cecas.clemson.edu/~stb/klt/)를 이용하여 matching을 한다. KLT tracker는 image gradient 방향으로 검색하며 pixel intensity 차이가 최소화 되는 지역을 matching 포인트로 삼는다. 자세한 내용은 [링크](https://cecas.clemson.edu/~stb/klt/)를 통해 확인할 수 있다. 이미지 $$I^t_l$$에서의 feature들을 $$F^t$$로 정의하고 $$I^{t+1}_l$$에서의 feature들을 $$F^{t+1}$$로 정의 했을때 코드에서 사용한 function은 아래와 같다.

```matlab
tracker = vision.PointTracker('MaxBidirectionalError', 1);
initialize(tracker, points1_l.Location, I1_l);
[points2_l, validity] = step(tracker, I2_l);
```

그리고 위와 같이 $$F^{t+1}$$를 matching feature로 사용하면 outlier도 다수 포함되어 있기 때문에 1) disparity값이 할당되지 않은 feature 2) tracking threshold가 낮은 feature는 검출에서 제외하였다.


### Feature Point Triangulation
이번 단계는 포인트의 disparity를 이용해서 각 프레임 ($$t, t+1$$) 에서의 3D point clouds를 구하는 과정이다. 이는 카메라의 intrinsic, extrinsic 파라미터를 이용해서 이미지의 feature point를 reprojection함으로써 획득할 수 있다. Reprojection matrix $$Q$$는 아래와 같으며 3D point clouds 연산은 다음과 같다. $$\hat{W}^t = Q\times[x,y,d,1]^T$$.

$$ Q=
  \left[ {\begin{array}{cccc}
   1 & 0 & 0 & -c_{x} \\
   0 & 1 & 0 & -c_{y} \\
   0 & 0 & 0 & -f \\
   0 & 0 & -1/T_{x} & 0
  \end{array} } \right]$$

- $$c_x=$$ x축 principal point (intrinsic parameter)
- $$c_y=$$ y축 principal point (intrinsic parameter)
- $$f=$$ focal length
- $$T_x=$$ Stereo baseline

### Inlier Detection step
Feature들의 3차원 포인트를 구하면 이제 Optimization에 들어가기 전에 Outlier들을 추려내는 과정이 필요하다. Optimization 기반 방법은 모든 입력들의 에러를 최소화 하는 방향으로 최적화 되기 때문에 입력들 중에 Outlier가 포함되어 있다면 결과가 좋지 않을 경우가 많다. 이를 위해 [Robust estimator (M-estimator)](https://en.wikipedia.org/wiki/M-estimator)등이 사용되기도 하나 가장 원론적인 방법은 Optimization에 사용할 입력을 정리해서 Outlier를 최대한 제거하는 것이 될 것이다.
우선 여기서 사용한 Maximum Inlier Set 검색 방법은 Point들의 [Maximal Clique Problem](https://en.wikipedia.org/wiki/Clique_problem)과 같다. 여기서 Graph의 한 노드는 한개의 feature set으로 볼 수 있다. 연속된 두 영상 ($$t$$, $$t+1$$) 각각에서 feature 들 간에 depth 차이가 작은 경우, 예를 들면 $$t$$에서의 1번과 2번 feature의 depth 차이와 $$t+1$$에서의 1번과 2번 feature의 depth 차이가 일정 이하로 작으면 1, 2번 feature는 서로 연관성이 있다고 보고 Consistency Matrix $$M(1,2) = 1$$로 설정한다. 이렇게 n개의 feature에 대해서 n-by-n matrix를 구성하고 이중에서 maximal clique를 만들어 나가는 과정이다.
가령 가장 많은 후보군이 링크된 feature를 $$f_m$$이라고 한다면 $$f_m$$과 링크된 feature들 중에서 다시 가장 많이 링크된 feature들을 찾아서 inlier로 정의한다. 코드는 아래와 같다.

```matlab
function cl = updateClique(potentialNodes, clique, M)

maxNumMatches = 0;
curr_max = 0;
for i = 1:length(potentialNodes)
    if(potentialNodes(i)==1)
        numMatches = 0;
        for j = 1:length(potentialNodes)
            if (potentialNodes(j) & M(i,j))
                numMatches = numMatches + 1;
            end
        end
        if (numMatches>=maxNumMatches)
            curr_max = i;
            maxNumMatches = numMatches;
        end
    end
end

if (maxNumMatches~=0)
    clique(length(clique)+1) = curr_max;
end

cl = clique;


function newSet = findPotentialNodes(clique, M)

newSet = M(:,clique(1));
if (size(clique)>1)
    for i=2:length(clique)
        newSet = newSet & M(:,clique(i));
    end
end

for i=1:length(clique)
    newSet(clique(i)) = 0;
end

```

### Pose Computation ($$R$$ and $$t$$)
이제 inlier pointset 까지 구했으면 마지막 과정은 inlier set을 이용해서 두 카메라간의 포즈를 구하는 과정이다. 보통 $$t$$ frame 기준으로 $$t+1$$ frame pose를 구한다. 여기서 사용한 방법은 Levenberg-Marquardt non-linear least squares를 사용하였으며 식은 다음과 같다.

$$\epsilon = \sum_{\mathcal{F}^{t}, \mathcal{F}^{t+1}} (\mathbf{j_{t}} - \mathbf{P}\mathbf{T}\mathbf{w_{t+1}})^{2} + (\mathbf{j_{t+1}} - \mathbf{P}\mathbf{T^{-1}}\mathbf{w_{t}})^{2}$$

- $$j^t, j^{t+1}$$: Inlier set의 2D pixel 위치
- $$w^t, w^{t+1}$$: 3D homogeneous coordinate
- $$P$$: projection matrix (3D to pixel)
- $$T$$: transfomation matrix

```matlab
function F = minimize(PAR, F1, F2, W1, W2, P1)
r = PAR(1:3);
t = PAR(4:6);
%F1, F2 -> 2d coordinates of features in I1_l, I2_l
%W1, W2 -> 3d coordinates of the features that have been triangulated
%P1, P2 -> Projection matrices for the two cameras
%r, t -> 3x1 vectors, need to be varied for the minimization
F = zeros(2*size(F1,1), 3);
reproj1 = zeros(size(F1,1), 3);
reproj2 = zeros(size(F1,1), 3);

dcm = angle2dcm( r(1), r(2), r(3), 'ZXZ' );
tran = [ horzcat(dcm, t); [0 0 0 1]];

for k = 1:size(F1,1)
    f1 = F1(k, :)';
    f1(3) = 1;
    w2 = W2(k, :)';
    w2(4) = 1;

    f2 = F2(k, :)';
    f2(3) = 1;
    w1 = W1(k, :)';
    w1(4) = 1;

    f1_repr = P1*(tran)*w2;
    f1_repr = f1_repr/f1_repr(3);
    f2_repr = P1*pinv(tran)*w1;
    f2_repr = f2_repr/f2_repr(3);

    reproj1(k, :) = (f1 - f1_repr);
    reproj2(k, :) = (f2 - f2_repr);
end

F = [reproj1; reproj2];
```

### Validation of results
위 알고리즘을 실제로 적용할 때는 두 가지 추가적인 validation을 적용해서 최종 포즈를 사용할 수 있는지 확인한다.
1. 각 Clique은 최소한 8개 이상 유사한 feature들이 있는 경우에만 노드를 추가할 것
2. Reprojection error $$\epsilon$$ 은 기준 error threshold 이하로 작아야 할 것
