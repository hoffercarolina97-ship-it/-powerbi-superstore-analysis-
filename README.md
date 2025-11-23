# Superstore Sales Analysis – Power BI Project 

## Project Overview 

This project demonstrates end-to-end data analysis using Power BI, including modeling, DAX, visualization design, and business insights.
It was developed as part of my Data Analyst portfolio to showcase strong analytical skills and dashboard design following best practices.

The final report **includes 5 pages**: a cover page, three analytical dashboards, and an insights/recommendations page.

---

# Project Structure 

## 1. Cover Page

- Project title

- Author: Carolina Ayelen Hoffer – Data Analyst

- Short description


## 2–4. Dashboard Pages 

- These pages include complete analyses of:

- Sales, Profit, Orders

- Category & Sub-category performance

- Regional & State analysis

- Monthly/Quarterly trends

- Cumulative metrics

- Customer behavior segmentation

- Top-performing and low-performing products


## 5. Insights & Recommendations 

A business-oriented summary with actionable insights.

---

# Data Cleaning & Preparation 

Although the Superstore dataset was relatively clean, a full data-quality audit was performed before building the data model. This process ensured that all fields were accurate, consistent, and ready for analysis.

### 1. Column Quality Review 

Enabled Column Quality and Column Distribution views in Power Query.

Verified that no columns contained:

- Null values

- Empty strings

- Unexpected outliers

- No missing values were detected.

### 2. Data Type Validation

Each field was reviewed and assigned the correct data type:

- Dates → Order Date, Ship Date

- Text categories → Category, Sub-Category, Customer Name, Region

- Numeric fields → Sales, Profit, Quantity, Discount

Correct data typing ensures proper aggregation, relationships, and DAX behavior.

### 3. Removal of Unnecessary Columns 

Columns not required for analysis were removed to improve:

- Model performance

- Data clarity

- DAX calculation speed

### 4. Column Renaming 

Column names were reviewed and standardized for better readability and consistency across the model.

### 5. Calculated Columns Created During Cleaning

A calculated column was created to support profitability analysis:

```DAX
Profit Margin % = DIVIDE('Superstore'[Profit], 'Superstore'[Sales])
```

### 6. Duplicate Checks

Verified that Order ID and Customer ID combinations did not contain duplicate entries.

Ensured that each order-line entry was unique.

### 7. Final Validation 

- Confirmed consistency between dates (no negative intervals, no invalid ship dates).

- Checked that discount values stayed within expected ranges (0–1).

- Ensured that profit values aligned with sales and discount logic.

Even though the dataset was clean, performing a structured data-quality review is essential in any analytics workflow to ensure the accuracy and reliability of the dashboard.

---

 # Data Modeling 

A dedicated Calendar table was created to enable accurate time-intelligence calculations.

## Calendar Table (DAX) 

```DAX
Calendar =
ADDCOLUMNS(
    CALENDAR( MIN('Superstore'[Order Date]), MAX('Superstore'[Order Date]) ),
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month Name", FORMAT([Date], "MMMM"),
    "Quarter", QUARTER([Date])
)
```

## Relationship used:

- **Calendar[Date] → Superstore[Order Date]** *(One-to-Many)*

This enables YoY analysis, cumulative metrics, and time slicing.

---

# DAX Measures Included 

## 1. Core KPI Measures 

```DAX
Total Sales = SUM('Superstore'[Sales])

Total Profit = SUM('Superstore'[Profit])

Total Orders = DISTINCTCOUNT('Superstore'[Order ID])

Total Customers = DISTINCTCOUNT('Superstore'[Customer ID])
```

---

## 2. Average Measures 

```DAX
Avg Sales per Customer = [Total Sales] / [Total Customers]

Avg Profit per Customer = [Total Profit] / [Total Customers]

Avg Discount % = AVERAGE('Superstore'[Discount])
```

---

## 3. Cumulative & Time Intelligence Measures 

```DAX
Cumulative Sales =
CALCULATE(
    [Total Sales],
    FILTER(
        ALL('Calendar'[Date]),
        'Calendar'[Date] <= MAX('Calendar'[Date])
    )
)

Sales LY =
CALCULATE(
    [Total Sales],
    SAMEPERIODLASTYEAR('Calendar'[Date])
)

YoY Sales Growth =
DIVIDE([Total Sales] - [Sales LY], [Sales LY])
```

---

## 4. Product Measures 

```DAX
Top Product Sales =
CALCULATE(
    [Total Sales],
    TOPN(1, 'Superstore', [Total Sales], DESC)
)
```

---

# Customer Analytics Measures (Frequency, Recency & Returning Customers) 

These measures were used for segmentation and customer-behavior visualizations.

## Frequency (Number of purchases per customer) 

```DAX
Frequency =
CALCULATE(
    COUNTROWS('Superstore'),
    FILTER('Superstore', 'Superstore'[Customer ID] = EARLIER('Superstore'[Customer ID]))
)
```

## Frequency Band

```DAX
Frequency Band =
SWITCH(
    TRUE(),
    [Frequency] <= 2, "Low (1–2)",
    [Frequency] <= 5, "Medium (3–5)",
    "High (6+)"
)
```

## Recency (Days since last purchase)

```DAX
Recency =
DATEDIFF(
    CALCULATE(MAX('Superstore'[Order Date]), ALLEXCEPT('Superstore', 'Superstore'[Customer ID])),
    MAX('Superstore'[Order Date]),
    DAY
)
```

## Recency Band 

```DAX
Recency Band =
SWITCH(
    TRUE(),
    [Recency] <= 30, "Hot (0–30)",
    [Recency] <= 90, "Warm (31–90)",
    "Cold (90+)"
)
```


## Scatter Discount (for scatter plots)

```DAX
Scatter Discount = AVERAGE('Superstore'[Discount])
```

## Returning Customers %

```DAX
Returning Customers % =
VAR CustomersThisPeriod =
    VALUES('Superstore'[Customer ID])
VAR CustomersBefore =
    CALCULATETABLE(
        VALUES('Superstore'[Customer ID]),
        DATEADD('Superstore'[Order Date], -1, YEAR)
    )
VAR Returning =
    INTERSECT(CustomersThisPeriod, CustomersBefore)
RETURN
    DIVIDE(COUNTROWS(Returning), COUNTROWS(CustomersThisPeriod))
```

---

# Screenshots Included

### File Naming:

01_Cover_Page.png
02_Sales_Overview.png
03_Category_Performance.png
04_Regional_Analysis.png
05_Insights_Recommendations.png

Stored in the /images/ folder.

---

# Repository Structure

```
/images
  01_Cover_Page.png
  02_Sales_Overview.png
  03_Category_Performance.png
  04_Regional_Analysis.png
  05_Insights_Recommendations.png


Superstore_Sales_Analysis.pbix
README.md
```

---

# Insights & Recommendations

## Key Insights

### Sales & Profitability 

- Technology is the highest-performing category, providing the strongest profit margins and stable year-over-year growth.

- Furniture shows the highest rate of negative profit, primarily driven by excessive discounting.

- The West region leads in overall sales performance, while the South underperforms across most categories.

- Approximately 18% of all products operate at a loss, mainly due to discount levels exceeding profitability thresholds.

### Customer Behavior (RFM Analysis)

- A large share of customers fall into the Cold – Low Frequency segment, indicating a retention opportunity.

- Corporate customers generate the highest average order value, while Consumer customers drive the highest sales volume.

- High-Frequency – Hot customers represent less than 10% of the customer base but contribute over 25% of total revenue.

- Customers with negative profit transactions are often associated with high-discount orders, especially in the Furniture category.

### Product Performance

- Top-selling items include Binders, Phones, Chairs, and Storage products.

- Sub-categories such as Tables and Bookcases show consistent negative profit trends across multiple states.

---

## Recommendations

### 1. Optimize Discount Strategy

- Reduce or cap discount levels in Furniture and Office Supplies, especially for loss-making products.

- Run A/B tests to determine profitable discount thresholds.

### 2. Reactivate Cold / Low-Frequency Customers 

- Launch automated campaigns with bundles or limited-time offers.

- Promote high-margin Technology products to encourage re-engagement.

### 3. Strengthen Corporate Customer Strategy

- Create premium bundles or incentives targeted at Corporate clients.

- Offer priority support and volume-based pricing programs.

### 4. Improve Inventory & Product Mix

- Reduce inventory of low-profit SKUs (Bookcases, specific Tables).

- Expand lines in Technology and Office Supplies, which show stronger and more consistent margins.

### 5. Future Analytical Improvements

- Build a predictive churn model using RFM metrics.

- Implement Customer Lifetime Value (CLV) scoring to prioritize marketing spend.

- Automate monthly refreshes using Power BI Service for continuous performance monitoring.

---

# Final Notes 
This project highlights:

- Data cleaning

- Data modeling 

- Advanced DAX

- Customer segmentation (RFM)  

- Visualization & UI/UX design  

- Actionable business insights  

It is part of my professional Data Analytics portfolio.
