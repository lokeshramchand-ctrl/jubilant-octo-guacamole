

# SupplySentinel

**SupplySentinel** is a lightweight AI agent system that proactively detects and responds to supply-chain risks by analyzing operational events, making structured decisions, executing actions, and providing full observability.

This project is designed to demonstrate how **production-style AI agent backends** are built: with guardrails, structured outputs, tool orchestration, auditability, and clean system architecture.

---

## Quick Start

### Backend Setup
```bash
cd backend
pip install -r requirements.txt
export OPENAI_API_KEY=your_api_key_here
python main.py
```

### Seed Database (Optional)
```bash
cd backend
python seed_data.py          # Add sample supply orders
python seed_data.py clear   # Clear all data
```

### Frontend Setup
```bash
cd frontend
npm install
npm run dev
```

## Services & Access
- **Backend API**: `http://localhost:8000` (FastAPI)
- **Frontend UI**: `http://localhost:3000` (Next.js)
- **Database**: SQLite at `backend/storage/agent_logs.db`

## Problem

In many operations teams (supply chain, finance ops, logistics, procurement), critical signals like:

* “Shipment delayed”
* “Supplier cancelled”
* “Delivery pending”
* “Order slightly late”

arrive via dashboards, emails, spreadsheets, or messages.

Typical problems:

* Humans react too late
* No consistent prioritization
* No audit trail of decisions
* No systematic automation

Teams become reactive instead of proactive.

---

## Solution

SupplySentinel acts as an **AI-powered operations assistant**.

It:

* Receives operational events (order updates)
* Analyzes risk using guardrails + LLM reasoning
* Decides what action to take (LOG, NOTIFY, ESCALATE)
* Executes tools (e.g. notify ops team)
* Logs every decision for auditability
* Exposes a history API and dashboard for observability

This mirrors how real-world agent systems are designed in modern startups.

---

## Core Capabilities

* **AI Agent Reasoning**

  * Local LLM (Ollama) integration
  * Structured JSON outputs
  * Retry + schema validation (Pydantic + Tenacity)

* **Guardrails & Reliability**

  * Config-based thresholds (keywords for HIGH/MEDIUM risk)
  * Deterministic fallbacks before trusting LLM

* **Tool Orchestration**

  * Agent selects actions like:

    * `notify_ops_team`
    * `log_risk_event`

* **Observability**

  * Every agent run logs:

    * Input
    * Reasoning
    * Decision
    * Tools used
    * Timestamp
  * Stored in SQLite
  * Exposed via `/agent/history` API

* **Full-stack Demo**

  * Backend: FastAPI agent system
  * Frontend: Next.js dashboard to visualize decisions
  * Dockerized backend for deployment

---

## Architecture

### System Diagram

![SupplySentinel Architecture Diagram](image.png)

### High-level Flow

```
User / System
     ↓
POST /events/order-update
     ↓
RiskAgent
  ├─ Guardrails (config-based rules)
  ├─ LLM reasoning (Ollama)
  ├─ Structured validation (Pydantic)
  ├─ Retry on failure (Tenacity)
  └─ Tool execution
     ↓
Observability Logger (SQLite)
     ↓
/agent/history endpoint
     ↓
Frontend Dashboard (Next.js)
```

This architecture emphasizes:

* Reliability over raw autonomy
* Clear system boundaries
* Auditability
* Production realism

---

## Project Structure

```
suppliesentinel/
├── backend/
│   ├── agents/
│   │   ├── risk_agent.py
│   │   └── local_llm_client.py
│   ├── api/
│   │   └── routes.py
│   ├── config/
│   │   └── settings.py
│   ├── models/
│   │   └── schemas.py
│   ├── observability/
│   │   └── logger.py
│   ├── tools/
│   │   └── actions.py
│   ├── prompts/
│   │   └── system_prompt.txt
│   ├── main.py
│   └── Dockerfile
│
├── frontend/
│   ├── app/
│   │   ├── page.tsx
│   │   └── submit/page.tsx
│   ├── components/
│   └── lib/
│
└── README.md
```

---

## Tech Stack

**Backend**

* Python 3.11
* FastAPI
* Pydantic
* SQLite
* Tenacity (retries)
* Ollama (local LLM)
* Docker

**Frontend**

* Next.js
* React + TypeScript
* Tailwind CSS

This stack mirrors what many YC-stage startups use.

---

## Running Locally

### 1. Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Start Ollama (in separate terminal):

```bash
ollama run mistral
```

Run API:

```bash
uvicorn main:app --reload
```

Open API docs:

```
http://127.0.0.1:8000/docs
```

---

### 2. Frontend

```bash
cd frontend
npm install
npm run dev
```

Open dashboard:

```
http://localhost:3000
```

---

## Example API Call

```bash
curl -X POST http://127.0.0.1:8000/events/order-update \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": "ORD124",
    "supplier": "SteelCorp",
    "expected_delivery": "2026-01-20",
    "current_status": "Slightly late"
  }'
```

Example response:

```json
{
  "risk_level": "MEDIUM",
  "decision": "NOTIFY",
  "reason": "Matched medium-risk keywords in status",
  "action_taken": "none",
  "timestamp": "2026-01-21T19:40:11"
}
```

---

## Agent History

You can inspect all previous decisions:

```
GET /agent/history
```

This returns:

* Input event
* Agent reasoning
* Decision
* Tools used
* Timestamp

This provides **agent observability and auditability**, which is critical for real-world AI systems.

---

## Design Tradeoffs

This project intentionally prioritizes:

* Reliability over raw autonomy
* Deterministic guardrails before LLM usage
* Structured outputs over free-form generation
* Auditability over black-box behavior

If extended further, improvements would include:

* Real notification integrations (Slack, email, webhooks)
* Role-based access
* More advanced risk models
* Persistent database (Postgres)
* Streaming agent observability

---

## Failure Modes & Safeguards

This system explicitly handles common agent failure cases:

- LLM returns invalid JSON → retried with schema-enforced prompt
- LLM hallucination → deterministic guardrails override
- Tool execution failure → logged and downgraded to LOG action
- Ambiguous input → default MEDIUM risk with human review

This mirrors real-world constraints where agents must fail safely.

---

## Why this project exists

This project was built to demonstrate:

* How AI agents can be engineered for production use
* How to combine guardrails + LLM reasoning safely
* How to design backend architecture for operational AI systems
* How observability and reliability should be treated as first-class features

It is not a toy chatbot — it is a **miniature, real-world agent system**.

---

## How This Would Extend in a Real System

If extended for a production supply-chain platform like Lumari:

- Replace SQLite with Postgres + event indexing
- Integrate Slack / email / webhook tools
- Add multi-tenant agent isolation
- Introduce agent confidence scoring
- Stream agent decisions to observability tools

This project intentionally stops before those layers.

---

## Author

Built by **Lokesh**

Focused on full-stack engineering and AI agent systems.

