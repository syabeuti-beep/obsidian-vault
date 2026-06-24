---
type: code-knowledge-note
topic: sophia-current-code-vs-manuals
aliases:
  - SOPHIA 코드와 매뉴얼 차이
tags:
  - sophia
  - manual
  - code-truth
  - documentation
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA Current Code vs Manuals

## 한 줄 요약

[[SOPHIA]] 지식그래프에서는 현재 코드가 정답입니다. PDF 문서는 이론, 입력 형식, 초기 구조 이해에는 유용하지만, 코드 수정/실행/입력파일 생성에서는 현재 `최신 SOPHIA코드`를 우선해야 합니다.

## 왜 이 노트가 필요한가

사용자님이 명시하신 조건:

> SOPHIA는 초기버전에서 많이 변화해서 코드와 맞지 않는 부분이 있을 것이다. 현재 코드가 가장 최신의 정답이니까 매뉴얼은 참고용으로 읽어라.

따라서 에이전트는 PDF에서 본 내용을 그대로 실행 절차로 쓰면 안 됩니다.

## 현재 확인된 차이/주의점

### 1. `input.txt` 부재

매뉴얼은 `input.txt`를 설명하고, 현재 코드도 `./input/input.txt`를 읽습니다.

하지만 현재 복사된 폴더에는 다음만 있습니다.

```text
input/solv.txt
input/data.txt
input/p_type.txt
```

따라서 실제 실행용 case를 만들려면 `input/input.txt`를 새로 생성해야 합니다.

### 2. 현재 코드는 SPH/DEM 분리 구조

초기 매뉴얼은 SOPHIA의 일반 SPH framework 설명에 가깝습니다. 현재 코드는 다음 배열을 별도 관리합니다.

```text
HP1_sph, HP1_dem
dev_P1_sph, dev_P1_dem
dev_SP1_sph, dev_SP1_dem
```

이는 [[SOPHIA SPH-DEM Coupling]]이 통합된 최신 구조입니다.

### 3. 현재 실행 경로는 ISPH 중심

`main()`은 WCSPH/ISPH/DFSPH switch를 갖고 있지만 현재 `solv.txt`는 `ISPH`입니다.

```text
solver_type: ISPH
```

따라서 당장은 [[SOPHIA ISPH Solver]]를 중심으로 이해해야 합니다.

### 4. output 함수가 DEM 중심으로 활성화

현재 `ISPH_Calc_seperate.cuh`에서 활성화된 출력은 다음입니다.

```cpp
save_plot_fluid_vtk_bin_solid_dem(file_P1_dem,file_P3_dem);
```

fluid output 함수들은 상당수 주석 처리되어 있습니다. 사용자가 유체장 출력을 원하면 `function_OUTPUT.cuh`와 이 호출부를 수정해야 할 수 있습니다.

### 5. domain 범위 하드코딩

`ISPH.cuh`에서 `find_minmax()` 뒤에 domain 범위가 하드코딩되어 있습니다.

```cpp
x_max = 0.09;
x_min = -0.01;
y_max = 0.02;
y_min = -0.01;
z_max = 0.251;
z_min = -0.01;
```

이는 새로운 geometry를 만들 때 매우 중요한 제약입니다.

### 6. CUB include 경로

`Makefile`은 다음 include를 가정합니다.

```text
-I./cub-1.8.0/
```

현재 복사된 파일 목록에는 `cub-1.8.0` 폴더가 보이지 않습니다. 이미 build된 `SOPHIA_gpu` binary는 있지만, 재컴파일에는 CUB dependency 확인이 필요합니다.

## 에이전트 운영 원칙

1. 어떤 기능을 설명할 때는 먼저 현재 코드의 함수/파일을 확인한다.
2. PDF는 이론적 배경, 변수 의미, 물리 모델을 보강하는 용도로 사용한다.
3. 입력파일 생성은 `function_init.cuh` parser 기준으로 한다.
4. 실행 가능성 판단은 `Makefile`, CUDA 환경, missing dependency를 확인한 뒤 한다.
5. 유체 시뮬레이션 목표가 주어지면 먼저 현재 코드가 해당 출력/물리를 활성화하고 있는지 확인한다.

## 관련 노트

- [[SOPHIA Source Documents]]
- [[SOPHIA Input Files]]
- [[SOPHIA Code Architecture]]
