---
title: SQL Basics
created: '2021-01-12T17:20:13.491Z'
modified: '2021-01-13T20:47:30.170Z'
---

# SQL Basics

| Logical data concept | SQL Equivalent     |
| -------------------- |------------------: |
| Entity               | Table              |
| Attribute            | Column Heading     |
| Value                | Row Contents       |
Apart from entities, the relationship between entities are also captured by tables. 

## Primary Key
(PK). A unique identifier for each entry of a table. Usually implemented as a sequence with initial value of 1 and interval of 1.
## Foreign Key

An alternative to having a separate table for represting relationships, we can add a column to one of our entity tables that represent the keys from the other entity. This new column is called the Foreign Key (FK) of the table.

Usually the FK assigned to the *child* table or entity.
>You can identify the parent table by asking which table can exists on its own without the presence of the other table.

Example: If we had two tables called `Customers` and `Orders`, it is easy to observe that customers can exists independently without orders. However, even though orders can exists without referencing customers, it doesn't make sense to talk about orders without them depending on customers. This `Orders` will be the child table and get assigned the FK (customer id).

## Aggregations

Aggregation functions are commonly known as `COUNT`, `AVG` and `SUM`. 
These agg functions take a column name and essentially performs a *reduce* operation on the entire row. For example, the following SQL statement returns the total number of orders in the orders table. 
```sql
SELECT COUNT(ORDER_ID)
FROM ORDERS

-- Returns: N (where is N is a number) 

```


### Group By
In most cases, we want to see aggregation results per entry of a table i.e. number of orders for each customer.
Therefore, the agg functions as seen above, are not very useful without an accompanying `GROUP BY` clause which performs aggregations per entry of the group by column. For example, the following SQL statement gives the number of orders per customer (assuming distinct, non-null data)


```sql
SELECT COUNT(ORDER_ID) -- PK
FROM ORDERS
GROUP BY CUSTOMER_ID -- FK

-- Returns: Table with columns: CUSTOMER_ID and Number of Orders.
```
Note: `GROUP BY N` means to group by the Nth column in the select statement regardless of the the column's name.
### Conditional aggregation
More often than not, we want to aggregate over entries that meet a certain condition. We can achieve that by using the `CASE WHEN [condition] THEN [on true] ELSE [on false] END` statement inside an aggregation function. For example if we wanted to find the amount of money each customer spent on books, we  can write:

```sql
SELECT SUM(CASE WHEN item_type = 'book' THEN AMOUNT END) -- implicit ELSE
FROM ORDERS
GROUP BY CUSTOMER_ID

-- Returns: Table with cols: CUSTOMER_ID, Amount spent on books.
```

### Having

If we want to sort our results post-aggregation, we cannot use the `WHERE` clause since it does not support aggregation. Instead, we have to use the `HAVING` clause. This clause must be a follow up to a `GROUP BY` clause. For example, the following SQL statement lists the number of customers in each country (only include countries with more than 5 customers):

```sql
SELECT COUNT(CustomerID) as num_customers, Country
FROM Customers
GROUP BY Country
HAVING num_customers > 5 -- WRONG!! This can be written using WHERE (due to no agg).
-- CORRECT STATEMENT WILL BE
HAVING COUNT(CustomerID) > 5
```

## SQL Joins

Logically, we want to perform a join when we want merge information from multiple tables and view them ordered by the primary key of a table.

An easy way to address the need for a JOIN operation is when columns from multiple tables are referenced in the select statement.

### Types of Joins
There are four types of JOINs in SQL: 
- Inner join
- Left join
- Right join
- Full  join

Based on the different joins types, the number of rows in the resulting table chanes. The number of columns returned on each type of join is the same (i.e. all columns from both tables).

This topic is well explained [here](http://www.sql-join.com/sql-join-types). The following examples are taken from the given link. Assume we have a table of customers and a table of orders (with an FK of customer_id). Not all customers have a corresponding order and not all orders have a customer_id (due to external orders).

*Fig: Customers Table*

| customer_id |	first_name |	last_name |	city |	state |	zipcode |
|:------------|:---------- |:-----------|:---- |:------ |:-------- |
| 1 |	George |	Washington |		Mount Vernon	| VA	| 22121 |
| 2 |	John |	Adams |		 Quincy	| MA	| 02169 |
| 3 |	Thomas |	Jefferson |	Charlottesville	| VA	| 22902 |
| 4 |	James |	Madison |		Orange	| VA	| 22960 |
| 5 |	James |	Monroe |	Charlottesville	| VA	| 22902 |


*Fig: Orders Table*

| order_id	| order_date |	amount |	customer_id |
|-|-|-|-|
| 1 |	07/04/1776 |	$234.56 |	1|
| 2 |	03/14/1760 |	$78.50 |	3|
| 3 |	05/23/1784 |	$124.00 |	2|
| 4 |	09/03/1790 |	$65.50 |	3|
| 5 |	07/21/1795 |	$25.50 |	10|
| 6 |	11/27/1787 |	$14.40 |	9|


#### Inner Join
This type of join returns all the rows that exactly match the join condition between the two tables i.e. returns only the records from Table 1 that has a matching entry in Table 2.

```sql

select first_name, last_name, order_date, order_amount
from customers c
inner join orders o
on c.customer_id = o.customer_id
```

|first_name|	last_name |	order_date |	order_amount |
|-|-|-|-|
George|	Washington |	07/4/1776 |	$234.56|
John|	Adams |	05/23/1784 |	$124.00|
Thomas|	Jefferson |	03/14/1760 |	$78.50|
Thomas|	Jefferson |	09/03/1790 |	$65.50|


#### Left Join:
Return all the rows from Table 1 that has a matching entry in Table 2 PLUS all the remaining rows of Table 1 (that do not have a matching condition on Table 2)

```sql

select first_name, last_name, order_date, order_amount
from customers c
left join orders o
on c.customer_id = o.customer_id
``````

| first_name |	last_name |	order_date |	order_amount |
| ---------- | ---------- | ----------- | ------------- |
| George |	Washington |	07/04/1776 |	$234.56 |
| John |	Adams |	05/23/1784 |	$124.00 |
| Thomas |	Jefferson |	03/14/1760 |	$78.50 |
| Thomas |	Jefferson |	09/03/1790 |	$65.50 |
| James |	Madison |	NULL |	NULL |
| James |	Monroe |	NULL |	NULL |

So why would this be useful? By simply adding a `where order_date is NULL` line to our SQL query, it returns a list of all customers who have not placed an order:

#### Right Join
Return all the rows from Table 2 that has a matching entry in Table 1 PLUS all the remaining rows of Table 2 (that do not have a matching condition on Table 1). This means that some entries of the PK can be null in the new table.

```sql
select first_name, last_name, order_date, order_amount
from customers c
right join orders o
on c.customer_id = o.customer_id
```

| first_name|	last_name	|order_date	|order_amount|
|-|-|-|-|
|George |	Washington |	07/04/1776 |	$234.56|
|Thomas |	Jefferson |	03/14/1760 |	$78.50|
|John |	Adams |	05/23/1784 |	$124.00|
|Thomas |	Jefferson |	09/03/1790 |	$65.50|
|NULL |	NULL |	07/21/1795 |	$25.50|
|NULL |	NULL |	11/27/1787 |	$14.40|

Why is this useful? Simply adding a `where first_name is NULL` line to our SQL query returns a list of all orders for which we failed to record information about the customers who placed them:

#### Full Join
Return the result of inner join PLUS remaining records on Table 1 that do not match the join condition PLUS the remaining records on Table 2 that do not match the join condition.

```sql
select first_name, last_name, order_date, order_amount
from customers c
full join orders o
on c.customer_id = o.customer_id
```

| first_name|	last_name	|order_date	|order_amount|
|-|-|-|-|
|George |	Washington |	07/04/1776 |	$234.56|
|Thomas |	Jefferson |	03/14/1760 |	$78.50|
|John |	Adams |	05/23/1784 |	$124.00|
|Thomas |	Jefferson |	09/03/1790 |	$65.50|
|NULL |	NULL |	07/21/1795 |	$25.50|
|NULL |	NULL |	11/27/1787 |	$14.40|
| James |	Madison |	NULL |	NULL |
| James |	Monroe |	NULL |	NULL |

### Using




