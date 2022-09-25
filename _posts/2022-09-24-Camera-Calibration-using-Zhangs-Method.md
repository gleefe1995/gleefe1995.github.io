---
layout: post
title: "Camera Calibration using Zhang's Method"
date: 2022-09-24 00:01:30 -0400
categories:SLAM
use_math: true
---

---

Zhang's method는 camera intrinsic parameter만을 알아내는 것을 목적으로 하며, 평면으로 된 체커보드로 진행한다. 체커보드의 중심을 월드좌표계의 원점으로 하며, 모든 체커보드 위의 모든 좌표값들은 Z=0으로 볼 수 있다.

그럼 projection 식이 다음과 같이

![image](https://user-images.githubusercontent.com/67038853/192126525-18977c66-5ea9-43eb-bf13-eccf5fda1999.png)

모두 Z=0이라고 가정했으므로 R 행렬의 3번째 column을 지워주면 아래의 식처럼 나타낼 수 있다.

![image](https://user-images.githubusercontent.com/67038853/192126603-005accea-b9bc-45df-86f6-78267777999d.png)

여기서 DLT 방법을 적용해서 모든 관찰값에 따라 이러한 행렬을 쌓아서 H 행렬을 구할 수 있다. H 행렬로부터 K를 구해야하는데, 이 때 rotation matrix에서 3번째 column을 제거했으므로 더이상 QR decomposition을 적용할 수 없다. 

그렇다면, 각 성분을 K행렬의 식으로 나타내면 

$h_{1}=Kr_{1}, h_{2}=Kr_{2}$=> $r_{1}=K^{-1}h_{1}, r_{2}=K^{-1}h_{2}$로 나타낼 수 있다.

![image](https://user-images.githubusercontent.com/67038853/192126709-3f309948-2406-411f-afb3-f06924cb4143.png)

회전행렬의 orthonomality를 활용하면, 

$r1^{T}r_{2}=0, $\left\|r_{1} \right\|=\left\|r_{2} \right\|=1$ 두 식을 얻을 수 있다. 이 식에 각각 $r_{1},r_{2}$를 대입해주고 $B=K^{-T}K^{-1}$로 정의해주면

$h_{1}^{T}BH_{2}=0$, $h_{1}^{T}Bh_{1}-h_{2}^{T}Bh_{2}=0$ 두 식을 얻게 된다.

여기서 $B$ 행렬은, 상삼각행렬 K와 $K^{T}$를 곱하고 inverse를 취한것이므로 대칭행렬이다. 따라서, B행렬을 알면 Cholesky decomposition으로 K를 구할 수 있다.

#### Cholesky decomposition

숄레스키 분해는 $A=LL^{T}$에서 L이 상삼각행렬일 때, A를 알면 L을 구할 수 있다는 것이다.

![image](https://user-images.githubusercontent.com/67038853/192129241-8c47a782-e98b-422f-bcef-dc01554a873f.png)

위와 같이 하나하나 차례대로 구할 수 있다.

---

![image](https://user-images.githubusercontent.com/67038853/192129267-1dff9f10-fffb-4189-9579-a83883e87391.png)

b에대한 벡터로 묶어서 각 요소를 나타내면 최종적으로 오른쪽의 식을 얻을 수 있다.

![image](https://user-images.githubusercontent.com/67038853/192129330-ce5fe07d-b60d-465b-af69-4040ad707c60.png)

그래서 위에서 나타냈던 두 제약조건을 다시 v 행렬로 나타낼 수 있고 V matrix를 위와 같이 정의한다면 $Vb=0$ 이라는 식을 얻게 된다. 다양한 View에서 찍은 n개의 image들을 이용한다면 V는 $2n\times 6$ 형태의 matrix가 된다.

![image](https://user-images.githubusercontent.com/67038853/192129407-add2d258-bcb9-4e0c-a927-96519c59002f.png)

이제 DLT에서 풀었던 것과 같은 방식으로 해를 구하면 된다.

H matrix를 구하기 위해선 하나의 plane에 최소 4개의 점이 필요하고,(미지수 K 5개 + R 2개 + T 1개, 한 포인트당 x,y에 대해 두개의 식이 나오므로) B matrix는 6DOF를 가지므로 최소 3개의 다른 View에서 찍은 이미지가 필요하다.

