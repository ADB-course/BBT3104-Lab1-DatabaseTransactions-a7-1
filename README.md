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
