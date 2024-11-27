# Lab 5: Serverless data ingestion and processing

_Duration of this lab: 6 periods_

## Pedagogical objectives:

In this lab you will use a real-world data product from MeteoSwiss, the current meteorology measurement values of all Swiss automatic weather stations, and use it in S3. You will write an Amazon Lambda function to download data from MeteoSwiss and store it in S3.

#### Tasks

In this lab you will perform a number of tasks and document your progress in a lab report (PDF file). Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

#### Prerequisites

You need an AWS account.

## Task 1: Explore MeteoSwiss data

[opendata.swiss](https://opendata.swiss/) is the Swiss public administration’s central portal for Open Government Data (OGD). It provides access to the public data of the Confederation, cantons and communes.

Its objectives are threefold:

- To provide an overview of the OGD currently published in Switzerland in order to facilitate an exchange between the data provider and data users;
- To create an environment promoting the use of public data;
- To support the creation of a key part of the infrastructure for Swiss data.

The portal is limited to referencing data. The data are hosted by their owner.

1. Use the portal's search function to search for meteorology data products from MeteoSwiss.
    
    Locate the following two data products:
    
    - "Automatic weather stations – Current measurement values"
    - "Weather stations of the automatic monitoring network"
2. For each data product download the zip file that contains its description. On your local machine create a folder for each data product where you store the content of the zip file in a `metadata` subfolder.
    
3. For each data product follow the instructions you find in the metadata to download a data sample to your folders.
    
    - If there are several versions in different languages available, choose the English version.
        
    - Note that the "Current measurement values" are dynamic: the data is updated every 10 minutes. There is also a separate document explaining the column meanings.
        
    - The data set containing the weather stations of the automatic monitoring network is static.
        
4. Explore the measurement values.
    
    - When you open the CSV file with the measurement table in Excel, you will see that Excel does not parse the fields correctly. Make a copy of the file and in the copy insert as first line `sep=;` to indicate that fields are separated by semicolons.
        
    - Data plausibility check: In the list of weather stations find one near you and write down its three-letter ID. In the description of columns find the temperature column. In the measurement table look up the current temperature measurement of the station. Does it correspond to what your thermometer or MeteoSwiss' website or mobile app shows?
        
    - In the measurement table examine the _Date_ column (you may have to change its format to see it properly). What does it contain exactly? Precision?
        

Deliverables:

- For the two data products copy the URLs where the data can be downloaded in the report.
- Document your exploration of the measurement values.
- What is your impression of the the opendata.swiss portal and of MeteoSwiss' data products?

## Task 2: Upload the current measurement data to S3 and run SQL queries on it

In this task you will upload the data sample of the current measurements to S3 and run SQL queries on it using Amazon Athena.

1. In S3 create a bucket for your group. Give it a name following the lab naming conventions, similar to `ist-meteo-grf-nicollier-piccard`.
    
2. In the bucket create a folder `current`. Into the folder upload the CSV file of the current measurements (the original, not the one with the `sep=;` line).
    
    Also create a folder `query-results`, which we will need in the next step.
    
3. Set up Athena and specify an S3 location to hold the query results: Open the **Athena** console. Using the menu open the **Query editor**. You will see a message "Before you run your first query, you need to set up a query result location in Amazon S3.". Click on the **Edit settings** button. Specify the `query-results` folder you created previously.
    
4. Create a new empty database. Give it a name following the lab naming conventions, similar to `meteoswiss_grf`. In the query editor, type the query:
    
    ```
     CREATE DATABASE meteoswiss_grf;
    ```
    
    and run it.
    
5. Create a table in the database from the uploaded data: In the **Database** drop-down select the newly created database. Then in **Tables and views** create a new table from S3 bucket data.
    
    - **Table details** > **Table name**: `current`
    - **Database configuration**: existing database `meteoswiss_grf`
    - **Dataset**: navigate to the `current` folder in your bucket
    - **Data format**
        - **File format**: CSV
        - **field.delim**: `;` (semicolon)
    - **Column details** > **Bulk add columns**
        
        ```
          station varchar, datetime bigint, temperature float, precipitation float, sunshine float, radiation float, humidity float, despoint float, wind_dir float, wind_speed float, gust_peak float, pressure float, press_sea float, press_sea_qnh float, height_850_hpa float, heigh_700_hpa float, wind_dir_vec float, wind_speed_tower float, gust_peak_tower float, temp_tool1 float, humidity_tower float, dew_point_tower float
        ```
        
        Click **Add**. Then complete the varchar length of column station (set it to 3).
        
        At the bottom of the page you will see the SQL query that is constructed from your input.
        
        Click **Create table**. You will be brought back to the query editor and you will see again the query. Athena has already executed the query and you should see below the message "Query successful".
        
6. Preview the content of the table: In **Tables and views** locate the table and click on the menu icon to the right of the table name. Choose **Preview Table**. Verify that you see the first 10 rows of the table.
    

## Task 3: Write a Python script to download the current measurement values from MeteoSwiss and upload them to S3

In this task you will write a Python script that downloads the current measurement values from MeteoSwiss to your local machine and then uploads the file to S3. You will later convert this script into an AWS Lambda function.

1. Create a project directory on your local machine. If you already have favorite tools to create a virtual environment and install packages, use them. Otherwise we recommend the standard venv tool. Ensure to use Python 3.11.
    
    - `cd` into the project directory. To create a virtual environment and start a shell in it, run
        
        ```
          python3.11 -m venv .venv
          source .venv/bin/activate
        ```
        
    - For each Python package that you are going to use in your script, install it into your virtual environment as follows:
        
        ```
          pip install <packagename>
        ```
        
2. Write the script.
    
    - For downloading files via HTTP, use the _requests_ package.
    - For uploading files to S3, use the _boto3_ package.
    - Avoid writing the data to disk. Instead use the Python object in memory. You can not only download and upload files, but also _file like objects_. They are supported by both _requests_ and _boto_.
    
    See the documentation provided below.
    
3. Test your script.
    

Deliverables: Copy the script into the report.

Documentation:

- Python SDK for AWS: [Boto3 documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
    - [S3 code examples: Uploading files](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/s3-uploading-files.html)
- Sending HTTP requests in Python: [Requests: HTTP for Humans™](https://requests.readthedocs.io/en/latest/)
    - [Quickstart](https://requests.readthedocs.io/en/latest/user/quickstart/)

## Task 4: Convert your script into an AWS Lambda function for data ingestion

In this task you will convert your script into an AWS Lambda function that we will later trigger every 10 minutes to fetch the current measurement values from MeteoSwiss and store them in S3.

1. To get familiar with AWS Lambda create a first sample function that does nothing. Follow the instructions in the AWS documentation [Getting started with Lambda](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html) to author a function with the Management Console.
    
    - Follow the lab naming convention and name the function similar to `my-function-grf`.
    - Choose the Python 3.11 runtime.
2. Now create a second function to ingest data.
    
    - Follow the lab naming convention and name the function similar to `meteoswiss-ingest-grf`.
    - You need to add an IAM policy to authorise the function to write to S3. When you created the function, the wizard created a role that the function assumes when it executes. The role is named after the function. Add a permission policy to allow writing to your bucket.
    - To avoid overwriting existing files in S3, rename the file and add a timestamp to the filename, similar to `VQHA80-2022-11-09T15:59.csv`. This particular format is the ISO 8601 date/format. You can read about its advantages on the blog post [Why You Should use the ISO Date Format](https://wadegibson.com/why-you-should-use-the-iso-date-format/). To produce this format, you can use the Python _datetime_ module, see [Stack Overflow: ISO time (ISO 8601) in Python](https://stackoverflow.com/questions/2150739/iso-time-iso-8601-in-python).
3. Test your function with the Management Console.
    

Deliverables: Copy the data ingestion function and the IAM policy into the lab report.

## Task 5: Create an event rule that triggers your function every 10 minutes

In this task you will create an event rule that triggers your function every 10 minutes. The Amazon EventBridge service is normally used to act as a publish-subscribe system for relaying events between AWS services and Lambda functions, but it is also able to generate events on a schedule.

1. Open the Amazon EventBridge console. Navigate to **Rules**. Create a new rule:
    
    - **Define rule detail**
        - **Name**: Follow the lab naming conventions and give a name similar to `MeteoswissIngestGrFNicollier`
        - **Event bus**: default
        - **Enable the rule**: enable
        - **Rule type**: Schedule
    - **Define schedule**: Regular rate, every 10 minutes
    - **Select targets**: Select the Lambda function you created
    
    Once the rule is created, it is immediately active. The generation of the first event may take some time, though.
    
2. Open the **Monitoring** tab of the rule to monitor the invocations. Wait for the first invocation.
    
3. In the S3 console, verify that the function was triggered and uploaded a file.
    
4. Let the event rule active for a few days.
    

Deliverables: none

## Task 6: Transform the weather stations file into a CSV file

The weather stations file is in JSON, and it does not have a simple table structure. We cannot join it with the weather measurements table. In this task you will transform the data into a simple data format. For that you will use a specialised tool for JSON data: [jq](http://stedolan.github.io/jq/).

1. Install jq by following the instructions on the [download page](https://stedolan.github.io/jq/download/).
    
2. jq is designed to work like a filter in a Unix pipeline. You give it JSON on standard input, and in produces filtered JSON on standard output. It takes as argument an query expression that tells it how to filter. The simplest expression is `.`, which lets through everything. jq always applies formatting to the output. So for nicely formatted JSON, you write
    
    ```
     cat file.json | jq '.'
    ```
    
3. Even formatted correctly, the structure of the JSON is hard to understand. It's easier in YAML. Let's use the [yq](https://mikefarah.gitbook.io/yq/) tool for that. Install yq by following the instructions in [the yq documentation](https://mikefarah.gitbook.io/yq/v/v3.x/). Convert the file to YAML:
    
    ```
     cat file.json | yq -P . - 
    ```
    
4. Examine the YAML. There are two top-level keys, what are their names? If `foo` is a top-level key, you can extract the value of that key (which may be a whole subtree) with
    
    ```
     cat file.json | jq '.foo'
    ```
    
    One of the keys has an array as value. Which one?
    
5. Select for the key that has an array as value. Your output should now start with `[`. We now want to dig into the elements of the array. jq lets us chain several queries with the pipe symbol `|`. To iterate over the elements of the array, we use the `.[]` query. So add to your query `|.[]`, your command should look similar to
    
    ```
     cat file.json | jq '.foo|.[]'
    ```
    
    You should now see in your output several JSON documents, one for each array element, like so:
    
    ```
     {
         "type": "Feature",
         ...
     }
     {
         "type": "Feature",
         ...
     }
     ...
    ```
    
6. In each of the JSON documents, you will find a key that has the three-letter ID of the station as value. Add a query, separated with the pipe symbol, to extract the value of that key (your query expression should be something like `.foo|.[]|.bar`. You should now have the output
    
    ```
     "RAG"
     "ELM"
     "ANT"
     ...
    ```
    
7. Let's say instead of the ID we want the station name in the output. Return to your previous query. What key contains the station name? This key is nested in another key. To query for nested keys, you can write `.key1.key2.key3`. You should now have the output
    
    ```
     "Bad Ragaz"
     "Elm"
     "Andermatt"
     ...
    ```
    
8. We now want to have both the station ID and the station name in the output. To combine the results of two sub queries, put both queries into the expression and separate them with a comma `,`. So if you have a query `.foo` and a query `.bar`, you can combine them with `.foo,.bar`. Your output should now be
    
    ```
     "RAG"
     "Bad Ragaz"
     "ELM"
     "Elm"
     "ANT"
     "Andermatt"
     ...
    ```
    
9. We want to remove the newlines. The `-j` option does that:
    
    ```
     cat file.json | jq -j <query_expression>
    ```
    
    Your output should now be
    
    ```
     RAGBad RagazELMElmANTAndermattMERMeiringen...
    ```
    
10. Now we need to separate the fields, let's say with a comma. To insert a literal string, put the string in quotes `"` and use again the comma `,` to combine: `.foo,",",.bar`. You should now have the output
    
    ```
    RAG,Bad RagazELM,ElmANT,AndermattMER,Meiringen...
    ```
    
    To have the stations on separate lines again, add a newline `\n` at the end of the query (don't forget to put it in quotes). You now should have the output
    
    ```
    RAG,Bad Ragaz
    ELM,Elm
    ANT,Andermatt
    ...
    ```
    

1. You have noticed that some station names have spaces in them. To protect the spaces, put a quote `"` in front and behind each station name. You need to escape the quote with a backslash like so: `\"`. You should now have the output:
    
    ```
    RAG,"Bad Ragaz"
    ELM,"Elm"
    ANT,"Andermatt"
    ...
    ```
    
2. The JSON contains also the elevation and the coordinates of the station. Add them as well. You now should have the output
    
    ```
    RAG,"Bad Ragaz",497,2756911,1209351
    ELM,"Elm",958,2732266,1198424
    ANT,"Andermatt",1435,2687445,1165044
    ...
    ```
    
3. Add a header to the output file to make it a CSV file:
    
    ```
    id,station_name,altitude,coord_lng,coord_lat
    ```
    
4. Verify that you can open the CSV file in Excel.
    
5. In S3 create a folder `stations_csv` and upload the file into it.
    

Deliverables: Copy the final jq command into the report.

## Task 7: Query the accumulated data

In this task you will make SQL queries on the accumulated measurement data. We assume your `current` folder contains now several files.

1. Inspect your `current` folder and remove any duplicates that remain from testing. It should contain several measurement files, each with a distinct timestamp in the filename.
    
2. Using Amazon Athena, make a query that returns all measurements for the Payerne station (PAY), sorted by ascending datetime.
    

1. For Payerne, make a query that returns the maximum temperature for each hour, sorted by increasing hour.

1. Create a table for the `stations` folder. Find all stations whose altitude is similar to Yverdon, i.e. 400 m >= altitude < 500 m, sorted by altitude.

1. Find the maximum temperature of all stations at an altitude similar to Yverdon, sorted by altitude.

Deliverables: Copy the queries and query results into the report.  

## Task 8: Write an S3 Object Lambda function to transform data

In this task you will write an S3 Object Lambda function that makes the measurement data easier to understand.

- Split the `date` column into six separate columns `year`, `month`, `day`, `hour`, `minute`.
- Replace the semicolon `;` separator with a comma `,`.
- Give the columns more readable names. Use the names from task 2.5.
    
- Follow the explanations in the blog post [Introducing Amazon S3 Object Lambda – Use Your Code to Process Data as It Is Being Retrieved from S3](https://aws.amazon.com/blogs/aws/introducing-amazon-s3-object-lambda-use-your-code-to-process-data-as-it-is-being-retrieved-from-s3/) to create the S3 Object Lambda function and the two access points.
    
    Name the resources using the lab naming convention as follows:
    
    - Access point: `meteoswiss-grf`
    - Object Lambda access point: `meteoswiss-olap-grf`
    - Lambda function: `MeteoSwissPrettifyGrF`
- Test your function.
    

Deliverable: Copy the code of your function into the report and document your tests.

## Task 9: Scenario

As an engineer working with data products, you might face unexpected issues with your application due to sudden modification of the source data. How could MeteoSwiss change the data product, and which modifications would you make to ensure that your code still functions correctly? How would you improve your code robustness for such changes? What can be done to detect schema changes in the source data?