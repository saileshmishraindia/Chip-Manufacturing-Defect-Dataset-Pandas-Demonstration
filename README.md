# Wafer Test Data Analysis Dashboard using Pandas | PYTHON

## 📌 Table of Contents
1. [Introduction](#introduction)
2. [Data Loading & Cleaning](#data-loading--cleaning)
3. [Yield Calculations](#yield-calculations)
4. [Defect Analysis](#defect-analysis)
5. [Daily Yield Trend](#daily-yield-trend)
6. [Visualization Dashboard](#visualization-dashboard)
7. [Recommendations](#recommendations)
8. [Full Python Code](#full-python-code)

---

## 1. Introduction
This analysis processes **wafer test data** from semiconductor manufacturing to:
- **Clean** and standardize data.
- Calculate **yield** at different levels (Lot, Wafer, Product).
- Identify **top defect types**.
- Visualize trends and distributions via a **multi-plot dashboard**.
- Provide actionable **recommendations** for yield improvement.

---

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
