# Alerta Agro (Boyacá)

**EN:** Open, reproducible **early‑warning data pipeline** for food‑security monitoring in Colombia (Boyacá). Processes **Sentinel‑2 (NDVI)** and **CHIRPS (rainfall)** in **Google Earth Engine**, aggregates to **municipality** (official **DIVIPOLA** codes), and publishes weekly datasets with **QA checks**, **data contracts**, optional **PostGIS** model, and **scheduled refresh**. Outputs feed a Spanish dashboard (Project 2) and a markets‑and‑climate study (Project 3).

**ES:** Pipeline de **alerta temprana** abierto y reproducible para el monitoreo de seguridad alimentaria en Colombia (Boyacá). Procesa **Sentinel‑2 (NDVI)** y **CHIRPS (precipitación)** en **Google Earth Engine**, agrega a nivel **municipio** (códigos **DIVIPOLA**) y publica datasets semanales con **controles de calidad**, **contratos de datos**, **modelo PostGIS** opcional y **refresco programado**. Los productos alimentan un tablero en español (Proyecto 2) y un estudio mercados‑clima (Proyecto 3).

---

## Features / Características

* **GEE → CSV** weekly pipeline: NDVI (median composite, cloud probability masking) + rainfall (CHIRPS weekly sum)
* **Municipality‑level** aggregation using official **DIVIPOLA** codes
* **Climatology pack** (2018–2024) for anomalies (used in Project 2/Phase 2)
* **QA checklist** and reproducible params (`params_week1.json`)
* **Open‑data compliant** with Spanish/English attribution blocks
* Optional **PostGIS** model (`dim_muni`, `fact_weekly`, materialized views)
* Optional **CI/CD + scheduler** for automated refresh

---

## Data sources & licensing / Fuentes de datos y licencias

* **Boundaries / Límites administrativos:** DANE **MGN – Municipio** (official), with **DIVIPOLA** codes. *Open data / Datos Abiertos* (atribución).
* **NDVI:** **Sentinel‑2 Level‑2A SR**, **Cloud Probability** join for masking. *Copernicus free & open data policy (atribución)*.
* **Rainfall / Precipitación:** **CHIRPS Daily**, aggregated to weekly sum. *Open reuse with attribution*.

> See `docs/licencias_atribucion_ES.md` for full text you’ll fill in Week 1 Day 5.

---

## Quickstart / Arranque rápido

### Prerequisites

* Google Earth Engine account
* GEE asset for **Boyacá municipalities** (fields: `MUNI_CODE` (Text), `MUNI_NAME` (Text))
* (Optional) Python 3.11+, Postgres 14+ with PostGIS

### 1) Clone & scaffold

```bash
git clone https://github.com/<you>/alerta-agro-boyaca.git
cd alerta-agro-boyaca
# minimal folders if not present
mkdir -p data docs outputs params
```

### 2) Configure parameters

Create `params/params_week1.json` (you’ll draft it during Week 1 and finalize on Day 5). Example stub:

```json
{
  "AOI": "Boyacá",
  "asset_munis": "users/<username>/boyaca_municipios",
  "week_rule": "Mon-Sun",
  "operational_weeks": 12,
  "operational_end": "YYYY-MM-DD",
  "climatology_start": "2018-01-01",
  "climatology_end": "2024-12-31",
  "cloud_method": "S2 Cloud Probability",
  "cloud_threshold": 60,
  "composite": "median",
  "scale_m": 250,
  "exports": {
    "target": "Drive",
    "folder": "gee/semana"
  }
}
```

### 3) Run GEE exports (Week 1)

* Open **Google Earth Engine Code Editor** → new script `week1_boyaca`.
* Import your municipality asset (`users/<username>/boyaca_municipios`).
* Build weekly **NDVI** (S2 SR + Cloud Probability masking) and **CHIRPS** weekly sum (matching week boundaries).
* Use `reduceRegions` to aggregate to municipalities at your chosen `scale`.
* **Export to CSV** (Drive) with columns below.

Expected outputs in `outputs/`:

* `weekly_ndvi_rain_boyaca.csv`
  `week_start, MUNI_CODE, MUNI_NAME, ndvi_mean, ndvi_qc, rain_mm`
* `climatology_2018_2024_ndvi_rain.csv`
  `MUNI_CODE, week_of_year, ndvi_mean_clim, ndvi_std_clim, rain_mm_clim, rain_std_clim`

### 4) (Optional) Load to PostGIS

```bash
# Example using ogr2ogr from outputs/ directory
ogr2ogr -f PostgreSQL \
  "PG:host=localhost dbname=agro user=postgres password=***" \
  weekly_ndvi_rain_boyaca.csv \
  -nln fact_weekly -lco GEOM_TYPE=NONE -oo X_POSSIBLE_NAMES=lon -oo Y_POSSIBLE_NAMES=lat
```

Create `dim_muni` from your Boyacá municipalities reference and index `MUNI_CODE`.

---

## Repository structure / Estructura del repositorio

```
alerta-agro-boyaca/
├─ data/
│  ├─ raw/                # original downloads (e.g., MGN municipio)
│  └─ processed/          # cleaned Boyacá subset (if stored)
├─ docs/
│  ├─ metodo_ES.md        # Week 1 Day 5
│  └─ licencias_atribucion_ES.md
├─ outputs/
│  ├─ weekly_ndvi_rain_boyaca.csv
│  └─ climatology_2018_2024_ndvi_rain.csv
├─ params/
│  └─ params_week1.json
├─ scripts/               # optional helpers / SQL / ETL
└─ README.md
```

---

## QA checklist (Week 1)

* Rows = **#municipalities × #weeks** (12 weeks initial)
* **NDVI** within **\[−1, 1]**; **rain\_mm ≥ 0**
* `ndvi_qc` present and documented (share of unmasked pixels per muni‑week)
* Spot‑check **Tunja, Duitama, Sogamoso** look reasonable
* `params_week1.json` committed; exports **reproducible**

---

## Outputs / Productos

* **weekly\_ndvi\_rain\_boyaca.csv** — municipality × week (operational window)
* **climatology\_2018\_2024\_ndvi\_rain.csv** — per muni, per week‑of‑year (for anomalies)
* **docs/método (ES)** + **docs/licencias (ES)**

---

## PostGIS model (optional)

* `dim_muni(MUNI_CODE text pk, MUNI_NAME text, DEPT_CODE text, …)`
* `fact_weekly(week_start date, MUNI_CODE text fk, ndvi_mean double, ndvi_qc double, rain_mm double, qc_flags text)`
* Materialized views: `mv_latest_week`, `mv_last_8_weeks`

---

## Testing & data contracts (optional)

* **Great Expectations / pytest** for ranges, nulls, duplicates (MUNI\_CODE × week\_start)
* **Frictionless / JSON Schema** descriptor for both CSVs

---

## Automation & releases (optional)

* **GitHub Actions**: tests → build → release tag (e.g., `v0.1-week1`)
* **Scheduler** (Cloud Run/Lambda + cron) for weekly refresh
* **Status page** in Project 2 dashboard (última actualización, filas procesadas)

---

## Attribution / Atribución

**EN:**

* Administrative boundaries: *DANE – MGN Municipio* (with DIVIPOLA codes). Open data (attribution).
* NDVI: *Copernicus Sentinel‑2 L2A SR* + *S2 Cloud Probability*. Free & open data policy.
* Rainfall: *CHIRPS*. Open use with attribution.

**ES:**

* Límites administrativos: *DANE – MGN Municipio* (con códigos DIVIPOLA). Datos abiertos (atribución).
* NDVI: *Copernicus Sentinel‑2 L2A SR* + *S2 Cloud Probability*. Política de datos libres y abiertos.
* Precipitación: *CHIRPS*. Uso abierto con atribución.

> Add versions/links in `docs/licencias_atribucion_ES.md` during Week 1 Day 5.

---

## License / Licencia

* **Code:** choose a permissive license (e.g., MIT or Apache‑2.0) and add `LICENSE`.
* **Data:** follow source licenses (Copernicus, CHIRPS, DANE Datos Abiertos) with attribution.

---

## Roadmap

* **v0.1 (Week 1):** weekly NDVI + rainfall, climatology, QA, método/licencias
* **v0.2 (Weeks 2–3):** PostGIS model + materialized views
* **v0.3 (Week 4):** QA tests + data contracts
* **v0.4 (Week 7):** CI/CD + scheduled refresh

---

## Maintainer

* Your Name — [your.email@example.com](mailto:your.email@example.com) — \[@yourhandle]
