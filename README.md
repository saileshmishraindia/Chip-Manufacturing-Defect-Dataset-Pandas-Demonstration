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

## 6. Visulaization Dashboard Architecture

<img width="1920" height="1080" alt="Screenshot (1821)" src="https://github.com/user-attachments/assets/59063f3b-3e00-434a-a08f-129911e392d2" />

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

# 1️⃣ Bar Chart - Top 3 Defects
axs[0,0].bar(top_defects.index, top_defects.values, color='tomato')
axs[0,0].set_title("Top 3 Defects (Count)")
axs[0,0].set_ylabel("Count")
axs[0,0].grid(axis='y', linestyle='--', alpha=0.7)

# 2️⃣ Pie Chart - Defect Distribution
axs[0,1].pie(defect_counts.head(5), labels=defect_counts.head(5).index, autopct='%1.1f%%', startangle=90)
axs[0,1].set_title("Defect Type Distribution (Top 5)")

# 3️⃣ Histogram - Vth Distribution
axs[0,2].hist(df['Vth_mV'], bins=20, color='skyblue', edgecolor='black')
axs[0,2].set_title("Threshold Voltage Distribution")
axs[0,2].set_xlabel("Vth (mV)")
axs[0,2].set_ylabel("Frequency")

# 4️⃣ Line Graph - Daily Yield
axs[1,0].plot(daily_yield.index, daily_yield.values, marker='o', color='green')
axs[1,0].set_title("Daily Yield Trend")
axs[1,0].set_xlabel("Date")
axs[1,0].set_ylabel("Yield (%)")
axs[1,0].grid(True, linestyle='--', alpha=0.7)

# 5️⃣ Scatter Plot - Vth vs Leakage
axs[1,1].scatter(df['Vth_mV'], df['Leakage_nA'], c=(df['Test_Result']=='FAIL'), cmap='coolwarm', alpha=0.7)
axs[1,1].set_title("Parametric Distribution (PASS vs FAIL)")
axs[1,1].set_xlabel("Vth (mV)")
axs[1,1].set_ylabel("Leakage (nA)")

# 6️⃣ Bar Chart - Product Yield
axs[1,2].bar(product_yield.index, product_yield.values, color='purple')
axs[1,2].set_title("Yield by Product Type")
axs[1,2].set_ylabel("Yield (%)")
axs[1,2].tick_params(axis='x', rotation=45)

plt.tight_layout(rect=[0, 0, 1, 0.96])
plt.show()

# --- Recommendations ---
print("\nLot Yield Summary:\n", lot_yield)
print("\nTop 3 Defects:\n", top_defects)
print("\nRecommendation: Focus on reducing '{}' defects first.".format(top_defects.index[0]))
```
























