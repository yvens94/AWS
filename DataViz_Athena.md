# Data visualization pipeline from s3 with Glue and Amazon Athena 

## Setting up AWS Glue for Data Lake Schema Inference:

### Navigate to AWS Console:

 Access the AWS Console.

### Open AWS Glue:

Go to AWS Glue in the console.
Access Data Catalog and Crawlers:

Click on "Data Catalog" and then "Crawlers."

### Create a Crawler:

Select "Create Crawler."
Provide a name, optional description, and tag.

### Specify Data Source:

Answer "No" to the question, "Is our data already mapped to Glue tables?"
Add a data source, choose S3, and provide the S3 path.
Browse S3 and select the desired bucket, ensuring to add a final slash.
Crawl subfolders and exclude unnecessary patterns (e.g., es/** to exclude OpenSearch exercise data).

### Add S3 Data Source:

Click "Add S3 Data Source" and proceed to the next step.

### Create IAM Role:

Create an IAM role,  "orderlogs."
Proceed to the next step.
Assign a Database:

Create a new database, e.g., "orderlogs."
Click "Create Database" and go back to the Glue console.

### Select Database and Frequency:

Choose the created database.
Set the frequency to "On Demand" (not on a schedule).

### Review and Create:

Review all configurations.
Click "Create Crawler."

### Run Crawler:

Run the crawler and wait for it to finish.

### Check Table Creation:

Under "Data Catalog," select the database created.
Examine the schema to ensure proper partitioning.

### Modify Schema for Human-Readability:

we'll modify the schema for human readability.
Edit the schema by assigning human-readable names to columns (e.g., InvoiceNo, StockCode, etc.).
Save as a new table version.

### Query with Athena:

Access Athena in the AWS Console.
Open the Query Editor and select the table we just created.
Query the data using SQL-like commands.

![638361810100681545](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/111f15cd-7590-4948-aa9c-8b76ef78331b)

![638361810306309410](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/9b31b4cf-0ebc-4b68-bb9c-a39ce24df2e5)

![638361810506796689](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/5db4c21f-89c4-43b1-b1b6-ba48193a03d8)



Highest selling Item in the UK

![638361810670600081](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/9112b194-4869-4d11-86a1-65fc41063ed6)

![638361810918633393](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/4818fda0-65e3-49a9-8a40-e1ede8a82e85)



