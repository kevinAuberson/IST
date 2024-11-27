# Lab 6: Data catalog and Parquet files

_Duration of this lab: 4 periods_

## Pedagogical objectives

At the end of this lab, you will be able to

- Explain the advantages of Parquet files over text files in a data lake
- Create tables and define data types in the AWS data catalog
- Query data in Amazon S3 from Athena
- Optimize queries with partitioning
- Create an AWS Glue crawler to automatically discover table schemas and add them to the data catalog.

#### Tasks

In this lab you will perform a number of tasks and document your progress in a lab report (PDF file). Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

#### Prerequisites

You need an AWS account.

## Task 1: Explore New York City taxi trip data

In this task you will explore a data product published by the New York City Taxi and Limousine commission.

> _New York is widely known for its yellow taxis, and hailing one is just as much a part of the experience of visiting New York as eating a hot dog from a street vendor or riding the elevator to the top of the Empire State Building.
> 
> Residents of New York have all kinds of tips based on their anecdotal experiences about the best times and places to catch a cab, especially during rush hour and when it’s raining. But there is one time of day when everyone will recommend that you simply take the subway instead: during the shift change that happens from 4 to 5 PM every day. During this time, yellow taxis have to return to their dispatch centers (often in Queens) so that one driver can quit for the day and the next one can start, and drivers who are late to return have to pay fines.
> 
> In March of 2014, the New York City Taxi and Limousine Commission shared an infographic on its Twitter account, @nyctaxi, that showed the number of taxis on the road and the fraction of those taxis that was occupied at any given time. Sure enough, there was a noticeable dip of taxis on the road from 4 to 6 PM, and two-thirds of the taxis that were driving were occupied.
> 
> This tweet caught the eye of self-described urbanist, mapmaker, and data junkie Chris Whong, who sent a tweet to the @nyctaxi account to find out if the data it used in its infographic was publicly available. The taxi commission replied that he could have the data if he filed a Freedom of Information Law (FOIL) request and provided the commission with hard drives that they could copy the data on to. After filling out one PDF form, buying two new 500 GB hard drives, and waiting two business days, Chris had access to all of the data on taxi rides from January 1st through December 31st 2013. Even better, he posted all of the fare data online, where it has been used as the basis for a number of beautiful visualizations of transportation in New York City._

From Sandy Ryza, Uri Laserson, Sean Owen and Josh Wills, _Advanced Analytics with Spark_, O'Reilly Media

The data set became famous among data engineers and data scientists, and in February 2019 the taxi commission decided to create a data product and make it publicly available: [TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page). The data is available for download from a Content Distribution Network (CDN). For example yellow cab data for August 2022 is available at

```
https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2022-01.parquet
```

Amazon has decided to include the data in its collection of public data sets [Registry of Open Data on AWS](https://registry.opendata.aws/). This collection contains copies of public data sets or data products which are stored in S3 Buckets. Amazon pays for the hosting.

1. Navigate to the TLC Trip Record Data website. The taxi commission publishes data on four types of cabs. Which are they?
    
2. Find the PDF file with the _data dictionary_ for the yellow cab data on web site. Does it contain the data types?
    
3. The yellow cab data is available in what types of files?
    
4. Find the copy of the data product in the Registry of Open Data on AWS. What is the bucket name? In which region is the bucket? Open the bucket in the S3 console.
    
5. In this lab we are going to use the yellow cab trip data. In which folder are the CSV files for yellow cabs? Does this folder only contain yellow cab data? In which folder are the Parquet files for yellow cabs? Does this folder only contain yellow cab data?
    
6. Is Amazon's copy up-to-date compared to the original data product?
    

Deliverables: responses to questions

## Task 2: Create an entry in the data catalog and query the data

A fundamental component of any data lakehouse is the _data catalog_. Its purpose is to store the schemas of relational data stored in the data lake and other information necessary to access the data. The first data catalog to appear was the [Hive Metastore](https://cwiki.apache.org/confluence/display/hive/design#Design-Metastore). It was developed as a part of [Apache Hive](https://cwiki.apache.org/confluence/display/Hive/Home), "a data warehouse software that facilitates reading, writing, and managing large data sets residing in distributed storage and queried using SQL syntax". Even though it has "Hive" in its name, it is separate from and completely independent of Hive.

Almost all data catalogs are derived from Hive Metastore or compatible with it. This is also the case of AWS. AWS provides a data catalog that it calls [_Glue data catalog_](https://docs.aws.amazon.com/glue/latest/dg/components-overview.html#data-catalog-intro). When you create an AWS account, AWS automatically creates a data catalog in each region.

In this task you will create definitions for the NYC taxi trip data in the data catalog. This information is then used by the AWS Athena service to allow you to run queries on the data. Athena offers an SQL query editor, and you will write SQL Data Definition Language (DDL) statements to define the data in the data catalog.

The data catalog has one important limitation concerning the organisation of the data files. The files of a table must be stored in a dedicated folder. You cannot have other files not belonging to the table in the same folder. What did you notice when you looked at the public AWS bucket of the NYC taxi trip data? The table data is mixed with other data in the same folder. We need to copy the data in a new fresh folder. Luckily Amazon has done that for us for a subset of the data. Explore the S3 folder

```
s3://aws-tc-largeobjects/CUR-TF-200-ACBDFO-1/Lab2/yellow/
```

What subset of the data does the folder contain? In what format?

You are now going to create a data catalog definition for this data.

1. Navigate to the **Athena** console.
    
2. Athena offers an SQL query editor with a history of queries and query results. By default, the history is shared by all IAM users in the account. To separate the history between groups of people, Athena introduces the concept of _workgroups_. Create a new workgroup for the members of you lab group: In the menu bar on the left, click on **Workgroups**. Create a new workgroup:
    
    - **Workgroup name**: follow the lab naming convention and choose a name similar to `ISTLabGrF`.
    - **Query result configuration** > **Location of query result**: Create a bucket in S3 that will contain a log of your queries and their results. Follow the lab naming convention and name the bucket similar to `athena-queries-ist-grf`.
3. Using the Athena query editor, create a new database (in the data catalog, a database is just a namespace for tables). As you share the data catalog with the other lab groups, follow the lab naming convention:
    
    ```
     CREATE DATABASE taxidata_grf;
    ```
    
    Click **Run**. You should see the message _Query successful_.
    
4. In the `taxidata_grf` database, create a new table `yellow` for the yellow cab data for 2017 in CSV format that is stored at
    
    ```
     s3://aws-tc-largeobjects/CUR-TF-200-ACBDFO-1/Lab2/yellow/
    ```
    
    You can use the query editor to write a DDL statements, or you can click on Tables and Views **Create** and let a UI wizard create the DDL for you.
    
    One important piece of information you need to provide is the table schema with the column names and types. Consult the data dictionary from task 1. Does it contain all the information you need? Answer: no, the type information is missing.
    
    One way to obtain the types is to download the CSV file on your local machine and visually inspect the file and guess the types. To avoid this work, we provide the types here:
    
    ```
     VendorID int,
     pickup timestamp,
     dropoff timestamp,
     passenger_count float,
     trip_distance float,
     RatecodeID float,
     store_and_fwd_flag string,
     PULocationID int,
     DOLocationID int,
     payment_type int,
     fare_amount float,
     extra float,
     mta_tax float,
     tip_amount float,
     tolls_amount float,
     improvement_surcharge float,
     total_amount float,
     congestion_surcharge float,
     airport_fee float
    ```
    
    You can copy/paste the columns and their types when you choose **Bulk add columns**.
    
5. Now that you have a table for querying the cab data, you can write queries to retrieve data from the data source in Amazon S3. You can also get a preview of the data in the table by choosing the vertical ellipsis (three dots) on the right of the Athena database window.
    
6. Run a query that displays the first 10 records of the table:
    
    ```
     SELECT * FROM taxidata_grf.yellow
     LIMIT 10;
    ```
    
    When the query completes, you will see a message "Completed - Time in queue: xxx ms Run time: yyy sec Data scanned: zzz MB". Write down the run time and the volume of data scanned.
    
7. When you work with large datasets, optimizing performance and minimizing costs are two major goals. You determine your costs for running Athena based on usage. Usage is based on the amount of data that is scanned. Prices vary based on your Region. For more information about Athena pricing, see [Amazon Athena Pricing](https://aws.amazon.com/athena/pricing/). How much did the last query cost?
    

Deliverables:

- In the Athena console display the DDL definition of the table and copy it into the report.
- Responses to questions

## Task 3: Optimise the query by scanning only a partition of the data

The Athena query engine is based on an open source tool called [Presto](https://prestodb.github.io/docs/0.172/overview/use-cases.html). To read more about the SQL functions that you can use with Athena, see [SQL Queries, Functions, and Operators](https://docs.aws.amazon.com/athena/latest/ug/functions-operators-reference-section.html).

As you have seen in the previous task, the cost of running Athena is based on the amount of data that is scanned. To minimise cost you have several options:

1. Compress the data (for example using gzip, bzip2 or others).
2. Convert the data to a compact binary format such as Parquet.
3. Partition the data.

In this task you are going to explore the last option, partitioning of the data. You will compare the query time and the amount of data that is scanned for two different ways of loading the taxi trip data into Athena. In the previous task, you created a table that contains all of the taxi trip data for 2017. Next, you will create a table that contains the data for a single month. By comparing the query processing time for the two tables, you will see which approach is faster and less expensive.

1. Create a new table that contains just the data for January 2017 called `jan`, stored at
    
    ```
     s3://aws-tc-largeobjects/CUR-TF-200-ACBDFO-1/Lab2/January2017/
    ```
    
2. Run the following query on the data for the whole of 2017 that summarises trips in January 2017:
    
    ```
     SELECT count(passenger_count) AS "Number of trips",
            sum(total_amount) AS "Total fares",
            pickup AS "Trip date"
     FROM yellow WHERE pickup
     between TIMESTAMP '2018-01-01 00:00:00'
         and TIMESTAMP '2018-02-01 00:00:01'
     GROUP BY pickup;
    ```
    
    Write down the amount of data that was scanned.
    
3. Run the following query on the data for January 2017 only:
    
    ```
     SELECT count(passenger_count) AS "Number of trips" ,
              sum(total_amount) AS "Total fares" ,
              pickup AS "Trip date"
     FROM jan
     GROUP BY pickup;
    ```
    
    Write down the amount of data that was scanned.
    

Deliverables:

- In the Athena console display the DDL definition of the table and copy it into the report.
- Responses to questions

## Task 4: Create a partitioned table in the data catalog with a Glue crawler

As you have seen in the previous task, partitioning the data can reduce enormously the amount of data that needs to be scanned for a query. It is therefore good practice to organise the data of a table in multiple folders. A folder is called a partition.

For example, we could partition the NYC taxi trip data by year like this:

```
s3://bucket/yellow/2019/yellow_tripdata_2019-01.parquet
s3://bucket/yellow/2019/yellow_tripdata_2019-02.parquet
s3://bucket/yellow/2019/yellow_tripdata_2019-03.parquet
...
s3://bucket/yellow/2020/yellow_tripdata_2020-01.parquet
s3://bucket/yellow/2020/yellow_tripdata_2020-02.parquet
s3://bucket/yellow/2020/yellow_tripdata_2020-03.parquet
...
```

We then have to tell the data catalog about our partitions. We need to store the information that all files for the year 2019 can be found in the folder `s3://bucket/yellow/2019/`, all files for the year 2020 in the folder `s3://bucket/yellow/2019/`, and so on.

Also, when we later make a query on the partitioned table, it would be good if we could tell the SQL query engine that we are only interested in, say, the year 2022 when we make a query so that it can avoid scanning the other years.

We can do this through the data catalog concept of partitioned tables. In addition to remembering the folder for each partition, the data catalog adds a _virtual column_ to the table schema that the SQL query engine can use. Let's say we call the virtual column _year_. When we write the query

```
SELECT * FROM yellow
WHERE year = '2020';
```

only the data for 2020 will be scanned.

We can also nest several partitions. For example we can partition by year _and_ month like this:

```
s3://bucket/yellow/2019/01/yellow_tripdata_2019-01.parquet
s3://bucket/yellow/2019/02/yellow_tripdata_2019-02.parquet
s3://bucket/yellow/2019/03/yellow_tripdata_2019-03.parquet
...
s3://bucket/yellow/2020/01/yellow_tripdata_2020-01.parquet
s3://bucket/yellow/2020/02/yellow_tripdata_2020-02.parquet
s3://bucket/yellow/2020/03/yellow_tripdata_2020-03.parquet
...
```

With these partitions we add _two_ virtual columns to our table schema, _year_ and _month_.

And we can put the names of the virtual columns directly into the folder names like this:

```
s3://bucket/yellow/year=2019/month=01/yellow_tripdata_2019-01.parquet
s3://bucket/yellow/year=2019/month=02/yellow_tripdata_2019-02.parquet
s3://bucket/yellow/year=2019/month=03/yellow_tripdata_2019-03.parquet
...
s3://bucket/yellow/year=2020/month=01/yellow_tripdata_2020-01.parquet
s3://bucket/yellow/year=2020/month=02/yellow_tripdata_2020-02.parquet
s3://bucket/yellow/year=2020/month=03/yellow_tripdata_2020-03.parquet
...
```

This way of naming partitions was invented for the Hive query engine and is called _Hive-style partitioning_.

In this task you will create a partitioned table in the data catalog by using a _Glue crawler_. A _Glue crawler_ recursively crawls the files in a given location, analyses their content, derives schema information, and detects partitions. At the end of the crawl it writes all the information it has found into data catalog.

We have prepared for you a partitioned version of the NYC taxi trip data at location

```
s3://heigvd-ist-prepared/nyc-tlc/yellow/
```

1. Navigate to the _AWS Glue_ console.
    
2. Create a new crawler: On the left menu choose **Crawlers**, then **Create crawler**.
    
    - **Crawler name**: following the lab naming convention name it similar to `nyc-tlc-grf`
    - **Data sources**: Choose **Add a data source**.
        - **S3 path**: Give the path of the partitioned NYC taxi trip data.
        - Choose **Add an S3 data source**.
    - After returning from the dialog, select the data source you just added.
    - **Existing IAM role**: Select _AWSGlueServiceRoleDefault_.
    - **Target database**: Select the name of your NYC taxi trip database.
    - **Table name prefix**: Enter `part` to distinguish the new table that the crawler will create from the existing one.
    
    After you have created the crawler, select it and run it. After a while the crawler should display _Succeeded_.
    
3. Examine the newly created table: On the left menu choose **Databases**. Select your NYC taxi trip database and locate the newly created table. Click on it to view detailed information. View the schema. What virtual columns were added to the schema? Also click on **Partitions** to view the partitions of the table.
    
4. In Athena write a query that makes use of the added virtual columns to restrict the scanning of data to August of 2022. How much data was scanned?
    

Deliverables:

- In the Athena console display the DDL definition of the table and copy it into the report.
- Responses to questions

## Task 5: Explore and transform data with Glue DataBrew

In this task you will use the AWS service Glue DataBrew to infer a data schema from the CSV data that is better than the schema of the Parquet files in the Amazon public data set.

Caution: When you work with DataBrew, you create a session with a virtual machine that is allocated to you. You are billed for the session in 30-minute increments. A 30-minute interval costs $1.00. The session is automatically closed at the end of the 30-minute period if there is no activity from you. With that in mind, you should avoid starting a session and then leave it to do something else.

1. Navigate to the DataBrew console. Create a new project.
    
    - **Project name**: Follow the lab naming convention and name it similar to `nyc-tlc-grf`.
        
    - **Recipe details**: Create new recipe.
        
    - **Select a dataset**: Choose new dataset. Select the bucket of the Amazon public data set of NYC taxi trip data, and select the folder containing CSV files. Use a regular expression (explained on the right) to select only files for the yellow cabs, and only from 2022 (there a