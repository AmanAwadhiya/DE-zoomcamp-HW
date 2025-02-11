## Module 3 Homework

<b>Important Note:<b>

For this homework, I will be using the Yellow Taxi Trip Records from January 2024 to June 2024, available as Parquet files from the New York City Taxi Data. The data can be found here:
</br> https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page


<b>BIG QUERY SETUP:</b></br>
-- Create an external table using the Yellow Taxi Trip Records. </br>
-- Create a (regular/materialized) table in BQ using the Yellow Taxi Trip Records (do not partition or cluster this table).

## Prerequisites
Go to [Google Cloud Console](https://console.cloud.google.com) (Ensure you have a Google Cloud project with billing enabled).

Create a GCS Bucket and set a unique name as `dezoomcamp_hw3_2025_a`.

Click on your bucket, then click `Upload Files` and select your `.parquet` files from the Downloads folder.

----Verification: Ensure all six `.parquet` files appear in your bucket before proceeding.

Go to BigQuery or click on [BigQuery](https://console.cloud.google.com/bigquery).

Create a dataset named `dezoomcamp_hw3_2025`.

Create an external table using the below SQL query in the BigQuery SQL workspace.
```sql
CREATE OR REPLACE EXTERNAL TABLE `dezoomcamp_hw3_2025.yellow_taxi_ext`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://dezoomcamp_hw3_2025_a/*.parquet']
);
```

Now, create a regular table using the external table.
```sql
CREATE OR REPLACE TABLE `(Replace_with_Your_Project).dezoomcamp_hw3_2025.yellow_taxi`
AS
SELECT * FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.yellow_taxi_ext`;
```

All steps are done. Now, proceed with the question and answer.

## Questions + Answers

Q1: What is count of records for the 2024 Yellow Taxi Data?
- 65,623
- 840,402
- 20,332,093
- 85,431,289

Answer: 20,332,093
```sql
SELECT COUNT(*) FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.yellow_taxi`;
```

Q2: Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.</br> 
What is the **estimated amount** of data that will be read when this query is executed on the External Table and the Materialized Table?
- 18.82 MB for the External Table and 47.60 MB for the Materialized Table
- 0 MB for the External Table and 155.12 MB for the Materialized Table
- 2.14 GB for the External Table and 0MB for the Materialized Table
- 0 MB for the External Table and 0MB for the Materialized Table

Answer: 0 MB for the External Table and 155.12 MB for the Materialized Table
```sql
SELECT COUNT(DISTINCT PULocationID) FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.yellow_taxi_ext`;

SELECT COUNT(DISTINCT PULocationID) FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.yellow_taxi`;

-- Don't execute. Check the estimated amount of data in the top-right corner.
```

Q3: Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?
- BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires 
reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.
- BigQuery duplicates data across multiple storage partitions, so selecting two columns instead of one requires scanning the table twice, 
doubling the estimated bytes processed.
- BigQuery automatically caches the first queried column, so adding a second column increases processing time but does not affect the estimated bytes scanned.
- When selecting multiple columns, BigQuery performs an implicit join operation between them, increasing the estimated bytes processed.

Answer: BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires 
reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.
```sql
SELECT PULocationID FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.yellow_taxi`;

SELECT PULocationID, DOLocationID FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.yellow_taxi`;
```

Q4: How many records have a fare_amount of 0?
- 128,210
- 546,578
- 20,188,016
- 8,333

Answer: 8333
```sql
SELECT COUNT(*) AS zero_fare_count
FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.yellow_taxi`
WHERE fare_amount = 0;
```

Q5: What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)?
- Partition by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on tpep_dropoff_datetime Partition by VendorID
- Partition by tpep_dropoff_datetime and Partition by VendorID

Answer: Partition by tpep_dropoff_datetime and Cluster on VendorID
```sql
CREATE OR REPLACE TABLE `(Replace_with_Your_Project).dezoomcamp_hw3_2025.optimized_table`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.yellow_taxi`;
```

Q6: Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime
2024-03-01 and 2024-03-15 (inclusive)

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values?

Choose the answer which most closely matches.

- 12.47 MB for non-partitioned table and 326.42 MB for the partitioned table
- 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table
- 5.87 MB for non-partitioned table and 0 MB for the partitioned table
- 310.31 MB for non-partitioned table and 285.64 MB for the partitioned table

Answer: 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table
```sql
-- Non-partitioned table or regular table
SELECT DISTINCT VendorID  
FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.yellow_taxi`  
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15';

-- partitioned table
SELECT DISTINCT VendorID  
FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.optimized_table`  
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15';
```

Q7: Where is the data stored in the External Table you created?
- Big Query
- Container Registry
- GCP Bucket
- Big Table

Answer: GCP Bucket

Q8: It is best practice in Big Query to always cluster your data:
- True
- False

Answer: False

However, clustering is not always necessary. If a table is small, clustering may not provide significant benefits and can add unnecessary overhead.

Q9: Write a `SELECT count(*)` query FROM the materialized table you created. How many bytes does it estimate will be read? Why?

Answer: Usually 0 byte

In BigQuery, when running COUNT(*) on a materialized table, the estimated bytes read is small (usually 0 or just a few bytes). Since the table is already materialized, BigQuery does not need to scan the entire dataset
```sql
SELECT COUNT(*)  
FROM `(Replace_with_Your_Project).dezoomcamp_hw3_2025.optimized_table`;
```
