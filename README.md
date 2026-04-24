# CART-Tumor-ModelRepository

Code and data accompanying the preprint:

> **A mechanistic model for dynamics between CAR T cells and target cells captures features that determine killing profiles**
> Suriya Selvarajan, Kevin L. Scrudders, Kenneth Rodriguez-Lopez, Suilan Zheng, Bo Huang, Philip S. Low, Raghu Pasupathy, Shalini T. Low-Nam
> *bioRxiv* 2025. https://doi.org/10.1101/2025.06.24.661290

---

## Overview

This repository contains the maximum likelihood estimation (MLE) framework and experimental contact-history data used to fit and validate the parametric hazard rate model described in the paper. The model captures the conditional lifetime of a tumour cell subject to stochastic CAR T cell visitation, framed as a time-discounted integral of cell–cell engagement.

The key biological insight is that **adherent-mode CAR T cell contacts dominate killing outcomes**, while suspension-mode contacts contribute minimally — implying that a phenotypically distinct subset of the CAR T population drives the population-level therapeutic response.

---

## Repository Structure

```
CART-Tumor-ModelRepository/
│
├── README.md                   # This file
│
├── data/
│   └── CAR_T_contact_data.xlsx # Organised contact-history data for all three
│                               # experimental conditions (three sheets):
│                               #   MDA_MB231  — breast cancer cell line
│                               #   KB         — cervical cancer cell line
│                               #   KB_Trypsin — KB cells, trypsin-treated
│
└── model/
    └── main_code.ipynb       # 4-parameter MLE + grid search sanity check
```

---

## Model

The tumour cell lifetime is modelled as a right-censored survival process with a parametric hazard rate of the form:
$$
d(t;\,\mathbf{v}) = \kappa_a \int_0^t e^{-\tilde{\kappa}_a\tau}\,v_a(\tau)\,e^{-\gamma(t-\tau)}\,d\tau
+ \kappa_s \int_0^t e^{-\tilde{\kappa}_s\tau}\,v_s(\tau)\,e^{-\gamma(t-\tau)}\,d\tau
$$

| Parameter | Symbol | Interpretation |
|---|---|---|
| Adherent killing rate | $\kappa_a$ | Potency per adherent CAR T contact |
| Adherent exhaustion rate | $\tilde{\kappa}_a$ | Decay of adherent killing signal over contact duration |
| Suspension killing rate | $\kappa_s$ | Potency per suspension CAR T contact |
| Resilience / memory decay | $\gamma$ | Rate at which accumulated lethal signal decays |

$N_a(t)$ and $N_s(t)$ are the instantaneous numbers of adherent and suspension CAR T cells in contact with the tumour cell at time $t$. The suspension exhaustion rate $\tilde{\kappa}_s$ is fixed at 0 (suspension contacts do not exhaust).

Parameters are estimated by maximum likelihood over the right-censored survival data:

$$\ell(\mathbf{v}) = \sum_{i \in \text{dead}} \left[\log d(T_i;\,\mathbf{v}) - D(T_i;\,\mathbf{v})\right] - \sum_{j \in \text{alive}} D(h_j;\,\mathbf{v})$$

where $D(t;\mathbf{v}) = \int_0^t d(s;\mathbf{v})\,ds$ is the cumulative hazard.

---

## Data Format

Each experimental dataset is a Python dictionary:

```python
data = {
    'c1': [[start, end, type], [start, end, type], ..., final_time],
    'c2': [...],
    ...
}
```

- Each inner list `[start, end, type]` is one CAR T cell contact event.
  - `start`, `end` — frame numbers (1 frame = 3 min = 0.05 hr)
  - `type` — `1` = adherent, `0` = suspension
- `final_time` — last observed frame for that tumour cell
  - `161` → censored (alive at end of experiment)
  - Any other value → death frame

The `.xlsx` file organises all three experimental conditions into separate colour-coded sheets with per-contact rows and per-cell summary columns.

---

## Usage

### Requirements

```bash
pip install numpy scipy matplotlib openpyxl
```

### Run the MLE

```python
python model/main_code.ipynb
```

This will:
1. Fit the 4-parameter model by L-BFGS-B optimisation
2. Compute 95% confidence intervals via the observed Fisher information matrix
3. Run a 6⁴ = 1296-point log-spaced grid search to confirm global optimality
4. Polish the grid-best point and compare with the gradient solution
5. Produce two figures:
   - `survival_fit.png` — Kaplan–Meier vs model-predicted survival
   - `ll_surface_slices.png` — 2-D log-likelihood surface slices for all parameter pairs

### Expected Output (MDA-MB-231)

```
kappa_a       = 0.62494 hr⁻¹
kappa_a_tilda = 0.14982 hr⁻¹
kappa_s       = 0.01914 hr⁻¹
gamma         = 2.01341 hr⁻¹
log-likelihood = -123.3187
AIC            = 254.6375
```

---

## Citation

If you use this code or data, please cite:

```bibtex
@article{selvarajan2025mechanistic,
  title   = {A mechanistic model for dynamics between {CAR} {T} cells and target
             cells captures features that determine killing profiles},
  author  = {Selvarajan, Suriya and Scrudders, Kevin L. and Rodriguez-Lopez, Kenneth
             and Zheng, Suilan and Huang, Bo and Low, Philip S. and
             Pasupathy, Raghu and Low-Nam, Shalini T.},
  journal = {bioRxiv},
  year    = {2025},
  doi     = {10.1101/2025.06.24.661290}
}
```

---

## Contact

For questions about the model or data, please open a GitHub issue or contact the corresponding authors via the bioRxiv preprint page.
