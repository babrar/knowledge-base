# Intermediate SQL

## Window functions

So far, we've been using `GROUP BY ` to cluster our data into groups. However, we've seen before that GROUP BY performs aggregation on groups of data and returns the result of the aggregation back to the user. In this process, information about the individual rows are lost since we get an aggregated metric per group of data rather than the rows of the entity.

*Problem*: 

We have a slowly changing dimension for customers where records are appended with a data creation timestamp. We wish to find the most recent address of all customers.

*Solution*: 

Note: The issue with just doing  

```sql
select name, email, address from customer
```

is that there will be duplicate rows since a customer can have their address updated -> in which case, a new row for that customer will exist with a different address and a newer data creation timestamp.

We can solve this problem by grouping together all customers using a Primary Key (i.e. email) and then picking the latest record for each customer.  This approach of bundling records into groups and then processing on the groups is achieved using  **window functions**.

```sql
select 
	*,
	row_number() over (partition by email order by date_created desc) as customer_rank
from customers
where customer_rank = 1
```

