---
layout: post
title: "Canny edge 검출"
date: 2021-02-02 21:08:30 -0400
categories: Cannyedge
use_math: true
---

# Canny edge 검출

  캐니 에지 검출 - 여러 단계의 알고리즘으로 구성된 에지 검출 방법

1. 블러링을 통한 노이즈 제거 (가우시안 블러링)
2. 화소 기울기(gradient)의 강도와 방향 검출 (소벨 마스크)
3. 비최대치 억제(non-maximum suppression)
4. 이력 임계값(hysteresis threshold)으로 에지 결정





### 1. 블러링 - 5x5 크기의 가우시안 필터 적용

🔷 불필요한 잡음 제거

🔷 필터 크기는 변경 가능



### 2. 화소 기울기(gradient) 검출

🔷 가로 방향과 세로 방향의 소벨 마스크로 회선 적용

🔷 회선화 된 행렬로 화소 기울기의 크기(magnitude)와 방향(direction) 계산

🔷 기울기 방향은 4개 방향(0,45,90,135)으로 근사하여 단순화



### 3. 비최대치 억제(non-maximum suppression)

🔷 기울기의 방향과 에지의 방향은 수직

![image](https://user-images.githubusercontent.com/67038853/106568794-18694d80-6577-11eb-916b-8ed99bf998c3.png)

[그림 1]

🔷 기울기 방향에 따른 이웃 화소 선택

![image](https://user-images.githubusercontent.com/67038853/106568868-3636b280-6577-11eb-9447-a1cd4a3bbbbf.png)

[그림 2]



### 4. 이력 임계값 방법(hysteresis thresholding)

🔷 두 개의 임계값(Thigh, Tlow)를 사용해 에지 이력 추적으로 에지 결정

🔷 각 화소에서 높은 임계값보다 크면 에지 추적 시작

​      -> 추적하면 추적하지 않은 이웃 화소로 낮은 임계값보다 큰 화소를 에지로 결정

![image](https://user-images.githubusercontent.com/67038853/106569081-785ff400-6577-11eb-80a7-0c92f03f3c51.png)

[그림 3]





### 코드

---



#### 방향 계산 함수

```c++
void calc_direct(Mat Gy, Mat Gx, Mat& direct)
{
	direct.create(Gy.size(), CV_8U);

	for (int i = 0; i < direct.rows; i++){
		for (int j = 0; j < direct.cols; j++){
			float gx = Gx.at<float>(i, j);
			float gy = Gy.at<float>(i, j);
			int theat = int(fastAtan2(gy, gx) / 45);
			direct.at<uchar>(i, j) = theat % 4;
		}
	}
}
```

direct 행렬은 각 포인트의 기울기 방향을 나타내는 행렬이다.

Gy, Gx는 Sobel 마스크 함수로 각각 y방향, x방향으로 미분한 행렬이다.

모든 포인트에 대해서 gx, gy를 얻어 fastAtan2(gy,gx)를 계산하면 (gy,gx) 벡터의 기울기가

0~360도 사이로 표시된다. 이것을 4방향으로 보기위해 45도로 나눈다.

이 때, 우리는 0~180도의 기울기만 필요하기 때문에 %4를 해주면

각 방향이 0,1,2,3 으로 나뉘게 된다.





#### 비최대치 억제 

```c++
void supp_nonMax(Mat sobel, Mat  direct, Mat& dst)		// 비최대값 억제
{
	dst = Mat(sobel.size(), CV_32F, Scalar(0));

	for (int i = 1; i < sobel.rows - 1; i++) {
		for (int j = 1; j < sobel.cols - 1; j++) 
		{
			int   dir = direct.at<uchar>(i, j);				// 기울기 값
			float v1, v2;
			if (dir == 0) {			// 기울기 방향 0도 방향
				v1 = sobel.at<float>(i, j - 1);
				v2 = sobel.at<float>(i, j + 1);
			}
			else if (dir == 1) {		// 기울기 방향 45도
				v1 = sobel.at<float>(i + 1, j + 1);
				v2 = sobel.at<float>(i - 1, j - 1);
			}
			else if (dir == 2) {		// 기울기 방향 90도
				v1 = sobel.at<float>(i - 1, j);
				v2 = sobel.at<float>(i + 1, j);
			}
			else if (dir == 3) {		// 기울기 방향 135도
				v1 = sobel.at<float>(i + 1, j - 1);
				v2 = sobel.at<float>(i - 1, j + 1);
			}

			float center = sobel.at<float>(i, j);
			dst.at<float>(i, j) = (center > v1 && center > v2) ? center : 0;
		}
	}
}
```
 sobel 행렬에서 기울기 방향으로 조사했을 때, center값이 그 두 값보다 크면 dst행렬에 저장한다.

sobel 행렬은 
$$
sobel= \left | G_{x} \right |+\left | G_{y} \right | 
$$
즉 방향의 크기(변화량의 크기)이다. 

기울기 방향에 따라 조사할 점이 다르다. 기울기 방향의 앞뒤 점을 조사해서 

cetner값이 앞뒤 값보다 크면 dst 행렬에 center값을 저장하고, 아니면 0을 저장한다.

기울기 방향의 점들 중에서 가장 영향력이 큰 부분(edge가 두드러지게 나타나는 부분)만 남긴다고 생각하면 될 것 같다.

기울기 방향은 [그림2]를 참조하면 이해가 될 것이다.

i와 j가 헷갈릴 수 있는데, 잘 보면 row가 i, column이 j여서 일반적인 x,y 축과 반대로 생각하면

편하다.



#### edge 추적

```c++
void trace(Mat max_so, Mat& pos_ck, Mat& hy_img, Point pt, int low)
{
	Rect rect(Point(0, 0), pos_ck.size());
	if (!rect.contains(pt)) return;			// 추적화소의 영상 범위 확인 

	if (pos_ck.at<uchar>(pt) == 0 && max_so.at<float>(pt) > low)
	{
		pos_ck.at<uchar>(pt) = 1;			// 추적 완료 좌표
		hy_img.at<uchar>(pt) = 255;			// 에지 지정

		// 추적 재귀 함수
		trace(max_so, pos_ck, hy_img, pt + Point(-1, -1), low);
		trace(max_so, pos_ck, hy_img, pt + Point( 0, -1), low);
		trace(max_so, pos_ck, hy_img, pt + Point(+1, -1), low);
		trace(max_so, pos_ck, hy_img, pt + Point(-1, 0), low);

		trace(max_so, pos_ck, hy_img, pt + Point(+1, 0), low);
		trace(max_so, pos_ck, hy_img, pt + Point(-1, +1), low);
		trace(max_so, pos_ck, hy_img, pt + Point( 0, +1), low);
		trace(max_so, pos_ck, hy_img, pt + Point(+1, +1), low);
	}
}
```

```c++
void  hysteresis_th(Mat max_so, Mat&  hy_img, int low, int high)
{
	Mat pos_ck(max_so.size(), CV_8U, Scalar(0));
	hy_img = Mat(max_so.size(), CV_8U, Scalar(0));

	for (int i = 0; i < max_so.rows; i++){
		for (int j = 0; j < max_so.cols; j++)
		{
			if (max_so.at<float>(i, j) > high)
				trace(max_so, pos_ck, hy_img, Point(j, i), low);
		}
	}
}
```

trace 함수는 edge를 추적하는 재귀함수이고, hysteresis_th 함수는 trace 함수가 시작될 조건을 알려준다.(max_so 행렬에서 high값보다 큰 point를 발견했을 때 trace 함수 시작)

trace 함수를 보면, 우선 rect는 point를 검사할 때 8방향을 다 검사하는데, 이 범위가 영상의 범위를 넘어가면 함수를 끝내라 라는 뜻이다.

pos_ck 행렬은 추적을 했다라는 표시를 남기는 행렬이다.

pos_ck의 포인트가 0이고, max_so 행렬의 포인트가 low값보다 크면, 

pos_ck의 포인트를 1로 저장(추적완료표시)하고, hy_img에 255를 저장 한다. (edge로 표시)

이 과정은 high값보다 큰 포인트가 나타나면 trace를 시작해서, high값 사이 사이의 edge가 끊기지 않게 이어주는 역할을 하는 함수이다. 



### 전체 코드

```c++
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

void calc_direct(Mat Gy, Mat Gx, Mat& direct)
{
	direct.create(Gy.size(), CV_8U);

	for (int i = 0; i < direct.rows; i++){
		for (int j = 0; j < direct.cols; j++){
			float gx = Gx.at<float>(i, j);
			float gy = Gy.at<float>(i, j);
			int theat = int(fastAtan2(gy, gx) / 45);
			direct.at<uchar>(i, j) = theat % 4;
		}
	}
}

void supp_nonMax(Mat sobel, Mat  direct, Mat& dst)		// 비최대값 억제
{
	dst = Mat(sobel.size(), CV_32F, Scalar(0));

	for (int i = 1; i < sobel.rows - 1; i++) {
		for (int j = 1; j < sobel.cols - 1; j++) 
		{
			int   dir = direct.at<uchar>(i, j);				// 기울기 값
			float v1, v2;
			if (dir == 0) {			// 기울기 방향 0도 방향
				v1 = sobel.at<float>(i, j - 1);
				v2 = sobel.at<float>(i, j + 1);
			}
			else if (dir == 1) {		// 기울기 방향 45도
				v1 = sobel.at<float>(i + 1, j + 1);
				v2 = sobel.at<float>(i - 1, j - 1);
			}
			else if (dir == 2) {		// 기울기 방향 90도
				v1 = sobel.at<float>(i - 1, j);
				v2 = sobel.at<float>(i + 1, j);
			}
			else if (dir == 3) {		// 기울기 방향 135도
				v1 = sobel.at<float>(i + 1, j - 1);
				v2 = sobel.at<float>(i - 1, j + 1);
			}

			float center = sobel.at<float>(i, j);
			dst.at<float>(i, j) = (center > v1 && center > v2) ? center : 0;
		}
	}
}

void trace(Mat max_so, Mat& pos_ck, Mat& hy_img, Point pt, int low)
{
	Rect rect(Point(0, 0), pos_ck.size());
	if (!rect.contains(pt)) return;			// 추적화소의 영상 범위 확인 

	if (pos_ck.at<uchar>(pt) == 0 && max_so.at<float>(pt) > low)
	{
		pos_ck.at<uchar>(pt) = 1;			// 추적 완료 좌표
		hy_img.at<uchar>(pt) = 255;			// 에지 지정

		// 추적 재귀 함수
		trace(max_so, pos_ck, hy_img, pt + Point(-1, -1), low);
		trace(max_so, pos_ck, hy_img, pt + Point( 0, -1), low);
		trace(max_so, pos_ck, hy_img, pt + Point(+1, -1), low);
		trace(max_so, pos_ck, hy_img, pt + Point(-1, 0), low);

		trace(max_so, pos_ck, hy_img, pt + Point(+1, 0), low);
		trace(max_so, pos_ck, hy_img, pt + Point(-1, +1), low);
		trace(max_so, pos_ck, hy_img, pt + Point( 0, +1), low);
		trace(max_so, pos_ck, hy_img, pt + Point(+1, +1), low);
	}
}

void  hysteresis_th(Mat max_so, Mat&  hy_img, int low, int high)
{
	Mat pos_ck(max_so.size(), CV_8U, Scalar(0));
	hy_img = Mat(max_so.size(), CV_8U, Scalar(0));

	for (int i = 0; i < max_so.rows; i++){
		for (int j = 0; j < max_so.cols; j++)
		{
			if (max_so.at<float>(i, j) > high)
				trace(max_so, pos_ck, hy_img, Point(j, i), low);
		}
	}
}

int main()
{
	Mat image = imread("../image/cannay_tset.jpg", IMREAD_GRAYSCALE); // 이미지경로 수정
	CV_Assert(image.data);

	Mat gau_img, Gx, Gy, direct, sobel, max_sobel, hy_img, canny;

	GaussianBlur(image, gau_img, Size(5, 5), 0.3);
	Sobel(gau_img, Gx, CV_32F, 1, 0, 3);
	Sobel(gau_img, Gy, CV_32F, 0, 1, 3);
	sobel = abs(Gx) + abs(Gy);
//	magnitude(Gx, Gy, sobel);

	calc_direct(Gy, Gx, direct);
	supp_nonMax(sobel, direct, max_sobel);
	hysteresis_th(max_sobel, hy_img, 100, 150);

	Canny(image, canny, 100, 150);

	imshow("image", image);
	imshow("canny", hy_img);
	imshow("OpenCV_canny", canny);
	waitKey();
	return 0;
}
```



### 결과

![image](https://user-images.githubusercontent.com/67038853/106764941-a3337080-667b-11eb-9536-c8f6a35a766f.png)

[그림4] 원본, canny edge, opencv canny edge 함수