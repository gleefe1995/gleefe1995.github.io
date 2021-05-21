---
layout: post
title: "Lie group and Lie algebra"
date: 2021-04-18 18:05:30 -0400
categories: Math
use_math: true
---

Lie group 과 Lie algebra 공부

### SO(3) - 3D rotation group

모든 3x3 orthogonal matrix들의 그룹을 O(3)라고 하고 $R^{T}R=RR^{T}=I$를 만족하며 det(R)=1을 만족하는 orthogonal matrix의 group을 SO(3)로 표기한다. SO(3)는 3차원 Special orthogonal의 약자이며 3D rotation group을 의미한다.  또한 이 rotation group은 manifold(다양체)의 구조를 갖고 있으므로 미분 가능하여 Lie group이라고 볼 수 있다.  

$R^{T}R=I$는 rotation행렬 R의 각 column vector들의 곱과 orthonormal한 성질을 생각해보면 쉽게 증명이 가능하다.


$$
R^{T}=\begin{pmatrix}
-r1-\\ 
-r2-\\ 
-r3-
\end{pmatrix} \\
R=\begin{pmatrix}
|&|&| \\ 
r1&r2&r3\\
| & |  &| 
\end{pmatrix}
\\ \therefore R^{T}R=\begin{pmatrix}
-r1-\\ 
-r2-\\ 
-r3-
\end{pmatrix}\begin{pmatrix}
|&|&| \\ 
r1&r2&r3\\
| & |  &| 
\end{pmatrix}=\begin{pmatrix}
r1\cdot r1 & r1\cdot r2 & r1\cdot r3\\ 
r2\cdot r1 & r2\cdot r2 &r2\cdot r3 \\ 
r3\cdot r1 & r3\cdot r2 &r3\cdot r3 
\end{pmatrix}=\begin{pmatrix}
1&0&0\\ 
0&1&0 \\ 
0&0&1 
\end{pmatrix}
$$




또, orthogonal matrix R에 대해서 $det(R^{T})=det(R)$이 성립하고, $RR^{T}=I$이므로 $det(R^{2})=1$이 되는데, 여기서 determinant가 1인 subgroup을 special orthogonal group, SO(3)라고 표기한다.

한 줄로 표기하자면,


$$
SO(3)=\{R\in \mathbb{R}^{3x3}|RR^{T}=I,det(R)=1\}
$$



추가적으로, SE(3)는 rotation 행렬과 translation vector를 함께 나타낸 homogeneous transformation matrix의 그룹이다.


$$
SE(3)=\{A|A=\begin{bmatrix}
R &T \\ 
0 & 1
\end{bmatrix},R\in\mathbb{R}^{3x3}|RR^{T}=I,det(R)=1\}
$$



#### Lie algebra

SO(3) group의 Lie algebra는 so(3)라고 표기하며 skew-symmetric 행렬이다. 


$$
so(3)=\{[\omega]=\begin{pmatrix}
0&-\omega_{3}&\omega_{2}\\ 
\omega_{3}&0&-\omega_{1} \\ 
-\omega_{2}&\omega_{1}&0 
\end{pmatrix}\in \mathbb{R}^{3x3}| \omega=\begin{bmatrix}
\omega_{x}\\
\omega_{y}\\
\omega_{z}
\end{bmatrix}\in \mathbb{R}^{3},||\omega||=1\}
$$


어떤 3차원 벡터 $\omega$에 브라켓을 씌운 형태 [$\omega$]는 그 벡터와의 외적곱을 나타내는 행렬로 나타낼 수 있다.

skew-symmetric 행렬은 $A^{T}=-A$를 만족한다.

또한, 케일리-해밀턴 정리에 의해서

$det(Is-[\omega])=s^3+s=0$을 만족하므로 s에 [w]를 넣어주면 $[w]^3=-[w]$을 유도할 수 있다.



$\omega$는 크기가 1인 각속도 벡터를 나타낸다.

![image](https://user-images.githubusercontent.com/67038853/115180943-bcdc4400-a111-11eb-9385-19f88f7d453b.png)



예를 들면, z축으로 각속도 $\pi/2$로 회전하는 회전벡터는


$$
\begin{bmatrix}0\\0\\1\end{bmatrix} \cdot \frac{\pi}{2}=\begin{bmatrix}
0\\ 
0\\ 
\frac{\pi}{2}
\end{bmatrix}
$$


다음과 같이 각속도가 [0 0 1] 인 벡터에 $\theta$를 곱하여 표현할 수 있다. 



왜 갑자기 SO(3)를 소개한 뒤에 Lie algebra가 나왔을까? 앞에서 SO(3)는 미분가능한 manifold의 형태를 하고 있다고 했다. 

![image](https://user-images.githubusercontent.com/67038853/115179245-d7acb980-a10d-11eb-829b-873f1ec2cef0.png)

예를 들어 설명하면, manifold의 모양은 3차원의 구 모양을 하고 있지만, 한 점에서 tangent space를 그리면 평면으로 인식할 수 있다. rotation group으로 이루어진 manifold 위의 한 점이 $\theta$만큼 회전하는 rotation matrix라고 해보자. 이 때 t=0 근처에서 테일러 전개로 회전 행렬을 나타낼 수 있는데, 이 때 사용되는 matrix가 $[\omega]$이고, SO(3)와 so(3)는 서로 exponential, logarithm mapping 으로 대응시킬 수 있다. 



그렇다면 rotation vector $\omega$의 skew-symmetric 형태인 $[\omega]=\begin{pmatrix}
0&-\omega_{3}&\omega_{2}\\\\ 
\omega_{3}&0&-\omega_{1} \\\\ 
-\omega_{2}&\omega_{1}&0 
\end{pmatrix}$ 는 어떻게 나왔을까?

$[\omega]$는 외적의 행렬 형태라고 했다. 이 어떤 벡터에 이 행렬을 곱하면 $w\times\vec{a}$를 연산하는 것인데, 고등학교 물리 시간에 배웠던 원운동에서 $v=r\omega$를 생각해보면 된다.

![image](https://user-images.githubusercontent.com/67038853/115186339-793b0780-a11c-11eb-8522-f1df7a4aeb34.png)

다음에서 속도 벡터 $\dot{p}(t)=\vec{w}\times p(t)$라는 것을 증명했다. 즉 $[\omega]$와 어떤 벡터를 곱하면, 속도 벡터가 나온다. 우리는 이제 이 skew-symmetric 행렬을 가지고 rotation matrix를 표현할 것이다.



### Exponential coordinates

$\dot{p}(t)=\vec{w}\times p(t)=[\omega]p$ 으로 표현이 가능하다. 여기서 1차 미분 방정식

$\dot{x}=Ax$를 풀면 $x=x(0)e^{At}$로 풀 수 있다는 것을 이용하면, $p(t)=e^{[\omega] t}p(0)$  으로 

나타낼 수 있고, $|\omega|=1$이면, 즉 각속도가 1rad/sec 이면, 시간 t만큼 회전한 각도 $\theta=t$가 

되어 서로 interchangable하다. 따라서, $p(\theta)=e^{[\omega] \theta}p(0)$와 같이 쓸 수 있다.



$e^{[\omega] \theta}$는 t=0 근처에서 $e^{[\omega] \theta}=I+[\omega]\theta+[\omega]^2\frac{\theta^2}{2!}+ \cdots$과 같이 나타낼 수 있는데, 여기서 $[\omega]^3=-[\omega]$를 이용해서 정리하면,

$R=e^{[\omega] \theta}=I+sin\theta[\omega]+(1-cos\theta)[\omega]^2$과 같이 나타낼 수 있다. 이렇게 회전 행렬을 so(3) matrix인 $[\omega]$로 나타낼 수 있게 된다. 

![image](https://user-images.githubusercontent.com/67038853/115198053-9d064980-a12c-11eb-9f77-ed892caf43a6.png)****



이 식은 rodrigues' rotation formula와 같은 것을 알 수 있다. rodrigues' rotation formula를 유도할 때는 회전축의 unit vector k와 각도 $\theta$를 이용해서 회전행렬을 나타냈는데, 여기선 각속도 벡터 $\omega$를 이용하여 나타낸다. 이 부분에서 약간 헷갈렸는데, 각속도 벡터의 크기가 1이므로 $\theta$를 움직였다 = $\theta$초 만큼 각속도 1rad/sec로 움직였다라는 의미가 되어 $[w]t=[w]\theta$로 쓸 수 있어  interchangable하게 바꿔 쓸 수 있기 때문에 가능한 점인 것 같다.



$R=e^{[\omega] \theta}$로 회전 행렬을 나타낼 수 있는 것은,

$e^{[\omega] \theta}$을 풀어 써보면, 



![image](https://user-images.githubusercontent.com/67038853/115193576-4f3b1280-a127-11eb-8b79-23dabbd788b7.png)



다음과 같이 쓸 수 있다. 이 행렬이 SO(3)의 조건을 만족하기 때문에 회전행렬로 쓸 수가 있다. 

이 때, $[\omega]\theta$를 logarithm of R이라고 한다. 









참조 : <http://stillbreeze.github.io/Optimization-On-a-Manifold/>

<https://edward0im.github.io/mathematics/2020/05/01/lie-theory/>

<https://www.youtube.com/watch?v=_uLRPqjdHjk&t=518s&ab_channel=SLAMKR>