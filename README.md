# Setup-of-Application-Load-Balancer-With-Auto-Scaling-Group-on-AWS

## Overview
In this project we will implement Application Load Balancer(ALB) with Auto Scaling Group (ASG) and test its functionality. Auto scaling group will scale in and out instances preloaded with Apache HTTP Server loaded with HTML Page. The scaling takes place once the Target tracking Policy is triggered. We will also confirm that traffic is load balanced across available instance available in the Target group attached to the auto scaling group

The following services will be utilized in this project: VPC, Auto Scaling Group, Launch template, EC2 Instance, Security group, Application Load Balancer(ALB), Target group, Cloud Watch and CLI
## Prerequisite 
Access to AWS account
# Instructions

## Step 1:		Create VPC
Log on to AWS with an admin account. Search for VPC in the search bar.

Click on Create VPC

 ![image](https://github.com/user-attachments/assets/794f0538-f33f-406c-89f5-1e22ce42a256)



Below, Select VPC and More so we can configure all necessary parameters at once.

Enter the name for your VPC as desired and specify the CIDR block you require.
We won’t be using IPv6

![image](https://github.com/user-attachments/assets/3f96c223-2657-44b9-be03-afcc74a04d3c)


Set AZs to be 2 for high availability and Public subnets to be 2. Leave the Tenancy at default and Private subnets at 0 as we wont be needing Private subnet in this project
![image](https://github.com/user-attachments/assets/05a87a25-fb87-44f4-8399-ee4fa0e0d992)
 


Here, Leave default setting for Subnet CIDR block. We don’t need NAT gateways nor VPC endpoints, hence set both to None. Ensure DNS hostname& Resolution are checked.

 ![image](https://github.com/user-attachments/assets/81ecab5f-ffa4-4c0d-88ac-bade49b7736f)



## Step 2: 	Create Security Group
Now we will need two security groups, one for ALB and the other for the ECs instances that will be spun up by Autoscaling Group.

In the VPC dashboard, scroll down and click on Security Group then Create Security Group

![image](https://github.com/user-attachments/assets/93a3f594-66d7-4258-96ad-2247e2221684)



Here, name the Security group, choose appropriate VPC.
For the Internet facing ALB, we would configure HTTP protocol for inbound traffic and source as AnyWhere-IPv4 . We would leave the Outbound rules and every other setting as default.

We would scroll all the way down and click Create Security Group
![image](https://github.com/user-attachments/assets/2357b420-a25c-43aa-a9db-741be46fe7e4)



We would create a second Security Group for the EC2 Instances that will host the Apache Service that will configured in the launch template. This Security group will only permit traffic from the ALB. We would leave the Outbound rules and every other setting as default.

We would scroll all the way down and click Create Security Group
![image](https://github.com/user-attachments/assets/dceacb1f-d685-4652-9fd1-7724c99b1f35)



## STEP 3: 	Create Target group
Next, we will create Target group, this will be needed in configuring ALB. Your load balancer routes requests to the targets in a target group and performs health checks on the targets.

Navigate to EC2 console, scroll down and click Target Group.

Click Create Target Group
![image](https://github.com/user-attachments/assets/13c4adfa-4869-4bf0-bdaf-004ac7de547f)



The Target group is used to identify & group resources that the ALB will distribute traffic to. In our case, the resources are Instances. 

Hence, I chose Instances in the Target type
![image](https://github.com/user-attachments/assets/3c99e604-577d-4573-bf5b-0699d75a9f5f)
 


Name your Target Group, For Protocol Port choose HTTP since the targets are Instances running Apache server. 

IP address type remains IPv4 and VPC is changed to the VPC we just created. 

For Protocol version, we will leave HTTP1. Health Checks will remain at default HTTP and every other setting will be left at default. 

Scroll down to click on NEXT button
![image](https://github.com/user-attachments/assets/b7c618fb-363b-4356-9111-160dbaab8cde)



This next stage requires us to Register Targets (Instances). However, we do not currently have instances as our instances will be spun up by AutoScaling group. 

When we are creating Auto Scaling group later in this project, we will attach this Target group to the Auto Scaling group so that any new instance will automatically be captured in the Target group
![image](https://github.com/user-attachments/assets/bafdcb68-050d-48db-8470-11a9844a15f0)



We will leave every other settings default and click on Create Target Group
![image](https://github.com/user-attachments/assets/5546d612-e409-4bab-834c-f60a92ea010a)



## STEP 4:	Create Load Balancer

Click on Load Balancer on the left pane, then Click Create Load Balancer top right.
![image](https://github.com/user-attachments/assets/37543f59-7403-4fda-91cc-507e696a4c7a)



We will select Application Load balance (ALB) which is suitable for our use case. Click Create under the ALB
 ![image](https://github.com/user-attachments/assets/1805a12e-ae0b-499b-96bd-7f779cc23f14)




In the Basic Configuration section, I named my Load Balancer ALB_Apapche.

Also, in the Scheme section, chose Internet-facing since the ALB would be accessible from the internet. We will be using IPv4 address type for communication
 ![image](https://github.com/user-attachments/assets/a2191a5a-775a-4d55-8082-46b9ec644692)



In the Network mapping section, we will select the VPC earlier created as well as the public subnets in the availability zones where the load balancer will route traffic
 ![image](https://github.com/user-attachments/assets/3efcea42-5081-4b03-ac6c-76891364b4f6)



Here we will choose the security group we created for ALB earlier on
 ![image](https://github.com/user-attachments/assets/8dec7136-012f-41ed-9a15-013dc05da0b1)



In the Listener & Routing Section, we will specify the port which the Load Balancer will listen on to forward traffic to an assigned Target group. 

Protocol is Http while the Target group is the one earlier configured for this project. Then click Add Listener
![image](https://github.com/user-attachments/assets/60976546-4601-4923-8010-a0cae7f65e0e)



We would leave other remaining features below unchecked and scroll down, 

Review and click on Create Load Balancer
![image](https://github.com/user-attachments/assets/cacac5de-3623-4a6f-92d3-d963d793b4d0)
 


## STEP 5 :	Create Launch Template
Click on Launch Template on the left pane, then Click Create Launch Template towards the bottom of the loaded page.
![image](https://github.com/user-attachments/assets/67f7b8e5-869a-48a4-b5ed-4444926b4843)

 


  
Here, specify suitable name for the Launch template
 ![image](https://github.com/user-attachments/assets/f5251cde-6f71-434f-b00f-f330919766fa)



Select Amazon Linux under the Quick Start option, then choose Amazon Linux 2023 AMI as your AMI.
 ![image](https://github.com/user-attachments/assets/6dc167dc-ce7c-43b2-9b54-977d34f49978)



Under the Instance type, we will select t2.micro for the purpose of this demonstration, while we would leave the key pair blank since we won’t be connecting to the EC2 
 ![image](https://github.com/user-attachments/assets/a49a1259-89f6-464b-945e-39a6eb8b7c0b)



In the Network Setting section, we would not include subnet details, this would be specified in the Auto Scaling Group that this template will be attached to. However, we would select appropriate Security group earlier created.
![image](https://github.com/user-attachments/assets/2741811f-c3d2-4112-b54b-27228a1300d6)


 
We would leave everything else default and scroll down to expand Advance details. 

Here will locate User Details section at the bottom of the page where we would enter some User data to launch an Apache Server and provide a unique index.html Web page for each Instance created
![image](https://github.com/user-attachments/assets/32f1696a-bfa0-4525-891e-c402138a0d28)

Above is a script that updates and installs Apache Server, start and enable Apache then Enter a line in the index.html that displays the IP of each instance created.


At the bottom of this page click Create Launch Template
 ![image](https://github.com/user-attachments/assets/57c3d6e9-af36-414a-bc3b-71472b8612d2)



At the time of creating our VPC, the public subnets did not have Enable auto assign public IPv4 address. So, we would enable the option for the 2 public subnets.

Go to VPC dashboard, locate and click Subnets, on the right pane, select each public subnet earlier created and click on actions, then Select Edit Subnet Settings. 
![image](https://github.com/user-attachments/assets/8b5f909e-4951-4dbb-a50b-d00f39054224)


 
Check the Enable auto-assign public IPV4 address and click Save at the bottom of the page. Do this for both Subnets. 

This will ensure that new instances get public IPs when the are created
 ![image](https://github.com/user-attachments/assets/889db8f7-67af-4cfb-a590-ce23064dc41f)



## Step 6: Create Auto Scaling Group
Goto EC2 view, scroll down to the bottom of the left pane and click Auto Scaling  Group.

Click Create Auto Scaling  Group.
![image](https://github.com/user-attachments/assets/414b496e-e176-4d33-b37f-739ab0eb342f)

 

Click Create Auto Scaling  Group (ASG), and I’ll just call this ASG_Project. 

Specify launch template earlier created. it will load all the features defined in the Launch template then click Next at the bottom of the page
![image](https://github.com/user-attachments/assets/cde95456-310e-4fae-94dd-6dc64b8060e6)




Choose Instance launch Option

Under the Network section, Choose the earlier configured VPC & the desired public subnets in the availability zones to launch your new instances. Click Next at the bottom of the page
![image](https://github.com/user-attachments/assets/9a05c08f-9fdb-4a46-b0e0-e94644dd714b)

 

Under Configure Advanced Options, Choose Attach to an existing load balancer and Specify the Target Group of the load balancer you earlier created.

This ensures that when instances are launched, they are registered in the target group and the Load balancer can start forwarding traffic to them.
![image](https://github.com/user-attachments/assets/7e8394c5-187d-4f91-ba46-eda6ad3390d7)

 

Scroll down, under the Health Check section, check “Turn on Elastic Load Balancing health checks”

it means the autoscaling group will be notified when the Load balancer stops sending traffic to an instance because it’s not healthy. 

Then Auto scaling does status checks and initiate a replacement for such unhealthy instance(s). then Click Next at the bottom of the page
![image](https://github.com/user-attachments/assets/5d2ad056-6a96-4e7f-b905-f33835ccf1f1)


 
Under Configure group size and scaling

For Desired capacity i will set it to 2 so it can distribute the instances to 2 availability zones
 ![image](https://github.com/user-attachments/assets/52a86f69-47cb-48ee-82b8-bbe1299f6de1)



In the Scaling Section,” Minimum Desired Capacity”, is set to 2 so as to always have 2 instance running for redundancy. 

While for “Maximum Desired Capacity”, I chose 4 to give room for scale out incase traffic build up.

For “Automatic Scaling”, chose "Target Tracking Scaling Policy" to monitor performance metric selected in the Metric Type option below.

In Metric type Chose "Application Load Balancer request count per target" & Target group as configured earlier. I entered 30 as the target value for the purpose of this demonstration.
![image](https://github.com/user-attachments/assets/a164e7fc-469a-4d70-9fd2-26148624298a)
 


At the bottom, I will click Skip to review and Scroll down to the bottom of the review page. Click on "Create Auto Scaling Group"

Click on the Auto Scaling Group and goto Activity tab to see the status of the Instances been launched
![image](https://github.com/user-attachments/assets/fb98375c-2053-4d5b-bedb-35d38fd4d235)


 

Navigate to EC2 Terminal to see the status of the 2 desired Instances initializing
![image](https://github.com/user-attachments/assets/553fce29-c52f-40ce-9693-979853b50c31)


Locate and Click on "Target Groups". Confirm that the Instances are registered in the Target group and the Health Status is Healthy. That's when the Load balancer can send traffic to them.
![image](https://github.com/user-attachments/assets/664c94a9-9699-4571-9f7b-5279b8ace4cd)

 

## Step 7: To test Load Balancer
Once the Instances are Active in the Target group, got to Load Balancer and copy the DNS name. 

Go over to your browser, paste the DNS name of the Load Balancer and refresh a couple of times. The browser displays the IP address of Each Instance been load balanced. 

This confirms the Load balancer is distributing traffic to the 2 instances created
![image](https://github.com/user-attachments/assets/a0e076fb-5db9-4e5b-b831-584e87df0c51)

![image](https://github.com/user-attachments/assets/6b90d156-1cdf-4ff7-9ff4-57b8d95402f9)




## Step 8: To test Autoscaling Group
To test the Autoscaling feature, copy the DNS name of the ALB into the script below.

Connect to the cloudshell terminal to run script.

for i in {1..500}; do curl http:// ALB-Apache-1750122120.ca-central-1.elb.amazonaws.com /; done 

Its going to launch connections to the ALB 500 times, this is done to trigger the Target tracking policy we configured in Auto scaling group. We will run this script a couple of times
 ![image](https://github.com/user-attachments/assets/68caaa0e-0556-4f65-a812-b5e4c4263ffe)



Now, Lets go to CloudWatch to see the status of the Target Tracking alarms
![image](https://github.com/user-attachments/assets/930653fd-0c0e-4d07-a16a-0caa39a94d7b)

We have alarm High and Alarm Low and the states are currently “OK” as seen above
Our mission is to trigger the Target Tracking Alarm high. Its going to happen after the RequestCountPerTarget > 30 for 3 data points in 3minutes.

I will rerun this script several times to trigger AutoScaling.

After some minutes, in CloudWatch, we will see the Alarm High state has changed to “In Alarm”
![image](https://github.com/user-attachments/assets/4b2910b1-28ac-4445-88c2-662f2575250e)



Lets go over to the EC2 instance Terminal. We will observe that 2 more instances are initializing. This is because the Target Tracking alarm has been triggered 
![image](https://github.com/user-attachments/assets/2ed7c64c-3685-4d1f-be02-3216e9dc0adc)

 

We can also confirm this in the Auto Scaling Group terminal that 4 instances have been spun up
![image](https://github.com/user-attachments/assets/2a2da976-4029-40bb-a406-b7afaa6fddfb)

 

Let go over to our browser and confirm that the 4 instances are been loads balanced. Copy the DNS name of the Load Balancer to the browser and refresh a couple of times. 

The browser displays the IP address of each Instance been load balanced. 

This confirms the Load balancer is distributing traffic to the additional 2 instances created

![image](https://github.com/user-attachments/assets/76fbbab8-28f4-475c-a46e-084369a513fe)

![image](https://github.com/user-attachments/assets/dfc6dfe5-5e24-47a7-997a-3c76d7a64499)

![image](https://github.com/user-attachments/assets/35eec2a0-ac6d-40bc-832b-37562c6cb1a1)

![image](https://github.com/user-attachments/assets/10eeec9f-0949-4994-8da8-c77ab324758d)




At this time, we have stopped generating traffic to the Load balancer, hence the 4 instances that were scaled out before have now scaled in to the minimum desired capacity of 2 instances.
 ![image](https://github.com/user-attachments/assets/57f080b4-c6c6-4d55-b3e0-d4ad01af0887)



The current status of the Auto scaling group is also indicated in the  Instances view here
 ![image](https://github.com/user-attachments/assets/bd1b9143-24af-41a1-bdd0-12505baa6e2b)



Here, to further test our ASG, we have Terminated 1 of the 2 instances remaining. (Click check box for an instance, then go to Actions drop down at the top, select Terminate Instance)
 ![image](https://github.com/user-attachments/assets/665da7c2-8f8c-491e-9875-117e29e44283)



As seen below, we are left with just 1 instance running
 ![image](https://github.com/user-attachments/assets/4ab77372-5c4a-4439-904f-718c4bd0cc9d)



While we are configuring our minimum desired capacity in Autoscaling group, we specified 2 hence ASG automatically detects less instances and in response, it automatically initializes 1 additional instance as seen below
![image](https://github.com/user-attachments/assets/a8e68fdb-0c4b-4067-a376-0c2f65f409ad)


 
Again, on our browser, a refresh on the Load Balancer DNS name indicates a new instance with a new IP address added.
![image](https://github.com/user-attachments/assets/8c81aba0-0576-477d-b61c-1aaa93fc3c97)

![image](https://github.com/user-attachments/assets/4469c6ec-a08f-46ba-a590-0e2069a1ab47)


 
 
We have successfully implemented Auto Scaling Group & Application Load Balancer in practical terms. Our solution seamlessly scaled out when traffic increased and scaled in when traffic subsided which accentuates 3 pillars (efficiency, reliability and cost optimization) in AWS Well Architected Framework 
To avoid holding down resources or been charged, I recommend deleting the Application Load Balancer & Auto Scale group which will automatically terminate any instances associated with it. Thanks




