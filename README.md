# 📉 Churn Analysis & Customer Intelligence

An end-to-end customer churn analysis for an **OTT (streaming) subscription platform**, built with **Python and SQL** on **10,000 customers**. The project answers three simple questions:

- **Who** is leaving (or about to leave)?
- **Why** are they leaving?
- **When and where** is the danger zone?

> 📄 A detailed, visual report is included: [`Churn_Analysis_Report_10000_Customers_v3.pdf`](Churn_Analysis_Report_10000_Customers_v3.pdf)

---

## 🎯 Business Problem

In the OTT market (Netflix, Hotstar, Prime and others), a customer can cancel with one click. Getting a new subscriber costs far more than keeping an existing one, so **retention** decides who wins.

This project identifies high-risk subscribers and the main drivers of churn using three data sources: customer demographics, subscription details and support interactions.

**Churn definition:** a customer is counted as churned when they have a cancellation date.

```
Churn rate = churned customers / total customers
```

---

## 🗂️ Dataset

Data comes from a SQLite database (`customer_churn`) with three related tables, joined on `customerid`:

| Table | What it contains |
|---|---|
| `db_customer` | Name, country, state, gender, date of birth |
| `db_subscription` | Start date, plan type (Basic / Standard / Premium), contract type (Monthly / Annual), joining channel, cancellation date and reason, monthly charges, CLTV, churn score |
| `db_support` | Complaint date, escalation (Y/N), CSAT score, complaint count |

The cleaned, final dataset (`exported_churn_data.csv`) has **10,000 customers** and 21 columns.

---

## 🛠️ Tech Stack

- **Python:** pandas, numpy
- **SQL:** SQLite (`sqlite3`)
- **Visualization:** matplotlib, seaborn
- **Environment:** Jupyter Notebook

---

## 🔄 Workflow

1. **Connect and extract:** connect Python to SQLite and pull all three tables with SQL queries.
2. **Clean the data:** fix data types, rename columns, remove empty columns (pincode, interests, col_1, comment), handle missing values, and keep only the latest complaint per customer so no customer is counted twice.
3. **Feature engineering:** create `churn_flag`, tenure, customer age and churn-risk tiers (Low below 50, Medium 50-69, High 70+).
4. **Analysis:** aggregations, group-bys and pivot tables to calculate 20+ KPIs.
5. **Visualization:** charts for churn by contract, plan, state, reason, year, age and risk tier.
6. **Reporting:** translate findings into business insights and an action plan.

---

## 📐 KPIs Calculated

| KPI | Formula |
|---|---|
| Churn rate | churned customers / total customers |
| Retention rate | 1 - churn rate |
| Churn by plan / contract / state | churn rate grouped by that field |
| ARPU | average monthly charge per customer |
| Average tenure | days from subscription start to cancellation (or to today if active) |
| Revenue lost to churn | sum of monthly charges of churned customers |
| CLTV lost | sum of CLTV of churned customers |
| Revenue at risk | monthly charges of active customers with Medium/High churn score |
| Escalation rate | escalated complaints / total complaints |
| Escalation vs churn | churn rate for escalated vs non-escalated customers |

---

## 📊 Key Insights

| # | Finding |
|---|---|
| 1 | **18.8% churn rate** (1,881 of 10,000 customers). Retention rate is 81.2%. |
| 2 | **Rs 44,805** monthly revenue lost (18.9%) and **Rs 13.6 lakh** CLTV lost (17.7%). |
| 3 | **Monthly contracts churn 1.6x more than annual** (23.7% vs 15.2%). The gap is the same in every plan. |
| 4 | **Plan type alone does not drive churn:** Basic 18.8%, Standard 19.0%, Premium 18.6%. |
| 5 | **Premium holds 64% of revenue lost**, even though it is only 33% of customers. |
| 6 | **31.9% left for a competitor.** With "too expensive" (17.9%), nearly half of cancellations are competitor or price related. |
| 7 | **Delhi (20.7%) and Karnataka (20.0%)** have the highest churn. The gap between states is small (about 4 points). |
| 8 | Cancellations **peaked in 2024** (419), then fell to 320 in 2025. |
| 9 | **Churn score works well:** Low risk churned 1.1%, Medium 51.7%, High 98.4% (correlation 0.79). |
| 10 | **553 active customers** are at Medium/High risk, carrying about **Rs 4.26 lakh CLTV**. |
| 11 | **Support is weak** (average CSAT 52/100), but escalations show almost no correlation with churn (0.006). |
| 12 | Churned customers stayed about **671 days** on average vs **1,967 days** for active customers. |

---

## ✅ Recommendations

1. Move monthly-contract customers to annual plans (discount or free month).
2. Contact the 553 at-risk active customers first, highest CLTV first.
3. Protect Premium and Standard customers before Basic.
4. Study competitor offers and review pricing and bundles.
5. Add more content and fix streaming quality.
6. Send reminders before trial ends and make the trial-to-paid step clear.
7. Resolve escalated support complaints faster.
8. Check Delhi and Karnataka for price, network or support issues.
9. Watch the first two years of a customer's life closely with onboarding offers.

---

## 📁 Project Structure

```
├── churn_analysis.ipynb                         # Full analysis notebook
├── exported_churn_data.csv                      # Cleaned final dataset (10,000 rows)
├── Churn_Analysis_Report_10000_Customers_v3.pdf # Visual report with insights
└── README.md
```

---

## ▶️ How to Run

1. Clone the repository
   ```bash
   git clone <your-repo-url>
   cd <your-repo-folder>
   ```
2. Install the required libraries
   ```bash
   pip install pandas numpy matplotlib seaborn jupyter
   ```
3. Open the notebook
   ```bash
   jupyter notebook churn_analysis.ipynb
   ```
4. Run all cells. To skip the SQL step, load the CSV directly:
   ```python
   import pandas as pd
   df = pd.read_csv("exported_churn_data.csv")
   ```

---

## ⚠️ Assumptions and Limitations

- The data does not specify a currency, so money values are shown as **"Rs"**.
- Tenure for active customers is calculated up to the date the analysis was run (26 Nov 2026), so tenure numbers change if you re-run the notebook on a different date.
- The **churn score** may have been updated after cancellation. Check how it is built before using it as a forecasting model.
- Only about 2% of customers (198) have a complaint record, so the support analysis is based on a small group.
- The action priority matrix in the report reflects the author's judgement, not a calculated score.

---

## 🚀 Possible Next Steps

- Build a predictive model (logistic regression or random forest) to forecast churn.
- Add survival analysis to find exactly when customers are most likely to leave.
- Build a dashboard (Power BI or Tableau) for live churn tracking.
- Test retention offers with an A/B experiment.

---

## 👤 Author

**[Your Name]**
Data Analyst
🔗 [LinkedIn](https://www.linkedin.com/) · 📧 your.email@example.com

⭐ If you found this project useful, please star the repository!
