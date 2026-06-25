---
type: code-knowledge-note
topic: sophia-sph-discrete-equations
aliases:
  - SOPHIA SPH 계산식
  - SOPHIA 입자-이웃 합산식
  - SOPHIA SPH Neighbor Summations
tags:
  - sophia
  - sph
  - numerical-method
  - equations
  - cuda
created: 2026-06-25
status: draft
source_priority: current-code-first
---

# SOPHIA SPH Discrete Equations

## 한 줄 요약

[[SOPHIA]]의 SPH 계산은 CUDA kernel 하나가 자기 입자 `i`를 맡고, cell-linked list로 주변 cell의 이웃 입자 `j`를 훑으면서 kernel weight $W_{ij}$ 또는 gradient $\nabla_i W_{ij}$를 합산하는 구조입니다.

이 노트는 논문 식을 SOPHIA 코드에 구현할 때 반드시 확인해야 하는 “코드 속 실제 이산화 형태”를 정리합니다.

## 코드 기준 파일

```text
function_KNL.cuh
function_PREP.cuh
function_PPE.cuh
function_SPH_DEM_COUPLING.cuh
function_TIME_ISPH.cuh
function_XSPH.cuh
function_BC.cuh
```

현재 코드가 기준입니다. PDF/manual은 보조 설명으로만 사용해야 합니다.

## 기본 neighbor loop 구조

대부분의 kernel은 다음 흐름을 반복합니다.

```cpp
uint_t i = threadIdx.x + blockIdx.x * blockDim.x;
// 자기 입자 i의 cell index 계산
for z=-1..1:
  for y=-1..1:
    for x=-1..1:
      k = idx_cell(icell+x, jcell+y, kcell+z)
      for j = g_str[k] .. g_end[k]-1:
        r_ij = x_i - x_j
        if |r_ij| < search_range:
          W_ij, dW_ij 계산
          i에 대한 합산량 += j의 contribution
```

수식으로 쓰면 대부분 다음 꼴입니다.

$$
A_i = \sum_{j \in \mathcal{N}(i)} C_{ij} W_{ij}
$$

또는 gradient가 필요한 항은 다음 꼴입니다.

$$
\nabla A_i \approx \sum_{j \in \mathcal{N}(i)} C_{ij} \nabla_i W_{ij}
$$

여기서 $\mathcal{N}(i)$는 `search_range = k_search_kappa * h_i` 안에 들어오는 이웃 입자 집합입니다.

## Kernel function

### `calc_tmpA(h)`

`function_KNL.cuh`에서 kernel normalization constant $A(h)$를 계산합니다. `solv.txt`에서 kernel type이 `Wendland6`이면 3D 기준 다음 계수를 씁니다.

$$
A = \frac{6.788953041263660}{8h^3}
$$

코드:

```cpp
else if(k_kernel_type==Wendland6){
  if(k_dim==2) tA=3.546881588905096/(4*tH*tH);
  else if(k_dim==3) tA=6.788953041263660/(8*tH*tH*tH);
}
```

### `calc_kernel_wij(A,h,r)`

`Wendland6`의 경우 코드상 $q = r/(2h)$이며,

$$
W_{ij} = A(1-q)^8(1+8q+25q^2+32q^3), \quad q<1
$$

그 외에는 0입니다.

### `calc_kernel_dwij(A,h,r)`

`dwij`는 $dW/dr$에 해당합니다. 실제 gradient vector는 각 kernel에서 다음처럼 만듭니다.

$$
\nabla_i W_{ij} = \frac{dW}{dr}\frac{\mathbf{x}_i-\mathbf{x}_j}{r_{ij}}
$$

코드 패턴:

```cpp
tdwij = calc_kernel_dwij(tmp_A, hi, tdist);
tdwx = tdwij * (xi - xj) / tdist;
tdwy = tdwij * (yi - yj) / tdist;
tdwz = tdwij * (zi - zj) / tdist;
```

## Gradient correction

`apply_gradient_correction_3D()`는 $\nabla W$에 correction matrix $C_i$를 곱합니다.

$$
\nabla_i^c W_{ij} = C_i \nabla_i W_{ij}
$$

코드:

```cpp
*dwcx = Cm[0][0]*dwx + Cm[0][1]*dwy + Cm[0][2]*dwz;
*dwcy = Cm[1][0]*dwx + Cm[1][1]*dwy + Cm[1][2]*dwz;
*dwcz = Cm[2][0]*dwx + Cm[2][1]*dwy + Cm[2][2]*dwz;
```

`KERNEL_clc_correction_KGC_3D_sph()`는 correction matrix를 만들기 위해 이웃 입자들에 대해 대략 다음 moment matrix를 합산합니다.

$$
M_i = -\sum_j \frac{m_j}{\rho_j}\frac{dW_{ij}}{dr}\frac{\mathbf{r}_{ij}\otimes\mathbf{r}_{ij}}{r_{ij}}
$$

그리고 $C_i = M_i^{-1}$ 형태로 저장합니다.

## Kernel interpolation: 값 보간

SOPHIA에서 이웃 입자 값을 보간하는 기본 꼴은 다음입니다.

$$
\langle A \rangle_i = \frac{\sum_j A_j \frac{m_j}{\rho_j} W_{ij}}{\sum_j \frac{m_j}{\rho_j} W_{ij}}
$$

예: `KERNEL_DEM_coupling3D_dem()`에서 DEM 입자 주변 SPH 유체 속도와 온도 보간:

```cpp
tmp_ufxf += uxj * mj * twij / rhoj;
tmp_ufyf += uyj * mj * twij / rhoj;
tmp_ufzf += uzj * mj * twij / rhoj;
tmp_temp_f += tempj * mj * twij / rhoj;

ufxf = tmp_ufxf / (flt_fluid + 1.0e-15);
temp_f = tmp_temp_f / (flt_fluid + 1.0e-15);
```

수식:

$$
\mathbf{u}_{f,i}^{interp} =
\frac{\sum_{j\in SPH} \mathbf{u}_j \frac{m_j}{\rho_j} W_{ij}}
{\sum_{j\in SPH} \frac{m_j}{\rho_j} W_{ij}}
$$

$$
T_{f,i}^{interp} =
\frac{\sum_{j\in SPH} T_j \frac{m_j}{\rho_j} W_{ij}}
{\sum_{j\in SPH} \frac{m_j}{\rho_j} W_{ij}}
$$

## SPH filter / volume sum

`flt_s`, `flt_sd`, `flt_sd_2` 같은 값은 이웃 입자 volume weight 합산입니다.

기본 꼴:

$$
F_i = \sum_j \frac{m_j}{\rho_j} W_{ij}
$$

예: DEM 입자 주변 fluid filter:

```cpp
tmp_DEMfltd += mj/rhoj * tmp_wij;     // ptypej == 1
```

따라서 filter가 0에 가까우면 주변에 해당 type의 SPH 입자가 없다는 뜻이고, 보간값을 나눌 때 `+1e-15` regularization이 들어갑니다.

## Porosity 계산

SPH 입자 위치에서 DEM solid volume fraction을 kernel sum으로 계산하고 porosity를 만듭니다.

`KERNEL_clc_prep3D_coupling_sph()`:

```cpp
tmp_por += W_ij * (4/3*pi*rad_j^3);
P1_sph[i].DEMpor = 1.0 - tmp_por / (tmp_SPHflt + 1e-20);
```

수식:

$$
\epsilon_i = 1 - \frac{\sum_{j\in DEM} V_j W_{ij}}{\sum_{j\in SPH} \frac{m_j}{\rho_j}W_{ij}}
$$

주의: `DEMpor` 이름은 porosity $\epsilon$로 쓰이고 있습니다. 즉 solid volume fraction이 아니라 void fraction에 가깝습니다.

## 밀도 관련 주의

전통적인 SPH 밀도 합산은 다음입니다.

$$
\rho_i = \sum_j m_j W_{ij}
$$

하지만 현재 SOPHIA의 활성 ISPH 경로에서는 단순히 이 식 하나만으로 밀도를 갱신한다고 보면 안 됩니다. `function_PREP.cuh`의 `KERNEL_clc_prep3D_prep_sph()`는 현재 설정에서 reference density와 mass를 다음처럼 설정합니다.

```cpp
m_ref = DENSITY_AIR * (h/1.6)^3 * pori;
P2[i].rho_ref = m_ref / (h/1.6)^3;
P1[i].rho = m_ref / (h/1.6)^3;
P1[i].m = m_ref;
```

즉 현재 코드에서는 porosity $\epsilon_i$가 mass/reference density에 직접 들어갑니다.

논문 구현 시 “밀도 계산식”을 바꿔야 한다면 반드시 다음을 함께 확인해야 합니다.

- `KERNEL_clc_prep3D_prep_sph()`
- `KERNEL_PPE3D_sph()`의 `drhostar`, `rho_ref`
- `function_TIME_ISPH.cuh`에서 `rho`가 실제로 갱신되는지 여부
- open boundary에서 `rho`를 다시 보간/설정하는지 여부

## Pressure-gradient force 형태

`KERNEL_advection_force3D_sph()`에서 pressure-gradient 관련 coupling source term은 다음과 비슷한 형태입니다.

```cpp
C_p = -pori * mj * (pi + pj) / (rhoi * rhoj);
tmp_pgf_x += rhoi / pori * C_p * tdwx;
```

수식으로 정리하면 대략:

$$
\mathbf{G}_{p,i}
= \frac{\rho_i}{\epsilon_i}
\sum_j \left[-\epsilon_i m_j\frac{p_i+p_j}{\rho_i\rho_j}\right]
\nabla_i^c W_{ij}
$$

이 값은 `P1[i].pgf_x/y/z`에 저장되고, DEM coupling에서 DEM이 받는 pressure-gradient force 계산에 사용됩니다.

## Fluid thermal conduction 형태

`KERNEL_advection_force3D_sph()`의 열전도는 현재 porosity가 들어간 diffusion 형태입니다.

주요 코드:

```cpp
kci = ((1 - sqrt(1 - epsi)) / epsi) * kfi;
chii = epsi * kci;
chih = 2 * chii * chij / (chii + chij + 1e-30);
sum_con_H = 2 * (mj/rhoj) * chih * (Ti - Tj)
          * (r_ij dot gradW) / (r_ij^2 + eta^2);
sum_con_H /= epsi * rhoi;
P3[i].dtemp = tmp_Rc / cpi - eulert;
```

수식:

$$
\chi_i = \epsilon_i k_{eff,i}
= \left(1-\sqrt{1-\epsilon_i}\right)k_{f,i}
$$

$$
\chi_{ij}^{H}=\frac{2\chi_i\chi_j}{\chi_i+\chi_j}
$$

$$
\frac{dT_i}{dt}
= \frac{1}{C_{p,i}}
\sum_j
\frac{2m_j}{\rho_j}
\frac{\chi_{ij}^{H}}{\epsilon_i\rho_i}
(T_i-T_j)
\frac{\mathbf{r}_{ij}\cdot\nabla_i^c W_{ij}}{r_{ij}^2+\eta^2}
- \text{Eulerian advection term}
$$

주의: 부호는 $\mathbf{r}_{ij}\cdot\nabla W$가 보통 음수인 점까지 포함해서 해석해야 합니다.

## XSPH correction

`function_XSPH.cuh`는 속도를 다음 형태로 smoothing합니다.

```cpp
tmpx += k_c_xsph * mj/rhoj * (-uxi + uxj) * W_ij;
```

수식:

$$
\Delta \mathbf{u}_i^{XSPH}
= c_{xsph}\sum_j \frac{m_j}{\rho_j}(\mathbf{u}_j-\mathbf{u}_i)W_{ij}
$$

온도도 유사하게 작은 계수 `0.01*k_c_xsph`로 smoothing합니다.

## 논문 구현 시 사용법

논문에 다음과 같은 식이 나오면, 바로 코드를 바꾸지 말고 이 노트의 이산화 형태와 비교해야 합니다.

- $\rho_i=\sum_j m_jW_{ij}$: 현재 활성 경로가 정말 밀도 합산형인지 확인
- $\nabla p$: SOPHIA의 symmetric pressure form과 porosity factor 확인
- $\nabla\cdot(k\nabla T)$: harmonic mean, porosity storage term, $C_p$ division 위치 확인
- $hA(T_f-T_s)$: DEM-SUPH coupling의 보간 온도, Nusselt correlation, 면적/체적 conversion 확인

## 관련 노트

- [[SOPHIA]]
- [[SOPHIA ISPH Solver]]
- [[SOPHIA SPH-DEM Coupling]]
- [[SOPHIA Solid-Fluid Heat Transfer]]
- [[SOPHIA Paper Equation Audit Workflow]]
