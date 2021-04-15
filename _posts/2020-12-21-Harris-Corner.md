---
layout: post
title: "Harris Corner 코드 분석"
date: 2020-12-27 14:42:30 -0400
categories: Harris Corner
use_math: true
---



# Harris Corner 코드 분석



### Edge나 직선보다 corner를 검출하는 이유

![image](https://user-images.githubusercontent.com/67038853/103173134-294de180-489c-11eb-904d-8e408d15650d.png)

그림 1

Edge나 직선은 영상 구조 파악 및 객체 검출에는 도움이 되지만, 영상 매칭에는 큰 도움이 되지 않는다. C와 같은 Edge는 강도와 위,아래 방향 정보만 가지므로 영상 매칭하기엔 정보가 부족하다.

D와 같은 부분은 모든 방향으로 밝기 변화가 적다. 

반면에 A,B와 같은 코너는 모든 방향에서 밝기 변화가 크기때문에 영상 매칭에 용이하다.

### 모라벡(Moravec)

영상 변화량(SSD: Sum of Squared Difference)

$E(u,v)=\sum_{y}\sum_{x}w(x,y)\cdot (I(x+u,y+v)-I(x,y))^2$

🔸 현재 화소에서 u,v 방향으로 이동했을 때의 밝기 변화량의 제곱

🔸 (u,v)를 (1,0), (0, 1), (-1, 0), (0, -1)의 4개 방향으로 한정

#### 문제점

---

🔹 0과 1의 값만 갖는 이진 윈도우 사용으로 노이즈에 취약

🔹 4개 방향으로 한정시켰기 때문에 45도 간격의 에지만 고려





### 해리스 검출

🔶 이진 윈도우 w(x,y) 대신 점진적으로 변화하는 가우시안 마스크 G(x,y) 사용

$E(u,v)=\sum_{y}\sum_{x}G(x,y)\cdot (I(x+u,y+v)-I(x,y))^2$

🔶 모든 방향에서 검출할 수 있도록 미분 도입

$I(x+u, y+v)\cong I(x,y)+vd_{y}(x,y)+ud_{x}(x,y)$

$E(u,v)\cong \sum_{y}\sum_{x}G(x,y)\cdot (vd_{y}(x,y)+ud_{x}(x,y))^2$

$E(u,v)\cong \sum_{y}\sum_{x}G(x,y)\cdot(vd_{y}+ud_{x})^2$

$=\sum_{y}\sum_{x}G(x,y)\cdot(v^2d_{y}^2+u^2d_{x}^2+2vud_{x}d_{y})$


$$
=\sum_{y}\sum_{x}G(x,y)\cdot (u,v)\begin{pmatrix}
d_{x}^2 & d_{x}d_{y} \\ 
d_{x}d_{y} & d_{y}^2 
\end{pmatrix}
\binom{u}{v}
$$

$$
=(u,v)M\binom{u}{v}, M=\sum_{y}\sum_{x}G(x,y)\begin{pmatrix}
d_{x}^2 & d_{x}d_{y} \\ 
d_{x}d_{y} & d_{y}^2 
\end{pmatrix}
$$



E(u,v)가 변화량의 합이므로 이 값이 크면 코너점이다. 이 때 E(u,v)값은 중간 행렬 M으로
판별한다. 

#### 고유값 분해 이용

M의 eigen value를 구하면 그에 대응되는 eigen vector를 얻을 수 있다. 

$Ax=\lambda x$

이 식이 의미하는 것은 vector x가 A 행렬에 의해 transformation 된 벡터가 x를 eigen value만큼

상수배한 벡터와 같다는 뜻이다. 

위의 식은 $(A-\lambda I)x=0$ 으로 다시 쓸 수 있는데, x가 trivial solution이 되지 않으려면

$det(A-\lambda I)=0$ 이어야 하고, 이 식을 통해 eigen value를 구한다.

이것을 이미지에 적용시켜보자.

두 개의 eigen value 중 더 큰 eigen value와, 그에 상응하는 eigen vector

방향으로 이미지의 변화가 크다고 말할 수 있다. 두 eigen value가 모두 크다면 코너, 둘중 하나만 크다면 edge, 둘 다 0에 가깝다면 flat이라고 할 수 있는 것이다.

하지만 이것을 계산하는 것은 복잡하기 때문에 determinant와 trace를 통해 코너 응답함수로 이용한다.

 

🔷 행렬식(determinant)과 대각합(trace)을 통해 코너 응답 함수로 이용한다.



$$
M=\begin{pmatrix}
d_{x}^2 & d_{x}d_{y} \\ 
d_{x}d_{y} & d_{y}^2 
\end{pmatrix} =\begin{pmatrix}
a & c \\
c & b
\end{pmatrix}
$$


$R=det(M)-k\cdot trace(M)^2=(ab-c^2)-k\cdot (a+b)^2$

#### 해리스 코너 검출 특징

---

🔶 영상의 평행 이동, 회전 변환에는 불변(invariant) 하는 특징이 있다.

🔶 어파인(affine) 변환이나 조명 변화에도 어느 정도 강인성을 가지고 있다.



### 전체 과정

---

1. 소벨 마스크로 미분 행렬 계산 ($d_{x},d_{y}$)
2. 미분 행렬의 곱 계산 ($d_{x}^2,d_{y}^2,d_{x}d_{y}$)
3. 곱 행렬에 가우시안 마스크 적용
4. 코너 응답함수 $C=det(M)-k\cdot trace(M)^2$ 계산
5. 비최대치 억제



### 소벨마스크 및 코너 응답 함수 계산

---

```
void cornerharris(Mat image, Mat& corner, int bSize, int ksize, float k)
{
	Mat dx, dy, dxy, dx2, dy2;
	corner = Mat(image.size(), CV_32F, Scalar(0));

	vector<int> a;


	Sobel(image, dx, CV_32F,1 ,0 , ksize); // 미분 행렬 - 수평 소벨 마스크
	Sobel(image, dy, CV_32F,0 ,1 , ksize); // 미분 행렬 - 수직 소벨 마스크
	multiply(dx, dx, dx2);				   // 미분 행렬 제곱
	multiply(dy, dy, dy2);
	multiply(dx, dy, dxy);
	Size msize(5, 5);
	GaussianBlur(dx2,dx2, msize, 0);      // 가우시안 블러 함수 적용
	GaussianBlur(dy2,dy2, msize, 0);
	GaussianBlur(dxy,dxy, msize, 0);

	for (int i = 0; i < image.rows; i++) {      // 코너 응답 함수 계산
		for (int j = 0; j < image.cols; j++)
		{
			float  a = dx2.at<float>(i,j);
			float  b = dy2.at<float>(i,j);
			float  c = dxy.at<float>(i,j);
			corner.at<float>(i, j) = (a*b-c*c)-k*(a+b)*(a+b);
		}
	}
}
```



### 코너 그리기

---

```
Mat draw_coner(Mat corner, Mat image, int thresh) // 임계값 이상인 코너 표시
{
	int cnt = 0; 
	normalize(corner, corner, 0, 100, NORM_MINMAX, CV_32FC1, Mat());

	for (int i = 1; i < corner.rows - 1; i++) {
		for (int j = 1; j < corner.cols - 1; j++)
		{
			float cur = (int)corner.at<float>(i,j) ; // 코너 응답값
			if (cur > thresh)                        // 임계값 이상이면
			{
				if (cur>corner.at<float>(i-1,j)&&    // 4개 방향 검사
					cur > corner.at<float>(i + 1, j)&&
					cur > corner.at<float>(i, j-1)&&
					cur > corner.at<float>(i, j+1))
				{
					circle(image, Point(j, i), 2, Scalar(255, 0, 0), -1);
					cnt++; // 반지름 2인 원 그리고 개수 +1
				}
			}
		}
	}
	cout << "코너수: " << cnt << endl; // 코너수 표시
	return image;
}
```



### 전체 코드 - Track bar를 이용하여 임계값 조절 하기

---

```
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

void cornerharris(Mat image, Mat& corner, int bSize, int ksize, float k)
{
	Mat dx, dy, dxy, dx2, dy2;
	corner = Mat(image.size(), CV_32F, Scalar(0));

	vector<int> a;


	Sobel(image, dx, CV_32F,1 ,0 , ksize); // 미분 행렬 - 수평 소벨 마스크
	Sobel(image, dy, CV_32F,0 ,1 , ksize); // 미분 행렬 - 수직 소벨 마스크
	multiply(dx, dx, dx2);				   // 미분 행렬 제곱
	multiply(dy, dy, dy2);
	multiply(dx, dy, dxy);
	Size msize(5, 5);
	GaussianBlur(dx2,dx2, msize, 0);      // 가우시안 블러 함수 적용
	GaussianBlur(dy2,dy2, msize, 0);
	GaussianBlur(dxy,dxy, msize, 0);

	for (int i = 0; i < image.rows; i++) {      // 코너 응답 함수 계산
		for (int j = 0; j < image.cols; j++)
		{
			float  a = dx2.at<float>(i,j);
			float  b = dy2.at<float>(i,j);
			float  c = dxy.at<float>(i,j);
			corner.at<float>(i, j) = (a*b-c*c)-k*(a+b)*(a+b);
		}
	}
}

Mat draw_coner(Mat corner, Mat image, int thresh) // 임계값 이상인 코너 표시
{
	int cnt = 0; 
	normalize(corner, corner, 0, 100, NORM_MINMAX, CV_32FC1, Mat());

	for (int i = 1; i < corner.rows - 1; i++) {
		for (int j = 1; j < corner.cols - 1; j++)
		{
			float cur = (int)corner.at<float>(i,j) ; // 코너 응답값
			if (cur > thresh)                        // 임계값 이상이면
			{
				if (cur>corner.at<float>(i-1,j)&&    // 4개 방향 검사
					cur > corner.at<float>(i + 1, j)&&
					cur > corner.at<float>(i, j-1)&&
					cur > corner.at<float>(i, j+1))
				{
					circle(image, Point(j, i), 2, Scalar(255, 0, 0), -1);
					cnt++; // 반지름 2인 원 그리고 개수 +1
				}
			}
		}
	}
	cout << "코너수: " << cnt << endl; // 코너수 표시
	return image;
}

Mat image, corner1, corner2;

void cornerHarris_demo(int  thresh, void*)
{
	Mat img1 = draw_coner(corner1, image.clone(), thresh);
	Mat img2 = draw_coner(corner2, image.clone(), thresh);

	imshow("img1-User harris", img1);
	imshow("img2-OpenCV harris", img2);
}

int main()
{
	image = imread("../image/harris_test.jpg", 1); // 이미지 경로 수정
	CV_Assert(image.data);

	int blockSize = 4;
	int apertureSize = 3;
	double k = 0.04;
	int  thresh = 20;
	Mat gray;

	cvtColor(image, gray, CV_BGR2GRAY);
	cornerharris(gray, corner1, blockSize, apertureSize, k); 	// 직접 구현 함수
	cornerHarris(gray, corner2, blockSize, apertureSize, k);	// OpenCV 제공 함수

	cornerHarris_demo(0, 0);
	createTrackbar("Threshold: ", "img1-User harris", &thresh, 100, cornerHarris_demo);
	waitKey();
}
```



### 결과

---

![image](https://user-images.githubusercontent.com/67038853/104084445-58942380-528a-11eb-8419-c775107479d8.png)

그림 2







### 참고

---

<https://bskyvision.com/21>