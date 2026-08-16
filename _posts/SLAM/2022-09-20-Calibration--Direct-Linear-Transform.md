---
layout: post
title: "Calibration : Direct Linear Transform"
date: 2022-09-20 00:01:30 -0400
categories: SLAM
use_math: true
---

Calibration의 목적은 주어진 2D image point와 3d wrold point pair를 가지고 camera의 extrinsic + intrinsic parameter를 알아내는 것이다. 

![image](https://user-images.githubusercontent.com/67038853/191521342-e5b41176-b2b5-4f52-865e-452e35a321ce.png)

다음과 같이 $x = PX$로 projection matrix를 나타낼 수 있는데, transformation matrix $P$는 5개의 intrinsic, 6개의 extrinsic parameter를 가지고 있으며 총 11개의 unknown parameter로 구성되어 있다. 

![image](https://user-images.githubusercontent.com/67038853/191522330-b4e9c547-0f61-4c89-81a1-ecda44e2e78f.png)

homogeneous coordinate으로 쓰게되면 다음과 같이 쓸 수 있고,

![image](https://user-images.githubusercontent.com/67038853/191522484-b7eebc9b-47f6-4aaa-90fb-6d8371762d2d.png)

1개의 2d point, 3d point pair 당 위의 두개의 식을 얻을 수 있다.

우리가 모르는 건 P matrix의 unknown paramter 11개 이므로, 최소 6개의 point가 필요하다는 것을 알 수 있다. 이렇게 구하는 것을 Direct Linear Transform이라고 부른다.

![image](https://user-images.githubusercontent.com/67038853/191523740-233e1ea2-b4a5-4be2-9bca-0389c0a735f3.png)

$P$ matrix에서 각 행을 A,B,C로 치환한다면

![image](https://user-images.githubusercontent.com/67038853/191524041-f047968b-51e8-4713-ac2f-a0ac6cb931a1.png)

다음과 같이 쓸 수 있게 되고, 

![image](https://user-images.githubusercontent.com/67038853/191524154-f399fb91-0fbe-4d8a-892a-b7a864f359b6.png)

위와 같은 식으로 표현할 수 있다.

![](https://user-images.githubusercontent.com/67038853/191524409-27de5a45-58ed-463c-b9fc-ffc540803c27.png)

$p=P^{T}$로 정의하고 $a^T_{x_{i}},a^T_{y_{i}} $를 위와 같이 정의한다면 

![image](https://user-images.githubusercontent.com/67038853/191525121-1ba4c625-10f2-4195-be5b-bbe03023b697.png)

다음과 같이 표현되게 된다. 

![image](https://user-images.githubusercontent.com/67038853/191525532-fd23e075-26e7-4bec-991e-09014e4891ca.png)

즉 우리는 1개의 pair당 x,y에 관한 식 두개가 나오므로 I개의 point에 대하여 위와 같은 형태의 matrix 식을 얻을 수 있게 되고, 이 식은 $Mp$가 최대한 0에 가깝도록 하는 M matrix의 right null space를 구하는 방정식으로 바뀐다.

![image](https://user-images.githubusercontent.com/67038853/191526187-21626a95-c555-4a86-8d05-acaea2a30768.png)



![image](https://user-images.githubusercontent.com/67038853/191526399-54ea7d92-b04a-48f2-8fd7-94726413e35d.png)

하지만 모든 point가 같은 plane에 있을 경우(모두 Z=0이라고 가정) rank deficiency가 발생하여 해를 구할 수 없게 된다.

![image](https://user-images.githubusercontent.com/67038853/191527747-c6d93773-d501-4770-ad18-db79f513a7ca.png)

그렇다면 이렇게 구한 $P$ matrix로 부터 어떻게 $K,R,X_{O}$ matrix들을 구할까?

P matrix를 H,h matrix로 나타내게 되면 위와 같다.

우선 camera center$X_{O}=-H^{-1}h$는 다음과 같이 나타낼 수 있다.

![image](https://user-images.githubusercontent.com/67038853/191528500-c073ed71-f450-44b5-9300-074b2886b66b.png)

Rotation matrix와 K matrix는 $H^_{-1}$ matrix를 QR decomposition을 이용해서 분해하여 얻게된다. Q는 orthogonal matrix, R은 상삼각행렬로, intrinsic matrix가 상삼각행렬이기 때문에 가능하다. 이러한 성질은 inverse matrix에서도 유지되기 때문에 위와 같은 식으로 분해가 가능하다.

![image](https://user-images.githubusercontent.com/67038853/191529253-0e245869-3a35-48a0-a305-64452eb90e37.png)

H는 homogenous matrix이기 때문에, 이렇게 구한 K matrix 또한 homogenous matrix이다. 따라서 K를 normalize해주기 위해 (3,3) 성분으로 나눠주게 되면 최종적으로 K를 구할 수 있게 된다.