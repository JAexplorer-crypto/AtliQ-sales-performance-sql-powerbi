# AtliQ-sales-performance-sql-powerbi
Sales Insights - A Data Analysis Project performed on Power BI &amp; SQL

<a href="https://www.linkedin.com/in/juhiajwani/" target="_blank" rel="noopener noreferrer" style="text-decoration: none; color: inherit; display: inline-flex; align-items: center;">
  <span style="text-decoration: underline;">Go to my LinkedIn</span>
  <span style="margin-left: 4 px;">🌐</span>
</a>
<br>
<br>

**🧩 Problem Statement**

AtliQ Hardware, a leading supplier of computer hardware and peripherals across India, is facing critical challenges in navigating a dynamically growing market. Despite having regional managers for North, South, and Central India, the Sales Director struggles with declining overall sales due to the absence of reliable, data-driven insights—current updates are shared verbally without supporting evidence, leading to frustration and ineffective decision-making. To address this, AtliQ Hardware requires a centralized, easily accessible data visualization solution that provides real-time, fact-based sales performance insights. This project aims to extract, clean, and wrangle (transform) data using SQL and Power Query, and to design and implement a Power BI sales dashboard that enables the Sales Director and leadership teams to make informed, data-driven decisions to drive business growth.


**🎯 AIMS Grid**
```
1. Purpose: To uncover actionable sales insights previously hidden from the sales team and provide automated reporting solutions
that reduce manual effort in data collection and transformation, enabling faster and more informed decision-making.

2. Stakeholders:
•	Sales Director
•	Marketing Team
•	Customer Service Team
•	Data and Analytics Team
•	IT Department

3. End Result: A fully automated Power BI dashboard delivering timely, accurate, and actionable sales insights, supporting data-driven
decision-making across key business units.

4. Success Criteria:
•	Dashboard reveals meaningful sales order insights using the most up-to-date data
•	Sales team demonstrates improved decision-making, achieving a 10% reduction in total operational spend by incorporating
        recommendations suggested
•	Manual data gathering is eliminated for sales analysis, saving 20% of business time, which is redirected to higher-value
        strategic activities.
```

**🛢️	Data Analysis using SQL:**

1.	Data Import and Environment Setup
   ```
  •	Load the provided MySQL dump file (ie. db_dump_version_2) into MySQL Workbench
  •	Establish database connections and verify the integrity of schema and table relationships
```

2. Data Quality Assessment
```
  •	Perform preliminary data exploration to identify anomalies:
            o	Market Table: Presence of incorrect or “garbage” entries
            o	Transaction Table: Negative sales_amount values, which are not logically valid
            o	Currency Issues: Presence of transactions in USD that need conversion to INR
```

3. Customer Data Analysis
```
  •	Extract all customer records:
        SELECT * FROM sales.customers;

  •	Calculate the total number of customers:
        SELECT COUNT(*) FROM sales.customers;
```

4. Market-Level Sales Analysis
```
  •	Query transactions by market_code (e.g., Mark001 for Chennai, Mark002 for Mumbai).
        SELECT * FROM sales.transactions where market_code='Mark001';

  •	Identify distinct products sold in each market:
        SELECT DISTINCT product_code FROM sales.transactions WHERE market_code='Mark001';
```

5. Currency-Based Filtering
```
  •	Identify transactions in USD:
        SELECT * FROM sales.transactions WHERE currency = 'USD';
```
6. Time-Based Transaction Analysis
```
  •	Join with sales.date table to filter data by year:
        SELECT t.*, d.* FROM sales.transactions t
        INNER JOIN sales.date d ON t.order_date = d.date
        WHERE d.year = 2020;
```
7. Revenue Aggregation Metrics
```
  •	Calculate total sales revenue:
      o	By year:
        SELECT SUM(transactions.sales_amount) FROM transactions
        INNER JOIN date ON transactions.order_date=date.date
        where date.year=2020 and transactions.currency="INR\r" or transactions.currency="USD\r";

      o	By year & month (e.g., January, February)
        SELECT SUM(transactions.sales_amount) FROM transactions INNER JOIN date ON
        transactions.order_date=date.date where date.year=2020 and and date.month_name="January"
        and (transactions.currency="INR\r" or transactions.currency="USD\r");

      o	By year & market using market codes (Mark001, Mark002, etc.)
        SELECT SUM(sales.transactions.sales_amount) FROM sales.transactions INNER JOIN sales.date ON
        sales.transactions.order_date=sales.date.date where sales.date.year=2020
        and sales.transactions.market_code="Mark002";
```
**🧹	Data Cleaning and ETL (Power Query) – SQL & Power BI Integration**

1.	 Connect to MySQL
```
  •	Establish a live connection from Power BI Desktop to the MySQL database using the built-in connector
```
2.	Load Data
```
  •	Import all relevant tables (transactions, market, customers, date, etc.) into the Power BI environment
  •	Verify the schema and inspect relationships in Model View to form a star schema with:
          o	Transactions (fact table)
          o	Market, Date, customers, Product (Dimension tables)
```
3.	Transform Data in Power Query
```
  •	Market Table Cleaning: Filter out rows with null or blank values in key columns

  •	Transactions Table Cleaning:
          o	Remove negative and zero sales amounts which are considered garbage values
          o	Filter out unwanted records via manual deselection or condition-based rules

  •	Currency Normalization:
        AtliQ operates only in India; convert all USD transactions to INR using a fixed exchange rate
        (e.g., 1 USD = ₹75).
          o	Create a new column using conditional logic:
                = Table.AddColumn(#"Filtered Rows", "norm_sales_amount", each if [currency] = "USD" or
                [currency] = "USD\cr"   then [sales_amount]*85 else [sales_amount])

•	Currency Data Deduplication:
        Identify duplicate currency values due to trailing characters in SQL workbench (e.g., "USD" vs "USD\ r").
          Query findings:
           o "INR\cr": ~150,000 records → Keep
           o "INR": ~279 records → Remove
           o "USD" & "USD\cr": Duplicate entries → Keep only "USD\cr"
        Final clean dataset retains records with currency codes: "INR\cr" and "USD\cr" only
```

**🧱	Data Modelling**
```
•	Created a star schema with:
        o	Transactions (fact table)
        o	Market, Date, customers, Product (Dimension tables)
```
![image](https://github.com/user-attachments/assets/cf703a53-4f06-49df-a852-66d754281512)


**🔣	Data Analysis (DAX):**

  Measures used in visualization are:
  ```
      •	Key Measures:
          1.	Total_Profit = sum('sales transactions (2)'[profit])
          2.	Total_Profit_Margin_% = DIVIDE([Total_Profit],[Revenue],0)
          3.	Profit_Contribution_% = DIVIDE(SUM('sales transactions (2)'[profit]),
                  CALCULATE([Total_Profit],ALL('sales products (2)'),all('sales customers (2)'),all('sales markets (2)')))
          4.	Revenue = SUM('sales transactions (2)'[sales_amount])
          5.	Revenue_Contribution_% = DIVIDE(SUM('sales transactions (2)'[sales_amount]),
                  CALCULATE([Revenue],ALL('sales products (2)'),all('sales customers (2)'),all('sales markets (2)')))
          6.	Revenue_LY = CALCULATE([Revenue],SAMEPERIODLASTYEAR('sales date (2)'[date]))
          7.	Sales_Qty = SUM('sales transactions (2)'[sales_qty])
```
```
      •	Profit Target:
          1.	Profit Target = GENERATESERIES(-0.05, 0.15, 0.01)
          2.	Profit_Target_Value = SELECTEDVALUE('Profit Target'[Profit Target])
          3.	Profit_Target_diff = [Total_Profit_Margin_%]-'Profit Target'[Profit_Target_Value]
```
**📈	Sales Report & Dashboard**
![img_AtliQ_Hardware_Sales_Dashboard](https://github.com/user-attachments/assets/539a765a-4c7a-4b01-bb63-8ff7c810347f)

**💡	Key Insights & Recommendations**
```
    A.	Market Performance Insights
          o	Mumbai & Delhi represents a scalable market with high sales volumes
          o	Bhubaneswar represents a high-efficiency market with optimized margin outcomes

       # Strategic implication:
          o	Scalable markets like Mumbai should undergo cost audits, pricing strategy evaluations, and logistics optimizations
          o	Efficient markets like Bhubaneswar could serve as for improving underperforming benchmark models or high-cost zones
```
```
  B.	Revenue Trend Insights
          o	Peak sales activity was observed in 2019
          o	Notable decline in early 2020(Q1 & Q2) – likely due to external factors (e.g., COVID impact)
          o	Consistent YoY underperformance in 2020 compared to 2019
      # Recommendations:
          o	Focus on pre-season marketing for Q4 (Oct–Dec)
          o	Rebuild pipeline earlier with sales promotions, bundling, or early bird offers
          o	Use historical high-performing months for flash campaigns or product launches
```
```
  C.	Customer Contribution Insights
        Top Customers by Revenue:
          o	Electricalsara Stores (₹413.35M) – accounts for ~42% of revenue, with 37.7% of profit
          o	Electricalslance and Electricallystical show negative or near-zero margins Profit Margins
        # Recommendation:
          o	Create customer tiers based on profitability
          o	Incentivize profitable customer segments with loyalty benefits
          o	Restrict discounts or renegotiate contracts with low-margin buyers
```
<a href="https://www.linkedin.com/in/juhiajwani/" target="_blank" rel="noopener noreferrer" style="text-decoration: none; color: inherit; display: inline-flex; align-items: center;">
  <span style="text-decoration: underline;">Go to my LinkedIn</span>
  <span style="margin-left: 4 px;">🌐</span>
</a>
<br>
<br>

