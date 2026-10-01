# AWS Cloud Security Monitoring Lab

## Objective
The objective of this lab is to design and build a secure AWS VPC with multiple segmented subnets to simulate an enterprise network environment. The lab focuses on practicing AWS networking, least-privilege access control, security monitoring, log collection, and incident detection using services and tools such as IAM, S3, Splunk, and endpoint security tools.

## Skills Learned
- Configured AWS networking components including Internet Gateways, NAT Gateways, route tables, and security groups to control communication between public and private resources and subnets.
- Created IAM users and groups and implemented identity-based policies following the principle of least privilege.
- Designed a segmented VPC containing separate public, private, and SOC subnets based on system roles and security requirements.
- Deployed and configured Splunk for centralized security monitoring and log collection.
- Integrate AWS security logs into Splunk for centralized log aggregation and monitoring.
- Deploy CrowdStrike Falcon EDR to endpoints for endpoint telemetry, threat detection, and response.
- Practiced troubleshooting Linux services, permissions, EC2 resource constraints, ports, security groups, and cross-subnet connectivity.
  
## Network Topology

<img width="1694" height="691" alt="Screenshot 2026-09-16 160255" src="https://github.com/user-attachments/assets/8a4054d2-1605-4747-bd69-2e106126a769" />

I configured a VPC using 192.168.0.0/16 and divided it into three subnets to separate public-facing web server, internal servers, and security monitoring.

| Subnet | CIDR | Resources | Purpose |
| --- | --- | --- | --- |
| Public | `192.168.2.0/24` | Nginx web server, bastion host, NAT Gateway | Hosts public-facing services and provides outbound internet connectivity for private resources. |
| Private | `192.168.1.0/24` | Splunk server, Samba file server, MySQL database server | Hosts internal applications, shared files, and centralized logging. |
| SOC | `192.168.3.0/24` | Windows SOC workstation with Splunk forwarder | Provides a dedicated environment for security monitoring and investigation. |

Routing and Access Controls
- The public subnet’s default route points to the Internet Gateway.
- The private and SOC subnets default routes point to the NAT Gateway in the public subnet for internet access.
- Each route table includes 192.168.0.0/16 -> local for communication within the VPC.
- Security groups added on each instance for inbound and outbound traffic control
- Access from the SOC workstation to the Samba file share uses TCP 445 over the VPC’s internal network.

----

# Server Setup and Configuration

## Samba File Server

<img width="696" height="191" alt="Screenshot 2026-09-21 213959" src="https://github.com/user-attachments/assets/92f61c70-d36d-4ff3-97c4-8dfab0097ffa" />
<img width="622" height="291" alt="Screenshot 2026-09-21 213224" src="https://github.com/user-attachments/assets/06521b6c-ab31-4eff-a012-c0e9e7e8ec6b" />

 - Installed Samba on an Ubuntu EC2 for file sharing and is located in private subnet. 
 - I configured a Samba file share on my Ubuntu file server to provide centralized file storage. I created a shared directory at /home/ubuntu/share and configured it in the Samba configuration file to allow authorized clients to browse, read, and write files. I also configured the server to require authenticated access instead of allowing guest connections.


    Security group:
   
       - Allow inbound to port 445 from 192.168.3.0/24 (SOC Subnet)
       - Allow inbound to port 22 from 68.187.54.129/32 (My IP)
   

## MySQL Server

<img width="625" height="167" alt="Screenshot 2026-09-21 214727" src="https://github.com/user-attachments/assets/60c629af-0c6d-43bb-842b-96f6e8927807" />

 - I installed and configured MySQL Server on my Ubuntu database server. After installation, I used “systemctl status mysql” to verify that the MySQL service was enabled and actively running.


   Security group:
   
       - Allow inbound to port 3306 from 192.168.1.0/24 (Private Subnet)
       - Allow inbound to port 22 from 68.187.54.129/32 (My IP)
   

## Splunk Server

<img width="1276" height="794" alt="Screenshot 2026-09-06 201935" src="https://github.com/user-attachments/assets/5fdebe45-f517-42d6-9e52-00c973977246" />

 - Installed and configured Splunk for centralized log management to collect VPC flow logs, CloudTrail, and S3 logs
 - Confirmed Splunk is running by checking if port 8000 is listening and URL is accessable


      Security group:
   
       - Allow inbound to port 8000 from 192.168.3.134/32 (SOC Workstation)
       - Allow inbound to port 8088 from  sg-0284be47ea87a9fae (Splunk Lambda Forwarder)
       - Allow inbound to port 22 from MyAdministrativeIP/32 

   
----

# IAM Roles and Access Control

I implemented IAM users, groups, policies, and service roles to follow the
principle of least privilege.


## IAM Users and Groups

- **IT** - Access to resources required for infrastructure administration.
- **HR** - Access limited to HR-related S3 resources.
- **Finance** - Access limited to Finance-related S3 resources.
- **SOC** - Access to security resources required for monitoring and incident response.


## VPC Flow Logs IAM Role

<img width="818" height="458" alt="image" src="https://github.com/user-attachments/assets/680b5a63-f5a7-441f-8ba6-48c446486076" />

- Created an IAM role to allow the VPC Flow Logs service to assume the role and publish network flow records to CloudWatch logs



<img width="1100" height="693" alt="Screenshot 2026-09-22 225141" src="https://github.com/user-attachments/assets/defe2b85-40db-45e4-89ee-7ce1c1df171c" />

- I attached a policy to the VPC Flow Log role that allows it to work with CloudWatch Logs
- It also allows the role to write events to any log group and can create log streams

----

# Forwarding AWS Logs to Splunk via Lambda Function

Configured a Lambda Function to forward logs from CloudWatch to Splunk using the HTTP Event Collector (HEC). This provides centralized log aggregation in Splunk for AWS security telemetry such as VPC Flow Logs and CloudTrail activity

I made two separate log pipelines:

**VPC Flow Logs**
- CloudWatch Logs/Log Group
- Subscription Filter
- Lambda
- Splunk HEC
- index="aws_vpc"

<img width="854" height="673" alt="Screenshot 2026-09-27 204654" src="https://github.com/user-attachments/assets/cbbf656b-0532-415d-9a2b-d777743a80f6" />


**CloudTrail Logs**
- CloudWatch Logs/Log Group
- Created log trail to assign to Log group
- Subscription Filter
- Lambda
- Splunk HEC
- index="aws_cloudtrail"

<img width="809" height="674" alt="Screenshot 2026-09-27 204727" src="https://github.com/user-attachments/assets/dafc5603-304a-478c-8bf0-8a78cb6d50ec" />

----

## I confirmed that VPC Flow logs and CloudTrail logs were successfully forwarded and indexed by Splunk

**VPC Flow Logs**
<img width="1628" height="843" alt="Screenshot 2026-09-27 121537" src="https://github.com/user-attachments/assets/1f40d141-67d1-462a-aa1c-b2c2f776b2d7" />

**CloudTrail Logs**
<img width="1297" height="790" alt="Screenshot 2026-09-27 202223" src="https://github.com/user-attachments/assets/73f13275-57df-4e41-9a2c-701a0cc691ff" />


----

# Troubleshooting

While trying to forward VPC Flow logs to Splunk, CloudWatch was successfully using the Lambda function, but the function initially timed out while attempting to connect to the Splunk HEC endpoint.

I troubleshot this by checking each step of the logging pipeline:
 - Confirmed VPC Flow logs were being written to CloudWatch Logs
 - Confirmed CloudWatch subscription filter was attached to Lambda function
 - Verified Splunk server was listening on '0.0.0.0:8088' for accepting HEC connections
 - Used tcpdump to check whether connections are being accepted on port 8088
 - Identified that Ubuntu UFW was configured with a default-deny inbound policy and was not allowing TCP 8088.
 - Updated UFW and AWS security group rules to permit HEC traffic.
 - Verified the TCP handshake and confirmed successful Lambda-to-Splunk communication.

After correcting the firewall and security group configuration, the Lambda function successfully forwarded AWS logs to Splunk.


