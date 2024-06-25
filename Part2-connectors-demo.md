# Part 2: Demo with Confluent Connectors

In this setup, we'll utilize Confluent's fully managed connectors, specifically Debezium V2 Source and S3 Sink, leveraging AWS PrivateLink for enhanced security.

## Understanding the Setup

Here's the schematic overview:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c929dee23ac648d93b1887e816ee599eae4d041d/images/demo-schema.png)

_In this diagram, S3 is shown separately because it's a fully managed service by AWS, residing outside any custom Virtual Private Cloud (VPC). For security reasons, we'll restrict access to S3 only from our VPC using VPC endpoints if the data shouldn't be accessible from the public internet. You can learn more about this setup here._ _More info at https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-s3.html._

______

# Some beloved theory in simple words:

When we talk about connecting Confluent Cloud to Amazon S3, we have two main ways to do it securely: VPC endpoints and S3 egress endpoints.

- **VPC Endpoint for S3:** Imagine you have a private network (VPC) on AWS where your services like EC2 or databases reside. Normally, if they need to access S3 (where you store data), they would have to go through the public internet, which isn't ideal for security. A VPC endpoint acts like a private tunnel that lets these services access S3 directly, without ever leaving your secure VPC. It's like having a private road just for your AWS services to reach S3, keeping everything safe and private.

- **S3 Egress Endpoint:** Now, let's say you're using Confluent Cloud, which might not be in the same AWS VPC as your S3 buckets. When Confluent Cloud needs to write data to S3 securely, it uses an S3 egress endpoint. This endpoint creates a secure connection from Confluent Cloud's environment directly to your S3 bucket, even if they're in different places. It's like setting up a secure delivery service specifically for Confluent Cloud to safely send data to your S3 storage, without any detours through the public internet.

In essence, both VPC endpoints and S3 egress endpoints ensure that your data travels safely and privately between your services and S3, maintaining high security standards without exposing your information to the risks of the public internet.
_______
> _Preliminary Notes :_
> _Ensure you have AWS credentials (Access Key and Secret Key) from IAM. If you don't have them, you can create them in the AWS Management Console._ [Here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) is explained in the AWS docs.
> _You'll also need an API Cluster Key for Confluent Cloud, which you can create through the UI of Confluent Cloud within your cluster settings or using the CLI. These credentials are essential for managing resources in Confluent._
___________

# **Create the AWS infrastructure**
## **Create a MySQL database and fill it with data**

First, we'll set up a MySQL database within AWS RDS (Relational Database Service):

- Choose MySQL as the database engine during setup.
- Opt for Self Managed credential management to set your own password for database access.
- Save this password securely as we'll need it later.


![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/7db0f8af95039a71498d167be52113c5ecc2cf03/images/select_rds_database.png)

Remember, to access the EC2 Bastion we created before, this is the command, or if possible just from the Connect AWS button:
```
ssh -i "<my-pem>.pem" ec2-user@ec2-xx-xxx-xxx-xxx.eu-west-1.compute.amazonaws.com
```
_Of course, you will need install in your instance all the MySQL and Python stuff. Remember that Amazon Linux EC2 = CentOs._
I will give some external documentations for this:
- [How to install MySQL on CentOS](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-centos-7)
- [How to install Python on CentOs](https://www.liquidweb.com/kb/how-to-install-python-3-on-centos-7/)
- [How to install MySQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04)
- [How to install Python on Ubuntu](https://phoenixnap.com/kb/how-to-install-python-3-ubuntu)
  
Then we can access the inside of the database _(and it will ask for your password here after pressing the command)_ :
```
mysql -h <my-rds-endpoint> -P 3306 -u your_mysql_username -p 
```

Note: you can also see from the UI the content of your database with some magic, for example by using PhpMyAdmin in Ubuntu or Adminer in CentOs.
- [How to Install phpMyAdmin on Ubuntu](https://www.hostinger.com/tutorials/how-to-install-and-setup-phpmyadmin-on-ubuntu)
- [Install phpMyAdmin on CentOS 7](https://www.ionos.com/digitalguide/server/know-how/install-phpmyadmin-on-centos-7/)


## Fill with some mock data with a Python script 

I filled the MySQL data with random data created by [a Python code](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/main/mysql-python-script.py). I have attached here in the repo. Change the values of the database to yours. You can also use nano or others, I like vi. :see_no_evil:	. Please feel free to change this code, it is just an example!
If you want to run it, you already know: 
```
sudo vi mysql-python-script.py #copy my code in here,
python3 mysql-python-script.py
```
____________

## **Create an S3 Bucket**
Select your desired region and name for the S3 bucket. For enhanced security, configure access to be blocked from the outside, leveraging AWS PrivateLink:

![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/5a258b2727bc5d801bd35ad2ad7fade560b9117b/images/bucket_block.png)


At this stage, you've set up your MySQL database, an S3 bucket, and your dedicated cluster in AWS. The next step is to establish connections between them.

![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/512e2f8a107246250d5092f2c5ddcb07a0ef1c5a/images/only_mysql_and_s3.png)
________
 
# Inside Confluent Cloud...
## Create MySQL CDC Debezium V2 Connector

Now, let's set up our connectors in Confluent Cloud:
- Navigate to your cluster and go to Connectors > Add Connector > MySQL Debezium CDC Source V2.
- Configure the connector settings as shown:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/5318c2a0b7168ad4ebfaf7e34bafe2d8268e1f65/images/debezium_creatiom.png)
Choose JSON as the data format for relational databases. You can set your preferred topic prefix, which organizes topics in the format ```<your-prefix>.<db_name>.>table-name>``` .

Verify successful setup by checking if the connector is operational:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/1a1b6d5c0741eab3c60cc8a09f75923092d94928/images/debezium_working_estrecho.png)

Certainly! Let's enhance your guide with detailed technical steps explained in simple terms for each part of the process:
_________

> **Problems troubleshooting:**
> 
> It might occur that you have this error and is really probable that is because your RDS is not reachable. Try to nslookup <the-rds-endpoint> to check if it give an IP. This IP should be public. Or check the Security Groups.
> ![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c86e7b32285b1af0df187ed3454565dbbdef44c2/images/debezium_connector_Error.png)
> Or you can have this other error about needing ```binlog_format=ROW  ```. As this:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/5bacea33e5f4e19f285591430173f91793062592/images/row_error_!.png)
> This is something you need to change in the Configuration parameters of your RDS. You will probably be attached to a default one, which cannot be changed. Go in   ```RDS instance > Configuration > DB instance parameter group  ``` and check it. So you will need to create a  new one and set   ```binlog_format=ROW  ```. REMEMBER TO REBOOT THE RDS, or it won't apply. :wink:	
  ________
  
## **Create an S3 Egress point in Confluent Cloud**

This step is straightforward within Confluent Cloud:

- Navigate to the Confluent Cloud network settings and click on Egress Endpoint.
- Configure the endpoint with the necessary details:
![Create_egress_point_s3 image](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c528c6f05d869e2146a93c7510e8b82c46520f6a/images/creating_s3_egress_point.png)
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c528c6f05d869e2146a93c7510e8b82c46520f6a/images/s3_access_point1.png)

The "VPC endpoint DNS name" generated here will be needed for setting up the S3 Sink connector.

![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/7db0f8af95039a71498d167be52113c5ecc2cf03/images/s3_egress_point.png)


________

### **Create an S3 Sink Connector**
Finally, let's configure the S3 Sink connector to send data from Confluent Cloud to the S3 bucket:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c528c6f05d869e2146a93c7510e8b82c46520f6a/images/s3linklogo.png)

- Set up the S3 Sink connector by selecting the desired topic and providing your API Key.
- Configure the connector settings, including the store URL constructed from the information obtained in the S3 egress endpoint setup:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/ad826c7c4c5caf026bd15c73640c2d95218dd5bd/images/s3_sink_connector_settings.png)

For the store URL, we need the code we had in the Egress access point and add it to
  ```
  https://bucket.<the-so-called-Service-inside-our-S3-egress endpoint-vpce-.........>
  ```

Bucket name is just the one you decided. Just its name.
  
______
# **Verify Data Flow in S3**

Check your S3 bucket to see the folder containing JSONs of the messages passed through the configured topic:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/a710c0cb30f41dc74d87635c94e77b540e50b46c/images/s3_topics.png)

**Thank you for reading till here! ** :sparkling_heart:	
