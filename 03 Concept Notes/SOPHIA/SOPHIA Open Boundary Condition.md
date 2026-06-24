---
type: code-knowledge-note
topic: sophia-open-boundary-condition
aliases:
  - SOPHIA OpenBC
  - OpenBC
tags:
  - sophia
  - open-boundary
  - boundary-condition
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA Open Boundary Condition

## 한 줄 요약

현재 [[SOPHIA]] 코드는 `solv.txt`에서 `open boundary: OPEN`을 지원하며, SPH 입자에 대해 open boundary extrapolation kernel을 호출합니다. OpenBC 이론 문서는 dummy particle과 Dirichlet/Neumann 조건을 설명하지만, 실제 구현은 `function_BC.cuh`와 `ISPH_Calc_seperate.cuh`의 kernel 호출을 기준으로 봐야 합니다.

## 이론 자료

관련 PDF:

- `4. OpenBC정리.pdf`
- `SOPHIA 초기 Theory Guide.pdf`

이론 키워드:

- Dirichlet boundary condition
- Neumann boundary condition
- dummy particle
- boundary에 대칭된 dummy particle 생성
- 경계 조건이 만족되도록 물리량 부여

## 코드 기준 파일

```text
function_BC.cuh
ISPH_Calc_seperate.cuh
Parameters.cuh
class_Cuda_Particle_Array2.cuh
```

## option 입력

`solv.txt`에는 다음 줄이 있습니다.

```text
open boundary (OPEN/PERIODIC/NO): OPEN
```

`read_solv_input()`은 이 값을 다음처럼 해석합니다.

```text
OPEN     → open_boundary = 1
PERIODIC → open_boundary = 2
NO       → open_boundary = 0
```

## particle field

`part1`에는 OpenBC 관련 field가 있습니다.

```text
buffer_type       # 0 active, 1 inlet, 2 outlet
OpenBC_pres
OpenBC_rho
OpenBC_m
OpenBC_temp
OpenBC_ux, OpenBC_uy, OpenBC_uz
```

`input.txt` label에는 다음이 있습니다.

```text
open_buffer_type = 37
```

따라서 open boundary particle을 만들 때는 `buffer_type` 또는 label 37을 제대로 설정해야 할 가능성이 큽니다.

## 현재 호출 흐름

`SOPHIA_single_ISPH()`에서 open boundary가 켜져 있으면 다음이 호출됩니다.

```cpp
if(open_boundary > 0) {
    KERNEL_open_boundary_extrapolation3D_1_sph(...);
    KERNEL_open_boundary_extrapolation3D_1_calc_sph(...);
}
```

이 호출은 pressure solve와 time update 이후에 위치합니다.

## 주의점

- `ISPH.cuh`에는 open boundary용 가상 격자 간격 `space = h0*1.6`, `Nsx`, `Nsy`, `buffer_size` 계산이 있으나 현재 `num_part2=num_part`로 되어 있고 buffer size 추가가 주석 처리되어 있습니다.
- 따라서 현재 OpenBC가 “입자 생성까지 자동 수행”하는지, 아니면 이미 존재하는 inlet/outlet buffer particle의 물성 extrapolation만 하는지 확인이 필요합니다.
- `function_BC.cuh`의 kernel 내부를 더 읽어야 실제 inlet/outlet 처리 규칙을 확정할 수 있습니다.

## 에이전트가 입력파일 생성 시 확인할 것

1. inlet/outlet particle을 `input.txt`에 명시해야 하는가?
2. `buffer_type` 1/2를 어떤 입자에 부여해야 하는가?
3. inlet velocity, density, temperature는 `Parameters.cuh`의 `Inlet_*` 상수를 쓰는가, 아니면 `input.txt` field를 쓰는가?
4. domain boundary와 `ISPH.cuh`의 하드코딩 범위가 일치하는가?

## 관련 노트

- [[SOPHIA Input Files]]
- [[SOPHIA ISPH Solver]]
- [[SOPHIA Simulation Execution Flow]]
