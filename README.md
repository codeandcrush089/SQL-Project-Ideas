
# Project 1: **Retail Sales Performance Analysis (“Global Superstore”)**

### 1. Dataset

* **Name**: Global Superstore / Superstore Sales

* **Source**: Kaggle

* **Link(s)**:

  * “Superstore Sales Dataset” – Sales forecasting & general performance of global superstore. [Kaggle](https://www.kaggle.com/datasets/rohitsahoo/sales-forecasting?utm_source=chatgpt.com%20%22Superstore%20Sales%20Dataset%20-%20Kaggle%22)
  * “Superstore Dataset – Final” – includes information about orders, profit, region, etc. [Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final?utm_source=chatgpt.com%20%22Superstore%20Dataset%20-%20Kaggle%22)
  * “Global Superstore Dataset” – for geography, product, profit, etc. [Kaggle](https://www.kaggle.com/datasets/fatihilhan/global-superstore-dataset?utm_source=chatgpt.com%20%22Global%20Superstore%20Dataset%20-%20Kaggle%22)

* **Contents / Attributes**: Typical columns include:

  * Order ID, Order Date, Ship Date
  * Customer ID, Customer Segment, Region / State / City
  * Product ID, Product Category, Sub-Category, Product Name
  * Sales, Quantity, Discount, Profit
  * Ship Mode or Shipping Type

* **Time Period**: Usually multiple years of sales data (e.g. 4 years) ([Kaggle][1])

### 2. Project Objective

Analyze sales operations for the superstore to derive insights on performance, profitability, product success, customer behavior, and geographic trends. The goal is to support decision-making: which products to promote, where to open more stores, how to improve shipping / discount strategy, etc.


### 3. Skills Used

* SQL Joins (inner, left, etc.)
* Aggregations (SUM, AVG, COUNT)
* Group By, Having clauses
* Window Functions (ROW\_NUMBER, RANK, PARTITION BY, etc.)
* Date/time functions (extracting year, month, quarter; handling date differences)
* CASE statements (for conditional bucketing)
* Subqueries and Common Table Expressions (CTEs)
* Possibly user-defined functions or stored procedures (if scaling or repeated tasks)


### 4. Tasks / SQL Queries to Do

Here are concrete tasks / queries one can perform using this dataset. You can implement them step-by-step; combine for dashboard-type summaries too.

| S.No.  | Task Description                                   | Example SQL / Steps / Expected Output                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| -- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | **Total Sales, Profit, and Quantity Over Time**    | - Compute yearly & monthly SUM of *Sales*, *Profit*, *Quantity*. <br> - Trend line: are they increasing? <br> *Sample SQL:*<br> `sql SELECT EXTRACT(YEAR FROM OrderDate) AS Year, EXTRACT(MONTH FROM OrderDate) AS Month, SUM(Sales) AS TotalSales, SUM(Profit) AS TotalProfit, SUM(Quantity) AS TotalQty FROM Orders GROUP BY Year, Month ORDER BY Year, Month; `                                                                                                                |
| 2  | **Top-Performing Products**                        | - Find top 10 products by revenue (Sales) and by profit. <br> - Also top categories/sub-categories. <br> *Sample SQL:*<br> `sql SELECT ProductCategory, ProductSubCategory, ProductName, SUM(Sales) AS Revenue, SUM(Profit) AS Profits FROM Orders GROUP BY ProductCategory, ProductSubCategory, ProductName ORDER BY Revenue DESC LIMIT 10; `                                                                                                                                    |
| 3  | **Profitability by Region / State / City**         | - Compare profit margin (profit ÷ sales) by region. <br> - Find states which have high sales but low margins. <br> *Sample SQL:*<br> `sql SELECT Region, State, SUM(Sales) AS Sales, SUM(Profit) AS Profit, SUM(Profit)/NULLIF(SUM(Sales),0) AS ProfitMargin FROM Orders GROUP BY Region, State ORDER BY ProfitMargin DESC; `                                                                                                                                                     |
| 4  | **Effect of Discounts on Profit**                  | - For different discount buckets (e.g. 0-10%, 10-20%, >20%), compute average profit margin. <br> - Does higher discount always reduce profit? <br> *Sample SQL:*<br> `sql SELECT CASE WHEN Discount BETWEEN 0 AND 0.1 THEN '0-10%' WHEN Discount BETWEEN 0.1 AND 0.2 THEN '10-20%' ELSE '>20%' END AS DiscountBucket, AVG(Profit/Sales) AS AvgMargin, SUM(Sales) AS TotalSales FROM Orders GROUP BY DiscountBucket ORDER BY DiscountBucket; `                                     |
| 5  | **Customer Segmentation – Top & Repeat Customers** | - Identify which customers purchase most in terms of revenue. <br> - Which customers have repeat order frequency. <br> - Possibly compute “RFM” metrics: Recency, Frequency, Monetary. <br> *Sample SQL:*<br> `sql -- Example: frequency & monetary SELECT CustomerID, COUNT(DISTINCT OrderID) AS NumOrders, SUM(Sales) AS TotalSpend FROM Orders GROUP BY CustomerID ORDER BY TotalSpend DESC LIMIT 20; `                                                                        |
| 6  | **Seasonality Analysis**                           | - Are there months or quarters with consistent high/low sales? <br> - Compare e.g. quarter over quarter, year over year. <br> *Sample SQL:*<br> `sql SELECT EXTRACT(MONTH FROM OrderDate) AS Month, AVG(Sales) AS AvgSalesOverYears FROM Orders GROUP BY Month ORDER BY Month; `                                                                                                                                                                                                  |
| 7  | **Shipping / Delivery Performance**                | - Compute time between OrderDate and ShipDate (shipping lead time). <br> - Find average, median, and outliers. <br> - See if lead time varies by region or ship mode. <br> *Sample SQL:*<br> `sql SELECT ShipMode, Region, AVG(DATEDIFF(day, OrderDate, ShipDate)) AS AvgLeadTime FROM Orders GROUP BY ShipMode, Region ORDER BY AvgLeadTime; `                                                                                                                                   |
| 8  | **Profit Loss by Products (or negative profits)**  | - Find products or categories where profit is negative. <br> - Compute how much loss, possibly suggest discontinuation or cost-review. <br> *Sample SQL:*<br> `sql SELECT ProductName, SUM(Profit) AS TotalProfit FROM Orders GROUP BY ProductName HAVING SUM(Profit) < 0 ORDER BY TotalProfit; `                                                                                                                                                                                 |
| 9  | **Discount vs. Quantity sold**                     | - Do higher discounts lead to significantly more quantity sold? <br> - Is there a break-even point where discount hurts profit more than it drives sales? <br> *Sample SQL:*<br> `sql SELECT DiscountBucket, SUM(Quantity) AS TotalQty, SUM(Profit) AS TotalProfit FROM Orders GROUP BY DiscountBucket ORDER BY DiscountBucket; `                                                                                                                                                 |
| 10 | **Year-over-Year Growth**                          | - Compute YoY growth in Sales and Profit by Region, Category. <br> - Which region/category has highest growth? <br> *Sample SQL:*<br> `sql WITH Yearly AS ( SELECT EXTRACT(YEAR FROM OrderDate) AS Year, Region, SUM(Sales) AS Sales FROM Orders GROUP BY Year, Region ) SELECT y1.Region, y1.Year, y1.Sales, (y1.Sales - y0.Sales)/NULLIF(y0.Sales,0) AS YoY_Growth FROM Yearly y1 JOIN Yearly y0 ON y1.Region = y0.Region AND y1.Year = y0.Year + 1 ORDER BY Region, y1.Year; ` |


### 5. Additional / Advanced Tasks (for more real-world complexity)

* Build materialized views or summary tables for frequent queries (e.g. monthly summaries by region)
* Write stored procedures for repeating tasks: e.g. updating monthly profit & loss dashboards
* Use window functions for ranking: e.g., rank products per category by profit or sales
* Handle data quality issues: missing dates, unbalanced data, outliers (e.g. negative sales or weird discounts)
* Partitioning / indexing if dataset large → to speed up queries


### 6. Possible Dashboard or Reporting Outputs

* Monthly dashboard with “Sales vs Profit vs Quantity”
* Top N Products by Region / Category
* Heatmap of sales by state / city
* Discount strategy impact analysis
* Customer loyalty metrics


---

# 🍔 Project 2: **Online Food Delivery System (Swiggy/Zomato Style)**

## 1. Dataset

* **Name**: Online Food Ordering / Delivery Dataset

* **Sources (Kaggle)**:

  * [Swiggy Restaurant Dataset](https://www.kaggle.com/datasets/iamsouravbanerjee/indian-food-101?utm_source=chatgpt.com) *(restaurant & food data)*
  * [Zomato Restaurants Dataset](https://www.kaggle.com/datasets/shrutimehta/zomato-restaurants-data?utm_source=chatgpt.com) *(real-world restaurant attributes)*
  * [Online Food Orders Dataset](https://www.kaggle.com/datasets/benroshan/factors-affecting-campus-placement?utm_source=chatgpt.com) *(contains delivery, orders, ratings – can be adapted)*

* **Sample Columns**:

  * `OrderID`, `OrderDate`, `CustomerID`, `RestaurantID`, `DeliveryPartnerID`
  * `FoodItem`, `Category`, `Cuisine`, `Price`, `Quantity`
  * `PaymentMode` (Cash, Card, Wallet, UPI)
  * `Rating`, `DeliveryTimeMinutes`, `OrderStatus` (Delivered, Cancelled)
  * `City`, `Area`, `RestaurantRating`


## 2. Project Objective

Analyze and optimize the performance of an **online food delivery platform** (like Swiggy/Zomato).
Goals:

* Track **customer ordering behavior**
* Monitor **restaurant performance**
* Evaluate **delivery efficiency**
* Support **marketing and growth decisions**


## 3. Skills Used

* **Joins** → Orders + Customers + Restaurants + Delivery Partners
* **Aggregations** → SUM, AVG, COUNT
* **Window Functions** → Ranking top restaurants & delivery partners
* **Date Functions** → Peak order times, daily/weekly analysis
* **Case Statements** → Categorize delivery performance, payment modes
* **CTEs & Subqueries** → Reusable order summaries, churn analysis
* **Pattern Matching (LIKE)** → Search cuisines or food items


## 4. Tasks / SQL Queries

| S.No.  | Task                                 | Description & SQL Idea                                                                                                                                                                                                                                                                                                                     |
| -- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1  | **Total Orders & Revenue Trend**     | Track daily, weekly, monthly order counts & total revenue. <br> `sql SELECT DATE(OrderDate) AS OrderDay, COUNT(OrderID) AS TotalOrders, SUM(Price*Quantity) AS Revenue FROM Orders GROUP BY OrderDay ORDER BY OrderDay; `                                                                                                                  |
| 2  | **Top Restaurants by Revenue**       | Find top 10 restaurants that generate the most revenue. <br> `sql SELECT r.RestaurantName, SUM(o.Price*o.Quantity) AS Revenue FROM Orders o JOIN Restaurants r ON o.RestaurantID=r.RestaurantID GROUP BY r.RestaurantName ORDER BY Revenue DESC LIMIT 10; `                                                                                |
| 3  | **Most Popular Cuisines**            | Rank cuisines by number of orders. <br> `sql SELECT Cuisine, COUNT(*) AS Orders FROM Orders GROUP BY Cuisine ORDER BY Orders DESC; `                                                                                                                                                                                                       |
| 4  | **Customer Loyalty / Repeat Orders** | Identify customers with repeat orders (frequency analysis). <br> `sql SELECT CustomerID, COUNT(DISTINCT OrderID) AS NumOrders FROM Orders GROUP BY CustomerID HAVING NumOrders > 1 ORDER BY NumOrders DESC; `                                                                                                                              |
| 5  | **Delivery Partner Performance**     | Average delivery time & total orders per delivery partner. <br> `sql SELECT d.DeliveryPartnerID, d.Name, COUNT(o.OrderID) AS TotalOrders, AVG(o.DeliveryTimeMinutes) AS AvgDeliveryTime FROM Orders o JOIN DeliveryPartners d ON o.DeliveryPartnerID=d.DeliveryPartnerID GROUP BY d.DeliveryPartnerID, d.Name ORDER BY TotalOrders DESC; ` |
| 6  | **Late Deliveries %**                | Calculate percentage of late deliveries (> 40 mins). <br> `sql SELECT COUNT(CASE WHEN DeliveryTimeMinutes > 40 THEN 1 END)*100.0/COUNT(*) AS LateDeliveryPercent FROM Orders; `                                                                                                                                                            |
| 7  | **Average Order Value (AOV)**        | How much does a customer spend per order? <br> `sql SELECT AVG(OrderTotal) AS AvgOrderValue FROM (SELECT OrderID, SUM(Price*Quantity) AS OrderTotal FROM Orders GROUP BY OrderID) t; `                                                                                                                                                     |
| 8  | **Peak Ordering Hours**              | Find hours of the day with max orders. <br> `sql SELECT EXTRACT(HOUR FROM OrderDate) AS Hour, COUNT(OrderID) AS Orders FROM Orders GROUP BY Hour ORDER BY Orders DESC; `                                                                                                                                                                   |
| 9  | **Payment Mode Analysis**            | Popularity of payment methods. <br> `sql SELECT PaymentMode, COUNT(*) AS TotalOrders, SUM(Price*Quantity) AS Revenue FROM Orders GROUP BY PaymentMode ORDER BY TotalOrders DESC; `                                                                                                                                                         |
| 10 | **Cancelled Orders Impact**          | Revenue lost due to cancellations. <br> `sql SELECT SUM(Price*Quantity) AS CancelledRevenue FROM Orders WHERE OrderStatus='Cancelled'; `                                                                                                                                                                                                   |


## 5. Advanced Tasks

* **Churn Analysis** → Find customers who ordered last month but not this month
* **Restaurant Rating vs. Orders** → Correlation between rating & order volume
* **Discount Impact** → Do discounts increase order count but reduce profit margin?
* **Geo Analysis** → Most popular areas / cities for food delivery
* **Cohort Analysis** → First-time users vs. repeat users over time


## 6. Possible Dashboards / Reports

* 📊 **Revenue & Orders Trend** by day/week/month
* 🥗 **Top Cuisines & Restaurants** (heatmap or leaderboard)
* 🚴 **Delivery Partner Performance** (avg time, success rate)
* 💳 **Payment Method Popularity** (Cash vs UPI vs Wallet)
* 📉 **Cancelled Orders Impact** (financial loss & reasons)


⚡ This project shows **business + SQL insights** → directly applicable to **Swiggy, Zomato, Uber Eats** type companies.

---

# 💳 Project 3: **Banking Transactions Fraud Detection**


## 1. Dataset

* **Possible Sources**:

  * [Credit Card Fraud Detection (Kaggle)](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud?utm_source=chatgpt.com)
  * [Synthetic Financial Datasets for Fraud Detection (IEEE-CIS)](https://www.kaggle.com/c/ieee-fraud-detection?utm_source=chatgpt.com)
  * [Bank Transactions Dataset (Kaggle)](https://www.kaggle.com/datasets/ealaxi/paysim1?utm_source=chatgpt.com)

* **Sample Columns**:

  * `TransactionID`, `CustomerID`, `AccountID`, `TransactionDate`
  * `TransactionType` (POS, ATM, Online, Transfer)
  * `Amount`, `BalanceBefore`, `BalanceAfter`
  * `MerchantID`, `Location`
  * `IsFraud` (0/1)


## 2. Project Objective

Detect **suspicious or fraudulent transactions** using SQL analysis.

Goals:

* Identify **unusual transaction patterns**
* Spot **duplicate/rapid transactions**
* Detect **sudden spikes in transaction amounts**
* Classify **high-risk customers/merchants**


## 3. Skills Used

* **Joins** → Customers, Transactions, Merchants
* **Window Functions** → Detect duplicate/rapid transactions
* **Aggregations** → Count transactions per user/day
* **Case Statements** → Fraud flags (e.g., > threshold amount)
* **Date Functions** → Time gap analysis
* **CTEs & Subqueries** → Unusual behavior detection
* **Pattern Matching** → Transactions with risky keywords (e.g., gambling)


## 4. Tasks / SQL Queries

| S.No.  | Task                                    | Description & SQL Idea                                                                                                                                                                                                                                                                                                                                                                          |
| -- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | **High-Value Transactions**             | Flag unusually large transactions. <br> `sql SELECT TransactionID, CustomerID, Amount FROM Transactions WHERE Amount > 10000; `                                                                                                                                                                                                                                                                 |
| 2  | **Multiple Transactions in Short Time** | Same customer, same merchant, within 5 min. <br> `sql SELECT CustomerID, MerchantID, COUNT(*) AS TxnCount FROM Transactions WHERE TransactionDate > CURRENT_TIMESTAMP - INTERVAL '5 MINUTES' GROUP BY CustomerID, MerchantID HAVING COUNT(*) > 3; `                                                                                                                                             |
| 3  | **Balance Inconsistency**               | Check mismatch between balances. <br> `sql SELECT TransactionID, CustomerID FROM Transactions WHERE BalanceBefore - Amount != BalanceAfter; `                                                                                                                                                                                                                                                   |
| 4  | **Rapid Transfers Across Accounts**     | Multiple transfers by one user in <30 min. <br> `sql SELECT CustomerID, COUNT(*) AS Transfers FROM Transactions WHERE TransactionType='Transfer' AND TransactionDate > CURRENT_TIMESTAMP - INTERVAL '30 MINUTES' GROUP BY CustomerID HAVING COUNT(*) > 5; `                                                                                                                                     |
| 5  | **Transactions at Odd Hours**           | Unusual late-night activity. <br> `sql SELECT TransactionID, CustomerID, Amount, EXTRACT(HOUR FROM TransactionDate) AS Hr FROM Transactions WHERE Hr BETWEEN 0 AND 4; `                                                                                                                                                                                                                         |
| 6  | **High-Risk Merchants**                 | Merchants with many fraud cases. <br> `sql SELECT MerchantID, COUNT(*) AS FraudCases FROM Transactions WHERE IsFraud=1 GROUP BY MerchantID ORDER BY FraudCases DESC; `                                                                                                                                                                                                                          |
| 7  | **Duplicate Transactions**              | Same amount + same merchant within 1 min. <br> `sql SELECT CustomerID, Amount, MerchantID, COUNT(*) FROM Transactions GROUP BY CustomerID, Amount, MerchantID, DATE_TRUNC('minute', TransactionDate) HAVING COUNT(*) > 1; `                                                                                                                                                                     |
| 8  | **Location Mismatch**                   | Two transactions far apart in short time (geo-fraud). <br> `sql SELECT t1.CustomerID, t1.Location AS Loc1, t2.Location AS Loc2, t1.TransactionDate, t2.TransactionDate FROM Transactions t1 JOIN Transactions t2 ON t1.CustomerID=t2.CustomerID WHERE t1.TransactionDate < t2.TransactionDate AND t2.TransactionDate - t1.TransactionDate < INTERVAL '1 HOUR' AND t1.Location != t2.Location; ` |
| 9  | **Daily Transaction Limits**            | Customers exceeding normal patterns. <br> `sql SELECT CustomerID, SUM(Amount) AS DailyTotal FROM Transactions WHERE DATE(TransactionDate)=CURRENT_DATE GROUP BY CustomerID HAVING SUM(Amount) > 20000; `                                                                                                                                                                                        |
| 10 | **Fraud Detection Rate**                | Fraud % by transaction type. <br> `sql SELECT TransactionType, COUNT(CASE WHEN IsFraud=1 THEN 1 END)*100.0/COUNT(*) AS FraudRate FROM Transactions GROUP BY TransactionType; `                                                                                                                                                                                                                  |


## 5. Advanced Tasks

* **Anomaly Detection (SQL-only)** → Compare current transaction to user’s historical average.
* **Velocity Checks** → Too many transactions in short intervals.
* **Merchant Risk Scoring** → Weighted fraud scores for merchants.
* **Customer Segmentation** → Flag high-risk customers.
* **Cross-Border Fraud** → Same user transacting in multiple countries in <1 hr.


## 6. Dashboards / Reports

* 📊 **Fraud Trends**: daily fraud %, type of fraud, affected regions
* 🏦 **High-Risk Customers**: flagged users with abnormal activity
* 🛒 **Merchant Risk Report**: top merchants linked with fraud
* ⏱ **Velocity Alerts**: unusual transaction bursts
* 🌍 **Geo-Fraud Map**: location mismatch & cross-border activity


⚡ This project is **very practical in BFSI (Banking, Financial Services, Insurance)** and commonly used in fraud prevention teams at **PayPal, Mastercard, Visa, HDFC, ICICI, American Express**.

---


# 🛍️ Project 4: **E-Commerce Customer Segmentation**


## 1. Dataset

* **Possible Sources**

  * [E-Commerce Customer Dataset (Kaggle)](https://www.kaggle.com/datasets/carrie1/ecommerce-data?utm_source=chatgpt.com)
  * [Retail Rocket Recommender Dataset (Kaggle)](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset?utm_source=chatgpt.com)
  * Any transactional dataset with invoices & customers.

* **Sample Columns**

  * `CustomerID`, `InvoiceNo`, `InvoiceDate`
  * `ProductID`, `ProductCategory`, `Quantity`, `UnitPrice`
  * `Country`, `PaymentMethod`
  * `TotalAmount` (Quantity × UnitPrice)


## 2. Project Objective

Segment customers based on **purchase behavior** to:

* Identify **loyal customers vs. one-time buyers**
* Detect **high-value customers (VIPs)**
* Spot **churn-risk customers**
* Enable **personalized marketing & offers**


## 3. Skills Used

* **Aggregations** → Total spend, order frequency
* **Window Functions** → Ranking customers by value
* **Date Functions** → Recency (last purchase date)
* **Case Statements** → Customer segmentation logic
* **Joins** → Orders + Customers + Products
* **CTEs** → Build RFM models (Recency, Frequency, Monetary)


## 4. Tasks / SQL Queries

| S.No.  | Task                                                    | Description & SQL Idea                                                                                                                                                                                                                                                                      |
| -- | ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | **Total Spend per Customer**                            | `sql SELECT CustomerID, SUM(TotalAmount) AS TotalSpend FROM Orders GROUP BY CustomerID ORDER BY TotalSpend DESC; `                                                                                                                                                                          |
| 2  | **Customer Frequency (No. of Orders)**                  | `sql SELECT CustomerID, COUNT(DISTINCT InvoiceNo) AS Orders FROM Orders GROUP BY CustomerID; `                                                                                                                                                                                              |
| 3  | **Recency (Last Purchase Date)**                        | `sql SELECT CustomerID, MAX(InvoiceDate) AS LastPurchase FROM Orders GROUP BY CustomerID; `                                                                                                                                                                                                 |
| 4  | **Average Order Value (AOV)**                           | `sql SELECT CustomerID, AVG(TotalAmount) AS AvgOrderValue FROM Orders GROUP BY CustomerID; `                                                                                                                                                                                                |
| 5  | **Top 10 High-Value Customers**                         | `sql SELECT CustomerID, SUM(TotalAmount) AS TotalSpend FROM Orders GROUP BY CustomerID ORDER BY TotalSpend DESC LIMIT 10; `                                                                                                                                                                 |
| 6  | **Country-wise Customer Segments**                      | `sql SELECT Country, COUNT(DISTINCT CustomerID) AS Customers, SUM(TotalAmount) AS Revenue FROM Orders GROUP BY Country ORDER BY Revenue DESC; `                                                                                                                                             |
| 7  | **Identify One-Time Buyers**                            | `sql SELECT CustomerID FROM Orders GROUP BY CustomerID HAVING COUNT(DISTINCT InvoiceNo)=1; `                                                                                                                                                                                                |
| 8  | **Churn Risk Customers** (no purchase in last 6 months) | `sql SELECT CustomerID FROM Orders GROUP BY CustomerID HAVING MAX(InvoiceDate) < CURRENT_DATE - INTERVAL '6 MONTHS'; `                                                                                                                                                                      |
| 9  | **RFM Model (Recency, Frequency, Monetary)**            | `sql WITH rfm AS ( SELECT CustomerID, MAX(InvoiceDate) AS LastPurchase, COUNT(DISTINCT InvoiceNo) AS Frequency, SUM(TotalAmount) AS Monetary FROM Orders GROUP BY CustomerID ) SELECT CustomerID, DATE_PART('day', CURRENT_DATE - LastPurchase) AS Recency, Frequency, Monetary FROM rfm; ` |
| 10 | **Customer Segmentation with CASE**                     | `sql SELECT CustomerID, CASE WHEN SUM(TotalAmount) > 10000 THEN 'VIP' WHEN COUNT(DISTINCT InvoiceNo) > 20 THEN 'Frequent Buyer' ELSE 'Occasional Buyer' END AS Segment FROM Orders GROUP BY CustomerID; `                                                                                   |


## 5. Advanced Tasks

* **CLV (Customer Lifetime Value)** → Predict long-term value of each customer.
* **Market Basket Analysis** → Which products are often bought together.
* **Churn Probability** → Customers inactive for X days.
* **Cohort Analysis** → Retention by join month.
* **Seasonality** → Peak purchase times & festive trends.


## 6. Dashboards / Reports

* 👥 **Customer Segments**: VIPs, regulars, one-time buyers
* 💰 **Revenue by Segment**: Contribution % of each segment
* 📅 **Recency Heatmap**: Active vs. inactive customers
* 🌍 **Geographic Trends**: Customers by country/region
* 🛒 **Product Affinity**: Frequently bought-together items


📌 This project is widely used in **E-Commerce & Retail (Amazon, Flipkart, Walmart, Myntra, Shopify sellers)** to improve **customer retention & marketing ROI**.

---

# 🏥 Project 5: **Healthcare Patient Records Analysis**


## 1. Dataset

* **Name**: Hospital / Healthcare Patient Records

* **Sources (Kaggle & Open Data)**:

  * [Hospital General Information Dataset (US Hospitals)](https://www.kaggle.com/datasets/cms/hospital-general-information?utm_source=chatgpt.com)
  * [MIMIC-III Clinical Database (MIT)](https://physionet.org/content/mimiciii/1.4/?utm_source=chatgpt.com) *(requires credentialed access – real ICU records)*
  * [Diabetes 130-US hospitals for years 1999–2008](https://www.kaggle.com/datasets/brandao/diabetes?utm_source=chatgpt.com) *(patient admission & readmission data)*

* **Sample Columns** (combined schema idea):

  * `PatientID`, `Gender`, `Age`, `AdmissionDate`, `DischargeDate`, `Diagnosis`, `Treatment`, `Medication`
  * `DoctorID`, `Department`, `HospitalID`
  * `LengthOfStay`, `ReadmissionFlag`, `Outcome` (Recovered / Expired)
  * `BillingAmount`, `InsuranceProvider`, `PaymentStatus`


## 2. Project Objective

Analyze **patient records** to improve hospital operations, patient outcomes, and financial efficiency.
Goals:

* Track **average length of stay** & readmission rates
* Identify **top diseases & treatments**
* Monitor **doctor/department performance**
* Analyze **hospital billing & insurance claims**


## 3. Skills Used

* **Joins** → Patients, Admissions, Doctors, Departments
* **Aggregations** → AVG, SUM, COUNT
* **Window Functions** → Ranking doctors/departments
* **Date Functions** → Stay duration, readmission time gap
* **Case Statements** → Categorize patient age groups, outcomes
* **CTEs & Subqueries** → Summaries for readmissions, mortality rates
* **Conditional Joins** → Compare admissions/discharges for readmission tracking


## 4. Tasks / SQL Queries

| S.No.  | Task                             | Description & SQL Idea                                                                                                                                                                                                                                                                                                                       |
| -- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | **Average Length of Stay (LOS)** | Measure avg days patients stay in hospital. <br> `sql SELECT AVG(DATEDIFF(day, AdmissionDate, DischargeDate)) AS AvgStay FROM Admissions; `                                                                                                                                                                                                  |
| 2  | **Top Diagnoses**                | Find most common diagnoses (disease frequency). <br> `sql SELECT Diagnosis, COUNT(*) AS PatientCount FROM Admissions GROUP BY Diagnosis ORDER BY PatientCount DESC LIMIT 10; `                                                                                                                                                               |
| 3  | **Readmission Rate**             | Percentage of patients readmitted within 30 days. <br> `sql SELECT COUNT(*)*100.0/(SELECT COUNT(*) FROM Admissions) AS ReadmissionRate FROM Admissions a1 JOIN Admissions a2 ON a1.PatientID=a2.PatientID AND a2.AdmissionDate BETWEEN a1.DischargeDate AND DATEADD(day,30,a1.DischargeDate); `                                              |
| 4  | **Department Workload**          | Total patients treated per department. <br> `sql SELECT Department, COUNT(DISTINCT PatientID) AS TotalPatients FROM Admissions GROUP BY Department ORDER BY TotalPatients DESC; `                                                                                                                                                            |
| 5  | **Doctor Performance**           | Avg patient satisfaction rating or recovery outcomes per doctor. <br> `sql SELECT d.DoctorID, d.Name, COUNT(a.PatientID) AS Cases, SUM(CASE WHEN a.Outcome='Recovered' THEN 1 ELSE 0 END)*100.0/COUNT(*) AS RecoveryRate FROM Admissions a JOIN Doctors d ON a.DoctorID=d.DoctorID GROUP BY d.DoctorID, d.Name ORDER BY RecoveryRate DESC; ` |
| 6  | **Mortality Rate**               | Calculate percentage of expired patients. <br> `sql SELECT SUM(CASE WHEN Outcome='Expired' THEN 1 ELSE 0 END)*100.0/COUNT(*) AS MortalityRate FROM Admissions; `                                                                                                                                                                             |
| 7  | **Age Group Analysis**           | Which age group has the highest admissions? <br> `sql SELECT CASE WHEN Age < 18 THEN 'Child' WHEN Age BETWEEN 18 AND 40 THEN 'Young Adult' WHEN Age BETWEEN 41 AND 60 THEN 'Middle Age' ELSE 'Senior' END AS AgeGroup, COUNT(*) AS Patients FROM Patients GROUP BY AgeGroup ORDER BY Patients DESC; `                                        |
| 8  | **Insurance Claims Analysis**    | Amount billed vs paid by insurance. <br> `sql SELECT InsuranceProvider, SUM(BillingAmount) AS TotalBilled, SUM(CASE WHEN PaymentStatus='Paid' THEN BillingAmount ELSE 0 END) AS TotalPaid FROM Billing GROUP BY InsuranceProvider ORDER BY TotalBilled DESC; `                                                                               |
| 9  | **Readmission by Disease**       | Which diagnoses have highest 30-day readmission rate. <br> `sql SELECT Diagnosis, COUNT(*) AS Readmissions FROM Admissions a1 JOIN Admissions a2 ON a1.PatientID=a2.PatientID AND a2.AdmissionDate BETWEEN a1.DischargeDate AND DATEADD(day,30,a1.DischargeDate) GROUP BY Diagnosis ORDER BY Readmissions DESC; `                            |
| 10 | **Revenue by Department**        | Total billing revenue generated by department. <br> `sql SELECT Department, SUM(BillingAmount) AS Revenue FROM Admissions GROUP BY Department ORDER BY Revenue DESC; `                                                                                                                                                                       |


## 5. Advanced Tasks

* **Cohort Survival Analysis** → Track recovery/mortality rate by admission month
* **Emergency vs Scheduled Admissions** → Compare average length of stay & outcomes
* **Predictive Query Prep** → Identify high-risk patients for readmission (using thresholds in SQL)
* **Trend Analysis** → Year-over-year admissions, seasonal disease spikes (e.g., flu in winter)


## 6. Possible Dashboards / Reports

* 📊 **Hospital KPIs**: Avg stay, readmission rate, mortality rate
* 🩺 **Top Diagnoses & Treatments** by frequency & outcome
* 👨‍⚕️ **Doctor & Department Performance** (cases, recovery, mortality)
* 💰 **Financial Analysis**: Billing vs insurance claim success
* 👵 **Demographics**: Age group distribution of patients


⚡ This project is highly relevant for **healthcare analytics roles** → used in hospitals, insurance companies, and health-tech startups.

---


# 👨‍💼 Project 6: **HR Employee Attrition Dashboard**


## 1. Dataset

* **Name**: HR Analytics / Employee Attrition Dataset

* **Sources (Kaggle)**:

  * [IBM HR Analytics Employee Attrition & Performance](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset?utm_source=chatgpt.com) *(most popular HR dataset)*
  * [HR Employee Attrition Dataset](https://www.kaggle.com/datasets/giripujar/hr-analytics?utm_source=chatgpt.com) *(smaller variant)*

* **Sample Columns**:

  * `EmployeeID`, `Age`, `Gender`, `MaritalStatus`, `Department`, `JobRole`
  * `MonthlyIncome`, `JobSatisfaction`, `OverTime`, `TotalWorkingYears`
  * `Attrition` (Yes/No)
  * `YearsAtCompany`, `YearsInCurrentRole`, `PerformanceRating`
  * `DistanceFromHome`, `EducationField`, `EnvironmentSatisfaction`


## 2. Project Objective

Analyze employee data to understand **attrition trends**, improve retention, and support HR decisions.

Goals:

* Track **attrition rate** by department, age group, salary band
* Identify factors that influence attrition (overtime, job satisfaction, distance from home, etc.)
* Help HR build **data-driven policies** for employee engagement and retention


## 3. Skills Used

* **Joins** (if multiple HR tables: Employees, Salaries, Departments)
* **Aggregations** → SUM, AVG, COUNT
* **Window Functions** → Ranking employees by salary, tenure
* **Case Statements** → Categorize attrition, age groups, salary bands
* **CTEs & Subqueries** → Attrition rate, churn segments
* **Date Functions** → Experience years, tenure
* **Conditional Filtering** → Employees at high attrition risk


## 4. Tasks / SQL Queries

| S.No.  | Task                               | Description & SQL Idea                                                                                                                                                                                                                                                                                                                                                            |
| -- | ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | **Overall Attrition Rate**         | % of employees who left. <br> `sql SELECT COUNT(CASE WHEN Attrition='Yes' THEN 1 END)*100.0/COUNT(*) AS AttritionRate FROM Employees; `                                                                                                                                                                                                                                           |
| 2  | **Attrition by Department**        | Find which departments lose the most employees. <br> `sql SELECT Department, COUNT(CASE WHEN Attrition='Yes' THEN 1 END)*100.0/COUNT(*) AS DeptAttritionRate FROM Employees GROUP BY Department ORDER BY DeptAttritionRate DESC; `                                                                                                                                                |
| 3  | **Attrition by Age Group**         | Younger or older employees leaving more? <br> `sql SELECT CASE WHEN Age < 25 THEN 'Under 25' WHEN Age BETWEEN 25 AND 35 THEN '25-35' WHEN Age BETWEEN 36 AND 45 THEN '36-45' ELSE '46+' END AS AgeGroup, COUNT(CASE WHEN Attrition='Yes' THEN 1 END)*100.0/COUNT(*) AS AttritionRate FROM Employees GROUP BY AgeGroup ORDER BY AttritionRate DESC; `                              |
| 4  | **Attrition by Salary Band**       | Check if low-income employees quit more often. <br> `sql SELECT CASE WHEN MonthlyIncome < 3000 THEN 'Low' WHEN MonthlyIncome BETWEEN 3000 AND 6000 THEN 'Medium' ELSE 'High' END AS SalaryBand, COUNT(CASE WHEN Attrition='Yes' THEN 1 END)*100.0/COUNT(*) AS AttritionRate FROM Employees GROUP BY SalaryBand ORDER BY AttritionRate DESC; `                                     |
| 5  | **Attrition vs. Overtime**         | Correlation between overtime and quitting. <br> `sql SELECT OverTime, COUNT(*) AS TotalEmployees, COUNT(CASE WHEN Attrition='Yes' THEN 1 END) AS AttritionCount, COUNT(CASE WHEN Attrition='Yes' THEN 1 END)*100.0/COUNT(*) AS AttritionRate FROM Employees GROUP BY OverTime; `                                                                                                  |
| 6  | **Attrition by Job Role**          | Identify most vulnerable job positions. <br> `sql SELECT JobRole, COUNT(CASE WHEN Attrition='Yes' THEN 1 END)*100.0/COUNT(*) AS RoleAttritionRate FROM Employees GROUP BY JobRole ORDER BY RoleAttritionRate DESC; `                                                                                                                                                              |
| 7  | **Distance from Home Impact**      | Does long commute increase attrition? <br> `sql SELECT CASE WHEN DistanceFromHome <= 5 THEN '0-5 km' WHEN DistanceFromHome BETWEEN 6 AND 15 THEN '6-15 km' ELSE '15+ km' END AS DistanceBand, COUNT(CASE WHEN Attrition='Yes' THEN 1 END)*100.0/COUNT(*) AS AttritionRate FROM Employees GROUP BY DistanceBand; `                                                                 |
| 8  | **Attrition by Education Field**   | Which education fields show higher attrition. <br> `sql SELECT EducationField, COUNT(CASE WHEN Attrition='Yes' THEN 1 END)*100.0/COUNT(*) AS EduAttritionRate FROM Employees GROUP BY EducationField ORDER BY EduAttritionRate DESC; `                                                                                                                                            |
| 9  | **Attrition vs. Job Satisfaction** | Are dissatisfied employees leaving more? <br> `sql SELECT JobSatisfaction, COUNT(CASE WHEN Attrition='Yes' THEN 1 END)*100.0/COUNT(*) AS AttritionRate FROM Employees GROUP BY JobSatisfaction ORDER BY JobSatisfaction; `                                                                                                                                                        |
| 10 | **Tenure vs Attrition**            | Does attrition occur more in first few years? <br> `sql SELECT CASE WHEN YearsAtCompany <= 2 THEN '0-2 years' WHEN YearsAtCompany BETWEEN 3 AND 5 THEN '3-5 years' WHEN YearsAtCompany BETWEEN 6 AND 10 THEN '6-10 years' ELSE '10+ years' END AS TenureGroup, COUNT(CASE WHEN Attrition='Yes' THEN 1 END)*100.0/COUNT(*) AS AttritionRate FROM Employees GROUP BY TenureGroup; ` |


## 5. Advanced Tasks

* **Survival Analysis**: Attrition probability over tenure years
* **Performance vs Attrition**: Do high performers leave or low performers?
* **Gender Gap**: Compare attrition by gender + job role
* **Work-Life Balance Impact**: Attrition by work-life balance rating
* **Predictive SQL Prep**: Flag high-risk employees (low salary + overtime + low satisfaction)


## 6. Possible Dashboards / Reports

* 📊 **Attrition KPIs**: overall attrition %, department-wise attrition
* 👥 **Employee Demographics**: attrition by age, gender, education
* 💰 **Salary Insights**: attrition by salary band, income distribution
* 🕒 **Work Insights**: overtime vs attrition, tenure analysis
* 🏢 **Department/Role Attrition**: top risky job roles & departments


⚡ This project is **highly practical** → companies like **IBM, Accenture, Infosys, and other IT/consulting firms** use such dashboards to reduce attrition.

---

# 🍿 Project 7: **Movie Streaming Platform Analytics**


## 1. Dataset

* **Name**: Movie / TV Shows Streaming Data

* **Sources (Kaggle & Open Data)**:

  * [Netflix Movies and TV Shows](https://www.kaggle.com/datasets/shivamb/netflix-shows?utm_source=chatgpt.com)
  * [Amazon Prime Movies & TV](https://www.kaggle.com/datasets/shivamb/amazon-prime-movies-and-tv-shows?utm_source=chatgpt.com)
  * [IMDb Dataset (via Kaggle/IMDb)](https://www.imdb.com/interfaces/?utm_source=chatgpt.com)

* **Sample Columns**:

  * `UserID`, `MovieID`, `Title`, `Genre`, `Director`, `Cast`
  * `WatchDate`, `WatchDurationMinutes`, `CompletionRate`
  * `SubscriptionType` (Basic, Standard, Premium)
  * `Region`, `DeviceType` (Mobile, Web, TV)
  * `Rating`, `ReleaseYear`


## 2. Project Objective

Analyze **user streaming behavior & content performance** to improve recommendations, reduce churn, and increase engagement.

Goals:

* Identify **top genres, movies, and shows**
* Track **subscription churn & upgrades**
* Analyze **watch-time trends by region, device, user type**
* Provide insights for **personalized recommendations**


## 3. Skills Used

* **Joins** → Users, Movies, Subscriptions, WatchHistory
* **Aggregations** → SUM, AVG, COUNT
* **Window Functions** → Ranking (top movies, top users)
* **Date Functions** → Monthly trends, daily peak times
* **Case Statements** → Categorize subscriptions, engagement levels
* **CTEs & Subqueries** → Churn analysis, completion rates
* **Pattern Matching (LIKE)** → Search movies by genre/title

---

## 4. Tasks / SQL Queries

| S.No.  | Task                          | Description & SQL Idea                                                                                                                                                                                            |
| -- | ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | **Most Watched Movies/Shows** | Find top 10 titles by total watch count. <br> `sql SELECT Title, COUNT(*) AS Views FROM WatchHistory GROUP BY Title ORDER BY Views DESC LIMIT 10; `                                                               |
| 2  | **Top Genres by Engagement**  | Which genres keep users watching longest? <br> `sql SELECT Genre, AVG(WatchDurationMinutes) AS AvgWatchTime FROM WatchHistory w JOIN Movies m ON w.MovieID=m.MovieID GROUP BY Genre ORDER BY AvgWatchTime DESC; ` |
| 3  | **User Engagement Score**     | Total watch time per user. <br> `sql SELECT UserID, SUM(WatchDurationMinutes) AS TotalWatchTime FROM WatchHistory GROUP BY UserID ORDER BY TotalWatchTime DESC; `                                                 |
| 4  | **Churned Users**             | Users inactive in last 30 days. <br> `sql SELECT u.UserID FROM Users u LEFT JOIN WatchHistory w ON u.UserID=w.UserID AND w.WatchDate > CURRENT_DATE - INTERVAL '30 days' WHERE w.UserID IS NULL; `                |
| 5  | **Subscription Breakdown**    | Active users by subscription type. <br> `sql SELECT SubscriptionType, COUNT(DISTINCT UserID) AS ActiveUsers FROM Users GROUP BY SubscriptionType; `                                                               |
| 6  | **Device Usage Analysis**     | Which devices are most used for streaming. <br> `sql SELECT DeviceType, COUNT(*) AS Streams FROM WatchHistory GROUP BY DeviceType ORDER BY Streams DESC; `                                                        |
| 7  | **Peak Viewing Hours**        | Most common hours users watch. <br> `sql SELECT EXTRACT(HOUR FROM WatchDate) AS Hour, COUNT(*) AS Views FROM WatchHistory GROUP BY Hour ORDER BY Views DESC; `                                                    |
| 8  | **Content Completion Rate**   | % of content fully watched. <br> `sql SELECT Title, AVG(CompletionRate)*100 AS AvgCompletion FROM WatchHistory GROUP BY Title ORDER BY AvgCompletion DESC; `                                                      |
| 9  | **Top Regions by Watch Time** | Regional content consumption. <br> `sql SELECT Region, SUM(WatchDurationMinutes) AS TotalWatchTime FROM WatchHistory w JOIN Users u ON w.UserID=u.UserID GROUP BY Region ORDER BY TotalWatchTime DESC; `          |
| 10 | **Yearly Content Trend**      | Content popularity by release year. <br> `sql SELECT ReleaseYear, COUNT(*) AS TitlesWatched FROM Movies m JOIN WatchHistory w ON m.MovieID=w.MovieID GROUP BY ReleaseYear ORDER BY ReleaseYear; `                 |


## 5. Advanced Tasks

* **Recommendation Engine Prep** → Find users with similar watch patterns
* **Binge-Watching Analysis** → Detect consecutive episodes watched in a session
* **Upgrade Analysis** → Users moving from Basic → Premium subscriptions
* **Content Gap** → Identify genres with high demand but low content supply
* **Regional Preferences** → Which genres dominate in which country


## 6. Possible Dashboards / Reports

* 🎬 **Content Leaderboard**: Top movies, genres, shows
* 👥 **User Engagement Dashboard**: watch time, completion rate, churn %
* 💳 **Subscription Analytics**: active users, upgrades, churned customers
* 📱 **Device & Region Usage**: where and how people watch
* 📊 **Trend Analysis**: monthly/yearly content popularity


⚡ This project is **perfect for OTT analytics roles** (Netflix, Prime Video, Hotstar, Sony LIV) and **portfolio projects** in SQL + Data Analytics.

---


# ✈️ Project 8: **Airline Flight Performance Analysis**


## 1. Dataset

* **Name**: Flight Delay / Airline Performance Data

* **Sources (Kaggle & Open Data)**:

  * [Flight Delay and Cancellations (US DOT)](https://www.kaggle.com/datasets/usdot/flight-delays?utm_source=chatgpt.com)
  * [Airline On-Time Performance Data (BTS)](https://www.transtats.bts.gov/OT_Delay/OT_DelayCause1.asp?utm_source=chatgpt.com)
  * [Flight Delay Prediction Dataset](https://www.kaggle.com/datasets/giovamata/airlinedelaycauses?utm_source=chatgpt.com)

* **Sample Columns**:

  * `FlightID`, `Airline`, `FlightNumber`, `OriginAirport`, `DestinationAirport`
  * `ScheduledDeparture`, `ActualDeparture`, `ScheduledArrival`, `ActualArrival`
  * `DelayMinutes`, `DelayCause` (Weather, Carrier, Security, Late Aircraft)
  * `CancelledFlag`, `CancellationReason`
  * `PassengerCount`


## 2. Project Objective

Analyze **flight performance** to improve operations, minimize delays, and optimize airline routes.

Goals:

* Track **delays & cancellations** by airline, airport, and route
* Find **causes of delays** and their financial impact
* Identify **busiest airports and routes**
* Measure **on-time performance KPI** for airlines


## 3. Skills Used

* **Joins** → Airlines, Airports, Flights, Delay Causes
* **Aggregations** → COUNT, SUM, AVG
* **Window Functions** → Ranking busiest routes & airlines
* **Date Functions** → Peak flight times, seasonal delays
* **Case Statements** → Categorize delays (On-Time, Minor, Major)
* **CTEs & Subqueries** → Delay % by airline/airport
* **Conditional Filtering** → Cancellations, delay thresholds


## 4. Tasks / SQL Queries

| S.No. | Task                             | Description & SQL Idea                                                                                                                                                                                                                                                                                   |
| -- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | **Overall On-Time Performance**  | % of flights on time. <br> `sql SELECT COUNT(CASE WHEN DelayMinutes <= 15 THEN 1 END)*100.0/COUNT(*) AS OnTimePercent FROM Flights; `                                                                                                                                                                    |
| 2  | **Average Delay by Airline**     | Which airline has worst delays? <br> `sql SELECT Airline, AVG(DelayMinutes) AS AvgDelay FROM Flights GROUP BY Airline ORDER BY AvgDelay DESC; `                                                                                                                                                          |
| 3  | **Top 10 Busiest Routes**        | Highest traffic origin-destination pairs. <br> `sql SELECT OriginAirport, DestinationAirport, COUNT(*) AS Flights FROM Flights GROUP BY OriginAirport, DestinationAirport ORDER BY Flights DESC LIMIT 10; `                                                                                              |
| 4  | **Delay Causes Breakdown**       | Major reasons for delays. <br> `sql SELECT DelayCause, COUNT(*) AS TotalDelays, SUM(DelayMinutes) AS TotalMinutes FROM Flights WHERE DelayMinutes > 0 GROUP BY DelayCause ORDER BY TotalMinutes DESC; `                                                                                                  |
| 5  | **Cancellation Rate by Airline** | Airlines with most cancellations. <br> `sql SELECT Airline, COUNT(CASE WHEN CancelledFlag=1 THEN 1 END)*100.0/COUNT(*) AS CancelRate FROM Flights GROUP BY Airline ORDER BY CancelRate DESC; `                                                                                                           |
| 6  | **Average Passenger Load**       | Average passengers per flight. <br> `sql SELECT Airline, AVG(PassengerCount) AS AvgPassengers FROM Flights GROUP BY Airline ORDER BY AvgPassengers DESC; `                                                                                                                                               |
| 7  | **Peak Delay Hours**             | Which hours experience highest delays? <br> `sql SELECT EXTRACT(HOUR FROM ScheduledDeparture) AS Hour, AVG(DelayMinutes) AS AvgDelay FROM Flights GROUP BY Hour ORDER BY AvgDelay DESC; `                                                                                                                |
| 8  | **Airport Performance**          | Which airports face the most delays? <br> `sql SELECT OriginAirport, COUNT(CASE WHEN DelayMinutes > 15 THEN 1 END) AS DelayedFlights, COUNT(*) AS TotalFlights, COUNT(CASE WHEN DelayMinutes > 15 THEN 1 END)*100.0/COUNT(*) AS DelayRate FROM Flights GROUP BY OriginAirport ORDER BY DelayRate DESC; ` |
| 9  | **Longest Delayed Flights**      | Top flights with max delay. <br> `sql SELECT FlightID, Airline, OriginAirport, DestinationAirport, DelayMinutes FROM Flights ORDER BY DelayMinutes DESC LIMIT 10; `                                                                                                                                      |
| 10 | **Yearly Trend of Delays**       | How delays vary over years. <br> `sql SELECT EXTRACT(YEAR FROM ScheduledDeparture) AS Year, AVG(DelayMinutes) AS AvgDelay FROM Flights GROUP BY Year ORDER BY Year; `                                                                                                                                    |


## 5. Advanced Tasks

* **Route Profitability Analysis** → Compare delays vs passenger count
* **Seasonal Delays** → Which months/seasons have highest delays
* **Weather vs Delays** → Identify if weather-related delays dominate
* **Missed Connections Impact** → Estimate cascading delays
* **Airline Comparison Report** → Combine cancellation + delay + load factors


## 6. Possible Dashboards / Reports

* 📊 **Airline KPIs**: Avg delay, cancellation %, on-time performance
* 🛫 **Airport Performance**: top delayed airports, busiest airports
* 🔁 **Route Insights**: busiest routes, worst routes for delays
* 🌦 **Delay Cause Analysis**: pie chart of weather vs carrier vs technical
* 📉 **Trend Analysis**: yearly/monthly delay & cancellation trends


⚡ This project is **directly used in aviation analytics** by airlines (Delta, American Airlines, IndiGo, Emirates) & airports to optimize operations.

---


# 📦 Project 9: **Supply Chain Inventory Management**

---

## 1. Dataset

* **Name**: Supply Chain / Inventory Dataset

* **Sources (Kaggle & Open Data)**:

  * [Supply Chain Dataset (Logistics, Demand, Inventory)](https://www.kaggle.com/datasets/prachi13/customer-analytics?utm_source=chatgpt.com)
  * [Inventory Demand Forecasting Dataset](https://www.kaggle.com/datasets/felixzhao/productdemandforecasting?utm_source=chatgpt.com)
  * [Global Superstore (Order + Shipping + Supplier Data)](https://www.kaggle.com/datasets/apoorvaappz/global-superstore-2016?utm_source=chatgpt.com)

* **Sample Columns**:

  * `ProductID`, `ProductName`, `Category`, `SupplierID`
  * `WarehouseID`, `StockLevel`, `ReorderLevel`, `SafetyStock`
  * `OrderID`, `OrderDate`, `QuantityOrdered`, `ShippedDate`
  * `SupplierName`, `LeadTimeDays`, `DeliveryStatus`
  * `Region`, `TransportMode`


## 2. Project Objective

Optimize **inventory levels & supplier performance** to reduce stockouts, improve delivery times, and control costs.

Goals:

* Track **low stock products** & reorder needs
* Analyze **supplier reliability** (on-time delivery %)
* Measure **inventory turnover**
* Identify **high-demand products** & seasonality trends

## 3. Skills Used

* **Joins** → Products, Suppliers, Orders, Warehouses
* **Aggregations** → SUM, AVG, COUNT
* **Window Functions** → Ranking products by demand, suppliers by delivery speed
* **Date Functions** → Lead time, order fulfillment cycle
* **Case Statements** → Stock status (In-stock, Low-stock, Out-of-stock)
* **CTEs & Subqueries** → Safety stock alerts, supplier performance
* **Conditional Filtering** → Late deliveries, critical reorder needs


## 4. Tasks / SQL Queries

| S.No.  | Task                            | Description & SQL Idea                                                                                                                                                                                                                                 |
| -- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1  | **Low Stock Alerts**            | Identify products below reorder level. <br> `sql SELECT ProductID, ProductName, StockLevel, ReorderLevel FROM Inventory WHERE StockLevel < ReorderLevel; `                                                                                             |
| 2  | **Top Products by Demand**      | Find highest ordered products. <br> `sql SELECT ProductID, SUM(QuantityOrdered) AS TotalDemand FROM Orders GROUP BY ProductID ORDER BY TotalDemand DESC LIMIT 10; `                                                                                    |
| 3  | **Supplier Performance**        | On-time delivery rate per supplier. <br> `sql SELECT SupplierID, SUM(CASE WHEN DeliveryStatus='On-Time' THEN 1 ELSE 0 END)*100.0/COUNT(*) AS OnTimeRate FROM Shipments GROUP BY SupplierID ORDER BY OnTimeRate DESC; `                                 |
| 4  | **Average Lead Time**           | Avg supplier lead time (order → delivery). <br> `sql SELECT SupplierID, AVG(DATEDIFF(day, OrderDate, ShippedDate)) AS AvgLeadTime FROM Orders GROUP BY SupplierID ORDER BY AvgLeadTime; `                                                              |
| 5  | **Warehouse Stock by Category** | Stock availability across categories. <br> `sql SELECT WarehouseID, Category, SUM(StockLevel) AS TotalStock FROM Inventory i JOIN Products p ON i.ProductID=p.ProductID GROUP BY WarehouseID, Category; `                                              |
| 6  | **Out-of-Stock Products**       | Products with zero stock. <br> `sql SELECT ProductID, ProductName FROM Inventory WHERE StockLevel=0; `                                                                                                                                                 |
| 7  | **Inventory Turnover Ratio**    | How quickly stock is sold & replaced. <br> `sql SELECT ProductID, SUM(QuantityOrdered)*1.0/AVG(StockLevel) AS InventoryTurnover FROM Orders o JOIN Inventory i ON o.ProductID=i.ProductID GROUP BY ProductID ORDER BY InventoryTurnover DESC; `        |
| 8  | **Late Deliveries Analysis**    | Suppliers causing frequent late deliveries. <br> `sql SELECT SupplierID, COUNT(*) AS LateDeliveries FROM Shipments WHERE DeliveryStatus='Late' GROUP BY SupplierID ORDER BY LateDeliveries DESC; `                                                     |
| 9  | **Seasonal Demand Trends**      | Monthly demand for top products. <br> `sql SELECT ProductID, DATEPART(MONTH, OrderDate) AS Month, SUM(QuantityOrdered) AS MonthlyDemand FROM Orders GROUP BY ProductID, DATEPART(MONTH, OrderDate) ORDER BY MonthlyDemand DESC; `                      |
| 10 | **Cost of Stockouts**           | Estimate lost sales due to zero inventory. <br> `sql SELECT ProductID, ProductName, (ReorderLevel - StockLevel)*AVG(UnitPrice) AS EstimatedStockoutCost FROM Inventory i JOIN Products p ON i.ProductID=p.ProductID WHERE StockLevel < ReorderLevel; ` |


## 5. Advanced Tasks

* **Safety Stock Simulation** → Estimate optimal safety stock per product
* **ABC Analysis** → Categorize products (A=high value, B=medium, C=low)
* **Supplier Risk Analysis** → Flag suppliers with >20% late delivery
* **Warehouse Optimization** → Identify least utilized warehouses
* **Transportation Mode Analysis** → Flight vs road vs ship efficiency

## 6. Possible Dashboards / Reports

* 📊 **Inventory Dashboard**: low stock alerts, out-of-stock products
* 📦 **Warehouse Insights**: stock by warehouse & category
* 🚛 **Supplier Performance**: on-time %, avg lead time
* 🔁 **Inventory Turnover**: fast vs slow moving products
* 📉 **Demand Trends**: monthly/seasonal product demand

---

⚡ This project is **super practical for manufacturing, retail, e-commerce, and logistics companies** (Amazon, Flipkart, Walmart, DHL).



# 🎶 Project 10: **Music Streaming App Insights**


## 1. Dataset

* **Possible Sources**:

  * [Spotify Dataset (Songs + Users + Streaming History)](https://www.kaggle.com/datasets/zaheenhamidani/ultimate-spotify-tracks-db?utm_source=chatgpt.com)
  * [Last.fm Music Dataset](https://www.kaggle.com/datasets/rymnikski/lastfm-dataset-1k-users?utm_source=chatgpt.com)
  * [Million Song Dataset (MSD)](http://millionsongdataset.com/)

* **Sample Columns**:

  * `UserID`, `UserAge`, `UserCountry`
  * `SongID`, `SongName`, `ArtistID`, `Genre`, `Duration`
  * `PlayID`, `PlayTimestamp`, `DeviceType`, `Location`
  * `SubscriptionType` (Free, Premium, Family)
  * `Skipped` (Yes/No)


## 2. Project Objective

Analyze **user listening behavior, popular tracks, and subscription impact** to drive **engagement & retention**.

Goals:

* Find **most streamed songs/artists/genres**
* Track **skip rates** to detect weak content
* Understand **subscription usage patterns**
* Measure **active user engagement (DAU, WAU, MAU)**
* Detect **regional trends** in music consumption


## 3. Skills Used

* **Joins** → Users, Songs, Plays
* **Aggregations** → Total streams, skips, active users
* **Window Functions** → Rank songs/artists by popularity
* **Date Functions** → Daily/weekly/monthly user activity
* **Case Statements** → Premium vs Free user behaviors
* **CTEs** → Active users, churn detection
* **Analytics Queries** → Retention & engagement


## 4. Tasks / SQL Queries

| S.No.  | Task                              | Description & SQL Idea                                                                                                                                                                                                                                       |
| -- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1  | **Most Streamed Songs**           | Top 10 songs by total plays. <br> `sql SELECT SongID, COUNT(*) AS PlayCount FROM Plays GROUP BY SongID ORDER BY PlayCount DESC LIMIT 10; `                                                                                                                   |
| 2  | **Top Artists by Streams**        | Find artists with highest total plays. <br> `sql SELECT ArtistID, COUNT(*) AS TotalStreams FROM Plays p JOIN Songs s ON p.SongID=s.SongID GROUP BY ArtistID ORDER BY TotalStreams DESC; `                                                                    |
| 3  | **Skip Rate Analysis**            | Find songs with highest skip rates. <br> `sql SELECT SongID, SUM(CASE WHEN Skipped='Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*) AS SkipRate FROM Plays GROUP BY SongID ORDER BY SkipRate DESC; `                                                                  |
| 4  | **User Engagement (DAU/WAU/MAU)** | Track daily active users. <br> `sql SELECT PlayDate, COUNT(DISTINCT UserID) AS DAU FROM Plays GROUP BY PlayDate ORDER BY PlayDate; `                                                                                                                         |
| 5  | **Premium vs Free Usage**         | Compare average daily plays. <br> `sql SELECT SubscriptionType, AVG(PlayCount) FROM (SELECT UserID, COUNT(*) AS PlayCount, SubscriptionType FROM Plays p JOIN Users u ON p.UserID=u.UserID GROUP BY UserID, SubscriptionType) t GROUP BY SubscriptionType; ` |
| 6  | **Genre Popularity**              | Which genres are most streamed. <br> `sql SELECT Genre, COUNT(*) AS TotalPlays FROM Plays p JOIN Songs s ON p.SongID=s.SongID GROUP BY Genre ORDER BY TotalPlays DESC; `                                                                                     |
| 7  | **Listening by Time of Day**      | Morning vs evening listening trends. <br> `sql SELECT DATEPART(HOUR, PlayTimestamp) AS Hour, COUNT(*) AS Plays FROM Plays GROUP BY DATEPART(HOUR, PlayTimestamp) ORDER BY Plays DESC; `                                                                      |
| 8  | **Regional Trends**               | Find most popular artist per country. <br> `sql SELECT UserCountry, ArtistID, COUNT(*) AS Streams FROM Plays p JOIN Users u ON p.UserID=u.UserID JOIN Songs s ON p.SongID=s.SongID GROUP BY UserCountry, ArtistID ORDER BY Streams DESC; `                   |
| 9  | **Churn Prediction Signals**      | Users inactive for last 30 days. <br> `sql SELECT UserID FROM Users WHERE UserID NOT IN (SELECT DISTINCT UserID FROM Plays WHERE PlayTimestamp >= DATEADD(day, -30, GETDATE())); `                                                                           |
| 10 | **Device Usage Analysis**         | Most common devices used. <br> `sql SELECT DeviceType, COUNT(*) AS Plays FROM Plays GROUP BY DeviceType ORDER BY Plays DESC; `                                                                                                                               |


## 5. Advanced Tasks

* **User Retention Analysis** → Track cohort of new users by month & measure retention.
* **Recommendation Engine** → Suggest songs based on listening history (via SQL filtering).
* **Playlist Popularity** → Which playlists retain users longer.
* **Skip Pattern Analysis** → Detect if skips cluster around specific artists/genres.
* **Revenue Impact** → Estimate revenue impact of premium vs free users.


## 6. Dashboards / Reports

* 🎵 **Top Songs & Artists Dashboard** (real-time charts)
* 🌍 **Regional Music Map** (country-wise trends)
* 📊 **Subscription Insights** (Premium vs Free usage)
* ⏰ **Listening Trends by Time** (hourly, daily, monthly patterns)
* 🚨 **Churn & Retention Report** (inactive users, skip analysis)


⚡ This project is **highly relevant for Spotify, YouTube Music, Apple Music, Gaana, JioSaavn** etc.

