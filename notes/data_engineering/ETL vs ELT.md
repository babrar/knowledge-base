---
title: ETL vs ELT
created: '2021-01-06T16:02:31.551Z'
modified: '2021-01-12T20:03:01.810Z'
---

# ETL vs ELT

Described in detail [here](https://www.xplenty.com/blog/etl-vs-elt/#:~:text=The%20five%20critical%20differences%20of,warehouse%20to%20do%20basic%20transformations.).


Key points:
**ELT** is ideal for a Data Lake setup where unformatted raw data is uploaded. In previous implementation, Redshift required the uploaded data to be formatted (i.e. ETL) into a relational format before loading into data warehouse.

In Summary:
- ETL stands for Extract, Transform, and Load, while ELT stands for Extract, Load, and Transform.
- In ETL, data flow from the data source to staging to the data destination.
- ELT lets the data destination do the transformation, eliminating the need for data staging.
- ETL can help with data privacy and compliance, cleansing sensitive data before loading into the data destination, while ELT is simpler and for companies with minor data needs.


NOTE: In ETL, there are additional implicit steps before Transforming the data. Those steps involve pulling the data from the warehouse into a separate machine for computing the transformation and another step to push back the data into the warehouse. This is required becuase the data storage is decoupled from the compute unit. However, in a cloud based data warehouse like Snowflake, these steps are not necessary since the transformation can take in place. Thus, it is called ELT.
