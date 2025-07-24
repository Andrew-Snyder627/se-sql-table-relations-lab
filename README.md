# Lab: SQL Table Relations

## Overview

This lab demonstrates advanced SQL querying techniques using a customer relationship management (CRM) database. It focuses on mastering JOIN operations, filtering, aggregation, and subqueries to retrieve insights across multiple related tables. The structure and logic were built using an ERD schema, SQLite, Pandas, and SQL within a Python environment.

The lab consists of 10 parts, each with a real-world business question, an SQL solution, and verified outputs using automated tests.

---

## Environment Setup

- **Database:** `data.sqlite`
- **Libraries:** `sqlite3`, `pandas`
- **Run Commands:**
  - `pipenv install`
  - `pipenv shell`
  - `python3 main.py`
  - `pytest`

---

## Step-by-Step Solutions

### **STEP 1**: Employees in Boston

**Goal:** List the first and last names of employees working in the Boston office.  
**Approach:** Use an `INNER JOIN` between `employees` and `offices` and filter `WHERE city = 'Boston'`.

```sql
SELECT e.firstName, e.lastName
FROM employees e
JOIN offices o ON e.officeCode = o.officeCode
WHERE o.city = 'Boston';
```

---

### **STEP 2**: Offices with No Employees

**Goal:** Identify any office locations that do not have assigned employees.  
**Approach:** Use a `LEFT JOIN` from `offices` to `employees`, filtering on `NULL` employeeNumber.

```sql
SELECT o.*
FROM offices o
LEFT JOIN employees e ON o.officeCode = e.officeCode
WHERE e.employeeNumber IS NULL;
```

---

### **STEP 3**: Employees and Their Office Info

**Goal:** Get a complete list of employees and the city/state of their office.  
**Approach:** Use a `LEFT JOIN` to include all employees, even those with missing office assignments.

```sql
SELECT e.firstName, e.lastName, o.city, o.state
FROM employees e
LEFT JOIN offices o ON e.officeCode = o.officeCode
ORDER BY e.firstName, e.lastName;
```

---

### **STEP 4**: Customers With No Orders

**Goal:** Find customers who have never placed an order.  
**Approach:** Use a `LEFT JOIN` from `customers` to `orders` and filter where `orderNumber IS NULL`.

```sql
SELECT c.contactFirstName, c.contactLastName, c.phone, c.salesRepEmployeeNumber
FROM customers c
LEFT JOIN orders o ON c.customerNumber = o.customerNumber
WHERE o.orderNumber IS NULL
ORDER BY c.contactLastName;
```

---

### **STEP 5**: Payment Audit by Amount

**Goal:** Return customer contact names, payment amount, and date, sorted by amount.  
**Approach:** Join `customers` and `payments` and cast `amount` to ensure numeric sorting.

```sql
SELECT c.contactFirstName, c.contactLastName, p.amount, p.paymentDate
FROM customers c
JOIN payments p ON c.customerNumber = p.customerNumber
ORDER BY CAST(p.amount AS REAL) DESC;
```

---

### **STEP 6**: Employees with High-Value Customers

**Goal:** Find employees whose customers have an average credit limit over $90,000.  
**Approach:** Join `employees` to `customers`, group by employee, and filter using `HAVING`.

```sql
SELECT e.employeeNumber, e.firstName, e.lastName, COUNT(c.customerNumber) AS num_customers
FROM employees e
JOIN customers c ON e.employeeNumber = c.salesRepEmployeeNumber
GROUP BY e.employeeNumber
HAVING AVG(c.creditLimit) > 90000
ORDER BY num_customers DESC;
```

---

### **STEP 7**: Top-Selling Products by Volume

**Goal:** For each product, return number of orders and total units sold.  
**Approach:** Join `products` to `orderdetails` and aggregate.

```sql
SELECT p.productName,
       COUNT(DISTINCT od.orderNumber) AS numorders,
       SUM(od.quantityOrdered) AS totalunits
FROM products p
JOIN orderdetails od ON p.productCode = od.productCode
GROUP BY p.productCode
ORDER BY totalunits DESC;
```

---

### **STEP 8**: Product Market Reach

**Goal:** Count how many distinct customers purchased each product.  
**Approach:** Join `products`, `orderdetails`, `orders`, and `customers`, then count distinct customer numbers.

```sql
SELECT p.productName, p.productCode,
       COUNT(DISTINCT o.customerNumber) AS numpurchasers
FROM products p
JOIN orderdetails od ON p.productCode = od.productCode
JOIN orders o ON od.orderNumber = o.orderNumber
GROUP BY p.productCode
ORDER BY numpurchasers DESC;
```

---

### **STEP 9**: Customer Count Per Office

**Goal:** Determine how many customers are associated with each office.  
**Approach:** Join `offices`, `employees`, and `customers` and group by office.

```sql
SELECT o.officeCode, o.city,
       COUNT(DISTINCT c.customerNumber) AS n_customers
FROM offices o
JOIN employees e ON o.officeCode = e.officeCode
JOIN customers c ON e.employeeNumber = c.salesRepEmployeeNumber
GROUP BY o.officeCode;
```

---

### **STEP 10**: Employees Who Sold Underperforming Products

**Goal:** Find employees who sold products ordered by fewer than 20 customers.  
**Approach:** Use a `WITH` subquery (CTE) to isolate those products, then join through the sales pipeline.

```sql
WITH under20_products AS (
    SELECT od.productCode
    FROM orderdetails od
    JOIN orders o ON od.orderNumber = o.orderNumber
    JOIN customers c ON o.customerNumber = c.customerNumber
    GROUP BY od.productCode
    HAVING COUNT(DISTINCT c.customerNumber) < 20
)
SELECT DISTINCT e.employeeNumber, e.firstName, e.lastName, o.city, o.officeCode
FROM employees e
JOIN offices o ON e.officeCode = o.officeCode
JOIN customers c ON e.employeeNumber = c.salesRepEmployeeNumber
JOIN orders odr ON c.customerNumber = odr.customerNumber
JOIN orderdetails od ON odr.orderNumber = od.orderNumber
WHERE od.productCode IN (SELECT productCode FROM under20_products)
ORDER BY e.firstName = 'Loui' DESC, e.firstName;
```

---

## Closing Notes

- This lab sharpened skills in real-world relational data modeling.
- Passing `pytest` ensured accuracy in results and test-driven confidence.
- Ordering, grouping, and filtering logic were tuned for exact test matches.

---

## Author

**Andrew Snyder**  
Flatiron Labs - SQL Relations  
July 2025
