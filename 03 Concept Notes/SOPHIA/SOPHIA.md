---
type: code-knowledge-note
topic: sophia
aliases:
  - SOPHIA 코드
  - SOPHIA 에이전트
  - SOPHIA_gpu
  - Smoothed Particle Hydrodynamics code In Advanced nuclear safety
tags:
  - sophia
  - sph
  - dem
  - isph
  - cuda
  - thermal-hydraulics
created: 2026-06-24
status: draft
source_priority: current-code-first
---

# SOPHIA

[[SOPHIA]]는 열유체/원자력 안전 해석을 위한 GPU 기반 SPH 코드입니다. 현재 사용자님이 복사한 폴더의 최신 코드 기준으로는 단순 초기 SPH 코드가 아니라, [[SOPHIA ISPH Solver]], [[SOPHIA DEM Model]], [[SOPHIA SPH-DEM Coupling]], [[SOPHIA Open Boundary Condition]], CUDA 병렬화가 결합된 코드로 이해해야 합니다.

## 현재 기준 경로

```text
/Users/hojin/Documents/SOPHIA에이전트
/Users/hojin/Documents/SOPHIA에이전트/최신 SOPHIA코드
```

## 이 노트를 읽는 순서

1. [[SOPHIA Code Knowledge Graph]]
2. [[SOPHIA Source Documents]]
3. [[SOPHIA Current Code vs Manuals]]
4. [[SOPHIA Code Architecture]]
5. [[SOPHIA Simulation Execution Flow]]
6. [[SOPHIA Input Files]]
7. [[SOPHIA Particle Data Structures]]
8. [[SOPHIA SPH Discrete Equations]]
9. [[SOPHIA ISPH Solver]]
10. [[SOPHIA SPH-DEM Coupling]]
11. [[SOPHIA Solid-Fluid Heat Transfer]]
12. [[SOPHIA Paper Equation Audit Workflow]]
13. [[SOPHIA DEM Model]]
14. [[SOPHIA Open Boundary Condition]]
15. [[SOPHIA CUDA GPU Parallelization]]

## 핵심 해석 원칙

현재 코드를 정답으로 보고, PDF 매뉴얼과 이론 자료는 참고로 봅니다. 특히 초기 Code Manual은 파일 형식과 기본 구조를 이해하는 데 유용하지만, 현재 코드의 SPH/DEM 분리 구조, `decouple_stride`, DEM 출력 중심 설정, domain hard-coding 같은 최신 변경점을 대체하지 못합니다.

논문 구현을 할 때는 `input.txt`나 geometry만 맞추는 것이 아니라, 논문 식과 현재 SOPHIA 식이 같은지도 확인해야 합니다. 밀도, 압력, 점성, 열전도, 고체-유체 열전달, drag, porosity source는 [[SOPHIA SPH Discrete Equations]]와 [[SOPHIA Paper Equation Audit Workflow]]를 기준으로 수식 수준에서 비교합니다.

## 최종 목표

이 지식그래프의 목적은 SOPHIA를 이해하는 에이전트가 다음을 할 수 있게 만드는 것입니다.

- 사용자가 원하는 유체/입자 시뮬레이션 조건을 이해하기
- `solv.txt`, `data.txt`, `input.txt` 생성/수정하기
- 현재 코드에서 필요한 수정 지점 찾기
- CUDA/GPU 실행 환경에서 빌드/실행하기
- VTK 및 generated input 결과 해석하기

## 관련 노트

- [[SOPHIA Code Knowledge Graph]]
- [[SOPHIA SPH Discrete Equations]]
- [[SOPHIA Solid-Fluid Heat Transfer]]
- [[SOPHIA Paper Equation Audit Workflow]]
- [[SOPHIA Input Files]]
- [[SOPHIA Simulation Execution Flow]]
- [[SOPHIA Current Code vs Manuals]]
