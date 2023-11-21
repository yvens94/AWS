# Redshift + quicksight for data wharehousing and Visualizations requirements.

1. **Create Redshift Cluster:**
   - In Redshift console, create a cluster with desired settings.

2. **IAM Role Setup:**
   - In IAM console, create "redshift-spectrum" role with S3 read-only and Glue permissions.
   - Copy the role's ARN.

3. **Associate IAM Role:**
   - In Redshift console, assign the IAM role to the cluster.

4. **Create External Schema:**
   - Use Redshift editor to run:
     ```sql
     CREATE external schema orderlog_schema FROM data catalog
     database 'orderlogs'
     iam_role 'ARN_COPIED_FROM_IAM_ROLE'
     region 'us-east-1';
     ```

5. **Explore Schema:**
   - Refresh Redshift editor, explore the external schema.

6. **Run Queries:**
   - Query data in S3 using Redshift Spectrum.

7. **Verification:**
   - Run a query to ensure consistency with Athena results.
  
![638361842467289096](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/056e22c7-06b7-4293-8a18-7ad23d094c08)

![638361842850486826](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/97cc2634-577e-4e7e-bdc0-98e1da1dc475)


We actually set up a new Redshift cluster from scratch,
a very simple one
using a very inexpensive Node instance type
and only one of them.
And rather than just importing data into the cluster itself,
we actually connected it to our data that's sitting
in our S3 data lake using Glue data catalog and S3
and a specific IAM role that allows everything
to talk to each other.
So we're using Redshift Spectrum to actually
look at that information that's being stored
in S3 in our data lake
and interpret that as yet another database table.


# Quicksight

1. **Sign Up for Amazon QuickSight:**
   - Choose the standard version.
   - Enter account name, email address, and select a region.
   - Enable auto-discovery.
   - Choose Amazon Athena as the data source and select S3.

2. **Create a New Analysis:**
   - Give the analysis a name.
   - Enter the instance ID, database name, username, and password for the Redshift cluster.
   - Create a source.

3. **Configure VPC and Security Groups:**
   - Set up VPC and security groups.
   - Open SSH for anywhere in the Redshift security group.
   - Save rules.

4. **Create Data Source:**
   - Select schema and tables.
   - Import into SPICE.

5. **Visualize Data:**
   - Explore and analyze data using QuickSight.

6. **Additional Configuration:**
   - Ensure auto-renewal is turned off in QuickSight settings (61 days). to prevent charges

Now, we're ready to analyze and visualize data in Amazon QuickSight using Amazon Athena and Redshift as data sources. Adjustments may be needed based on specific preferences and requirements.

![638361844381204552](https://github.com/yvens94/AWSEcommerceAnalyticsInfrastructure/assets/68969793/06afc244-c382-4ca2-9468-9b0719583f09)

