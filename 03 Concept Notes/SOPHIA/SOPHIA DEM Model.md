---
type: code-knowledge-note
topic: sophia-dem-model
aliases:
  - SOPHIA DEM
  - Discrete Element Method
tags:
  - sophia
  - dem
  - solid-particle
  - contact-model
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA DEM Model

## 한 줄 요약

[[SOPHIA]]의 DEM 부분은 고체 입자의 병진/회전 운동, 입자-입자 contact, 벽 충돌, sliding/rolling을 계산하며, 현재 코드에서는 [[SOPHIA SPH-DEM Coupling]]과 결합되어 유체-고체 상호작용을 계산합니다.

## 이론 자료

관련 PDF:

- `1. DEM정리.pdf`
- `SOPHIA에 DEM결합 추가 논문.pdf`

핵심 내용:

- Newton's second law of motion
- soft-sphere collision model
- Hertz-Mindlin contact force
- normal/tangential force
- wall collision
- sliding and rolling
- restitution coefficient
- friction coefficient
- torque/angular velocity

## 코드 기준 파일

```text
function_DEM_INTERACTION.cuh
function_DEM_BC.cuh
function_TIME_ISPH.cuh
function_SPH_DEM_COUPLING.cuh
class_Cuda_Particle_Array2.cuh
```

## DEM particle state

`part1`에서 DEM에 중요한 field:

| field | 의미 |
|---|---|
| `x,y,z` | 위치 |
| `ux,uy,uz` | 병진 속도 |
| `wx,wy,wz` | 각속도 |
| `m` | 질량 |
| `ri` | 회전 관성 |
| `rad` | 입자 반지름 |
| `dem_idx` | DEM index |
| `ct_idx[16]` | contact particle index |
| `del_s[16][3]` | tangential overlap 누적값 |

`part3`에서 DEM에 중요한 field:

- `ftotalx/y/z`
- `torqx/y/z`
- `ftotal`

## DEM-DEM contact

`ISPH_Calc_seperate.cuh`에서는 다음 kernel이 DEM contact를 계산합니다.

```cpp
KERNEL_DEM_interaction3D_dem<<<...>>>(1, g_str_dem, g_end_dem, dev_SP1_dem, dev_P1_dem, dev_P3_dem, time);
```

이 kernel은 DEM 입자끼리의 접촉에 의한 힘과 토크를 `dev_P3_dem`에 저장하는 것으로 주석에 설명되어 있습니다.

## DEM wall boundary condition

벽 경계는 다음 kernel에서 처리됩니다.

```cpp
KERNEL_treat_DEM_box_x_dem<<<...>>>(1, dev_SP1_dem, dev_P1_dem, dev_SP2_dem, dev_P3_dem);
KERNEL_treat_DEM_box_y_dem<<<...>>>(1, dev_SP1_dem, dev_P1_dem, dev_SP2_dem, dev_P3_dem);
```

현재 코드 기준으로 x, y 방향 box wall 처리가 활성화되어 있습니다. z 방향 함수 호출은 현재 보이지 않으므로, 3D geometry에서 z wall이 필요한 경우 확인해야 합니다.

## DEM time update

Predictor와 corrector/time update는 유체와 별도로 DEM kernel이 호출됩니다.

```cpp
KERNEL_clc_projection_dem<<<...>>>(count, dt, time, dev_SP1_dem, dev_SP2_dem, dev_P3_dem);
KERNEL_time_update_projection_dem<<<...>>>(dt, dev_SP1_dem, dev_P1_dem, dev_SP2_dem, dev_P2_dem, dev_P3_dem);
```

SPH는 `decouple_stride*dt`를 쓰는 경우가 많고, DEM은 `dt`를 직접 쓰는 구조입니다.

## DEM output

현재 활성 출력은 DEM solid VTK입니다.

```cpp
save_plot_fluid_vtk_bin_solid_dem(file_P1_dem, file_P3_dem);
```

또한 0.5초마다 DEM 입자 상태를 `input_generated` 폴더에 저장하는 코드가 있습니다.

```cpp
save_input_dem(file_P1_dem);
```

## 시뮬레이션 생성 시 필요한 DEM 입력

DEM 입자를 만들려면 적어도 다음 값들이 필요합니다.

- 위치 `x,y,z`
- 속도 `ux,uy,uz`
- 질량 `m`
- particle type `p_type`
- smoothing length `h` 또는 search 관련 길이
- DEM radius `rad`
- rotational inertia `ri`
- DEM index `dem_idx`
- angular velocity `wx,wy,wz`

## 관련 노트

- [[SOPHIA SPH-DEM Coupling]]
- [[SOPHIA Particle Data Structures]]
- [[SOPHIA Input Files]]
- [[SOPHIA CUDA GPU Parallelization]]
