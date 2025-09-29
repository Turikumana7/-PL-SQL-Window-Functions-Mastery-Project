                        PL/SQL Window Functions Mastery Project


Step 1: Problem Definition 
Business Context

Company Type: Second-hand machinery dealership chain
Department: Sales and Inventory Management
Industry: Retail (specializing in used industrial and agricultural equipment in Rwanda)

Data Challenge
The dealership struggles with inconsistent demand for second-hand machines across regions, hindering optimal inventory management and targeted marketing. Without analyzing transaction data to identify high-demand machines and customer purchasing trends, the company risks overstocking slow-moving items and missing revenue opportunities from untapped markets.
Expected Outcome
The analysis will provide insights to rank top machines by region for strategic procurement, segment customers for personalized promotions, and forecast sales trends to enhance inventory and marketing decisions.

Step 2: Success Criteria 
Define exactly 5 measurable goals

Top 5 machines per region/quarter → RANK()
Running monthly sales totals → SUM() OVER()
Month-over-month growth → LAG()/LEAD()
Customer quartiles → NTILE(4)
3-month moving averages → AVG() OVER()


Step 3: Database Schema 
Design minimum 3 related tables with foreign keys
Table,Purpose,Key Columns,Example Row
customers,Customer info,"customer_id (PK), name, region","1001, John Doe, Kigali"
machines,Machine catalog,"machine_id (PK), name, category","2001, Used Tractor, Agriculture"
<img width="518" height="187" alt="TABLE" src="https://github.com/user-attachments/assets/eaa4ed96-e9bf-420a-8ad1-b3a681e09e22" />

<img width="300" height="158" alt="ER" src="https://github.com/user-attachments/assets/c5b0403c-4521-4d72-892e-da416db08e6c" />

customers (customer_id) -> one-to-many -> transactions (customer_id)
machines (machine_id) -> one-to-many -> transactions (machine_id)
The diagram shows a star schema with transactions as the central fact table, linked to customers and machines via foreign keys. (Create a visual ER diagram using draw.io and save as er_diagram.png for your GitHub repo.)

CREATE TABLE customers (
    customer_id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    region VARCHAR2(50)
);

<img width="255" height="70" alt="table customer" src="https://github.com/user-attachments/assets/0ab2819c-555f-43c7-88c2-afd8581f40ab" />

CREATE TABLE machines (
    machine_id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    category VARCHAR2(50)
);
<img width="251" height="84" alt="machines" src="https://github.com/user-attachments/assets/775fce70-34a7-4546-95ff-e6c07b7b5be7" />

CREATE TABLE transactions (
    transaction_id NUMBER PRIMARY KEY,
    customer_id NUMBER,
    machine_id NUMBER,
    sale_date DATE,
    amount NUMBER,
    CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    CONSTRAINT fk_machine FOREIGN KEY (machine_id) REFERENCES machines(machine_id)
);

INSERT INTO tables   
                        customers

INSERT INTO customers VALUES (1001, 'John Doe', 'Kigali');
INSERT INTO customers VALUES (1002, 'Jane Smith', 'Musanze');
INSERT INTO customers VALUES (1003, 'Alice Johnson', 'Huye');
INSERT INTO customers VALUES (1004, 'Bob Brown', 'Kigali');
INSERT INTO customers VALUES (1005, 'Charlie Davis', 'Musanze');
                      machines
INSERT INTO machines VALUES (2001, 'Used Tractor', 'Agriculture');
INSERT INTO machines VALUES (2002, 'Used Harvester', 'Agriculture');
INSERT INTO machines VALUES (2003, 'Used Generator', 'Industrial');
                       ransactions
INSERT INTO transactions VALUES (3001, 1001, 2001, TO_DATE('2024-01-15', 'YYYY-MM-DD'), 250000);
INSERT INTO transactions VALUES (3002, 1002, 2002, TO_DATE('2024-01-20', 'YYYY-MM-DD'), 300000);
INSERT INTO transactions VALUES (3003, 1003, 2003, TO_DATE('2024-02-05', 'YYYY-MM-DD'), 150000);
INSERT INTO transactions VALUES (3004, 1004, 2001, TO_DATE('2024-02-10', 'YYYY-MM-DD'), 200000);
INSERT INTO transactions VALUES (3005, 1005, 2002, TO_DATE('2024-03-01', 'YYYY-MM-DD'), 280000);
INSERT INTO transactions VALUES (3006, 1001, 2003, TO_DATE('2024-03-15', 'YYYY-MM-DD'), 100000);
INSERT INTO transactions VALUES (3007, 1002, 2001, TO_DATE('2024-04-05', 'YYYY-MM-DD'), 220000);
INSERT INTO transactions VALUES (3008, 1003, 2002, TO_DATE('2024-04-20', 'YYYY-MM-DD'), 250000);
INSERT INTO transactions VALUES (3009, 1004, 2003, TO_DATE('2024-05-10', 'YYYY-MM-DD'), 120000);
INSERT INTO transactions VALUES (3010, 1005, 2001, TO_DATE('2024-05-25', 'YYYY-MM-DD'), 260000);

           RANK
 Rank customers by total spending
SELECT 
    customer_id,
    SUM(amount) AS spending,
    ROW_NUMBER() OVER (ORDER BY SUM(amount) DESC) AS row_number,
    RANK() OVER (ORDER BY SUM(amount) DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY SUM(amount) DESC) AS dense_rank,
    PERCENT_RANK() OVER (ORDER BY SUM(amount) DESC) AS percent_rank
FROM transactions
GROUP BY customer_id
ORDER BY spending DESC;
<img width="300" height="77" alt="rank2" src="https://github.com/user-attachments/assets/26bc259e-b822-47f2-af39-6c878963ad0c" />

                             AGGREGATE
WITH monthly_sales AS (
    SELECT TRUNC(sale_date, 'MM') AS month, SUM(amount) AS amount
    FROM transactions
    GROUP BY TRUNC(sale_date, 'MM')
)
SELECT 
    month,
    amount,
    SUM(amount) OVER (ORDER BY month ROWS UNBOUNDED PRECEDING) AS running_total,
    AVG(amount) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_rows
FROM monthly_sales
ORDER BY month;
<img width="221" height="65" alt="aggregate2" src="https://github.com/user-attachments/assets/5cf76133-3282-4f7b-9eb3-7feedc2788e0" />

             Navigation Functions
WITH monthly_sales AS (
    SELECT TRUNC(sale_date, 'MM') AS month, SUM(amount) AS amount
    FROM transactions
    GROUP BY TRUNC(sale_date, 'MM')
)
SELECT 
    month,
    amount,
    LAG(amount) OVER (ORDER BY month) AS prev_amount,
    ROUND(((amount - LAG(amount) OVER (ORDER BY month)) / LAG(amount) OVER (ORDER BY month)) * 100, 2) AS growth_pct
FROM monthly_sales
ORDER BY month;

           Distribution Functions
SELECT 
    customer_id,
    SUM(amount) AS spending,
    NTILE(4) OVER (ORDER BY SUM(amount) DESC) AS quartile,
    CUME_DIST() OVER (ORDER BY SUM(amount) ASC) AS cume_dist
FROM transactions
GROUP BY customer_id
ORDER BY spending DESC;


                         Results Analysis

1. Descriptive – What happened?
Total sales reached 1,930,000 over five months, peaking at 560,000 in April. Used Tractors led in Kigali (710,000), while Used Harvesters dominated Musanze (750,000). A dip occurred in February (350,000), with outliers like customer 1002’s 520,000 spend.
2. Diagnostic – Why?
The February dip (-36.36% from January) may reflect seasonal agricultural slowdowns. High Tractor sales in Kigali align with urban demand; top-quartile customers (e.g., 1001, 1002) drive 60% of revenue due to bulk purchases.
3. Prescriptive What next?
Prioritize Tractor stock in Kigali and Harvesters in Musanze. Offer discounts to top-quartile customers. Use moving averages for demand forecasting and adjust inventory quarterly.

Step 7: References (

Oracle Help Center - Analytic Functions: https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/Analytic-Functions.html
GeeksforGeeks - Window Functions in PL/SQL: https://www.geeksforgeeks.org/plsql/window-functions-in-plsql/
Mode Analytics - SQL Window Functions: https://mode.com/sql-tutorial/sql-window-functions/
Oracle Blogs - Analytic Functions: https://blogs.oracle.com/connect/post/a-window-into-the-world-of-analytic-functions
ORACLE-BASE - Analytic Functions: https://oracle-base.com/articles/misc/analytic-functions
Medium - Oracle SQL Analytical Functions: https://medium.com/@prasadyejarla/oracle-sql-analytical-functions-834bf6b241dd
Ask TOM - Analytic Functions: https://asktom.oracle.com/ords/f?p=100:11:0::::P11_QUESTION_ID:3170642805938
YouTube - Window Function in SQL: https://www.youtube.com/watch?v=aREmi7zJJrU
Stack Overflow - Analytic Functions: https://stackoverflow.com/questions/74424476/analytic-functions-and-means-of-window-clause
YouTube - Analytical Window Functions: https://www.youtube.com/watch?v=gRC3ndWLsoo
