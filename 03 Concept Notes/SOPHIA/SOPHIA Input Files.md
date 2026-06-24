---
type: code-knowledge-note
topic: sophia-input-files
aliases:
  - SOPHIA 입력파일
  - solv.txt
  - data.txt
  - input.txt
  - p_type.txt
tags:
  - sophia
  - input-file
  - simulation-setup
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA Input Files

## 한 줄 요약

현재 [[SOPHIA]]를 원하는 시뮬레이션에 맞게 돌리려면 `input/solv.txt`, `input/data.txt`, 그리고 현재 폴더에 없는 `input/input.txt`를 올바르게 만들어야 합니다. 코드 기준으로 실제 입자 좌표/물성/초기조건은 `input.txt`에서 읽습니다.

## 현재 복사된 input 폴더

현재 폴더에는 다음 파일이 있습니다.

```text
input/solv.txt
input/data.txt
input/p_type.txt
```

그러나 `ISPH.cuh`와 `function_init.cuh`는 실제 입자 입력을 다음 파일에서 읽습니다.

```text
./input/input.txt
```

따라서 현재 복사본은 즉시 실행 가능한 full case라기보다, solver 설정과 일부 보조 데이터가 포함된 상태로 보입니다.

## `solv.txt`

`solv.txt`는 solver option을 사람이 읽기 쉬운 label과 값으로 저장합니다. `function_init.cuh`의 `read_solv_input()`이 이 파일을 token 단위로 읽어서 `vii`, `vif` 배열에 채웁니다.

현재 샘플의 핵심 설정:

```text
solver_type: ISPH
dimension: 3
property_table: NO
kernel type: Wendland6
filter type: Shepard
density calculation: Continuity
time integration: Predictor_Corrector
fluid type: Liquid
simulation type: Single_Phase
pressure-force solve: YES
viscous-force solve: YES
turbulence-model: Laminar
gravitational-force solve: YES
XSPH solve: YES
kernel-gradient-correction: KGC
delta-SPH solve: Antuono
open boundary: OPEN
time-step: 1.0e-7
start-time: 0.0
end-time: 5.0
plot-output frequency: 100000
plot-variables: 3 rho pres p_type
decouple-stride: 1
```

## `vii`와 `vif`로 들어가는 구조

`function_init.cuh`와 `Parameters.cuh`는 solver option을 두 배열에 담습니다.

```text
vii: int option/state array
vif: Real option/state array
```

예:

```text
solver_type     → vii[0]
dim             → vii[1]
open_boundary   → vii[8]
num_part_sph    → vii[53]
num_part_dem    → vii[55]
decouple_stride → vii[57]

dt              → vif[3]
time            → vif[4]
time_end        → vif[5]
soundspeed      → vif[25]
rho0_eos        → vif[26]
dcell           → vif[29]
```

## `data.txt`

`data.txt`는 `read_table()`이 읽는 property table입니다.

현재 형식:

```text
#p_type 0
//T
10, 100, 200, 300, 400
//h
0, 1000, 2000, 3000, 4000
//k
1, 1, 1, 1, 1
//mu
1e-3, 1e-3, 1e-3, 1e-3, 1e-3
#end
```

의미:

- `#p_type N`: particle type별 table 시작
- `//T`: temperature grid
- `//h`: enthalpy table
- `//k`: conductivity table
- `//mu`: viscosity table

코드는 이 값을 `host_Tab_T`, `host_Tab_h`, `host_Tab_k`, `host_Tab_vis`에 저장하고 GPU constant memory로 복사합니다.

## `input.txt`

`input.txt`는 실제 입자 데이터를 담는 핵심 파일입니다. 현재 복사본에는 없지만 코드에서 요구합니다.

`function_init.cuh`의 `read_input()`은 첫 줄을 variable label list로 읽고, 이후 값을 순서대로 읽습니다.

현재 코드에 정의된 label 일부:

| label | 의미 | part1 field |
|---:|---|---|
| 1 | x | `x` |
| 2 | y | `y` |
| 3 | z | `z` |
| 4 | ux | `ux` |
| 5 | uy | `uy` |
| 6 | uz | `uz` |
| 7 | mass | `m` |
| 8 | particle type | `p_type` |
| 9 | smoothing length | `h` |
| 10 | temperature | `temp` |
| 11 | pressure | `pres` |
| 12 | density | `rho` |
| 13 | reference density | `rho_ref` 쪽 초기값으로 연결될 가능성 |
| 27 | DEM radius | `rad` |
| 28 | DEM rotational inertia | `ri` |
| 29 | DEM index | `dem_idx` |
| 30-32 | angular velocity | `wx`, `wy`, `wz` |
| 36 | volumetric power | `vol_power` |
| 37 | open boundary buffer type | `buffer_type` |

실제 `input.txt`는 첫 줄에 사용할 column label을 탭으로 나열하고, 이후 각 particle에 대해 해당 변수 값들을 순서대로 기록하는 방식입니다.

개념 예시:

```text
1	2	3	4	5	6	7	8	9	10	11	12	27	28	29
x1
y1
z1
ux1
uy1
uz1
m1
p_type1
h1
T1
p1
rho1
rad1
ri1
dem_idx1
...
```

주의: 실제 줄/열 배치는 `read_input()`이 `fscanf`로 읽는 방식에 맞춰야 하므로, 초기 Code Manual과 현재 parser를 함께 확인해야 합니다.

## `p_type.txt`

현재 `p_type.txt`는 122,574줄이며 대부분 숫자 `0`으로 시작합니다. 파일명상 particle type field로 보이지만 현재 `SOPHIA_gpu.cu`의 실행 경로에서 직접 읽는 코드는 확인되지 않았습니다.

가능한 역할:

1. `input.txt` 생성을 위한 보조 particle type list
2. 과거 GUI/전처리기의 출력
3. 특정 case의 p_type field만 분리 저장한 데이터

향후 SOPHIA 에이전트가 실제 case를 만들 때는 `p_type.txt`를 무조건 실행 입력으로 가정하지 말고, `function_init.cuh`의 parser와 `input.txt` 형식을 기준으로 해야 합니다.

## 시뮬레이션 case 생성 시 에이전트 체크리스트

1. 원하는 physics 선택
   - ISPH인지, DEM 포함인지, SPH-DEM coupling인지
2. domain 크기와 현재 하드코딩 범위 비교
   - `ISPH.cuh`의 `x_min/x_max/...` 수정 필요 여부
3. 입자 배치 생성
   - fluid SPH particles
   - boundary particles
   - DEM solid particles
   - open boundary dummy/inlet/outlet particles
4. `input.txt` column label 결정
5. 각 particle의 `p_type`, `i_type`, `buffer_type`, `h`, `m`, `rho`, `rad` 등 설정
6. `solv.txt`의 model option 조정
7. `data.txt` 물성 table 필요 여부 확인
8. output 함수가 원하는 field를 저장하는지 확인

## 관련 노트

- [[SOPHIA Simulation Execution Flow]]
- [[SOPHIA Particle Data Structures]]
- [[SOPHIA Current Code vs Manuals]]
