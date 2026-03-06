# 🏠 DVF+ La Réunion — Intelligence Foncière

> End-to-end data pipeline analyzing 10 years of real estate and agricultural land transactions in La Réunion (French overseas department 974), enriched with agricultural parcels, natural hazards, and building data.

**DE Zoomcamp 2026 — Final Project** | Kalou (Mangaman Mouny Wilson)

---

## 📋 Table of Contents

- [Problem Description](#-problem-description)
- [Dataset](#-dataset)
- [Architecture](#-architecture)
- [Technologies](#-technologies)
- [Dashboard](#-dashboard)
- [AI Chat (Text-to-SQL)](#-ai-chat-text-to-sql)
- [Data Transformations (dbt)](#-data-transformations-dbt)
- [Pipeline Orchestration](#-pipeline-orchestration)
- [Reproducibility](#-reproducibility)
- [Project Structure](#-project-structure)
- [What I Learned](#-what-i-learned)

---

## 🎯 Problem Description

### Context

La Réunion is a French island in the Indian Ocean with extreme land pressure: a growing population (~900,000) on a limited territory where only ~42,000 hectares are farmable. The price gap between agricultural and residential land reaches **1 to 800** — making land management a critical public policy issue.

**SAFER Réunion** (Société d'Aménagement Foncier et d'Établissement Rural) is the public agency responsible for regulating the agricultural land market. They process ~2,000 notary notifications per year and manage 37 agricultural land groups covering 2,700+ hectares.

### The Problem

SAFER currently relies on **Vigifoncier** (a national tool) for real-time sale monitoring, but has **no analytical intelligence layer**:
- No historical trend analysis across 10 years of transactions
- No cross-referencing with agricultural data (crop types, organic farming)
- No quantified impact of natural hazards (cyclones, volcanic activity, landslides) on prices
- Reports for the technical committee (12 meetings/year) are built manually in PowerPoint
- Price evaluation for pre-emption decisions relies on field experience, not data

### The Solution

**DVF+ La Réunion** is a data product that transforms open government data into an interactive decision-support tool:

1. **Batch pipeline** ingesting DVF+ (Cerema) transaction data, enriched with RPG (agricultural parcels), Géorisques (natural hazards), and BDNB (building data)
2. **dbt transformations** building a Kimball dimensional model (fact table + 5 dimensions)
3. **Interactive dashboards** (Metabase) with choropleth maps of all 24 municipalities
4. **AI-powered chat** (Streamlit + DeepSeek) allowing natural language queries in French → SQL

> "Vigifoncier tells you WHAT is being sold today. DVF+ tells you WHY prices are changing."

---

## 📊 Dataset

### Primary Source — DVF+ Open Data

| Attribute | Detail |
|-----------|--------|
| **Publisher** | Cerema (Centre d'études et d'expertise sur les risques, l'environnement, la mobilité et l'aménagement) |
| **Content** | All real estate transactions in France (sales by notarial deed) |
| **Period** | January 2014 — December 2024 |
| **Volume (dept. 974)** | ~60,000 transactions (5,000–7,000/year) |
| **Format** | PostgreSQL/PostGIS SQL dump (17 relational tables) |
| **Update frequency** | Twice per year (April + October) |
| **License** | Licence Ouverte v2 (French open data license) |
| **URL** | [datafoncier.cerema.fr](https://datafoncier.cerema.fr/donnees/autres-donnees-foncieres/dvfplus-open-data) |

### Enrichment Sources

| Source | Publisher | Content | Join Key |
|--------|-----------|---------|----------|
| **RPG** (Registre Parcellaire Graphique) | IGN/ASP | Agricultural parcels declared under EU Common Agricultural Policy — crop type, area | Geospatial intersection (PostGIS) |
| **Géorisques** | BRGM | Natural hazards: flooding, seismic, volcanic, landslide zones | INSEE municipal code or GPS coordinates |
| **BDNB** (Base de Données Nationale des Bâtiments) | CSTB | National building database — 400+ attributes per building | Building group ID via cadastral parcel |
| **GeoJSON 974** | IGN | Municipal boundaries for all 24 communes of La Réunion | INSEE code |

### Key Data Characteristics

- **Geographic projection**: EPSG:2975 (RGR92 / UTM zone 40S) — specific to La Réunion, NOT the mainland Lambert-93
- **Transaction types**: Sales only (no inheritances or donations)
- **Agricultural transactions**: Identified by cross-referencing `sbati=0`, `sterr>0`, and RPG spatial overlay
- **Known gaps**: DPE (energy performance) data is sparse in overseas territories
- **RGPD**: DVF contains real transaction data. While no personal names are included, address + price + date combinations could allow re-identification. Data is never indexed by search engines and individual addresses are not exposed without protection.

> 📄 For the complete list of all data sources (20+ datasets), download URLs, join keys, and coverage notes, see [`docs/DATA_SOURCES.md`](docs/DATA_SOURCES.md).

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                             │
│  DVF+ (Cerema)  │  RPG (IGN)  │  Géorisques (BRGM)  │  BDNB  │
└────────┬────────┴──────┬──────┴──────────┬───────────┴────┬────┘
         │               │                 │                │
         ▼               ▼                 ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    INGESTION (Python)                            │
│  download_dvf.py  │  load_rpg.py  │  load_georisques.py  │ ... │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│               POSTGRESQL + POSTGIS (Data Warehouse)             │
│                                                                 │
│  ┌──────────┐    ┌──────────────┐    ┌───────────────────────┐  │
│  │ raw      │───▶│ staging (dbt)│───▶│ marts (dbt)           │  │
│  │ schema   │    │ stg_dvf__*   │    │ fct_transactions      │  │
│  │          │    │ stg_rpg__*   │    │ dim_communes          │  │
│  │          │    │ stg_geo__*   │    │ dim_types_biens       │  │
│  │          │    │              │    │ dim_dates              │  │
│  │          │    │              │    │ dim_parcelles          │  │
│  └──────────┘    └──────────────┘    └───────────────────────┘  │
│                                                                 │
│  Partitioned by year │ Indexed on commune, date, type           │
└──────────┬──────────────────────────────┬───────────────────────┘
           │                              │
           ▼                              ▼
┌─────────────────────┐    ┌──────────────────────────────────┐
│   METABASE           │    │   STREAMLIT + DEEPSEEK           │
│   4 Dashboards       │    │   Text-to-SQL Chat (French)      │
│   - Overview map     │    │   "Combien de transactions        │
│   - Agricultural     │    │    agricoles à Saint-Paul          │
│   - Natural hazards  │    │    en 2024 ?"                     │
│   - Trends           │    │                                  │
└─────────┬────────────┘    └──────────────┬───────────────────┘
          │                                │
          ▼                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                 NGINX (Reverse Proxy + SSL)                      │
│           dashboard.reunia.re / dashboard.reunia.re/chat        │
└─────────────────────────────────────────────────────────────────┘

Orchestration: Kestra (batch — triggered every 6 months when DVF updates)
Infrastructure: Docker Compose on OVH VPS (4 vCores / 8 GB RAM / 75 GB SSD)
```

---

## 🛠 Technologies

| Layer | Technology | Why |
|-------|-----------|-----|
| **Containerization** | Docker + Docker Compose | Reproducible deployment, all services in one command |
| **Data Warehouse** | PostgreSQL 16 + PostGIS | DVF+ ships as PostgreSQL SQL, geospatial queries needed for RPG enrichment |
| **Transformations** | dbt (dbt-postgres) | Staging → intermediate → marts pattern, built-in testing, Kimball dimensional model |
| **Orchestration** | Kestra | Batch pipeline triggered semiannually (DVF updates in April/October) |
| **Dashboards** | Metabase | Open-source BI with native SQL, choropleth maps, interactive filters |
| **AI Chat** | Streamlit + LangChain + DeepSeek | Text-to-SQL in French — users ask questions in natural language |
| **Reverse Proxy** | Nginx + Let's Encrypt | HTTPS, routing, password protection |
| **IaC** | Docker Compose | Infrastructure-as-code for the full stack |
| **Language** | Python 3.11+ | Ingestion scripts, Streamlit app, API calls |

---

## 📈 Dashboard

The project includes **4 interactive Metabase dashboards**:

### 1. Overview — La Réunion Real Estate Map
- Choropleth map of all 24 municipalities colored by median price/m²
- Filters: year, transaction type, municipality
- Key metrics: total volume, median price, year-over-year change

### 2. Agricultural Land Focus
- Agricultural transactions filtered via RPG crop type overlay
- Price comparison: sugarcane vs. market gardening vs. livestock land
- Comparison with official ministerial price benchmarks
- Relevant for SAFER's CDAF (land division committee) decisions

### 3. Natural Hazards × Prices
- Impact of Géorisques hazard zones on transaction prices
- Overlay: flood zones, volcanic risk, landslide areas
- Quantified price discount in hazard zones vs. safe zones

### 4. Trends & Pressure Detection
- 10-year price evolution by municipality and property type
- Volume trends and seasonality
- Detection of abnormal price pressure zones

> Each dashboard loads in < 3 seconds, with drill-down from island-wide to individual municipality level.

---

## 🤖 AI Chat (Text-to-SQL)

An AI-powered interface built with Streamlit where users ask questions about the data **in French**:

```
User: "Combien de transactions agricoles à Saint-Paul en 2024 ?"
       (How many agricultural transactions in Saint-Paul in 2024?)

→ Generated SQL:
  SELECT COUNT(*)
  FROM marts.fct_transactions t
  JOIN marts.dim_communes c ON t.commune_id = c.commune_id
  WHERE c.nom = 'Saint-Paul'
    AND t.annee = 2024
    AND t.est_agricole = true;

→ Result: 47 transactions
```

**Security measures:**
- Read-only PostgreSQL user (SELECT only)
- SQL validation (no INSERT/UPDATE/DELETE/DROP)
- Query timeout (30 seconds)
- Rate limiting

---

## 🔄 Data Transformations (dbt)

### Model Structure (Kimball Dimensional Model)

```
models/
├── staging/              -- Clean, type, filter to dept. 974
│   ├── stg_dvf__mutations.sql
│   ├── stg_dvf__dispositions.sql
│   ├── stg_dvf__locaux.sql
│   ├── stg_rpg__parcelles.sql
│   └── stg_georisques__risques.sql
│
├── intermediate/         -- Enrichment joins
│   └── int_transactions__enriched.sql
│
└── marts/                -- Final dimensional model
    ├── fct_transactions.sql    -- Fact: 1 row = 1 transaction
    ├── dim_communes.sql        -- 24 municipalities + geometry
    ├── dim_types_biens.sql     -- Property types
    ├── dim_dates.sql           -- Date dimension
    └── dim_parcelles.sql       -- Parcels + hazard zone + crop type
```

### Key Transformations
- **Deduplication** of DVF mutations using `ROW_NUMBER()` (learned in Module 4)
- **Geospatial join** between DVF parcels and RPG agricultural parcels (PostGIS `ST_Intersects`)
- **Price per m² calculation** with outlier filtering (excluding transactions with `valeur_fonciere <= 0` or `> 50M€`)
- **Hazard zone enrichment** via Géorisques API by INSEE code
- **Partitioning** by year for query performance (applied Module 3 concepts: achieved 12x scan reduction in BigQuery, adapted for PostgreSQL)

### dbt Tests
- `unique` and `not_null` on all primary keys
- `accepted_values` for property types and municipality codes
- `relationships` tests for foreign key integrity
- Custom tests: no transactions before 2014, all communes have `code_insee LIKE '974%'`

---

## ⚙️ Pipeline Orchestration

**Kestra** orchestrates the semiannual batch pipeline (triggered when DVF publishes new data in April and October):

```
Flow: dvf_reunion_pipeline
│
├── Task 1: Download DVF+ 974 (Cerema SQL dump)
├── Task 2: Load into PostgreSQL raw schema
├── Task 3: Download RPG GeoPackage (IGN)
├── Task 4: Load RPG into PostgreSQL
├── Task 5: Fetch Géorisques data (API calls by commune)
├── Task 6: Run dbt (staging → intermediate → marts)
├── Task 7: Run dbt tests
└── Task 8: Refresh Metabase cache
```

The pipeline is designed for **batch processing** since DVF data is published only twice per year. This aligns with the data's nature — real estate transactions are inherently batch-oriented.

---

## 🔁 Reproducibility

### Prerequisites
- Docker & Docker Compose
- Git
- 8 GB RAM minimum (for all services)

### Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/kalou/dvf-reunion.git
cd dvf-reunion

# 2. Configure environment
cp .env.example .env
# Edit .env with your settings (API keys, passwords)

# 3. Start all services
docker compose up -d

# 4. Run the data pipeline
docker compose exec ingestion python download_dvf.py
docker compose exec ingestion python load_dvf.py
docker compose exec ingestion python load_rpg.py
docker compose exec ingestion python load_georisques.py

# 5. Run dbt transformations
docker compose exec dbt dbt run
docker compose exec dbt dbt test

# 6. Access the dashboards
# Metabase:  http://localhost:3000
# Chat AI:   http://localhost:8501
```

### Environment Variables

See `.env.example` for the full list. Key variables:

```
DVF_DB_HOST=postgres
DVF_DB_PORT=5432
DVF_DB_NAME=dvf_reunion
DVF_DB_USER=dvf_app
DVF_DB_PASSWORD=<your_password>
DVF_DEEPSEEK_API_KEY=<your_api_key>
DVF_METABASE_ADMIN_EMAIL=<your_email>
```

### Docker Services

| Service | Port | Description |
|---------|------|-------------|
| PostgreSQL + PostGIS | 5432 (internal) | Data warehouse |
| Metabase | 3000 | BI dashboards |
| Streamlit | 8501 | AI chat interface |
| Kestra | 8080 | Pipeline orchestration UI |
| Nginx | 80/443 | Reverse proxy (production) |

All services include **health checks** and restart policies.

---

## 📁 Project Structure

```
dvf-reunion/
├── docker-compose.yml          # All services orchestrated
├── .env.example                # Environment variables template
├── README.md                   # This file
│
├── docker/
│   ├── postgres/init.sql       # Schema creation (raw/staging/marts)
│   ├── nginx/default.conf      # Reverse proxy routing
│   └── streamlit/Dockerfile    # Custom Streamlit image
│
├── ingestion/                  # Python ingestion scripts
│   ├── download_dvf.py
│   ├── load_dvf.py
│   ├── load_rpg.py
│   └── load_georisques.py
│
├── dbt_dvf/                    # dbt project
│   ├── dbt_project.yml
│   ├── models/
│   │   ├── staging/
│   │   ├── intermediate/
│   │   └── marts/
│   ├── tests/
│   ├── macros/
│   └── seeds/                  # Official price benchmarks
│
├── streamlit_app/              # AI chat application
│   ├── app.py
│   ├── prompts/
│   └── utils/
│
├── kestra/flows/               # Orchestration workflows
│   └── dvf_pipeline.yml
│
├── geojson/                    # Municipality boundaries
│   └── communes_974.geojson
│
└── tests/                      # Python tests
```

---

## 🧠 What I Learned

### Technical Skills (mapped to Zoomcamp modules)

| Module | Topic | Applied In This Project |
|--------|-------|------------------------|
| 1 | Docker & Terraform | Full Docker Compose stack (6 services), IaC approach |
| 2 | Workflow Orchestration (Kestra) | Semiannual batch pipeline with 8 tasks |
| 3 | Data Warehouse (BigQuery) | PostgreSQL with partitioning by year, indexing strategy (12x scan reduction concept adapted) |
| 4 | Analytics Engineering (dbt) | Kimball dimensional model, staging → marts, ROW_NUMBER() deduplication, dbt tests |
| 5 | Batch Processing (Spark/Bruin) | Batch ingestion pattern (PostgreSQL sufficient for ~60K records) |
| 6 | Streaming (Kafka) | Understood streaming concepts; DVF is inherently batch (2x/year updates) |

### Domain Knowledge
- French real estate data ecosystem (DVF, DV3F, Cerema, DGFiP)
- Agricultural land management (SAFER, RPG, crop types, official price benchmarks)
- Geographic information systems (PostGIS, EPSG projections, geospatial joins)
- Natural hazard assessment (Géorisques API, risk zone classification)

### Beyond the Course
- **Text-to-SQL with LLM**: Built an AI interface allowing non-technical users to query data in French
- **Multi-source enrichment**: Cross-referenced 4+ open data sources via geospatial joins
- **Real-world application**: This project is being presented to SAFER Réunion as a decision-support tool prototype

---

## 👤 About

**Kalou** (Mangaman Mouny Wilson) — Data Engineer based in Saint-Denis, La Réunion.

Building data and AI solutions for local impact in the Indian Ocean region.

- 🔗 LinkedIn: [Coming soon]
- 🐦 Twitter/X: #DEZoomcamp

---

## 📄 License

This project is open-source. Data sources are published under French government open licenses (Licence Ouverte v2).

---

*Built with ❤️ in La Réunion 🇷🇪 — DE Zoomcamp 2026*