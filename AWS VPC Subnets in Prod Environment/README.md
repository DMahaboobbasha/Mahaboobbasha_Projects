# **AWS VPC with Public-Private Subnets in Prod Environment Project**

## **About the Project**
* This Project demonstrates how to create a VPC that you can use for servers in a Prod Environment.
* To improve resiliency 
### <ins>Overview:-</ins>
- This VPC has public & private subnets in two Availability Zones.
- Each Public subnet contains a NAT Gateway & a single Load Balancer node.
- Servers run in the private subnets, launched (scaled up) & terminated by using Auto-Scaling Group 
  and receive traffic through Load Balancer.
  Servers can connect to the internet using NAT Gateway.
