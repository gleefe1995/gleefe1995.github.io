---
layout: post
title: "cv::triangulatePoints 사용법"
date: 2021-04-28 18:05:30 -0400
categories: opencv
use_math: true
---



![image](https://user-images.githubusercontent.com/67038853/116400574-2a504900-a865-11eb-8684-ea6adb068c5f.png)



![image](https://user-images.githubusercontent.com/67038853/116400771-62578c00-a865-11eb-83c6-b35e1fe980e1.png)



projMatr1, projMatr2 -> first camera, second camera의 projection matrix, 즉 world 좌표를 각 camera로 projection해주는 3x4 homogeneous matrix이다. 

![image](https://user-images.githubusercontent.com/67038853/116401109-d003b800-a865-11eb-90ff-6095e2a58b11.png)

출처:<https://darkpgmr.tistory.com/82?category=460965>





![image](https://user-images.githubusercontent.com/67038853/116401154-db56e380-a865-11eb-9a21-55c444099a28.png)



projpoints1, projpoints2 -> 각 camera에서 뽑은 feature points들이다.

```
 Mat K = (Mat_<float>(3,3)<< focal, 0, pp.x,
                             0, focal, pp.y,
                             0,  0,   1);
                                                        
Mat Kd;
K.convertTo(Kd, CV_64F); 
Mat point3d_homo;

Mat Rt0 = Mat::eye(3, 4, CV_64FC1);
Mat Rt1 = Mat::eye(3, 4, CV_64FC1);
```

여기서 Rt0, Rt1에 extrinsic parameter를 넣어주면 된다.

```c++
for(int i = 0; i < point3d_homo.cols; i++) {
_p3h = point3d_homo.col(i); 
p3d2 = _p3h/_p3h.at<float>(3);
p3d2.convertTo(p3d22, CV_64F);
      
point.x = p3d22.at<double>(0);
point.y = p3d22.at<double>(1);
point.z = p3d22.at<double>(2);
}
```

이렇게 구한 point3d_homo matrix는 1x4 homogeneous coordinate로 되어있으므로 4번째 값을 1로 만들어주게 되면 월드 좌표계로 나타낼 수 있다.