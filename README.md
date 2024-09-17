[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/r-tQZu0l)
# BBT3104-Lab1of6-DatabaseTransactions


| **Key**                                                               | Value                                                                                                                                                                              |
|---------------|---------------------------------------------------------|
| **Group Name**                                                               | A7 |
| **Semester Duration**                                                 | 19<sup>th</sup> August - 25<sup>th</sup> November 2024                                                                                                                       |

## Flowchart
![flowdchart](flowchart.png)

## Pseudocode
START  
USE database classicmodels  

START TRANSACTION  

// Set order number  
SET @orderNumber = (SELECT MAX(orderNumber) FROM orders) + 1  

// Insert into orders table  
INSERT INTO orders (orderNumber, orderDate, requiredDate, shippedDate, status, customerNumber)  
VALUES (@orderNumber, CURRENT_DATE, ADDDATE(CURRENT_DATE, INTERVAL 2 DAY), NULL, 'In Process', customerNumber)  

// Create a savepoint before the first product  
SAVEPOINT before_product_1  

// Insert first product into orderdetails  
INSERT INTO orderdetails (orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber)  
VALUES (@orderNumber, 'S18_1747', 30, 77.72, 1)  

// Get quantity in stock for the first product  
SET @quantityInStock = (SELECT quantityInStock FROM products WHERE productCode = 'S18_1747')  

// Update quantity in stock  
UPDATE products SET quantityInStock = @quantityInStock - 30 WHERE productCode = 'S18_1747'  

// Create a savepoint before the second product  
SAVEPOINT before_product_2  

// Insert second product into orderdetails  
INSERT INTO orderdetails (orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber)  
VALUES (@orderNumber, 'S18_2247', 54, 109.28, 2)  

// Get quantity in stock for the second product  
SET @quantityInStock = (SELECT quantityInStock FROM products WHERE productCode = 'S18_2247')  

// Update quantity in stock  
UPDATE products SET quantityInStock = @quantityInStock - 54 WHERE productCode = 'S18_2247'  

// Create a savepoint before the third product  
SAVEPOINT before_product_3  

// Insert third product into orderdetails  
INSERT INTO orderdetails (orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber)  
VALUES (@orderNumber, 'S18_2022', 12, 69.45, 3)  

// Get quantity in stock for the third product  
SET @quantityInStock = (SELECT quantityInStock FROM products WHERE productCode = 'S18_2022')  

// Update quantity in stock  
UPDATE products SET quantityInStock = @quantityInStock - 12 WHERE productCode = 'S18_2022'  

// Insert payment details  
INSERT INTO payments (customerNumber, checkNumber, paymentDate, amount)  
VALUES (customerNumber, '1024', CURRENT_DATE, 300000)  

// Commit the transaction  
COMMIT  

// Select order details  
SELECT * FROM orderdetails WHERE orderNumber = @orderNumber  

STOP  
## Support for the Sales Departments' Report
Suggested Improvements to Database Design
Payment Table Enhancement:

Add a Status Column: Include a status column in the payments table to represent whether a payment is complete, partial, or overdue. This will help differentiate between fully paid and partially paid orders.
Track Payment Amount: Ensure that the table records the amount paid in each transaction so that you can calculate balances easily.
CREATE TABLE payments (  
    payment_id INT PRIMARY KEY,  
    customer_number INT,  
    order_number INT,  
    payment_date DATE,  
    amount DECIMAL(10, 2),  
    status ENUM('full', 'partial', 'overdue'),  
    FOREIGN KEY (customer_number) REFERENCES customers(customer_number),  
    FOREIGN KEY (order_number) REFERENCES orders(order_number)  
);  
Balance Calculation Fields:

Order Table Modification: Add a total_amount column to the orders table to store the total amount due for each order. This will provide a reference point for comparing against payment amounts.
ALTER TABLE orders ADD total_amount DECIMAL(10, 2);  
Record Payment Installments:

Each installment payment could be recorded as a separate entry in the payments table, with a relation to the respective order. This would allow tracking of multiple payments per order easily.
Outstanding Balance View:

Create a view that calculates the remaining balance for each order by subtracting the total amount paid from the total cost of the order. This view can be used for reporting purposes.
CREATE VIEW outstanding_balances AS  
SELECT   
    o.order_number,  
    o.total_amount,  
    COALESCE(SUM(p.amount), 0) AS total_paid,  
    (o.total_amount - COALESCE(SUM(p.amount), 0)) AS remaining_balance,  
    CASE   
        WHEN (o.total_amount - COALESCE(SUM(p.amount), 0)) > 0 THEN 'Outstanding'   
        ELSE 'Paid'   
    END AS payment_status  
FROM orders o  
LEFT JOIN payments p ON o.order_number = p.order_number  
GROUP BY o.order_number, o.total_amount;  
Client Contact Information:

Ensure relevant contact information is present in the customer records, making it easier for the sales department to follow up with clients who owe money.