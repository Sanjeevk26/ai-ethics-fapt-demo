

## Utilities

### 1) `now_iso()`

**What it does:** Gives you “what time is it right now?” in a standard format.

**Why it matters:** In AI governance, you always want to know *when* something was produced (model card, fairness report, audit log).

**Simple example:**
If you create `model_card.json` today, it can store:
`created_at = 2026-01-03T22:15:10`
So later you can prove “this file was generated on this date.”

---

### 2) `ensure_dir(path)`

**What it does:** Makes sure a folder exists (creates it if missing).

**Why it matters:** Your script writes lots of outputs in `./artifacts`. If the folder isn’t there, writing will fail.

**Simple example:**
If `artifacts/` doesn’t exist and you try saving `coefficients.csv`, it crashes.
`ensure_dir(artifacts)` prevents that.

---

### 3) `write_json(path, payload)`

**What it does:** Saves structured data (dictionary/list) into a clean readable JSON file.

**Why it matters:** JSON is easy to share on GitHub and easy to read.

**Simple example:**
You want to save:

* accuracy: 0.78
* precision: 0.73
  This function writes it neatly to `metrics.json`.

---

### 4) `write_jsonl(path, events)`

**What it does:** Writes logs in **JSON Lines** format (one JSON per line).

**Why it matters:** Logs are append-only. JSONL is perfect because you can keep adding events without rewriting the whole file.

**Simple example:**
Audit log file looks like:

* line 1: model registered
* line 2: prediction made
* line 3: hitl policy decision
  Each line is independent.

---

## Data Generation

### 5) `make_synthetic_hiring_data(n, seed)`

**What it does:** Creates a fake hiring dataset that includes:

* direct identifiers (name, email)
* quasi-identifiers (age, zipcode, education)
* sensitive attribute (`group`)
* target label (`recommended_for_review`)

**Why it matters:** You can publish it on GitHub safely (no real personal data), but still demonstrate privacy + fairness.

**Simple example:**
One row might look like:

* name: Candidate_12
* email: [user12@example.com](mailto:user12@example.com)
* age: 28
* zipcode: 10006
* education: Masters
* skill_score: 72
* group: 1
* recommended_for_review: 0

---

## Privacy

### 6) `pseudonymize(df, salt, id_cols=("name","email"))`

**What it does:** Replaces direct identifiers with a `pseudo_id` and removes name/email.

**Key idea:** “I don’t need to know your name to analyze fairness or train a model.”

**Simple example:**
Before:

* name: “Rahul”
* email: “[rahul@gmail.com](mailto:rahul@gmail.com)”

After:

* pseudo_id: “a81f3c9d12ab”
* (name/email removed)

You can still refer to the same person by `pseudo_id`, but you can’t easily identify them.

---

### 7) `generalize_quasi_identifiers(df, k=15, quasi=("age","zipcode","education"))`

**What it does:** Reduces re-identification risk by making quasi-identifiers less specific:

* age → age band (25–29)
* zipcode → prefix (100**)
  …and then counts how many people share the same combo.

**Why it matters:** Even without name/email, “28 + 10006 + Masters” might still identify someone.

**Simple example:**
Exact info:

* age=28, zipcode=10006, education=Masters

Generalized:

* age_band=25-29, zip_prefix=100**, education=Masters

Then it checks: “How many rows share (25-29, 100**, Masters)?”
If only 2 people share it → high re-identification risk.
If 30 share it → safer.

---

### 8) `dp_like_mean(values, epsilon, value_range, seed)`

**What it does:** Adds a small amount of noise to an average to simulate **differential privacy** conceptually.

**Mental model:** “Share trends without leaking individuals.”

**Simple example:**
Real average skill score for group 1 = **71.2**
DP-like average (after noise) = **71.7**

So you can publish “around 71” without revealing exact contributions.

**What does epsilon mean?**

* **small epsilon (0.1)** → more noise → more privacy → less accuracy
* **bigger epsilon (3.0)** → less noise → less privacy → more accuracy

---

## Transparency

### 9) `build_pipeline(cat_cols, num_cols)`

**What it does:** Builds a training pipeline:

* One-hot encodes categorical columns
* Keeps numeric columns as-is
* Trains logistic regression

**Why it matters:** Consistency. Whatever preprocessing you do in training is automatically used in prediction.

**Simple example:**
Education = “Masters” becomes:

* education_Bachelors = 0
* education_Masters = 1
* education_PhD = 0

---

### 10) `feature_names_from_pipeline(pipe, cat_cols, num_cols)`

**What it does:** Extracts the real feature names after one-hot encoding.

**Why it matters:** Otherwise you can’t interpret coefficients properly.

**Simple example:**
Instead of “education”, you now have:

* education_Bachelors
* education_Masters
* education_PhD

So when a coefficient is large, you know exactly what it refers to.

---

### 11) `coefficient_table(pipe, cat_cols, num_cols)`

**What it does:** Produces a ranked list of features and their coefficients.

**How to remember coefficients:**

* positive coefficient → pushes decision toward “REVIEW”
* negative coefficient → pushes decision toward “NO REVIEW”
* larger magnitude → stronger influence

**Simple example:**

* skill_score coef = +0.05 → higher skill increases review chance
* education_PhD coef = +0.60 → PhD strongly increases review chance
* experience_years coef = +0.08 → more experience increases chance

---

### 12) `reason_codes(pipe, row_df, cat_cols, num_cols, top_k=5)`

**What it does:** Explains a *single prediction* by listing the top features that pushed the result up/down.

**Simple example:**
Candidate has:

* skill_score = 90 (strong positive push)
* experience_years = 10 (positive push)
* education = Masters (moderate push)

Reason codes might say:

1. skill_score contributed +0.9
2. experience_years contributed +0.5
3. education_Masters contributed +0.2

So the “why” is visible.

---

### 13) `make_model_card(...)`

**What it does:** Creates a structured “documentation summary” for the model.

**Why it matters:** This is transparency + accountability in real life.

**Simple example model card sections:**

* Intended use: “Recommend candidates for human review”
* Not intended: “Do not auto-reject/hire”
* Limitations: “Synthetic data; not real hiring”
* Metrics: accuracy/precision/recall
* Ethics notes: privacy, fairness, monitoring

---

## Accountability

### 14) `df_fingerprint(df)`

**What it does:** Creates a hash signature of the dataset.

**Mental model:** “Dataset version number, but automatic.”

**Simple example:**
If someone changes even one row, fingerprint changes completely.
So you can prove: “Model v1 trained on dataset fingerprint XYZ.”

---

### 15) `pipeline_fingerprint(pipe)`

**What it does:** Hash signature of the model configuration (pipeline parameters).

**Why it matters:** If you change:

* threshold
* preprocessing rules
* model parameters
  the fingerprint changes.

**Simple example:**
Model built with max_iter=2000 has a different fingerprint from max_iter=500.

---

### 16) `AuditEvent` (dataclass)

**What it is:** A structured format for an audit record:

* timestamp
* event_type
* payload (details)

**Simple example:**
Event type: “prediction”
Payload: probability=0.62, threshold=0.50, decision=REVIEW, reasons=[…]

---

### 17) `Auditor` (class)

**What it does:** Stores audit events and writes them to a JSONL log.

**Why it matters:** Audit logs are a key accountability requirement.

**Simple example:**
Every time a decision happens:

* who asked
* what input
* what output
* why (reason codes)
  gets logged.

---

### 18) `Auditor.log(event_type, payload)`

**What it does:** Adds an event into memory buffer.

**Simple example:**
log(“model_registry”, {model_name:…, dataset_hash:…})

---

### 19) `Auditor.flush()`

**What it does:** Writes buffered events to `audit_log.jsonl`.

**Simple example:**
You log 10 events, then flush writes all 10 lines to disk.

---

### 20) `predict_with_audit(...)`

**What it does:** Makes prediction + logs everything about it.

**Why it matters:** You don’t want “silent” AI decisions.

**Simple example:**
Candidate probability = 0.58
Threshold = 0.50
Decision = REVIEW
Reason codes = [skill_score high, Masters, experience moderate]
All of this is recorded.

---

### 21) `hitl_policy(probability, low=0.45, high=0.65)`

**What it does:** Decides if the system should:

* auto-approve review
* auto-reject review
* or send to a human

**Simple example:**

* p=0.20 → AUTO_NO_REVIEW
* p=0.90 → AUTO_REVIEW
* p=0.55 → ESCALATE_TO_HUMAN (uncertain)

---

## Fairness

### 22) `subgroup_report(y_true, y_pred, sensitive)`

**What it does:** Calculates metrics for each group separately.

**Why it matters:** Overall accuracy can hide harm.

**Simple example:**
Group 0 recall = 0.75
Group 1 recall = 0.45
That means the model misses many true positives in group 1.

---

### 23) `disparate_impact(y_pred, sensitive)`

**What it does:** Compares selection rates:

* selection rate = % predicted positive

**DI ratio = selection_rate(group1) / selection_rate(group0)**

**Simple example:**

* group0 selection_rate = 0.50
* group1 selection_rate = 0.25
  DI = 0.25 / 0.50 = **0.5**
  Meaning group1 gets selected half as often.

---

### 24) `equal_opportunity_diff(y_true, y_pred, sensitive)`

**What it does:** Compares True Positive Rate (recall) between groups.

**EOD = TPR(group1) - TPR(group0)**

**Simple example:**

* TPR group0 = 0.80
* TPR group1 = 0.60
  EOD = -0.20
  Meaning group1 is being missed more often among truly qualified positives.

---

### 25) `apply_group_thresholds(proba, sensitive, t0, t1)`

**What it does:** Uses different thresholds for different groups (demo mitigation).

**Simple example:**

* group0: threshold 0.50
* group1: threshold 0.45
  This increases positives for group1, reducing disparity.

(Real-world note: must be governed carefully.)

---

### 26) `search_thresholds(proba, y_true, sensitive, grid)`

**What it does:** Tries many threshold combinations (t0, t1) and picks the one that:

* makes DI closer to 1.0
* while keeping accuracy decent

**Simple example:**
It tries:

* (0.50, 0.50)
* (0.50, 0.45)
* (0.55, 0.45)
  … and selects the best compromise.

---

