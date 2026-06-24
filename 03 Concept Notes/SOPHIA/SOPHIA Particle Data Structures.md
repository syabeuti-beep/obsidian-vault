---
type: code-knowledge-note
topic: sophia-particle-data-structures
aliases:
  - SOPHIA 입자 자료구조
  - part1
  - part2
  - part3
tags:
  - sophia
  - data-structure
  - particle
  - cuda
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA Particle Data Structures

## 한 줄 요약

현재 [[SOPHIA]]는 particle 정보를 `part1`, `part2`, `part3`라는 AoS 구조체로 나누어 저장합니다. `part1`은 주로 현재 상태, `part2`는 reference/initial 상태, `part3`는 derivative/force/correction 값을 담는 작업 배열로 이해할 수 있습니다.

정의 파일:

```text
최신 SOPHIA코드/class_Cuda_Particle_Array2.cuh
```

## `part1`: 현재 particle state

`part1`은 현재 step에서 직접 움직이고 출력되는 핵심 particle state입니다.

주요 field:

| field | 의미 |
|---|---|
| `i_type` | inner/outer/dummy 등 내부 상태 |
| `buffer_type` | open boundary buffer type. 0 active, 1 inlet, 2 outlet |
| `p_type` | particle type. 예: FLUID, BOUNDARY, DEM_SOLID 등 |
| `dem_idx` | DEM particle index |
| `x,y,z` | 현재 또는 predicted 위치 |
| `x_star,y_star,z_star` | predictor 단계 위치 |
| `ux,uy,uz` | 속도 |
| `wx,wy,wz` | DEM angular velocity |
| `m` | 질량 |
| `ri` | rotational inertia |
| `rad` | DEM particle radius |
| `h` | smoothing length/kernel distance |
| `temp` | 온도 |
| `pres` | 압력 |
| `rho` | 밀도 |
| `grad_rhox/y/z` | density gradient |
| `pgf_x/y/z` | pressure gradient force term for DEM-SPH coupling |
| `Fdx_b/Fdy_b/Fdz_b` | DEM particle이 받는 pressure-gradient force |
| `Fdx_df/Fdy_df/Fdz_df` | DEM drag force 계열 |
| `Fdx_da/Fdy_da/Fdz_da` | SPH에 의한 drag force |
| `DEMpor` | SPH particle porosity due to DEM |
| `dDEMpor` | porosity change rate |
| `DEMvf` | DEM volume fraction |
| `ct_idx`, `del_s` | DEM contact index 및 tangential overlap |
| `vol_power` | volumetric power |
| `k_turb`, `e_turb` | turbulence variables |
| `cond` | conductivity |
| `vol0`, `vol` | reference/current volume |
| `elix,eliy,eliz` | Eulerian/ALE 관련 alpha-like marker로 보임 |
| `fbx/fby/fbz`, `fcx/fcy/fcz` | body/coupling force 계열 |
| `PPE1..PPE4` | PPE 관련 intermediate |
| `XSPH_ux/y/z`, `XSPH_temp` | XSPH correction |
| `OpenBC_*` | open boundary extrapolation 저장값 |

## `part2`: 초기/reference state

`part2`는 초기값이나 기준값을 저장합니다.

주요 field:

- `rho_ref`
- `x0,y0,z0`
- `ux0,uy0,uz0`
- `wx0,wy0,wz0`
- `rho0`, `drho0`
- `rad0`
- `vol0`, `dvol0`
- `concn0`, `enthalpy0`
- `temp0`, `temp10`, `temp20`, `temp30`
- turbulence strain rate `SR`

## `part3`: derivative/force/work array

`part3`는 각 step에서 계산되는 미분값, 힘, 보정 행렬, 표면장력/normal 등을 담습니다.

주요 field:

- `drho`, `dconcn`, `denthalpy`, `dtemp`
- `torqx/y/z`
- `ftotalx/y/z`, `ftotal`
- `fpx/fpy/fpz`: pressure force
- `vis_t`: turbulent viscosity
- `Sxx/Sxy/...`: strain tensor
- `Cm[3][3]`: correction matrix
- `nx/ny/nz`: normal vector
- `curv`: curvature
- `fsx/fsy/fsz`: surface tension force

## SPH와 DEM 분리

현재 코드는 전체 particle array `HP1`을 읽은 뒤 다음 배열로 분리합니다.

```text
HP1      : 전체 particle
HP1_sph  : SPH particle만
HP1_dem  : DEM particle만
```

GPU에서도 별도 배열을 유지합니다.

```text
dev_P1_sph, dev_SP1_sph, dev_P2_sph, dev_SP2_sph, dev_P3_sph
dev_P1_dem, dev_SP1_dem, dev_P2_dem, dev_SP2_dem, dev_P3_dem
```

이 구조 때문에 현재 코드는 순수 SPH 코드가 아니라 [[SOPHIA SPH-DEM Coupling]]에 맞게 재구성된 코드로 보는 것이 맞습니다.

## 코드 수정 시 주의점

- field를 추가하면 `part1/part2/part3` 구조체뿐 아니라 input parser, output writer, device copy 크기, VTK writer도 함께 확인해야 합니다.
- `part1`은 host/device copy가 잦기 때문에 구조체 크기 증가는 GPU memory와 bandwidth에 영향을 줍니다.
- SPH/DEM 분리 배열에 동일한 field가 필요한지, 한쪽에만 필요한지 구분해야 합니다.
- `dev_SP1`은 sorted particle array이고 `dev_P1`은 업데이트 대상/current array처럼 쓰입니다. 두 배열의 의미를 혼동하면 neighbor search 이후 값이 꼬일 수 있습니다.

## 관련 노트

- [[SOPHIA Input Files]]
- [[SOPHIA ISPH Solver]]
- [[SOPHIA SPH-DEM Coupling]]
- [[SOPHIA DEM Model]]
