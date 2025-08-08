# Wafer Test Data Analysis Dashboard using Pandas | PYTHON

## Table of Contents 
1. [Introduction](#introduction)
2. [Data Loading & Cleaning](#data-loading--cleaning)
3. [Yield Calculations](#yield-calculations)
4. [Defect Analysis](#defect-analysis)
5. [Daily Yield Trend](#daily-yield-trend)
6. [Visualization Dashboard](#visualization-dashboard)
7. [Recommendations](#recommendations)
8. [Full Python Code](#full-python-code)

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




























