# FoodLence Backend - AI & Allergen Detection Engine

FoodLence Backend is a high-performance Python FastAPI service providing AI/NLP-driven food ingredient parsing, semantic allergen matching, additive identification, and personalized dietary profile evaluation for the FoodLence mobile application.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Architecture & Processing Pipeline](#architecture--processing-pipeline)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Module Descriptions](#module-descriptions)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Local Setup](#local-setup)
  - [Docker Setup](#docker-setup)
- [API Documentation](#api-documentation)
- [Team Details & Module Ownership](#team-details--module-ownership)
- [Supervision & Mentorship](#supervision--mentorship)
- [Development Workflow](#development-workflow)

---

## Project Overview

Food label analysis requires processing complex scientific synonyms, varying regional naming conventions, and chemical additives (E-numbers / INS numbers). The FoodLence Backend provides:
1. **Semantic Allergen Matching**: Mapping alternative ingredient names and synonyms (e.g., casein, whey, ovalbumin) to core allergen categories.
2. **Additive & Chemical Intelligence**: Parsing and categorizing preservatives, artificial colorings, emulsifiers, and sweeteners.
3. **Personalized Health Evaluation**: Cross-referencing detected ingredients against user and family allergy profiles to calculate danger levels (Safe, Caution, Warning).
4. **Confidence Scoring**: Scoring extraction certainty and semantic distance to inform the user when manual inspection is recommended.

---

## Architecture & Processing Pipeline

```
[Mobile OCR Result] 
       │
       ▼
[FastAPI REST API (/api/v1/analyze)]
       │
       ├─► [NLP Cleaning & Tokenization] (spaCy / Regex)
       │
       ├─► [Exact Matching] (Knowledge Base SQL / Direct Lookup)
       │
       ├─► [Semantic & Fuzzy Matching] (RapidFuzz / Embeddings)
       │
       ├─► [Allergen & Additive Evaluation Engine]
       │         ▲
       │         │ (User / Family Profiles)
       │
       ├─► [Confidence Scoring Engine]
       │
       ▼
[Structured Response (Allergens, Additives, Safety Rating, Confidence)]
```

---

## Tech Stack

* **Framework**: Python 3.10+, FastAPI
* **NLP & AI Engine**: spaCy, scikit-learn, RapidFuzz / FuzzyWuzzy
* **Database**: PostgreSQL (SQLAlchemy ORM + Alembic migrations)
* **Authentication**: JWT (python-jose, passlib[bcrypt])
* **Validation & Settings**: Pydantic v2, Pydantic-Settings
* **Testing**: pytest, pytest-asyncio
* **Containerization**: Docker, Docker Compose

---

## Repository Structure

```text
FoodLence-BackEnd/
├── app/
│   ├── api/                     # API routing and controllers
│   │   └── v1/
│   │       ├── endpoints/
│   │       │   ├── analyze.py      # Label text analysis & allergen evaluation
│   │       │   ├── auth.py         # User registration & JWT authentication
│   │       │   ├── history.py      # Scan history and audit logs
│   │       │   ├── ingredients.py  # Ingredient & additive knowledge lookup
│   │       │   └── profiles.py     # User and family allergy profiles
│   │       ├── deps.py             # Dependency injection (DB session, current user)
│   │       └── router.py           # Combined API v1 router
│   ├── core/                    # Core configuration and security
│   │   ├── config.py               # Pydantic BaseSettings environment loader
│   │   ├── exceptions.py           # Custom exception definitions & error handlers
│   │   └── security.py             # Password hashing (bcrypt) & JWT token handlers
│   ├── db/                      # Database configuration & migrations
│   │   ├── base.py                 # Declarative Base metadata
│   │   ├── session.py              # SQLAlchemy engine & session maker
│   │   └── migrations/             # Alembic migration scripts
│   ├── knowledge_base/          # Knowledge base & rule-based engine
│   │   ├── allergen_engine.py      # Allergen risk & cross-profile evaluation
│   │   ├── exact_matcher.py        # Direct string & E-number matching
│   │   ├── semantic_matcher.py     # Fuzzy & semantic similarity matching
│   │   └── verification.py         # Confidence check & rule validation
│   ├── models/                  # SQLAlchemy ORM database models
│   │   ├── additive.py             # Additive and chemical entities
│   │   ├── allergen.py             # Allergen categories and severities
│   │   ├── inference_log.py        # Model inference & scan history logs
│   │   ├── ingredient.py           # Ingredient entities and taxonomy
│   │   ├── ingredient_allergen_mapping.py # Cross-references and mappings
│   │   ├── ingredient_embedding.py # Vector/numerical embeddings store
│   │   ├── profile.py              # User & family member allergy profiles
│   │   └── user.py                 # User account credentials & preferences
│   ├── nlp/                     # NLP processing & inference pipeline
│   │   ├── classifier.py           # Classification routines
│   │   ├── confidence.py           # Confidence scoring algorithm
│   │   ├── model_loader.py         # spaCy / ML model loader & warm-up
│   │   └── pipeline.py             # End-to-end NLP ingestion pipeline
│   ├── schemas/                 # Pydantic request and response schemas
│   │   ├── analyze.py              # Schemas for scan analysis requests/responses
│   │   ├── auth.py                 # Schemas for login, token, user registration
│   │   ├── history.py              # Schemas for scan history records
│   │   ├── ingredient.py           # Schemas for ingredient queries & details
│   │   └── profile.py              # Schemas for family allergy profiles
│   ├── services/                # Business logic services
│   │   ├── embedding_service.py    # Embedding generation and lookup
│   │   ├── history_service.py      # Scan history management
│   │   ├── profile_service.py      # Profile management logic
│   │   ├── scan_service.py         # End-to-end scan orchestrator
│   │   └── user_service.py         # User account management
│   └── main.py                  # FastAPI application entrypoint
├── knowledge_base_seed/         # Initial dataset seeds
│   ├── seed_additives.sql          # Seed SQL data for food additives & E-numbers
│   ├── seed_allergens.sql          # Seed SQL data for allergen taxonomy
│   ├── seed_embeddings.py          # Script to generate embeddings for seed data
│   ├── seed_ingredients.sql        # Seed SQL data for common ingredients
│   └── seed_mappings.sql           # Seed SQL data for ingredient-allergen relations
├── ml/                          # ML training & dataset storage
│   ├── data/
│   │   ├── annotated/              # Labeled ingredient datasets
│   │   ├── raw/                    # Raw ingredient strings & packaging dumps
│   │   └── test/                   # Benchmark evaluation sets
│   ├── models/                     # Trained ML weights / serialized artifacts
│   ├── notebooks/                  # Model research and EDA Jupyter notebooks
│   └── reports/                    # Training metrics, confusion matrices & curves
├── tests/                       # Pytest test suite
│   ├── conftest.py                 # Test fixtures, DB mock, test client setup
│   ├── test_allergen_engine.py     # Unit tests for allergen evaluation
│   ├── test_analyze.py             # Integration tests for analysis endpoints
│   ├── test_auth.py                # Tests for JWT auth and user flows
│   ├── test_knowledge_base.py      # Tests for knowledge base matching
│   ├── test_nlp_pipeline.py        # Tests for NLP pipeline & tokenizers
│   └── test_profiles.py            # Tests for multi-profile management
├── .env.example                 # Example environment variables
├── .gitignore                   # Git ignore specifications
├── Dockerfile                   # Docker container build script
├── docker-compose.yml           # Multi-container orchestration (FastAPI + PostgreSQL)
└── requirements.txt             # Python dependencies
```

---

## Getting Started

### Prerequisites
* Python 3.10+
* PostgreSQL 14+ (or Docker)
* Git

### Local Setup
1. **Navigate to the backend directory**:
   ```bash
   cd FoodLence-BackEnd
   ```

2. **Create and activate a virtual environment**:
   * Windows (PowerShell):
     ```powershell
     python -m venv venv
     .\venv\Scripts\Activate.ps1
     ```
   * macOS / Linux:
     ```bash
     python3 -m venv venv
     source venv/bin/activate
     ```

3. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   python -m spacy download en_core_web_sm
   ```

4. **Configure environment variables**:
   ```bash
   cp .env.example .env
   ```
   *Edit `.env` to configure your PostgreSQL credentials and secret key.*

5. **Start the FastAPI server**:
   ```bash
   uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
   ```

### Docker Setup
To run the complete stack (FastAPI server + PostgreSQL):
```bash
docker-compose up --build
```

---

## API Documentation

When the application is running, interactive API documentation is available at:
* **Swagger UI**: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
* **ReDoc**: [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

---

## Team Details & Module Ownership

| Name | Index / Reg No | Role & Module Ownership |
| :--- | :--- | :--- |
| **H.G.K Rashmika** | `21CSE0190` | **AI & Semantic Engine Lead**: NLP Pipeline, Semantic Matching, Ingredient Classification, AI Explanations, Confidence Scoring (`app/nlp/`, `app/knowledge_base/semantic_matcher.py`, `ml/`) |
| **S.K.P.H. Deemantha** | `22CSE0348` | **Knowledge Base & Detection Lead**: Knowledge Base Schema, Allergen/Additive Detection, E-Number Mapping, Product Comparison (`app/knowledge_base/`, `knowledge_base_seed/`, `app/models/additive.py`, `app/models/allergen.py`) |
| **P.A.A. Ishara Dilshan** | `22CSE0360` | **Platform, OCR & Profile Lead**: User & Family Profiles, Personalized Warning Engine, OCR Pipeline Integration, Dockerized Services & API Infrastructure (`app/api/`, `app/core/`, `app/db/`, `app/services/`, `Dockerfile`, `docker-compose.yml`) |
| **H.G.B.A Mekala** | `22CSE0361` | **Mobile & UI Lead**: Mobile App UI, Scanner Interface, Analysis Dashboard, Scan History, Mobile Client Integration |

---

## Supervision & Mentorship

* **Academic Institution**: Department of Software Engineering, Faculty of Computing, Sabaragamuwa University of Sri Lanka
* **Course Unit**: SE5104 Mini Project
* **Internal Supervisor**: Ms. B.R. Madhurangi (Lecturer)
* **Industry Mentor**: Mr. G.A.C.G. Dhanawardhana (Associate Software Engineer, Surecore (Pvt) Ltd)

---

## Development Workflow

* Branching Model: GitHub Flow (`main`, `feature/<feature-name>`, `fix/<bug-name>`).
* Formatters & Linters: `black`, `ruff`.
* Automated Testing: Run `pytest` prior to submitting pull requests.
