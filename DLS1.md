Tclsh dls_reset.tcl
Copy Base.CFG run

#vlans 
Vlan 10
Name Staff
Vlan 20 
Name Students
Vlan 100
Name Managements
Vlan 110
Name Voice
Vlan 800
Name Native
Vlan 999
Name Parking
state suspend
exit
o8im,
#createvlan150
vlan 150
name Server_Farm

#trunks
Int range g1/0/1-6
Switchport trunk native vlan 800
Switchport mode trunk
Switchport nonegotiate
switchport trunk allowed vlan except 1,999
No shut

#portchannels 
Int range g1/0/1-2
Shutdown
Channel-group 1 mode desirable
No shut

Int range g1/0/3-4
Shutdown
Channel-group 3 mode on
No shut

Int range g1/0/5-6
No switchport
Shutdown
Channel-group 10 mode active
No shut
exit

interface port-channel 10
no switchport
ip address 172.16.200.1 255.255.255.252
no shut
exit

#vtp 
Vtp domain CISCO
Vtp version 3
Vtp mode server

//vtp primary vlan 
//in enable mode (DLS1)
//vtp mode transparent
//in config mode (ALL Switches)

#dhcpserver
Int range g1/0/7
Sw mode acc
Sw acc vlan 100
no shut

#unusedinterfaces
Int range g1/0/8-10
Sw mode acc
Sw acc vlan 999

Int range g1/0/12-24
Sw mode acc
Sw acc vlan 999

Int range g1/1/1-4
Sw mode acc
Sw acc vlan 999

#DLS1-mst
spanning-tree mst configuration
name CISCO
revision 1
instance 1 vlan 10,20
instance 2 vlan 100,110
exit
spanning-tree mst 1 root primary
spanning-tree mst 2 root secondary

#DLS1-svi
Int vlan 100
Ip address 172.16.100.2 255.255.255.0

int vlan 10
Ip address 172.16.10.2 255.255.255.0

int vlan 20
Ip address 172.16.20.2 255.255.255.0

int vlan 110
Ip address 172.16.110.2 255.255.255.0

int vlan 150
ip address 172.16.150.2 255.255.255.0

#layer3port
Int g1/0/11
No switchport
Ip address 192.168.0.2 255.255.255.252

#HSRP 
ip routing

int vlan 10
standby 10 ip 172.16.10.1
standby 10 priority 110
standby 10 preempt
standby 10 track 100 decrement 30
exit

int vlan 20
standby 20 ip 172.16.20.1
standby 20 priority 110
standby 20 preempt
standby 20 track 100 decrement 30


int vlan 100
standby 100 ip 172.16.100.1
standby 100 preempt
exit

int vlan 110
standby 100 ip 172.16.110.1
standby 100 preempt
exit

#serverfarmHSRP
int vlan 150
standby 150 ip 172.16.150.1
standby 150 preempt

#sla/tracking
ip sla 10
icmp-echo 192.168.0.1
frequency 5
exit
ip sla schedule 10 life forever start-time now
track 100 ip sla 10
exit

#portfast
spanning-tree portfast default

#bpduguard
spanning-tree portfast bpduguard default

#udld
udld aggressive
int range g1/0/1-24
udld port aggressive
int range g1/1/1-4
udld port aggressive
exit

#DLSwitches-dhcp-trust
ip dhcp relay information trust-all

#eigrp
router eigrp 100
 network 172.16.10.0 0.0.0.255
 network 172.16.20.0 0.0.0.255
 network 172.16.100.0 0.0.0.255
 network 172.16.110.0 0.0.0.255
 network 172.16.200.0 0.0.0.3
 network 192.168.0.0 0.0.0.3
 #serverfarmrouting
 network 172.16.150.0 0.0.0.255
 no auto-summary
 
//Suminda also did route to other DLS's L3 Etherchannel
//ip route 172.16.0.0 255.255.0.0 172.16.200.1
//route all internal traffic to always go through L3 Etherchannel

//Suminda added this default route to the gateway, but i think we can advertise the default route from gateway router to other switches
//#defaultroutetogateway
//ip route 0.0.0.0 0.0.0.0 192.168.1.1


------
#RACL-separate-Student-StaffVLANs 
//RACL that allows the staff VLAN (10) to access the student VLAN (20), and deny student VLAN access to the staff VLAN.

#DLS1
ip access-list extended ACLVLAN10
permit tcp 172.16.20.0 0.0.0.255 172.16.10.0 0.0.0.255 established
permit icmp 172.16.20.0 0.0.0.255 172.16.10.0 0.0.0.255 echo-reply
deny ip 172.16.20.0 0.0.0.255 172.16.10.0 0.0.0.255
permit ip any any

interface vlan 10
ip access-group 10 in
exit

//The keyword `established` ensures that only return traffic from established TCP sessions is allowed. 
// This means students cannot initiate TCP connections to the staff VLAN, but they can respond to connections initiated by the staff VLAN.
//echo reply means ping from The destination to source **(?)**
//so if block echo reply, it blocks ping from dest to source
//so above commands, allows ping from dest to source
//then blocks ping from source to dest

![[Pasted image 20240519220200.png]]

//Read show commands and testing procedures (especially for ACL and PVLAN) from lab doc or GPT

#ntp
#enable 
clock set 02:00:00 May 19 2024

#configure
clock summer-time AEDT recurring 1 Sun Oct 2:00 1 Sun Apr 3:00
clock timezone AEST 10 0
ntp master 10








