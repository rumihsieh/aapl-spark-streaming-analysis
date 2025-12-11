# AAPL Stock Streaming Analysis with Apache Spark

This project demonstrates how to build a real-time streaming analytics pipeline using **Apache Spark SQL** and **Structured Streaming**.  
We analyze intraday AAPL (Apple Inc.) stock data, compute technical indicators, and generate trading signals based on static + streaming data.

The project simulates how real-world trading systems merge historical data with real-time ticks to make minute-level trade decisions.

---

##  Project Overview

This project performs:

1. **Static data processing**  
   - Load AAPL29.csv (one-minute OHLCV data)  
   - Clean & transform timestamps  
   - Engineer technical indicators:  
     - `r_close` = (close − open) / (high − low)  
     - `pct_change`  
     - Moving averages: `ma3` and `ma6`  
   - Shift timestamps to create the next-minute join key

2. **Structured Streaming ingestion**  
   - Read streaming minute-level CSVs from `liveStream/`  
   - Preprocess timestamps to match static dataset  
   - Rename columns for joining

3. **Real-time trading signal generation**  
   Join historical data with new streaming rows and filter signals using:

   - `ma3 > ma6`  
   - `close <= new_close`  
   - `r_close > 0.9`

4. **Output results to console in streaming mode**

---

##  Technical Indicators Used

| Indicator | Meaning |
|----------|---------|
| **r_close** | Measures trend strength (1 = strong uptrend, -1 = strong downtrend) |
| **pct_change** | 1-minute percent change |
| **ma3** | 3-minute moving average of pct_change |
| **ma6** | 6-minute moving average of pct_change |

---

##  Architecture

    Static Data (AAPL29.csv)
              │
 ┌────────────┴────────────┐
 │ Feature Engineering & MA │
 └────────────┬────────────┘
              ▼
     hist_data (next_timestamp key)
              │
      real-time join
              │
   Streaming Data (liveStream/*.csv)
              ▼
     Apply trading rules
              ▼
 Real-time output to console

---

##  Project Structure

.
├── aapl-spark-streaming-analysis.ipynb # Main notebook
├── AAPL29.csv # Historical static data
├── liveStream/
│ ├── stream1.csv # Streaming data files
│ └── stream2.csv
└── README.md # You're reading it!

---

##  How to Run the Project

### **1. Prepare streaming files**
Place streaming CSV files in a folder:

liveStream/
stream1.csv
stream2.csv

Spark will ingest them one at a time using:

```scala
.option("maxFilesPerTrigger", 1)
2. Run the notebook in a Spark environment
You need:
Spark 3.x
Scala kernel (Apache Toree / Databricks / EMR / local Spark)
Run all cells; at the bottom you will see:
Batch 0
--------------------------------------------------------
| date | time | new_close | ma3 | ma6 | r_close | close|
--------------------------------------------------------
| ... streaming output ... |
Each new file in liveStream/ produces a new streaming batch.
 Key Features
✔ Uses both static & streaming data together
✔ Real-time factor computation
✔ Window functions & lag operations
✔ Custom UDFs for timestamp manipulation
✔ Join between historical + streaming minute-level ticks
✔ Streaming rules to generate actionable trade signals
 Example Output
Batch: 0
+----------+------+----------+---------+---------+--------+-------+
|   date   | time | new_close|   ma3   |   ma6   | r_close| close |
+----------+------+----------+---------+---------+--------+-------+
|2024-10-29| 1636 |   232.50 |   1.39  |  -3.57  | 1.0    | 232.29|
+----------+------+----------+---------+---------+--------+-------+
 Technologies Used
Apache Spark 3.x
Structured Streaming
Spark SQL Window Functions
Scala
UDFs & Case Classes
