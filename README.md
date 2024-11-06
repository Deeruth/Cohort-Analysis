# Cohort-Analysis

This analysis tracks customer retention patterns and purchase behavior by cohort using SQL and Power BI, with segmentation across order, order group, store, agent, and regions tables.

# Data Requirements:

Orders table: Holds purchase information, including order date, customer ID, and order amount.
Order Group table: Groups orders if needed for advanced segmentation.
Store table: Provides information about the store where the purchase was made.
Agent table: Information about the sales agent, useful for tracking engagement.
Regions table: Customer regions to analyze geographic behavior patterns.

Step 1: Data Preparation in MySQL
1. Define Cohorts Based on First Purchase Date
To track customer cohorts, each customer is grouped by the month they made their first purchase. This enables us to analyze subsequent purchases and retention over time.

``WITH first_purchase AS (
    SELECT customer_id,
           MIN(order_date) AS first_purchase_date
    FROM orders
    GROUP BY customer_id
)
SELECT customer_id,
       first_purchase_date,
       DATE_FORMAT(first_purchase_date, '%Y-%m') AS cohort_month
FROM first_purchase;``

2. Calculate Monthly Retention by Cohort
With cohorts defined, we track customer activity across each month to measure retention

``WITH customer_orders AS (
    SELECT o.customer_id,
           DATE_FORMAT(f.first_purchase_date, '%Y-%m') AS cohort_month,
           DATE_FORMAT(o.order_date, '%Y-%m') AS order_month,
           COUNT(o.order_id) AS monthly_orders,
           SUM(o.total_amount) AS monthly_revenue
    FROM orders o
    JOIN first_purchase f ON o.customer_id = f.customer_id
    GROUP BY o.customer_id, cohort_month, order_month
)
SELECT cohort_month,
       order_month,
       COUNT(DISTINCT customer_id) AS active_customers,
       SUM(monthly_revenue) AS total_revenue
FROM customer_orders
GROUP BY cohort_month, order_month
ORDER BY cohort_month, order_month;``

# Step 2: Calculate Retention Rates
We can add retention rate calculations in SQL for a clearer view of customer loyalty:
Explanation: By joining the initial cohort size with active customers over time, we calculate the retention rate as a percentage of the initial cohort that made repeat purchases in each subsequent month 
``WITH cohort_data AS (
    SELECT cohort_month,
           order_month,
           COUNT(DISTINCT customer_id) AS active_customers,
           SUM(monthly_revenue) AS total_revenue
    FROM customer_orders
    GROUP BY cohort_month, order_month
),
initial_cohort_size AS (
    SELECT cohort_month,
           COUNT(DISTINCT customer_id) AS cohort_size
    FROM customer_orders
    WHERE order_month = cohort_month
    GROUP BY cohort_month
)
SELECT c.cohort_month,
       c.order_month,
       c.active_customers,
       c.total_revenue,
       i.cohort_size,
       ROUND((c.active_customers / i.cohort_size) * 100, 2) AS retention_rate
FROM cohort_data c
JOIN initial_cohort_size i ON c.cohort_month = i.cohort_month
ORDER BY c.cohort_month, c.order_month;``

# Step 3: Power BI Visualization
Load the Cohort Data: Import the SQL output into Power BI to visualize retention rates, active customers, and revenue trends.

![image](https://github.com/user-attachments/assets/4eb952c4-c2b5-4553-8c32-b39eaddee83e)
![image](https://github.com/user-attachments/assets/70e3032d-63f2-4cc0-8c98-d4bec3ed08be)
![image](https://github.com/user-attachments/assets/def5436a-56d0-4187-a263-24b3359fa223)

### Churn Cohort
![image](https://github.com/user-attachments/assets/81cf8dad-bbfb-4903-93bc-a3e14363fe37)
![image](https://github.com/user-attachments/assets/bb743303-e7f8-4e89-b733-8da800ac2577)


Cohort Analysis Heatmap:

Matrix: Set up a matrix with Cohort Month as rows and Order Month as columns, displaying Retention Rate as values.
Conditional Formatting: Use a color gradient to highlight retention rates, where higher rates are darker or in contrasting colors.
Revenue Over Time by Cohort:

Line Chart: Plot Total Revenue on the y-axis and Order Month on the x-axis. Add a Legend for each Cohort Month to track revenue trends for each cohort over time.
Customer Count and Revenue by Region:

Stacked Column Chart: Display Region on the x-axis with Customer Count and Revenue as values to assess geographic cohort trends.

# Key Insights:

Retention Trends: Monthly retention rates help identify customer loyalty.

Revenue Trends: Reveals revenue contributions by cohort, indicating high-value segments.

Geographic Patterns: Shows retention and revenue distribution by region.
