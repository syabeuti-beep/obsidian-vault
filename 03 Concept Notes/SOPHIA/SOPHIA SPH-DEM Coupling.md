---
type: code-knowledge-note
topic: sophia-sph-dem-coupling
aliases:
  - SOPHIA SPH DEM Coupling
  - SPH-DEM Coupling
tags:
  - sophia
  - sph-dem
  - coupling
  - dem
  - sph
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA SPH-DEM Coupling

## 한 줄 요약

현재 [[SOPHIA]] 최신 코드는 SPH 유체 입자와 DEM 고체 입자를 분리 관리하면서, pressure-gradient force와 drag force 계열을 통해 two-way coupling을 계산합니다. 열전달이 켜져 있으면 같은 coupling kernel에서 고체-유체 convective heat transfer도 계산하므로, 논문 구현 시 [[SOPHIA Solid-Fluid Heat Transfer]]와 [[SOPHIA Paper Equation Audit Workflow]]를 함께 확인해야 합니다.

## 이론 자료

관련 PDF:

- `5. SPH-DEM Coupling정리.pdf`
- `SOPHIA에 DEM결합 추가 논문.pdf`

핵심 이론 키워드:

- Averaged Volume Navier-Stokes Equation, AVNS
- porosity `ε`
- DEM coupling force
- pressure-gradient force
- drag force
- SPH reaction force
- unresolved SPH-DEM coupling
- debris bed/self-leveling

## 코드 기준 파일

```text
function_SPH_DEM_COUPLING.cuh
function_PREP.cuh
function_DEM_INTERACTION.cuh
ISPH_Calc_seperate.cuh
class_Cuda_Particle_Array2.cuh
```

## 현재 코드의 coupling 흐름

`SOPHIA_single_ISPH()` 안에서 coupling 관련 순서는 다음입니다.

```text
1. SPH/DEM 각각 NNPS 정렬
2. KERNEL_clc_prep3D_coupling_sph
   - SPH particle의 DEMpor 등 계산
3. KERNEL_clc_prep3D_coupling_dem
   - DEM particle 주변 SPH filter 값 등 계산
4. KERNEL_DEM_interaction3D_dem
   - DEM-DEM contact force 계산
5. KERNEL_DEM_coupling3D_dem
   - DEM 입자가 SPH로부터 받는 pressure-gradient/drag force 계산
6. KERNEL_SPH_coupling3D_sph
   - SPH 입자가 DEM으로부터 받는 reaction force 계산
7. ISPH pressure/PPE/time update로 진행
```

## particle field 연결

`part1`에서 coupling 관련 field는 다음이 중요합니다.

| field | 의미 |
|---|---|
| `DEMpor` | SPH particle porosity due to DEM particle |
| `dDEMpor` | porosity change rate |
| `dDEMpor_prev` | previous porosity change rate |
| `DEMvf` | DEM volume fraction immersed/sinked into fluid |
| `pgf_x/y/z` | pressure gradient force term |
| `Fdx_b/Fdy_b/Fdz_b` | DEM이 받는 pressure-gradient force 계열 |
| `Fdx_df/Fdy_df/Fdz_df` | drag force 계열 |
| `Fdx_da/Fdy_da/Fdz_da` | SPH에 의한 drag/action force 계열 |

`part3`에서는 `ftotalx/y/z`, `fpx/y/z` 등이 최종 force/time update에 관여합니다.

## 문서 이론의 해석

`5. SPH-DEM Coupling정리.pdf`는 coupling force를 크게 다음으로 나눕니다.

```text
F_a = F_a^P + F_a^D
```

- `F_a^P`: pressure gradient force
- `F_a^D`: drag force

그리고 SPH 유체 방정식에는 DEM이 유체에 주는 반작용 force source term이 들어갑니다.

현재 코드에서도 이 개념이 다음 함수명으로 반영되어 있습니다.

```text
KERNEL_DEM_coupling3D_dem
KERNEL_SPH_coupling3D_sph
```

## 현재 코드의 coupling 이산화 형태

SOPHIA coupling은 기본적으로 자기 DEM 또는 SPH 입자 `i` 주변의 이웃 `j`를 kernel로 합산합니다. 자세한 SPH kernel 정의와 보간식은 [[SOPHIA SPH Discrete Equations]]를 보세요.

### DEM 입자 주변 유체 보간

`KERNEL_DEM_coupling3D_dem()`는 DEM 입자 `i` 주변 SPH 입자 속도와 온도를 다음처럼 보간합니다.

$$
\mathbf{u}_{f,i}=
\frac{\sum_{j\in SPH}\mathbf{u}_j\frac{m_j}{\rho_j}W_{ij}}
{\sum_{j\in SPH}\frac{m_j}{\rho_j}W_{ij}}
$$

$$
T_{f,i}=
\frac{\sum_{j\in SPH}T_j\frac{m_j}{\rho_j}W_{ij}}
{\sum_{j\in SPH}\frac{m_j}{\rho_j}W_{ij}}
$$

이 보간값으로 drag와 [[SOPHIA Solid-Fluid Heat Transfer]] source를 계산합니다.

### DEM이 받는 pressure-gradient force

`KERNEL_advection_force3D_sph()`에서 SPH 입자 `j`에 저장한 `pgf_x/y/z`를 DEM 입자 `i`로 다시 보간합니다.

코드 형태:

```cpp
tmpx += pgf_xj * (mj/rhoj) * W_ij * V_i / (m_i * flt_fluid);
```

수식으로는 대략 다음 꼴입니다.

$$
\mathbf{a}_{p,i}^{DEM}
= \sum_{j\in SPH}
\mathbf{G}_{p,j}\frac{m_j}{\rho_j}W_{ij}
\frac{V_i}{m_i F_i}
$$

### SPH가 받는 DEM drag/heat 반작용

`KERNEL_SPH_coupling3D_sph()`는 DEM 입자 `j`에 저장된 drag acceleration과 heat source를 SPH 입자 `i`에 분배합니다.

$$
\mathbf{a}_{i}^{SPH,coupling}
= -\sum_{j\in DEM}\frac{m_j}{\rho_i}\frac{\mathbf{a}_{D,j}W_{ij}}{F_j}
$$

열전달 반작용은 다음 꼴입니다.

$$
\left(\frac{dT}{dt}\right)_i^{SPH,coupling}
= -\sum_{j\in DEM}\frac{Q_{sd,j}}{\rho_i C_{p,i}}\frac{W_{ij}}{F_j}
$$

논문 식이 Eulerian/grid source term이면, 이 SPH 분배식과 보존성이 맞는지 별도 audit이 필요합니다.

## decouple stride

현재 `solv.txt`에는 다음 옵션이 있습니다.

```text
decouple-stride: 1
```

코드에서는 SPH 관련 kernel 상당수가 다음 조건 아래 실행됩니다.

```cpp
if((count % decouple_stride) == 0) { ... }
```

DEM은 매 step 계산되고, SPH는 `decouple_stride`마다 계산될 수 있는 구조입니다. 이는 DEM과 SPH의 time scale을 분리하기 위한 구조로 보입니다.

## 에이전트가 시뮬레이션을 만들 때 확인할 것

- DEM particle의 `rad`, `ri`, `m`, `rho`, `dem_idx`가 적절한가?
- fluid/SPH particle과 DEM particle이 `p_type`으로 제대로 구분되는가?
- `num_part_sph`, `num_part_dem`이 코드에서 올바르게 계산되는가?
- `decouple_stride`가 물리적으로 적절한가?
- SPH/DEM interaction radius가 `h`, `kappa`, `rad`에 비해 적절한가?
- 출력이 DEM만 저장되도록 되어 있는지, 유체 field도 필요한지?

## 관련 노트

- [[SOPHIA DEM Model]]
- [[SOPHIA ISPH Solver]]
- [[SOPHIA SPH Discrete Equations]]
- [[SOPHIA Solid-Fluid Heat Transfer]]
- [[SOPHIA Paper Equation Audit Workflow]]
- [[SOPHIA Particle Data Structures]]
- [[SOPHIA Simulation Execution Flow]]
