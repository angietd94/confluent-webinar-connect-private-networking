# Confluent demo: ---- WORK IN PROGRESS!! ---------
# Using Fully managed connector with AWS private networking, for dummies.
Here we are going to take advantage of fully managed connector using a Private Link connection.


Let's summarize all of we need by now.

Private Link offers a highly secure option as it establishes a private, unidirectional connection from your VPC/VNet to Confluent Cloud. This means Confluent Cloud services cannot initiate connections to your environment, enhancing security by minimizing attack surfaces. Note: is TCP not HTTP/S so this is why more networking configuration is needed.
With Private Link note that no /16 is needed , only 1 IP address is fine. And it is better for the customer as since is unidirectional we cannot access the customers’ resources.
Accessing the AWS VPC console allows you to manage networking components such as VPCs, subnets, route tables, and endpoints. This is where you will configure the necessary resources for setting up AWS PrivateLink

- You need to have access to a Confluent cloud account.
- You need to probably have a credit card attached to a Confluent cloud accountbecause it could be attached to more cost to create this infra.
- You need to have a AWS account access.

  Smart suggestion: Ope whatever notepad. Save all the Network Overview Confluent Cloud data, you will need it. Also we will use this notepad later. You don't want to do mistakes! :)

If you want to have more documentation please open https://docs.confluent.io/cloud/current/networking/private-links/aws-privatelink.html .


## Phase 1: We will mount the proxy with NGINX
Some theory: as the Dedicated cluster is inside a VPC (private), if you want to see anything from your UI, from your browser, you need to actually create some proxy to reach that. And this is what all this part is about.
- https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/main/Part1-networking-and-proxy.md

- ![Screenshot of the setup]([https://myoctocat.com/assets/images/Connect%20to%20dedicated%20PL.png))


## Phase 2: We will mount the actual connect demo
- https://github.com/angietd94/confluent-webinar-connect-private-networking/blob/main/Part2-connectors-demo.md 

