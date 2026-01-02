# AI Ethics in Action: Privacy, Transparency, Accountability & Fairness (PTAF)

This repository demonstrates **how core AI ethics principles can be implemented in practice**, not just discussed in theory.  
It provides an end-to-end, runnable machine learning example that shows how **responsible AI systems are designed, evaluated, and governed**.

The focus is deliberately on **clarity, auditability, and ethical safeguards**, using a simple and interpretable model rather than a complex black box.

---

## What This Project Demonstrates

The project operationalizes four widely accepted AI ethics principles:

### 1. Privacy
- Removal of direct identifiers (names, emails)
- Pseudonymization using salted hashing
- Generalization of quasi-identifiers (k-anonymity–style)
- Differential-privacy–inspired noisy aggregate reporting (educational demo)

### 2. Transparency
- Interpretable logistic regression model
- Exported feature coefficients
- Per-decision “reason codes” explaining predictions
- A structured **model card** describing intent, data usage, metrics, and limitations

### 3. Accountability
- Dataset and model fingerprinting for traceability
- Structured audit logs for predictions and decisions
- Human-in-the-loop escalation policy for low-confidence cases
- Explicit ownership and decision thresholds

### 4. Fairness
- Subgroup performance metrics (accuracy, precision, recall)
- Selection rate comparison across groups
- Disparate Impact ratio calculation
- Equal Opportunity difference evaluation
- Demonstration of threshold-based mitigation (for learning purposes)

---

## Why the Model Is Simple (By Design)

The model used is **logistic regression**, intentionally chosen because:
- Its decisions are explainable
- Feature contributions are inspectable
- Ethical risks are easier to reason about
- It aligns with governance and audit requirements

The goal is **ethical clarity**, not leaderboard performance.

---

## Dataset

- Fully **synthetic dataset** (safe to share publicly)
- Simulates a hiring-screening scenario
- Includes:
  - Direct identifiers (removed during preprocessing)
  - Quasi-identifiers (generalized)
  - A sensitive attribute (used only for fairness evaluation)
- Labels represent *recommendation for human review*, not automated decisions

---

## Repository Structure

```text
ai-ethics-ptaf-demo/
│
├── ethics_in_action.py        # Main runnable script
├── requirements.txt           # Python dependencies
├── README.md                  # Project documentation
└── artifacts/                 # Generated outputs (after running)
    ├── model_card.json
    ├── coefficients.csv
    ├── sample_explanations.json
    ├── audit_log.jsonl
    ├── fairness_summary.json
    ├── fairness_subgroup_metrics_baseline.csv
    ├── fairness_subgroup_metrics_mitigated.csv
    ├── dp_like_aggregates.json
    └── k_anonymity_group_sizes.csv
