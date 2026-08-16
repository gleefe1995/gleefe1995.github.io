---
layout: post
title: "Triangulation for Image Pairs"
date: 2022-09-27 00:01:30 -0400
categories: SLAM
use_math: true
---

---

![image](https://user-images.githubusercontent.com/67038853/192810965-7c828c3c-3773-47e2-9a93-ea181765cbc1.png)

전혀 오차가 없는 상황에서는 F,G는 일치한다. 하지만 일반적으로는 일치하지 않는다. 이럴 경우, 두 꼬인 위치에 있는 직선에 수직인 가장 짧은 선분을 구하고, 그 중점을 구한다.

![image](https://user-images.githubusercontent.com/67038853/192811603-17a26d26-88ad-4063-a85b-929885abaaae.png)

- $\lambda$와 $\mu$는 scalar (unknown)
- r과 s는 방향벡터 (known)
  - calibration matrix와 pixel 좌표를 가지고 구할 수 있다
- p와 q는 카메라 중심 (known)
- r과 s를 구할 때 R를 transpose한 것은 R이 cam-to-world matrix이기 때문
  - 3d world 상의 좌표를 계산해야 하므로 transpose 해야 함

![image](https://user-images.githubusercontent.com/67038853/192812344-7e9bd51c-4342-46dc-a39a-2a1cea70ac03.png)

그림상에서 $f-g$ 벡터(FG)는 각각 r,s 방향벡터와 수직이어야 한다. 따라서 위의 constraint가 생기고, 두 방정식을 얻을 수 있다. 여기서 $\lambda$와 $\mu$를 구하고 중점을 구하면 H를 얻을 수 있다. 

 ![image](https://user-images.githubusercontent.com/67038853/192813700-b0304cdb-a620-4094-b446-84b9fc559def.png)

Ax=b 형태에서 풀어서 F,G를 구한 뒤 H를 구한다. (아마 least square?)

실제 상황에서는 다음과 같은 요인에 의해 오차가 생길 수 있다.

1. 카메라 캘리브레이션에서 intrinsic parameter의 오차
2. Optical flow, descriptor matching에서 오차
3. Pixel Plane에서 Normal Plane으로 갈 때 오차
4. Rotation, Translation 추정 오차
5. 두 이미지의 base line이 충분히 확보되지 않았을 때. Depth에 특히 더 큰 에러가 발생

#### Stereo Normal Case

![image](https://user-images.githubusercontent.com/67038853/192815112-18743608-9df6-456e-b7f1-8ed526baa32a.png)

Stereo 셋업이기 때문에 이미지 상의 두 Point는 y와 z 좌표 값은 동일하고 x만 다르다.

![image](https://user-images.githubusercontent.com/67038853/192816914-93cb623c-c32b-419c-8851-7d939ffd2b6c.png)

P는 3d point, 카메라 중심 $O',O''$, pixel plane이 위와 같이 나타나 있다. Base Line이 B, c가 focal length라 할 수 있다. 삼각형의 닮음비에 의해 Z를 위와 같이 나타낼 수 있다. 

![image](https://user-images.githubusercontent.com/67038853/192817559-3663a8a2-49c0-4daa-b517-cd53f28b9bc5.png)

또 pinhole 카메라 모델에서 $x' = c*X/Z$이므로 X 좌표도 구할 수 있다.

![image](https://user-images.githubusercontent.com/67038853/192817768-d0b8b5fe-036d-414a-b664-772cc83184d1.png)

카메라를 세워서 봤을 때, 역시 닮음비를 이용해서 Y좌표를 구한다. 이 때, Y가 일치하지 않을 수 있으므로 두 y좌표를 평균내서 구해준다.

![image](https://user-images.githubusercontent.com/67038853/192817994-71b83e90-9285-4b57-8326-3809ea4c18de.png)

#### Quality of 3d points

![image](https://user-images.githubusercontent.com/67038853/192826397-0aa57758-b6a3-4cfc-a2df-10bc6a990eba.png)

우리가 구한 XYZ 좌표에서 상수부분을 M으로 치환하면 다음과 같다. pixel의 measure를 $\sigma$로 표현하면 x는 그대로 쓰고, y의 경우 두 좌표를 평균내서 사용했으므로 위와 같다. 

![image](https://user-images.githubusercontent.com/67038853/192827003-1c982889-3a87-4165-9ac0-eff7c9be8799.png)

Z에 대해서도 다음과 같이 나타낼 수 있다. 여기서 알 수 있는 점은 parallax p가 크면 클수록 Z의 품질은 좋아진다라는 것이다.

