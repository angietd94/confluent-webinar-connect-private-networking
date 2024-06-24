# Part 2: Demo with Confluent Connectors

Here the idea is to take advantage of Confluent fully managed connectors, Debezium V2 Source and S3 Sink, using Private link.

This is the setup we will mount:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c929dee23ac648d93b1887e816ee599eae4d041d/images/demo-schema.png)

_From this picture you would see S3 all alone in a corner. I dediced to picture it this way as S3 as a PaaS service, fully managed by AWS, lives outside of any custom VPC. For security reasons you should make it accessible only from your VPC with VPC Endpoints if the content you are going to store on S3 musn't be available from the public Internet, so this is what we are trying here._ _More info at https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-s3.html._


_______
> _Preliminary Notes :_
> - _Note: here you will need some AWS Keys, if you do not have them, create one in the IAM service. [Here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) is explained in the AWS docs. _
> - You will need some API Cluster Key. You can create in inside the UI of CC, inside your cluster, or using the CLI. It comes in a form key and password, save them as they are important everytime you need to create things in Confluent.
___________
#**Create the AWS infrastructure**
## **Create a MySQL database and fill it with data**
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
- [How to install MySQL on CentOS](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-centos-7)
- [How to install Python on CentOs](https://www.liquidweb.com/kb/how-to-install-python-3-on-centos-7/)
- [How to install MySQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04)
- [How to install Python on Ubuntu](https://phoenixnap.com/kb/how-to-install-python-3-ubuntu)
  
Then we can access the inside of the database:
```
mysql -h <my-rds-endpoint> -P 3306 -u your_mysql_username -p # (and it will ask for your password here)
```

Note: you can also see from the UI the content of your database with some magic, for example by using PhpMyAdmin in Ubuntu or Adminer in CentOs.

I filled the MySQL data with random data created by [a Python code](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/main/mysql-python-script.py). I have attached here in the repo. Change the values of the database to yours. You can also use nano or others, I like vi. :see_no_evil:	. Please feel free to change this code, it is just an example!
If you want to run it, you already know: 
```
sudo vi mysql-python-script.py #copy my code in here,
python3 mysql-python-script.py
```

## **Create an S3 Bucket**
Select your region and the name you prefer.
Select the Access to be blocked from the outside as we want to take advanged of Private Link here.
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/5a258b2727bc5d801bd35ad2ad7fade560b9117b/images/bucket_block.png)


Ok, at this point we have our MySQL database, an S3 bucket and our Dedicated Cluster all alone in the AWS world, but how to connect them?
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/512e2f8a107246250d5092f2c5ddcb07a0ef1c5a/images/only_mysql_and_s3.png)
________
# Inside Confluent Cloud...
## Create MySQL CDC Debezium V2 Connector

Finally! We can't create our amazing connectors.
Go inside your cluster and then select   ```Connectors > Add Connector > MySQL Debezium CDC Source V2 ```.
This is the setup:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/5318c2a0b7168ad4ebfaf7e34bafe2d8268e1f65/images/debezium_creatiom.png)



![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/3e301b08c9fa0249752cfcdca4c5fc721f31f0fa/debezium_setup.png)
We select JSON as this is a relational database. The topic prefix is literally up to you.
It will create the topics in the form of:
```
<your-prefix>.<db_name>.>table-name>
```

If you see this, it is a really good sign:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/1a1b6d5c0741eab3c60cc8a09f75923092d94928/images/debezium_working_estrecho.png)

> Problems troubleshooting:
> 
> It might occur that you have this error and is really probable that is because your RDS is not reachable. Try to nslookup <the-rds-endpoint> to check if it give an IP. This IP should be public. Or check the Security Groups.
> ![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c86e7b32285b1af0df187ed3454565dbbdef44c2/images/debezium_connector_Error.png)
> Or you can have this other error about needing ```binlog_format=ROW  ```. As this:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/5bacea33e5f4e19f285591430173f91793062592/images/row_error_!.png)
> This is something you need to change in the Configuration parameters of your RDS. You will probably be attached to a default one, which cannot be changed. Go in   ```RDS instance > Configuration > DB instance parameter group  ``` and check it. So you will need to create a  new one and set   ```binlog_format=ROW  ```. REMEMBER TO REBOOT THE RDS, or it won't apply. :wink:	
  
## **Create an S3 Egress point in Confluent Cloud**

This is extremely easy. In the CC network just click on Egress Endpoint and type the following.
![Create_egress_point_s3 image](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c528c6f05d869e2146a93c7510e8b82c46520f6a/images/creating_s3_egress_point.png)
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/c528c6f05d869e2146a93c7510e8b82c46520f6a/images/s3_access_point1.png)

This is what you should see when created. Consider that this "VPC endpoint DNS name" will be then needed to create the S3 Sink connector.
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/7db0f8af95039a71498d167be52113c5ecc2cf03/images/s3_egress_point.png)



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
  
______
Now check what you see in S3.
You should be seeing a folder created with inside the JSONs of the messages that passed through that topic.
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/a710c0cb30f41dc74d87635c94e77b540e50b46c/images/s3_topics.png)

Thank you for reading till here! :sparkling_heart:	
