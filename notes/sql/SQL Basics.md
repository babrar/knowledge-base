# SQL Basics

## Relational Algebra -> SQL
| Logical data concept | SQL Equivalent |
| -------------------- | -------------: |
| Entity               |          Table |
| Attribute            | Column Heading |
| Value                |   Row Contents |

Apart from entities, the relationship between entities are also captured by tables. 


## Schema Design


### Primary Key
(PK). A unique identifier for each entry of a table. Usually implemented as a sequence with initial value of 1 and interval of 1.

### Foreign Key
An alternative to having a separate table for represting relationships, we can add a column to one of our entity tables that represent the keys from the other entity. This new column is called the Foreign Key (FK) of the table.

Usually the FK assigned to the *child* table or entity.
>You can identify the parent table by asking which table can exists on its own without the presence of the other table.

Example: If we had two tables called `Customers` and `Orders`, it is easy to observe that customers can exists independently without orders. However, even though orders can exists without referencing customers, it doesn't make sense to talk about orders without them depending on customers. This `Orders` will be the child table and get assigned the FK (customer id).

### Designing relationships using tables

As mentioned before, relationships between entities are represented using a table (or a FK column in a table). Based on the nature of the relationships, we have to decide which table design to pick from. The 3 possible cases are outline below. Full discussion [here](https://stackoverflow.com/questions/7296846/how-to-implement-one-to-one-one-to-many-and-many-to-many-relationships-while-de).

#### One-to-one relationships

Simply use a foreign key referenced table. This involves adding a FK column in the child table. Additionally, apply a `unique` constraint on the FK such that no two rows of the child table can reference the same row from the parent table. For example:
```
student: student_id, first_name, last_name, address_id (UNIQUE FK)
address: address_id, address, city, zipcode, student_id (UNIQUE FK)
```
Note: we use 2 FKs across each other for link-back purposes. We could have used one FK only i.e. `student_id`.

We can view the addresses for each student using the following query:

```sql
SELECT first_name, last_name, address, city
FROM student
INNER JOIN address USING student_id
```

 We'll cover SQL joins in more detail in a later section.

#### Many-to-one relationships

Use a foreign key on the many side of the relationship linking back to the "one" side:

```
teachers: teacher_id, first_name, last_name # the "one" side
classes:  class_id, class_name, teacher_id  # the "many" side
```

We can view information about each class using the following query:

```sql
SELECT class_name, first_name, last_name
FROM classes
LEFT JOIN teaches USING teacher_id
```

#### Many-to-many relationships
Take for example `students` and `classes`. As you may realize, the relationship cannot be represented by using a FK column in either of the tables. This is because a student can have multiple classes and a class can have multiple students.

The right approach here will be creating another table called a **junction table** which will hold references between student and classes.

```
students: student_id, first_name, last_name
classes: class_id, name, teacher_id
students_classes: class_id, student_id    # the junction table
```

The students table and the classes table will only interact with the junction table, not with each other. For example, the following lists some queries that can run to obtain information that related students to classes and vice versa.

```sql
 -- Getting all students for a class:

    SELECT s.student_id, last_name
      FROM student_classes sc 
INNER JOIN students s ON s.student_id = sc.student_id
     WHERE sc.class_id = X

 -- Getting all classes for a student: 

    SELECT c.class_id, name
      FROM student_classes sc 
INNER JOIN classes c ON c.class_id = sc.class_id
     WHERE sc.student_id = Y
```

>The students table and the classes table will only interact with the junction table, not with each other

Notice how that's true for the queries above.


## Aggregations

Aggregation functions are commonly known as `COUNT`, `AVG` and `SUM`. 
These agg functions take a column name and essentially performs a *reduce* operation on the entire row. For example, the following SQL statement returns the total number of orders in the orders table. 
```sql
SELECT COUNT(ORDER_ID)
FROM ORDERS

-- Returns: N (where N = number of rows in ORDERS) 
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
Note: `GROUP BY 1` means to group by the 1st column in the select statement regardless of the the column's name. `GROUP BY 2` means to group by the 2nd column and so on.
### Conditional aggregation
More often than not, we want to aggregate over entries that meet a certain condition. We can achieve that by using the `CASE WHEN [condition] THEN [on true] ELSE [on false] END` statement inside an aggregation function. For example if we wanted to find the amount of money each customer spent on books, we  can write:

```sql
SELECT SUM(CASE WHEN item_type = 'book' THEN amount ELSE 0 END)
FROM ORDERS
GROUP BY CUSTOMER_ID

-- Returns: Table with cols: CUSTOMER_ID, Amount spent on books.
```

### 'Having' Keyword

If we want to sort our results post-aggregation, we cannot use the `WHERE` clause since it does not support aggregation. Instead, we have to use the `HAVING` clause. This clause must be a follow up to a `GROUP BY` clause. For example, the following SQL statement lists the number of customers in each country (only include countries with more than 5 customers):

```sql
SELECT COUNT(CustomerID) as num_customers, Country
FROM Customers
GROUP BY Country
HAVING num_customers > 5 -- using WHERE here will fail
```

Note: In addition to not supporting aggregations, a `WHERE` clause must be used before a `GROUP BY` clause. So you can see the proble if we want to filter results post-aggregation. 

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

| customer_id | first_name | last_name  | city            | state | zipcode |
| :---------- | :--------- | :--------- | :-------------- | :---- | :------ |
| 1           | George     | Washington | Mount Vernon    | VA    | 22121   |
| 2           | John       | Adams      | Quincy          | MA    | 02169   |
| 3           | Thomas     | Jefferson  | Charlottesville | VA    | 22902   |
| 4           | James      | Madison    | Orange          | VA    | 22960   |
| 5           | James      | Monroe     | Charlottesville | VA    | 22902   |


*Fig: Orders Table*

| order_id | order_date | amount  | customer_id |
| -------- | ---------- | ------- | ----------- |
| 1        | 07/04/1776 | $234.56 | 1           |
| 2        | 03/14/1760 | $78.50  | 3           |
| 3        | 05/23/1784 | $124.00 | 2           |
| 4        | 09/03/1790 | $65.50  | 3           |
| 5        | 07/21/1795 | $25.50  | 10          |
| 6        | 11/27/1787 | $14.40  | 9           |


#### Inner Join
This type of join returns all the rows that exactly match the join condition between the two tables i.e. returns only the records from Table 1 that has a matching entry in Table 2.

```sql
select first_name, last_name, order_date, order_amount
from customers c
inner join orders o
on c.customer_id = o.customer_id
```

| first_name | last_name  | order_date | order_amount |
| ---------- | ---------- | ---------- | ------------ |
| George     | Washington | 07/4/1776  | $234.56      |
| John       | Adams      | 05/23/1784 | $124.00      |
| Thomas     | Jefferson  | 03/14/1760 | $78.50       |
| Thomas     | Jefferson  | 09/03/1790 | $65.50       |


#### Left Join:
Return all the rows from Table 1 that has a matching entry in Table 2 PLUS all the remaining rows of Table 1 (that do not have a matching condition on Table 2)

```sql
select first_name, last_name, order_date, order_amount
from customers c
left join orders o
on c.customer_id = o.customer_id
``````

| first_name | last_name  | order_date | order_amount |
| ---------- | ---------- | ---------- | ------------ |
| George     | Washington | 07/04/1776 | $234.56      |
| John       | Adams      | 05/23/1784 | $124.00      |
| Thomas     | Jefferson  | 03/14/1760 | $78.50       |
| Thomas     | Jefferson  | 09/03/1790 | $65.50       |
| James      | Madison    | NULL       | NULL         |
| James      | Monroe     | NULL       | NULL         |

So why would this be useful? By simply adding a `where order_date is NULL` line to our SQL query, it returns a list of all customers who have not placed an order:

#### Right Join
Return all the rows from Table 2 that has a matching entry in Table 1 PLUS all the remaining rows of Table 2 (that do not have a matching condition on Table 1). This means that some entries of the PK can be null in the new table.

```sql
select first_name, last_name, order_date, order_amount
from customers c
right join orders o
on c.customer_id = o.customer_id
```

| first_name | last_name  | order_date | order_amount |
| ---------- | ---------- | ---------- | ------------ |
| George     | Washington | 07/04/1776 | $234.56      |
| Thomas     | Jefferson  | 03/14/1760 | $78.50       |
| John       | Adams      | 05/23/1784 | $124.00      |
| Thomas     | Jefferson  | 09/03/1790 | $65.50       |
| NULL       | NULL       | 07/21/1795 | $25.50       |
| NULL       | NULL       | 11/27/1787 | $14.40       |

Why is this useful? Simply adding a `where first_name is NULL` line to our SQL query returns a list of all orders for which we failed to record information about the customers who placed them:

#### Full Join
Return the result of inner join PLUS remaining records on Table 1 that do not match the join condition PLUS the remaining records on Table 2 that do not match the join condition.

```sql
select first_name, last_name, order_date, order_amount
from customers c
full join orders o
on c.customer_id = o.customer_id
```

| first_name | last_name  | order_date | order_amount |
| ---------- | ---------- | ---------- | ------------ |
| George     | Washington | 07/04/1776 | $234.56      |
| Thomas     | Jefferson  | 03/14/1760 | $78.50       |
| John       | Adams      | 05/23/1784 | $124.00      |
| Thomas     | Jefferson  | 09/03/1790 | $65.50       |
| NULL       | NULL       | 07/21/1795 | $25.50       |
| NULL       | NULL       | 11/27/1787 | $14.40       |
| James      | Madison    | NULL       | NULL         |
| James      | Monroe     | NULL       | NULL         |

### 'Using' keyword
If two tables (i.e. orders and customers) are being joined using a column name  that is same across both the tables (i.e. customer_id), then instead of specifying the joining condition using the `join on order.customer_id = customer.customer_id` syntax, we can concisely write `join using customer_id`.

## Things to watch out for
### Column name collisions

The following SQL doesn't behave as expected due a column name collision. 
```sql
orders_unknown_customer as (
  select 
    orders.id as order_id, -- PK
    orders.customer, -- contains FK (customer_id)
    customers.id as customer_id -- COLUMN NAME ALIAS DOES NOT TAKE EFFECT HERE
  from orders left join customers
  on (orders.customer_id = customers.id)
  where customers_id is null -- customer_id inherited from orders, not the current table
)
select * from orders_unknown_customer;
```

`orders`already  has a column named `customer_id`. When we build a new table called `orders_unknown_customer` we are introducing a new column in the table called `customer_id`.

