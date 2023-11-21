# ORDER HISTORY SERVICE


Order History App Integration with Kinesis Data Stream on AWS
To establish real-time order history access on the Cadabra Store, we'll set up a Kinesis Data Stream to capture and process server logs. The ultimate goal is to connect an EC2 instance to the Kinesis Data Stream using the Kinesis Agent.

Steps:
### 1. Kinesis Data Stream Setup:
Open the AWS Management Console.
Navigate to Kinesis.
Create a new data stream named "CadabraOrders."
Choose the "On Demand" option for scalability.

### 2. EC2 Instance Integration:
Access the EC2 console.
Ensure that the EC2 instance has the necessary IAM roles with permissions to interact with Kinesis.
Install and configure the Kinesis Agent on the EC2 instance.
Configure the Kinesis Agent to publish server logs to the "CadabraOrders" Data Stream.
###  3. Console Configuration:
Utilize the AWS Management Console for Kinesis.
Validate that the "CadabraOrders" Data Stream is active and ready to receive data.
Monitor stream activity through the console to ensure proper functioning.

### Considerations:
For bursty testing and development workloads, "On Demand" capacity for the Data Stream is recommended.
Provisioned mode may be suitable for fixed load capacities or steady traffic streams.
By connecting the EC2 instance to the Kinesis Data Stream, server logs from the Cadabra Store will be seamlessly ingested, paving the way for real-time availability. This integration lays the foundation for future connections to AWS Lambda and DynamoDB for more advanced analytics and order history functionalities.

# Setting the kinesis agent 

connect in to the EC2 instance from the terminal, by instance connect or using putty


1- Navigate to the AWS-Kinesis Folder:

![image](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/af9bb961-c47e-4c6f-a48c-bce691557175)

2- Open the agent.json file using Nano:

![image](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/7d4eccaf-2f5c-4231-a3b7-5546cc25c2d9)

3- Locate the "flows" section in the agent.json file.
Add the following configuration:
enter the name of the columns in Customfieldname
![image](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/86e0f146-78be-490a-bf8c-341f4e23f62c)

This will send the stream of data to the kinesis stream, partitioned randomly to evenly distribute the processing of that data and for dat processing option it will use the csv to json converter
and we provided the fields name which it will use to name the data in the json format.

![image](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/6e98a092-f789-45a4-82ea-4bf45fc151df)

If we go to the console, under data stream and go to monitoring, we see that we've succesfully put some record in our stream

![image](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/48a7e01e-bf9a-480a-9354-7c7c48b1e1b6)



# Dynamo DB set up

Let's introduce dynamoDB into the mix we are not using lambda yet


we will use a custom consumer script   that will just sit on our EC2 Instance listening for new data on our data stream that we created earlier and funneling that into an Amazon DynamoDB table.
let's build our Amazon DynamoDB table and create a consumer script to fill the gap between the data stream and DynamoDB.

1- console 
2- dynamoDB-createTable
3- partitionKey CustomerID as a number

The reason being that, our order application is meant for an individual customer to look up his or her order history. So what makes sense to partition our data by
that customer ID?
So DynamoDB can very quickly retrieve the information for a given customer as a whole. We'll also add a sort key because customer ID
isn't unique enough for us, there could be multiple record associated with a given customer ID representing individual items that they've ordered.
So the partition key by itself is not sufficient to provide a unique key. So instead we'll add an order ID as well that represents an actual line item order ID
and we're gonna have to fabricate that as you'll see shortly. That will remain as a string. We can keep all the provisioning set to the default settings.


while the table is being created let's setup our customer script on ec2 we will replace it by a lambda function later
install BOTO3 library aws API on ec2 for python

![image](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/6a2bb5c6-bb26-4ee8-a8d5-552d6ff34cd2)

### Configure AWS Credentials:

![image](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/d1aba203-5398-49b9-95e1-f7eac80a67d8)

In the credentials file, add your AWS access key and secret key:

![image](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/e4bfc866-af0d-45ce-a895-676a1b9cb673)


Back to home directory to download our consumer script

wget http://media.sundog-soft.com/awsBigData/Consumer.py

![638360981401647414](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/a672f985-c5ce-4fd9-97fb-87e9aa483a09)


![638360981670684145](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/2b072fb6-2ba9-4543-a05f-267fb935c5ce)



chmod a+x Consumer.py to make it executable, 
a run it: ./Consumer.py

no action


open another terminal to the ec2 generate 10 logs using
sudo ./LogGenerator.py 10

see some action in the previous terminal after 1-2 min

can also see the data in the dynamoDB table 


![638360982038746526](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/68e2f1f0-8d8f-49bf-8aab-53c11841d970)



# LAMBDA FUNCTION

serverlogs-ec2-aws lambda** dynamoDB

Transitioning to a serverless architecture, AWS Lambda offers scalability advantages over running a script on an EC2 instance. Here's how you can set up the required IAM role for your Lambda function:

### 1. Create an IAM Role:
Use the AWS Management Console or AWS CLI to create an IAM role.
Choose the use case "Lambda."
### 2. Attach Policies:
Kinesis Read-Only Access:
Attach the "Amazon Kinesis ReadOnlyAccess" policy to allow the Lambda function to read from the Kinesis stream.
DynamoDB Full Access:
Attach the "Amazon DynamoDB FullAccess" policy to grant the Lambda function full access to DynamoDB.
### 3. Provide Details:
Give the role a descriptive name, such as "LambdaExecutionRole."
Include a meaningful description for clarity.
### 4. Create Role:
Complete the role creation process to generate the necessary IAM role for your Lambda function.

By creating this IAM role, your Lambda function will have the required permissions to consume data from the Kinesis stream and write to DynamoDB. This serverless approach enhances scalability and eliminates the need for managing EC2 instances.

### 1. Create Lambda Function:
Open the AWS Lambda console.
Choose "Create function" and select "Author from scratch."
Provide a name for your function, e.g., "ProcessOrders."
Choose the runtime (Python 3.11 in this case).
For the execution role, select "Use an existing role" and choose the IAM role you previously created.
Click "Create function."
### 2. Add Kinesis Trigger:
In the Lambda function console, scroll down to the "Add triggers" section.
Choose "Kinesis" as the trigger type.
Select the Kinesis stream you want to monitor.
Click "Add."
### 3. Configure Kinesis Trigger:
For batch size, use the default or adjust based on your requirements.
Since you're not adding a consumer, the batch size should be fine.


We need to write some python code to process that data

click on function, scroll to the function code

The python code of the lambda function:



    import base64
    import json
    import boto3
    import decimal
    import uuid

    def lambda_handler(event, context):
        dynamo_db = boto3.resource('dynamodb')
        table = dynamo_db.Table('CadabraOrders')

      for record in event['Records']:
          try:
              decoded_data = base64.b64decode(record['kinesis']['data'])
              item = json.loads(decoded_data)
  
              invoice = item['InvoiceNo']
              customer = int(item['Customer'])
              order_date = item['InvoiceDate']
              quantity = item['Quantity']
              description = item['Description']
              unit_price = item['UnitPrice']
              country = item['Country'].rstrip()
              stock_code = item['StockCode']
          
              # Construct a unique sort key for this line item
              order_id = f"{invoice}-{stock_code}-{uuid.uuid4().hex}"
  
              table.put_item(Item={
                  'CustomerID': decimal.Decimal(customer),
                  'OrderID': order_id,
                  'OrderDate': order_date,
                  'Quantity': decimal.Decimal(quantity),
                  'UnitPrice': decimal.Decimal(unit_price),
                  'Description': description,
                  'Country': country
              })
  
              print("Item successfully processed.")
          except Exception as e:
              print(f"Error processing record: {str(e)}")


click on deploy


We don't have to add a destination, the script is taking care of that for us


Back to EC2 connection

add some logs, to verify everything is working 

![image](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/0842a736-ad1c-48a2-8346-e9df5d779af8)



"We successfully deployed our end-to-end Order History App, leveraging AWS cloud services and analytics technologies. The process begins with raw data being sent to an EC2 instance, where a Kinesis agent efficiently captures and streams log data into a Kinesis Data Stream. This stream acts as a trigger for a Lambda function, which, in turn, inserts the data seamlessly into DynamoDB. The app can now query this database for comprehensive order history. This architecture ensures scalability, real-time data processing, and seamless integration within the AWS ecosystem."














