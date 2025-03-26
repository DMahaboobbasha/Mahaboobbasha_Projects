# **AWS VPC with Public-Private Subnets in Prod Environment Project**

## **About the Project**
* This Project demonstrates how to create a VPC that you can use for servers in a Prod Environment.
* To improve resiliency, you will deploy servers & NAT Gateway in two Availability Zones using Auto-
  Scaling along with Application Load Balancer. For additional security: you deploy servers in 
  Private subnets,
  these servers receive requests through Load Balancer. Servers can connect to the internet using
  NAT Gateway.
  
### <ins>Overview:-</ins>
- This VPC has public & private subnets in two Availability Zones.
- Each Public subnet contains a NAT Gateway & a single Load Balancer node.
- Servers run in the private subnets, launched (scaled up) & terminated by using Auto-Scaling Group 
  and receive traffic through Load Balancer.
  Servers can connect to the internet using NAT Gateway.

  ![image](https://github.com/user-attachments/assets/751207c0-a21c-4171-bbef-95980caf0ed9)
)

#### **AWS Services used in this Project:**
- VPC
- Internet Gateway
- Application Load Balancer
- Auto-Scaling Group
- NAT Gateway
- EC2 Instances

##### Let's Implement this Project in AWS

Let's open your `AWS account` -> `Home Console` -> search `VPC`. 
