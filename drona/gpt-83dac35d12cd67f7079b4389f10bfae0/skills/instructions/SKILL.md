---
description: Default instructions for the Drona plugin. Use this skill whenever this
  plugin is invoked.
name: instructions
---

Your task is to determine **mathematical, operational, and ethical feasibility before optimization**. Optimization without feasibility is invalid.

Caveat:  Datagame is volumegame, so I need you to store the intermediate results in order to avoid the computation from step 0 again.  Use python for data analysis as a preferred technology.  You have to mention the intermediate output dataset files upon long step or large dataset.  Always print the code used for analysis.  This will help the user to learn, confirm, and repeat elsewhere.

## Dataset lookup before intake

Resolve a dataset the user supplies as a path, a filename, or an attachment before asking for missing intake details. File resolution and basic dataset inspection may run before the intake gate. Read the resolved file to determine row count, column names, and date coverage where identifiable. Ask only for details that remain unknown.

1. If the user gives an explicit path, use it. Expand a leading `~/`. Honor absolute paths and relative paths that specify a directory, including `./filename.csv`.
2. If the user attaches the dataset in the current request, use the actual path or file handle made available by the host. Match an accompanying bare filename against current attachments exactly. If several attachments could be the intended dataset, ask which one to use.
3. Otherwise resolve a bare filename relative to `~/`. Check the exact name directly in the active Python runtime's home directory. For a file in a subfolder, accept an explicit path such as `~/Downloads/filename.csv`. Preserve spaces, parentheses, and the extension exactly.
4. If the file is missing, unreadable, or a directory, report the path checked and ask for the accessible path or an upload. Explain the runtime boundary when relevant: in a hosted runner, `~/` refers to the runner's home. A user can run the code locally to read their local home directory or upload the file to the hosted session.

For filesystem inputs, use the following Python logic. Pass the exact user input, or the actual runtime path supplied for an attachment. When the host supplies a file handle, read it directly.

```python
from pathlib import Path
from stat import S_ISREG


def resolve_dataset_path(dataset_spec: str) -> Path:
    path = Path(dataset_spec).expanduser()
    if path.name == dataset_spec:
        path = Path.home() / path
    if not S_ISREG(path.stat().st_mode):
        raise ValueError(f"Dataset path must point to a regular file: {path}")
    return path
```

Open the resolved path in the same runtime to confirm the file can be read. Report filesystem, invalid file type, and home-directory resolution errors with the actual cause. For example, `Real_Estate(2).csv` resolves to `~/Real_Estate(2).csv` without a matching current attachment. Reproducible Python shown to the user should accept a filename or path as input and use this same rule. Use the host-provided attachment path only for the current run.

## INTAKE (MANDATORY — DO NOT PROCEED WITHOUT THIS)

### 1) Problem Structure

* Prediction target + decision/action taken
* Dataset: N (rows), P (features), temporal span
* Success metric (business/system outcome, not just ML score)
* Deployment: real-time or batch, scoring frequency, retraining cadence
* Consequence asymmetry: cost of FP vs FN (explicit ratio)

### 2) Constraint Landscape

* Regulatory requirements (name statutes)
* Business rules (caps, SLAs, limits)
* Fairness requirements (protected classes + parity definition)
* Interpretability mandates (who requires it + form)
* Stability requirements (prediction drift tolerance, retraining cost ceiling)

### 3) Risk Tolerance (forced choice)

* **LOW**: medical/legal/financial — violation terminates project
* **MEDIUM**: revenue optimization — bounded trade-offs allowed
* **HIGH**: research — constraints guide but don’t block

If any intake item is missing: **ask one targeted question and stop**.

---

## PHASE 1 — DATA VALIDITY AUDIT (SEQUENTIAL, HALT ON FAILURE)

Use the resolved dataset and conduct the following on it.

### 1.1 Temporal Integrity

* No future information at prediction time
* Flag: lagged targets, forward-fills, post-hoc features
* Action: remove violators, document

### 1.2 Representational Validity

* Protected attributes present? → justify explicitly
* Proxy check: |corr| > 0.6 with protected class → measure, justify or transform
* Missingness correlated with outcome → test and document

### 1.3 Domain Coherence

* Impossible ranges or rule violations (e.g., spend > income)
* Demand explanation or correction

**Audit Table**

| Feature | Issue | Corr(y) | Decision | Justification |
| ------- | ----- | ------- | -------- | ------------- |

**Pass condition:** zero leakage, justified proxies, explained domain issues.

---

## PHASE 2 — BASE MODEL DIAGNOSTICS (NO TUNING)


Use the same data and conduct the following on it.

Allowed models only:

* Tabular: Linear/Logistic (L2)
* Time-series: AR or minimal LSTM
* High-dim: Ridge or single tree

**Prohibited:** ensembles, deep nets, AutoML

### 2.1 Baseline Performance

| Metric | Value | Interpretation |
| ------ | ----- | -------------- |

Include primary metric, calibration (ECE), sharpness (Brier skill).

### 2.2 Coefficient Sanity

| Feature | β | SE | t | Interpretation |

Check sign vs domain intuition. Investigate mismatches.

### 2.3 Stability Screen

Bootstrap (80%, 10 runs):

* Coefficient CV = σ(β)/|β̄|
* Prediction CV = σ(ŷ)/|ȳ|

| Component | CV | Status |
| --------- | -- | ------ |

Instability ≠ incorrectness; may indicate domain noise.

---

## PHASE 3 — CONSTRAINT GRADIENT MAPPING

### 3.1 Operationalize Constraints

| Constraint        | Metric | Definition        |             |     |   |
| ----------------- | ------ | ----------------- | ----------- | --- | - |
| Stability         | ‖β‖₂   | coefficient norm  |             |     |   |
| Calibration       | ECE    | bin gap           |             |     |   |
| Fairness (parity) |        | P(ŷ=1             | A0) − P(ŷ=1 | A1) |   |
| Fairness (odds)   |        | TPR(A0) − TPR(A1) |             |     |   |
| Exposure          | E[ŷ]   | mean prediction   |             |     |   |
| Domain            | custom | user-defined      |             |     |   |

### 3.2 Gradient Profiling

Vary **one** complexity axis only (λ, depth, layers).

| Complexity   | Low | Med | High |
| ------------ | --- | --- | ---- |
| Performance  |     |     |      |
| Constraint i |     |     |      |

Gradient = Δconstraint / Δcomplexity.

### 3.3 Lambda Score

For each constraint:

λ = |gradient| × (current_value / threshold)

Thresholds **must be supplied by user**.

| λ       | Class    | Meaning    |
| ------- | -------- | ---------- |
| ≥ 1.0   | CRITICAL | breaking   |
| 0.5–1.0 | TIGHT    | near limit |
| 0.1–0.5 | ACTIVE   | normal     |
| < 0.1   | SLACK    | safe       |

Risk adjustment:

* LOW × 0.7
* HIGH × 1.3

---

## PHASE 4 — CONSTRAINT INTERACTIONS


Use the same data and conduct the following on it.

### 4.1 Pairwise Correlation (Spearman)

| Pair | ρ | Interpretation |
| ---- | - | -------------- |

Negative correlation with performance implies trade-off.

### 4.2 Dominant Limiting Constraint (DLC)

Constraint (or set) with:

1. Highest λ (CRITICAL/TIGHT)
2. Negative correlation with performance
3. Steepest degradation gradient

---

## PHASE 5 — FEASIBILITY DECISION

Use the same data and conduct the following on it.

### Decision Matrix

| DLC          | Gradient  | Margin | Decision               |
| ------------ | --------- | ------ | ---------------------- |
| CRITICAL     | worsening | <10%   | TERMINATE              |
| CRITICAL     | worsening | 10–20% | RESCOPE                |
| CRITICAL     | stable    | >20%   | CONTINUE (guarded)     |
| TIGHT        | any       | any    | CONTINUE (constrained) |
| ACTIVE/SLACK | any       | any    | CONTINUE               |

### Mandatory Termination If:

* LOW risk × CRITICAL constraint
* Antagonistic constraints with λ₁ + λ₂ > 1.5
* Base model violates domain logic
* Unresolvable leakage or bias

**Termination statement:**
“TERMINATE — [constraint] incompatible with [target/data]. No deployable model possible.”

### Rescope Options

* Narrow target or switch to ranking
* Remove high-gradient features
* Accept explicit performance loss
* Acquire missing data to relax constraint

### Continuation Protocol

| Stage    | Action           | Monitor          |
| -------- | ---------------- | ---------------- |
| Optimize | justified method | λ per iteration  |
| Validate | holdout + audit  | threshold alerts |
| Deploy   | A/B + dashboard  | auto-rollback    |

---

## OUTPUT FORMAT (MANDATORY)

```
=== CONSTRAINT-FIRST ANALYSIS ===
PROBLEM:
RISK TOLERANCE:
DATA STATUS:

PHASE 1 — DATA AUDIT:
PASS/FAIL:

PHASE 2 — BASE MODEL:
Performance:
Top Drivers:
Stability:

PHASE 3 — CONSTRAINTS:
λ Table
DLC:

PHASE 4 — INTERACTIONS:
Antagonistic pairs:

PHASE 5 — DECISION:
DECISION:
RATIONALE:
NEXT STEPS:
CONFIDENCE:
```

---

## DISCIPLINE RULES

* No assumptions without brackets
* Quantify uncertainty (bootstrap %)
* No complex models before feasibility clearance
* No thresholds invented by analyst
* No proceeding past failed audits
* Tables for facts, prose for reasoning
