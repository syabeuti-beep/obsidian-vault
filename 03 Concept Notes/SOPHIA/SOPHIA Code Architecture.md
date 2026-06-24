---
type: code-knowledge-note
topic: sophia-code-architecture
aliases:
  - SOPHIA 아키텍처
  - SOPHIA 코드 구조
tags:
  - sophia
  - code-architecture
  - cuda
  - sph-dem
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA Code Architecture

## 한 줄 요약

현재 [[SOPHIA]] 코드는 `SOPHIA_gpu.cu` 하나가 여러 `.cuh` 파일을 include해서 단일 CUDA translation unit처럼 빌드되는 구조입니다. 실행은 `main()`에서 시작하지만 실제 계산은 [[SOPHIA ISPH Solver]]와 `ISPH_Calc_seperate.cuh`의 time-step 루프가 담당합니다.

## 빌드 구조

`Makefile`의 현재 컴파일 명령은 다음 형태입니다.

```text
nvcc -w -use_fast_math -arch=sm_60 -O3 -expt-relaxed-constexpr SOPHIA_gpu.cu -o SOPHIA_gpu -I./cub-1.8.0/ -lpthread
```

의미:

- `nvcc`: CUDA compiler
- `-arch=sm_60`: Pascal 세대 이상의 GPU target
- `-O3`, `-use_fast_math`: 성능 최적화
- `-I./cub-1.8.0/`: CUB sorting/reduction 라이브러리 include
- `-lpthread`: multi-GPU thread 실행용 pthread

현재 Mac mini가 NVIDIA CUDA GPU를 갖고 있지 않다면 이 코드는 여기서 직접 실행하기 어렵고, CUDA 가능한 Linux 서버/GPU 머신에서 실행해야 합니다.

## Entry point

`SOPHIA_gpu.cu`의 `main()` 흐름은 현재 코드 기준으로 다음과 같습니다.

```cpp
memset(vii, 0, sizeof(int_t)*vii_size);
memset(vif, 0, sizeof(Real)*vif_size);

ngpu = atoi(argv[1]);

read_solv_input(vii, vif, "./input/solv.txt");
read_table("./input/data.txt");

switch(solver_type){
  case Wcsph: break;
  case Isph: ISPH(vii, vif); break;
  case Dfsph: break;
}
```

즉 현재 샘플 `solv.txt`가 `ISPH`를 선택하고 있으므로 실질 실행 경로는 다음입니다.

```text
SOPHIA_gpu.cu main()
→ read_solv_input
→ read_table
→ ISPH(vii, vif)
→ pthread로 ISPH_Calc 실행
→ SOPHIA_single_ISPH 반복
```

## Include graph

`SOPHIA_gpu.cu`는 다음 순서로 핵심 모듈을 include합니다.

```text
Cuda_Error.cuh
Variable_Type.cuh
Parameters.cuh
class_Cuda_Particle_Array2.cuh
function_init.cuh
function_NNPS.cuh
function_ALE.cuh
function_PROP.cuh
function_MASS.cuh
function_KNL.cuh
function_PREP.cuh
function_PPE.cuh
function_DEM_INTERACTION.cuh
function_SPH_DEM_COUPLING.cuh
function_BC.cuh
function_TIME_ISPH.cuh
function_DEM_BC.cuh
function_XSPH.cuh
function_OUTPUT.cuh
function_INPUT_GEN.cuh
ISPH_Calc_seperate.cuh
ISPH.cuh
```

이 순서가 중요합니다. 앞의 파일에서 정의된 구조체, 상수, 전역 변수, kernel이 뒤의 파일에서 사용됩니다.

## 모듈별 책임

| 파일 | 책임 |
|---|---|
| `SOPHIA_gpu.cu` | main, 전역 변수, include graph, solver dispatch |
| `Parameters.cuh` | solver/particle/kernel/model 상수 |
| `function_init.cuh` | `solv.txt`, `data.txt`, `input.txt` 읽기 |
| `class_Cuda_Particle_Array2.cuh` | particle AoS 구조체 정의 |
| `ISPH.cuh` | 상위 solver 실행, 입자 분리, GPU domain setup |
| `ISPH_Calc_seperate.cuh` | time-step main loop와 CUDA kernel launch 순서 |
| `function_NNPS.cuh` | cell indexing, sorting, neighbor search |
| `function_PREP.cuh` | gradient correction, filter, porosity/preparation |
| `function_PPE.cuh` | pressure Poisson equation, pressure/advection force |
| `function_DEM_INTERACTION.cuh` | DEM contact force |
| `function_SPH_DEM_COUPLING.cuh` | SPH↔DEM two-way coupling |
| `function_DEM_BC.cuh` | DEM wall collision/sliding/rolling |
| `function_BC.cuh` | wall/open boundary pressure/extrapolation |
| `function_TIME_ISPH.cuh` | predictor/corrector, time update |
| `function_OUTPUT.cuh` | VTK/restart 저장 |

## 현재 코드에서 보이는 큰 변화

초기 매뉴얼은 일반적인 SOPHIA SPH 코드 설명이지만, 현재 코드는 다음 특징이 강합니다.

- SPH 입자와 DEM 입자를 분리해서 `HP1_sph`, `HP1_dem`, `dev_P1_sph`, `dev_P1_dem` 등으로 별도 관리합니다.
- `decouple_stride`를 사용해서 SPH와 DEM time step을 분리하는 구조가 들어가 있습니다.
- 출력 경로가 현재는 `save_plot_fluid_vtk_bin_solid_dem(file_P1_dem,file_P3_dem)` 중심으로 설정되어 있어 DEM 쪽 출력이 활성화되어 있습니다.
- `input_generated`에 DEM 상태를 다시 저장하는 기능이 있습니다.

## 관련 노트

- [[SOPHIA Simulation Execution Flow]]
- [[SOPHIA ISPH Solver]]
- [[SOPHIA Particle Data Structures]]
- [[SOPHIA CUDA GPU Parallelization]]
