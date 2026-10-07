# Vancouver Crime Analytics on Microsoft Fabric

An end-to-end analytics platform on Microsoft Fabric that turns 20+ years of Vancouver Police open data into operational intelligence: incident trends, resourcing patterns, and seasonal analysis across 24 neighbourhoods.

Built as a portfolio project with a production mindset: medallion architecture, config-driven ingestion, and conformed dimensions designed to scale across jurisdictions.

## The question it answers

A Director of Operations asks: where are incidents rising, when do they peak, and how should resourcing follow? This project answers that from public data. It is framed around resourcing and trends, never around ranking neighbourhoods by danger.

## Architecture

Medallion on a Fabric lakehouse.

**Bronze: raw ingestion**
Raw VPD extracts land untouched, one table per jurisdiction, each row carrying a source envelope: jurisdiction, load timestamp, and the raw payload. Bronze is immutable. Reprocessing never touches the source again.

**Silver: standardized**
Every jurisdiction maps into one standard schema. Timestamps normalized to a single grain, UTM Zone 10N coordinates converted to WGS84, privacy-offset records flagged, data quality rules applied and logged. The conversion and cleansing logic lives in shared functions, not in per-city code.

**Gold: modeled for analysis**
A star schema serves the dashboards. `fact_incidents` joins to conformed `dim_date`, `dim_geography`, `dim_crime_type`, and `dim_jurisdiction`. A Power BI semantic model sits on top.

**Designed for scale**
- Ingestion is config-driven. A new jurisdiction is a config entry (source URL, format, refresh cadence), not a new pipeline.
- Crime categories map through one conformed taxonomy via mapping tables. Vancouver's 11 offence types are the first mapping.
- Partitioned by jurisdiction and year-month from day one, so the second city never triggers a rebuild.

## Data

Source: Vancouver Police Department open data, https://geodash.vpd.ca/opendata/

- Coverage: 2003 to present, 24 Vancouver neighbourhoods, roughly 880,000 incidents
- Refresh: weekly (every Sunday)
- Licence: VPD open data terms
- Privacy: the VPD deliberately offsets coordinates for offences against a person. Those records are flagged in silver and excluded from precise mapping.

## Project structure

```
fabric-vancouver-crime/
├── README.md
├── docs/
│   ├── architecture.md        # Lakehouse, pipeline, and model design
│   ├── data-dictionary.md     # Bronze, silver, gold schemas
│   └── jurisdiction-onboarding.md  # How to add a new city
├── config/
│   └── jurisdictions.json     # One entry per source jurisdiction
├── bronze/                    # Ingestion notebooks and pipeline definitions
├── silver/                    # Standardization notebooks (shared functions)
├── gold/                      # Star schema DDL and model definitions
├── powerbi/                   # Semantic model and report definitions
└── purview/                   # Glossary terms and lineage notes
```

## Roadmap

- **Phase 1 (now):** Fabric build. Bronze to gold, Power BI dashboard, Purview catalog and lineage wired in.
- **Phase 2:** public crime-trends app on samrath.ai, served from the gold layer.
- **Phase 3:** additional Metro Vancouver jurisdictions, one at a time. Each follows `docs/jurisdiction-onboarding.md`.

## Tech

Microsoft Fabric (Lakehouse, Data Factory, Power BI), Microsoft Purview, Python/PySpark notebooks.
