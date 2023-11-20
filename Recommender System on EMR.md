

# Product Recommendation System for Cadabra.com

## Overview:

The recommendation system involves processing server logs from an EC2 instance, streaming the data through Kinesis Data Firehose into S3, and utilizing EMR (Elastic MapReduce)
with Apache Spark's MLLib for generating a recommendation model. we will use ALS alternating least square, it already comes with spark on AWS , it is a collaborative filtering matrix factorization tecnique t
to generate recommendation


![638361024510884560](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/5ffcf986-c201-4071-9ddb-aa4f45364508)

## Steps:

### 1. Server Logs and Kinesis Data Firehose:
- Server logs are populated on an EC2 instance.
- Kinesis Data Firehose streams the logs into an S3 bucket.

### 2. Setting Up EMR Cluster:

#### a. Create EMR Cluster:
- Navigate to EMR in the AWS Console.
- Create a new cluster:
  - Name: Specify a name for the cluster.
  - Release: Choose the EMR release.
  - Applications: Select Spark.
  - Hardware: Choose primary and task nodes (m5.xlarge) for processing.
  - Set cluster size manually.

#### b. Configuration:
- Select VPC and Security Group.
- Configure steps for running jobs automatically (if required).
- Set cluster termination to automatically terminate after 1 hour of idle time.
- Enable protection for EC2 instances from accidental termination.

#### c. Security Configuration:
- Use an existing key pair for SSH access.
- Create a service role and instance profile.
- Provide full S3 access permissions for the instance profile.

#### d. Create Cluster:
- Wait for the cluster to be in the "Waiting" state.

### 3. Connecting to EMR Cluster:

#### a. Adjust Security Group:
- In the AWS Console, go to the security group for the primary node.
- Add inbound rules to allow SSH access from your IP.

#### b. Connect via SSH:
- Copy the primary node's hostname.
- Use Putty to paste the hostname, specify the key pair for authentication, and connect via SSH.

#### c. Verification:
- Once connected, explore the EMR cluster via the command line interface.



## Alternating Least Square (ALS) with Spark ML
Alternating Least Square (ALS) is also a matrix factorization algorithm and it runs itself in a parallel fashion. ALS is implemented 
in Apache Spark ML and built for a larges-scale collaborative filtering problems.
ALS is doing a pretty good job at solving scalability and sparseness of the Ratings data, 
and it’s simple and scales well to very large datasets.Some high-level ideas behind ALS are:
Its objective function is slightly different than Funk SVD: 

ALS uses L2 regularization while Funk uses L1 regularization

Its training routine is different: ALS minimizes 
two loss functions alternatively; It first holds user matrix fixed and runs gradient descent with item matrix;
then it holds item matrix fixed and runs gradient descent with user matrix

Its scalability: ALS runs its gradient descent in 
parallel across multiple partitions of the underlying training data from a cluster of machines




![638361025965464913](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/a01a6dd3-9184-4fa3-a59f-5ac14b2e6b98)




let's make a copy of spark directory in our own directory

     cp /usr/lib/spark/examples/src/main/python/ml/als_example.py ./

    nano als_oexample.py

![638361028870584545](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/f74e9b31-6b42-4848-9f6a-90e48ab555d6)


![638361029022723860](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/3cabc3e3-a0f2-44d4-9313-c85febbc47de)


![638361029142173999](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/73f40a29-1049-4dd0-9598-03a21b8af4db)

![638361029492000092](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/9555e00f-4407-445d-bf96-cf14f9bcb746)


To run the previous script we have to make sure the data it needs is in HDFS so we can access it from every node in the cluster

    hadoop fs -mkdir -p /user/hadoop/data/mllib/als

    hadoop fs -copyFromLocal /usr/lib/spark/data/mllib/als/sample_movielens_ratings.txt /user/hadoop/data/mllib/als/sample_movielens_ratings.txt


    spark-submit als_example.py
    
The results are pretty burried under running INFO


![638361030029681880](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/f5d3957c-b965-4209-998e-1eb365fd4aad)

Let's edit our driver script to see the out put better getting rid of the INFO messages

    nano als_example.py


Right after where the spark session is created let's add

    spark.sparkContext.setLogLevel("ERROR")


Do ctrl + o to write it out, enter, and ctrl +x to exit


Run it again

    spark-submit als_example.py

And we get a cleaner look

![638361031064603754](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/638680f9-e4bb-4956-9b63-00e6a21fee46)



It gave us a list of recommendation for every user but for the 
movie lens dara



### We'll use the model on our own real world data 


In the console, we go to s3 get the name and the path to s3

    lines = spark.read.text("s3://orderlogs-analytics/year=2023/month=11/day=11/hour=05/*").rdd
    parts = lines.map(lambda row: row.value.split(','))
    #Filter out postage, shipping, bank charges, discounts, commissions
    productsOnly = parts.filter(lambda p: p[1][0:5].isdigit())
    #Filter out empty customer ID's
    cleanData = productsOnly.filter(lambda p: p[6].isdigit())
    ratingsRDD = cleanData.map(lambda p: Row(customerId=int(p[6]), \
        itemId=int(p[1][0:5]), rating=1.0))

We'll modify the spark script putting the previous code in it

In the part where we loaded the data we'll replace that code with our code to load our data


Change the column names userID to CustomerID
movieID to columnID

ctrl +o  and ctrl +x

rerun   

    spark-submit als_example.py

error: 
Caused by: com.amazon.ws.emr.hadoop.fs.shaded.com.amazonaws.services.s3.model.AmazonS3Exception: Access Denied (Service: Amazon S3; Status Code: 403; Error Code: AccessDenied; Request ID: H8WT5Q2R9VF6KJX5; S3 Extended Request ID: s1aaEF+04iKGNlYeGprdUumFbDSn7QNzgMt+zDCBLE6HW3CLXZmb4eAKKZDPgITgafpfgp4UTeI=; Proxy: null), S3 Extended Request ID: s1aaEF+04iKGNlYeGprdUumFbDSn7QNzgMt+zDCBLE6HW3CLXZmb4eAKKZDPgITgafpfgp4UTeI=


Changed IAM policy to grant list objectv2 permission and 
yeiiii

We got recommendations!

Remember to terminate the cluster, we did program it to terminate after 1 hour, let's not rely on that
it cost real money











