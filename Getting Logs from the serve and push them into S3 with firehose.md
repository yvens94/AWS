# Log server to S3 by Kinesis data Firehose

To push data from a server to Amazon S3 using Kinesis Data Firehose, you can follow these general steps:

1- Create a Kinesis Data Firehose Delivery Stream

2- Set Up an Amazon S3 Bucket

3- Obtain Firehose Endpoint 

4- Send Data to Kinesis Data Firehose

5- Monitor Data Flow



We start by creating a firehose stream
we to set up S3 bucket prefix -  so a Glue crawler can identify what the directory names represent:

    year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/hour=!{timestamp:HH}/

And also an error outpout format: 

S3-error-outpout prefix:

    fherroroutpoutbase/!{firehose:random-string}/!{firehose:error-output-type}/!{timestamp:yyy/MM/dd}/

buffer interval 60 second, the minimun for firehose

### EC2 to feed data into the firehose stream
Launching AMI Linux EC2 Instance:

Choose AMI Linux.

1-Select t2-Micro instance type to stay within the free tier. 

2-Setting Up Security Group:

3-Create a new Security Group (SG).

4-Allow SSH traffic restricted to your IP only.



### Connecting Using SSH:

Connect to the EC2 instance using SSH.

Download Putty for Windows.

Convert the .pem key to .ppk format using PuttyGen.

Paste the DNS hostname in Putty.

Save the session for future use.


### Installing the kinesis agent 

it allows us to send data into kinesis firehose

    $ sudo yum install –y aws-kinesis-agent


### Getting the log data

    wget http://media.sundog-soft.com/AWSBigData/LogGenerator.zip

    unzip LogGenerator.zip

    reponse:
    Archive:  LogGenerator.zip
      inflating: LogGenerator.py
       inflating: OnlineRetail.csv

Making the python file executable and change permission on it to all so we can run it

    chmod a+x LogGenerator.py
And  to examine the log generator document we use the command

      less LogGenerator.py


![638361008451850861](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/5d15f45e-cf36-4dae-b6d2-48077565b984)

![638361010003564028](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/1a80103c-7235-4b50-82f5-d1d261eff95b)


To see the file with the logs

    less OnlineRetail.csv 
    Q to get out


    sudo mkdir /var/log/cadabra

    cd /etc/aws-kinesis/

Customize the kinesis agent configuration

    sudo nano agent.json

to open it

### Enhanced Security Setup for EC2 Instance:

IAM Role Configuration:

Navigate to the EC2 instance.

Under Actions, go to Security and modify IAM role.

Create a new IAM role with EC2 use case, granting administrator access for simplicity.

Assign a name, e.g., "EC2cadabra," and create the role.


### IAM Role Association:

Go back to the EC2 dashboard.

Select the EC2 instance and associate the newly created IAM role.


### Adjust Kinesis Agent Configuration:

Open the Kinesis agent configuration file using Nano.

Delete the Kinesis stream section from the "flows" section.

Update the file to include only the Firehose delivery stream configuration.


### Save Changes:

Save the changes in Nano by pressing Ctrl + O, then Enter.

Exit Nano with Ctrl + X.

At the end the config file should look like:

![image](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/756628ac-e4f7-4abd-916f-4cd92359898c)



#### Start the AWS Kinesis agent
    sudo service aws-kinesis-agent start
    
#### If encountering an error, check the agent configuration file for typos

#### Fix any identified issues and try starting the agent again
    sudo service aws-kinesis-agent start

#### Set the AWS Kinesis agent to start automatically on boot
    sudo chkconfig aws-kinesis-agent on

#### Navigate to the home directory
    cd ~

# Generate 500,000 logs
    sudo ./LogGenerator.py 500000

#### Check if the logs are present in the specified directory
      cd /var/log/cadabra/
      ls

![638361018141174868](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/da02fe4a-3a6f-40f9-b317-dd1560375ac2)
      

#### View real-time updates on Kinesis agent activity
    tail -f /var/log/aws-kinesis-agent/aws-kinesis-agent.log


In the console we can also verify that the data is actually in the S3 bucket

Success!
