
Part 1: Configure the network according to specifications.

Switch(config)# hostname DLS1

Switch(config)# hostname DLS2

Switch(config)# hostname ALS1

Switch(config)# hostname ALS2

1. Disable the links between ALS1 and ALS2.

Switch(config-if-range)# interface range GigabitEthernet 0/1 - 2

Switch(config-if-range)# shutdown

2. Use multiple class C 192.168.X.0/24 networks as required for this SBA.

3. Configure the Fa0/11 link between DLS1 and DLS2 as a Layer 3 link and assign a network to it.

[DLS1] Switch(config)# interface Fa0/11

[DLS1] Switch(config-if)# no switchport

[DLS1] Switch(config-if)# ip address 192.168.0.1 255.255.255.0

[DLS1] Switch(config-if)# exit

[DLS2] Switch(config)# interface Fa0/11

[DLS2] Switch(config-if)# no switchport

[DLS2] Switch(config-if)# ip address 192.168.0.2 255.255.255.0

[DLS2] Switch(config-if)# exit

4. Configure the Fa0/12 link between DLS1 and DLS2 as an ISL trunk, and statically set all other interswitch links

as 802.1q trunks.

[DLS1] Switch(config)# interface FastEthernet 0/12

[DLS1] Switch(config-if)# switchport trunk encapsulation isl1

[DLS1] Switch(config-if)# switchport mode trunk

[DLS1] Switch(config)# interface range fastEthernet 0/7 - 11

[DLS1] Switch(config-if-range)# switchport trunk encapsulation dot1q

[DLS1] Switch(config-if-range)# switchport mode trunk

/**Optional BestPractice**/

[DLS1] Switch(config-if)# switchport nonegotiate optional

[DLS1] Switch(config-if)# switchport trunk allowed vlan 1-100

[DLS1] Switch(config-if)# no shutdown

[DLS1] Switch(config-if)# end

/**End BestPractice**/

[DLS2] Switch(config)# interface FastEthernet 0/12

[DLS2] Switch(config-if)# switchport trunk encapsulation isl1

[DLS2] Switch(config-if)# switchport mode trunk

[DLS2] Switch(config)# interface range fastEthernet 0/7 - 11

[DLS2] Switch(config-if-range)# switchport trunk encapsulation dot1q

[DLS2] Switch(config-if-range)# switchport mode trunk

/* Switch-urilor ASL1 si 2 li se aplica configuratia de trunchi*/

ALS1(config)# interface range fastEthernet 0/7 12

ALS1(config-if)# switchport mode trunk

ALS2(config)# interface range fastEthernet 0/7 12

ALS2(config-if)# switchport mode trunk

/* Verification trunk configuration */

ALS2# show interfaces fastEthernet 0/7 switchport

DLS1# show interfaces trunk

5. Bind the links between DLS1 and ALS1 in an EtherChannel and configure the two switches to actively negotiate

a PAgP link.

ALS1# show interfaces trunk

ALS1(config-if-range)# interface range GigabitEthernet 0/7 - 8

ALS1(config-if-range)# shutdown

ALS1(config)# interface range fastEthernet 0/7 - 8

ALS1(config-if-range)# channel-group 1 mode desirable

ALS1(config)# interface port-channel 1

ALS1(config-if)# switchport mode trunk

DLS1(config)# interface range fastEthernet 0/7 - 8

DLS1(config-if-range)# channel-group 1 mode desirable

/* Creating a port-channel interface Port-channel 1

DLS1(config)# interface port-channel 1

DLS1(config-if)# switchport mode trunk

/* Verificam configurarea pe ambele switch-uri*/

#show etherchannel summary

6. Place all switches in the VTP domain CISCO with DLS1 as the VTP server using VTP version 2. Configure all

other switches as VTP clients.

DLS1# show vtp status

All contents are Copyright © 1992–2011 Cisco Systems, Inc. All rights reserved. This document is Cisco Public Information. 
Page 3 of 7



DLS1(config)# vtp domain CISCO

Changing VTP domain name from NULL to CISCO

DLS1(config)# vtp version 2

DLS1(config)# vtp mode server

Device mode already VTP SERVER.

DLS2(config)# vtp mode client

Setting device to VTP CLIENT mode.

ALS1(config)# vtp mode client

Setting device to VTP CLIENT mode.

ALS2(config)# vtp mode client

Setting device to VTP CLIENT mode.

7. On DLS1, create VLAN 10 named CLIENT, VLAN 20 named VOICE, VLAN 30 named SERVER and VLAN 99

named MGMT. Choose a 192.168.X.0/24 network for each VLAN for use in subsequent steps.

DLS1(config)# vlan 10

DLS1(config-vlan)# name CLIENT

DLS1(config-vlan)# exit

DLS1(config)# vlan 20

DLS1(config-vlan)# name VOICE

DLS1(config-vlan)# exit

DLS1(config)# vlan 30

DLS1(config-vlan)# name SERVER

DLS1(config-vlan)# exit

DLS1(config)# vlan 99

DLS1(config-vlan)# name MGMT

DLS1(config-vlan)# exit

8. Ensure that the VLAN 1 interface on all switches is not used for administrative management or user traffic.

ALS1(config)# vlan 1

ALS1(config-vlan)# shutdown

ALS1# show vlan brief

ALS2(config)# vlan 1

ALS2(config-vlan)# shutdown

ALS2# show vlan brief

DLS1(config)# vlan 1

DLS1(config-vlan)# shutdown

DLS1# show vlan brief

DLS2(config)# vlan 1

DLS2(config-vlan)# shutdown

DLS2# show vlan brief

9. Configure the Rapid PVST (PVRST+) protocol on all switches. Ensure that DLS1 becomes the spanning tree

root of VLANs 10 and 20 and DLS2 becomes the backup. Ensure that DLS2 becomes the spanning tree root of

VLANs 30 and 99 and DLS1 becomes the backup.

ALS1(config)# spanning-tree mode rapid-pvst

ALS2(config)# spanning-tree mode rapid-pvst

DLS1(config)# spanning-tree mode rapid-pvst

DLS2(config)# spanning-tree mode rapid-pvst

/* Pentru a compara configuratia: DLS1# show spanning-tree */

DLS1(config)#spanning-tree vlan 10,20 root primary

DLS1(config)#spanning-tree vlan 30,99 root secondary

DLS2(config)#spanning-tree vlan 30,99 root primary

DLS2(config)#spanning-tree vlan 10,20 root secondary

10. On DLS1 and DLS2 configure SVIs and HSRP to provide gateway redundancy for access layer clients in

VLANs 10, 20, 30 and 99. Create an SVI in VLANs 10, 20, 30 and 99, each with an IP address and mask from

their respective networks chosen in Step 7.

DLS1(config)# interface vlan 10

DLS1(config-if)# ip address 192.168.10.3 255.255.255.0

DLS1(config-if)# no shutdown

DLS1(config)# interface vlan 20

DLS1(config-if)# ip address 192.168.20.3 255.255.255.0

DLS1(config-if)# no shutdown

DLS1(config)# interface vlan 30

DLS1(config-if)# ip address 192.168.30.3 255.255.255.0

DLS1(config-if)# no shutdown

DLS1(config)# interface vlan 99

DLS1(config-if)# ip address 192.168.99.3 255.255.255.0

DLS1(config-if)# no shutdown

/* Activam rutarea pentru a permite switch-ului sa actioneze ca layer 3 */

DLS1(config)# ip routing

DLS1(config)# sh ip route

/*Configuración interface DLS2*/

All contents are Copyright © 1992–2011 Cisco Systems, Inc. All rights reserved. This document is Cisco Public Information. 
Page 4 of 7



DLS2(config)# interface vlan 10

DLS2(config-if)# ip address 192.168.10.4 255.255.255.0

DLS2(config-if)# no shutdown

DLS2(config)# interface vlan 20

DLS2(config-if)# ip address 192.168.20.4 255.255.255.0

DLS2(config-if)# no shutdown

DLS2(config)# interface vlan 30

DLS2(config-if)# ip address 192.168.30.4 255.255.255.0

DLS2(config-if)# no shutdown

DLS2(config)# interface vlan 99

DLS2(config-if)# ip address 192.168.99.4 255.255.255.0

DLS2(config-if)# no shutdown

/* No se si es necesario activarlo también en el DLS2 */

DLS2(config)# ip routing

11. Configure DLS1 as the active HSRP router for VLANs 10 and 20 and configure DLS2 as backup. Configure

DLS2 as the active router for VLANs 30 and 99 and configure DLS1 as backup.

**** HSRP Configuration for DLS1 ****

DLS1(config)# ip routing

DLS1(config)# interface vlan 10

DLS1(config-if)# standby 1 ip 192.168.10.1

DLS1(config-if)# standby 1 preempt

DLS1(config-if)# standby 1 priority 150

DLS1(config-if)# exit

DLS1(config)# interface vlan 20

DLS1(config-if)# standby 1 ip 192.168.20.1

DLS1(config-if)# standby 1 preempt

DLS1(config-if)# standby 1 priority 150

DLS1(config-if)# exit

DLS1(config)# interface vlan 30

DLS1(config-if)# standby 1 ip 192.168.30.1

DLS1(config-if)# standby 1 preempt

DLS1(config-if)# standby 1 priority 100

DLS1(config-if)# exit

DLS1(config)# interface vlan 20

DLS1(config-if)# standby 1 ip 192.168.99.1

DLS1(config-if)# standby 1 preempt

DLS1(config-if)# standby 1 priority 100

DLS1(config-if)# exit

**** HSRP Configuration for DLS2 ****

DLS2(config)# ip routing

DLS2(config)# interface vlan 10

DLS2(config-if)# standby 1 ip 192.168.10.1

DLS2(config-if)# standby 1 preempt

DLS2(config-if)# standby 1 priority 100

DLS2(config-if)# exit

DLS2(config)# interface vlan 20

DLS2(config-if)# standby 1 ip 192.168.20.1

DLS2(config-if)# standby 1 preempt

DLS2(config-if)# standby 1 priority 100

DLS2(config-if)# exit

DLS2(config)# interface vlan 30

DLS2(config-if)# standby 1 ip 192.168.30.1

DLS2(config-if)# standby 1 preempt

DLS2(config-if)# standby 1 priority 150

DLS2(config-if)# exit

DLS2(config)# interface vlan 99

DLS2(config-if)# standby 1 ip 192.168.99.1

DLS2(config-if)# standby 1 preempt

DLS2(config-if)# standby 1 priority 150

DLS2(config-if)# exit

DLS1# show standby

DLS1# show standby brief

DLS2# show standby brief

/* Si quisiera verificar la configuración de HSRP */

DLS2(config)# interface range fastEthernet 0/7 - 12

DLS2(config-if-range)# shutdown

12. On ALS1 and ALS2 create an SVI for MGMT VLAN 99 with an IP address from the VLAN 99 network assigned

in Step 7.

All contents are Copyright © 1992–2011 Cisco Systems, Inc. All rights reserved. This document is Cisco Public Information. 
Page 5 of 7



ALS1(config)# interface vlan 99

ALS1(config-if)# ip address 192.168.99.5 255.255.255.0

ALS1(config-if)# no shutdown

ALS2(config)# interface vlan 99

ALS2(config-if)# ip address 192.168.99.6 255.255.255.0

ALS2(config-if)# no shutdown

13. For ALS1 and ALS2, specify the HSRP gateway address of VLAN 99 as the default gateway.

ALS1(config)# ip default-gateway 192.168.99.1

ALS2(config)# ip default-gateway 192.168.99.1

14. Enable PortFast on all access layer switch ports.

ALS1(config)# interface fa0/6

ALS1(config-if)# spanning-tree portfast default

ALS1(config-if)# no shutdown

ALS2(config)# interface fa0/6

ALS2(config-if)# spanning-tree portfast default

ALS2(config-if)# no shutdown

15. Permit the links between DLS2 and ALS2 to carry traffic only for the VLANs created in Step 7.

ALS2(config)# (config)#interface range fastEthernet 0/7 - 8

ALS2(config-if)#switchport trunk allowed vlan 10,20,30,99

DLS2(config)# (config)#interface range fastEthernet 0/7 ± 8

DLS2(config-if)#switchport trunk allowed vlan 10,20,30,99

16. Enable QoS globally on all switches.

DLS1#set qos enable

DLS2#set qos enable

ALS1#set qos enable

ALS2#set qos enable

ALS1(config)#mls qos

ALS2(config)#mls qos

DLS1(config)#mls qos

DLS2(config)#mls qos

17. On ALS1 configure Fa0/6 as an access port in CLIENT VLAN 10 and to trust Cisco IP phones CoS using

AutoQoS. Use VOICE VLAN 20 as the voice VLAN.

ALS1(config)# interface fastEthernet 0/6

ALS1(config-if)# switchport mode access

ALS1(config-if)# switchport access vlan 10

ALS1(config-if)# switchport voice vlan 20

ALS1(config-if)# mls qos trust device cisco-phone

ALS1(config-if)# auto qos voip cisco-phone

/* verify configuration */

ALS1# show mls qos interface fastEthernet 0/15

ALS1# show run interface fastEthernet 0/15

18. On ALS1, configure port Fa0/6 with port security. Allow up to two MAC addresses to be learned for IP phone

support. Enable sticky learning. Shut down the port if a violation occurs.

ALS1(config-if)# switchport port-security

ALS1(config-if)# switchport port-security maximum 2

ALS1(config-if)# switchport port-security mac-address sticky

ALS1(config-if)# switchport port-security violation shutdown

ALS1(config-if)# exit

19. On ALS2 configure port Fa0/6 as an access port in SERVER VLAN 30.

ALS2(config)# interface fastEthernet 0/6

ALS2(config-if)# switchport mode access

ALS2(config-if)# switchport access vlan 30

20. Configure IP routing on DLS1 and DLS2, and use EIGRP to advertise 192.168.0.0/16 with automatic

summarization disabled.

DLS1(config)#router EIGRP 1

DLS1(config-router)#no auto-summary

DLS1(config-router)#network 192.168.0.0

DLS2(config)#router EIGRP 1

DLS2(config-router)#no auto-summary

DLS2(config-router)#network 192.168.0.0

21. Configure client PC-A with an IP address in the VLAN 10 network and specify the VLAN 10 HSRP virtual

address as the default gateway. Configure server PC-B with an IP address in VLAN 30 and specify the VLAN 30

HSRP virtual address as the default gateway.



Part 2: Test network connectivity and configured options.

a. Create a Tcl script and test connectivity from each distribution layer switch to the addresses you assigned in the

topology.


Note: The IOS for the access layer switches used in this SBA does not support Tcl scripting.

b. Verify that the correct VLANs exist on all switches and contain the correct ports.

All contents are Copyright © 1992–2011 Cisco Systems, Inc. All rights reserved. This document is Cisco Public Information. 
Page 6 of 7



c. Verify that the EtherChannel between DLS1 and ALS1 is configured correctly.

d. Verify the spanning tree configuration and that the root bridge (DLS1 or DLS2) is correct for each VLAN.

e. Verify that the correct SVIs exist and that the correct HRSP routers are primary and standby for each VLAN.

f. Verify that Auto-QoS VoIP support is configured for port Fa0/6 on ALS1.

g. Verify that client PC-A can ping server PC-B.

h. Verify the traced route from client PC-A to server PC-B.







