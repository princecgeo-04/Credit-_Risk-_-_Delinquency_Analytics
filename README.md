# Credit Risk & Delinquency Analytics

An interactive **Power BI dashboard** that analyses 500 credit customers to show who is delinquent, where the risk is concentrated, and which customers to watch next.

![Executive overview](docs/images/page1_executive_overview.png)

## Table of contents

- [Project overview](#project-overview)
- [Key results](#key-results)
- [Dataset](#dataset)
- [Data preparation](#data-preparation)
- [Data model and DAX measures](#data-model-and-dax-measures)
- [Risk score](#risk-score)
- [Dashboard pages](#dashboard-pages)
- [Key findings](#key-findings)
- [Limitations](#limitations)
- [Recommendations and next steps](#recommendations-and-next-steps)
- [Tools used](#tools-used)
- [Repository structure](#repository-structure)
- [How to use](#how-to-use)
- [Author](#author)

## Project overview

When customers fall behind on payments, a lender loses income and must set money aside for possible losses. This project turns a customer-level dataset into a dashboard that answers four questions:

1. How many customers are delinquent, and how much money is at risk?
2. Which groups (card type, city, age, employment) have the highest delinquency rates?
3. Which behaviours warn of trouble?
4. Which customers should be watched first?

## Key results

| Metric | Value |
|---|---|
| Customers analysed | 500 |
| Delinquent customers | 80 (16.0%) |
| Total loan balance | about $23M |
| Loan balance in delinquent accounts | 15.4% |
| Customers rated High Risk | 198 (39.6%) |
| High Risk customers not yet delinquent | 171 |
| On-time payment rate (6 months) | 32.9% |
| Customers late or missed in Month 6 | 340 |

## Dataset

The source table is `Delinquency_prediction_dataset`: **500 rows (one per customer) and 19 columns**.

| Group | Columns |
|---|---|
| Customer profile | `Customer_ID`, `Age`, `Employment_Status`, `Location` |
| Financial | `Income`, `Loan_Balance`, `Debt_to_Income_Ratio` |
| Credit | `Credit_Score`, `Credit_Utilization`, `Credit_Card_Type`, `Account_Tenure` |
| Payment history | `Missed_Payments`, `Month_1` to `Month_6` (On-time, Late, Missed) |
| Target | `Delinquent_Account` (1 = delinquent, 0 = not delinquent) |

## Data preparation

The source data was left unchanged. All fixes are calculated columns in the model, and data issues are tracked by data quality measures.

- Standardised inconsistent employment labels (for example `EMP` and `employed` became **Employed**) in `Employment_Clean`
- Flagged **39** missing incomes (7.8%) and **29** missing loan balances (5.8%)
- Flagged **4** credit utilization values above 100%
- Created **10** calculated columns:

| Column | Purpose |
|---|---|
| `Employment_Clean` | Standardised employment status |
| `Age_Group` | 18-24, 25-34, 35-44, 45-54, 55-64, 65+ |
| `Credit_Score_Band` | Poor, Fair, Good, Very Good, Exceptional |
| `Income_Band` | Low, Middle, Upper-Middle, High, Unknown |
| `Utilization_Band` | Low (<30%), Medium, High, Very High (90%+) |
| `Delinquency_Status` | "Delinquent" or "Non-Delinquent" label |
| `Late_Months` / `Missed_Months` | Count of Late / Missed across Month 1 to Month 6 |
| `Risk_Score` | Rule-based score (see below) |
| `Risk_Category` | Low, Medium or High risk |

## Data model and DAX measures

All **39 measures** live in a dedicated `_Measures` table, organised into seven display folders.

| Folder | Count | Examples |
|---|---|---|
| Core KPIs | 7 | Total Customers, Delinquent Customers, Delinquency Rate %, Overall Delinquency Rate %, Rate vs Overall (pp) |
| Customer Profile | 7 | Avg Age, Avg Income, Avg Credit Score, Avg Credit Utilization %, Avg Account Tenure (Yrs) |
| Exposure | 4 | Total Loan Balance, Delinquent Loan Balance, Exposure at Risk % |
| Delinquent vs Non-Delinquent | 8 | Avg Credit Score, Income, Utilization and Missed Payments for each group |
| Risk Segmentation | 6 | High / Medium / Low Risk Customers, High Risk %, High Risk Not Yet Delinquent |
| Payment Behaviour | 4 | Late Payment Count (6M), Missed Payment Count (6M), On-Time Payment Rate % |
| Data Quality | 3 | Missing Income, Missing Loan Balance, Utilization Above 100% |

Example measure:

```dax
Delinquency Rate % =
DIVIDE ( [Delinquent Customers], [Total Customers], 0 )
```

## Risk score

`Risk_Score` adds points across five factors. The maximum score is **17**.

| Factor | Points rule | Max |
|---|---|---|
| Credit score | Below 580 = 3, 580 to 669 = 2, 670 to 739 = 1 | 3 |
| Credit utilization | 90% or more = 3, 60 to 89% = 2, 30 to 59% = 1 | 3 |
| Missed payments | 4 or more = 3, 2 to 3 = 2, 1 = 1 | 3 |
| Debt-to-income ratio | 0.50 or more = 2, 0.35 to 0.49 = 1 | 2 |
| 6-month history | 2 per missed month + 1 per late month, capped at 6 | 6 |

`Risk_Category`: **Low** (8 or less), **Medium** (9 to 11), **High** (12 or more).

> The weights are judgement-based and not learned from data. The score is a monitoring aid, not a validated prediction model.

## Dashboard pages

| Page | Purpose | Main visuals |
|---|---|---|
| 1. Executive Overview | Headline numbers and rates by group | KPI cards, donut by status, bars by card type, city and employment, slicers |

<img width="1548" height="868" alt="Screenshot 2026-11-18 050846" src="https://github.com/user-attachments/assets/27bb320a-c654-42a7-a436-36854c9cc1f8" />


| 2. Customer Profile | Who the customers are | KPI cards, income band chart, city by card matrix, employment treemap, age combo chart 

<img width="1546" height="872" alt="Screenshot 2026-11-18 050909" src="https://github.com/user-attachments/assets/159dcd19-11ac-45ee-bf56-cfd4340f8c06" />

| 3. Risk Drivers | Factors linked to delinquency | Credit score, utilization and missed payment charts, credit score vs utilization scatter |

|<img width="1551" height="872" alt="Screenshot 2026-11-18 050943" src="https://github.com/user-attachments/assets/a8840798-9b63-4b44-a411-4fc28aff9174" />
| 4. Payment Behaviour | Six months of payment history | On-time, late and missed KPIs, delinquency by late months and missed months |
<img width="1545" height="872" alt="Screenshot 2026-11-18 051010" src="https://github.com/user-attachments/assets/5a8ef506-69d0-4ddc-8424-95ba5b0928f6" />

| 5. Risk Monitoring | Who to watch first | High Risk KPIs, risk category combo chart, customer action table, risk by location |
<img width="1545" height="867" alt="Screenshot 2026-11-18 051033" src="https://github.com/user-attachments/assets/ed60696b-07da-4f02-8e10-797a8840af42" />



## Key findings

- Delinquency is **not evenly spread**. Business cards (21.3%), Los Angeles (19.6%) and ages 45 to 54 (20.8%) are well above the 16% average.
- The lowest rates are Platinum cards (11.8%), New York (12.0%) and ages 25 to 34 (9.6%).
- Unemployed customers have the highest rate by employment type, at about 19%.
- **High credit utilization** (60 to 90%) shows the highest rate, at about 19%, against about 12% for low utilization.
- Credit score and missed payments show only weak patterns, and delinquent and non-delinquent customers overlap across all scores.
- Only 32.9% of payments over six months were on time.
- 171 High Risk customers are not yet delinquent, which gives an early-warning list.

## Limitations

- Only 500 customers, so small groups can look extreme.
- One six-month window, so no trend over time can be shown.
- The rule-based risk score is not validated: the High Risk group (13.6% delinquent) is below Medium (17.9%).
- Missing income and loan balance values and four utilization values above 100% remain in the data.

## Recommendations and next steps

1. **Short term:** contact the 171 High Risk customers not yet delinquent, watch the 340 customers late or missed in Month 6, and review Business and Student card policies.
2. **Medium term:** fix missing and out-of-range data and standardise employment labels at source.
3. **Long term:** train a predictive model (logistic regression or a decision tree), compare it with the rule-based score, and refresh the dashboard monthly.

## Tools used

- Microsoft Power BI Desktop (data model, DAX measures, report design)
- DAX (calculated columns and measures)

## Repository structure

```
.
├── README.md
├── data/
│   └── Delinquency_prediction_dataset.csv
├── Credit_Risk_Delinquency_Analytics.pbix
├── docs/
│   ├── images/                  # dashboard screenshots used in this README
│   └── Presentation.pptx
```

## Author

**Prince George**, Data Analyst 

Email: prince.c.geo@gmail.com
