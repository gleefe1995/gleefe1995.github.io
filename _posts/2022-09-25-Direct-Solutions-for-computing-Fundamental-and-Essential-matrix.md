---
layout: post
title: "Direct Solutions for computing Fundamental and Essential matrix"
date: 2022-09-25 00:01:30 -0400
categories: SLAM
use_math: true
---

---

![image](https://user-images.githubusercontent.com/67038853/192197342-3b00de04-baa7-4da3-9f30-fa616c9f94b8.png)

$X'$은 오른쪽 카메라 좌표계에서 바라본 $X'$의 좌표이고, $R,t$는 왼쪽 카메라 좌표계->오른쪽 카메라 좌표계로 변환하는 rotation 행렬, translation 벡터이다.

$\overline{o'X'}, \overline{o'o}$ 는 각각 오른쪽 카메라 좌표계에서 바라본 $X', o$ 벡터이다. 이 두 벡터를 외적하면 $X'\times t = [t]^{T}_{X}X'$으로 표현할 수 있다.

![image](https://user-images.githubusercontent.com/67038853/192197873-50beadef-da28-47dd-91bd-9e4e73ef2b25.png)

$\overline{oX}=X'-o=(RX+t)-(Ro+t)=RX$(오른쪽 카메라 좌표계에서 바라본 oX 벡터, X는 왼쪽 카메라 좌표계에서 바라본 X 좌표)이고,

$(\overline{o'X'}\times \overline{o'o})\cdot \overline{oX}=0$이므로 $[t]_{X}R=E$로 둔다면 $X'^{T}EX=0$이라는 식을 얻을 수 있다. 이 때 E를 Essential Matrix라고 한다.

Fundamental matrix는 카메라 파라미터까지 포함한 두 이미지의 실제 픽셀 좌표 사이의 기하학적 관계를 표현하는 행렬이다.

$x'_{img}^{T}Fp_{img}=0$ => $E=K^{T}FK$(동일한 카메라일 경우)로 표현할 수 있다.

![image](https://user-images.githubusercontent.com/67038853/192199359-17f50091-f73a-41d9-ae13-42f97be51c85.png)

F matrix를 구하기 위해서 다음과 같은 식을 풀면 된다. 매칭된 feature 한 쌍당 1개의 식이 나오게 된다.

![image](https://user-images.githubusercontent.com/67038853/192199755-8346833d-1cc2-4543-a0d0-6dfae68e8694.png)

$a_{n}^T$를 위와 같이 두면,

![image](https://user-images.githubusercontent.com/67038853/192199876-c7b3c489-f987-4477-b8fc-abfc59e58945.png)

N개의 point에 대해서 A를 a 행렬에 관해 나타낼 수 있고, $Af=0$을 풀게 되면 우리는 F matrix를 풀 수 있게 되는 것이다. 여기서 f는 right-sigular vector corresponding to a singular value of A that is zero(or smallest sigular value)이다. F matrix는 homogeneous 하기 때문에, ($F_{33}=1$) rank가 최대 8이다. 따라서, 적어도 8 point가 필요하다. F matrix가 homogeneous한 이유는, 우리가 homogeneous coordinate을 사용하고 있기 때문에 $F_{33}$의 값을 풀 수 없기 때문이다.

#### Enforcing Rank 2

F matrix는 singular 해야하고, rank 2를 가진다. F는 E로부터 구하는데, E는 

![image](https://user-images.githubusercontent.com/67038853/192209453-3b6436e5-cdd4-4d91-9d8b-b56be1d07a63.png)

다음의 translation vector로 skew-matrix를 구성한 matrix로 구한다. $T_{x}$ matrix는 한 컬럼이 다른 두 컬럼의 linear combination으로 표현 가능하기 때문에 rank가 2인데, 이로부터 F를 구하므로 F도 rank가 2인 것이다. 만약 F matrix가 일반적으로는 rank 2를 가지지 않는데, 이를 위해서 rank-2 constraint를 주어 rank=2가 되도록 해야 한다. $||F-F'||, det\ F'=0$인 $F'$으로 교체하는 것이다. 

![image](https://user-images.githubusercontent.com/67038853/192242394-ef606bc4-424e-4a29-b245-49c5e45e7e04.png)

한번더 SVD를 하여 가장 작은 singular value값을 0으로 두어 rank를 2로 강제할 수 있다. 

1. Homogeneous, 2. F의 rank=2 이므로 F의 DOF는 7이다. 왜냐면 1번에 의해 DOF가 8이 되고, 2에 의해서 F의 determinant가 0이 되는데, $det F = (f1*f5*f8)+(f2*f6*f7)+(f3*f4*f8)-(f3*f5*f7)-(f2*f4*f9)-(f1*f6*f8)=0$에서  $f9=a*f8+b, f9=c*f8+d$ 두 식을 이용해서 f8,f9를 구할 수 있다. a,b,c,d는 f1~f7에 의해서 결정된다. 따라서 7개의 parameter만이 남게 되는것이다.

![image](https://user-images.githubusercontent.com/67038853/192242676-31a4aca5-77df-48c8-a977-1eb5d6fbef23.png)

또한, coordinate system에서 각 값의 차이가 크게되면 수치적으로 불안정하게 되므로, normalization을 진행한다. 

#### Essential Matrix

![image](https://user-images.githubusercontent.com/67038853/192243453-edc45fd7-b224-4478-9663-6381ad9788e2.png)

Essential Matrix도 비슷하게 진행한다.

![image](https://user-images.githubusercontent.com/67038853/192245711-6835a4ce-18fa-4d20-9b31-a0a44cb104e6.png)

E를 다음과 같이 SVD를 할 수 있는데, singular하기 때문에 singular value를 다음과 같이 치환할 수 있다.

![image](https://user-images.githubusercontent.com/67038853/192246115-92ea3778-e1ea-4c6f-9e7a-273c459c3d10.png)

여기서 가운데 행렬을 ZW의 곱으로 치환하게 되면,

![image](https://user-images.githubusercontent.com/67038853/192246221-d6069ce7-8e5b-4295-a9a0-f46a22dba161.png)

다음과 같고, $UZU^T, UWV^T$ 두 행렬로 나누어 각각 translation vector의 skew-symmertric 형태, rotation 행렬로 구할 수 있다.

![image](https://user-images.githubusercontent.com/67038853/192246473-2c6547e0-72d1-44e0-8e51-d58ba3bbf018.png)

앞서 구한 ZW는 다음의 네가지 경우가 있는데, 이에 따라 기하학적으로 네 가지 경우가 있다.

![image](https://user-images.githubusercontent.com/67038853/192246613-58a5a73e-f340-4074-830f-a9c64f149fde.png)

우리는 이 네 가지 경우에 대해서, 모든 point들이 카메라 앞쪽에 위치한 경우를 찾으면 된다. 이러한 경우가 실제 상황에서 일어날 법한 경우이기 때문이다.

![image](https://user-images.githubusercontent.com/67038853/192246900-c094ff3a-f8cf-4aeb-9524-0078bcfab2c4.png)

7개 이상의 point에 대해서는 8-point algorithm, F는 7개의 point로, E는 5개의 point로 camera pose를 추정할 수 있다. overdetermined case에 대해서는 handling 할 수 없으므로 outlier 제거를 위해 RANSAC 알고리즘 등을 사용해야 한다.