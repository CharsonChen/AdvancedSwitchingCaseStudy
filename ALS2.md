Tclsh als_reset.tcl
Copy Base.CFG run

#trunks
Int range f0/1-6
Switchport trunk native vlan 800
Switchport mode trunk
Switchport nonegotiate
sw trunk allowed vlan except 1,999
No shut

#portchannels 
Int range f0/1-2
Shutdown
Channel-group 4 mode desirable
No shut

Int range f0/3-4
Shutdown
Channel-group 3 mode on
No shut

#vtp 
Vtp domain CISCO
Vtp version 3
Vtp mode server

#accessport
int range f0/13-24
switchport mode access
switchport access vlan 10

int range f0/7-12
switchport mode acc
switchport access vlan 20

#unusedports 
int range g0/1-2
switchport mode acc
switchport access vlan 999

#ALS2-MST
spanning-tree mst configuration
name CISCO
revision 1
instance 1 vlan 10,20
instance 2 vlan 100,110
exit

#ALS2-svi
Int vlan 100
Ip address 172.16.100.102 255.255.255.0

#portfast
spanning-tree portfast default

#bpduguard
spanning-tree portfast bpduguard default

#udld
udld aggressive
int range f0/1-24
udld port aggressive
int range g0/1-2
udld port aggressive
exit

#ALS2-port-security
interface Fa0/18
switchport mode access
switchport port-security
switchport port-security mac-address 1234.1234.1234
switchport port-security violation protect
exit

#ALS2-dhcp-snooping
ip dhcp snooping
int port-channel 3
ip dhcp snooping trust
ip dhcp snooping limit rate 15
exit

int port-channel 4
ip dhcp snooping trust
ip dhcp snooping limit rate 15
exit

#ntp
#configure
ntp server 172.16.100.2
clock summer-time AEDT recurring 1 Sun Oct 2:00 1 Sun Apr 3:00
clock timezone AEST 10 0
