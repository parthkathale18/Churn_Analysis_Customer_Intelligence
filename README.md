# Churn Analysis & Customer Intelligence

An end-to-end churn analytics project for an **OTT (streaming) subscription platform**. It connects Python to a relational SQL database, cleans and prepares data from three tables, analyses churn across plans, contracts and regions, visualises the results, and turns the findings into actionable business recommendations.

The same approach applies to E-commerce, SaaS, Fintech and Adtech businesses.

---

## Table of Contents

1. [Business Problem](#1-business-problem)
2. [Tech Stack](#2-tech-stack)
3. [Project Workflow](#3-project-workflow)
4. [Database Schema](#4-database-schema)
5. [Step 1: Connecting the SQL Database to Python](#5-step-1-connecting-the-sql-database-to-python)
6. [Step 2: Importing Data with SQL Queries](#6-step-2-importing-data-with-sql-queries)
7. [Step 3: Data Cleaning](#7-step-3-data-cleaning)
8. [Step 4: Feature Engineering / Data Processing](#8-step-4-feature-engineering--data-processing)
9. [Step 5: Churn Analysis (EDA)](#9-step-5-churn-analysis-eda)
10. [Step 6: Data Visualization](#10-step-6-data-visualization)
11. [KPIs Calculated](#11-kpis-calculated)
12. [Results & Key Insights](#12-results--key-insights)
13. [Conclusion](#13-conclusion)
14. [Recommended Action Items / Next Steps](#14-recommended-action-items--next-steps)
15. [Resume-Ready Project Summary](#15-resume-ready-project-summary)

---

## 1. Business Problem

In the hyper-competitive OTT landscape (Netflix, Hotstar, Prime), retention is the only way to survive. As a Data Analyst, the task is to identify **high-risk subscribers** using a multi-dimensional dataset covering:

- Customer demographics
- Subscription tiers and contracts
- Support escalations

Churn analysis answers three questions:

| Question | Meaning |
|----------|---------|
| **Who** | Which customers left, or will leave in the future |
| **Why** | What their behaviour looked like before they left |
| **When** | The "danger zone": the point at which users leave |

### How churn is defined by business type

| Business Type | Churn Definition |
|---------------|------------------|
| SaaS | Subscription canceled |
| E-commerce | No purchase in 90 days |
| Adtech | Not using the service/app for 90 to 120 days |
| **Streaming (this project)** | **Membership inactive / cancelled** |
| Telecom | Account terminated |
| Banking | No transactions for X months |

---

## 2. Tech Stack

- **Language:** Python
- **Database:** SQLite (`customer_churn`)
- **Libraries:** sqlite3, pandas, numpy, matplotlib, seaborn
- **Skills covered:** SQL and Python integration, data cleaning, feature engineering, analytics, behavioural visualization, writing actionable insights

---

## 3. Project Workflow

| Stage | Tools | What happens |
|-------|-------|--------------|
| 1. Connect | sqlite3, pandas | Link the SQL database to Python |
| 2. Import | sqlite3, pandas | Pull multi-table data using SQL queries |
| 3. Clean | numpy, pandas | Fix data types, rename columns, select columns, run QC checks, handle missing/null values |
| 4. Feature Engineering | numpy, pandas | Create calculated columns, transform data, apply filters |
| 5. Analysis | numpy, pandas | Aggregation, group by, pivot tables |
| 6. Visualization | matplotlib, seaborn | Build behavioural and churn charts |
| 7. Insights | Presentation | Key insights, next steps, resume impact |

---

## 4. Database Schema

**Database:** `customer_churn`, with three tables linked by `customerid`.

### `db_customer`
| Column | Description |
|--------|-------------|
| customerid | Unique customer ID (key) |
| name | Customer name |
| country | Country |
| State | State |
| gender | Gender |
| dob | Date of birth |
| interests | Customer interests |
| pincode | Postal code |

### `db_subscription`
| Column | Description |
|--------|-------------|
| customerid | Unique customer ID (key) |
| subscription_start_date | Subscription start date |
| subscription_type | Subscription type |
| renewal_date | Renewal date |
| plan_type | Basic / Standard / Premium |
| contract_type | Monthly / Annual |
| cancellation_date | Date of cancellation (empty if still active) |
| cancellation_reason | Reason for cancelling |
| monthly_charges | Monthly charge |
| cltv | Customer lifetime value |
| churn_score | Predicted churn risk score |

### `db_support`
| Column | Description |
|--------|-------------|
| customerid | Unique customer ID (key) |
| complaint_date | Date of complaint |
| Escalations | Whether the complaint was escalated |
| csat_score | Customer satisfaction score |
| col_1 | Unnamed extra column (reviewed/renamed during cleaning) |
| comment | Free-text comment |

---

## 5. Step 1: Connecting the SQL Database to Python

The first step is to establish a connection between Python and the `customer_churn` SQL database using the **sqlite3** library, with **pandas** used to work with the data afterwards.

- Open a connection to the database file.
- Check that the connection works by listing the tables available in the database.
- Confirm that the three expected tables (`db_customer`, `db_subscription`, `db_support`) are present.

The connection stays open for the rest of the workflow and is closed once all data has been loaded.

---

## 6. Step 2: Importing Data with SQL Queries

Data is pulled from the database into pandas tables (DataFrames) using SQL queries.

- Each of the three tables is loaded separately so it can be checked on its own.
- The customer and subscription tables are joined on `customerid` to create a single customer-level view.
- Support records are summarised per customer before joining, so that customers with many complaints do not create duplicate rows.
- After import, the number of rows and columns in each table is checked to make sure nothing was lost.

---

## 7. Step 3: Data Cleaning

Cleaning makes sure the data is reliable before any analysis. The tasks are:

1. **Inspect the data:** review structure, data types, summary statistics, missing values and duplicates.
2. **Standardise column names:** convert to a consistent format (for example lower case, no extra spaces).
3. **Fix data types:** convert date fields (date of birth, subscription start, renewal, cancellation, complaint dates) to proper dates, and convert monthly charges, CLTV and churn score to numbers.
4. **Handle missing values:**
   - Empty cancellation date and cancellation reason are *expected* for active customers, so they are not errors. The reason is labelled "Not Applicable".
   - Genuinely missing numeric values such as monthly charges are filled with a sensible value such as the median.
5. **Remove duplicates:** keep one record per customer.
6. **Select relevant columns:** drop unused or unnamed columns such as `col_1`.

### Quality checks (QC)

- No duplicate customer IDs
- Cancellation dates are never earlier than subscription start dates
- No negative charges
- Category labels (plan type, contract type) are consistent

---

## 8. Step 4: Feature Engineering / Data Processing

New columns are created from the cleaned data to power the analysis:

| New feature | Purpose |
|-------------|---------|
| **Churn flag** | Marks a customer as churned (1) if a cancellation date exists, otherwise active (0) |
| **Tenure (days)** | Time between subscription start and cancellation date (or today's date if still active) |
| **Customer age** | Calculated from date of birth (customer aging) |
| **Cancellation month** | Used to find when churn peaks, the "danger zone" |
| **Risk tier** | Groups customers into Low / Medium / High churn risk using the churn score |
| **Total complaints** | Number of support complaints per customer |
| **Total escalations** | Number of escalated complaints per customer |
| **Average CSAT** | Average satisfaction score per customer |
| **Has escalation** | Simple yes/no flag for customers with one or more escalations |

The revenue-at-risk metric uses a churn score above 70 as the high-risk threshold. Risk-tier cut-offs should be adjusted to match your scoring scale.

---

## 9. Step 5: Churn Analysis (EDA)

Exploratory analysis uses aggregation, grouping and pivot tables to answer the "Who, Why and When" questions.

- **Overall churn and retention:** share of customers who left versus stayed.
- **Churn by plan type:** compares Basic, Standard and Premium.
- **Churn by contract type:** compares monthly and annual contracts.
- **Churn by location:** compares churn across countries and states to find hotspots.
- **Churn timing:** counts cancellations per month to find when churn peaks.
- **Revenue impact:** total revenue, revenue lost to churn, percentage revenue loss, CLTV lost and revenue at risk.
- **Customer value metrics:** ARPU and average tenure.
- **Support analysis:** escalation rate, average complaints per customer, and churn rate for customers with escalations versus without.
- **Pivot analysis:** churn rate across plan type and contract type together.
- **Cancellation reasons:** ranks why customers left (competitor switching, pricing, content dissatisfaction).

---

## 10. Step 6: Data Visualization

Charts are built with **matplotlib** and **seaborn** to make the patterns easy to see:

| Chart | What it shows |
|-------|---------------|
| Pie chart | Overall churn vs retention |
| Bar chart by plan type | Which plan loses the most customers |
| Bar chart by contract type | Monthly vs annual churn gap |
| Line chart by month | Churn trend over time and the peak month |
| Bar chart by state | Regions most affected by churn |
| Bar chart for escalations | Churn rate with and without escalations |
| Count chart by risk tier | How many customers are Low / Medium / High risk |
| Correlation heatmap | Relationships between charges, CLTV, churn score, tenure, complaints and churn |

---

## 11. KPIs Calculated

| KPI | Formula |
|-----|---------|
| Churn rate | Churned customers / Total customers |
| Retention rate | 1 - Churn rate |
| Churn by plan type | Churn rate grouped by plan type (Basic / Standard / Premium) |
| Churn by state | Churn rate grouped by country and state |
| ARPU | Sum of monthly charges / Count of active customers |
| Average customer tenure | Average days from subscription start to cancellation date (or today if active) |
| Revenue at risk | Sum of monthly charges where churn score > 70 |
| Escalation rate | Total escalations / Total complaints x 100 |
| Avg complaints per customer | Total complaints / Distinct customers |
| Correlation: escalations to churn | Churn rate where escalations >= 1 vs 0 |

---

## 12. Results & Key Insights

| Metric | Result |
|--------|--------|
| **Churn rate** | **28.6%** |
| **Retention rate** | **71.4%** |
| Average tenure | 1,451 days |
| ARPU | 18.8 |
| Total revenue | 395 |
| Revenue loss due to churn | 74 (about 73.94 per month) |
| % revenue loss | 18% |
| CLTV lost | 2,047 |
| Monthly-contract churn | 55.6% |
| Annual-contract churn | 8.3% (monthly is about **6.7x** higher) |
| Peak churn month | September 2024 |
| Most affected state | Karnataka |
| Plan with most churn | Basic |

### Key findings

- **Overall churn is 28.6%**, so roughly 3 in 10 customers have left; retention stands at 71.4%.
- **Contract type is the strongest churn driver.** Monthly-contract subscribers churn at 55.6% against 8.3% for annual subscribers.
- **Most churn comes from the Basic plan.** Because Basic is the lowest-priced tier, the direct revenue impact is limited (about 18% of revenue).
- **Churn spiked in September 2024**, and **Karnataka** is the most affected state. Both point to a possible trigger such as a price change, technical issue or service problem.
- **Revenue and CLTV erosion is concentrated in a small high-risk group.** About 74 in monthly revenue and 2,047 in CLTV were lost, attributed to six at-risk customers.
- **Support escalations are concentrated among churned customers**, making unresolved complaints a leading indicator of churn.
- **Cancellation drivers** break down into competitor switching, pricing sensitivity and content dissatisfaction. At least one user is known to have moved to a competitor.
- **Lifetime value gap:** churned cohorts show markedly lower lifetime value than retained ones, which supports prioritising Premium annual-plan retention over Basic monthly acquisition.

> **Note on currency:** the source analysis shows ARPU in Rs (INR) while the resume summary uses `$`. Standardise to a single currency before publishing the numbers.

---

## 13. Conclusion

The analysis shows a churn rate of **28.6%** (retention **71.4%**), with churn heavily skewed towards **monthly-contract and Basic-plan subscribers**. Moving customers from monthly to annual contracts is the single biggest retention lever, since annual subscribers churn at only 8.3%. Churn peaked in **September 2024** and is geographically concentrated in **Karnataka**, which suggests a specific event or regional issue that should be investigated. Support escalations act as an early-warning signal, so combining churn scores, CLTV and complaint history makes it possible to build a prioritised outreach list before customers leave.

---

## 14. Recommended Action Items / Next Steps

1. **Investigate Karnataka:** check for price increases, spikes in user complaints, technical or streaming issues, or regional competitor activity.
2. **Review September 2024:** find out whether the Basic plan price or features changed, or whether any other change was rolled out that month.
3. **Monitor competitors:** analyse competitor offers and promotions, since churned users are switching to them.
4. **Prioritise High and Medium churn-risk customers:** rank them by CLTV, review their complaint history, and reach out via email, SMS or calls to resolve their issues.
5. **Run a contract-migration campaign:** offer incentives to move monthly subscribers to annual plans.
6. **Improve support handling:** speed up resolution of escalated complaints and track CSAT after each interaction.
7. **Build a product and pricing roadmap** addressing the main cancellation drivers: pricing sensitivity, competitor switching and content dissatisfaction.

---

## 15. Resume-Ready Project Summary

**Churn & Revenue Impact**
> Engineered an end-to-end churn analytics pipeline for an OTT subscription dataset (20+ KPIs), identifying a 28.6% overall churn rate and surfacing that monthly-contract subscribers churned at 55.6%, which is 6.7 times the 8.3% annual-contract rate. Attributed 73.94/month in MRR leakage and 2,047 in CLTV erosion to six at-risk customers, enabling a targeted contract-migration retention strategy.

**Risk Scoring & Segmentation**
> Developed a multi-dimensional churn risk scoring model by synthesising subscription tenure, plan type and support escalation signals across three relational tables (20+ KPIs). Segmented the customer base into risk tiers using composite churn scores, exposed a significant lifetime value gap between churned and retained cohorts, and recommended prioritising Premium annual-plan retention over Basic monthly acquisition.

**Support Intelligence & Cancellation Analysis**
> Performed cross-functional support-churn correlation analysis by joining complaint, escalation and CSAT data with subscription records. Identified that escalated support interactions were disproportionately concentrated among churned customers, decomposed cancellation drivers into competitor switching, pricing sensitivity and content dissatisfaction, and translated the findings into a prioritised product and pricing roadmap for the growth team.
