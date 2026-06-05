# ☕ Monday Coffee Offline Expansion Analysis

## 📌 Project Overview

Monday Coffee is a successful online coffee brand planning to expand into the offline retail market by opening physical outlets in major Indian metropolitan cities. This project analyzes customer behavior, sales performance, population data, and rental costs to identify the most profitable cities for launching offline stores.

The analysis leverages historical sales data and city-level demographics to recommend the optimal locations for Monday Coffee's expansion strategy.

---

## 🎯 Business Objectives

* Identify the most suitable city for the first offline outlet.
* Maximize revenue and profitability.
* Utilize existing online customer data for strategic decision-making.
* Develop a scalable framework for future offline expansion.

---

## 📊 Business Questions Addressed

# ☕ Coffee Sales Analysis Using SQL

## Project Overview

This project analyzes coffee sales data across multiple cities to uncover insights related to revenue, customer behavior, product performance, and market opportunities. SQL was used to perform Exploratory Data Analysis (EDA) and answer key business questions.

---

# Database Tables

* **Sales** – Transaction details
* **Customer** – Customer information
* **Product** – Product catalog
* **City** – City demographics and market information

## 📂 Dataset Overview

The analysis is based on the following datasets:

### City Table

Contains city-level information such as:

* City ID
* Population
* Estimated Rent
* City Name
* Rank

### Customer Table

Contains:

* Customer ID
* Customer Name
* City ID

### Product Table

Contains:

* Product ID
* Product Name
* Price

### Sales Table

Contains:

* Sale ID
* Customer ID
* Product ID
* Rating
* Sale Amount
* Sale Date

  ### Date Table Created With the help of DAX

  * Date
  * Month
  * Year
  * Month Name

# Exploratory Data Analysis (EDA)

## 1. City-wise Total Sales

### Query

```sql
SELECT
    ci.city_name,
    SUM(s.total) AS Total_sale
FROM Sales s
INNER JOIN Customer c
    ON s.customer_id = c.customer_id
INNER JOIN City ci
    ON ci.city_id = c.city_id
GROUP BY ci.city_name
ORDER BY Total_sale DESC;
```

---

## 2. Total Customers

### Query

```sql
SELECT COUNT(customer_id) AS Total_Customer
FROM Customer;
```

### Result

* Total Customers = 497

---

## 3. Total Revenue

### Query

```sql
SELECT SUM(total) AS Total_Sale
FROM Sales;
```

### Result

* Total Revenue = ₹6,070,190

---

## 4. Top 10 Highest Priced Products

### Query

```sql
SELECT
    product_name,
    SUM(price) AS price
FROM Product
GROUP BY product_name
ORDER BY price DESC
LIMIT 10;
```

---

## 5. Top Ranked Cities

### Query

```sql
SELECT *
FROM City
ORDER BY city_rank ASC
LIMIT 5;
```

---

## 6. Population Analysis

### Query

```sql
SELECT
    city_name,
    population
FROM City
ORDER BY population DESC;
```

### Insight

* Delhi is the most populated city.

---

# Business Questions

## Q1. Estimated Coffee Consumers by City

Assuming 25% of the population consumes coffee.

### Query

```sql
SELECT
    city_name,
    ROUND(population * 0.25 / 1000000, 2)
        AS coffee_consumers_in_million
FROM City
ORDER BY coffee_consumers_in_million DESC;
```

---

## Q2. Revenue Generated in Q4 2023

### Query

```sql
SELECT
    ci.city_name,
    YEAR(s.sale_date) AS years,
    QUARTER(s.sale_date) AS quarters,
    SUM(s.total) AS total_sale
FROM Sales s
INNER JOIN Customer c
    ON s.customer_id = c.customer_id
INNER JOIN City ci
    ON ci.city_id = c.city_id
WHERE YEAR(s.sale_date) = 2023
  AND QUARTER(s.sale_date) = 4
GROUP BY ci.city_name, years, quarters
ORDER BY total_sale DESC;
```

---

## Q3. Sales Count for Each Product

### Query

```sql
SELECT
    p.product_name,
    COUNT(s.product_id) AS qty_ordered
FROM Sales s
INNER JOIN Product p
    ON s.product_id = p.product_id
GROUP BY p.product_name
ORDER BY qty_ordered DESC;
```

---

## Q4. Average Sales Amount per Customer by City

### Query

```sql
SELECT
    ci.city_name,
    SUM(s.total) /
    COUNT(DISTINCT c.customer_id)
    AS average_sale_per_customer
FROM Customer c
INNER JOIN Sales s
    ON c.customer_id = s.customer_id
INNER JOIN City ci
    ON ci.city_id = c.city_id
GROUP BY ci.city_name
ORDER BY average_sale_per_customer DESC;
```

---

## Q5. Top 3 Selling Products by City

### Query

```sql
WITH CTE AS
(
    SELECT
        ci.city_name,
        p.product_name,
        SUM(s.total) AS total_sale,
        DENSE_RANK() OVER
        (
            PARTITION BY ci.city_name
            ORDER BY SUM(s.total) DESC
        ) AS ranks
    FROM Sales s
    INNER JOIN Customer c
        ON s.customer_id = c.customer_id
    INNER JOIN Product p
        ON p.product_id = s.product_id
    INNER JOIN City ci
        ON ci.city_id = c.city_id
    GROUP BY ci.city_name,
             p.product_name
)

SELECT *
FROM CTE
WHERE ranks <= 3;
```

---

## Q6. Monthly Sales Trend for Ground Espresso Coffee (250g)

### Query

```sql
SELECT
    MONTH(sale_date) AS months,
    SUM(s.total) AS total_sales,
    p.product_name
FROM Sales s
INNER JOIN Product p
    ON s.product_id = p.product_id
WHERE p.product_name =
'Ground Espresso Coffee (250g)'
GROUP BY MONTH(sale_date),
         p.product_name
ORDER BY months;
```

---

## Q7. Customer Segmentation by City

Unique customers who purchased coffee products.

### Query

```sql
SELECT
    ci.city_name,
    p.product_name,
    COUNT(DISTINCT c.customer_name)
        AS distinct_customer
FROM City ci
LEFT JOIN Customer c
    ON ci.city_id = c.city_id
INNER JOIN Sales s
    ON s.customer_id = c.customer_id
INNER JOIN Product p
    ON p.product_id = s.product_id
WHERE p.product_name LIKE '%Coffee%'
GROUP BY ci.city_name,
         p.product_name;
```

---

## Q8. Monthly Sales Growth Rate by City

### Query

```sql
WITH CTE AS
(
    SELECT
        ci.city_name,
        YEAR(s.sale_date) AS years,
        MONTH(s.sale_date) AS months,
        SUM(s.total) AS total_sales
    FROM City ci
    INNER JOIN Customer c
        ON ci.city_id = c.city_id
    INNER JOIN Sales s
        ON s.customer_id = c.customer_id
    GROUP BY ci.city_name,
             years,
             months
),

Growth AS
(
    SELECT
        city_name,
        years,
        months,
        total_sales AS current_sales,
        LAG(total_sales,1)
        OVER
        (
            PARTITION BY city_name
            ORDER BY years, months
        ) AS previous_sales
    FROM CTE
)

SELECT *,
       ROUND(
           ((current_sales - previous_sales)
            / previous_sales) * 100,
           2
       ) AS growth_percentage
FROM Growth
WHERE previous_sales IS NOT NULL;
```

---

## Q9. City Population vs Coffee Consumers

### Query

```sql
SELECT
    city_name,
    population,
    ROUND(population * 0.25)
    AS estimated_coffee_consumers
FROM City
ORDER BY estimated_coffee_consumers DESC;
```

---

## Q10. Average Sale vs Average Rent per Customer

### Query

```sql
WITH Avg_sale_customer AS
(
    SELECT
        ci.city_name,
        SUM(s.total) /
        COUNT(DISTINCT c.customer_name)
        AS avg_sale_per_customer,
        COUNT(DISTINCT c.customer_name)
        AS total_customer
    FROM Sales s
    INNER JOIN Customer c
        ON s.customer_id = c.customer_id
    INNER JOIN City ci
        ON ci.city_id = c.city_id
    GROUP BY ci.city_name
),

Rent AS
(
    SELECT
        city_name,
        estimated_rent
    FROM City
)

SELECT
    avs.city_name,
    avs.avg_sale_per_customer,
    r.estimated_rent /
    avs.total_customer
    AS avg_rent_per_customer
FROM Rent r
JOIN Avg_sale_customer avs
    ON r.city_name = avs.city_name
ORDER BY avg_sale_per_customer DESC,
         avg_rent_per_customer ASC;
```

---

# SQL Concepts Used

* Joins (INNER JOIN, LEFT JOIN)
* Aggregate Functions (SUM, COUNT, AVG)
* Window Functions (DENSE_RANK, LAG)
* Common Table Expressions (CTEs)
* Date Functions (YEAR, MONTH, QUARTER)
* GROUP BY
* ORDER BY
* Filtering with WHERE

---

# Key Insights

* Total Revenue: ₹6.07 Million+
* Total Customers: 497
* Delhi has the highest population.
* Coffee consumer estimation highlights cities with strong market potential.
* Top-selling products vary across cities.
* Monthly growth analysis helps identify expanding and declining markets.
* Revenue-per-customer and rent analysis assist in evaluating profitability and expansion opportunities.

# Tools Used

* MySQL
* SQL
* GitHub

# Author

Your Name




---

## 📈 Key Insights

### Delhi

* Highest estimated coffee consumers per million population.
* Strong customer concentration and market potential.
* Suitable for brand visibility and rapid customer acquisition.

### Pune

* Highest overall revenue generation.
* Lowest average rent per customer among major cities.
* Excellent balance between profitability and operating costs.

### Chennai

* Growing coffee consumer market.
* Strong revenue performance.
* Attractive option for future expansion.

---

## 🏆 Expansion Recommendations

### 🥇 First Outlet – Delhi

Delhi is recommended as the first offline outlet due to:

* Largest coffee-consuming customer base.
* High market penetration potential.
* Strong long-term growth opportunities.

### 🥈 Second Outlet – Pune

Pune is recommended as the second outlet because:

* Highest revenue contribution.
* Cost-efficient operations.
* Strong café culture and customer demand.

### 🥉 Third Outlet – Chennai

Chennai is recommended as the third outlet due to:

* Growing coffee market.
* Strong sales trends.
* Future scalability potential.

---

## 📊 Expected Business Impact

By implementing the recommended expansion strategy, Monday Coffee can:

* Increase offline revenue streams.
* Strengthen brand presence in metropolitan markets.
* Optimize operational costs.
* Build a scalable offline retail model for future growth.

---

## 👨‍💻 Author

**Amit Bhowate**

Data Analyst | Business Intelligence Enthusiast

LinkedIn: www.linkedin.com/in/amitbhowate98





