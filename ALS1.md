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
Channel-group 1 mode desirable
No shut

Int range f0/3-4
Shutdown
Channel-group 2 mode on
No shut

#vtp 
Vtp domain CISCO
Vtp version 3
Vtp mode server

//vtp primary vlan 
//in enable mode (DLS1)
//vtp mode transparent
//in config mode (ALL Switches)

#accessport
int range f0/7-12
switchport mode access
switchport access vlan 10

int range f0/13-24
switchport mode acc
switchport access vlan 20

#unusedports 
int range g0/1-2
switchport mode acc
switchport access vlan 999

#ALS1-MST
spanning-tree mst configuration
name CISCO
revision 1
instance 1 vlan 10,20
instance 2 vlan 100,110
exit

#ALS1-SVI
Int vlan 100
Ip address 172.16.100.101 255.255.255.0

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

#ALS1-port-security
interface range Fa0/13 - 24
switchport mode access
switchport port-security
switchport port-security maximum 3
switchport port-security violation shutdown
exit

#ALS1-ip-dhcp-snooping
ip dhcp snooping
int port-channel 1
ip dhcp snooping trust
ip dhcp snooping limit rate 15
exit

int port-channel 2
ip dhcp snooping trust
 ip dhcp snooping limit rate 15
exit


//conf t
//ip dhcp snooping vlan 10,20
//Configure the VLANs that will use DHCP snooping. In this scenario, DHCP snooping will be used on both the student and staff VLANs
//idk if case study need or not, note Suminda have the command

#ALS1-VACL-temporarystaff
#traffic-match-acl
-

vlan access-map block-temp 10
match ip address temp-host
action drop
vlan access-map block-temp 20
action forward
exit

vlan filter block-temp vlan-list 10

int f0/9
sw mode acc
sw acc vlan 100

//change the host on f0/9 to use 
//ip = 172.16.10.150
//dg = 172.16.10.2 

#ntp 
#configure
ntp server 172.16.100.2
clock summer-time AEDT recurring 1 Sun Oct 2:00 1 Sun Apr 3:00
clock timezone AEST 10 0


#dhcpsnooping-notes
```
Turn snooping on globally using the ip dhcp snooping command.  


Configure the trusted interfaces with the ip dhcp snooping trust command. By default, all ports are considered untrusted unless statically configured to be trusted. 

* Very Important * : 
* The topology used for this lab is not using EtherChannels. 

* Remember that when an EtherChannel created, the virtual port channel interface is used by the switch to pass traffic; the physical interfaces (and importantly their configuration) is not referenced by the switch. 

* Therefore, if this  
topology was using EtherChannels, the ip dhcp snooping trust command would need to be applied to the Port Channel interfaces and not to the physical interfaces that make up the  
bundle.  

Configure a DHCP request rate limit on the user access ports to limit the number of DHCP 

requests that are allowed per second. This is configured using the ip dhcp snooping limit rate rate_in_pps. 

This command prevents DHCP starvation attacks by limiting the rate of the DHCP requests on untrusted ports.  
```

#vacl-notes
```
VACL Case Study Scenario
Assign temp staff to the Staff VLAN and create a VACL to block access to the rest of the Staff VLAN, and still have access to the rest of the network (DG of staff subnet to connect to the rest of the network and the ISP).

for traffic-match-acl
must be explicit about what traffic to match -- this isn't a traffic filtering ACL, it is a traffic matching ACL. 
If you were to leave the second line of the example below out, pings would work.

VACL Configuration
vlan access-map block-temp 10
//defines vacl name and then the rule(seq#)
match ip address temp-host
action drop
//rule has 10 so it will be higher than the next rule that is 20
//will drop all matched ip to the temp host access list
vlan access-map block-temp 20
action forward
//will allow all other traffic, cause there is implicit deny
exit

vlan filter block-temp vlan-list 10
//define which VLAN the access map is applied to

```
 







