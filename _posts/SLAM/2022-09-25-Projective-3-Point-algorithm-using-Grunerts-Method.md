---
layout: post
title: "Projective-3-Point algorithm using Grunert's Method"
date: 2022-09-25 00:01:30 -0400
categories: SLAM
use_math: true
---

PnP는 카메라의 위치를 추정하기 위한 방법으로, 3d world coordinate 상의 점과 2d image coordinate상의 매칭되는 point pair가 적어도 3개 있을 때 사용할 수 있다. 여기서 3개의 point 쌍을 가지고 카메라 위치를 추정하는 P3P 방법을 알아보자.

![image](https://user-images.githubusercontent.com/67038853/192155732-108e1be3-306f-406a-8133-b1031e691b9d.png)

위 그림과 같이 삼각뿔을 그리고, 위의 식을 얻는다. 

![image](https://user-images.githubusercontent.com/67038853/192156137-d80c0799-e170-4b88-9bc0-31fc16c16498.png)

각 ray는 위와 같이 계산되며, N은 normalization, sign은 ray가 camera center에서 3d points로 가는 방향이 되도록 결정한다.

![image](https://user-images.githubusercontent.com/67038853/192155797-8d18f855-e56f-4457-8d0d-ea0eff891057.png)

각 ray vector를 알게되면 내적을 통해 ray 사이의 각도를 위와 같이 구할 수 있다.

![image](https://user-images.githubusercontent.com/67038853/192156235-c9f109b3-b4ba-456a-99c7-a26cbf6fb889.png)

$s_{1}$을 u,v로 나타내면 $a^{2},b^{2},c^{2}$으로 부터 위의 세 식을 얻을 수 있다. 

이 때 v만을 남겨서 정리해보면,

![image](https://user-images.githubusercontent.com/67038853/192156366-a7fe732c-f4d0-41d1-89e6-aaaf4a37d1f2.png)

다음과 같이 나타낼 수 있다. 

![image](https://user-images.githubusercontent.com/67038853/192156389-202cda16-de4e-4df9-bc46-8351b8fc38f9.png)

![image](https://user-images.githubusercontent.com/67038853/192156418-a4b35d80-80f0-457f-9ca4-8057ce8a715e.png)

각각 결과값은 위와 같다. 여기서 우리는 4개의 $s_{1},s_{2},s_{3}$ 쌍을 해로 가질 수 있다.

![image](https://user-images.githubusercontent.com/67038853/192156508-b92130aa-fc9b-44ae-92f1-711241c102fb.png)

왜냐하면, 밑면의 삼각형을 기울이더라도 각 점과의 거리 $a,b,c$와 각도에는 영향을 미치지 못하는 경우가 있기 때문이다. 즉, $a,b,c$, $\alpha,\beta,\gamma$가 같아도 $s_{1},s_{2},s_{3}$가 같은 경우가 있다. 이러한 경우가 4가지가 있는 것인데, 어떻게 optimal한 solution을 찾을까?

#### 모호성 해결

1. Good initial Guess(GPS정보)를 활용하여 정확성을 높일 수 있다.
2. 다른 한 point를 추가적인 정보로 활용한다.

추가로 한 점이 주어지게 된다면, 카메라 원점으로부터 거리 계산을 통해 적절한 $s_{1},s_{2},s_{3}$를 정할 수 있다.

#### 카메라 위치 추정

앞서 구한 $s_{1},s_{2},s_{3}$를 통해 카메라 위치를 추정해보자.

camera coordinate에서 3d point의 위치는

$^{K}X_{i}=s_{i}{^{k}x_{i}^{s}}$로, 카메라 중심으로 부터 3d point를 향해 쏜 ray에 s를 곱하여 얻을 수 있다. 이 X 좌표는 $^{k}X_i=R(X_i-X_O)$와 같게되고, 여기서 R과 T를 구할 수 있게 된다. (Rigid body transformation, TODO : 미지수 6개, 식 3*3=9개? )

#### RANSAC

Problem at real situation -> solving by trial & error

1. Select 3 points randomly
2. Estimate parameters of P3P
3. Count the number of other points that support current hypothesis
4. Select best solution

실제 문제에선 feature matching이 정확하지 않을 수 있으므로 더 많은 포인트가 필요하다. p3p 알고리즘에 추가적으로 한 포인트의 정보를 더 이용하여 계산하는 것 보다, 세 점으로부터 추정한 pose로 3d points들을 projection 시켜 매칭된 point와의 reprojection error가 일정 threshold 이내로 들어오는 비율을 이용해 RANSAC을 활용하는 방법이 더 유용할 수 있다.