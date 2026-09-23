# AWS Cloud Security Monitoring Lab

## Objective
The objective of this lab is to design and build a secure AWS VPC with multiple segmented subnets to simulate an enterprise network environment. The lab focuses on practicing AWS networking, least-privilege access control, security monitoring, log collection, and incident detection using services and tools such as IAM, S3, Splunk, and endpoint security tools.

## Skills Learned
- Configured AWS networking components including Internet Gateways, NAT Gateways, route tables, and security groups to control communication between public and private resources and subnets.
- Created IAM users and groups and implemented identity-based policies following the principle of least privilege.
- Designed a segmented VPC containing separate public, private, and SOC subnets based on system roles and security requirements.
- Deployed and configured Splunk for centralized security monitoring and log collection.
- Integrate AWS security logs, Nginx web server logs, and Samba file server logs into Splunk for centralized log aggregation and monitoring.
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




