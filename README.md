# Office-Lab
This Lab contains a simple topology for a small business office. Has proper subnetting, VLANs, DHCP, DNS, MSTP, OSPF, QoS, and Security features enagled


Lab Set-up:
sw mode trunk
sw trunk native vlan 1000
sw trunk allowed vlan 10,20,30,99
sw nonegotiate 

vlan 10
	name PCs
vlan 20
	name Phones
vlan 40
	name Wi-Fi
vlan 99
	name Management

sw nonegotiate
sw mode acc 
sw acc vlan 10
sw voice vlan 20

int range g1/0/6-24, g1/1/3-4
shut
do w

int range f0/3-24
shut
do w

0000.0C07.AC04

DSW-A1
int vlan 99
	ip add 10.0.0.2 255.255.255.240
	st v 2
	st 1 prio 105
	stan 1 pre
	stan 1 ip 10.0.0.1

access-list 1 permit 10.1.0.0 0.0.0.255 


int vlan 10
	ip add 10.1.0.2 255.255.255.0
	st v 2
	st 2 prio 105
	st 2 pre
	st 2 ip 10.1.0.1

int vlan 20
	ip add 10.2.0.2 255.255.255.0
	st v 2
	st 3 ip 10.2.0.1

int vlan 40
	ip add 10.6.0.2 255.255.255.0
	st v 2
	st 4 ip 10.6.0.1

DSW-A2
int vlan 99
	ip add 10.0.0.3 255.255.255.240
	stan ver 2
	no stan 1 ip 10.0.0.1

int vlan 10
	ip add 10.1.0.3 255.255.255.0
	st v 2
	no st 2 ip 10.1.0.1

int vlan 20
	ip add 10.2.0.3 255.255.255.0
	st v 2
	st 3 prio 105
	st 3 pre
	no st 3 ip 10.2.0.1

int vlan 40
	ip add 10.6.0.3 255.255.255.0
	st v 2
	st 4 prio 105
	st 4 pre
	no st 4 ip 10.6.0.1

DSW-B1
int vlan 99
	ip add 10.0.0.18 255.255.255.240
	st v 2
	st 1 prio 105
	stan 1 pre
	stan 1 ip 10.0.0.17

int vlan 10
	ip add 10.3.0.2 255.255.255.0
	st v 2
	st 2 prio 105
	st 2 pre
	st 2 ip 10.3.0.1

int vlan 20
	ip add 10.4.0.2 255.255.255.0
	st v 2
	st 3 ip 10.4.0.1

int vlan 30
	ip add 10.5.0.2 255.255.255.0
	st v 2
	st 4 ip 10.5.0.1

DSW-B2
int vlan 99
	ip add 10.0.0.19 255.255.255.240
	stan ver 2
	no stan 1 ip 10.0.0.17

int vlan 10
	ip add 10.3.0.3 255.255.255.0
	st v 2
	no st 2 ip 10.3.0.1

int vlan 20
	ip add 10.4.0.3 255.255.255.0
	st v 2
	st 3 prio 105
	st 3 pre
	no st 3 ip 10.4.0.1

int vlan 30
	ip add 10.5.0.3 255.255.255.0
	st v 2
	st 4 prio 105
	st 4 pre
	no st 4 ip 10.5.0.1

int f0/1
	spanning-tree bpduguard enable
	spanning-tree  portfast

ip dhcp pool A-PC
	network 10.1.0.0 255.255.255.0
	default-router 10.1.0.1
	domain-name officelab.com
	dns-server 10.5.0.4
	

ip dhcp pool A-PHONE
	network 10.2.0.0 255.255.255.0
	default-router 10.2.0.1
	domain-name officelab.com
	dns-server 10.5.0.4

ip dhcp pool  B-Mgmt
	network 10.0.0.16 255.255.255.240
	default-router 10.0.0.17
	domain-name officelab.com
	dns-server 10.5.0.4
	option 43  ip 10.0.0.7

ip dhcp pool  B-PC
	network 10.3.0.0 255.255.255.0
	default-router 10.2.0.1
	domain-name officelab.com
	dns-server 10.5.0.4

ip dhcp pool  B-Phone
	network 10.4.0.0 255.255.255.0
	default-router 10.4.0.1
	domain-name officelab.com
	dns-server 10.5.0.4

ip dhcp pool  Wi-Fi
	network 10.6.0.0 255.255.255.0
	default-router 10.6.0.1
	domain-name officelab.com
	dns-server 10.5.0.4

int vlan 10 
	ip helper 10.0.0.76
int vlan 20 
	ip helper 10.0.0.76
int vlan 30 
	ip helper 10.0.0.76
int vlan 99 
	ip helper 10.0.0.76

ip domain-name officelab.com
ip name-server 10.5.0.4

ntp authentication-key 1 md5 ccna
ntp trusted 1
ntp server 10.0.0.76 key 1
logging 10.5.0.4
loggin trap debugging 
logging buffered 8192

access-list 1 permit 10.1.0.0 0.0.0.255 
ip domain-name officelab.com
crypto key generate rsa 
4096
ip ssh version 2 
line vty 0 15
	transport input ssh
	logging synchronous
	login local
	access 1 in

no cdp run
lldp run
int f0/1
	no lldp trans 

ip acc exten OfficeA_to_OfficeB
	permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
	deny ip 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
	permit ip any any
int vlan 10
	ip acc OfficeA_to_OfficeB in

int f0/1
	sw port
	sw port max 2
	sw port viol res
	sw port mac sticky

ip dhcp snooping
ip dhcp snooping vlan 10,20,40,99 
no ip dhcp snooping infor option
int range g0/1-2
	no ip dhcp snooping limit rate 15
	ip dhcp snooping trust
int f0/1
	ip dhcp snooping limit rate 15

ip arp inspection validate dst ip src 
ip arp inspection vlan 10,20,30,99
int range g0/1-2
	ip arp inspection trust 
