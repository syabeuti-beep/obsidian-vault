---
type: code-knowledge-note
topic: sophia-solid-fluid-heat-transfer
aliases:
  - SOPHIA 고체 유체 열전달
  - SOPHIA Solid Fluid Heat Transfer
  - SOPHIA SPH DEM Heat Transfer
tags:
  - sophia
  - heat-transfer
  - sph-dem
  - equations
  - paper-implementation
created: 2026-06-25
status: draft
source_priority: current-code-first
---

# SOPHIA Solid-Fluid Heat Transfer

## 한 줄 요약

현재 [[SOPHIA]]의 고체-유체 열전달은 `function_SPH_DEM_COUPLING.cuh`의 `KERNEL_DEM_coupling3D_dem()`와 `KERNEL_SPH_coupling3D_sph()`에서 계산됩니다. DEM 입자 주변 SPH 유체 온도/속도를 kernel interpolation으로 보간한 뒤, Nusselt correlation으로 convective heat transfer source를 만들고, 그 반작용을 SPH 온도 방정식에 분배합니다.

논문 구현 시 이 노트는 “논문 고체-유체 열전달식이 현재 코드와 같은가?”를 비교하는 기준입니다.

## 코드 기준 파일과 함수

```text
function_SPH_DEM_COUPLING.cuh
  - KERNEL_DEM_coupling3D_dem
  - KERNEL_SPH_coupling3D_sph
function_PREP.cuh
  - KERNEL_clc_prep3D_coupling_sph
  - KERNEL_clc_prep3D_coupling_dem
function_TIME_ISPH.cuh
  - KERNEL_time_update_projection_dem
  - KERNEL_time_update_projection_sph
function_PROP.cuh
  - conductivity, conductivity_1
  - diffusivity_1
  - heat_capacity
function_OUTPUT.cuh
  - save_plot_fluid_vtk_bin_solid_dem
```

## Data flow

```text
SPH/DEM positions and temperatures
→ coupling prep에서 porosity/filter 계산
→ DEM coupling kernel에서 DEM 입자 주변 SPH 유체 속도/온도 보간
→ Re, Nu, h 계산
→ DEM dT/dt와 Q_sd 계산
→ SPH coupling kernel에서 Q_sd 반작용을 SPH dT/dt에 분배
→ time update에서 T^{n+1} = T^n + Δt dT/dt
→ VTK로 temperature_surface, Q_sd, dtemp_dt 출력
```

## Coupling prep: filter와 porosity

### SPH 위치의 porosity

`KERNEL_clc_prep3D_coupling_sph()`는 SPH 입자 `i` 주변 DEM 입자의 체적을 kernel로 합산합니다.

코드:

```cpp
tmp_por += tmp_wij * 4.18879 * radj * radj * radj;
P1_sph[i].DEMpor = 1.0 - tmp_por / (tmp_SPHflt + 1e-20);
```

수식:

$$
\epsilon_i^{SPH}
= 1 - \frac{\sum_{j\in DEM} V_j W_{ij}}{\sum_{j\in SPH} \frac{m_j}{\rho_j}W_{ij}}
$$

여기서 $V_j=\frac{4}{3}\pi r_j^3$입니다.

### DEM 위치의 fluid/air filter

`KERNEL_clc_prep3D_coupling_dem()`는 DEM 입자 `i` 주변 SPH 입자를 type별로 나누어 filter를 계산합니다.

코드:

```cpp
if(ptypej == 1) tmp_DEMfltd += mj/rhoj * W_ij;
if(ptypej == 3) tmp_DEMflt  += mj/rhoj * W_ij;
P1_dem[i].flt_s = tmp_DEMflt;
P1_dem[i].flt_sd = tmp_DEMfltd;
```

수식:

$$
F_{fluid,i}=\sum_{j\in SPH,ptype=1}\frac{m_j}{\rho_j}W_{ij}
$$

$$
F_{air,i}=\sum_{j\in SPH,ptype=3}\frac{m_j}{\rho_j}W_{ij}
$$

이 값들은 뒤에서 DEM 입자 주변 평균 유체 속도/온도를 정규화하는 denominator로 쓰입니다.

## DEM 입자 주변 유체 온도/속도 보간

`KERNEL_DEM_coupling3D_dem()`에서 DEM 입자 `i`를 기준으로 이웃 SPH 입자 `j`를 훑습니다.

코드:

```cpp
tmp_ufxf += uxj * mj * W_ij / rhoj;
tmp_ufyf += uyj * mj * W_ij / rhoj;
tmp_ufzf += uzj * mj * W_ij / rhoj;
tmp_temp_f += tempj * mj * W_ij / rhoj;

ufxf = tmp_ufxf / (flt_fluid + 1e-15);
temp_f = tmp_temp_f / (flt_fluid + 1e-15);
```

수식:

$$
\mathbf{u}_{f,i}
= \frac{\sum_{j\in fluid} \mathbf{u}_j \frac{m_j}{\rho_j}W_{ij}}{F_{fluid,i}}
$$

$$
T_{f,i}
= \frac{\sum_{j\in fluid} T_j \frac{m_j}{\rho_j}W_{ij}}{F_{fluid,i}}
$$

상대속도와 온도차:

$$
\mathbf{u}_{rel,i}=\mathbf{u}_{f,i}-\mathbf{u}_{s,i}
$$

$$
\Delta T_i = T_{f,i}-T_{s,i}
$$

코드:

```cpp
uxijf = ufxf - uxi;
temp_ijf = temp_f - tempi;
mag_uijf = sqrt(uxijf*uxijf + uyijf*uyijf + uzijf*uzijf);
```

## 현재 코드의 고체-유체 열전달식

현재 fluid-dominant 조건에서는 다음 코드가 실행됩니다.

```cpp
Ref = di * mag_uijf * pori / (vis_kif + 1e-15);
Pr_f = 0.7;
Nu_f = 2.0 + 0.6 * pow(Ref, 0.5) * pow(Pr_f, 1 / 3);
h_conv_f = Nu_f * 0.0518 / di;
a_ht = 6.0 / di;
dq_vol = h_conv_f * a_ht * temp_ijf;
P3_dem[i].dtemp = dq_vol / (rhoi * cpi);
P1_dem[i].Q_sd = dq_vol * mi / rhoi;
```

수식으로 정리하면:

$$
Re_i = \frac{d_i |\mathbf{u}_{f,i}-\mathbf{u}_{s,i}|\epsilon_i}{\nu_f}
$$

$$
Nu_i = 2 + 0.6 Re_i^{1/2} Pr^{1/3}
$$

$$
h_i = \frac{Nu_i k_f}{d_i}
$$

현재 fluid branch에서는 $k_f$ 자리에 코드 상수 `0.0518`이 직접 들어갑니다.

$$
a_i = \frac{6}{d_i}
$$

$$
\dot q_i^{vol} = h_i a_i (T_{f,i}-T_{s,i})
$$

$$
\frac{dT_{s,i}}{dt} = \frac{\dot q_i^{vol}}{\rho_{s,i} C_{p,s,i}}
$$

$$
Q_{sd,i} = \dot q_i^{vol}\frac{m_i}{\rho_{s,i}}
$$

여기서 $Q_{sd,i}$는 DEM 입자 `i`가 받는 총 열량률[J/s]처럼 저장됩니다.

## Air/mixed branch 주의

air-dominant branch와 mixed branch도 유사하지만 현재 코드에서는 conductivity에 `ki`가 들어갑니다.

```cpp
h_conv_a = Nu_a * ki / di;
h_conv_fa = Nu_fa * ki / di;
```

여기서 `ki`는 DEM solid conductivity입니다. 논문이 유체 열전도도 $k_f$를 쓰는 식이라면 이 부분은 반드시 수정 후보입니다.

## SPH 쪽 반작용 열전달

`KERNEL_SPH_coupling3D_sph()`는 DEM의 `Q_sd`를 주변 SPH 입자에 kernel로 분배합니다.

코드:

```cpp
Q_sdj = P1_dem[j].Q_sd;
tmpt += -Q_sdj / (rhoi * cpi) * W_ij / (flt_fj + 1e-20);
P3_sph[i].dtemp = dtempi + tmpt;
```

수식:

$$
\left(\frac{dT_f}{dt}\right)_i^{coupling}
= -\sum_{j\in DEM}
\frac{Q_{sd,j}}{\rho_i C_{p,i}}
\frac{W_{ij}}{F_j + \epsilon_{num}}
$$

여기서 $F_j$는 DEM 입자 주변 SPH filter `flt_fluidj + flt_airj`입니다.

주의할 점:

- DEM이 얻는 열량 $Q_{sd}$를 SPH에서 빼는 구조입니다.
- 실제 전체 에너지 보존이 되는지는 $W_{ij}/F_j$의 차원과 SPH control volume 처리까지 확인해야 합니다.
- 논문이 Eulerian cell 기반으로 fluid-solid heat exchange를 분배한다면, 이 SPH 분배식과 다를 수 있습니다.

## Drag와 열전달의 결합

같은 kernel 안에서 drag도 함께 계산됩니다.

```cpp
Cdf = (0.63 + 4.8/sqrt(Ref))^2;
betaf = 3.7 - 0.65*exp(-(1.5-log10(Ref))^2/2);
dfxf = 1/8*Cdf*rho_f*pi*d_i^2*u_rel_x*|u_rel|*pori^(2-betaf)/m_i;
```

수식:

$$
a_{D,x} = \frac{1}{8}\frac{C_D \rho_f \pi d_i^2}{m_i}
(u_{f,x}-u_{s,x})|\mathbf{u}_f-\mathbf{u}_s|\epsilon_i^{2-\beta}
$$

논문에서 drag correlation과 heat-transfer correlation이 서로 연결되어 있으면, 둘을 따로 고치면 안 됩니다.

## Time update

`function_TIME_ISPH.cuh`는 DEM/SPH 모두 다음 형태로 온도를 갱신합니다.

```cpp
TP1[i].temp = P2[i].temp0 + P3[i].dtemp * dt;
```

수식:

$$
T_i^{n+1}=T_i^n + \Delta t\left(\frac{dT_i}{dt}\right)
$$

`P2[i].temp0`는 projection 단계에서 현재 온도로 저장됩니다.

## 논문 equation audit에서 반드시 비교할 항목

논문이 고체-유체 열전달식을 제시하면 다음 항목을 표로 비교해야 합니다.

| 항목 | 현재 SOPHIA 코드 | 논문에서 확인할 내용 |
|---|---|---|
| 유체 온도 | SPH kernel interpolation $T_f=\sum T_j m_j/\rho_j W/F$ | cell 평균인지, SPH 보간인지 |
| 상대속도 | SPH 보간 유체속도 - DEM 속도 | superficial/interstitial velocity 구분 |
| Reynolds number | $Re=d|u_{rel}|\epsilon/\nu$ | porosity factor 위치 |
| Nusselt correlation | $Nu=2+0.6Re^{1/2}Pr^{1/3}$ | 논문 correlation과 계수 |
| 열전도도 | fluid branch는 `0.0518`, air/mixed는 `ki` | $k_f$인지 $k_s$인지 |
| 면적/체적 변환 | $a=6/d$ | sphere surface area/volume인지, projected area인지 |
| DEM 온도식 | $dT_s/dt=q^{vol}/(\rho_s C_{p,s})$ | lumped capacitance인지, 내부전도 포함인지 |
| SPH 반작용 | $-Q_{sd}W/(\rho C_p F)$ | 논문의 fluid energy source와 일치하는지 |
| 부호 | $T_f-T_s$ | 논문 sign convention |

## 코드 수정 시 추천 절차

1. 논문 식을 `[[SOPHIA Paper Equation Audit Workflow]]` 형식으로 먼저 적습니다.
2. 위 표의 항목을 모두 채웁니다.
3. `KERNEL_DEM_coupling3D_dem()`에서 DEM source term을 고칩니다.
4. `KERNEL_SPH_coupling3D_sph()`에서 fluid 반작용 source term을 고칩니다.
5. `function_OUTPUT.cuh`에 `Nu`, `Re`, `h_conv`, `Q_sd`, `dtemp_dt` 같은 diagnostic field를 추가합니다.
6. 간단한 1 DEM + 균일 SPH 온도장 probe로 $Q=hA(T_f-T_s)$가 손계산과 맞는지 확인합니다.

## 현재 코드에서 의심해야 할 부분

- `pow(Pr_f, 1 / 3)`에서 `1/3`이 C/C++ integer division이면 0이 됩니다. 즉 실제로는 $Pr^0=1$일 수 있습니다. `1.0/3.0`인지 반드시 확인해야 합니다.
- fluid branch의 `h_conv_f = Nu_f * 0.0518 / di`는 `function_PROP.cuh`의 current gas conductivity와 다를 수 있습니다.
- air/mixed branch에서 `ki`가 solid conductivity라면 논문 식과 다를 가능성이 큽니다.
- `Q_sd`를 SPH로 되돌리는 분배식의 차원/보존성을 별도 검증해야 합니다.

## 관련 노트

- [[SOPHIA SPH Discrete Equations]]
- [[SOPHIA SPH-DEM Coupling]]
- [[SOPHIA ISPH Solver]]
- [[SOPHIA Particle Data Structures]]
- [[SOPHIA Paper Equation Audit Workflow]]
