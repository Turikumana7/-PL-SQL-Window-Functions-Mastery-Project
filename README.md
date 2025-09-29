Step 1: Problem Definition (2 pts)
Business Context

Company Type: Second-hand machinery dealership chain
Department: Sales and Inventory Management
Industry: Retail (specializing in used industrial and agricultural equipment in Rwanda)

Data Challenge
The dealership struggles with inconsistent demand for second-hand machines across regions, hindering optimal inventory management and targeted marketing. Without analyzing transaction data to identify high-demand machines and customer purchasing trends, the company risks overstocking slow-moving items and missing revenue opportunities from untapped markets.
Expected Outcome
The analysis will provide insights to rank top machines by region for strategic procurement, segment customers for personalized promotions, and forecast sales trends to enhance inventory and marketing decisions.

Step 2: Success Criteria (part of 2 pts)
Define exactly 5 measurable goals

Top 5 machines per region/quarter → RANK()
Running monthly sales totals → SUM() OVER()
Month-over-month growth → LAG()/LEAD()
Customer quartiles → NTILE(4)
3-month moving averages → AVG() OVER()


Step 3: Database Schema (part of 2 pts)
Design minimum 3 related tables with foreign keys
Table,Purpose,Key Columns,Example Row
customers,Customer info,"customer_id (PK), name, region","1001, John Doe, Kigali"
machines,Machine catalog,"machine_id (PK), name, category","2001, Used Tractor, Agriculture"
<img width="518" height="187" alt="TABLE" src="https://github.com/user-attachments/assets/eaa4ed96-e9bf-420a-8ad1-b3a681e09e22" />


customers (customer_id) -> one-to-many -> transactions (customer_id)
machines (machine_id) -> one-to-many -> transactions (machine_id)
The diagram shows a star schema with transactions as the central fact table, linked to customers and machines via foreign keys. (Create a visual ER diagram using draw.io and save as er_diagram.png for your GitHub repo.)

SQL to Create Tables
sqlCREATE TABLE customers (
    customer_id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    region VARCHAR2(50)
);

CREATE TABLE machines (
    machine_id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    category VARCHAR2(50)
);

CREATE TABLE transactions (
    transaction_id NUMBER PRIMARY KEY,
    customer_id NUMBER,
    machine_id NUMBER,
    sale_date DATE,
    amount NUMBER,
    CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    CONSTRAINT fk_machine FOREIGN KEY (machine_id) REFERENCES machines(machine_id)
);
