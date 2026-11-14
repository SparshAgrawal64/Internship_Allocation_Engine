# AI-Based Smart Allocation Engine for PM Internship Scheme

## Overview

SmartAlloc is an AI-based internship allocation engine for the PM Internship Scheme. It combines rule-based eligibility filtering, fuzzy-logic fit scoring, machine-learning acceptance prediction, and global optimization using Google OR-Tools CP-SAT. The project also includes dynamic reallocation and a decision-explainability layer.

The project now has a working prototype architecture:

```text
React + Vite Frontend
        |
        | HTTP/JSON
        v
FastAPI Backend
        |
        | runs existing Python modules
        v
Python Allocation Engine
        |
        +--> Eligibility filtering
        +--> Fuzzy scoring
        +--> ML acceptance prediction
        +--> OR-Tools global optimization
        +--> final_allocation.csv
        |
        v
FastAPI returns results
        |
        v
React dashboard displays allocations
```

The current prototype uses synthetic/demo data. PostgreSQL, authentication/RBAC, cloud deployment, and production security are not part of the current prototype.

---

## How the allocation engine works

### 1. Eligibility Filter

Hard constraints are applied to candidate-internship pairs:

- Minimum qualification score
- Candidate location preference must match internship location
- At least one required skill must overlap

### 2. Fuzzy Scoring

For every eligible pair, the engine calculates a 0–100 fuzzy fit score using:

- Skill match percentage
- Qualification score

This uses `scikit-fuzzy` rather than a simple flat weighted average.

### 3. Acceptance Prediction

A scikit-learn Logistic Regression model predicts the probability that a candidate will accept an internship offer.

The current model is trained on synthetic/simulated data because historical PM Internship Scheme acceptance data is not available.

### 4. Global Optimization

Google OR-Tools CP-SAT solves the allocation globally while enforcing:

- At most one internship per candidate
- Internship capacity limits
- Reserved-category restrictions

The objective maximizes:

```text
fit_score × acceptance_probability
```

### 5. Dynamic Reallocation

If an allocated candidate declines an offer, the Python reallocation module can identify the vacated internship and search for an eligible replacement while preserving unrelated allocations and constraints.

### 6. Explainability

The explainability module provides evidence-based information about eligibility, skill matching, fuzzy fit, acceptance probability, allocation status, capacity, reservation constraints, and reallocation history.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite |
| Frontend icons | Lucide React |
| Frontend language | JavaScript |
| Backend API | FastAPI |
| API server | Uvicorn |
| Data handling | pandas, numpy |
| Eligibility | Python rule-based logic |
| Fuzzy scoring | scikit-fuzzy |
| Acceptance prediction | scikit-learn Logistic Regression |
| Optimization | Google OR-Tools CP-SAT |
| Data storage | CSV files for prototype |
| Database | PostgreSQL - planned |
| Authentication/RBAC | Planned |
| Deployment | Planned |

---

## Project Structure

```text
SIH_MINI/
│
├── backend/
│   ├── __init__.py
│   └── app.py                 # FastAPI bridge to Python engine
│
├── demoData/
│   ├── candidates.csv
│   ├── internships.csv
│   ├── eligible_pairs.csv
│   ├── scored_pairs.csv
│   ├── scored_pairs_with_probability.csv
│   ├── final_allocation.csv
│   └── allocation_audit.csv
│
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       └── index.css
│
├── src/
│   ├── generate_data.py
│   ├── eligibility.py
│   ├── fuzzy_scoring.py
│   ├── train_acceptance_model.py
│   ├── optimizer.py
│   ├── reallocation.py
│   ├── explainability.py
│   └── acceptance_model.pkl
│
├── requirements.txt
└── README.md
```

---

# Setup and Running the Final Working Prototype

## Prerequisites

Install:

- Python 3.10+ recommended
- Node.js 18+ recommended
- npm

Check installations:

```bash
python --version
node --version
npm --version
```

---

## Step 1 - Create Python virtual environment

From the project root:

### Windows PowerShell

```powershell
python -m venv venv
venv\Scripts\activate
```

### macOS/Linux

```bash
python3 -m venv venv
source venv/bin/activate
```

---

## Step 2 - Install Python dependencies

From the project root:

```bash
pip install -r requirements.txt
```

This installs the Python engine dependencies as well as FastAPI and Uvicorn.

---

# Step 3 - Start the FastAPI backend

**Keep this terminal open.**

From the project root:

```bash
python -m uvicorn backend.app:app --reload --port 8000
```

The backend will be available at:

```text
http://127.0.0.1:8000
```

Health check:

```text
http://127.0.0.1:8000/api/health
```

You should receive a response similar to:

```json
{
  "status": "ok",
  "service": "SmartAlloc API"
}
```

FastAPI documentation is also available at:

```text
http://127.0.0.1:8000/docs
```

---

# Step 4 - Start the React frontend

Open a **second terminal**.

```powershell
cd frontend
npm install
npm run dev
```

The frontend will normally be available at:

```text
http://localhost:5173
```

---

# Final Prototype: Two Terminals

The final prototype requires both processes to be running:

### Terminal 1 - Backend

```powershell
venv\Scripts\activate
python -m uvicorn backend.app:app --reload --port 8000
```

### Terminal 2 - Frontend

```powershell
cd frontend
npm run dev
```

Then open:

```text
http://localhost:5173
```

---

# How the Frontend Connects to the Python Engine

The React frontend does not directly import Python code.

Instead:

```text
React
  |
  | fetch()
  v
FastAPI
  |
  | subprocess
  v
Python scripts
  |
  v
CSV outputs
  |
  v
FastAPI JSON response
  |
  v
React UI
```

When **Run Smart Allocation** is clicked in the React dashboard, the frontend sends:

```http
POST /api/allocation/run
```

FastAPI then executes the existing Python pipeline in this order:

```text
generate_data.py
        ↓
eligibility.py
        ↓
fuzzy_scoring.py
        ↓
train_acceptance_model.py
        ↓
optimizer.py
```

The backend then reads:

```text
demoData/final_allocation.csv
```

and returns the allocation results as JSON to React.

Therefore, the allocation displayed in the **Results** page comes from the actual Python engine and OR-Tools optimizer rather than a hard-coded frontend simulation.

---

# Frontend API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/health` | Check backend status |
| GET | `/api/candidates` | Get candidate data |
| GET | `/api/opportunities` | Get internship data |
| GET | `/api/results` | Get current allocation |
| POST | `/api/allocation/run` | Execute the Python allocation pipeline |

---

# Testing the Complete Prototype

After starting both backend and frontend:

1. Open `http://localhost:5173`.
2. Confirm the dashboard loads.
3. Open **Candidates** and verify candidate data is loaded from the backend.
4. Open **Opportunities** and verify internship data is loaded from the backend.
5. Open **Smart Allocation**.
6. Click **Run Smart Allocation**.
7. Wait while the backend executes the Python pipeline.
8. Open **Results**.
9. Verify that the displayed allocations correspond to `demoData/final_allocation.csv`.
10. Open **Analytics** to view the current allocation statistics.

---

# Testing the Python Engine Separately

The Python engine can still be tested without the frontend.

From the project root:

```powershell
cd src
python generate_data.py
python eligibility.py
python fuzzy_scoring.py
python train_acceptance_model.py
python optimizer.py
```

Each stage reads the previous stage's CSV output and writes the next output into `demoData/`.

The final allocation is written to:

```text
demoData/final_allocation.csv
```

---

# Dynamic Reallocation

After an allocation has been generated, the existing Python reallocation module can handle a candidate declining an offer:

```powershell
cd src
python reallocation.py <candidate_id>
```

Example:

```powershell
python reallocation.py C0001
```

The module validates the updated allocation before writing it back to `demoData/final_allocation.csv` and records the event in `demoData/allocation_audit.csv`.

---

# Explainability

Candidate-level explanation:

```powershell
cd src
python explainability.py <candidate_id>
```

Candidate-internship explanation:

```powershell
python explainability.py <candidate_id> <internship_id>
```

The explainability layer is read-only and does not change allocation decisions.

---

# Frontend Production Build Test

To verify that the React frontend can be built successfully:

```powershell
cd frontend
npm run build
```

A successful build creates the Vite production output in:

```text
frontend/dist/
```

---

# Prototype Limitations

The current prototype intentionally uses:

- Synthetic candidate and internship data
- CSV files instead of a production database
- Synthetic labels for acceptance-model training
- Local FastAPI backend
- Local React frontend

The following are planned for a production implementation:

1. PostgreSQL database
2. Persistent backend data management
3. Authentication and RBAC
4. Production security and encryption
5. Production deployment
6. Real historical acceptance data for model training
7. Additional analytics and reporting

---

# Project Status

### Implemented

- Rule-based eligibility filtering
- Fuzzy fit scoring
- ML acceptance prediction
- Global OR-Tools CP-SAT allocation
- Dynamic reallocation
- Decision explainability
- Synthetic data generation
- FastAPI backend bridge
- React/Vite frontend dashboard
- Frontend-to-Python allocation integration

### Planned

- PostgreSQL integration
- Production authentication/RBAC
- Production deployment
- Real historical scheme data
- Advanced analytics/reporting

---

## Demo Architecture Summary

```text
             ┌─────────────────────────┐
             │      React Frontend     │
             │       localhost:5173    │
             └────────────┬────────────┘
                          │
                    HTTP / JSON
                          │
             ┌────────────▼────────────┐
             │       FastAPI API       │
             │       port 8000         │
             └────────────┬────────────┘
                          │
                    subprocess
                          │
             ┌────────────▼────────────┐
             │    Python AI Engine     │
             │                          │
             │ Eligibility              │
             │ Fuzzy Scoring             │
             │ ML Prediction             │
             │ OR-Tools Optimization     │
             │ Reallocation              │
             │ Explainability            │
             └────────────┬────────────┘
                          │
                     CSV outputs
                          │
             ┌────────────▼────────────┐
             │        demoData/        │
             │ final_allocation.csv    │
             └─────────────────────────┘
```

The browser UI and Python engine are now connected through FastAPI, while the existing Python allocation logic remains unchanged.
