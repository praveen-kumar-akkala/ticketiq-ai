🎫 TicketIQ-AI
### Ticket Intelligence, Analyst Performance & AI-Powered Support Insights

> An end-to-end AI/ML project built on real ServiceNow support data — covering SQL, Python, Machine Learning, Generative AI, and Agentic AI. Designed to be both a **practical tool** for tech support analysts and a **portfolio-grade project** for AI/ML engineering interviews.

---

## 📌 Project Overview

TicketIQ-AI is an intelligent performance and ticket analytics system built for tech support analysts who work on ServiceNow tickets, chats, and calls. It autonomously ingests daily ticket exports, analyzes individual and team performance, detects patterns, flags risks, and generates human-readable feedback and suggestions — all powered by a multi-layer AI pipeline.

This project is intentionally designed to demonstrate hands-on expertise across the full modern AI/ML stack:

| Domain | Technologies |
|---|---|
| **Data & Storage** | SQL (SQLite / PostgreSQL), CSV pipeline |
| **Engineering** | Python, ETL, scheduling, file watching |
| **Machine Learning** | Scikit-learn, Prophet, DBSCAN, K-Means |
| **NLP** | Sentence Transformers, text clustering, embeddings |
| **Generative AI** | OpenAI API / Ollama (local LLMs), prompt engineering |
| **Agentic AI** | LangChain / CrewAI, autonomous multi-step agents |
| **Visualization** | Streamlit dashboard |

---

## 🎯 Goals & Motivation

- Work as a **Senior Tech Support Analyst** handling ServiceNow tickets, chats, and calls across multiple service lines and internal applications
- Pull daily ticket reports manually as CSV exports from ServiceNow (no direct API access initially)
- Tickets are generated **24/7 × 365** and processed in **daily batches**
- Build a system that makes performance visible, patterns actionable, and feedback intelligent
- Cover **SQL → Python → ML → GenAI → Agentic AI** in one cohesive, real-world project
- Become **interview-ready** as an AI/ML Engineer with genuine hands-on experience

---

## 🏗️ Full System Architecture


┌─────────────────────────────────────────────┐
│           DATA INGESTION LAYER              │
│  Manual CSV Export → Watched Folder         │
│  Python: validate, deduplicate, load        │
│  Storage: SQLite (dev) / PostgreSQL (prod)  │
└────────────────────┬────────────────────────┘
↓
┌─────────────────────────────────────────────┐
│              SQL ANALYTICS LAYER            │
│  • Daily/Weekly/Monthly/Quarterly metrics   │
│  • Aging ticket detection & bucketing       │
│  • Escalation flagging                      │
│  • Analyst performance scoring              │
└────────────────────┬────────────────────────┘
↓
┌─────────────────────────────────────────────┐
│               ML LAYER                      │
│  • Ticket similarity clustering (NLP)       │
│  • Anomaly detection (performance dips)     │
│  • Time-to-resolution forecasting           │
│  • Repeat issue pattern detection           │
└────────────────────┬────────────────────────┘
↓
┌─────────────────────────────────────────────┐
│              GenAI LAYER                    │
│  • Human-readable feedback generation       │
│  • Cluster naming & root cause analysis     │
│  • Resolution suggestions for aged /        │
│    escalated / repeated tickets             │
└────────────────────┬────────────────────────┘
↓
┌─────────────────────────────────────────────┐
│           AGENTIC ORCHESTRATION             │
│  LangChain / CrewAI Agent                   │
│  Triggered daily after CSV drop             │
│  Autonomously runs full pipeline            │
│  Outputs: Dashboard + Daily Summary Report  │
└─────────────────────────────────────────────┘
Unknown
---

## 📥 Data Ingestion Layer — CSV Pipeline

### The "Watched Folder" Pattern

Since direct ServiceNow API access is not available, the system uses a **file ingestion layer** — a Python script that watches a folder for new daily CSV exports and automatically triggers the pipeline.


You manually export CSV from ServiceNow
↓
Drop it into /data/incoming/
↓
Python watcher (watchdog library) detects new file
↓
Validates → Cleans → Deduplicates → Loads into SQL database
↓
Rest of pipeline triggers automatically
Unknown
### Why This Approach Is Valid

- Mirrors how **real enterprise pipelines** handle restricted data access
- The ingestion module can be swapped for a REST API call later without changing the rest of the pipeline
- Demonstrates **modular, production-minded design**

### Handling the 24/7 → Daily Batch Reality

| Design Decision | Approach |
|---|---|
| **Daily snapshot** | Each CSV = one day's extract (or cumulative — must decide at schema design time) |
| **Deduplication** | Use `ticket_id` as primary key — skip already-loaded records on every import |
| **Processing timestamp** | Always log ingestion time separately from ticket creation time |
| **Backfilling** | Support loading historical CSVs to seed the database for ML training |

### Future: ServiceNow REST API

When API access becomes available, ServiceNow uses a standard REST API with:
- Instance URL: `https://yourcompany.service-now.com`
- Auth: Basic auth or OAuth 2.0
- Python `requests` library for calls
- The ingestion module gets replaced with a single API polling script — no other changes needed

---

## 🗄️ SQL Analytics Layer

### Core Metrics Tracked

- Ticket volume (daily / weekly / monthly / quarterly)
- Average resolution time
- SLA compliance rate
- Reopened / bounced ticket rate
- First-call resolution (FCR) rate
- Escalation rate
- Aging buckets

### Aging Ticket Detection

```sql
SELECT 
    ticket_id,
    created_date,
    DATEDIFF(CURRENT_DATE, created_date) AS age_in_days,
    priority,
    assigned_analyst,
    CASE 
        WHEN DATEDIFF(CURRENT_DATE, created_date) > 30 THEN 'Critical Aging'
        WHEN DATEDIFF(CURRENT_DATE, created_date) > 14 THEN 'High Aging'
        WHEN DATEDIFF(CURRENT_DATE, created_date) > 7  THEN 'Moderate Aging'
        ELSE 'Normal'
    END AS aging_bucket
FROM tickets
WHERE status NOT IN ('Closed', 'Resolved')
ORDER BY age_in_days DESC;

Temporal Analysis

Window functions for rolling averages
Period-over-period comparisons (this week vs. last week, this month vs. last month)
Quarter-to-date aggregations
Trend direction indicators (↑ ↓ →)


🤖 Machine Learning Layer
1. Performance Scoring Model

Phase 1 (Rule-based): Weighted composite score from key metrics
Phase 2 (ML-based): Use rule-based scores as pseudo-labels → train a regression model
Score range: 0–100 with daily, weekly, monthly, quarterly views

2. Ticket Similarity Clustering — Identical & Repeated Issues
UnknownTicket descriptions (free text)
        ↓
Text preprocessing (lowercase, stopword removal, lemmatization)
        ↓
Embeddings via sentence-transformers or OpenAI embeddings
        ↓
Clustering — K-Means or DBSCAN
        ↓
Each cluster = a "recurring issue type" with frequency count
        ↓
GenAI layer names the cluster and suggests permanent fix

Interview value: NLP + Unsupervised ML + GenAI in one feature.
3. Anomaly Detection

Detect performance dips: flag when a metric drops >X% from personal baseline
Detect unusual ticket spikes by category or service line
Algorithm: Isolation Forest or Z-score based detection

4. Time-to-Resolution Forecasting

Predict how long a new ticket will take to close based on its properties at creation
Flag tickets trending toward SLA breach before it happens
Algorithm: Random Forest Regressor / XGBoost / Facebook Prophet for volume forecasting


🔍 Key Feature Modules
⏳ Aging Tickets

Tickets open beyond SLA thresholds, bucketed by severity
Daily aging report with action flags
Escalation risk scoring for near-breach tickets

🚨 Escalated Incidents
Explicit detection: Use ServiceNow's escalation_flag or escalated_to fields if present
Inferred detection (if no explicit field):

Priority upgraded (Low → High)
Reassignment count > 2
Ticket age + still open
Keywords in notes: "manager involved", "urgent escalation", "executive visibility"

GenAI on escalations: "Summarize the pattern of escalations this week and identify the top 3 root causes."
🔁 Repeated & Identical Issues

NLP clustering groups tickets describing the same problem in different words
Frequency ranking: most common issue types by volume
Trend tracking: is a recurring issue getting worse week over week?

💡 AI-Powered Suggestions
The Agentic layer retrieves similar tickets that were successfully resolved in the past and generates:

Probable root cause
Suggested resolution steps
Whether a knowledge base article should be created
Whether this is a permanent fix vs. workaround situation
Whether the issue needs to be escalated to a service owner


🧠 Generative AI Layer
What the LLM Does

Converts structured score data into natural language feedback
Names ticket clusters with human-readable labels
Generates improvement suggestions based on personal patterns
Writes escalation summaries and root cause narratives
Drafts knowledge base article outlines for recurring issues

Example Prompt Pattern
UnknownAnalyst resolved 23 tickets this week. SLA breach rate was 15%, 
above the 8% team average. Resolution time improved by 18% vs. 
last week. 3 tickets were reopened.

Generate:
1. Constructive performance feedback (2-3 sentences)
2. Three specific, actionable improvement suggestions
3. One positive highlight to acknowledge

LLM Options



Option
Use Case




OpenAI API (GPT-4o)
Best quality, requires API key & cost


Ollama + Llama 3 / Mistral
Free, runs locally, good for dev


Groq API
Free tier, very fast inference




🦾 Agentic AI Layer
What the Agent Does
Triggered automatically each day after a new CSV is detected:

Ingests new ticket data
Runs SQL analytics — computes all metrics
Runs ML models — scores performance, detects anomalies, clusters issues
Calls GenAI — generates feedback, suggestions, summaries
Compiles the daily intelligence report
Outputs to Streamlit dashboard and/or sends a summary notification

Tools / Frameworks

LangChain — agent orchestration, tool use, memory
CrewAI — multi-agent setup (one agent for analytics, one for GenAI, one for reporting)
APScheduler / cron — daily trigger post-CSV drop


📊 Sample Daily Output Report
Unknown📅 Daily Intelligence Report — [DATE]

👤 YOUR PERFORMANCE
  Score: 87/100 | Trend: ↑ +4 from yesterday
  Tickets resolved: 18 | Avg resolution: 2.3 hrs | SLA: 94%

⚠️  AGING TICKETS (Action Required)
  3 tickets > 14 days | 1 ticket > 30 days (Critical)
  → INC0045231: Password reset loop — suggest KB article creation

🔁 REPEATED ISSUES THIS WEEK
  Cluster: "VPN connectivity after Windows update" — 12 tickets
  → Suggested fix: Push GPO update, create self-service guide

🚨 ESCALATIONS
  2 escalated this week | Root cause: Missing access permissions
  → Pattern: All from same service line — flag to service owner

💡 TOP SUGGESTIONS
  1. Create knowledge article for VPN cluster (saves ~4 hrs/week)
  2. INC0045231 needs L2 involvement — recommend escalation
  3. Your Wednesday resolution time spikes — review workload pattern


📅 Development Roadmap
Phase 1 — Data Foundation (SQL + Python)

[ ] Design SQL schema (tickets, analysts, service lines, categories)
[ ] Build CSV ingestion script with validation and deduplication
[ ] Set up watched folder pipeline using watchdog
[ ] Load historical CSVs for backfilling
[ ] Write core SQL queries: aging, escalation flags, basic metrics
[ ] Build daily/weekly/monthly/quarterly aggregation views

Phase 2 — Performance Engine (Python + ML Basics)

[ ] Build composite performance scoring (rule-based first)
[ ] Implement aging bucket logic and SLA breach detection
[ ] Build escalation detection (explicit + inferred)
[ ] Period-over-period trend comparisons
[ ] Streamlit dashboard — basic metrics view

Phase 3 — Machine Learning (Scikit-learn, Prophet)

[ ] Ticket similarity clustering with sentence-transformers + DBSCAN/K-Means
[ ] Anomaly detection for performance dips (Isolation Forest)
[ ] Time-to-resolution prediction model (XGBoost / Random Forest)
[ ] Volume forecasting (Prophet)
[ ] Repeat issue frequency ranking and trend tracking

Phase 4 — Generative AI (LLM Integration)

[ ] Integrate LLM (OpenAI / Ollama)
[ ] Build prompt templates for feedback generation
[ ] Generate resolution suggestions for aged/escalated/repeated tickets
[ ] Cluster naming and root cause narrative generation
[ ] Structured output parsing (JSON responses from LLM)

Phase 5 — Agentic AI (LangChain / CrewAI)

[ ] Build autonomous agent that orchestrates the full pipeline
[ ] Implement agent memory (track trends across multiple days)
[ ] Add multi-agent setup: Analytics Agent + GenAI Agent + Report Agent
[ ] Daily report auto-generation on CSV drop
[ ] Optional: Slack / email notification on report completion


📁 Suggested Repository Structure
Unknownticketiq-ai/
│
├── data/
│   ├── incoming/          # Drop new CSV exports here
│   ├── processed/         # Moved here after ingestion
│   └── sample/            # Anonymized sample data for demo
│
├── db/
│   └── schema.sql         # Full SQL schema definition
│
├── ingestion/
│   ├── watcher.py         # Folder watcher using watchdog
│   ├── validator.py       # CSV validation logic
│   └── loader.py          # Dedup + DB load logic
│
├── analytics/
│   ├── metrics.py         # Core metric calculations
│   ├── aging.py           # Aging ticket detection
│   ├── escalations.py     # Escalation detection logic
│   └── scoring.py         # Performance scoring engine
│
├── ml/
│   ├── clustering.py      # Ticket similarity clustering
│   ├── anomaly.py         # Anomaly detection
│   ├── forecasting.py     # Time-to-resolution + volume forecasting
│   └── models/            # Saved model artifacts
│
├── genai/
│   ├── feedback.py        # Performance feedback generator
│   ├── suggestions.py     # Resolution suggestion generator
│   ├── summaries.py       # Escalation & cluster summaries
│   └── prompts/           # Prompt templates
│
├── agent/
│   ├── orchestrator.py    # Main agent controller
│   ├── tools.py           # Agent tool definitions
│   └── memory.py          # Agent memory/context management
│
├── dashboard/
│   └── app.py             # Streamlit dashboard
│
├── reports/               # Auto-generated daily reports
│
├── notebooks/             # Jupyter notebooks for exploration
│
├── tests/                 # Unit tests
│
├── requirements.txt
├── .env.example
├── config.yaml
└── README.md


🛠️ Tech Stack
UnknownLanguage        : Python 3.10+
Database        : SQLite (dev) → PostgreSQL (prod)
ML Libraries    : scikit-learn, XGBoost, Facebook Prophet
NLP             : sentence-transformers, NLTK / spaCy
GenAI           : OpenAI API / Ollama (Llama 3 / Mistral)
Agentic AI      : LangChain / CrewAI
Dashboard       : Streamlit
File Watching   : watchdog
Scheduling      : APScheduler
Version Control : Git + GitHub
Environment     : python-dotenv, conda / venv


⚠️ Important Design Notes
Data Privacy

Ticket data may contain user/client PII
Always anonymize real data before committing to GitHub
Use the data/sample/ folder for synthetic/anonymized demo data only
Never commit .env files or credentials

Scalability Thinking

Built for a single analyst first — designed to scale to full team
Multi-analyst support makes this a team performance intelligence tool, not just a personal tracker
All queries and models are parameterized by analyst_id

Modular Design

Each layer (ingestion, SQL, ML, GenAI, agent) is independently replaceable
The CSV ingestion module can be swapped for a REST API call with zero changes to downstream layers
This demonstrates production-grade architectural thinking


🎓 Interview Talking Points This Project Covers



Topic
Where You Demonstrate It




SQL window functions, CTEs
Metric aggregations, period-over-period comparisons


Python ETL pipeline
CSV ingestion, validation, deduplication


Feature engineering
Composite scoring, aging buckets, escalation signals


Unsupervised ML
Ticket clustering with DBSCAN / K-Means


Supervised ML
Resolution time prediction, scoring model


Time series forecasting
Volume prediction with Prophet


Anomaly detection
Isolation Forest for performance dips


NLP & embeddings
Sentence transformers for ticket similarity


Prompt engineering
Structured LLM feedback with JSON output


LLM integration
OpenAI / Ollama API calls, output parsing


Agentic AI
Multi-step autonomous agent with tools & memory


System design
Modular pipeline, swap-in API, batch vs. streaming


Real-world constraints
No API access, manual exports, enterprise limitations




🚀 Getting Started
Bash# Clone the repo
git clone https://github.com/yourusername/ticketiq-ai.git
cd ticketiq-ai

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env with your LLM API key and DB settings

# Initialize the database
python db/init_db.py

# Drop a sample CSV and run the pipeline
cp data/sample/sample_tickets.csv data/incoming/
python ingestion/watcher.py


📄 License
MIT License — feel free to adapt this project for your own portfolio.

🙋 Author
Built by a Senior Tech Support Analyst transitioning into AI/ML Engineering.
This project represents real data, real problems, and a real path to mastery.

"The best portfolio projects solve problems you actually face."
Unknown
---

