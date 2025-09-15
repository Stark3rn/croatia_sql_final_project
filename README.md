# croatia_sql_final_project
An essay about SQLite, and implementation of an PostgreSQL DB with libre office base, with its documentation.

## Entities and Attributes

### Customer
| Attribute  | Type           | Description                                |
|------------|---------------|--------------------------------------------|
| id         | BIGSERIAL PK   | Unique identifier for each customer         |
| firstName  | VARCHAR(25)    | Customer’s first name                      |
| lastName   | VARCHAR(25)    | Customer’s last name                       |
| email      | VARCHAR(50)    | Customer’s email                           |
| address    | VARCHAR(100)   | Customer’s address                         |

---

### Product
| Attribute     | Type           | Description                                |
|---------------|---------------|--------------------------------------------|
| id            | BIGSERIAL PK   | Unique identifier for each product         |
| name          | VARCHAR(50)    | Product name                               |
| description   | VARCHAR(200)   | Product description                        |
| price         | MONEY          | Price of the product                       |
| stockQuantity | BIGINT         | Quantity available in stock for this product |

---

### Order
| Attribute   | Type           | Description                                 |
|-------------|---------------|---------------------------------------------|
| id          | BIGSERIAL PK   | Unique identifier for each order            |
| customerId  | BIGINT FK      | Id of the customer that made the order      |
| date        | DATE           | Date of the order                           |
| status      | VARCHAR(15)    | Status of the order                         |
| amount      | MONEY          | Total amount of the order (sum of products) |

---

### OrderItem
| Attribute  | Type           | Description                                  |
|------------|---------------|----------------------------------------------|
| id         | BIGSERIAL PK   | Unique id for each item of an order          |
| orderId    | BIGINT FK      | Id of the order containing the product       |
| productId  | BIGINT FK      | Id of the product (e.g., corsair keyboard)   |
| quantity   | INT            | Quantity of the product ordered              |
| unitPrice  | MONEY          | Individual price of the product (per unit)   |

---

### Payment
| Attribute       | Type           | Description                                  |
|-----------------|---------------|----------------------------------------------|
| id              | BIGSERIAL PK   | Unique id of a payment                       |
| orderId         | BIGINT FK      | Id of the order associated with the payment  |
| amount          | MONEY          | Total amount of the payment                  |
| date            | DATE           | Date of the payment                          |
| paymentPlatform | VARCHAR(25)    | Platform used to pay (Card, PayPal, etc.)    |

---

## Relationships

- **Customer - Order**
  - A customer can make 0 or many orders.
  - An order is always made by one customer.

- **Order - OrderItem**
  - An order contains one or many items.
  - Each item belongs to one unique order.

- **OrderItem - Product**
  - An item refers to only one product.
  - A product can appear in 0 or many items.

- **Order - Payment**
  - An order is associated with one unique payment.
  - A payment is associated with one unique order.

---

## Queries Implemented

### 1. All orders from a specific customer
Retrieves every order that a customer made, using their email address.  

```sql
SELECT
    c.firstName,
    c.lastName,
    o.id AS orderID,
    o.date AS orderDate,
    o.amount AS totalAmount
FROM
    customer AS c
JOIN
    "order" AS o ON c.id = o.customerID
WHERE
    c.email = 'jean.dupont@email.com';
```

### 2 – All items of a specific order
Retrieves all items included in an order using its order id.
```sql
SELECT
    oi.orderID,
    p.name AS productName,
    oi.quantity,
    oi.unitPrice
FROM
    orderitem AS oi
JOIN
    product AS p ON oi.productID = p.id
WHERE
    oi.orderID = 1;
```

### 3 – All products with low stocks
Retrieves every products with a stock less than 15
```sql
SELECT
    p.name AS productName,
    p.stockQuantity
FROM
    product AS p
WHERE
    p.stockQuantity < 15
ORDER BY
    p.stockQuantity ASC;
```

### 4 – Most trending products
Retrieve the orders that are the most popular to the customers (best sellers)
```sql
SELECT
    p.id,
    p.name,
    SUM(oi.quantity) AS totalOrdered
FROM
    product AS p
JOIN
    orderitem AS oi ON p.id = oi.productID
GROUP BY
    p.id, p.name
ORDER BY
    totalOrdered DESC
```

### 5 – Average order value
Average price of an order (according to the order made by all the clients)
```sql
SELECT
    ROUND(AVG(o.amount), 2) AS averageOrderValue
FROM
    "order" AS o;
```
