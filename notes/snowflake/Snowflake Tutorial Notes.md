---
tags: [ELT, Snowflake]
title: Snowflake Tutorial Notes
created: '2021-01-06T16:46:47.801Z'
modified: '2021-01-11T18:22:40.069Z'
---

# Snowflake Tutorial Notes


To run any query in Snowflake a warehouse is needed. Warehouses in Snowflake is treated as compute units.

>In Snowflake, when speaking about a Warehouse, we are not speaking of a place for storing data, we are always speaking about compute power that is brought to bear for data processing. 

Comparing real-life warehouses with Snowflake, we get

| Real-world terms     | Snowflake Equivalent |
| -------------------- |:------------------:  |
| Warehouse            | Cluster              |
| Workers              | Servers              |
| Goods                | Data                 |



N credits/hr for a warehouse indicates that there are N servers(workers) in that warehouse.


1 cluster = 1 warehouse.

##  Scaling

Scaling warehousing *UP/DOWN* refers to increasing the size of the warehouse (i.e. increasing the number of servers in the warehouse.). This is a manual process. Warehouse sizes can vary from XS to 3X-Large.
### Elastic Data Warehousing

This refered to  as scaling *IN/OUT* which happens automatically and the extent of scaling is configurable. This feature is only available for Enterprise editions or above.

There can be multiple clusters of warehouse from a base warehouse template (i.e. each warehouse will have the same number of workers within it).

## Relational Algebra \& SQL basics 
| Logical data concept | SQL Equivalent     |
| -------------------- |------------------: |
| Entity               | Table              |
| Attribute            | Column Heading     |
| Value                | Row Contents       |

Apart from entities, the relationship between entities are also captured by tables. 

### Sequences
A primary key that acts as a unique identifier for entries of a table (i.e. Authors of a book can have the same name, but their sequences will be unique.). Sequences need to be created with an initial value and an interval.

### DDL vs DML
Both DDL and DML refers to SQL statements. DDL expands to Data Definition Language which mostly consists of table/data definitions (i.e. statements like `CREATE`, `DROP`, `ALTER`). By contrast, DML is an acronym for Data Manipulation Language which refers to SQL statements like `SELECT`, `INSERT`, `UPDATE`. 

## Snowflake ETL Basics

In terms of Snowflake, ETL can be described by the following table:

| ETL Stage     | SF SQL Equivalent  |
| ------------- |------------------: |
| Extract       | `FROM`             |
| Transform     | `REPLACE`          |
| Load          | `INSERT INTO`      |



### Staging

#### Internal Staging
This is analogous to storing data in temporary tables. Refer to tables with an INGEST suffix created during the Snowflake tutorials.

#### External Staging
- S3, GCP, Azure data can be directly **loaded** into a table using COPY INTO command. I.E.  
```sql
COPY INTO "USDA_NUTRIENT_STD_REF"."PUBLIC"."WEIGHT_INGEST"
FROM @my_s3_bucket/load/ -- the external stage we pull from
FILES = ('WEIGHT.txt')
FILE_FORMAT = (FORMAT_NAME = USDA_FILE_FORMAT);
```
- S3 (csv-like) data can be queried without loading into table first. The column names have to be $1, $2 and so on..
This allows a nice way to skip loading raw data by **transforming** the data enroute. I.E.
```sql
-- PREVIEW from S3 directly
select REPLACE($1, '~'), REPLACE($2, '~')
from @my_s3_bucket/load/LANGDESC.txt
(FILE_FORMAT => USDA_FILE_FORMAT);

-- WRITE TO DB
COPY INTO LANGDESC(FACTOR_CODE, DESCRIPTION)
FROM (
  select REPLACE($1, '~'), REPLACE($2, '~')
  from @my_s3_bucket/load/LANGDESC.txt
  (FILE_FORMAT => USDA_FILE_FORMAT)
)
```

## Semi-structured data (non-nested)

This type of data is labelled as `VARIANT` datatype in Snowflake. Common data semi-structured data formats are JSON, XML, Parquet. 

Supported semi-structured data formates (see [here](https://docs.snowflake.com/en/user-guide/semistructured-intro.html)) can be loaded into Snowflake. Each outermost entry will be placed in a new row in the database (given that we strip the outermost container using the `STRIP_OUTER_ELEMENT` property).

### XML
Lesson 10 covers extensively on how to extact values out of nested fields and tags. This way, we can make semi-structured query-able. Some common symbols for the use case are "$" - Field value and "@" - Attribute value. Example:
```sql
SELECT
raw_author:"@AUTHOR_UID" as AUTHOR_ID
,XMLGET(raw_author, 'FIRST_NAME'):"$"::STRING as FIRST_NAME
,XMLGET(raw_author, 'MIDDLE_NAME'):"$"::STRING as MIDDLE_NAME
,XMLGET(raw_author, 'LAST_NAME'):"$"::STRING as LAST_NAME
FROM AUTHOR_INGEST_XML
```

### JSON

```sql
SELECT
 raw_author:AUTHOR_UID
,raw_author:FIRST_NAME::STRING as FIRST_NAME
,raw_author:MIDDLE_NAME::STRING as MIDDLE_NAME
,raw_author:LAST_NAME::STRING as LAST_NAME
FROM AUTHOR_INGEST_JSON;
```


## Semi-structured nested data

**Cast**: specified as a suffix by using the `::` symbol.

For almost all nested operation, the `FLATTEN` command provided by Snowflake is necessary. (see: [doc](https://docs.snowflake.com/en/sql-reference/functions/flatten.html)). There are two possible ways to flatten data:
- LATERAL FLATTEN
- TABLE FLATTEN

Flattening works by taking a param to the `FLATTEN` function and for each occurenence of that param in a nested data structure, it places that occurence in a new row. i.e. this analogous to sifiting through multiple twitter tweets (each in its own row) and finding the hashtags for each tweet (one tweet can have multiple hashtags). The hashtags would then be placed into separate rows of their own upon flattening.

The LATERAL modifier joins the provided table with the flattened data . That way, we can SELECT columns that are not directly present in the flatten operation. 

For example:
```sql
SELECT RAW_STATUS
,value:text::VARCHAR AS HASHTAG_TEXT
FROM TWEET_INGEST
,LATERAL FLATTEN
(input => RAW_STATUS:entities:hashtags);

```
`RAW_STATUS` is only present in the `TWEET_INGEST` table and not in the result of the `FLATTEN` operation.


### JSON
After flattening, we can extract data from each key as we would for json using the `:` syntax. Example:
```sql
-- Use these example flatten commands to explore flattening the nested book and author data
SELECT VALUE:first_name -- look at the docs for flatten. VALUE is a column returned by FLATTEN.
FROM NESTED_INGEST_JSON
,LATERAL FLATTEN(input => RAW_NESTED_BOOK:authors); -- lambda param name has to be 'input' according to the docs.
```

And here's the same example as above but using `TABLE FLATTEN`:
```SQL
SELECT value:first_name
FROM NESTED_INGEST_JSON
,table(flatten(RAW_NESTED_BOOK:authors));
```

## VIEW

Views are just SQL queries that can be run on a dataset to generate tables.


























