---
type: code-knowledge-note
topic: sophia-simulation-execution-flow
aliases:
  - SOPHIA 실행 흐름
tags:
  - sophia
  - execution-flow
  - isph
  - cuda
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA Simulation Execution Flow

## 전체 실행 흐름

현재 코드 기준 실행 흐름은 다음입니다.

```text
./SOPHIA_gpu <ngpu>
        │
        ▼
main() in SOPHIA_gpu.cu
        │
        ├─ read_solv_input(vii, vif, "./input/solv.txt")
        ├─ read_table("./input/data.txt")
        └─ ISPH(vii, vif)
                │
                ├─ GPU device properties 출력
                ├─ gpu_count_particle_numbers2("./input/input.txt")
                ├─ read_input(HP1)
                ├─ SPH/DEM 입자 수 계산
                ├─ read_sph_dem(HP1, HP1_sph, HP1_dem)
                ├─ domain/cell 범위 설정
                ├─ pthread로 GPU별 ISPH_Calc 실행
                └─ free host memory
```

## `main()` 단계

`main()`은 세 가지를 수행합니다.

1. `vii`, `vif` solver parameter array 초기화
2. `solv.txt`, `data.txt` 읽기
3. `solver_type`에 따라 solver 호출

현재 `input/solv.txt`는 `solver_type = ISPH`입니다. 따라서 `WCSPH`, `DFSPH` 코드는 분기로만 남아 있고 실제 경로는 [[SOPHIA ISPH Solver]]입니다.

## `ISPH()` 단계

`ISPH.cuh`의 `ISPH(vii,vif)`는 다음 일을 합니다.

1. CUDA device 정보 출력
2. `./input/input.txt`에서 전체 입자 개수 계산
3. `HP1` host particle array 할당
4. `read_input(HP1)`로 particle input 읽기
5. `gpu_count_particle_numbers_sph`, `gpu_count_particle_numbers_dem`으로 SPH/DEM 개수 계산
6. `HP1_sph`, `HP1_dem`을 따로 할당
7. `read_sph_dem`으로 particle을 SPH/DEM 배열로 분리
8. domain min/max 및 cell 크기 설정
9. GPU별 pthread 생성
10. 각 thread에서 `ISPH_Calc` 실행

## 현재 코드의 domain 설정 주의점

`ISPH.cuh`에는 `find_minmax(vii,vif,HP1)` 호출 뒤에 다음 값들이 하드코딩되어 있습니다.

```cpp
x_max = 0.09;
x_min = -0.01;
y_max = 0.02;
y_min = -0.01;
z_max = 0.251;
z_min = -0.01;
```

따라서 새로운 시뮬레이션 geometry를 만들 때 `input.txt`만 바꾸면 충분하지 않을 수 있습니다. 계산 영역이 현재 코드에서 하드코딩된 범위를 벗어나면 cell indexing/neighbor search 문제가 생길 수 있습니다.

이 부분은 향후 SOPHIA 에이전트가 시뮬레이션 케이스를 생성할 때 반드시 확인해야 할 수정 후보입니다.

## `ISPH_Calc`와 `SOPHIA_single_ISPH`

`ISPH_Calc_seperate.cuh`에는 두 수준의 함수가 있습니다.

- `ISPH_Calc(void* arg)`: GPU thread별 메모리 할당, device copy, CUB storage 준비, time loop 실행
- `SOPHIA_single_ISPH(...)`: 매 time step마다 실제 물리 kernel들을 순서대로 launch

## 한 time step의 주요 계산 순서

현재 코드 주석과 kernel launch 순서를 기준으로 하면 한 step은 대략 다음입니다.

```text
1. ESPH/ALE alpha 초기화
2. NNPS: cell index 계산, CUB sorting, reorder
3. P3 derivative/force array reset
4. mass/volume 초기화
5. kernel gradient correction
6. SPH-DEM coupling 준비: porosity/filter 계산
7. SPH filter/delta-SPH/normal gradient 준비
8. advection force 계산
9. DEM-DEM contact force 계산
10. DEM이 받는 SPH coupling force 계산
11. SPH가 받는 DEM reaction force 계산
12. predictor velocity/position update
13. pressure boundary condition
14. PPE pressure solve
15. PPE calc/update
16. pressure force 계산
17. corrector/time update
18. open boundary extrapolation
19. DEM wall boundary condition
20. XSPH correction
21. output 저장
22. DEM input_generated 저장
```

## 출력 흐름

`freq_output`마다 다음과 같은 출력 경로가 실행됩니다.

```cpp
save_plot_fluid_vtk_bin_solid_dem(file_P1_dem, file_P3_dem);
```

즉 현재 활성 출력은 DEM solid 쪽 VTK 출력으로 보입니다. SPH fluid 출력 함수들은 코드에 남아 있지만 주석 처리되어 있습니다.

## 다음에 확인할 질문

- `input/input.txt`의 정확한 column format을 초기 Code Manual과 현재 `function_init.cuh` 기준으로 재구성해야 합니다.
- `p_type.txt`가 `input.txt` 생성용 원천 데이터인지, 별도 전처리 산출물인지 확인해야 합니다.
- 하드코딩된 domain 범위를 사용자 입력 기반으로 일반화할지 결정해야 합니다.
- 출력이 DEM만 저장되도록 되어 있는 현재 상태가 의도인지 확인해야 합니다.

## 관련 노트

- [[SOPHIA Code Architecture]]
- [[SOPHIA Input Files]]
- [[SOPHIA ISPH Solver]]
- [[SOPHIA CUDA GPU Parallelization]]
