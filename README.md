<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 420 110" width="420" height="110">

  <!-- ═══════════════════════════════════════════
       ICON MARK — Stylized "L" from 3 blocks
       Negative gaps = database rows
       Top block crowned with upward arrow = AI progress
  ════════════════════════════════════════════ -->

  <!-- Upward arrow (cyan) — sits above top block, symbolises training progression -->
  <polygon points="26,6 50,34 2,34" fill="#00E5FF"/>

  <!-- Top block — left-aligned (vertical stroke of L) -->
  <rect x="2" y="38" width="48" height="19" rx="3" fill="#336791"/>

  <!-- Row gap — negative space = database row separator -->

  <!-- Middle block — left-aligned (vertical stroke of L) -->
  <rect x="2" y="63" width="48" height="19" rx="3" fill="#336791"/>

  <!-- Row gap -->

  <!-- Bottom block — wide (horizontal stroke of L, forms the base) -->
  <rect x="2" y="88" width="88" height="19" rx="3" fill="#336791"/>

  <!-- Subtle cyan accent bar on right edge of bottom block — ties icon to arrow -->
  <rect x="82" y="88" width="8" height="19" rx="3" fill="#00E5FF"/>

  <!-- ═══════════════════════════════════════════
       WORDMARK — LEDGER
       Geometric bold caps, tight tracking
  ════════════════════════════════════════════ -->
  <text
    x="112"
    y="90"
    font-family="'Trebuchet MS', 'Gill Sans MT', 'Century Gothic', sans-serif"
    font-size="72"
    font-weight="bold"
    letter-spacing="3"
    fill="#0F1B2D"
  >LEDGER</text>

  <!-- Cyan underline accent beneath wordmark — grounds the type -->
  <rect x="112" y="98" width="295" height="5" rx="2.5" fill="#00E5FF" opacity="0.7"/>

</svg>



<div align="center">
<br/>

<img src="logo.svg" alt="Ledger" width="380"/>

<br/><br/>

*Persistent memory infrastructure for AI experimentation.*

<br/>

[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=00E5FF)](https://react.dev/)
[![Python](https://img.shields.io/badge/Python-0F1B2D?style=flat-square&logo=python&logoColor=00E5FF)](https://python.org/)
[![Chart.js](https://img.shields.io/badge/Chart.js-0F1B2D?style=flat-square&logo=chartdotjs&logoColor=00E5FF)](https://www.chartjs.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-336791?style=flat-square)](LICENSE)

<br/>
</div>

---

## Overview

Ledger is a relational database platform for recording and analyzing the complete lifecycle of AI experiments — models, training runs, execution steps, metrics, and datasets — through a unified web interface.

Modern AI workflows generate large volumes of disconnected records scattered across logs and scripts. Ledger centralizes this data into a structured schema, making experiment history queryable and reproducible.

---

## Data Model

```
MODEL
  └── MODEL_VERSION
        └── EXPERIMENT
              └── TRAINING_RUN
                    ├── EXECUTION_STEP
                    │     └── METRIC_SCALAR
                    ├── CHECKPOINT
                    └── RUN_DATASET_MAP
                          └── DATASET
```

| Entity | Description |
|---|---|
| MODEL | AI model metadata |
| MODEL_VERSION | Architecture versions and hyperparameter configs |
| EXPERIMENT | Research hypothesis linked to a model version |
| TRAINING_RUN | A single training session |
| EXECUTION_STEP | Step-level progress log during training |
| METRIC_SCALAR | Numeric metrics (loss, accuracy, F1) per step |
| CHECKPOINT | Model state snapshots saved during training |
| DATASET | Dataset metadata |
| RUN_DATASET_MAP | Dataset-to-run associations |

---

## Architecture

```
  React Frontend
  (Dashboard, Registry, Metrics Explorer, Timeline)
         |
      REST/JSON
         |
  FastAPI Backend
  (Route Handlers, ORM, Query Engine)
         |
        SQL
         |
  PostgreSQL Database
```

---

## Interface

**Dashboard** — aggregate statistics across the experiment database.

**Model Registry** — browse models, versions, and architecture configurations.

**Experiment Manager** — define experiments linked to model versions; compare multiple training runs per experiment.

**Training Runs** — per-session records with configuration snapshots and status tracking (`queued → running → completed → failed`).

**Metrics Explorer** — Chart.js visualizations of metric progression across training steps.

**Execution Timeline** — step-by-step reconstruction of training progress for any run.

**Dataset Registry** — dataset metadata with lineage tracking across experiments and runs.

---

## Query Examples

**Training runs per model**
```sql
SELECT m.name, COUNT(tr.runid) AS total_runs
FROM MODEL m
JOIN MODEL_VERSION mv ON m.modelid = mv.modelid
JOIN TRAINING_RUN tr  ON mv.versionid = tr.versionid
GROUP BY m.name
ORDER BY total_runs DESC;
```

**Loss curve for a specific run**
```sql
SELECT es.globalstepnumber AS step, ms.metricvalue AS loss
FROM EXECUTION_STEP es
JOIN METRIC_SCALAR ms ON es.stepid = ms.stepid
WHERE ms.metricname = 'loss' AND es.runid = :run_id
ORDER BY es.globalstepnumber;
```

**All runs using a specific dataset**
```sql
SELECT e.name, tr.runid, d.datasetname
FROM EXPERIMENT e
JOIN TRAINING_RUN tr     ON e.experimentid = tr.experimentid
JOIN RUN_DATASET_MAP rdm ON tr.runid = rdm.runid
JOIN DATASET d           ON rdm.datasetid = d.datasetid
WHERE d.datasetname = :target_dataset;
```

---

## Stack

| Layer | Technology |
|---|---|
| Frontend | React, Chart.js |
| Backend | Python, FastAPI |
| Database | PostgreSQL |

---

## Setup

```bash
git clone https://github.com/yourusername/ledger
cd ledger
```

```bash
export DATABASE_URL="postgresql://user:password@localhost:5432/ledger"
psql -d ledger -f schema.sql
```

```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

```bash
cd frontend && npm install && npm run dev
```

---

## Project Structure

```
ledger/
├── backend/
│   ├── main.py
│   ├── models/
│   ├── routes/
│   └── database.py
├── frontend/
│   └── src/
│       ├── pages/
│       ├── components/
│       └── api/
├── logo.svg
├── schema.sql
└── README.md
```

---

## Author

**Kumaran Chandrashekar**
B.Tech Computer Science (Data Science), VIT Chennai
