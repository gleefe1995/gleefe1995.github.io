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



#### Hello world!

```
cd ceres-bin/bin
./helloworld
```

ceres-solver에서 가장 간단한 예제인 Hello wolrd를 통해 10-x 함수값이 최소가 되는 x값을 구해보자.

```c++
struct CostFunctor {
  template <typename T>
  bool operator()(const T* const x, T* residual) const {
    residual[0] = 10.0 - x[0];
    return true;
  }
};
```

cost function $f(x)=10-x$로 설정하는 부분이다. template을 써서 어떤 input과 output이 들어오던 T 형태로 받게 된다. residual 값이 double일수도 있고, vector 형태일수도 있기 때문이다. 



```c++
int main(int argc, char** argv) {
  google::InitGoogleLogging(argv[0]);

  // The variable to solve for with its initial value. It will be
  // mutated in place by the solver.
  double x = 0.5;
  const double initial_x = x;

```

initial_x 값을 0.5로 설정한다.



```c++
  // Build the problem.
  Problem problem;

  // Set up the only cost function (also known as residual). This uses
  // auto-differentiation to obtain the derivative (jacobian).
  CostFunction* cost_function =
      new AutoDiffCostFunction<CostFunctor, 1, 1>(new CostFunctor);
  problem.AddResidualBlock(cost_function, nullptr, &x);
```

Problem을 선언하고, cost_function에 AutoDiffCostFunction을 통해 미분을 해준다. 만약 residual에 여러 값이 있다면 T=Jet으로 jacobian을 나타내게 된다. 

이 cost_function을 ResidualBlock에 넣어 풀게 되면,

```c++
  // Run the solver!
  Solver::Options options;
  options.minimizer_progress_to_stdout = true;
  Solver::Summary summary;
  Solve(options, &problem, &summary);

  std::cout << summary.BriefReport() << "\n";
  std::cout << "x : " << initial_x << " -> " << x << "\n";
  return 0;
}
```

![image](https://user-images.githubusercontent.com/67038853/114900658-0d704a80-9e4f-11eb-9040-e6761dfef536.png)

iteration 2번만에 x값이 10이 나오는 것을 볼 수 있다.