Tclsh dls_reset.tcl
Copy Base.CFG run

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
Channel-group 4 mode desirable
No shut

Int range g1/0/3-4
Shutdown
Channel-group 2 mode on
No shut

Int range g1/0/5-6
No switchport
Shutdown
Channel-group 10 mode active
No shut
exit

interface port-channel 10
no switchport
ip address 172.16.200.2 255.255.255.252
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

#unusedinterfaces
Int range g1/0/7-11
Sw mode acc
Sw acc vlan 999

Int range g1/0/13-14
Sw mode acc
Sw acc vlan 999

Int range g1/0/21-24
Sw mode acc
Sw acc vlan 999

Int range g1/1/1-4
Sw mode acc
Sw acc vlan 999

#DLS2-mst
spanning-tree mst configuration
name CISCO
revision 1
instance 1 vlan 10,20
instance 2 vlan 100,110
exit
spanning-tree mst 2 root primary
spanning-tree mst 1 root secondary

#DLS2-svi
Int vlan 100
Ip address 172.16.100.3 255.255.255.0

int vlan 10
Ip address 172.16.10.3 255.255.255.0

int vlan 20
Ip address 172.16.20.3 255.255.255.0

int vlan 110
Ip address 172.16.110.3 255.255.255.0

int vlan 150
ip address 172.16.150.3 255.255.255.0

#layer3port
Int g1/0/12
No switchport
Ip address 192.168.1.2 255.255.255.252

#DLS2-HSRP
ip routing

int vlan 10
standby 10 ip 172.16.10.1
standby 10 preempt
exit

int vlan 20
standby 20 ip 172.16.20.1
standby 20 preempt
exit

int vlan 100
standby 100 ip 172.16.100.1
standby 100 priority 110
standby 100 preempt
standby 100 track 200 decrement 30
exit

int vlan 110
standby 100 ip 172.16.110.1
standby 100 priority 110
standby 100 preempt
standby 100 track 200 decrement 30
exit

#serverfarmHSRP
int vlan 150
standby 150 ip 172.16.150.1
standby 150 priority 110
standby 150 preempt
standby 150 track 200 decrement 30
exit

#sla/tracking
ip sla 20
icmp-echo 192.168.1.1
frequency 5
exit
ip sla schedule 20 life forever start-time now
track 200 ip sla 20
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
 network 192.168.1.0 0.0.0.3
 #serverfarmrouting
 network 172.16.150.0 0.0.0.255
 no auto-summary


#DLS2-VLAN151-isolatedvlan
vlan 151
private-vlan isolated
vlan 152
private-vlan community
vlan 150
private-vlan primary
private-vlan association 151,152
exit

int vlan 150
private-vlan mapping 151-152
exit

#DLS2-isolated-Pvlaninterface
int range g1/0/15-17
sw mode private-vlan host
switchport private-vlan host-association 150 151
no shut

#DLS2-community-Pvlaninterface
int range g1/0/18-20
sw mode private-vlan host
switchport private-vlan host-association 150 152
no shut


``The switchport mode private-vlan host command sets the mode on the interface

``the switchport private-vlan host-association primary-vlan-id secondary-vlan-id command assigns the appropriate VLANs to the interface

[MicroNugget: Private VLANs (youtube.com)](https://www.youtube.com/watch?v=xl3_zgaZuH8)

#RACL
ip access-list extended ACLVLAN10
permit tcp 172.16.20.0 0.0.0.255 172.16.10.0 0.0.0.255 established
permit icmp 172.16.20.0 0.0.0.255 172.16.10.0 0.0.0.255 echo-reply
deny ip 172.16.20.0 0.0.0.255 172.16.10.0 0.0.0.255
permit ip any any

interface vlan 10
ip access-group ACLVLAN10 in
exit

#ntp
#configure
ntp server 172.16.100.2
clock summer-time AEDT recurring 1 Sun Oct 2:00 1 Sun Apr 3:00
clock timezone AEST 10 0
