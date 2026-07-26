# Router ACL-Based Network Segmentation Lab

## Objective
### This project focused on implementing traffic segmentation on a Cisco router using extended access control lists (ACLs). The goal was to restrict internal lateral communication between hosts based on protocol and subnet, allowing only specific, sanctioned traffic between endpoints while blocking everything else. The lab also involved verifying rule behavior through test cases and evaluating ACL placement and performance.

### Skills Learned
- Designing and implementing extended ACLs to control traffic by protocol, source, and destination
- Applying least-privilege principles to internal network segmentation
- Testing and validating ACL rules using protocol-specific traffic (HTTP and HTTPS)
- Understanding ACL placement strategy and its impact on performance
- Monitoring ACL hit counts to confirm rule effectiveness

### Tools Used
- Cisco IOS for ACL configuration
- Packet Tracer for topology simulation
- Browser-based testing for traffic verification

## Steps

1: Network Topology

<img width="1192" height="490" alt="Screenshot 2026-07-26 154208" src="https://github.com/user-attachments/assets/2af71be6-fda2-497b-b4d0-eb00a7b6e493" />

Built the topology in Cisco Packet Tracer with Router1 segmenting two internal subnets:
 - 10.1.1.0/24 - HTTP Server1 (10.1.1.100) and HTTP Server2 (10.1.1.101), behind Switch1 
 - 10.1.2.0/24 - Inside PC1 (10.1.2.101) and Inside PC2 (10.1.2.102), behind Multilayer Switch0

---

2: Created extended ACL on Router1

<img width="550" height="102" alt="Screenshot 2026-07-26 152529" src="https://github.com/user-attachments/assets/0de8ea01-0721-49ad-8d4b-675d735b44b2" />

- Line 10: Inside PC1 (10.1.2.101) can reach HTTP Server1 (10.1.1.100) on HTTP only
- Line 20: Inside PC2 (10.1.2.102) can reach HTTP Server2 (10.1.1.101) on HTTPS only
- Line 30: Explicit deny for all other traffic from 10.1.2.0/24 to 10.1.1.0/24
- Line 40: All other outbound traffic from 10.1.2.0/24 permitted

---

3: Verified the configuration with test cases

<img width="845" height="530" alt="Screenshot 2026-07-26 153542" src="https://github.com/user-attachments/assets/95c8546a-9ea4-4400-8c6a-0b69b57dce11" />

- Inside PC1 successfully loaded "http://10.1.1.100" in its browser, confirming HTTP access to HTTP Server1


<img width="781" height="437" alt="Screenshot 2026-07-26 153623" src="https://github.com/user-attachments/assets/6f975d34-93cc-4810-8713-a62497074925" />

- Inside PC2 successfully loaded "https://10.1.1.101" in its browser, confirming HTTPS access to HTTP Server2


<img width="577" height="97" alt="Screenshot 2026-07-26 153721" src="https://github.com/user-attachments/assets/e74a07e8-0a62-4a49-90cd-616798c3088c" />

Confirmed via "show access-lists" that hit counters incremented as expected: 
 - 6 matches on the PC1 -> Server1 HTTP rule
 - 6 matches on the PC2 -> Server2 HTTPS rule
 - 30 matches on the deny rule
