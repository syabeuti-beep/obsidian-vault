---
type: code-knowledge-note
topic: sophia-cuda-gpu-parallelization
aliases:
  - SOPHIA CUDA
  - SOPHIA GPU
tags:
  - sophia
  - cuda
  - gpu
  - parallelization
  - hpc
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA CUDA GPU Parallelization

## 한 줄 요약

현재 [[SOPHIA]]는 CUDA kernel, CUB sorting/reduction, pthread 기반 multi-GPU 실행 구조를 사용합니다. particle method 특성상 가장 중요한 병렬화 패턴은 “particle마다 kernel thread를 배정하고, neighbor search를 cell indexing + sorting으로 가속하는 것”입니다.

## 관련 파일

```text
SOPHIA_gpu.cu
ISPH.cuh
ISPH_Calc_seperate.cuh
function_NNPS.cuh
function_OUTPUT.cuh
Cuda_Error.cuh
```

## 실행 인자

`main()`에서 GPU 개수는 command line argument로 받습니다.

```cpp
ngpu = atoi(argv[1]);
```

실행 예시:

```text
./SOPHIA_gpu 1
./SOPHIA_gpu 2
```

## pthread 구조

`ISPH.cuh`에서 GPU 개수만큼 thread를 만듭니다.

```cpp
pthread_t* solve_thread;
for(int i=0;i<ngpu;i++) {
    tid[i] = i;
    pthread_create(&solve_thread[i], NULL, ISPH_Calc, (void*)&tid[i]);
}
```

각 thread는 `ISPH_Calc()` 안에서 다음을 수행합니다.

```cpp
cudaSetDevice(tid);
cudaStreamCreate(&str1[tid]);
cudaStreamCreate(&str2[tid]);
```

## GPU memory 구조

SPH/DEM 분리 이후 GPU에도 별도 배열을 둡니다.

```text
dev_P1_sph, dev_SP1_sph, dev_P2_sph, dev_SP2_sph, dev_P3_sph
dev_P1_dem, dev_SP1_dem, dev_P2_dem, dev_SP2_dem, dev_P3_dem
```

`P`와 `SP`는 현재/정렬된 particle array 역할로 보입니다.

## NNPS: neighbor search 병목

SPH/DEM 모두 주변 입자 탐색이 필요합니다. 현재 코드는 다음 순서를 사용합니다.

```text
1. particle → cell index 계산
2. CUB DeviceRadixSort::SortPairs로 cell index 기준 정렬
3. reorder kernel로 sorted particle array 생성
4. g_str/g_end로 각 cell의 입자 범위 저장
5. 이후 kernel들이 neighbor cell을 순회
```

이 흐름은 `function_NNPS.cuh`와 `ISPH_Calc_seperate.cuh`에 구현되어 있습니다.

## CUB 사용

현재 코드는 CUB를 사용합니다.

```cpp
cub::DeviceRadixSort::SortPairs(...)
cub::DeviceReduce::Max(...)
```

CUB는 정렬과 최대값 reduction에 사용됩니다.

## multi-GPU 경계 교환

코드에는 `p2p`, `send_P1`, `recv_P1`, `cudaDeviceEnablePeerAccess` 등이 있습니다.

```cpp
cudaDeviceEnablePeerAccess(j,0);
```

다만 현재 시뮬레이션 생성 에이전트의 첫 목표는 single-GPU case가 더 안전합니다. multi-GPU는 domain division, ghost/p2p particle, boundary exchange까지 검증해야 합니다.

## 성능/실행 환경 주의

- Mac mini에는 일반적으로 NVIDIA CUDA GPU가 없으므로 직접 실행이 안 될 가능성이 높습니다.
- 실제 실행은 CUDA 가능한 Linux 서버에서 해야 합니다.
- `Makefile`은 `sm_60`과 `cub-1.8.0` include를 가정합니다. 현재 폴더에 `cub-1.8.0`가 없으면 빌드가 실패할 수 있습니다.
- `SOPHIA_gpu` binary가 이미 있으나, Mac에서는 Linux/CUDA binary일 가능성이 높아 실행 불가일 수 있습니다.

## 관련 노트

- [[SOPHIA Code Architecture]]
- [[SOPHIA Simulation Execution Flow]]
- [[SOPHIA Particle Data Structures]]
