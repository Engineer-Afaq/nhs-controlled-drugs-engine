# 💊 The NHS Controlled Drugs Data Engine

![Project Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Tech Stack](https://img.shields.io/badge/Tech-Microsoft_Fabric%20%7C%20PySpark%20%7C%20Power_BI%20%7C%20OpenAI-blue)

An automated data pipeline and interactive dashboard that extracts, cleans, and translates NHS controlled drug prescription data into human-readable insights.

**[INSERT YOUR BEST DASHBOARD SCREENSHOT HERE: e.g., `![Main Dashboard](./images/dashboard_main.png)`]**

---

## 🚨 The Problem

The NHS holds massive amounts of publicly available data on controlled drug prescriptions, but it’s trapped in messy, inconsistent API endpoints. 

*   **For the Public:** The data is unreadable. A normal person can't look at a raw CSV of chemical codes and understand what these drugs do to their brain over time. 
*   **For NHS Executives:** The data is too fragmented. Decision-makers cannot easily see the big picture to spot real-time prescribing surges, identify loopholes, or know which drugs need urgent oversight.

## 💡 The Solution

We built an automated, highly scalable data pipeline on **Microsoft Fabric**. It extracts millions of records, cleans them using **PySpark**, enriches them using **OpenAI**, and serves them directly into a **Power BI** dashboard. 

It is designed to translate cold, raw data into human-readable health insights and executive-level alerts.

---

## ⚙️ How It Works: The Medallion Architecture

To handle the sheer volume and messiness of the NHS API, we built a three-stage "Medallion" data lake (Bronze, Silver, Gold). Here is how the data flows from the internet to the dashboard:

### 🥉 Stage 1: The Raw Catch (Bronze Layer)
First, our PySpark engine connects to the NHS API. For this project, we successfully pulled every single record for a specific, highly controlled list of drugs (e.g., Diazepam, Fentanyl) spanning from **January 2021 to October 2025**. 

The NHS API is notorious for rate limits and connection drops. To fix this, we built "digital shock absorbers." Our code handles API pagination automatically and uses exponential backoff—meaning if the NHS server gets overwhelmed, our engine politely waits and retries without crashing. All data is saved exactly as we found it into a **Delta Table**.
```python
def datastore_fields(resource_id: str):
    """
    Get field IDs in original order using datastore_search metadata.
    (NHSBSA doesn't expose datastore_info, so this is the reliable approach.)
    """
    payload = http_get_json(DATASTORE_SEARCH, params={"resource_id": resource_id, "limit": 0}, timeout=120)

    fields = payload.get("result", {}).get("fields", None)
    if not fields:
        # fallback if portal doesn't return fields for limit=0 (rare): request 1 record
        payload = http_get_json(DATASTORE_SEARCH, params={"resource_id": resource_id, "limit": 1}, timeout=120)
        fields = payload["result"]["fields"]

    ids = [f["id"] for f in fields]
    if DROP_INTERNAL_ID:
        ids = [c for c in ids if c != "_id"]
    return ids

RESOURCE_IDS = [f"EPD_SNOMED_{ym}" for ym in ym_range(START_YM, END_YM)]

print(f"Resources: {len(RESOURCE_IDS)}")
print("First:", RESOURCE_IDS[:3], "Last:", RESOURCE_IDS[-3:])


### 🥈 Stage 2: The Deep Clean (Silver Layer)
Next, we clean the data. The biggest technical hurdle was schema drift—the NHS constantly changes column names from year to year.

We wrote PySpark functions that act as a universal translator. It maps old column names to new ones without deleting data. We then clean up the numbers—converting text into proper mathematical formats (bigint for quantities, decimal for costs) so the math on the dashboard is perfect.

def load_resource(resource_id: str, canonical_cols: list[str], canonical_fields_for_mapping: list[str]):
    ym = int(resource_id.split("_")[-1])
    filter_field = filter_field_for_ym(ym)

    old_fields = datastore_fields(resource_id)
    rename_map = build_rename_map(old_fields, canonical_fields_for_mapping)

    month_df = None
    pages = 0
    rows = 0

    for records in datastore_search_pages(resource_id, filter_field, BNF_CODES, limit=LIMIT):
        pages += 1
        rows += len(records)

        df = records_to_df(records)

        # Apply renames (to match 202503+ column names)
        for old_name, new_name in rename_map.items():
            if old_name in df.columns and new_name != old_name:
                df = df.withColumnRenamed(old_name, new_name)

        df = normalize_to_canonical(df, canonical_cols)
        df = df.withColumn("SOURCE_RESOURCE", F.lit(resource_id))

        month_df = df if month_df is None else month_df.unionByName(df)

    return month_df, pages, rows


### 🧠 Stage 3: The AI Brain (OpenAI Integration)
This is what sets this project apart. We don't just want to show numbers; we want to generate meaning.

Once the data is clean, we securely pass the drug codes and trends to the OpenAI API. We prompt the AI to act as a medical explainer. It looks at the chemicals and automatically generates simple, plain-English summaries about:

What the drug is.

The severe side effects and addiction risks.

What prolonged use actually does to the human brain.

Summaries of prescribing trends.


### 🥇 Stage 4: The Display (Gold Layer & Power BI)
Finally, all the perfectly clean numbers from PySpark and the brilliant text summaries from OpenAI are merged into the Gold Layer. This feeds directly into Power BI, creating an interactive map and dashboard where anyone can explore the data instantly.

<img width="1445" height="806" alt="Screenshot 2026-05-02 153757" src="https://github.com/user-attachments/assets/eac7823a-d16a-4adc-b3bf-b9f2890f152c" />
<img width="1446" height="803" alt="Screenshot 2026-05-02 153728" src="https://github.com/user-attachments/assets/d3e81337-9714-4974-ab05-e85013a8f08a" />


### 🏗️ Tech Stack
Data Orchestration & Compute: Microsoft Fabric, Apache Spark (PySpark)

Storage: Delta Lake (Medallion Architecture)

AI/LLM Integration: OpenAI API

Business Intelligence: Power BI

### 💰 Cost-Smart & Fully Scalable
Cloud computing can get incredibly expensive. To be smart with funding, this project was designed as a static historical load (running the massive dataset from 2021 to 2025 once) rather than an auto-refreshing daily pipeline. This proves the concept works flawlessly while keeping compute costs near zero.

However, the architecture is fully enterprise-ready. If adopted:

Scale the Data: We can simply remove the specific drug filter and ingest every single drug in the UK.

Automate the Engine: We can wrap this notebook in a Fabric Pipeline and schedule it to run automatically every single month.

### 🚀 Repository Contents
nhs_pipeline.ipynb: The core PySpark notebook containing all extraction, transformation, and AI logic.

nhs_dashboard.pbix: The Power BI dashboard file (requires Power BI Desktop to open).

/images: Folder containing screenshots of the working dashboard.
