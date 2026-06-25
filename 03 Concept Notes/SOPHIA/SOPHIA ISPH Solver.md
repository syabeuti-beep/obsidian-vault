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

## 주요 SPH 이산화 식

현재 ISPH solver의 각 kernel은 자기 입자 `i`와 이웃 입자 `j` 사이의 kernel 합산으로 항을 만듭니다. 자세한 kernel 정의와 수식은 [[SOPHIA SPH Discrete Equations]]를 보세요.

### Pressure/advection force 쪽 기본 형태

`function_PPE.cuh`의 `KERNEL_advection_force3D_sph()`는 pressure-gradient, viscous, artificial viscosity, surface tension, heat diffusion 등을 한 이웃 loop 안에서 계산합니다.

Pressure-gradient coupling source는 대략 다음 형태입니다.

$$
\mathbf{G}_{p,i}
= \frac{\rho_i}{\epsilon_i}
\sum_j \left[-\epsilon_i m_j\frac{p_i+p_j}{\rho_i\rho_j}\right]
\nabla_i^c W_{ij}
$$

Viscous force는 코드상 다음 계수로 계산됩니다.

$$
C_{v,ij}
=4\epsilon_i\frac{m_j}{\rho_i\rho_j}
\frac{\mu_i\mu_j}{\mu_i+\mu_j}
\frac{\mathbf{r}_{ij}\cdot\nabla_i^cW_{ij}}{r_{ij}^2}
$$

$$
\mathbf{a}_{v,i}=\sum_j C_{v,ij}(\mathbf{u}_i-\mathbf{u}_j)
$$

### Porosity thermal diffusion

현재 열전도가 켜져 있으면 유체 온도 방정식에는 porosity-modified diffusion이 들어갑니다.

$$
\chi_i=(1-\sqrt{1-\epsilon_i})k_{f,i}
$$

$$
\chi_{ij}^{H}=\frac{2\chi_i\chi_j}{\chi_i+\chi_j}
$$

$$
\frac{dT_i}{dt}
=\frac{1}{C_{p,i}}
\sum_j \frac{2m_j}{\rho_j}\frac{\chi_{ij}^{H}}{\epsilon_i\rho_i}
(T_i-T_j)\frac{\mathbf{r}_{ij}\cdot\nabla_i^cW_{ij}}{r_{ij}^2+\eta^2}
-\text{Eulerian advection term}
$$

논문 열전달식을 구현할 때 이 fluid conduction 식과 [[SOPHIA Solid-Fluid Heat Transfer]]의 DEM-fluid source 식을 모두 확인해야 합니다.

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
- [[SOPHIA SPH Discrete Equations]]
- [[SOPHIA Solid-Fluid Heat Transfer]]
- [[SOPHIA Paper Equation Audit Workflow]]
- [[SOPHIA Particle Data Structures]]
- [[SOPHIA SPH-DEM Coupling]]
- [[SOPHIA Open Boundary Condition]]
