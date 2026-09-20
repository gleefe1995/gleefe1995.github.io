---
layout: post
title: "CUDA Memory Hierarchy: Register, Shared, Global Memory와 Cache"
date: 2026-09-20 12:00:00 +0900
categories: cuda
use_math: true
---

[CUDA Memory Model 강의](https://www.youtube.com/watch?v=ipARGT0HfBM)의 내용을 바탕으로 CUDA의 메모리 공간, 접근 범위, 수명, 물리적 위치를 정리한다. 강의가 만들어진 뒤 GPU 구조와 CUDA 문서가 바뀐 부분은 현재 NVIDIA 문서에 맞추어 보완했다.

앞 글인 [CUDA 기초: 스레드 구조, 메모리 복사, 벡터 덧셈과 SGEMM]({% post_url /cuda/2026-08-16-CUDA-기초 %})에서는 Host와 Device 사이의 복사와 기본 SGEMM을 살펴보았다. 이번 글에서는 GPU 안에서 데이터가 어디에 놓이고, 어떤 스레드가 접근하며, 이 선택이 성능에 어떤 영향을 주는지에 집중한다.

## CUDA 메모리 계층을 보는 두 가지 관점

CUDA 메모리는 하나의 세로줄로만 외우면 혼동하기 쉽다. 다음 두 관점을 분리해서 보는 것이 중요하다.

1. **Memory space**: register, local, shared, global, constant, texture처럼 프로그램이 데이터를 어떤 범위에서 사용하는가
2. **Hardware cache**: L1, L2, constant cache, texture cache처럼 DRAM 접근을 하드웨어가 어떻게 줄이는가

Register와 shared memory는 프로그래머가 사용하는 저장 공간이고, L1과 L2는 하드웨어가 관리하는 캐시다. 특히 shared memory는 L1 아래에 있는 캐시 단계가 아니라, 스레드 블록이 직접 관리하는 scratchpad다.

[![CUDA 메모리 계층과 물리적 배치](/assets/cuda/cuda-memory-hierarchy.svg)](/assets/cuda/cuda-memory-hierarchy.svg)

*그림을 클릭하면 원본 크기로 볼 수 있다.*

대략적으로 SM 안의 register와 shared memory는 작고 빠르며, Device DRAM은 크지만 접근 비용이 높다. 그러나 실제 속도는 cache hit, coalescing, bank conflict, 데이터 의존성, GPU 아키텍처에 따라 달라지므로 고정된 지연 시간 숫자로 외우는 것은 좋지 않다.

## 접근 범위와 수명

CUDA 실행 구조는 `thread → block → grid`로 중첩된다. 메모리 공간도 이 구조와 연결해서 볼 수 있다.

[![CUDA 메모리 공간의 접근 범위와 수명](/assets/cuda/cuda-memory-scope.svg)](/assets/cuda/cuda-memory-scope.svg)

*그림을 클릭하면 원본 크기로 볼 수 있다.*

| 종류 | 일반적인 물리 위치 | 접근 범위 | 수명 | 관리 주체 |
|---|---|---|---|---|
| Register | SM의 register file | Thread | Thread 실행 동안 | Compiler |
| Local memory | Device DRAM, L1/L2에 cache 가능 | Thread | Thread 실행 동안 | Compiler |
| Shared memory | SM 내부, L1과 물리 자원을 공유할 수 있음 | Block | Block 실행 동안 | Programmer |
| Global memory | Device DRAM | Device의 thread들 | 할당부터 해제까지 | Programmer / Runtime |
| Constant memory | Device memory와 SM별 constant cache | Device의 thread들, kernel에서는 read-only | CUDA context / application 수명 | Programmer / Hardware cache |
| Texture access | CUDA array 또는 device memory와 texture 경로 | Device의 thread들, texture fetch는 read-only | Resource와 texture object 수명 | Programmer / Hardware unit |

표의 접근 범위와 수명은 서로 다른 개념이다. 예를 들어 global memory는 한 grid의 모든 스레드가 접근할 수 있지만, `cudaMalloc()`으로 얻은 allocation은 kernel이 끝난 뒤에도 `cudaFree()`를 호출할 때까지 유지된다. 반대로 shared memory는 block 안에서만 보이고 그 block이 끝나면 사라진다.

L1과 L2 같은 cache에는 CUDA 언어 수준의 thread 접근 범위나 변수 수명이 없다. Programmer가 cache 안의 주소를 직접 선택하는 것이 아니라 hardware가 memory request를 처리하면서 관리한다.

| Hardware cache / unit | 위치와 공유 범위 | 역할 |
|---|---|---|
| L1TEX | 각 SM | Global/local/texture request를 SM 범위에서 처리 |
| L2 cache | GPU 전체가 공유 | Device-memory request를 모든 SM 범위에서 cache |
| Constant cache | 각 SM | Constant load와 warp broadcast 지원 |

## Register

Register는 SM 내부의 register file에 있고 각 스레드가 자기 몫을 사용한다. 커널 안의 scalar 지역 변수와 loop counter 등은 컴파일러가 가능한 경우 register에 배치한다.

한 SM에 resident한 warp의 실행 context는 PC와 register 등을 위한 on-chip 자원에 유지된다. 따라서 scheduler가 실행할 warp를 바꿀 때 CPU thread처럼 register 상태를 메모리에 저장했다가 다시 복원하는 과정이 필요하지 않다. 강의에서 이를 **zero-overhead warp scheduling**의 배경으로 설명한다. 다만 register file은 SM의 한정된 물리 자원이며 resident thread들이 나누어 사용한다.

```cpp
__global__ void saxpy(float a, const float *x, float *y, int n)
{
    int i = blockIdx.x * blockDim.x + threadIdx.x;

    if (i < n) {
        float xi = x[i];       // xi와 result는 보통 register 후보
        float result = a * xi + y[i];
        y[i] = result;
    }
}
```

Register 수는 유한하다. 스레드 하나가 많은 register를 사용하면 SM에 동시에 머물 수 있는 warp나 block 수가 줄어 occupancy가 낮아질 수 있다. 그렇다고 register 수를 무조건 줄이면 좋은 것은 아니다. Register가 부족하면 값이 local memory로 **spill**되어 오히려 느린 메모리 트래픽이 늘 수 있다.

Compiler가 사용한 자원은 다음처럼 확인할 수 있다.

```bash
nvcc --ptxas-options=-v source.cu
```

출력에서는 kernel별로 `Used ... registers`에 표시되는 thread당 register 수, `... bytes smem`에 표시되는 block당 static shared memory, stack frame과 spill load/store 정보를 확인할 수 있다. CUDA Toolkit 버전에 따라 출력 항목과 형식은 조금씩 다를 수 있다.

## Local memory

`local`이라는 이름 때문에 스레드 가까이에 있는 빠른 메모리로 오해하기 쉽다. 여기서 local은 **각 스레드만 접근하는 논리적 주소 공간**이라는 뜻이다. 물리적으로는 보통 Device DRAM에 있고 L1/L2 cache의 도움을 받을 수 있다.

다음과 같은 값이 local memory에 놓일 수 있다.

- Register에 모두 담기지 못해 spill된 값
- Compile time에 index를 결정하기 어려운 thread별 배열
- 크기가 큰 자동 배열이나 stack frame

```cpp
__global__ void kernel(const float *input, float *output)
{
    float scratch[32];  // 실제 배치는 compiler가 결정한다.
    // runtime index로 폭넓게 접근하면 local memory에 놓일 가능성이 커진다.
}
```

지역 배열이라고 항상 local memory가 되는 것도 아니고, 지역 scalar라고 항상 register가 되는 것도 아니다. 최종 배치는 compiler와 target architecture가 결정하므로 `ptxas` 출력과 profiler로 확인해야 한다.

## Shared memory

Shared memory는 같은 block의 스레드가 함께 사용하는 on-chip 메모리다. Global memory에서 여러 번 읽을 데이터를 한 번 가져와 재사용하거나, 스레드 사이에 중간 결과를 전달할 때 사용한다.

> 기본 모델에서는 shared memory의 범위가 한 block이다. Compute capability 9.0 이상의 thread block cluster와 distributed shared memory를 사용하는 고급 기능은 예외지만, 일반적인 kernel은 block 범위로 이해하면 된다.

### Static allocation

```cpp
__global__ void static_shared_kernel()
{
    __shared__ float tile[32][33];

    // block마다 서로 독립적인 tile 하나가 만들어진다.
}
```

`__shared__` 선언은 각 스레드마다 배열을 하나씩 만드는 것이 아니다. **Block마다 하나의 shared allocation**이 생기고 그 block의 모든 스레드가 공유한다.

두 번째 차원을 `33`으로 잡는 패턴은 32×32 행렬을 transpose할 때 흔히 사용한다. Shared memory는 일반적으로 여러 bank로 나뉘는데, padding 한 칸을 두면 같은 warp의 접근이 한 bank에 몰리는 bank conflict를 피할 수 있다. 항상 `+1`이 필요한 것은 아니며 실제 접근 패턴을 보고 결정해야 한다.

### Dynamic allocation

```cpp
__global__ void dynamic_shared_kernel()
{
    extern __shared__ float scratch[];

    // scratch의 크기는 kernel launch에서 결정된다.
}

int threads = 256;
size_t shared_bytes = threads * sizeof(float);

dynamic_shared_kernel<<<grid, threads, shared_bytes>>>();
```

Kernel launch의 세 번째 execution configuration 인자가 block당 dynamic shared memory의 바이트 수다.

```cpp
kernel<<<grid_size, block_size, shared_memory_bytes, stream>>>();
```

Shared memory로 협력할 때는 데이터가 준비된 시점을 맞추어야 한다.

```cpp
tile[threadIdx.x] = input[index];
__syncthreads();

float value = tile[other_index];
__syncthreads();
```

`__syncthreads()`는 같은 block만 동기화한다. 모든 스레드가 barrier에 도달하지 않는 분기에서 호출하면 deadlock이나 정의되지 않은 동작으로 이어질 수 있다. 다른 block 사이의 동기화에는 kernel 경계나 그 목적에 맞는 cooperative mechanism이 필요하다.

Shared memory도 무한하지 않다. Block 하나가 많이 사용하면 SM에 동시에 resident할 수 있는 block 수가 줄어든다. 강의의 예처럼 SM에 shared memory가 48 KiB 있고 block 하나가 24 KiB를 사용한다고 가정하면, shared memory만을 기준으로는 최대 두 block이 들어갈 수 있다. 실제 resident block 수는 register, thread 수, architecture limit에도 함께 제한된다.

## Global memory

Global memory는 큰 입력, 출력, 중간 결과를 저장하는 기본 공간이다. `cudaMalloc()`으로 할당한 메모리가 대표적이며 kernel에 pointer를 전달하면 grid의 모든 스레드가 접근할 수 있다.

```cpp
float *d_data;
cudaMalloc(&d_data, bytes);
cudaMemcpy(d_data, h_data, bytes, cudaMemcpyHostToDevice);

kernel<<<grid, block>>>(d_data);

cudaMemcpy(h_data, d_data, bytes, cudaMemcpyDeviceToHost);
cudaFree(d_data);
```

Discrete GPU에서 Host가 `cudaMalloc()` pointer를 일반 CPU pointer처럼 직접 역참조하는 것은 아니다. Host는 CUDA Runtime API를 통해 할당하고 복사하며, kernel이 그 주소를 사용한다. Unified Memory나 mapped memory는 별도의 메모리 관리 방식이다.

Global memory는 DRAM bandwidth가 높아도 접근 지연이 크다. 성능을 높이려면 warp의 요청이 적은 수의 memory transaction으로 합쳐지도록 **coalescing**해야 한다.

```text
좋은 예
lane:     0      1      2      3            31
access:  x[0]   x[1]   x[2]   x[3]   ...   x[31]

나쁜 예
lane:     0      1      2      3            31
access:  x[0]  x[32]  x[64]  x[96]   ...  큰 stride
```

Compute capability 6.0 이상에서 global memory transaction은 필요한 32-byte segment 수를 기준으로 설명할 수 있다. 인접한 스레드가 정렬된 인접 데이터를 읽도록 만드는 것이 기본 원칙이다.

## Constant memory

Constant memory는 Host가 값을 채우고 kernel에서는 읽기만 하는 공간이다.

```cpp
__constant__ float coeff[64];

float h_coeff[64] = { /* ... */ };
cudaMemcpyToSymbol(coeff, h_coeff, sizeof(h_coeff));
```

Constant memory가 특히 유리한 경우는 한 warp의 스레드들이 **같은 주소**를 읽을 때다. Constant cache가 한 값을 warp 전체에 broadcast할 수 있다. 반대로 각 lane이 서로 다른 constant 주소를 읽으면 요청이 직렬화될 수 있어 효과가 줄어든다.

예시는 filter coefficient, 작은 lookup table, 모든 스레드가 공통으로 읽는 parameter다. CUDA 문서에서 constant memory의 전체 공간은 64 KiB로 설명하지만, SM의 constant cache 크기와 working set은 별개이며 architecture에 따라 달라진다. 따라서 “constant cache 자체가 64 KiB”라고 이해하면 안 된다.

## Texture memory

Texture memory도 별도의 DRAM 덩어리라기보다 CUDA array, linear device memory, pitched allocation 같은 resource를 **texture object를 통해 읽는 접근 경로**에 가깝다.

Texture unit은 다음 기능을 제공한다.

- 2차원 spatial locality를 고려한 cache 경로
- Address clamp, wrap과 normalized coordinate
- Filtering과 format conversion 같은 texture 기능

강의에서는 2D 읽기 패턴에서의 texture cache 장점을 설명한다. 현재 CUDA Programming Guide는 지원 중인 최신 GPU에서 일반적인 non-graphics load를 texture 또는 surface API로 바꾸는 것만으로는 성능 이점이 없다고 안내한다. 따라서 단순한 read-only global load의 대체재보다는 **filtering, addressing 같은 texture semantics가 필요할 때** 선택하는 것이 명확하다.

## L1 cache와 L2 cache

L1과 L2는 코드에서 배열처럼 직접 할당하는 memory space가 아니라 하드웨어가 관리하는 cache다.

- **L1 cache**: 각 SM에 있으며 해당 SM의 memory request를 처리한다.
- **L2 cache**: GPU 전체의 SM이 공유하며 Device DRAM 앞에서 동작한다.
- **Constant/texture cache**: 각각의 읽기 패턴과 기능에 맞춘 경로다.

현대 NVIDIA GPU에서는 L1 data cache와 shared memory가 unified data cache의 물리 자원을 나눌 수 있다. 이 말은 shared memory가 자동 cache라는 뜻이 아니다. L1은 하드웨어가 교체 정책을 관리하지만 shared memory는 kernel이 명시적으로 데이터를 넣고, 동기화하고, 다시 사용한다.

일부 architecture에서는 다음 API로 L1/shared 비율에 대한 선호를 전달할 수 있다.

```cpp
cudaFuncSetCacheConfig(kernel, cudaFuncCachePreferShared);
```

이 설정은 **선호도**이며 요청한 구성이 반드시 그대로 적용된다는 보장은 없다.

## SGEMM으로 보는 데이터 이동

기본 SGEMM은 각 스레드가 A와 B를 global memory에서 반복해서 읽는다. Tiled SGEMM은 block이 필요한 부분을 shared memory에 올려 재사용한다.

```text
Device DRAM의 A, B
        │ coalesced load
        ▼
      L2 / L1
        │ block의 thread들이 협력해 적재
        ▼
  Shared-memory tile
        │ 여러 번 재사용
        ▼
  Thread별 register accumulator
        │ 한 번 저장
        ▼
Device DRAM의 C
```

핵심 형태는 다음과 같다.

```cpp
template <int TILE>
__global__ void tiled_sgemm(
    const float *A, const float *B, float *C,
    int N, int M, int K)
{
    __shared__ float As[TILE][TILE];
    __shared__ float Bs[TILE][TILE];

    // blockDim == dim3(TILE, TILE)인 launch를 가정한다.

    int row = blockIdx.y * TILE + threadIdx.y;
    int col = blockIdx.x * TILE + threadIdx.x;
    float acc = 0.0f;

    for (int tile = 0; tile < (K + TILE - 1) / TILE; ++tile) {
        int a_col = tile * TILE + threadIdx.x;
        int b_row = tile * TILE + threadIdx.y;

        As[threadIdx.y][threadIdx.x] =
            (row < N && a_col < K) ? A[row * K + a_col] : 0.0f;
        Bs[threadIdx.y][threadIdx.x] =
            (b_row < K && col < M) ? B[b_row * M + col] : 0.0f;

        __syncthreads();

        for (int k = 0; k < TILE; ++k) {
            acc += As[threadIdx.y][k] * Bs[k][threadIdx.x];
        }

        __syncthreads();
    }

    if (row < N && col < M) {
        C[row * M + col] = acc;
    }
}
```

이 kernel에서 각 공간의 역할은 분명하다.

- A, B, C 전체는 global memory에 있다.
- `As`, `Bs`는 block이 공유하는 shared memory tile이다.
- `acc`, `row`, `col`은 thread별 register 후보이다.
- Global load는 warp가 인접 원소를 읽도록 구성한다.
- `__syncthreads()`는 tile 적재와 사용의 경계를 만든다.

Shared memory로 옮기는 것 자체가 목적은 아니다. **비싼 global load 한 번으로 가져온 값을 block 안에서 여러 번 재사용**할 수 있을 때 이득이 생긴다.

## 어떤 메모리를 선택할 것인가

| 상황 | 먼저 고려할 공간 | 확인할 점 |
|---|---|---|
| Thread만 쓰는 작은 scalar | Register | Register pressure와 spill |
| Block 안에서 반복 재사용 | Shared memory | 용량, synchronization, bank conflict |
| 큰 입력과 출력 | Global memory | Coalescing, alignment, 재사용률 |
| Warp가 같은 read-only 값을 반복 사용 | Constant memory | 같은 주소 broadcast 여부 |
| 2D addressing, filtering이 필요 | Texture object | Texture semantics와 resource 형식 |
| 큰 thread별 배열이나 spill | Local memory가 될 수 있음 | `lmem`과 profiler에서 실제 traffic 확인 |

GPU별 resource 크기는 하드코딩하지 말고 query하는 것이 안전하다.

```cpp
cudaDeviceProp prop{};
cudaGetDeviceProperties(&prop, 0);

printf("registers / SM: %d\n", prop.regsPerMultiprocessor);
printf("shared / SM: %zu bytes\n", prop.sharedMemPerMultiprocessor);
printf("shared / block (default): %zu bytes\n", prop.sharedMemPerBlock);
printf("shared / block (opt-in): %zu bytes\n", prop.sharedMemPerBlockOptin);
printf("L2 cache: %d bytes\n", prop.l2CacheSize);
printf("constant memory: %zu bytes\n", prop.totalConstMem);
```

`sharedMemPerBlockOptin`은 opt-in 가능한 장치 한도다. 기본 한도를 넘는 dynamic shared memory를 실제로 사용하려면 지원 여부를 확인하고 `cudaFuncSetAttribute()`로 kernel의 최대 dynamic shared memory 크기를 설정해야 한다.

## 자주 생기는 오해

- **Local memory는 thread 가까이에 있다**: 이름은 local이지만 물리적으로는 보통 Device DRAM에 있다.
- **Shared memory는 자동 L1 cache다**: Shared는 kernel이 직접 관리하는 scratchpad이고 L1은 하드웨어 cache다.
- **Register 사용량은 적을수록 좋다**: 너무 많으면 occupancy가 줄 수 있지만 너무 줄이면 spill이 발생할 수 있다.
- **Occupancy가 100%면 항상 가장 빠르다**: Instruction-level parallelism, memory latency, register와 shared 사용량을 함께 profile해야 한다.
- **Global memory는 Host와 모든 thread가 같은 방식으로 접근한다**: Device thread는 load/store하고, 일반적인 discrete GPU의 Host는 Runtime API로 복사한다.
- **메모리 종류마다 속도가 고정되어 있다**: Cache hit, coalescing, bank conflict와 architecture에 따라 실제 성능은 크게 달라진다.
- **`__syncthreads()`가 grid 전체를 동기화한다**: 동기화 범위는 한 block이다.

## 정리

- CUDA memory space는 먼저 접근 범위에 따라 thread, block, device 수준으로 나눌 수 있다.
- Register는 thread별 on-chip 공간이고, local memory는 thread 전용이지만 보통 Device DRAM에 있다.
- Shared memory는 block별 on-chip scratchpad이며 데이터 재사용과 thread 간 협력에 사용한다.
- Global, constant, texture의 backing data는 Device memory에 있고 각각 다른 접근 방식과 cache 경로를 사용한다.
- L1과 L2는 하드웨어 cache이며 shared memory와 같은 programmer-managed 공간과 구분해야 한다.
- 성능 최적화의 핵심은 무조건 빠른 공간을 쓰는 것이 아니라, coalescing과 재사용으로 불필요한 데이터 이동을 줄이는 것이다.
- 고정된 용량이나 지연 시간을 외우기보다 실제 GPU 속성을 query하고 compiler 출력과 profiler로 확인해야 한다.

## 참고 자료

- [SPIN Lab, Lec 9. CUDA Memory Model (1/4)](https://www.youtube.com/watch?v=ipARGT0HfBM)
- [HPC Lab, OpenMP/CUDA 강의 자료 모음](https://hpclab.tistory.com/4)
- [NVIDIA CUDA Programming Guide: Writing CUDA Kernels](https://docs.nvidia.com/cuda/cuda-programming-guide/02-basics/writing-cuda-kernels.html)
- [NVIDIA CUDA Programming Guide: Programming Model](https://docs.nvidia.com/cuda/cuda-programming-guide/01-introduction/programming-model.html)
- [NVIDIA CUDA C++ Best Practices Guide](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/)
- [NVIDIA CUDA Runtime API: Memory Management](https://docs.nvidia.com/cuda/cuda-runtime-api/cuda_runtime_api/group__CUDART__MEMORY.html)
