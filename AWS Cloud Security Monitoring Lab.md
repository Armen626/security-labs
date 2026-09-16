# AWS Cloud Security Monitoring Lab

## Objective
The objective of this lab is to design and build a secure AWS VPC with multiple segmented subnets to simulate an enterprise network environment. The lab focuses on practicing AWS networking, least-privilege access control, security monitoring, log collection, and incident detection using services and tools such as IAM, S3, Splunk, and endpoint security solutions.

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
