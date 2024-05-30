# Home_Sales
This project demonstrates a comprehensive workflow for analyzing home sales data using SparkSQL, showcasing various capabilities of Apache Spark for big data processing. Below is an elaboration of the steps involved:

### 1. **Importing Packages**
   - **Findspark**: Helps in initializing Spark in the environment.
   - **Pyspark.sql**: Provides functions and classes for working with SparkSQL.

### 2. **Creating a SparkSession**
   - A SparkSession is created to serve as the entry point to Spark. This establishes a connection to the Spark engine and allows for the execution of SQL queries on the DataFrame.
   ```python
   from pyspark.sql import SparkSession
   spark = SparkSession.builder.appName("SparkSQL").getOrCreate()
   ```

### 3. **Reading Data**
   - The home sales data is read from an AWS S3 bucket into a Spark DataFrame. This DataFrame holds the data in a distributed manner, allowing for efficient processing.
   ```python
   home_sales_df = spark.read.csv(SparkFiles.get("home_sales_revised.csv"), sep=",", header=True)
   ```

### 4. **Creating a Temporary View**
   - A temporary view named "home_sales" is created from the DataFrame. This view enables running SQL queries directly on the DataFrame.
   ```python
   home_sales_df.createOrReplaceTempView("home_sales")
   ```

### 5. **Querying the Data**
   - Several SQL queries are executed to analyze the home sales data. Examples include:
     - Calculating the average price for four-bedroom houses sold each year.
     - Determining the average price of homes based on various attributes (bedrooms, bathrooms, square footage).
   ```python
        avg_price_4bed = """
        SELECT
        YEAR(date) AS YEAR,
        ROUND(AVG(price), 2) AS AVERAGE_PRICE
        FROM home_sales
        WHERE bedrooms = 4
        GROUP BY YEAR
        ORDER BY YEAR DESC
        """
        spark.sql(avg_price_4bed).show()
   ```

### 6. **Caching the Data**
   - The temporary table "home_sales_df" is cached to improve query performance by storing data in memory.
   ```python
   spark.catalog.cacheTable("home_sales")
   ```

### 7. **Query Runtime Comparison**
   - The runtime of queries is compared between the cached table and the Parquet data. This involves recording the start time, executing the query, and calculating the difference in time to measure runtime.
   ```python
   import time

   start_time = time.time()
   
   print("Runtime for cached table: %s seconds" % (time.time() - start_time))
   ```

### 8. **Writing Parquet Data**
   - The home sales data is written to Parquet format, partitioned by the "date_built" field. Parquet is a columnar storage file format optimized for big data processing.
   ```python
   home_sales_df.write.partitionBy('date_built').parquet('p_home_sales', mode= 'overwrite')
   ```

### 9. **Reading Parquet Data**
   - The Parquet data is read back into a DataFrame for further analysis.
   ```python
   p_home_sales_df = spark.read.parquet('p_home_sales')
   ```

### 10. **Creating a Temporary Table for Parquet Data**
   - A temporary table named "p_home_sales_df" is created for the Parquet DataFrame.
   ```python
   p_home_sales_df.createOrReplaceTempView('p_home_sales_df')
   ```

### 11. **Querying Parquet Data**
   - SQL queries are executed on the Parquet DataFrame to filter data, such as finding view ratings with an average price greater than or equal to $350,000. Runtime is measured and compared to the cached version.
   ```python
   start_time = time.time()
   ravg_price_per_view_rating = """
    SELECT
    view,
    ROUND(AVG(price), 2) AS AVERAGE_PRICE
    FROM home_sales
    GROUP BY view
    HAVING AVG(price) >= 350000
    ORDER BY view DESC
    """
    spark.sql(avg_price_per_view_rating).show()
   print("Runtime for Parquet data: %s seconds" % (time.time() - start_time))
   ```

### 12. **Uncaching the Temporary Table**
   - The temporary table "p_home_sales_df" is uncached to release memory resources.
   ```python
   spark.catalog.uncacheTable('home_sales')
   ```

### 13. **Checking Caching Status**
   - The caching status of the table is checked to confirm whether it is still cached.
   ```python
   if spark.catalog.isCached('home_sales'):
        print ("The 'home_sales' is cached.")
    else:
        print ("The 'home_sales' is no longer cached.")
   ```

### Conclusion
This project effectively demonstrates how SparkSQL can be utilized to manage, process, and analyze large datasets. It highlights key functionalities such as data reading, transformation, caching for performance optimization, and querying for insightful analytics, making it a robust solution for big data challenges.