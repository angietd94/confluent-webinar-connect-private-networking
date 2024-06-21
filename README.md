# Confluent demo: 
# Using Fully managed connector with private networking, for dummies
Here we are going to take advantage of fully managed connector using a Private Link connection.


Let's summarize all of we need by now.

- You need to have access to a Confluent cloud account.
- You need to probably have a credit card attached to a Confluent cloud accountbecause it could be attached to more cost to create this infra.
- You need to have a AWS account access.

If you want to have more documentation please open https://docs.confluent.io/cloud/current/networking/private-links/aws-privatelink.html .
Some theory: as the Dedicated cluster is inside a VPC (private), if you want to see anything from your UI, from your browser, you need to actually create some proxy to reach that.

Phase 1: We will mount the proxy with NGINX
Phase 2: We will mount the actual connect demo


##Phase1: Configure a the CC Network and the Proxy

This part follows what stated in https://docs.confluent.io/cloud/current/networking/ccloud-console-access.html.
In my own example, we will use NGINX as proxy, but you can also use other tools like HAProxy or Envoy.

- [CC] Inside an environment, create a new network in the region that you want. Select Private Link as networking type.
- [CC] Create inside your CC VPC, a PrivateLink Access. It will take time.
- [AWS] Create an EC2 Instance, that will work as a bastion host for us here, it will have NGINX installed inside.
- 
