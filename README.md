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
# [INSERT CODE SNIPPET 1 HERE]
# Paste your code for: http_get_json, the retry/backoff logic, 
# and the datastore_search_pages function that handles the API pagination.
