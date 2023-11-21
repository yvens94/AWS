# NEAR REAL TIME LOG ANALYSIS IN AMAZON OPENSEARCH
We mentioned near real-time because we we'll be using firehose for the streaming process, which is good for operational purposes.

The objective of a log analysis system is for when a crashed happens, we can use a dashboard to narrow down when it happened
and probably why it happened.

Let's start:

First we download server error data from the website.
real web server logs


We connect to the EC2 host, and run the command
    
    wget http://media.sundog-soft.com/AWSBigData/httpd.zip

We unzip it with,

    unzip httpd.zip

![638361736773438577](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/9f5b07d5-e334-43d7-880f-61d8ab63da00)


Next let move the acquired data to a chosen directory where the kinesis agent can find it

    sudo mv httpd /var/log/httpd

![638361737505957344](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/8aac5255-f983-4b2a-803c-ac43b9b01143)

Make sure it is where we want it to be

    cd /var/log/httpd

    ls


![638361738169520512](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/4b674e56-c0b7-44c3-a277-4a1ed78465bc)



Now we move to the console to the opensearch service

### Create a New Domain:

Opt for the standard creation to manage costs effectively.
Deploy without a standby to further economize resources.
Choose a 2-Availability Zone (AZ) configuration for cost efficiency (consider using 3 AZs in a real-world scenario).
Select Latest Version of Amazon OpenSearch:

Opt for the latest version of Amazon OpenSearch during the domain creation.

### Configure Node Settings:

Utilize a single node to minimize expenses.
Set up 1-hour snapshots to ensure regular data backups.
Omit the dedicated master node to streamline the deployment.

### Network Configuration:

Set the Virtual Private Cloud (VPC) to public access.
Enable fine-grained access control for enhanced security.
Configure domain-level access policies to control permissions.
Change IAM to IPv4 based on the current IP address obtained from whatismyip.com.
Copy and paste the relevant IP address to allow access.


### Enable Autotune:

Turn on Autotune to optimize performance automatically.
Create Amazon OpenSearch Domain:

Complete the configuration and create the Amazon OpenSearch domain.

### Firehose Stream Configuration Steps:
Set Up Firehose Stream:

Spin up a new delivery stream for data processing.


### Configure Source and Destination:

Choose the source as 'Direct Put' and the destination as 'OpenSearch.'
Assign a name to the delivery stream, e.g., 'Weblogs.'


### Data Format Transformation:

Acknowledge that the raw data is in Apache format.
Since OpenSearch requires JSON format, enable data transformation.


### AWS Lambda Configuration:

Create an AWS Lambda function for data transformation.
Create a Lambda function named 'apachelog to json' for Apache log to JSON conversion.
Refer to the provided source code and adjust it according to your project needs.
Note: The Amazon link may be incorrect; search for 'apachelog to json' one level down in the directory.


### Kinesis Firehose Configuration:

Link the Lambda function to the Kinesis Firehose delivery stream.
Ensure that the Kinesis Firehose Apache log to JSON transformation is appropriately selected.


## Back Up Bucket on S3 and Configuring Kinesis Agent on EC2:

### S3 Bucket Configuration:

Create two 1 buckets: and use index es/ and eserror/ for storing the main index and error logs respectively.



### Install and Set Up Kinesis Agent on EC2:

Install the Kinesis Agent on the EC2 instance.
Connect to the EC2 instance.


### Connect to the EC2 instance
    ssh -i <your-key-pair.pem> ec2-user@<your-ec2-instance-ip> 

### Configure Kinesis Agent:

Edit the Kinesis Agent configuration file.


    sudo nano /etc/aws-kinesis/agent.json

Verify that your region is correctly set in the endpoint.

### Add Configuration to Agent File:
In the "flows" section, replace existing information with the following:


    {
      "filePattern": "/var/log/httpd/ssl_acces*",   // Path where logs are stored
      "deliveryStream": "webLogs",   // Name of the Firehose delivery stream
      "initialPosition": "START_OF_FILE"
    }
Save changes (Ctrl O + Enter) and exit (Ctrl X).

![638361797416816317](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/392e64f9-da87-4d2c-9fc0-452407f9c6e7)


### Restart Kinesis Agent:
Restart the Kinesis Agent to apply the new configuration.

    sudo service aws-kinesis-agent restart

    
Now, we have a configured Kinesis Agent on your EC2 instance, and the Firehose delivery stream is ready to receive data from the specified S3 buckets. The data will be ingested into the Amazon OpenSearch cluster for further analysis.



### Verifying Data Flow and Visualization on OpenSearch:

1. **Data Verification:**
   - Navigate to the OpenSearch console.
   - Select the created domain.
   - Go to "Indices" to check the data flow.

2. **Access OpenSearch Dashboards:**
   - Click on the OpenSearch Dashboards URL link.
   - Since the cluster is publicly accessible but limited to your IP, proceed without security concerns.

3. **Explore and Manage Data:**
   - Click on "Explore my own."
   - Go to the "Manage" link.

4. **Set Up Index Pattern:**
   - Click on "Create an index pattern."
   - Enter the index pattern name as `webLogs*` (the assigned index name).
   - Click "Next."

5. **Configure Time Field:**
   - Set the time field as "timestamp."
   - Navigate to the "Discover" menu.
   - Initially, it looks at the last 15 minutes, so adjust the time range.

6. **Adjust Time Range:**
   - Click on "Show dates."
   - Change to "Absolute."
   - Set the start date to January 27th (your specific start date).
   - Set the end date to February 2nd, 2019.
   - Click "Update."

7. **Explore Visualizations:**
   - Explore visualizations in the OpenSearch Dashboards.
   - Review the JSON transformation.

8. **Investigate Operational Issues:**
   - In case of an operational issue investigation:
      - Navigate to the "Menu."
      - Choose "Visualize."
      - Create a new visualization for tracking 500 errors over time.

9. **Create Bar Chart:**
   - Select "Bar Chart."
   - Add a filter to focus on 500 errors.
   - Set the response field value to 500.

10. **Configure X-Axis:**
    - Adjust buckets by adding a bucket on the x-axis.
    - Make it a date histogram with an hourly interval.
    - Click "Update."

11. **Review Visualization:**
    - Observe the visualization to identify patterns and anomalies.
    - Verify if there was a spike in 500 errors during the specified time range.
      
![638361799266690892](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/f9be2762-104c-434f-ae04-406a0c32b53f)

and if we disable the filter we can see that we have periodic pikes of error

![638361799922925319](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/69e0b9ad-1783-4ee4-8d6c-41e071363936)


We've actually gone through the entire pipeline
of setting up some server log data on an EC2 host
using the Kinesis agent to put that
into a Kinesis Firehose delivery stream.
That delivery stream is using a Lambda application
to transform the data using Lambda
from Apache log format to JSON format.
And then finally feeding that JSON data into OpenSearch.
And from OpenSearch you can see that we can
interactively query and visualize that data as need be.

