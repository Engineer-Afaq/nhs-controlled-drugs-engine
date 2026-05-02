💊 The NHS Controlled Drugs Data Engine
An automated data pipeline and interactive dashboard that extracts, cleans, and translates NHS controlled drug prescription data into human-readable insights.

[INSERT YOUR BEST DASHBOARD SCREENSHOT HERE: e.g., ![Main Dashboard](./images/dashboard_main.png)]

🚨 The Problem
The NHS holds massive amounts of publicly available data on controlled drug prescriptions, but it’s trapped in messy, inconsistent API endpoints.

For the Public: The data is unreadable. A normal person can't look at a raw CSV of chemical codes and understand what these drugs do to their brain over time.

For NHS Executives: The data is too fragmented. Decision-makers cannot easily see the big picture to spot real-time prescribing surges, identify loopholes, or know which drugs need urgent oversight.

💡 The Solution
We built an automated, highly scalable data pipeline on Microsoft Fabric. It extracts millions of records, cleans them using PySpark, enriches them using OpenAI, and serves them directly into a Power BI dashboard.

It is designed to translate cold, raw data into human-readable health insights and executive-level alerts.

⚙️ How It Works: The Medallion Architecture
To handle the sheer volume and messiness of the NHS API, we built a three-stage "Medallion" data lake (Bronze, Silver, Gold). Here is how the data flows from the internet to the dashboard:

🥉 Stage 1: The Raw Catch (Bronze Layer)
First, our PySpark engine connects to the NHS API. For this project, we successfully pulled every single record for a specific, highly controlled list of drugs (e.g., Diazepam, Fentanyl) spanning from January 2021 to October 2025.

The NHS API is notorious for rate limits and connection drops. To fix this, we built "digital shock absorbers." Our code handles API pagination automatically and uses exponential backoff—meaning if the NHS server gets overwhelmed, our engine politely waits and retries without crashing. All data is saved exactly as we found it into a Delta Table.

[INSERT CODE SNIPPET 1 HERE]
Paste your code for: http_get_json, the retry/backoff logic, and the datastore_search_pages function that handles the API pagination.

🥈 Stage 2: The Deep Clean (Silver Layer)
Next, we clean the data. The biggest technical hurdle was schema drift—the NHS constantly changes column names from year to year.

We wrote PySpark functions that act as a universal translator. It maps old column names to new ones without deleting data. We then clean up the numbers—converting text into proper mathematical formats (bigint for quantities, decimal for costs) so the math on the dashboard is perfect.

[INSERT CODE SNIPPET 2 HERE]
Paste your code for: The schema drift rename_map, the safe column renaming functions, and the PySpark casting logic.

🧠 Stage 3: The AI Brain (OpenAI Integration)
This is what sets this project apart. We don't just want to show numbers; we want to generate meaning.

Once the data is clean, we securely pass the drug codes and trends to the OpenAI API. We prompt the AI to act as a medical explainer. It looks at the chemicals and automatically generates simple, plain-English summaries about:

What the drug is.

The severe side effects and addiction risks.

What prolonged use actually does to the human brain.

Summaries of prescribing trends.

[INSERT CODE SNIPPET 3 HERE]
Paste your code for: The OpenAI API calls, how you pass the PySpark DataFrame rows to the model, and the specific prompts you use.

🥇 Stage 4: The Display (Gold Layer & Power BI)
Finally, all the perfectly clean numbers from PySpark and the brilliant text summaries from OpenAI are merged into the Gold Layer. This feeds directly into Power BI, creating an interactive map and dashboard where anyone can explore the data instantly.

[INSERT A SECOND SCREENSHOT HERE SHOWING A SPECIFIC AI SUMMARY OR MAP]

🏗️ Tech Stack
Data Orchestration & Compute: Microsoft Fabric, Apache Spark (PySpark)

Storage: Delta Lake (Medallion Architecture)

AI/LLM Integration: OpenAI API

Business Intelligence: Power BI

💰 Cost-Smart & Fully Scalable
Cloud computing can get incredibly expensive. To be smart with funding, this project was designed as a static historical load (running the massive dataset from 2021 to 2025 once) rather than an auto-refreshing daily pipeline. This proves the concept works flawlessly while keeping compute costs near zero.

However, the architecture is fully enterprise-ready. If adopted:

Scale the Data: We can simply remove the specific drug filter and ingest every single drug in the UK.

Automate the Engine: We can wrap this notebook in a Fabric Pipeline and schedule it to run automatically every single month.

🚀 Repository Contents
nhs_pipeline.ipynb: The core PySpark notebook containing all extraction, transformation, and AI logic.

nhs_dashboard.pbix: The Power BI dashboard file (requires Power BI Desktop to open).

/images: Folder containing screenshots of the working dashboard.
