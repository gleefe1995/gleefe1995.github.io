---
layout: post
title: "ceres-solver tutorial"
date: 2021-04-16 17:05:30 -0400
categories: optimization
use_math: true
---

### ceres-solver tutorial



#### dependency

```
# CMake
sudo apt-get install cmake
# google-glog + gflags
sudo apt-get install libgoogle-glog-dev libgflags-dev
# BLAS & LAPACK
sudo apt-get install libatlas-base-dev
# Eigen3
sudo apt-get install libeigen3-dev
# SuiteSparse and CXSparse (optional)
sudo apt-get install libsuitesparse-dev
```



#### ceres-solver installation

```
cd catkin_ws/src
git clone https://ceres-solver.googlesource.com/ceres-solver
// git clone 받은 디렉토리 옆에 ceres-bin 생성
mkdir ceres-bin
cd ceres-bin
make -j3
make test
make install
```



#### Introduction


$$
min \frac{1}{2}\sum_{i}\rho(||f_{i}(x_{i_{1}},\cdots,x_{i_{k}})||^{2})
$$


Ceres는 $x_{j}$가 어떤 범위 안에 있을 때 non-linear least square 문제를 풀 수 있게 해준다. 


$$
\rho(||f_{i}(x_{i_{1}},\cdots,x_{i_{k}})||^{2}
$$


이 **ResidualBlock**이고, $f_{i}(\cdot)$이 **CostFunction**,이다.  또한 $x_{i}$ 값들의 집합을 **ParameterBlock**이라고 한다. $\rho_{i}$는 LossFunction이다.

특별한 케이스로 $\rho_{i}(x)=x$, x값의 범위는 $-infty$ 부터 $infty$까지로 설정한다. 그러면 식을


$$
\frac{1}{2}\sum_{i}||f_{i}(x_{i_{1}},\cdots,x_{i_{k}})||^{2}
$$


로 간단하게 나타낼 수 있다.

#### Helloworld

```
cd ceres-bin/bin
./helloworld
```

![image](https://user-images.githubusercontent.com/67038853/114900658-0d704a80-9e4f-11eb-9040-e6761dfef536.png)



