---
layout: post
title: "CUDA Matrix Transpose: Shared Memory로 Coalesced Access 만들기"
date: 2026-09-20 12:00:00 +0900
categories: cuda
use_math: true
---

CUDA에서 행렬 전치(matrix transpose)를 구현하면서, 왜 shared memory를 사용하는지와 블록 및 블록 내부 좌표가 어떻게 전치되는지 정리한다.

![CUDA tiled matrix transpose](/assets/cuda/cuda-matrix-transpose-tiles.svg)

## 행렬 전치

행렬 전치는 행과 열을 서로 교환하는 연산이다.

$$
B[y][x] = A[x][y]
$$

예를 들어 다음 행렬은

```text
1 2 3
4 5 6
```

전치 후 다음과 같이 변한다.

```text
1 4
2 5
3 6
```

## Naive 구현

가장 단순한 CUDA 커널에서는 스레드 하나가 입력 원소 하나를 읽어서 전치된 출력 좌표에 기록한다.

```cpp
__global__ void matrix_transpose_naive(int *input, int *output)
{
    int x = threadIdx.x + blockIdx.x * blockDim.x;
    int y = threadIdx.y + blockIdx.y * blockDim.y;

    int index = y * N + x;
    int transposedIndex = x * N + y;

    output[transposedIndex] = input[index];
}
```

한 warp에서는 일반적으로 `threadIdx.x`가 연속해서 변한다. 따라서 `input[y * N + x]`는 연속된 주소를 읽어 global memory load가 coalesced된다.

하지만 출력 주소는 다음처럼 `N`개 원소 간격으로 떨어진다.

```text
output[0 * N + y]
output[1 * N + y]
output[2 * N + y]
...
```

즉 global memory store가 연속되지 않아 메모리 대역폭을 효율적으로 사용하지 못한다.

## Shared memory를 사용한 타일 전치

행렬 전체를 한 번에 생각하는 대신 `BLOCK_SIZE × BLOCK_SIZE` 크기의 타일로 나눈다.

```cpp
#define BLOCK_SIZE 32

__shared__ int sharedMemory[BLOCK_SIZE][BLOCK_SIZE];
```

각 블록이 행렬의 `32×32` 타일 하나를 처리한다. 전치 과정은 다음 두 단계로 나눌 수 있다.

1. 출력에서 블록의 위치를 전치한다.
2. shared memory를 이용해 각 블록 내부 원소를 전치한다.

## 1. 입력 좌표 계산

```cpp
int indexX = threadIdx.x + blockIdx.x * blockDim.x;
int indexY = threadIdx.y + blockIdx.y * blockDim.y;

int index = indexY * N + indexX;
```

`indexX`와 `indexY`는 현재 스레드가 입력 행렬에서 담당하는 전역 좌표이다.

```text
indexX = block의 x 시작점 + block 내부 x 좌표
indexY = block의 y 시작점 + block 내부 y 좌표
```

## 2. 출력 블록 위치 전치

```cpp
int tindexX = threadIdx.x + blockIdx.y * blockDim.x;
int tindexY = threadIdx.y + blockIdx.x * blockDim.y;
```

여기서는 `blockIdx.x`와 `blockIdx.y`가 서로 바뀐다.

```text
입력 블록 위치 (blockY, blockX)
              ↓
출력 블록 위치 (blockX, blockY)
```

블록 행렬이 다음과 같다면

```text
A B
C D
```

블록 위치를 전치한 결과는 다음과 같다.

```text
A C
B D
```

그러나 이것만으로는 완전한 행렬 전치가 아니다. 각 블록 내부도 전치해야 한다.

## 3. 블록 내부 전치

입력은 global memory에서 연속적으로 읽은 뒤 shared memory에 저장한다.

```cpp
sharedMemory[threadIdx.x][threadIdx.y] = input[index];
```

출력할 때는 shared memory의 좌표 순서를 바꿔 읽는다.

```cpp
output[transposedIndex] =
    sharedMemory[threadIdx.y][threadIdx.x];
```

즉 다음 변환이 일어난다.

```text
저장: sharedMemory[x][y]
읽기: sharedMemory[y][x]
```

예를 들어 타일 내부가 다음과 같다면

```text
a00 a01
a10 a11
```

좌표를 바꿔 읽은 결과는 다음과 같다.

```text
a00 a10
a01 a11
```

## `__syncthreads()`가 필요한 이유

```cpp
sharedMemory[threadIdx.x][threadIdx.y] = input[index];

__syncthreads();

output[transposedIndex] =
    sharedMemory[threadIdx.y][threadIdx.x];
```

각 스레드는 자신이 기록한 위치가 아니라 다른 스레드가 기록한 shared memory 위치를 읽을 수 있다.

예를 들어:

```text
thread (x=3, y=7) → sharedMemory[3][7] 기록
thread (x=7, y=3) → sharedMemory[3][7] 읽기
```

따라서 모든 스레드가 shared memory 기록을 끝낸 후 읽기를 시작해야 한다. `__syncthreads()`는 같은 블록의 스레드를 이 지점에서 동기화한다.

## 왜 global memory 접근이 빨라지는가

Shared memory 버전의 데이터 흐름은 다음과 같다.

```text
Global Memory             Shared Memory             Global Memory
연속 주소에서 읽기   →    타일 내부 전치       →    연속 주소에 쓰기
     coalesced                                        coalesced
```

Naive 버전은 읽기 또는 쓰기 중 하나가 큰 stride를 갖는다. Shared memory를 중간 버퍼로 사용하면 global memory에서는 양쪽 모두 연속 주소에 접근하고, 불연속적인 좌표 교환은 빠른 on-chip shared memory에서 수행할 수 있다.

## 전체 커널

```cpp
__global__ void matrix_transpose_shared(int *input, int *output)
{
    __shared__ int sharedMemory[BLOCK_SIZE][BLOCK_SIZE];

    int indexX = threadIdx.x + blockIdx.x * blockDim.x;
    int indexY = threadIdx.y + blockIdx.y * blockDim.y;

    int tindexX = threadIdx.x + blockIdx.y * blockDim.x;
    int tindexY = threadIdx.y + blockIdx.x * blockDim.y;

    int index = indexY * N + indexX;
    int transposedIndex = tindexY * N + tindexX;

    sharedMemory[threadIdx.x][threadIdx.y] = input[index];

    __syncthreads();

    output[transposedIndex] =
        sharedMemory[threadIdx.y][threadIdx.x];
}
```

## 주의할 점

### 경계 검사

현재 예제는 `N`이 `BLOCK_SIZE`로 정확히 나누어지는 것을 가정한다. 일반적인 크기의 행렬을 처리하려면 grid 크기를 올림으로 계산하고 입력 및 출력 경계를 검사해야 한다.

### Shared memory bank conflict

`32×32` 배열의 열 방향 접근은 shared memory bank conflict를 만들 수 있다. 전형적인 최적화 구현에서는 한 열을 padding으로 추가한다.

```cpp
__shared__ int tile[BLOCK_SIZE][BLOCK_SIZE + 1];
```

`+1` padding은 각 행의 시작 bank를 어긋나게 하여 전치 접근에서 발생하는 bank conflict를 줄인다.

## 정리

CUDA의 tiled matrix transpose는 두 종류의 전치를 결합한다.

```text
전체 행렬 전치
    = 블록 위치 전치
    + 블록 내부 원소 전치
```

- `blockIdx.x`와 `blockIdx.y`를 바꿔 출력 블록의 위치를 정한다.
- shared memory의 `[x][y]`와 `[y][x]`를 이용해 타일 내부를 전치한다.
- `__syncthreads()`로 shared memory 쓰기와 읽기 사이를 동기화한다.
- Global memory의 load와 store를 모두 coalesced하게 만드는 것이 shared memory를 사용하는 핵심 목적이다.
