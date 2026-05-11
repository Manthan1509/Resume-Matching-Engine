# Resume Matching Engine
**Redrob AI Campus Hackathon — Individual Competition**

A from-scratch implementation of a TF-IDF based resume matching system that ranks candidates against job descriptions using cosine similarity. Built using only Python standard library — no external dependencies.

---

## Problem Statement

Given 10 resume datasets from Indian university students and 3 Job Descriptions (JDs) from Korean technology companies, build a program that:

- Normalizes noisy resume skill data using a predefined alias map
- Computes TF-IDF vectors for resumes
- Builds binary vectors for job descriptions
- Calculates cosine similarity between resumes and JDs
- Outputs the Top 3 matching candidates per JD

---

## Project Structure

```
resume_matching_engine.py   # Complete solution — single file, no dependencies
README.md                   # This file
```

---

## How It Works

### Step 1 — Skill Normalization
Each resume's raw skill string is split on commas, lowercased, and looked up in the `SKILL_ALIASES` map. Multi-word phrases (e.g. `"spring boot"`, `"feature engineering"`) are preserved as intact tokens after comma-splitting, so they match naturally. Tokens not present in the alias map are discarded.

```
Raw:          "Pyhton, MachineLearning, SQL, pandas, numpy, Deep-learning"
Lowercase:     pyhton, machinelearning, sql, pandas, numpy, deep-learning
After alias:   python, machine_learning, sql, pandas, numpy, deep_learning
```

### Step 2 — Deduplication
Each canonical skill may appear only once per resume. Skills that map to the same canonical form (e.g. `"data-viz"` and `"matplotlib"` both → `data_visualization`) are collapsed to a single entry.

### Step 3 — Vocabulary Construction
A shared vocabulary is built from all normalized, deduplicated resume skills. Sorted alphabetically and used as a consistent index for all vectors.

> **Vocabulary size: 48 skills**  
> Skills in JDs but absent from all resumes (e.g. `pytorch`, `redis`) are excluded from the vocabulary and silently ignored when building JD vectors.

### Step 4 — TF-IDF Vectors (Resumes only)

```
TF(skill, resume)  =  1 / N
```
Where N = total unique skills in that resume (always 1 after deduplication, so TF = 1/N).

```
IDF(skill)  =  ln( 10 / df(skill) )
```
Where df = number of resumes containing the skill. Natural log, no smoothing.

```
TF-IDF  =  TF × IDF
```

### Step 5 — JD Binary Vectors
Each JD skill is normalized using the same alias map and looked up against the vocabulary. Present → 1.0, absent → 0.0.

### Step 6 — Cosine Similarity & Ranking

```
Cosine(A, B)  =  (A · B) / (|A| × |B|)
```
Where A = resume TF-IDF vector, B = JD binary vector. Candidates ranked by descending score; ties broken alphabetically by name.

---

## Results

```
JD-1 — Kakao (ML Engineer)
Sneha Patel(0.57), Karan Mehta(0.53), Arjun Sharma(0.40)

JD-2 — Naver (Backend Engineer)
Rahul Gupta(0.81), Ananya Krishnan(0.28), Deepika Rao(0.19)

JD-3 — Line (Frontend Engineer)
Aditya Kumar(0.67), Priya Nair(0.58), Ananya Krishnan(0.35)
```

---

## Result Insights

**JD-1 (Kakao — ML Engineer)**
Sneha Patel edges out Karan Mehta despite both having 5 overlapping skills. Sneha's overlap includes `bert`, `nlp`, and `tensorflow` — all with IDF = 2.30 (appear in only 1 resume). Karan's overlap includes `sql` (IDF = 1.61) and `data_visualization` (IDF = 1.20), which are more common and thus weighted lower.

**JD-2 (Naver — Backend Engineer)**
Rahul Gupta scores a dominant 0.81 — all 6 of his skills (`java`, `spring_boot`, `mysql`, `microservices`, `docker`, `kubernetes`) directly match JD-2 requirements, and most are rare skills (IDF = 2.30) that appear in only one resume.

**JD-3 (Line — Frontend Engineer)**
Aditya Kumar leads with 6 overlapping skills (`react`, `typescript`, `graphql`, `redux`, `jest`, `nodejs`). Priya Nair follows with 5 overlaps but slightly lower score because her N=6 gives higher TF weight than Aditya's N=7.

---

## Key IDF Values

| Skill | df | IDF |
|---|---|---|
| `python` | 6 | 0.51 — very common, low weight |
| `machine_learning` | 3 | 1.20 |
| `data_visualization` | 3 | 1.20 |
| `java` | 2 | 1.61 |
| `javascript` | 2 | 1.61 |
| `react` | 2 | 1.61 |
| `rest_api` | 2 | 1.61 |
| `sql` | 2 | 1.61 |
| `nodejs` | 2 | 1.61 |
| All other skills | 1 | 2.30 — rare, high weight |

---

## Running the Code

```bash
python3 resume_matching_engine.py
```

**Requirements:** Python 3.x — uses only the `math` module from the standard library. No pip installs needed.

---

## Design Decisions

- **No external libraries** — `math.log`, `math.sqrt`, and list comprehensions replace numpy entirely, as required by the problem rules.
- **Multi-word phrase matching** — Since skills are comma-separated, tokens like `"spring boot"` or `"feature engineering"` survive intact after splitting. A direct dictionary lookup handles both single and multi-word aliases uniformly.
- **JD normalization uses the same pipeline** — JD skills are passed through the same `normalize()` function, ensuring consistent canonical names. Skills not in the resume vocabulary (pytorch, redis) are excluded only at the vector-building stage, not during normalization.
- **Tie-breaking** — Scores are rounded to 2 decimal places before sorting, then alphabetical order breaks ties, as specified.

---

## Validated Intermediate Output

```
Arjun Sharma   (N=6):  python, machine_learning, sql, pandas, numpy, deep_learning
Priya Nair     (N=6):  javascript, react, nodejs, mongodb, rest_api, html_css
Rahul Gupta    (N=6):  java, spring_boot, mysql, microservices, docker, kubernetes
Sneha Patel    (N=6):  python, tensorflow, keras, nlp, bert, data_visualization
Vikram Singh   (N=5):  cpp, algorithms, data_structures, competitive_programming, python
Ananya Krishnan(N=7):  javascript, vue, python, flask, postgresql, aws, ci_cd
Karan Mehta    (N=6):  python, machine_learning, xgboost, feature_engineering, sql, data_visualization
Deepika Rao    (N=7):  java, android, kotlin, firebase, rest_api, ui_ux, figma
Aditya Kumar   (N=7):  react, typescript, graphql, redux, tailwind, nodejs, jest
Meera Iyer     (N=7):  python, r, statistics, machine_learning, regression, clustering, data_visualization
```

---

*Redrob AI Campus Hackathon · Powered by McKinley Rice*
