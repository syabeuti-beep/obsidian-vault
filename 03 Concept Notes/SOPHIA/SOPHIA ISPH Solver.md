---
type: code-knowledge-note
topic: sophia-isph-solver
aliases:
  - SOPHIA ISPH
  - Incompressible SPH in SOPHIA
tags:
  - sophia
  - isph
  - sph
  - ppe
  - solver
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA ISPH Solver

## 한 줄 요약

현재 [[SOPHIA]]의 실제 실행 경로는 [[Incompressible SPH]] 기반 `ISPH` solver입니다. `solv.txt`에서 `solver_type = ISPH`로 설정되어 있고, `main()`은 `ISPH(vii,vif)`를 호출합니다.

## 이론 배경

`3. ISPH정리.pdf`는 ISPH 흐름을 다음 단계로 설명합니다.

- pressure distribution 계산
- pressure force 계산
- advection force 계산
- velocity update predictor
- velocity update corrector
- PPE 또는 Poisson-type pressure equation을 통한 incompressibility enforcement

초기 Theory Guide도 mass/momentum conservation, pressure force, viscous force, time integration을 설명합니다.

하지만 현재 해석은 반드시 `ISPH_Calc_seperate.cuh`의 kernel launch 순서를 기준으로 해야 합니다.

## 현재 코드의 ISPH 구성

주요 파일:

```text
ISPH.cuh
ISPH_Calc_seperate.cuh
function_PPE.cuh
function_TIME_ISPH.cuh
function_PREP.cuh
function_KNL.cuh
```

## Solver option

현재 `solv.txt`에서 ISPH 관련 핵심 옵션은 다음입니다.

```text
solver_type: ISPH
pressure-force solve: YES
viscous-force solve: YES
time integration: Predictor_Corrector
minimum-iteration: 3
maximum-iteration: 10
pressure-convergence criterion: 10
density-convergence criterion: 0.005
pressure-relaxation factor: 1
```

## 한 time step 내부의 ISPH 관련 순서

`SOPHIA_single_ISPH()`에서 SPH 쪽은 `decouple_stride` 조건을 만족할 때 실행됩니다.

```text
1. NNPS: cell indexing + sorting + reorder
2. mass/volume initialization
3. kernel gradient correction
4. preparation/filter/delta-SPH related values
5. advection force
6. predictor projection
7. pressure boundary condition
8. PPE solve
9. PPE calc
10. pressure boundary condition again
11. pressure force
12. time update projection/corrector
13. open boundary extrapolation
14. XSPH correction
```

## PPE 관련 코드

PPE 관련 kernel들은 `function_PPE.cuh`에 있습니다.

대표 함수:

- `KERNEL_advection_force3D_sph`
- `KERNEL_PPE3D_sph`
- `KERNEL_PPE3D_calc_sph`
- `KERNEL_pressureforce3D_sph`

`ISPH_Calc_seperate.cuh`에서 PPE는 다음처럼 호출됩니다.

```cpp
KERNEL_PPE3D_sph<<<...>>>(decouple_stride*dt, ...);
KERNEL_PPE3D_calc_sph<<<...>>>(decouple_stride*dt, ...);
KERNEL_pressureforce3D_sph<<<...>>>(1, ...);
```

`decouple_stride*dt`를 쓰는 점이 중요합니다. 현재 코드는 SPH와 DEM의 time step 분리를 고려하고 있습니다.

## Predictor-Corrector

현재 `solv.txt`의 `time integration`은 `Predictor_Corrector`입니다.

코드에서는 다음 흐름이 보입니다.

```text
KERNEL_clc_projection_sph
→ pressure/PPE solve
→ KERNEL_time_update_projection_sph
```

DEM 쪽은 별도로 `KERNEL_clc_projection_dem`, `KERNEL_time_update_projection_dem`이 호출됩니다.

## ISPH와 DEM coupling의 결합

현재 ISPH solver는 순수 유체 solver가 아닙니다. pressure/advection 이전에 [[SOPHIA SPH-DEM Coupling]] 준비가 들어갑니다.

```text
KERNEL_clc_prep3D_coupling_sph
KERNEL_clc_prep3D_coupling_dem
KERNEL_DEM_coupling3D_dem
KERNEL_SPH_coupling3D_sph
```

따라서 `dev_P3_sph.ftotalx` 등에는 유체 힘뿐 아니라 DEM 반작용 항도 들어갈 수 있습니다.

## 에이전트가 이해해야 할 핵심

- ISPH는 incompressibility를 압력장 계산으로 맞추는 solver입니다.
- 현재 구현은 `PPE` 관련 kernel을 통해 pressure를 계산하고, pressure force를 `part3`에 저장한 뒤 time update합니다.
- 현재 코드는 DEM과 분리 time stepping을 고려하므로 `dt`가 그대로 쓰이는 곳과 `decouple_stride*dt`가 쓰이는 곳을 구분해야 합니다.
- 새로운 물리 모델을 추가할 때는 preparation → force calculation → pressure solve 전후 위치를 잘 정해야 합니다.

## 관련 노트

- [[SOPHIA Simulation Execution Flow]]
- [[SOPHIA Particle Data Structures]]
- [[SOPHIA SPH-DEM Coupling]]
- [[SOPHIA Open Boundary Condition]]
