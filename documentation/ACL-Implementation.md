ACL Implementation Documentation
1. Objective

The objective of this lab was to implement Cisco Extended Access Control Lists (ACLs) to control communication between different departmental VLANs.

The security policy was designed to restrict specific ICMP traffic while maintaining normal communication between authorized departments.

2. VLAN Design
VLAN	Department	Network		Gateway
10	HR		10.10.10.0/24	10.10.10.1
20	ADMIN		10.10.20.0/24	10.10.20.1
30	IT		10.10.30.0/24	10.10.30.1
40	SALES		10.10.40.0/24	10.10.40.1

The router performs inter-VLAN routing using Router-on-a-Stick.

3. End Devices
Device	Department	IP Address	Default Gateway
PC1-HR	HR	10.10.10.10/24	10.10.10.1
PC2-ADMIN	ADMIN	10.10.20.10/24	10.10.20.1
PC3-IT	IT	10.10.30.10/24	10.10.30.1
PC4-SALES	SALES	10.10.40.10/24	10.10.40.1
4. ACL Security Policy

The following security requirements were implemented:

HR

HR users must not be able to send ICMP echo requests to:

ADMIN
IT
SALES
IT

IT users are permitted to communicate with:

HR
ADMIN
SALES
ADMIN

ADMIN users are permitted to communicate with SALES.

SALES

SALES users must not be able to send ICMP echo requests to HR.

5. HR ACL

The following extended ACL was created:

ip access-list extended HR-RESTRICT
 deny icmp 10.10.10.0 0.0.0.255 10.10.20.0 0.0.0.255 echo
 deny icmp 10.10.10.0 0.0.0.255 10.10.30.0 0.0.0.255 echo
 deny icmp 10.10.10.0 0.0.0.255 10.10.40.0 0.0.0.255 echo
 permit ip any any

The ACL was applied inbound on:

interface G0/0.10
 ip access-group HR-RESTRICT in

This prevents HR-originated ICMP echo requests from reaching the other departmental networks.

The final permit ip any any ensures that traffic not explicitly denied by the ACL is permitted.

6. Sales ACL

A second ACL was created for the SALES VLAN:

ip access-list extended SALES-TO-HR
 deny icmp 10.10.40.0 0.0.0.255 10.10.10.0 0.0.0.255 echo
 permit ip any any

It was applied inbound on:

interface G0/0.40
 ip access-group SALES-TO-HR in

This prevents SALES users from initiating ICMP echo requests toward HR.

7. Why ICMP Echo Was Filtered

The ACLs deny the ICMP echo request rather than blocking all IP traffic.

This is important because ICMP communication consists of both an echo request and an echo reply.

Filtering only the request provides more controlled behavior and avoids unnecessarily blocking return traffic.

8. ACL Direction

ACL direction was an important part of troubleshooting this lab.

An ACL applied inbound processes packets as they enter the router interface.

An ACL applied outbound processes packets as they leave the interface.

During testing, an overly broad outbound ACL caused return ICMP traffic to be blocked. This demonstrated that ACLs are evaluated independently on packet direction.

The final configuration therefore uses targeted ICMP echo-request filtering.

9. Verification

The following tests were performed.

Blocked Traffic
HR → ADMIN     BLOCKED
HR → IT        BLOCKED
HR → SALES     BLOCKED
SALES → HR     BLOCKED
Allowed Traffic
IT → HR        SUCCESSFUL
IT → ADMIN     SUCCESSFUL
IT → SALES     SUCCESSFUL
ADMIN → SALES  SUCCESSFUL

All departmental PCs were also able to reach their respective default gateways.

10. Verification Commands

The following Cisco IOS commands were used during verification:

show vlan brief
show ip interface brief
show access-lists
show running-config

Connectivity was tested from the VPCS endpoints using:

ping <destination-ip>
11. Troubleshooting
Issue

An initial ACL implementation used a broad IP deny on an interface.

This caused return traffic to be blocked and resulted in unexpected ping failures in the opposite direction.

Resolution

The ACL was redesigned to deny only the required ICMP echo requests:

deny icmp <source-network> <destination-network> echo

A final:

permit ip any any

was also included.

Result

The required traffic restrictions were successfully implemented while authorized communication remained functional.

12. Final Result

The ACL lab successfully demonstrated practical implementation of network security policies using Cisco Extended ACLs.

The completed configuration provides:

VLAN-based network segmentation
Inter-VLAN routing
Department-level access control
Directional ACL processing
ICMP traffic filtering
Network troubleshooting
Verification through connectivity testing

The final Cisco configurations were saved to startup configuration.