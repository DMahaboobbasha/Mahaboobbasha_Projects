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

Let's open your `AWS account` -> `Home Console` -> search `VPC`. Open `VPC` -> Here you'll 
have 2 options:-   1.`VPC Only`  2.`VPC and more`.

Select 2.`VPC and more` on right-side you can see a PREVIEW of the `Architecture of the resources`
which AWS will be creating for us.
      - Two Private & Public Subnets -> each one in 2 Availability Zones.
      - Route Tables
      - Internet Gateway
      - NAT Gateway
      - S3 Endpoint (optional).

Next, `#Name tag auto-generation` -> "aws-prod-example". Can leave IPv4 (default) * select `No IPv6`
-> Number of Availability Zones `2` , Number of Public Subnet `2` , Number of Private Subnet `2`
NAT Gateway `1 per AZ` -> VPC Endpoints `None` (no need in this project) -> `Tick` Enable DNS Hostname and Enable DNS Resolution, -> click **`Create VPC`**

On next page AWS will show all the resources that are being created & also `Elastic IP Address`.
-> _Here Elastic IP is attached to NAT Gateway which is used to hide IP Address of servers in Private subnet when these servers are accessing Internet._

Now go to `VPC` Service -> Open your VPC **aws-prod-example**. Scroll down  -> **Resource Map** to 
see `VPC Architecture` you created.

Till now we created VPC Componenets and `Other things` which needs to be created are **`EC2 Instances (servers) in Private subnets`** using `Auto-Scaling Group` and `Load Balaner`. But here
you also need to create a **Bastion Host** in `Public Subnet`. Because since you are creating 2 EC2 Instances in Private subnets but to access these instances and install Python Application and 
`Web Page` we need a Bastion Host in public subnet to access your Private Subnet Instance. 

So let's create `Auto-Scaling Group`. 1st Search **EC2** -> click it and on left-side search Auto-Scaling Groups. Open it and click `Create Auto Scaling Group`. -> 
              _`In AWS, ASG can,t be created without Launch Template. So 1st you need create a ASG 
                Launch Template which is used as a reference in creating other ASG and also when 
                **Scaling-Up** new instances during high traffic.`_
                
Here, Launch template -> click `Create a launch template` -> **Launch template Name** `aws-prod-example` -> **Template Version Description** `Proof of AWS Private Subnet for App deploy`. -> **AMI** `ubuntu 22.04` (as per your need). -> **Instance type** ` t2.micro` -> **key pair value**
_`Pick one u already downloaded into your system or create new and download pem file for logging in SSH.`_ 

 ###### **Network Settings:-**
 **Subnet** `no change` -> **Create Security Group** ` aws-prod-example` -> **Description** 
 `Allows SSH Access` **VPC** select`which we created`  --> click **Add inbound Secrurity Group 
  rule**  _`As you are going to create/install a Python Application in our server/instance and 
  you are going to access this application through PORT 8000.`_  Hence **Type** `SSH` -> **Source
  Type** choose `Anywhere IPv4`. Again add one more rule with **Type** `Custom TCP` -> **PORT**
  `8000` -> **Source** `Anywhere IPv4` -->  _leave other things default_ -> click **`Create Launch 
  Template`**.
                    _`These configurations may look similar to creating EC2 Instances. Yes, because
                    with **ASG**  you are creating EC2 Instances to **Scale-Up** our application 
                    during high traffic`_

  Back to creating Auto-Scaling Group `refresh page` -> **ASG Name** ` aws-prod-example`  **Launch 
  Template** `aws-prod-example`(which you created just now) -> click `Next` **VPC** `aws-prod- 
  example`. **Availability Zones & Subnets** select _2 AZs and select only private subnet in each
  AZ_  -->   _because you'll deploy ASG in private subnets and attach to other private instances     in 2 AZs._ -> click **Next** -> **Load Balancing** choose `No Load Balancer` (as you're going to 
  it separately late).   _Because in this project " ASG " is in Private Subnets and You'll create
  " Load Balancer " in Public Subnets. -> select `No VPC Lattice Services` -> **Health Checks**  
  `300` -> **NEXT** -> **Group Size** -> **Desired Capacity** `2`  **Minimum Capacity** `1` 
  **Maximum Capacity** `4`.  -> **Scaling Policies** `None`. -> **NEXT** -> **NEXT** -> **NEXT**.
  --> **Create Auto Scaling Group.**

  _Now let's check if your ASG created EC2 Instances you asked above in different AZs_.   Go to EC2
  Instance -> check these instances AZs beside their ID.

  Before creating Load Balancer ->> Let's install Python application in one of the Private subnet 
  (to check the working of Load Balancer in both private instances). But to install application in 
  private subnet you need to login to that subnet, -->> to login we need `public IP` which is not 
  attached to private subnet.  So you need a **Bastion Host** in public subnet through which you 
  are going to Login to Private subnet. 

  _**Bastion Host** which acts as a mediator between Private subnet & External person who is trying
   to login to Private subnet through Public subnet._ 
  
  **TO CREATE A BASTION HOST** >> go to EC2 Console -> `Instances` -> **Launch Instance.**
  -->> **Name** `Bastion Host` -> **AMI** `ubuntu 22.04` -> **Instance Type** `t2.micro` ->
  **Key Pair Value** `select one you downloaded` -> **Network Settings** -> **Edit**  -->
  **VPC** `aws-prod-example` **Subnet** select`public-east-1a` **Auto-assign public IP** `Enable`
  select `Create Security Group` (_as this SG is in public subnet_)  **Security Group name**
  `default(wizard)` but in **inbound rule** add `SSH Port 22`. -->>  **Launch Instance**.

  _Now to install Python application in Private subnet, you need to login to Private subnet and 
  for this you need Key Pair Value to be inside your **Bastion Host** we copy this value in 
  Terminal . Even to login to Bastion Host we need same Key Pair Value as you gave same Key Pair   
  Value to both Private and Bastion instances. 

  ####### **Open Terminal** 
  here go to dowloads folder where you have your instances `Key Pair Values` and use 
  _#ls | grep aws-login.pem_
  `aws-login.pem`  will show file. _now you'll copy this file from your local system to Bastion 
  Host before even logging into Bastion using command  _#scp -i /users/Mahaboob/Downloads/aws-
  login.pem    /users/Mahaboob/Downloads/aws-login.pem  ubuntu@**Bastion PublicIP**:/home/ubuntu._
  Now terminal will ask to click `Yes`.   -->> Key pair value copied into Bastion.

  Now let's login to **Bastion Host** and check pem file using command
  _#ssh -i aws-login.pem ubuntu@BastionPublicIP_  `logged into Bastion`
  Now let's login to **Private Subnet 1st** to install Python Applicatoin using command
  _#ssh -i aws-login.pem ubuntu@privateInstancePrivateIP_    `$ yes`   `logged in`


  Now You're inside Private subnet, let's install Python Application and create a small html page
  using commands _#vim index.html_   this opens text editor -> let's write a small html code 
  (index.html) inside this vim file and save with press ESCAPE + _#:wq!._  Now you will run Python 
  server in this private subnet through Terminal (which is preinstalled with ubuntu OS, if not use 
  _#sudo apt install Python3_ to install python) and using this command run Python server
  _#python3 -m http.server 8000_  
 
  > [!NOTE]
  > 8000 is python port we allowed in private SG.

  ######## **Let's Create Load Balancer**
  Now let's create your Load Balancer and attach these 2 Private subnets as `Target Groups` 
  and try to access your `html page` from internet.
  Go to __EC2__ -> Left-side -> Click **Load Balancers** --> click **Create Load Balancer** 
  -> select **Application Load Balancer**  _`Other Load Balancers have different purpose_`
  -> **Create**. 

  > [!NOTE]
  > Application Load Balancer distributes **HTTP** & **HTTPS** traffic across multiple
    Targets as `EC2`  `Microservices`  `Containers`.  **This takes place at Layer 7.**

  On next page **Load Balancer Name** `aws-prod-example` -> **Scheme** `Internet-facing` ->
  select `IPv4` -->> **Network Mapping** -> **VPC** `aws-prod-example` -> **Mapping** 
  `select both availability zones and select public subnet in both AZs`. 
  **Security Group** `aws-prod-example`  _because in this SG you allowed ports you need_
  **Listener & Routing** -> **Protocol** `HTTP`**Port** `80`  click **Create Target Group** ->
  select **Instances** -> **Target Group Name** `aws-prod-example` -> select `HTTP1` -> **NEXT**
  select `2 Private Instances` as Target Groups -> below click **include as pending below** ->
  **Create Target Group**  _<ins>Wait till Target Group is created.</ins>_

  Now go back to Load Balancer page -> refresh near `Target Group` -->> select **Now created 
  <ins>Target Group</ins>**  -> scroll down -> click **Create Load Balancer**. 

  After few minutes click `View Load Balancer` -> open your Load Balancer -> scroll down -> 
  in **Listeners** -> check `<ins>PORT you got ERROR as HTTP:80</ins>`. Because in your Security
  Group you didn't allow **HTTP 80** as an `inbound rule`.

  So, now go to **Security Group** in EC2 or there only. Scroll down click **Add inbound rule**
  **Type** `HTTP` **PORT** `80` **Source** `Anywhere IPv4` -> click **Save Rule.**
  Now go to **Load Balancer** `refresh`

  Let's Access **HTML Page** you created in your 1st Private Subnet by copying DNS link of
  Load Balancer and Paste in web browser tab **Got our page output**

  **Congratulations, You implemented your AWS VPC Project successfully.**

  But,
  > [!IMPORTANT]
  > This application and html page is running only in 1st private subnet but not on 2nd Private
    subnet. So after creating Load Balancer when you will access this html page from internet,
    only in 1st 2-3 web Tabs you will access this page later you will get `ERROR` as entire
    traffic will distributed to 1st subnet and when traffic will be distributed to 2nd subnet.
    will get  `ERROR`  since in this subnet applicaton and html page wasn't installed. Because
    as you are using Load Balancer the user traffic is 1st equally [50%-50%] distributed among
    two private instances.

 Now to fix this, open **Target Group** and can see **Unhealthy**, because your entire internet
 traffic is going to only one healthy `private subnet` and not 2nd. 
 Login [SSH] into 2nd Private Subnet through Terminal. Install Python3 application and save your 
 **HTML Page** and refresh Load Balancer and access the site (HTML) in multiple tabs and web 
 browsers. 


 You successfully completed and did all troubleshooting required to launch your VPC and access
 site. 


    
  

  
  
  

  
