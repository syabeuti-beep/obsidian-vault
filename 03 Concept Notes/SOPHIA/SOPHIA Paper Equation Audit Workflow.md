---
type: workflow-note
topic: sophia-paper-equation-audit-workflow
aliases:
  - SOPHIA 논문 식 검증 workflow
  - SOPHIA Equation Audit
tags:
  - sophia
  - paper-implementation
  - equation-audit
  - code-modification
created: 2026-06-25
status: draft
source_priority: current-code-first
---

# SOPHIA Paper Equation Audit Workflow

## 한 줄 요약

논문을 [[SOPHIA]]에 구현할 때는 input geometry를 만드는 것보다 먼저, 논문의 지배방정식/closure correlation이 현재 SOPHIA 코드의 실제 계산식과 일치하는지 확인해야 합니다. 일치하지 않으면 `source_modified/` 안의 코드를 논문 식에 맞게 수정하고, 차이와 수정 이유를 문서화해야 합니다.

## 왜 필요한가

SOPHIA에는 이미 SPH, DEM, SPH-DEM coupling, 열전달 코드가 들어 있습니다. 하지만 “비슷한 물리량을 계산한다”는 것과 “논문 식과 같은 식을 계산한다”는 것은 다릅니다.

예를 들어 고체-유체 열전달은 다음 항목 중 하나만 달라도 결과가 바뀝니다.

- Nusselt correlation
- Reynolds number 정의
- porosity factor 위치
- 유체/고체 열전도도 선택
- heat-transfer area 또는 area per volume
- 온도차 부호
- $Q$를 DEM/SPH에 나누는 방식
- $C_p$, $\rho$, mass/volume으로 나누는 위치

따라서 앞으로 논문 구현은 반드시 equation audit을 거쳐야 합니다.

## 기본 산출물

논문 기반 case workspace에는 다음 파일을 만들어야 합니다.

```text
equation_audit.md
knowledge_grounding.md
experiment_spec.yaml
patch_summary.yaml
validation/verification_report.md
```

Obsidian 쪽에는 장기 지식으로 다음 노트와 연결합니다.

```text
[[SOPHIA SPH Discrete Equations]]
[[SOPHIA Solid-Fluid Heat Transfer]]
[[SOPHIA SPH-DEM Coupling]]
[[SOPHIA ISPH Solver]]
[[SOPHIA Particle Data Structures]]
```

## Step 1. 논문 식 추출

논문에서 다음을 모두 추출합니다.

| 범주 | 예시 |
|---|---|
| 지배방정식 | mass, momentum, energy equation |
| closure | drag, Nusselt, conductivity, porosity, EOS |
| source term | fluid-solid heat exchange, DEM contact heat, body force |
| boundary/initial condition | inlet velocity, temperature, wall, periodic/open |
| 수치 모델 | SPH interpolation, grid equation, DEM coupling, coarse graining |

각 식은 다음 형식으로 기록합니다.

```md
### Paper Eq. (n): 이름

원문 식:
$$ ... $$

기호:
- $T_s$: solid temperature
- $T_f$: fluid temperature
- $h$: heat transfer coefficient

물리 의미:
- DEM 입자와 주변 유체 사이의 convective heat exchange

필요 코드 위치 후보:
- function_SPH_DEM_COUPLING.cuh / KERNEL_DEM_coupling3D_dem
- function_SPH_DEM_COUPLING.cuh / KERNEL_SPH_coupling3D_sph
```

## Step 2. 현재 SOPHIA 코드 식 역추적

현재 코드가 실제로 계산하는 식을 수식으로 다시 씁니다. 이때 매뉴얼이 아니라 current source를 기준으로 합니다.

예시 형식:

```md
### Current SOPHIA: solid-fluid heat transfer

코드 위치:
- function_SPH_DEM_COUPLING.cuh: KERNEL_DEM_coupling3D_dem

코드 식:
$$
T_{f,i}=\frac{\sum_j T_j m_j/\rho_j W_{ij}}{\sum_j m_j/\rho_j W_{ij}}
$$

$$
Nu_i = 2 + 0.6 Re_i^{1/2}Pr^{1/3}
$$

$$
\frac{dT_{s,i}}{dt}=\frac{h_i(6/d_i)(T_{f,i}-T_{s,i})}{\rho_s C_{p,s}}
$$
```

## Step 3. 논문 식 vs 코드 식 비교표

반드시 표로 차이를 기록합니다.

| 물리 항 | 논문 식 | 현재 SOPHIA 식 | 일치 여부 | 수정 계획 |
|---|---|---|---|---|
| 유체 온도 | cell average | SPH kernel interpolation | 다름 | 논문이 요구하면 cell/grid average 구현 또는 SPH 근사 정당화 |
| Nu | 논문 correlation | $2+0.6Re^{1/2}Pr^{1/3}$ | 확인 필요 | 계수와 지수 수정 |
| $k_f$ | gas conductivity | 코드 상수 또는 solid `ki` | 다름 가능 | `conductivity(temp, ptype)` 사용 |
| heat source | $hA(T_f-T_s)$ | $ha_v(T_f-T_s)$ | 확인 필요 | area/volume convention 맞추기 |

## Step 4. 코드 수정 범위 결정

식이 다르면 다음 중 하나로 분류합니다.

1. 단순 parameter 차이
   - 계수, 물성, 상수만 변경
2. closure correlation 차이
   - `Nu`, `Cd`, `Re`, `Pr` 계산 함수화 필요
3. source term 구조 차이
   - DEM source와 SPH 반작용 source를 함께 수정해야 함
4. 수치 방법 차이
   - 논문이 Eulerian grid equation인데 SOPHIA는 SPH interpolation이면 근사 구현인지 새 solver가 필요한지 결정
5. 데이터 구조/output 부족
   - 새 diagnostic field 추가 필요

## Step 5. SPH 이산화 형태로 구현

논문 식이 continuum form이면 SOPHIA 코드에 들어갈 discrete form을 먼저 씁니다.

예: scalar 보간

$$
\phi_i = \frac{\sum_j \phi_j \frac{m_j}{\rho_j}W_{ij}}{\sum_j \frac{m_j}{\rho_j}W_{ij}}
$$

예: gradient

$$
\nabla \phi_i = \sum_j \frac{m_j}{\rho_j}(\phi_j-\phi_i)\nabla_i^c W_{ij}
$$

예: diffusion

$$
\frac{dT_i}{dt} = \frac{1}{\rho_i C_{p,i}}\sum_j 2\frac{m_j}{\rho_j}k_{ij}^{H}(T_i-T_j)\frac{\mathbf{r}_{ij}\cdot\nabla_i^cW_{ij}}{r_{ij}^2+\eta^2}
$$

예: DEM-fluid convective source

$$
Q_{sf,i}=h_iA_i(T_{f,i}-T_{s,i})
$$

$$
\frac{dT_{s,i}}{dt}=\frac{Q_{sf,i}}{m_{s,i}C_{p,s,i}}
$$

SPH fluid 반작용은 보존성을 확인한 뒤 다음 꼴 중 하나로 정합니다.

$$
\left(\frac{dT_f}{dt}\right)_j
\mathrel{+}= -\frac{Q_{sf,i}}{\rho_jC_{p,j}}\frac{W_{ij}}{\sum_k V_kW_{ik}}
$$

또는 case에 맞는 control-volume source term으로 바꿉니다.

## Step 6. Diagnostic output 추가

논문 식을 구현했는지 확인하려면 결과 온도만 보면 부족합니다. 가능한 경우 다음을 VTK 또는 text report로 출력합니다.

- `Re`
- `Pr`
- `Nu`
- `h_conv`
- `Q_sd`
- `dtemp_dt`
- `T_fluid_interp`
- `solid_volume_fraction` 또는 `porosity`
- `k_eff`

## Step 7. 최소 probe 검증

복잡한 full case 전에 작은 계산을 만듭니다.

예: DEM 1개 + 균일 SPH 온도장

```text
T_s = 300 K
T_f = 400 K
u_f, k_f, Pr, d known
relative velocity known
```

손계산:

$$
Re, Nu, h, Q, dT_s/dt
$$

코드 출력:

```text
Q_sd
P3_dem[i].dtemp
P3_sph[j].dtemp
```

두 값이 일치해야 full case로 넘어갑니다.

## Step 8. 문서화와 GitHub

수정 후 다음 파일에 반영합니다.

```text
equation_audit.md
patch_summary.yaml
validation/verification_report.md
README.md
```

GitHub repo는 사용자의 선호에 따라 public으로 만들고, input 형상 PNG도 README에서 보이게 track합니다.

## 논문 구현 전 체크리스트

- [ ] 논문 식과 equation number/page를 추출했다.
- [ ] 현재 SOPHIA 코드의 대응 함수와 실제 식을 적었다.
- [ ] 논문 식과 코드 식의 차이를 표로 정리했다.
- [ ] continuum 식을 SOPHIA의 SPH/DEM discrete form으로 바꿨다.
- [ ] DEM source와 SPH 반작용 source를 함께 검토했다.
- [ ] diagnostic output을 추가했다.
- [ ] 작은 probe 또는 정적 검증으로 손계산과 비교했다.
- [ ] full input 생성, input plot, VTK interval, README, GitHub push를 완료했다.

## 관련 노트

- [[SOPHIA]]
- [[SOPHIA Code Knowledge Graph]]
- [[SOPHIA SPH Discrete Equations]]
- [[SOPHIA Solid-Fluid Heat Transfer]]
- [[SOPHIA SPH-DEM Coupling]]
- [[SOPHIA ISPH Solver]]
