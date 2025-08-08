# Wafer Test Data Analysis Dashboard using Pandas | PYTHON

## Table of Contents 
1. [Introduction](#introduction)
2. [Data Loading & Cleaning](#data-loading--cleaning)
3. [Yield Calculations](#yield-calculations)
4. [Defect Analysis](#defect-analysis)
5. [Daily Yield Trend](#daily-yield-trend)
6. [Visualization Dashboard](#visualization-dashboard)
7. [Full Python Code](#full-python-code)

## 1. Introduction
This analysis processes **wafer test data** from semiconductor manufacturing to:
- **Clean** and standardize data.
- Calculate **yield** at different levels (Lot, Wafer, Product).
- Identify **top defect types**.
- Visualize trends and distributions via a **multi-plot dashboard**.
- Provide actionable **recommendations** for yield improvement.

## 2. Data Loading & Cleaning

**Steps Performed:**
- Removed extra spaces and standardized text formatting.
- Converted all `Test_Result` values to uppercase (`PASS` / `FAIL`).
- Filled missing numeric values with **mean** or **median** depending on the parameter.

```python
df['Defect_Type'] = df['Defect_Type'].str.strip().str.title()
df['Test_Result'] = df['Test_Result'].str.upper()

df['Vth_mV'] = df['Vth_mV'].fillna(df['Vth_mV'].mean())
df['Leakage_nA'] = df['Leakage_nA'].fillna(df['Leakage_nA'].median())
```

## 3. Yield Calculations

Computed:
- Lot Yield → Yield per Lot ID
- Wafer Yield → Yield per Wafer ID
- Product Yield → Yield per Product Type

```bash
lot_yield = df.groupby('Lot_ID')['Test_Result'].apply(lambda x: (x=='PASS').mean() * 100)
wafer_yield = df.groupby('Wafer_ID')['Test_Result'].apply(lambda x: (x=='PASS').mean() * 100)
product_yield = df.groupby('Product_Type')['Test_Result'].apply(lambda x: (x=='PASS').mean() * 100)
```
## 4. Defect Analysis

Identified:
- Defect counts for FAIL results.
- Top 3 defect types for prioritizing corrective actions.

```bash
defect_counts = df[df['Test_Result']=='FAIL']['Defect_Type'].value_counts()
top_defects = defect_counts.head(3)
```

## 5. Daily Yield Trend

Monitor yield variation over time to identify process drift or anomalies.

```bash
df['Test_Date'] = pd.to_datetime(df['Test_Date'])
daily_yield = df.groupby('Test_Date')['Test_Result'].apply(lambda x: (x=='PASS').mean()*100)
```

## 6. Visualization Dashboard Architecture

<img width="1920" height="1080" alt="Screenshot (1821)" src="https://github.com/user-attachments/assets/59063f3b-3e00-434a-a08f-129911e392d2" />

- With this setup, we can:

  - Compare multiple views at once instead of jumping between plots.
  - Instantly spot defect patterns and yield trends.
  - Easily showcase results to your team in a single snapshot.

| Plot | Description                          |
| ---- | ------------------------------------ |
| 1  | Top 3 defects (Bar Chart)            |
| 2  | Defect type distribution (Pie Chart) |
| 3  | Vth distribution histogram           |
| 4  | Daily yield trend (Line Chart)       |
| 5  | Parametric scatter (Vth vs Leakage)  |
| 6  | Yield by product type (Bar Chart)    |

## 7. Python Code including Pandas

```bash
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Load data
df = pd.read_csv("wafer_test_data.csv")

# --- Cleaning ---
df['Defect_Type'] = df['Defect_Type'].str.strip().str.title()
df['Test_Result'] = df['Test_Result'].str.upper()

# Handle missing numeric values
df['Vth_mV'] = df['Vth_mV'].fillna(df['Vth_mV'].mean())
df['Leakage_nA'] = df['Leakage_nA'].fillna(df['Leakage_nA'].median())

# --- Yield Calculation ---
lot_yield = df.groupby('Lot_ID')['Test_Result'].apply(lambda x: (x=='PASS').mean() * 100)
wafer_yield = df.groupby('Wafer_ID')['Test_Result'].apply(lambda x: (x=='PASS').mean() * 100)
product_yield = df.groupby('Product_Type')['Test_Result'].apply(lambda x: (x=='PASS').mean() * 100)

# --- Defect Analysis ---
defect_counts = df[df['Test_Result']=='FAIL']['Defect_Type'].value_counts()
top_defects = defect_counts.head(3)

# --- Yield Over Time ---
df['Test_Date'] = pd.to_datetime(df['Test_Date'])
daily_yield = df.groupby('Test_Date')['Test_Result'].apply(lambda x: (x=='PASS').mean()*100)

# --------------------------
#   MULTI-PLOT DASHBOARD
# --------------------------
fig, axs = plt.subplots(2, 3, figsize=(16, 10))
fig.suptitle('Wafer Test Data Analysis Dashboard', fontsize=16, fontweight='bold')

# 1. Bar Chart - Top 3 Defects
axs[0,0].bar(top_defects.index, top_defects.values, color='tomato')
axs[0,0].set_title("Top 3 Defects (Count)")
axs[0,0].set_ylabel("Count")
axs[0,0].grid(axis='y', linestyle='--', alpha=0.7)

# 2. Pie Chart - Defect Distribution
axs[0,1].pie(defect_counts.head(5), labels=defect_counts.head(5).index, autopct='%1.1f%%', startangle=90)
axs[0,1].set_title("Defect Type Distribution (Top 5)")

# 3. Histogram - Vth Distribution
axs[0,2].hist(df['Vth_mV'], bins=20, color='skyblue', edgecolor='black')
axs[0,2].set_title("Threshold Voltage Distribution")
axs[0,2].set_xlabel("Vth (mV)")
axs[0,2].set_ylabel("Frequency")

# 4. Line Graph - Daily Yield
axs[1,0].plot(daily_yield.index, daily_yield.values, marker='o', color='green')
axs[1,0].set_title("Daily Yield Trend")
axs[1,0].set_xlabel("Date")
axs[1,0].set_ylabel("Yield (%)")
axs[1,0].grid(True, linestyle='--', alpha=0.7)

# 5. Scatter Plot - Vth vs Leakage
axs[1,1].scatter(df['Vth_mV'], df['Leakage_nA'], c=(df['Test_Result']=='FAIL'), cmap='coolwarm', alpha=0.7)
axs[1,1].set_title("Parametric Distribution (PASS vs FAIL)")
axs[1,1].set_xlabel("Vth (mV)")
axs[1,1].set_ylabel("Leakage (nA)")

# 6. Bar Chart - Product Yield
axs[1,2].bar(product_yield.index, product_yield.values, color='purple')
axs[1,2].set_title("Yield by Product Type")
axs[1,2].set_ylabel("Yield (%)")
axs[1,2].tick_params(axis='x', rotation=45)

plt.tight_layout(rect=[0, 0, 1, 0.96])
plt.show()
```

### Walkthrough of the Script – Data Generation & Importing Data from Datasets

- The script begins by creating a realistic, synthetic dataset that mimics the kind of manufacturing data collected in a semiconductor fabrication facility. It simulates daily production runs over a given period, generating information that would typically be captured in a Manufacturing Execution System (MES) or a dedicated yield tracking database. The dataset includes a Date column to represent each day’s manufacturing batch, along with the Product Type field, which categorizes the type of integrated circuit (IC) being produced, such as Memory, Logic, or Mixed-Signal devices. The Yield (%) column measures the efficiency of production by indicating the proportion of functional chips produced per wafer. The Defect Type column classifies manufacturing faults into categories like Particle contamination, Scratches, or Lithography errors — each of which could have distinct root causes and process implications. Additionally, two parametric test measurements are included: Vth (V), representing the transistor threshold voltage variation, and Leakage (nA), indicating the leakage current of devices. These electrical parameters are crucial for assessing process stability and device performance. By generating this synthetic dataset, the script provides a controlled yet realistic environment for yield and defect analysis without requiring access to sensitive, proprietary fab data.

### Physical Significance 

- The real physical significance of this kind of VLSI manufacturing yield and defect analysis lies in its ability to connect process data with actual device performance and manufacturing efficiency in a semiconductor fab.

- In real life, yield is a direct measure of how many chips per wafer are functional after fabrication. Low yield means more defective chips, which directly translates into higher production cost per chip. Defects such as particle contamination, scratches, or lithography misalignments have a clear physical cause — for example,
  - Particles may come from equipment wear or airborne contaminants.
  - Scratches could be due to mechanical handling damage.
  - Lithography defects might result from focus/exposure errors in photomask projection.
  - Parametric measurements like Vth (threshold voltage) and leakage current are critical because they are electrical indicators of transistor health. For example:
  - Vth variation can signal dopant fluctuation, oxide thickness variation, or process temperature drift.
  - Leakage current can point to gate oxide damage, junction leakage, or subthreshold conduction issues.

- By correlating defect type distribution, parametric scatter plots, and yield trends over time, engineers can pinpoint process problems early, link them to physical phenomena inside the fab, and take corrective action — such as tool maintenance, process tuning, or mask correction — before they cause large-scale yield loss.

- In short: This analysis bridges microscopic physical events in chip manufacturing with macroscopic business outcomes like production cost, product quality, and delivery timelines. It turns raw fab data into actionable insights that directly improve device reliability and profitability.























