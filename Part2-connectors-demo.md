# Part 2: Demo with Confluent Connectors

Here the idea is to take advantage of Confluent fully managed connectors, Debezium V2 Source and S3 Sink, using Private link.

This is the setup we will mount:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c929dee23ac648d93b1887e816ee599eae4d041d/images/demo-schema.png)

_From this picture you would see S3 all alone in a corner. I dediced to picture it this way as S3 as a PaaS service, fully managed by AWS, lives outside of any custom VPC. For security reasons you should make it accessible only from your VPC with VPC Endpoints if the content you are going to store on S3 musn't be available from the public Internet, so this is what we are trying here._ _More info at https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-s3.html._

Preliminary Notes :
- _Note: here you will need some AWS Keys, if you do not have them, create one in the IAM service._
- You will need some API Cluster Key. You can create in inside the UI of CC, inside your cluster, or using the CLI. It comes in a form key and password, save them as they are important everytime you need to create things in Confluent.
___________

### **Create a MySQL database**
First we create a MySQL database inside the RDS service of AWS
When creating the database please select MySQL as it is the example we are touching here.
Also dset Credential Management as Self Managed, in order to choose your own password. Save this password as we will need it later.
First we create some mock data with a Python script that goes inside the MySQL database (which is an RDS instance).

![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/7db0f8af95039a71498d167be52113c5ecc2cf03/images/select_rds_database.png)


Remember, to access the EC2 Bastion we created before, this is the command, or if possible just from the Connect AWS button:
```
ssh -i "<my-pem>.pem" ec2-user@ec2-xx-xxx-xxx-xxx.eu-west-1.compute.amazonaws.com
```
_Of course, you will need install in your instance all the MySQL and Python stuff. Remember that Amazon Linux EC2 = CentOs._
I will give some external documentations for this:
- https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-centos-7
- https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04
- https://phoenixnap.com/kb/how-to-install-python-3-ubuntu
- https://www.liquidweb.com/kb/how-to-install-python-3-on-centos-7/
  
Then we can access the inside of the database:
```
mysql -h <my-rds-endpoint> -P 3306 -u your_mysql_username -p # (and it will ask for your password here)
```

Note: you can also see from the UI the content of your database with some magic, for example by using PhpMyAdmin in Ubuntu or Adminer in CentOs.

________
### **Create an S3 Bucket**

Select your region and the name you prefer.
Select the Access to be blocked from the outside as we want to take advanged of Private Link here.
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/5a258b2727bc5d801bd35ad2ad7fade560b9117b/images/bucket_block.png)

________

### **Create an S3 Egress point in Confluent Cloud**

This is extremely easy. In the CC network just click on Egress Endpoint and type the following.
![Create_egress_point_s3 image](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c528c6f05d869e2146a93c7510e8b82c46520f6a/images/creating_s3_egress_point.png)
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c528c6f05d869e2146a93c7510e8b82c46520f6a/images/s3_access_point1.png)

This is what you should see when created. Consider that this "VPC endpoint DNS name" will be then needed to create the S3 Sink connector.
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/7db0f8af95039a71498d167be52113c5ecc2cf03/images/s3_egress_point.png)


________


### **Create an S3 Sink Connector**
Now we have the easiest part, just create our S3 Sink connector. Select the topic you want to insert there, the usual API Key.
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c528c6f05d869e2146a93c7510e8b82c46520f6a/images/s3linklogo.png)

You will the configuration here:

![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/ad826c7c4c5caf026bd15c73640c2d95218dd5bd/images/s3_sink_connector_settings.png)

For the store URL, we need the code we had in the Egress access point and add it to
  ```
  https://bucket.<what-we-copied>
  ```


- Bucket name is just the one you decided.
  
