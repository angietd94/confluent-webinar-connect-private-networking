# Part 2: Demo with Confluent Connectors

Here the idea is to take advantage of Confluent fully managed connectors, Debezium V2 Source and S3 Sink, using Private link.
_Note: here you will need some AWS Keys, if you do not have them, create one in the IAM service._

This is the setup we will mount:
- ![screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/3577fc6387dd30cd4aef9d906c536714bc038d9a/images/demo-schema.png)



- **Create a MySQL database**
First we create a MySQL database inside the RDS service of AWS
When creating the database please select MySQL as it is the example we are touching here.
Also dset Credential Management as Self Managed, in order to choose your own password. Save this password as we will need it later.
First we create some mock data with a Python script that goes inside the MySQL database (which is an RDS instance).

image select_rds_database

ssh -i "<my-pem>.pem" ec2-user@ec2-xx-xxx-xxx-xxx.eu-west-1.compute.amazonaws.com


mysql -h <my-rds-endpoint> -P 3306 -u your_mysql_username -p (and it will ask for your password here).
- **Create an S3 Bucket**

- **Create an S3 Egress point in Confluent Cloud**
- **Create an S3 Sink Connector**
- Now we have the easiest part, just create our S3 Sink connector. Select the topic you want to insert there, the usual API Key.
- image s3linklogo
- For the store URL, we need the code we had in the Egress access point and add it to
- httpz://bucket.<what-we-copied>
Bucket name is just the one you decided.
- 
