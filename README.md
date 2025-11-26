# Data-Analysis-Online-Retail-Dataset

## 📊 Project Overview
This project focuses on advanced retail data analysis using a transactional dataset from a UK-based online retailer (2009–2011). [cite_start]The core objective was to perform complex, multi-layered customer analysis.

The dashboard serves as a functional demo of expertise in creating a **Star Schema** and utilizing iterator functions (`SUMX`, `RANKX`, `LASTNONBLANK`) for high-value business metrics.

**Goal:** Provide actionable insights into customer loyalty, churn risk, and retention rates, demonstrating proficiency in Power BI modeling and advanced DAX.

---

## 💾 Repository Contents
* **Dataset.zip:** The cleaned transactional data file (included for reproducibility).
* **Online Retail data Analytics.pbix:** The final, complete Power BI report file.
* **README.md:** This documentation file.

---

## 🔍 Data Quality & Constraints
While the dataset nominally covers the period of 2009–2011, the analysis revealed specific data constraints that influenced the modeling strategy.

* **2009 Data Limitation:** The year 2009 contains only one month of sales data.
* **Activity Gaps:** The raw data presented several months without transactional activity.
* **Impact:** These constraints highlight the importance of robust data cleaning and the decision to focus heavily on **RFM (Recency, Frequency, Monetary)** segmentation, which remains effective even with irregular transactional timelines, rather than relying solely on time-series forecasting.

---

## 🛠️ Technical Deep Dive and Methodology

The foundation of this report is a clean and efficient **Star Schema** designed for optimal query performance.



### 1. Data Preparation and Modeling
The raw transactional data (Fact Sales table) was cleaned in Power Query and then structured into a Star Schema with dedicated dimension tables.

#### A. Star Schema Implementation
The model consists of a central fact table and multiple dimension tables linked via Many-to-One relationships.

| Table | Type | Creation Method | Key Relationship | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Clean Data 2009 - 2011** | Fact | Power Query | Many to Dim/RFM | Contains line-item sales data. |
| **Dim_Customer** | Dimension | DAX Calculated Table | One to Fact | Unique list of customers, anchors Cohort Date. |
| **Dim_Date** | Dimension | DAX Calculated Table | One to Fact | Continuous calendar used for Time Intelligence. |
| **RFM_Metrics** | Calculated | DAX Calculated Table | One to Dim_Customer | Contains R, F, M scores for segmentation. |

#### B. DAX Calculated Tables (Dimensions)
The core dimension tables were built using DAX:
* **Dim Customer:** Created using `SUMMARIZE` and `ADDCOLUMNS` to ensure one row per unique Customer ID.
* **Cohort Date:** The customer's acquisition date was fixed using a calculated column.

```DAX
Cohort Date = 
CALCULATE(
    MIN('Clean Data 2009 - 2011'[InvoiceDate]),
    ALLEXCEPT('Clean Data 2009 - 2011','Clean Data 2009 - 2011'[Customer ID]) 
)
```
### 2. Advanced DAX Expertise (Measures & Segmentation)
The complexity of the project is contained within the following advanced DAX measures:

### A. RFM Segmentation (RFM_Metrics Table)
RFM metrics (Recency, Frequency, Monetary) were calculated per customer and then scored using `RANKX`.


R, F, M Calculation: The RFM_Metrics table was generated using `SUMMARIZE` to aggregate sales data at the customer level.

Scoring (1-5): Scores were assigned using `RANKX`. The common pitfall of incorrect ranking context was solved by explicitly referencing the full table using ALL('RFM Metrics').

Code snippet
```DAX
R_Score =
5 - RANKX(
    ALL('RFM_Metrics'),
    'RFM_Metrics'[Recency],
    ,
    ASC,
    Dense
) + 1
```
Segmentation: A multi-layered `SWITCH(TRUE(), ...)` function was used to classify customers into actionable groups (e.g., Champions, At-Risk, Hibernating).

Code snippet
```DAX
RFM_Segment = 
SWITCH(
    TRUE(),
    -- 1. Champions
    'RFM_Metrics'[R_Score] >= 4 && 'RFM_Metrics'[F_Score] >= 4, "Champions",
    -- 2. Loyal Customers
    'RFM_Metrics'[R_SCore] >= 4 && 'RFM_Metrics'[F_Score] >= 2, "Loyal Customers",
    -- 3. Potential Loyalist
    'RFM_Metrics'[R_Score] >= 3 && 'RFM_Metrics'[F_Score] >= 3, "Potential Loyalist",
    -- 4. New Customers
    'RFM_Metrics'[R_Score] >= 4 && 'RFM_Metrics'[M_Score] <= 2, "New Customers",
    -- 5. At Risk
    'RFM_Metrics'[R_Score] <= 2 && 'RFM_Metrics'[F_Score] >= 3, "At Risk",
    -- 6. Can't Lose Them
    'RFM_Metrics'[R_Score] <= 2 && 'RFM_Metrics'[F_Score] <= 2, "Can't Lose Them",
    -- 7. Hibernating (catch remaining low recency)
    'RFM_Metrics'[R_Score] <= 3, "Hibernating",
    -- 8. Everything else
    "Other / Promising"
)
```
### B. Order Value

Invoice Total Value: A calculated column using `SUM(FILTER(EARLIER(...)))` was created on the Fact table to calculate the total transaction amount for a given InvoiceNo across all line items.

Code snippet
```DAX
TOTAL_SALES_PER_INV = 
CALCULATE(
    SUM('Clean Data 2009 - 2011'[Total Sales Amount]), 
    FILTER(
        'Clean Data 2009 - 2011',
        'Clean Data 2009 - 2011'[Invoice] = EARLIER('Clean Data 2009 - 2011'[Invoice])
    )
)
```
