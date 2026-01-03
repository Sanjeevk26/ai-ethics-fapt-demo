# AI Ethics in Action (PTAF) — 1-Page Cheat Sheet

This repo demonstrates **AI Ethics in practice** using the four core principles:

- **P**rivacy
- **T**ransparency
- **A**ccountability
- **F**airness

It produces portfolio-ready artifacts in `./artifacts/`.

---

## 1) Privacy — “Protect personal and sensitive data”

### What Privacy means (in this project)
Prevent re-identification and avoid leaking individual-level information.

### Where it happens in the code
- **Synthetic PII creation (for learning only)**
  - `make_synthetic_hiring_data()`
- **Pseudonymization (remove direct identifiers)**
  - `pseudonymize()`
- **Generalization / k-anonymity style check (reduce quasi-identifier risk)**
  - `generalize_quasi_identifiers()`
- **DP-like noisy aggregates (share statistics safely)**
  - `dp_like_mean()`

### Artifacts you can point to
- `raw_data.csv` → contains direct identifiers (demo only)
- `pseudonymized_data.csv` → identifiers removed, `pseudo_id` added
- `generalized_data.csv` → quasi-identifiers generalized (age bands, zip prefixes)
- `k_anonymity_group_sizes.csv` → shows which quasi-identifier combos are rare (risk)
- `dp_like_aggregates.json` → noisy mean statistics (privacy-preserving sharing)

### How to explain it in one line
**“We remove direct identifiers, reduce uniqueness of quasi-identifiers, and share only noisy aggregates.”**

---

## 2) Transparency — “Make AI explainable and understandable”

### What Transparency means (in this project)
Make the model’s behavior inspectable and provide simple explanations for decisions.

### Where it happens in the code
- **Interpretable model pipeline**
  - `build_pipeline()`
- **Feature naming after one-hot encoding**
  - `feature_names_from_pipeline()`
- **Global explanation: coefficients**
  - `coefficient_table()`
- **Local explanation: reason codes per prediction**
  - `reason_codes()`
- **Plain-language model documentation**
  - `make_model_card()`

### Artifacts you can point to
- `coefficients.csv` → global interpretability (what matters most)
- `sample_explanations.json` → decision-level explanations (“why this outcome?”)
- `model_card.json` → intended use, limitations, metrics, ethical notes

### How to explain it in one line
**“We use an interpretable model, export coefficients, generate reason codes, and publish a model card.”**

---

## 3) Accountability — “Humans remain responsible”

### What Accountability means (in this project)
Track what happened, who ran what, and on which data/model version—so decisions are auditable.

### Where it happens in the code
- **Timestamping**
  - `now_iso()`
- **Dataset versioning**
  - `df_fingerprint()`
- **Model configuration versioning**
  - `pipeline_fingerprint()`
- **Audit logging system**
  - `AuditEvent` (structure)
  - `Auditor` (collector/writer)
  - `Auditor.log()` / `Auditor.flush()`
- **Audit every prediction**
  - `predict_with_audit()`
- **Human-in-the-loop escalation**
  - `hitl_policy()`

### Artifacts you can point to
- `audit_log.jsonl` → append-only log of model registry + predictions + HITL outcomes
- `model_card.json` → includes ethical responsibility boundaries
- `metrics.json` → metrics attached to a time + run

### How to explain it in one line
**“We fingerprint data+model, record an audit trail for decisions, and include a human escalation policy.”**

---

## 4) Fairness — “Prevent bias and unequal outcomes”

### What Fairness means (in this project)
Measure whether outcomes differ across a sensitive attribute (`group`) and apply a mitigation.

### Where it happens in the code
- **Subgroup metrics**
  - `subgroup_report()`
- **Disparate impact (selection rate parity)**
  - `disparate_impact()`
- **Equal opportunity (TPR parity / recall by group)**
  - `equal_opportunity_diff()`
- **Mitigation: group-specific thresholds (demo)**
  - `apply_group_thresholds()`
  - `search_thresholds()`

### Artifacts you can point to
- `fairness_subgroup_metrics_baseline.csv` → metrics by group before mitigation
- `fairness_subgroup_metrics_mitigated.csv` → metrics by group after mitigation
- `fairness_summary.json` → DI ratio, TPR per group, thresholds chosen

### How to explain it in one line
**“We evaluate subgroup performance, compute disparate impact + equal opportunity, then mitigate using threshold tuning.”**

---

## AI Lifecycle Mapping (Where PTAF fits)

| Lifecycle Stage | What this repo shows | Key functions |
|---|---|---|
| Data Collection + Preprocessing | Privacy controls & risk checks | `pseudonymize`, `generalize_quasi_identifiers`, `dp_like_mean` |
| Model Training + Evaluation | Transparency + baseline fairness metrics | `build_pipeline`, `coefficient_table`, `subgroup_report` |
| Deployment + Monitoring | Accountability + fairness monitoring + HITL | `Auditor`, `predict_with_audit`, `hitl_policy`, `fairness_summary` |

---

## “Explain this repo in 3 lines” (for interviews / GitHub)

1. **Privacy:** remove identifiers, generalize quasi-identifiers, and share noisy aggregates.  
2. **Transparency:** export coefficients, decision reason codes, and a model card.  
3. **Accountability + Fairness:** log audits with fingerprints, measure subgroup bias, and apply a mitigation demo.

---

## Quick Pointers for Your GitHub Description

- Portfolio demo of **Responsible AI** practices across PTAF principles  
- Fully reproducible run → generates auditable artifacts  
- Emphasizes **learning + explainability + governance**, not “model accuracy hype”

---
