# Phase1: Configure a the CC Network and the Proxy

This part follows what stated in https://docs.confluent.io/cloud/current/networking/ccloud-console-access.html.
In my own example, we will use NGINX as proxy, but you can also use other tools like HAProxy or Envoy.
- [**<span style="color:orange">AWS</span>**]  $${\color{blue}Blue}$$	
 Create a new VPC if you want to start from scratch. I will strongly suggest you the enhanced creationn setting because in this way it will create all the possible private and public subnets with Internet Gateway with Route Table automatically.
- [**<span style="color:rgb(0,0,255)">Confluent Cloud</span>**]
 Inside an environment, create a new network in the region that you want. Select Private Link as networking type.
- [**<span style="color:rgb(0,0,255)">Confluent Cloud</span>**] Create inside your CC VPC, a PrivateLink Access. Save that com.amazonaws.vpce.<region>.xxxxxx name for later. It will take time.
- [**<span style="color:orange">AWS</span>**] Create an EC2 Instance, that will work as a bastion host for us here, it will have NGINX installed inside.
- [CC] Create a Dedicated Cluster inside that CC VPC.



- [**<span style="color:orange">AWS-EC2</span>**] Setup NGINX Proxy on EC2 Instance: This will act as a gateway for your local machine to connect to Confluent Cloud through the PrivateLink.

```
sudo apt update
sudo apt install nginx
nginx -t
sudo apt-get install net-tools
sudo apt -y install libnginx-mod-stream
sudo vi /etc/nginx/nginx.conf # to resolve the route 53 hosted zones we created earlier.
```
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
Then closing the text editor:
```
sudo systemctl restart nginx
sudo systemctl status nginx
```
Now we need a final step but we will do it later!


- [AWS-VPC]
  - Setup PrivateLink in AWS
        By creating a VPC endpoint, AWS allocates a “special network interface” inside your VPC. This interface acts like a “private doorway” that only your VPC can use to reach Confluent Cloud. This keeps all data traffic between your VPC and Confluent Cloud inside the secure AWS network.



      - Create VPC Endpoint
  - Configure Security Groups
 Security groups act like virtual firewalls around your AWS resources. Configuring them ensures only authorized data traffic can pass through the VPC endpoint to and from Confluent Cloud.
By adjusting security group rules, you specify which types of data traffic (like emails or file transfers) are allowed to travel between your AWS network and Confluent Cloud through the private VPC endpoint. This tight control improves network security by blocking unauthorized access attempts.

Open to your VPC CIDR, for example 10.0.0.0/16, the ports 9092, 443 and 80.
    
  - Update Route Table

   -  Create Private Hosted Zones in Route 53 - check each region with correct match
Ok, now, this part is tricky and you need to be VERY careful. Please use the notepad.
  DNS (Domain Name System) resolution lets computers translate website names (like www.example.com) into IP addresses (like 192.0.2.1) that they can use to find each other on the internet.
Enabling DNS resolution means your AWS network can translate Confluent Cloud's website names (like services.confluentcloud.com) into private IP addresses used only within your VPC. This ensures smooth communication between your VPC and Confluent Cloud, even if those IP addresses change.
Here you need to create a custom “private DNS zone” for the Confluent domain specified by the Confluent network
Identify the DNS name or IP addresses of the interface endpoints and the DNS wilcarsd records that need to be created to point to each of the endpoints.
Create wildcard CNAME records (AWS) or A records (for Azure). They take each zonal DNS subdomain and resolve it to a VPC endpoint.


Route 53 is AWS's service for managing DNS. Configuring “hosted zones” in Route 53 ensures that your AWS network can resolve (find) Confluent Cloud's services using their names internally, without relying on public internet DNS services.
By setting up private hosted zones in Route 53, you make sure that any requests from your AWS network to find Confluent Cloud services are handled internally within AWS. This keeps your communications secure and compliant with privacy standards.

DNS name of the CC cluster in the record name - private hosted zone.
CNAME for the main VPC endpoint, that goes with a * only.
Zonal endpoint record for the AZ *.xxxx.

- NOW , that we did ALL of this.
- [From your computer] Open sudo vi /etc/hosts.

  Add a line with:
```
  <Public-IP-of-your-bastion-host> <Bootstrap-of-the-cluster> #without any port so no :9092, no :443.
```

Ta-daaaan. You will be able to see your topics from your browser. That's amazing.
