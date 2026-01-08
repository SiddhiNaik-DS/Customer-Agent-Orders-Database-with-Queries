# Customer-Agent-Orders-Database-with-Queries
Sales Database Analysis Project  Goal: Efficiently manage and analyze a commercial sales system using relational data.  Key Entities: Agents (performance tracking), Customers (financial health), and Orders (transactional history).  Core Logic: Established primary and foreign key relationships to link sales performance directly to specific agents 
SQL Project - Customer, Agent, Orders Database with Queries
Step 1: Database & Table Creation

CREATE DATABASE customer_db;
USE customer_db;

CREATE TABLE Agent (
  AGENT_CODE VARCHAR(10) PRIMARY KEY,
  AGENT_NAME VARCHAR(50),
  WORKING_AREA VARCHAR(50),
  COMMISSION DECIMAL(4,2),
  PHONE_NO VARCHAR(30),
  COUNTRY VARCHAR(50)
);

 


CREATE TABLE Customer(
  CUST_CODE VARCHAR(50) PRIMARY KEY,
  CUST_NAME VARCHAR(100),
  CUST_CITY VARCHAR(50),
  WORKING_AREA VARCHAR(50),
  CUST_COUNTRY VARCHAR(50),
  GRADE INT,
  OPENING_AMT DECIMAL(12,2),
  RECEIVE_AMT DECIMAL(12,2),
  PAYMENT_AMT DECIMAL(12,2),
  OUTSTANDING_AMT DECIMAL(12,2),
  PHONE_NO VARCHAR(50),
  AGENT_CODE VARCHAR(30),
  FOREIGN KEY (AGENT_CODE) REFERENCES Agent(AGENT_CODE)
);

 


CREATE TABLE Orders (
  ORD_NUM INT PRIMARY KEY,
  ORD_AMOUNT DECIMAL(12,2),
  ADVANCE_AMOUNT DECIMAL(12,2),
  ORD_DATE DATE,
  CUST_CODE VARCHAR(50),
  AGENT_CODE VARCHAR(30),
  ORD_DESCRIPTION VARCHAR(100),
  FOREIGN KEY (CUST_CODE) REFERENCES Customer(CUST_CODE),
  FOREIGN KEY (AGENT_CODE) REFERENCES Agent(AGENT_CODE)
);

 

Step 2: Importing Data from Excel/CSV

1. Open MySQL Workbench → Connect to your Database.
2. Right click on your schema (customer_db) → 'Table Data Import Wizard'.
3. Select your Excel/CSV file (Customer.csv, Agent.csv, Orders.csv).
4. Choose the destination table (Customer, Agent, Orders).
5. Click 'Next' → Finish.
This way your Excel/CSV data will be imported into MySQL tables.


MAKE A DIAGRAM OR FLOWCHART FOR IMPORTING
Q1. Retrieve customer details along with the agent's name who helped them, showing the total outstanding amount
Query:
SELECT c.CUST_CODE, c.CUST_NAME, c.CUST_CITY, c.OUTSTANDING_AMT, a.AGENT_NAME
FROM Customer c
JOIN Agent a ON c.AGENT_CODE = a.AGENT_CODE;

 
 

Explanation: This query joins Customer and Agent tables to show customer details along with the agent’s name and outstanding amount.
Q2. Find the agents whose total order amount exceeds 10,000.
Query:
SELECT a.AGENT_CODE, a.AGENT_NAME, COALESCE(SUM(o.ORD_AMOUNT),0) AS total_order_amount
FROM Agent a
LEFT JOIN Orders o ON a.AGENT_CODE = o.AGENT_CODE
GROUP BY a.AGENT_CODE, a.AGENT_NAME
HAVING SUM(o.ORD_AMOUNT) > 10000;

 
 

Explanation: We join Agent and Orders, sum the order amount per agent, and filter those greater than 10,000.
Q3. Create a view to list all orders along with customer names and their respective agent's names. 
Query:
CREATE OR REPLACE VIEW OrderDetails AS
SELECT o.ORD_NUM, o.ORD_AMOUNT, c.CUST_NAME, a.AGENT_NAME
FROM Orders o
JOIN Customer c ON o.CUST_CODE = c.CUST_CODE
JOIN Agent a ON o.AGENT_CODE = a.AGENT_CODE;

SELECT * FROM OrderDetails;

 
 

Explanation: We create a view (virtual table) combining orders with customer and agent names.
Q4. Find all orders placed by customers who reside in New York. 
Query:
SELECT o.ORD_NUM, o.ORD_DATE, o.ORD_AMOUNT, c.CUST_NAME, c.CUST_CITY
FROM Orders o
JOIN Customer c ON o.CUST_CODE = c.CUST_CODE
WHERE UPPER(c.CUST_CITY) = 'NEW YORK';

 
 

Explanation: Filter customers whose city is New York and display their orders.

Q5. Find the total number of orders handled by each agent. 
Query:
SELECT a.AGENT_CODE, a.AGENT_NAME, COUNT(o.ORD_NUM) AS Total_Orders
FROM Agent a
LEFT JOIN Orders o ON a.AGENT_CODE = o.AGENT_CODE
GROUP BY a.AGENT_CODE, a.AGENT_NAME
ORDER BY Total_Orders DESC;
 
 

Explanation: Count how many orders each agent has handled using GROUP BY.
Q6. Get the list of customers who placed an order with an advance amount greater than or equal to 50% of the order amount. 
Query:
SELECT DISTINCT c.CUST_CODE, c.CUST_NAME, o.ORD_NUM, o.ORD_AMOUNT, o.ADVANCE_AMOUNT
FROM Customer c
JOIN Orders o ON c.CUST_CODE = o.CUST_CODE
WHERE o.ADVANCE_AMOUNT >= 0.5 * o.ORD_AMOUNT;

 
 

Explanation: Compare advance amount with 50% of order amount to filter customers.
Q7. Find the number of customers in each city and the total outstanding amount for each city.  
Query:
SELECT 
CUST_CITY,
 COUNT(*) AS NUM_CUSTOMERS,
  SUM(OUTSTANDING_AMT) AS TOTAL_OUTSTANDING
FROM customer
GROUP BY CUST_CITY;
 
 

Explanation: Group customers by city, count them, and sum their outstanding amounts.
Q8. List all customers along with their orders, including customers who have not placed any orders
Query:
SELECT 
  c.CUST_CODE, c.CUST_NAME, 
  o.ORD_NUM, o.ORD_DATE, o.ORD_AMOUNT
FROM customer c
LEFT JOIN orders o ON c.CUST_CODE = o.CUST_CODE;
 
 

Explanation: LEFT JOIN ensures customers without orders are still shown.
Q9. Find the customer who placed the highest order amount using a subquery. 
Query:
select
c.CUST_CODE,
c.CUST_NAME,
c.CUST_CITY,
c.WORKING_AREA,
c.CUST_COUNTRY,
o.ORD_AMOUNT as HIGHEST_ORDER_AMOUNT,
o.ORD_DATE
from Customer c
join Orders o on c.CUST_CODE=o.CUST_CODE
where o.ORD_AMOUNT=(select max(ORD_AMOUNT) from Orders);
 
 

Explanation: We use a subquery to find the maximum order amount and match it.
Q10. Find the total order amount for customers who have placed more than 2 orders using a subquery
Query:
SELECT 
    c.CUST_CODE,
    c.CUST_NAME,
    COUNT(o.ORD_NUM) AS order_count,
    SUM(o.ORD_AMOUNT) AS total_order_amount
FROM Customer c
JOIN orders o ON c.CUST_CODE = o.CUST_CODE
WHERE c.CUST_CODE IN (
    SELECT CUST_CODE
    FROM orders
    GROUP BY CUST_CODE
    HAVING COUNT(ORD_NUM) > 2
)
GROUP BY c.CUST_CODE, c.CUST_NAME
ORDER BY total_order_amount DESC;
 
 

Explanation: Group by customer and filter only those with more than 2 orders.
Q11. List the orders placed in the month of May 2008. 
Query:
SELECT * FROM Orders
WHERE MONTH(ORD_DATE) = 5 AND YEAR(ORD_DATE) = 2008;

 
 

Explanation: Extract month and year from ORD_DATE and filter May 2008.


Q12. List orders where the amount is greater than 1000 or the order date is before '2008-12-31'. 
Query:
SELECT * FROM Orders
WHERE ORD_AMOUNT > 1000 OR ORD_DATE < '2008-12-31';
 
 

Explanation: Apply OR condition to filter orders by amount and date.
Q13. Find all orders placed by customers in 'New York' or whose order amount is greater than 5000
Query:
SELECT o.ORD_NUM, o.ORD_AMOUNT, c.CUST_NAME, c.CUST_CITY
FROM Orders o
JOIN Customer c ON o.CUST_CODE = c.CUST_CODE
WHERE c.CUST_CITY = 'New York' OR o.ORD_AMOUNT > 5000;
 
 

Explanation: Filter orders where city = 'New York' OR order amount > 5000.
Q14. Find the agents who have handled orders for more than 3 distinct customers
Query:
SELECT a.AGENT_CODE, a.AGENT_NAME, COUNT(DISTINCT c.CUST_CODE) AS Customers_Handled
FROM Agent a
JOIN Orders o ON a.AGENT_CODE = o.AGENT_CODE
JOIN Customer c ON o.CUST_CODE = c.CUST_CODE
GROUP BY a.AGENT_CODE, a.AGENT_NAME
HAVING COUNT(DISTINCT c.CUST_CODE) > 3;

 
 

Explanation: We count distinct customers per agent and filter those handling more than 3.
Q15. Find customers who have made at least one payment but still have an outstanding amount greater than 7000
Query:
SELECT c.CUST_CODE, c.CUST_NAME, c.OUTSTANDING_AMT
FROM customer c
WHERE c.OUTSTANDING_AMT > 7000
  AND EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.CUST_CODE = c.CUST_CODE
      AND o.ADVANCE_AMOUNT > 0
  );
 
 

Explanation: Filter customers where payment is made but outstanding is still more than 7000.




 


 


 

 

