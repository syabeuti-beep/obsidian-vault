---
type: code-knowledge-note
topic: sophia-source-documents
aliases:
  - SOPHIA 자료 목록
tags:
  - sophia
  - source
  - manual
  - pdf
  - code
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA Source Documents

## 기준 폴더

- Mac 복사본: `/Users/hojin/Documents/SOPHIA에이전트`
- 최신 코드: `/Users/hojin/Documents/SOPHIA에이전트/최신 SOPHIA코드`

## 현재 코드 파일

현재 정답으로 취급해야 하는 파일들입니다.

- `SOPHIA_gpu.cu`: entry point, include graph, 전역 변수, `main()`
- `ISPH.cuh`: `ISPH(vii,vif)` 상위 실행 함수, GPU 정보 출력, 입력 입자 로딩, SPH/DEM 분리, pthread 실행
- `ISPH_Calc_seperate.cuh`: 실제 time-step 계산 루프와 CUDA kernel launch 순서
- `function_init.cuh`: `solv.txt`, `data.txt`, `input.txt` parser와 초기화
- `class_Cuda_Particle_Array2.cuh`: `part1`, `part2`, `part3`, `p2p_part3` 구조체
- `Parameters.cuh`: solver type, particle type, kernel, boundary, turbulence, table size 등 상수
- `function_NNPS.cuh`: nearest neighboring particle search, cell indexing, reorder
- `function_PREP.cuh`: mass/volume/filter/gradient correction/preparation
- `function_PPE.cuh`: pressure Poisson equation, pressure force, advection force
- `function_DEM_INTERACTION.cuh`: DEM-DEM contact force
- `function_SPH_DEM_COUPLING.cuh`: SPH-DEM coupling force
- `function_DEM_BC.cuh`: DEM wall boundary condition
- `function_BC.cuh`: SPH/open boundary pressure/extrapolation 관련 kernel
- `function_TIME_ISPH.cuh`: predictor/corrector 및 time update
- `function_XSPH.cuh`: XSPH velocity correction
- `function_OUTPUT.cuh`: VTK/restart/output 저장
- `function_INPUT_GEN.cuh`: DEM input 재생성 저장
- `function_PROP.cuh`: property table interpolation 및 물성 계산
- `function_KNL.cuh`: SPH kernel 및 gradient correction 수식

## 입력/출력 파일

- `input/solv.txt`: solver option과 수치 모델 on/off 설정
- `input/data.txt`: property table. `#p_type`, `//T`, `//h`, `//k`, `//mu` 형식
- `input/p_type.txt`: 대량의 particle type 값. 현재 복사본에서는 122,574줄
- `plotdata/demDEM_decouple1_dt1E-07_0stp.vtk`: 예시/출력 VTK로 보이는 파일

현재 코드의 `ISPH.cuh`는 `./input/input.txt`를 읽도록 되어 있습니다. 하지만 복사된 폴더에는 `input.txt`가 없습니다. 따라서 실제 시뮬레이션을 돌릴 때는 [[SOPHIA Input Files]]의 규칙에 따라 `input/input.txt`를 생성해야 합니다.

## PDF 문서

PDF는 모두 읽어 텍스트를 추출했습니다. 단, 현재 코드와 다를 수 있으므로 “이론 및 초기 설계 참고용”입니다.

- `SOPHIA 초기 Code Manual.pdf`
  - 설치/컴파일/실행, `input.txt`, `solv.txt`, `data.txt`, 출력, 테스트 케이스 설명
- `SOPHIA 초기 Theory Guide.pdf`
  - SPH kernel, particle approximation, correction, governing equations, boundary condition, time integration, GPU parallelization
- `SOPHIA에 DEM결합 추가 논문.pdf`
  - GPU 기반 SPH-DEM coupled code, incompressible 3-phase flow, debris bed/self-leveling, two-way coupling, GPU parallelization
- `1. DEM정리.pdf`
  - DEM contact, Hertz-Mindlin, wall collision, sliding/rolling, DEM algorithm
- `2. ESPH정리.pdf`
  - Eulerian incompressible SPH, Eulerian/Lagrangian Navier-Stokes/continuity 비교
- `3. ISPH정리.pdf`
  - PPE, pressure force, advection force, predictor/corrector velocity update
- `4. OpenBC정리.pdf`
  - Dirichlet/Neumann/open boundary, dummy particle 개념
- `5. SPH-DEM Coupling정리.pdf`
  - AVNS, porosity, pressure gradient force, drag, reaction force
- `Eulerian SPH.pdf`
  - ALE/Eulerian SPH, convective term, mass/momentum conservation, sorting 비용 감소

## 문서와 코드의 우선순위

1. 현재 코드의 실제 실행 경로
2. 현재 코드의 주석
3. 현재 `solv.txt`, `data.txt`, `p_type.txt` 샘플
4. PDF의 이론 설명
5. 초기 Code Manual의 파일 형식 설명

[[SOPHIA Current Code vs Manuals]]에서 차이를 계속 추적해야 합니다.
