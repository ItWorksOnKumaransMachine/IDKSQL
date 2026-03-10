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

## What is Ledger?

AI experiments leave a trail — configurations, training runs, loss curves, checkpoints, datasets. That trail almost always ends up as a mess of flat files, disconnected logs, and half-remembered scripts.

**Ledger is the structured layer between your experiments and amnesia.**

It captures the complete lifecycle of AI experimentation inside a relational database and exposes it through a clean web interface — so every run is reproducible, every metric is queryable, and no experiment is ever lost.

---

## Why Relational?

| Approach | Problem |
|---|---|
| 📁 Log files | Unstructured. Unsearchable. Lost after a week. |
| 🗒️ Spreadsheets | No relationships. Breaks at scale. |
| 🧠 MLflow / W&B | Heavy. Opinionated. Black boxes. |
| ✅ **Ledger** | Transparent relational schema. Fully queryable. You own the data. |

The core insight: **an AI experiment is a tree of structured facts**, not a stream of text.

---

## Experiment Lifecycle Model

Every AI experiment maps onto Ledger's data model:

```
┌─────────────┐
│    MODEL    │  ←  What architecture are you training?
└──────┬──────┘
       │ has many
┌──────▼──────────┐
│  MODEL_VERSION  │  ←  Which iteration? What hyperparams?
└──────┬──────────┘
       │ belongs to
┌──────▼──────────┐
│   EXPERIMENT    │  ←  What hypothesis are you testing?
└──────┬──────────┘
       │ generates
┌──────▼──────────┐
│  TRAINING_RUN   │  ←  A single training session
└──────┬──────────┘
       │ produces
   ┌───┴──────────────────┐
   │                      │
┌──▼────────────┐  ┌──────▼──────┐
│EXECUTION_STEP │  │  CHECKPOINT │  ←  Progress + Saved states
└──────┬────────┘  └─────────────┘
       │ emits
┌──────▼──────────┐
│  METRIC_SCALAR  │  ←  Loss, accuracy, F1 — anything numeric
└─────────────────┘

       + DATASET ──► RUN_DATASET_MAP  ←  What data was used?
```

Every node is a first-class database record — queryable across the entire history.

---

## System Architecture

```
              ╔══════════════════════════════╗
              ║        React Frontend         ║
              ║   Dashboard · Registry ·      ║
              ║   Metrics Explorer · Timeline ║
              ╚═════════════╤════════════════╝
                            │  REST / JSON
              ╔═════════════▼════════════════╗
              ║       FastAPI Backend          ║
              ║   Route Handlers · ORM ·       ║
              ║   Query Engine                 ║
              ╚═════════════╤════════════════╝
                            │  SQL
              ╔═════════════▼════════════════╗
              ║      PostgreSQL Database       ║
              ║   Models · Experiments ·       ║
              ║   Runs · Metrics · Datasets    ║
              ╚══════════════════════════════╝
```

No external services. No cloud lock-in. Everything runs locally.

---

## Interface Modules

### 📊 Dashboard
High-level snapshot of the database — models, active experiments, training runs, datasets.

### 🗄️ Model Registry
Central catalog of all AI models and version histories. Browse architectures, compare versions, track hyperparameter evolution.

### 🧪 Experiment Manager
Define research hypotheses linked to specific model versions. Each experiment can spawn multiple training runs for side-by-side comparison.

### 🏃 Training Runs
Per-session records — configuration snapshots, status tracking (`queued → running → completed → failed`), timestamps and duration.

### 📈 Metrics Explorer
Chart.js visualizations of training dynamics. Query any metric across any step range — loss curves, accuracy plots, custom scalars.

### 🕐 Execution Timeline
Step-by-step reconstruction of training progress. Identify exactly where training diverged, stalled, or spiked.

### 📦 Dataset Registry
Metadata store for every dataset used in training, with full lineage tracking across experiments and runs.

---

## Query Examples

**Count all registered experiments**
```sql
SELECT COUNT(*) FROM EXPERIMENT;
```

**Rank models by number of training runs**
```sql
SELECT m.name, COUNT(tr.runid) AS total_runs
FROM MODEL m
JOIN MODEL_VERSION mv ON m.modelid = mv.modelid
JOIN TRAINING_RUN tr  ON mv.versionid = tr.versionid
GROUP BY m.name
ORDER BY total_runs DESC;
```

**Reconstruct the loss curve for a specific run**
```sql
SELECT
  es.globalstepnumber  AS step,
  ms.metricvalue       AS loss
FROM EXECUTION_STEP es
JOIN METRIC_SCALAR ms ON es.stepid = ms.stepid
WHERE ms.metricname = 'loss'
  AND es.runid = :run_id
ORDER BY es.globalstepnumber;
```

**Find all runs using a specific dataset**
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
| Frontend | React + Chart.js |
| Backend | Python + FastAPI |
| Database | PostgreSQL |

---

## Getting Started

**Clone**
```bash
git clone https://github.com/yourusername/ledger
cd ledger
```

**Configure the database**
```bash
export DATABASE_URL="postgresql://user:password@localhost:5432/ledger"
psql -d ledger -f schema.sql
```

**Start the backend**
```bash
pip install -r requirements.txt
uvicorn main:app --reload
# → http://localhost:8000
```

**Start the frontend**
```bash
cd frontend
npm install
npm run dev
# → http://localhost:5173
```

---

## Project Structure

```
ledger/
├── backend/
│   ├── main.py              # FastAPI entrypoint
│   ├── models/              # SQLAlchemy ORM models
│   ├── routes/              # API route handlers
│   └── database.py          # DB connection config
├── frontend/
│   ├── src/
│   │   ├── pages/           # Dashboard, Registry, Explorer
│   │   ├── components/      # Reusable UI components
│   │   └── api/             # Backend API calls
│   └── package.json
├── logo.svg                 # Project logo
├── schema.sql               # Full database schema
└── README.md
```

---

## Motivation

Reproducibility is the foundation of science. In AI research, it's also the hardest thing to achieve.

Ledger was built from a simple frustration: **experiment data should not live in your head, your terminal history, or a folder called `final_final_v3`.**

By treating AI experiments as structured, relational data, Ledger enables questions that flat logs simply can't answer — across models, across versions, across time.

---

## Author

**Kumaran Chandrashekar**  
B.Tech Computer Science (Data Science) · VIT Chennai

---

<div align="center">
<br/>
<img src="logo.svg" alt="Ledger" width="220"/>
<br/><br/>
<sub>Structured memory for artificial intelligence experiments.</sub>
<br/><br/>
</div>
