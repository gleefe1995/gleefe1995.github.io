---
layout: post
title: "Lie group and Lie algebra"
date: 2021-04-18 18:05:30 -0400
categories: math
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
SO(3)=\{R\in\real^{3x3}|RR^{T}=I,det(R)=1\}
$$


추가적으로, SE(3)는 rotation 행렬과 translation vector를 함께 나타낸 homogeneous transformation matrix의 그룹이다.
$$
SE(3)=\{A|A=\begin{bmatrix}
R &T \\ 
0 & 1
\end{bmatrix},R\in\real^{3x3}|RR^{T}=I,det(R)=1\}
$$


#### Lie algebra

SO(3) group의 Lie algebra는 so(3)라고 표기하며 skew-symmetric 행렬이다. 
$$
so(3)=\{\omega^{\wedge}=\begin{pmatrix}
0&-\omega_{3}&\omega_{2}\\ 
\omega_{3}&0&-\omega_{1} \\ 
-\omega_{2}&\omega_{1}&0 
\end{pmatrix}\in \real^{3x3}| \omega=\begin{bmatrix}
\omega_{x}\\
\omega_{y}\\
\omega_{z}
\end{bmatrix}\in \real^{3}\}
$$
어떤 3차원 벡터 $\omega$에 hat을 붙이면, 그 벡터와의 외적곱을 나타내는 행렬로 나타낼 수 있다.

skew-symmetric 행렬은 $A^{T}=-A$를 만족하며, 어떤 벡터와의 cross product matrix와도 같다.

SO(3)의 Lie algebra는 간단하게 말하면, 미소 각도 변화를 나타내는 rotation matrix들을 나타낸다. 여기서, $\omega$는 Euler vector를 나타내는데,   







참조 : http://stillbreeze.github.io/Optimization-On-a-Manifold/

https://edward0im.github.io/mathematics/2020/05/01/lie-theory/

https://www.youtube.com/watch?v=_uLRPqjdHjk&t=518s&ab_channel=SLAMKR