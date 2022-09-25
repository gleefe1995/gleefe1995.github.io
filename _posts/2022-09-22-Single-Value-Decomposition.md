---
layout: post
title: "Single Value Decomposition"
date: 2022-09-22 00:01:30 -0400
categories: Math
use_math: true
---

---

---

#### 고유값과 고유벡터

---

![image](https://user-images.githubusercontent.com/67038853/191995158-d75456b4-8b87-47b4-acce-57c6ec1b6a80.png)

우선, 행렬 A는 $\lambda_{1}a_{1},\cdots,\lambda_{n}a_{n}$의 열벡터로 이루어 져 있다고 볼 수 있고, 위와 같이 열벡터와 scale factor로 이루어진 diagonal matrix의 구성으로 인수분해 할 수 있다.

고유값과 고유벡터란를 구하는 다음의 의미를 가진다.

""어떤 벡터 x에 선형 변환 A를 곱하였을 때, 원래 벡터에 크기만 곱한 벡터가 되는 벡터 x와 그 크기는 얼마인가?"

이러한 벡터들을 여러 개 구했다면, $Av_{i}=\lambda_{i}v_{i} \ for \ i=1,2,\cdots,n$ 과 같이 식을 표현할 수 있고, 이러한 고유벡터들을 열벡터로 가지는 행렬을 V, scale factor들로 이루어진 diagonal matrix를 $\lambda$라고 정의해보자.

식으로 나타내면 $AV=V\lambda$로 쓸 수 있고, $A=V\lambda V^{-1}$로 A를 decomposition 할 수 있다.

그래서 선형 변환 A에 대해서 이러한 벡터 x와 그 크기를 왜 구할까?

A의 제곱을 하거나 여러 연산을 할 때, 편리함을 위해서라고 할 수 있을 것 같다.



#### Single Value Decomposition

SVD는 다음과 같은 의미를 가진다.

"직교하는 벡터 집합에 대하여, 선형 변환 후에 그 크기는 변하지만, 여전히 직교하는 직교 집합은 무엇인가?"

SVD란, $m\times n$ 차원의 행렬 A에 대하여 $A = U\Sigma V^{T}$

와 같이 행렬을 분해하는 것을 의미한다. 이 때 U, V는 각각 $m\times m$, $n\times n$ 차원의 orthogonal matrix, $\Sigma$는 $m\times n$차원의 diagonal matrix이다.

#### UV 계산법

$$
A=U\Sigma V^{T} \\
A^{T} = V\Sigma^{T}U^{T} \\
AA^{T} = U\Sigma \Sigma^{T}U^{T} \\
AA^{T} = U\Sigma^{2}U^{T}
$$

여기서 EVD를 생각해보면, $\Sigma^{2}$ 행렬은 $AA^{T}$ 행렬의 고유값 행렬이다. 따라서 U 행렬은 고유벡터의 집합이 된다. V 행렬도 마찬가지로 계산할 수 있다.

이 때 scaling factor는 정보량의 크기를 의미한다. 

=> 정보량의 크기에 따라 A를 여러 layer로 쪼개준다.

=> 이미지로 따지면, 정보량의 크기가 크다면 대략적인 윤곽을 나타내고, 정보량의 크기가 작다면 이미지의 디테일한 부분을 나타낸다고 볼 수 있다.



#### SVD 활용

$AX=B$의 선형 연립방정식이 있을 때, A의 역행렬이 존재한다면 바로 해를 구할 수 있다. 하지만 데이터들이 상당히 많고 노이즈가 껴있는 경우, 역행렬이 당연하게도 존재하지 않는 경우가 많은데, 이 때 사용하는 방법이 바로 A의 pseudo inverse $A^{+}$를 이용하는 것이다. $X=A^{+}B$로 X를 구하면, least square로 근사하는 포물선을 구할 수 있다. A를 SVD한다면 $A=U\Sigma V^{T}$로 나타낼 수 있는데, 이 때 $A^{+}=V\Sigma^{+}U^{T}$로 계산된다. 이 때 $\Sigma^{+}$는, 0이 아닌 singular value들을 역수를 취한 후 transpose를 시킨 행렬이다. 만약 모든 singular value들이 양수이고,$m>=n$이면, $A^{+}A$는 $n\times n$ 단위 행렬이 되고, 만일 0인 singular value가 포함되면 어떤 순서로 곱해도 단위행렬이 나오지 않는다.

#### Calibration에서..

![image](https://user-images.githubusercontent.com/67038853/191526187-21626a95-c555-4a86-8d05-acaea2a30768.png)

calibration에서 최종적으로 $Mp\doteq 0$이라는 선형 연립방정식을 풀어서 p matrix를 구해야하는데, 이 문제는 M matrix의 null space를 구하는 것과 같다. 우변이 0이 아닐 경우에는, 이 때 0에 가장 가깝도록 하는 p 벡터는 M 행렬의 right singular vector $p=v_{12}$이다. 왜냐면 Singular value는 diagonal에서 내려갈수록 0에 가까워지는데, 이 0에 가장 가까운 Singular value와 곱해지는 것이 right singular vector이기 때문이다.
