# Cisco ACL-Based Network Security

## Overview



This project demonstrates the implementation of Cisco Extended Access Control Lists (ACLs) to control inter-VLAN communication and enforce department-level network security policies.



The lab was built in GNS3 using a Cisco router configured with Router-on-a-Stick and a Layer 2 switch with multiple VLANs.



The objective was to restrict selected traffic between departments while allowing authorized communication to continue.



Technologies Used

GNS3

Cisco IOS Router

Cisco IOS Layer 2 Switch

VLANs

802.1Q Trunking

Router-on-a-Stick

Extended IPv4 ACLs

ICMP traffic filtering

VPCS

### Network Topology

&#x20;

&#x09;	    Cisco Router

&#x20;                    R1 / G0/0

&#x20;                        |

&#x20;                   802.1Q Trunk

&#x20;                        |

&#x20;                   Cisco Switch

&#x20;                        |

&#x20;      +-----------------+-----------------+-----------------+

&#x20;      |                 |                 |                 |

&#x20;   VLAN 10           VLAN 20           VLAN 30           VLAN 40

&#x20;      |                 |                 |                 |

&#x20;   HR PC             ADMIN PC           IT PC            SALES PC

10.10.10.10        10.10.20.10       10.10.30.10       10.10.40.10

IP Addressing

Department	VLAN	Network	Gateway	PC

HR	10	10.10.10.0/24	10.10.10.1	10.10.10.10

ADMIN	20	10.10.20.0/24	10.10.20.1	10.10.20.10

IT	30	10.10.30.0/24	10.10.30.1	10.10.30.10

SALES	40	10.10.40.0/24	10.10.40.1	10.10.40.10

Router Configuration



The router uses sub interfaces on G0/0 to provide inter-VLAN routing.



G0/0.10  → 10.10.10.1/24

G0/0.20  → 10.10.20.1/24

G0/0.30  → 10.10.30.1/24

G0/0.40  → 10.10.40.1/24



Each sub interface uses IEEE 802.1Q encapsulation for its corresponding VLAN.

### 

### Security Requirements



The following communication policies were implemented:



Source	Destination	Result

HR	ADMIN	❌ Blocked

HR	IT	❌ Blocked

HR	SALES	❌ Blocked

IT	HR	✅ Allowed

IT	ADMIN	✅ Allowed

IT	SALES	✅ Allowed

ADMIN	SALES	✅ Allowed

SALES	HR	❌ Blocked



The ACLs specifically filter ICMP echo requests rather than blocking all IP traffic. This allows normal IP communication and return traffic to continue where permitted.



### ACL Configuration

HR Restriction



The HR-RESTRICT ACL was applied inbound on the HR VLAN sub interface.



ip access-list extended HR-RESTRICT

&#x20;deny icmp 10.10.10.0 0.0.0.255 10.10.20.0 0.0.0.255 echo

&#x20;deny icmp 10.10.10.0 0.0.0.255 10.10.30.0 0.0.0.255 echo

&#x20;deny icmp 10.10.10.0 0.0.0.255 10.10.40.0 0.0.0.255 echo

&#x20;permit ip any any



Applied to:



interface G0/0.10

&#x20;ip access-group HR-RESTRICT in

Sales-to-HR Restriction

ip access-list extended SALES-TO-HR

&#x20;deny icmp 10.10.40.0 0.0.0.255 10.10.10.0 0.0.0.255 echo

&#x20;permit ip any any



Applied to:



interface G0/0.40

&#x20;ip access-group SALES-TO-HR in

Verification



The ACL implementation was tested using ICMP ping between the departmental PCs.



### Expected Results

HR → ADMIN     ❌ Blocked

HR → IT        ❌ Blocked

HR → SALES     ❌ Blocked



IT → HR        ✅ Successful

IT → ADMIN     ✅ Successful

IT → SALES     ✅ Successful



ADMIN → SALES  ✅ Successful



SALES → HR     ❌ Blocked



All PCs were also able to reach their respective default gateways.



### Key Learning



One important lesson from this project was understanding that extended ACLs are evaluated on packets traveling in a specific direction on an interface.



A previous test using an outbound ACL caused return ICMP traffic to be blocked, which demonstrated why simply blocking a destination network can unintentionally affect reverse communication.



The final design uses ICMP echo-request filtering to enforce the required restrictions while allowing echo replies and other permitted IP traffic.



### Project Outcome



The project successfully demonstrates:



VLAN segmentation

Inter-VLAN routing

Router-on-a-Stick configuration

Extended ACL implementation

Department-level traffic filtering

Directional ACL application

ICMP traffic control

Network security troubleshooting

Verification and testing using Cisco IOS



The final configuration was tested successfully and saved to the device startup configuration.



### Environment



Platform: GNS3

Network OS: Cisco IOS

Testing Tool: VPCS

Project Type: Network Security / ACL Lab



### Screenshots



Screenshots demonstrating the topology, VLAN configuration, ACL configuration, and connectivity tests will be stored in the screenshots/ directory.



#### **Author**

#### 

#### **Muneeb Ur Rehman**

#### 

#### **Networking \& IT Support**

#### **Cisco | GNS3 | VLANs | Network Security**

