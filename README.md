 ## About The Project
This project demonstrates the configuration of Network Address Translation (NAT) and Access Control Lists (ACL) on Cisco routers using the Cisco Packet Tracer simulation tool. The primary goal is to enable a private local area network (LAN) to access an external network (simulated as the Internet) via NAT, and to control traffic flow using ACLs.

## Simulated Scenario:

Router (R1): Acts as the gateway between the internal and external networks.

Switch (SW1): Connects the devices within the LAN.

PC (PC1): Represents a host within the internal network.

Server (Server1): Simulates an Internet server or an external host.

Addressing Scheme Used (Example):

Internal Network (LAN): 192.168.1.0/24

PC1: 192.168.1.10

R1 (GigabitEthernet0/0 interface): 192.168.1.1

External Network (WAN/Internet): 203.0.113.0/30

R1 (GigabitEthernet0/1 interface): 203.0.113.1

Server1: 203.0.113.2

## Configuration Features
NAT Overload (PAT): Translates multiple private IP addresses to a single public IP address using port numbers.

Default Route: Configures a default route to direct traffic towards the simulated Internet.

Standard Access List: Used to identify internal addresses permitted for NAT translation.

Extended Access Lists: Used for granular control of traffic based on protocol, port, and source/destination addresses.

Outbound LAN Traffic Control: Blocking ICMP (ping) and permitting HTTP traffic from the internal network to the external network.

Inbound WAN Traffic Control: Blocking ICMP (ping) from the external network to a specific internal host.

## Setup in Cisco Packet Tracer
To replicate this project in Cisco Packet Tracer, follow these steps:

1. Design the Topology
Devices: Place one Router (e.g., 2911), one Switch (e.g., 2960), one PC, and one Server on the workspace.

Connections:

Router (GigabitEthernet0/0) to Switch (FastEthernet0/1) using a Straight-through cable.

Switch (FastEthernet0/2) to PC (FastEthernet0) using a Straight-through cable.

Router (GigabitEthernet0/1) to Server (FastEthernet0) using a Straight-through cable.

2. Configure IP Addresses on End Devices
PC1:

IP Address: 192.168.1.10

Subnet Mask: 255.255.255.0

Default Gateway: 192.168.1.1

Server1:

IP Address: 203.0.113.2

Subnet Mask: 255.255.255.252

Default Gateway: 203.0.113.1

## Router Configuration Commands (CLI)
Click on the Router (R1), navigate to the CLI tab, and enter the following commands in sequence:

Router> enable
Router# configure terminal
Router(config)# hostname R1

! --- Configure Router Interfaces ---
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# description *** LAN Interface ***
R1(config-if)# ip nat inside
R1(config-if)# exit

R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip address 203.0.113.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# description *** WAN Interface / Internet ***
R1(config-if)# ip nat outside
R1(config-if)# exit

! --- Configure Default Route ---
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2

! --- Configure NAT (Network Address Translation) ---
! Define a Standard ACL to identify internal addresses allowed for NAT
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
! Implement NAT Overload (PAT) using the egress interface's IP address
R1(config)# ip nat inside source list 1 interface GigabitEthernet0/1 overload

! --- Configure ACLs for Traffic Control ---
! ACL to control outbound traffic from LAN (block ping, permit HTTP)
R1(config)# ip access-list extended NO_PING_OUT
R1(config-ext-nacl)# permit tcp any any eq 80  ! Permit HTTP traffic
R1(config-ext-nacl)# deny icmp any any        ! Deny ICMP (ping) traffic
R1(config-ext-nacl)# permit ip any any        ! Permit all other IP traffic
R1(config-ext-nacl)# exit

! ACL to control inbound traffic to LAN (block ping to PC1)
R1(config)# ip access-list extended BLOCK_PING_INBOUND
R1(config-ext-nacl)# deny icmp any host 192.168.1.10  ! Deny ping to PC1
R1(config-ext-nacl)# permit ip any any                ! Permit all other IP traffic
R1(config-ext-nacl)# exit

! --- Apply ACLs to the Appropriate Interfaces ---
R1(config)# interface GigabitEthernet0/0  ! Inside (LAN) interface
R1(config-if)# ip access-group NO_PING_OUT out  ! Apply outbound ACL
R1(config-if)# exit

R1(config)# interface GigabitEthernet0/1  ! Outside (WAN) interface
R1(config-if)# ip access-group BLOCK_PING_INBOUND in ! Apply inbound ACL
R1(config-if)# exit

! --- Save Configuration ---
R1(config)# end
R1# write memory

## Testing the Configuration
After applying all the above commands:

Test Ping from PC1 to Server1:

On PC1, open Command Prompt and enter ping 203.0.113.2.

Expected Result: Request timed out (due to NO_PING_OUT ACL).

Test HTTP Access from PC1 to Server1:

First, ensure the HTTP service is enabled on Server1 (under the Services tab of the server).

On PC1, open Web Browser and enter the address 203.0.113.2.

Expected Result: The default webpage from Server1 should be displayed (as NO_PING_OUT ACL permits HTTP).

Test Ping from Server1 to PC1:

On Server1, open Command Prompt and enter ping 192.168.1.10.

Expected Result: Request timed out (due to BLOCK_PING_INBOUND ACL).

## Common Troubleshooting
Red link between Router and Server/Switch:

Ensure router interfaces are enabled with the no shutdown command. (e.g., interface GigabitEthernet0/1 -> no shutdown)

Verify the correct cable type is used (typically Straight-through).

Confirm IP addresses and Subnet Masks are correctly configured.

ACL not working (traffic not blocked/blocked unexpectedly):

The order of lines in an ACL is crucial. They are evaluated top-down.

Ensure the ACL is applied to the correct interface and in the correct direction (in or out).

Use show access-lists to verify ACL definition and show ip interface [interface-type/number] to check applied ACLs.

Clear the NAT cache with clear ip nat translation *.

## Contributing
We welcome contributions! If you have ideas for improving this project, find a bug, or want to add a feature, please:

Open a new Issue and describe your problem or suggestion in detail.

Submit your changes by creating a Pull Request.

