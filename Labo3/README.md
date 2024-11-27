# Lab 3: Object Storage (Amazon S3)
   
   _Duration of this lab: 2 periods_
   
   ## Pedagogical objectives:
   
   - Become familiar with _buckets_, objects and object metadata
   - Store and retrieve data via the command line
   - Become familiar with the semantic differences between object storage and file systems
   
   #### Tasks
   
   In this lab you will perform a number of tasks and document your progress in a lab report (PDF file). Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.
   
   #### Prerequisites
   
   This lab requires
   
   - an AWS account;
   - the installation of the AWS Command-Line Interface (instructions below).
   
   #### Allocating and freeing cloud resources
   
   HEIG-VD is paying for your use of Amazon cloud. Please use the service responsibly:
   
   - Compute resources (virtual machines)
       - Launch VMs only when you are ready to work with them.
       - Stop VMs when not in use, for example when you finish working for the day.
       - Once you are done with your work delete all VMs, except when indicated otherwise.
   - Storage resources
       - Once you are done with your work determine which data you want to keep and which data you no longer need. Delete the latter.
   
   #### Lab convention for naming cloud resources
   
   You will work together with the other students in the same AWS account, and you will see the cloud resources created by other students. To avoid chaos it is therefore important that everybody follows the same convention for naming the resources. This includes public/private key pairs, security groups, EC2 Instances, load balancers, etc.
   
   We will use the following **lab naming convention:** the name of every cloud resource starts with the course name, followed by the lab number, the group, and optionally your last name. You can append more information if you want. For example when student Nicollier in course IST working on lab 03 in Group F creates an EC2 Instance “master”, he names it
   
   ```
   IST_L03_GrF_Nicollier_master
   ```
   
   ## Task 1: Use the web console to create an S3 bucket and upload and download objects (files)
   
   In this task you will use the S3 web console to create a bucket and upload and download objects (files). Objects in Amazon S3 can be up to 5 TB.
   
   1. Navigate to the S3 console.
       
   2. Create a new bucket. Choose a name for the bucket following the lab naming convention.
       
       **Note**: Bucket names must be unique across all buckets in Amazon S3. If you get a conflict with another bucket, you can try to make it unique by for example adding a digit.
       
       **Note**: Write down the bucket name as you will need it in future steps.
       
   3. Create on your local machine a text file named `lab.csv` with the following content:
       
       ```
        CustomerID,First Name,Last Name,Join Date,Street Address,City,State,Phone
        001,Alejandro,Rosalez,12/12/2013,123 Main St.,Baltimore,MD,765-234-2349
        002,Jane,Doe,10/5/2014,456 State St.,Seattle,WA,415-889-4932
        003,John,Stiles,9/20/20016,1980 8th St.,Brooklyn,NY,917-123-9308
        004,Li,Juan,6/29/2011,1323 22nd Ave.,Albany,NY,917-332-3432
       ```
       
       Upload the file to the bucket.
       
   4. To verify that it was uploaded successfully, we will do an SQL query on the file, and at the same time learn that S3 has a buil-in SQL engine that works directly on CSV files.
       
       - Select the object. From the **Object actions** menu, choose **Query with S3 Select**.
           
       - Scroll down the page and choose **Run SQL query**. You should see the first few records from the file.
           
       - Choose **Add SQL from templates**. Choose **SELECT COUNT * FROM s3object s**. Choose **Copy SQL**. Replace the previous query by deleting it and then paste the query you copied. Choose **Run SQL query**. In the **Result** pane, you should get the total number of records, which is _5_.
           
   5. Use the web console to download the file to your local machine.
       
   
   ## Task 2: Use the AWS Command-Line Interface to manage buckets and objects
   
   In this task you will use the AWS Command-Line Interface to create a bucket, upload and download objects.
   
   1. Install the AWS CLI by following the instructions in [Installing or updating the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html). **Note for macOS users:** The AWS CLI is available in homebrew as `awscli`.
       
   2. To get your credentials go to the management console, click on your profile name (upper right corner), then click on **My Security Credentials**. Go to **Access Keys** and select **Create New Access Key**. Click on **Show Access Key** and copy/paste the access key ID and secret access key to your local machine.
       
   3. Set up your security credentials, default Region and default output format by following the instructions in [Authenticate with IAM user credentials - Configure the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-user.html#cli-authentication-user-configure.title).
       
       - Set the default Region to `us-east-1`.
       - Set the default output format to JSON.
   4. Verify that the tool is configured correctly.
       
       List all available regions:
       
       ```
        aws ec2 describe-regions
       ```
       
       Display account attributes:
       
       ```
        aws ec2 describe-account-attributes
       ```
       
       Display available EC2 Instance types:
       
       ```
        aws ec2 describe-instance-type-offerings
       ```
       
       List all S3 buckets in the account:
       
       ```
        aws s3 ls
       ```
       
   5. Manipulate buckets and objects. Use the documentation [Using high-level (S3) commands with the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html) as a reference.
       
       - Create a new bucket.
       - Upload an object.
       - List the objects in the bucket.
       - Copy the object.
       - Delete the copied object.
   6. S3 folders behave differently from file system folders. In fact a folder in S3 is a 0-byte object whose name ends with a slash (`/`). Read the introduction on the page [Organizing objects in the Amazon S3 console using folders](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-folders.html).
       
       - Create a folder (using the Management Console).
       - Make a copy of the object and move the copy into the folder.
       - What happens if you move an object to a folder that does not exist?
   
   ## Task 3: Create a static web site
   
   In this task you will make the objects in a bucket publicly accessible by using the static web site feature of S3.
   
   1. Follow the instructions of the [Tutorial: Configuring a static website on Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/HostingWebsiteOnS3Setup.html) to create a new bucket for a static web site.
       
       - You can use web site content (index.html, CSS stylesheet, error document) of your liking.
   2. On which URL is your new website reachable?
       
   
   ## Task 4: Explore a public bucket with a large dataset
   
   In this task you will explore a bucket that
   
   - is owned by another account
   - is public
   - is truly “big data”.
   
   The dataset is the Common Crawl corpus: a monthly-update archive of the HTML pages of the entire World-Wide Web, which comprises 3.15 billion web pages and has an uncompressed size of 380 TiB.
   
   1. The data location of the Common Crawl datasets is described on the page [Get Started](https://commoncrawl.org/the-data/get-started/). When was the latest crawl? What is the bucket name? Under which prefix is the latest crawl stored?
       
   2. Log into the AWS S3 Management Console. Replace the browser URL with
       
       ```
        https://s3.console.aws.amazon.com/s3/buckets/<bucketname>/
       ```
       
       Where you replace `<bucketname>` with the name of the bucket. You should see a bucket with objects and folders.
       
   3. Navigate to the root folder of the latest crawl. Click on the object `index.html`. Click the `Open` button to load it into your browser. What is the URL of this object?
       
   4. Explore a bit the objects and folders.
       
       - What are WARC, WAT and WET files (look at the Get Started guide)?
       - What is the typical size of a WARC file (ballpark)?
       - Why is it not sufficient to just store the WARC, WAT and WET files in the bucket? What other type of file is needed?
       - What storage classes have the Common Crawl developers chosen to store the data?
   
   ## Task 5: Scenario
   
   You are a data engineer at Coop responsible for a data product. The product is the global Coop sales data, which is shared with several Coop departments. Your task is to add each week the new sales data of the week to an S3 bucket.
   
   How would you do this task? Describe your thought process.