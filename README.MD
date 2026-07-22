
# 🏏 IPL 2026 Data Analytics — End-to-End Azure Data Pipeline

An end-to-end data engineering and analytics project built on the **IPL 2026** dataset, covering the full journey from raw CSV files to an interactive Power BI dashboard. I chose cricket data because it's a domain I genuinely enjoy exploring — and it offers rich, real-world modelling challenges (matches, teams, players, venues, contracts) that map naturally to dimensional design.

---

## 🔧 Tech Stack

| Layer | Tool |
|---|---|
| Orchestration / Ingestion | Azure Data Factory (ADF) |
| Staging Storage | Azure Blob Storage |
| Database | Microsoft SQL Server (SQLEXPRESS) |
| Transformation | T-SQL (Stored Procedures) |
| Data Modelling | Star Schema (Kimball methodology) |
| Reporting | Power BI |
| Source Control | GitHub (ADF Git integration) |

---

## 🏗️ Architecture Overview

```
IPL 2026 CSVs  →  Azure Blob Storage  →  ADF Copy Activity  →  Staging (dbo.IplData)
                                                                      │
                                                        Stored Procedures (T-SQL)
                                                                      │
                                                     Star Schema (Dims + Fact)
                                                                      │
                                                              Power BI Dashboard
```

---

## ⚙️ ADF Pipeline — `PL_MAIN`

![ADF Pipeline](Images/ADF%20Pipeline.png)

The pipeline orchestrates ingestion and transformation as a single dependency-ordered flow:

1. **Copy Activity (`adls_to_sql`)** — Ingests the raw IPL CSV data from Azure Blob Storage and lands it into a staging table (`dbo.IplData`) in SQL Server. A **Self-Hosted Integration Runtime (SHIR)** bridges cloud ADF to the on-premises SQL Server Express instance.
2. **Copy Activity (`Sql_to_sql`)** — Loads supplementary player attribute data (contracts, nationality, date of birth) into the player-profile staging table.
3. **Stored Procedure — `Player`** — Loads `Dim_Player` from staging.
4. **Stored Procedure — `Players Additional Info`** — Enriches `Dim_Player` with contract value, contract year, nationality, date of birth, and team, merged from the player-profile source.
5. **Stored Procedures — `Team` → `Venue` → `Fact`** — Load `Dim_Team`, `Dim_Venue`, and finally `Fact_Match`, executed in dependency order so all dimension keys exist before the fact table references them.

The full pipeline runs end-to-end with a **Succeeded** status.

---

## ⭐ Data Modelling — Star Schema

The warehouse follows **Kimball dimensional modelling** principles:

- The **fact table** holds the measurable values (scores, match outcomes, winners)
- The **dimension tables** hold the descriptive context used for slicing and filtering

![Data Model](Images/Data%20Model.png)

| Table | Type | Description |
|---|---|---|
| `Fact_Match` | Fact | One row per match — measures and foreign keys (Team1_ID, Team2_ID, Match_winner_ID, Player_of_MatchID, VenueID) |
| `Dim_Team` | Dimension | Team attributes — role-playing (team 1 / team 2 / winner) |
| `Dim_Player` | Dimension | Player attributes — name, nationality, date of birth, **contract value**, contract year, team |
| `Dim_Venue` | Dimension | Stadium / city / state / location attributes |

**Design decisions**
- **Surrogate keys** (via `IDENTITY`) on all dimensions
- **Role-playing dimensions** — `Dim_Team` and `Dim_Player` each play multiple roles in `Fact_Match`
- **Single-direction cross-filtering** — every dimension → fact relationship filters one way, keeping the model clean and predictable
- **MERGE-based loads** — dimension loads use `MERGE` (upsert) so re-runs update existing rows and insert new ones without duplication

---

## 🔄 Transformation Approach — ELT

This project follows an **ELT** pattern rather than traditional ETL:

- **E — Extract:** Raw CSV files sourced and placed in Azure Blob Storage
- **L — Load:** Data loaded as-is into SQL Server staging (`dbo.IplData`, `dbo.playerprofile`) via ADF
- **T — Transform:** All transformation happens inside SQL Server using T-SQL stored procedures, reshaping staged data into the star schema

![Store Procedue](Images/STORE%20PROD.png)


This keeps the raw data intact in staging and makes transformations **repeatable, debuggable, and version-controlled**.

---

## 📊 Power BI Dashboard — *IPL 2026 Analysis*

The final star schema feeds an interactive Power BI dashboard for analysis across teams, players, venues, contracts, and match outcomes.

**Highlights**
- **KPI cards** — Total Matches, Total Teams, Total Players, Average Contract Value (₹), Total Venues
- **Teams Info** — full team reference list
- **Match table** — match-level results with team and winner IDs
- **Match Location** — geographic map of venues across India
- **Highest Score of Top 10 Batsmen** — filtered bar chart ranking the season's top run-scorers
- **Interactive cross-filtering** — selecting a player or team updates related visuals through the modelled relationships

![PowerBi Dashboad Page](Images/Dashboad%20Home%20page.png)
---

## 📁 Repository Structure

This repo is the **live Git integration** for the Azure Data Factory instance — the pipeline itself, as code:

```
├── dataset/              # ADF dataset definitions (JSON)
├── factory/              # Factory-level configuration
├── integrationRuntime/   # Self-Hosted Integration Runtime definition
├── linkedService/        # Connections to Blob Storage & SQL Server
├── pipeline/             # Pipeline definition (PL_MAIN, JSON)
└── publish_config.json   # ADF publish configuration
```

---

## 🚀 Key Learnings

- Configuring **ADF Git integration** with a save → publish workflow (source-controlled development vs. live deployment)
- Setting up a **Self-Hosted Integration Runtime** to bridge cloud ADF and a local SQL Server Express instance
- Applying **Kimball methodology** — staging vs. presentation area, atomic grain, surrogate keys, role-playing dimensions
- Writing **MERGE-based stored procedures** for idempotent dimension loads, and handling real-world data quality issues (duplicate keys, encoding artifacts, type mismatches)
- Designing a **clean star schema in Power BI** — single-direction relationships and controlled cross-filtering
- Choosing **ELT over ETL** to leverage SQL Server's transformation power

---

## 📌 Dataset

**IPL 2026 dataset (Kaggle)** — match-level cricket data across the 2026 Indian Premier League season, enriched with player contract and profile data.

---

*Built by **Sehaj Kaur** — aspiring Data Analyst / Engineer | SQL · Power BI · Azure · Python*
