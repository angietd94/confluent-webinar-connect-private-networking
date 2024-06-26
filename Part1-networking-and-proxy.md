# Part1: Configure a the CC Network and the Proxy - not optional, you must do it before Part2

This part follows what stated in this documentation, [Use Confluent Cloud with Private Networking](https://docs.confluent.io/cloud/current/networking/ccloud-console-access.html). In my own example, we will use NGINX as proxy, but you can also use other tools like HAProxy or Envoy.
The following is the set we want to create here:
![https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/ae64e83b2224dc67ca633ec89122fbe0e3c3602d/images/general_schema.png](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/3577fc6387dd30cd4aef9d906c536714bc038d9a/images/general_schema.png)

_______
## **Inside Confluent Cloud**
  - **Create a CC Network**


Within your Confluent Cloud environment, create a new network in your desired region. 
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/78a6e8d7294ebb96597d46cfd2110c1782ecba31/images/new_cc_network.png)

Choose _Private Link_ as the networking type. This network setup ensures secure and private communication between your resources in AWS and Confluent Cloud.
 
 ![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/02d8e389d68c2ff5dcd2ea44df0e7331b5358b56/images/create_new_network.png)
  - **Create PrivateLink Access in CC VPC**

Generate a **PrivateLink Access** in your Confluent Cloud VPC. Save the ```com.amazonaws.vpce-xxxxx.<ID>``` name for later use. This step establishes a private connection endpoint in AWS that Confluent Cloud can use securely.
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/78a6e8d7294ebb96597d46cfd2110c1782ecba31/images/private_link_creation.png)

  - **Create a Dedicated Cluster** inside that CC VPC.
Set up a Dedicated Cluster inside your Confluent Cloud VPC. Upon completion, note down the bootstrap link provided. This link is crucial for connecting your applications to Confluent Cloud securely.

![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/7166c30d2561fb28ae77f9f8367e841ff3327644/images/bootstrap.png)
 
__________
## **AWS**

- **Create a AWS VPC**
If starting from scratch, use AWS's enhanced VPC creation settings. This automatically sets up private and public subnets with an Internet Gateway and Route Table. This setup ensures your network is well-structured and secure.

  ![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/02d8e389d68c2ff5dcd2ea44df0e7331b5358b56/images/create_vpc_smartly.png)



- **Setup VPC Endoint in AWS**
By creating a VPC endpoint, AWS allocates a “special network interface” inside your VPC. This interface acts like a “private doorway” that only your VPC can use to reach Confluent Cloud. This keeps all data traffic between your VPC and Confluent Cloud inside the secure AWS network.
Since we are from Confluent, choose the Partner one here:



![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/f6dfba100159dc17e8c465541a22614823af061d/images/Create%20endpoint.png)


Save all of this data or keep it a open tab because we will need it for hosted zones.
    
- **Configure Security Groups**
Security groups act like virtual firewalls around your AWS resources. Configuring them ensures only authorized data traffic can pass through the VPC endpoint to and from Confluent Cloud.
By adjusting security group rules, you specify which types of data traffic (like emails or file transfers) are allowed to travel between your AWS network and Confluent Cloud through the private VPC endpoint. This tight control improves network security by blocking unauthorized access attempts.

Open to your VPC CIDR, for example 10.0.0.0/16, the ports 9092, 443 and 80.
In the end it will look like this:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/e63aba60b173fabad30bd0ca060f5bd0a81c8117/images/vpc_endpoint_security_group.png)

## **Create Private Hosted Zones in Route 53** - 
_check each region with correct match_

Ok, now, this part is tricky and you need to be VERY careful. Please use the notepad.
DNS (Domain Name System) resolution lets computers translate website names (like www.example.com) into IP addresses (like 192.0.2.1) that they can use to find each other on the internet. Enabling DNS resolution means your AWS network can translate Confluent Cloud's website names (like services.confluentcloud.com) into private IP addresses used only within your VPC. This ensures smooth communication between your VPC and Confluent Cloud, even if those IP addresses change.
Here you need to create a custom “private DNS zone” for the Confluent domain specified by the Confluent network

Identify the DNS name or IP addresses of the interface endpoints and the DNS wilcarsd records that need to be created to point to each of the endpoints.
Here you reate wildcard CNAME records (AWS) or A records (for Azure). They take each zonal DNS subdomain and resolve it to a VPC endpoint.
Route 53 is AWS's service for managing DNS. Configuring “hosted zones” in Route 53 ensures that your AWS network can resolve (find) Confluent Cloud's services using their names internally, without relying on public internet DNS services.

By setting up private hosted zones in Route 53, you make sure that any requests from your AWS network to find Confluent Cloud services are handled internally within AWS. This keeps your communications secure and compliant with privacy standards.

DNS name of the CC cluster in the record name - private hosted zone.
CNAME for the main VPC endpoint, that goes with a * only.
Zonal endpoint record for the AZ *.xxxx.
Going back to your VPC endpoint, make this connection in your head:

![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/07c7a9ac329af2850c60fb7ec55ee38ba72c24b9/matching.png)

So you will add the following 4 records to your hosted zone _(Record Type: CNAME for everyone)_
1. Record name: *, Record Type: Cname, Value: the "general" DNS name
2. Record name: *eu-west-1a (or whichever your AZ), Value its corrispondent DNS name
3. Record name: *eu-west-1b (or whichever your AZ), Value its corrispondent DNS name
4. Record name: *eu-west-1c (or whichever your AZ), Value its corrispondent DNS name

   
This is how it will look like:
![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/7166c30d2561fb28ae77f9f8367e841ff3327644/images/Hosted_zones_setup.png)


**Some theory here sorry!**

_Why Route 53 Private Hosted Zones?_

_**Secure Communication**: Private hosted zones in Route 53 ensure that DNS resolution occurs within the AWS network. This means when your AWS resources need to communicate with Confluent Cloud services, they resolve domain names to private IP addresses that are accessible only within your Virtual Private Cloud (VPC). This keeps all communication secure and private, as traffic never leaves the AWS infrastructure._

_**Compliance and Data Security**: By using private IP addresses for DNS resolution, you maintain compliance with security standards and regulations. It prevents exposure of your infrastructure to the public internet, reducing the risk of unauthorized access and potential data breaches._

_**Efficient Network Traffic**: Resolving domain names to private IP addresses within AWS reduces latency and improves network performance. It ensures that communications between your AWS resources and Confluent Cloud services are optimized and reliable._

- _**Private vs. Public Hosted Zones**_

_**Private Hosted Zones**: These are used for internal DNS resolution within your AWS VPC. They resolve domain names to private IP addresses that are only accessible within your VPC. This setup is ideal for applications and services that do not need to be publicly accessible, ensuring a high level of security._

_**Public Hosted Zones**: These are used for DNS resolution that is accessible from the public internet. They resolve domain names to public IP addresses that can be accessed globally. This is typically used for websites, APIs, and other services that need to be publicly available_

____

## **Setup EC2 Instance as Bastion Host with NGINX**:
   
Launch an EC2 Instance in AWS, configured as a bastion host. Install NGINX on this instance to act as a proxy gateway. NGINX will facilitate secure communication between your local machine and Confluent Cloud through the PrivateLink. You can connect to this EC2 in various forms, either from the connect button in the AWS UI, of if you have a keypair with SSH (in this case, please remember to check Security groups port 443). Important: Select for the EC2 to have a Public IP because we will need this.

 - Choose wether Amazon Linux or Ubuntu. In Amazon Linux is like a CentOs and Ubuntu is Linux so some commands may differ! But there is Google for that ;)  Here is use Amazon Linux Free Tier.
- [**<span style="color:orange">AWS-EC2</span>**] Setup NGINX Proxy on EC2 Instance: This will act as a gateway for your local machine to connect to Confluent Cloud through the PrivateLink.

```
sudo apt update
sudo apt install nginx
nginx -t
sudo apt-get install net-tools
sudo apt -y install libnginx-mod-stream #this stream module is needed if not you will get an error.
sudo vi /etc/nginx/nginx.conf # to resolve the route 53 hosted zones we created earlier.
```
Some more info on this process [here](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=nginx&f=12#google_vignette).
If you are in Ubuntu it will be all with YUM, so for example search quickly online and you will find stuff such as ```sudo yum install nginx-mod-stream``` , etc.
Depending on your cloud provider, reconfigure to use the cloud provider’s resolver:
```
For AWS: resolver 169.254.169.253
For Azure: resolver 168.63.129.16
For Google Cloud: resolver 169.254.169.254
```
Here, we use AWS.
```
stream {
  map $ssl_preread_server_name $targetBackend {
     default $ssl_preread_server_name;
 }

 server {
   listen 9092;

   proxy_connect_timeout 1s;
   proxy_timeout 7200s;

   resolver 169.254.169.253;

   proxy_pass $targetBackend:9092;
   ssl_preread on;
 }

 server {
   listen 443;

   proxy_connect_timeout 1s;
   proxy_timeout 7200s;

   resolver 169.254.169.253;

   proxy_pass $targetBackend:443;
   ssl_preread on;
 }
}
```
It will look like this more or less:

![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/7166c30d2561fb28ae77f9f8367e841ff3327644/images/nginx_conf.png )
Then closing the text editor:
```
sudo systemctl restart nginx
sudo systemctl status nginx
```
This is what we expect to see:

![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/28276f3754ab5380ce86fa051b2847b8343a7f72/images/nginx_working.png)

Now we need a final step but we will do it later...
____


- **NOW** , that we did ALL of this.
- 
- [From your computer] Open ```sudo vi /etc/hosts```. I like VI but you can use another text editor. _Note: With vi you save clicking_ ```ESC``` _and then typing_ ```:wq!``` _._

  Add a line with:
```
  <Public-IP-of-your-bastion-host> <Bootstrap-of-the-cluster> #without any port so no :9092, no :443.
```
In this way.


![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/7166c30d2561fb28ae77f9f8367e841ff3327644/images/code_hosts.png)

Ta-daaaan. You will be able to see your topics from your browser. That's amazing.


![Screenshot](https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/ea7dab092189c44c963d1a579029ddbdb3b3f197/images/Topics_visible.png)
