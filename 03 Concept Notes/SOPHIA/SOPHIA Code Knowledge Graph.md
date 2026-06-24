---
type: code-knowledge-note
topic: sophia-code-knowledge-graph
aliases:
  - SOPHIA
  - SOPHIA 에이전트
  - SOPHIA 코드
tags:
  - sophia
  - sph
  - dem
  - isph
  - cuda
  - knowledge-graph
  - thermal-hydraulics
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA Code Knowledge Graph

## 핵심 원칙

이 지식그래프는 `/Users/hojin/Documents/SOPHIA에이전트`를 기준으로 만든 [[SOPHIA]] 코드 이해용 그래프입니다.

가장 중요한 해석 원칙은 다음입니다.

> 현재 폴더의 `최신 SOPHIA코드`가 정답이고, PDF 매뉴얼과 이론 문서는 참고 자료입니다.

이유는 사용자님이 알려주신 대로 [[SOPHIA]]는 초기 버전 이후 많이 바뀌었고, 특히 현재 코드는 [[SOPHIA SPH-DEM Coupling]], [[SOPHIA DEM Model]], [[SOPHIA ISPH Solver]], [[SOPHIA Open Boundary Condition]] 등이 합쳐진 최신 형태이기 때문입니다.

## 이 그래프를 왜 만들었는가

사용자님의 최종 목표는 단순히 코드를 읽는 것이 아니라, SOPHIA를 이해하는 에이전트를 구성해서 다음 작업을 시키는 것입니다.

1. 원하는 유체/입자 시뮬레이션 조건을 설명한다.
2. 에이전트가 [[SOPHIA Input Files]]를 생성하거나 수정한다.
3. 필요하면 [[SOPHIA Code Architecture]]를 수정한다.
4. SOPHIA를 컴파일/실행한다.
5. 출력 VTK나 restart/input_generated 결과를 해석한다.

따라서 이 지식그래프는 “이론 정리”보다 “코드 수정과 입력파일 생성을 위한 작동 지도”에 초점을 둡니다.

## 중심 노트

- [[SOPHIA Code Architecture]]: 현재 코드 파일들이 어떤 역할을 하는지
- [[SOPHIA Simulation Execution Flow]]: `main → read_solv_input → read_input → ISPH_Calc` 실행 흐름
- [[SOPHIA Input Files]]: `solv.txt`, `data.txt`, `input.txt`, `p_type.txt`의 의미
- [[SOPHIA Particle Data Structures]]: `part1`, `part2`, `part3` 구조체 필드 해석
- [[SOPHIA ISPH Solver]]: 현재 코드의 핵심 solver 루프
- [[SOPHIA SPH-DEM Coupling]]: SPH 입자와 DEM 입자 사이의 two-way coupling
- [[SOPHIA DEM Model]]: DEM contact, wall collision, sliding/rolling 모델
- [[SOPHIA Open Boundary Condition]]: open boundary 처리 흐름
- [[SOPHIA CUDA GPU Parallelization]]: CUDA, CUB sorting, multi-GPU/pthread 구조
- [[SOPHIA Current Code vs Manuals]]: 현재 코드와 초기 문서의 관계 및 주의점
- [[SOPHIA Source Documents]]: 읽은 PDF와 코드 파일 출처 목록

## 빠른 전체 그림

```text
SOPHIA_gpu.cu
→ read_solv_input("./input/solv.txt")
→ read_table("./input/data.txt")
→ solver_type == ISPH 이면 ISPH(vii, vif)
→ input/input.txt에서 입자 정보 읽기
→ SPH/DEM 입자 분리
→ GPU 메모리 할당
→ 매 time step마다 ISPH_Calc/SOPHIA_single_ISPH 실행
→ NNPS 정렬
→ SPH 준비/보정
→ DEM contact 계산
→ SPH-DEM coupling 계산
→ predictor/corrector + PPE pressure solve
→ time update
→ OpenBC/XSPH/DEM wall BC
→ VTK/input_generated 출력
```

## 에이전트가 특히 조심해야 할 점

- `SOPHIA 초기 Code Manual.pdf`는 초기 버전 기준입니다. 현재 코드는 `SOPHIA_gpu.cu`, `ISPH.cuh`, `ISPH_Calc_seperate.cuh`를 기준으로 해석해야 합니다.
- 현재 `main()`은 `WCSPH`, `DFSPH` 분기도 남아 있지만 실제 실행 경로는 `ISPH(vii,vif)` 중심입니다.
- 현재 `ISPH.cuh`는 `./input/input.txt`를 읽지만, 복사된 폴더에는 `input.txt`가 없고 `p_type.txt`, `solv.txt`, `data.txt`만 있습니다. 따라서 실제 실행용 case 생성 시 `input/input.txt` 생성이 핵심입니다.
- `p_type.txt`는 122,574줄의 입자 type 리스트처럼 보입니다. 직접 실행 입력인지, 전처리/보조 데이터인지 추가 확인이 필요합니다.
- `function_INPUT_GEN.cuh`는 DEM 상태를 `input_generated` 형태로 저장하는 기능을 갖습니다.

## 관련 연구/물리 키워드

- [[Smoothed Particle Hydrodynamics]]
- [[Incompressible SPH]]
- [[Eulerian SPH]]
- [[Discrete Element Method]]
- [[SPH-DEM Coupling]]
- [[Open Boundary Condition]]
- [[High Performance Computing]]
- [[CUDA]]
- [[Thermal-Hydraulics]]
