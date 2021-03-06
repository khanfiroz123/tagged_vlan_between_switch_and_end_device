# tagged_vlan_between_switch_and_end_device
![Image vlan](https://github.com/NileshChandekar/tagged_vlan_between_switch_and_end_device/blob/master/vlan.png)


# Router configuration :- 

## First interface 0/0 configuration 
    ~~~
    configure terminal
    interface FastEthernet 0/0
    ip address 192.168.122.2 255.255.255.0
    no shutdown
    exit
    exit
    wr
    ~~~

## Configure a default gateway:

    ~~~
    configure terminal
    ip route 0.0.0.0 0.0.0.0 192.168.122.1
    end
    wr
    ~~~


## Ping the router’s default gateway:


    ~~~
    ping 192.168.122.1

    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 192.168.122.1, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 4/12/16 ms



    ping 8.8.8.8

    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 16/28/48 ms
    ~~~



## Ensure that the router is configured to use the correct DNS server:

    ~~~
    configure terminal
    ip domain-lookup
    ip name-server 8.8.8.8
    end
    wr
    ~~~



## Second interface 1/0 configuration 


    ~~~
    conf t 
    interface fastEthernet 1/0
    ip add 10.10.10.1 255.255.255.0
    no shut     
    exit
    exit
    wr
    ~~~


## Interface details 

    ~~~
    sh ip int br
    Interface                  IP-Address      OK? Method Status                Protocol
    FastEthernet0/0            192.168.122.2   YES manual up                    up      
    FastEthernet1/0            10.10.10.1      YES manual up                    up      
    ~~~


## Configure NAT  - for internet access


    ~~~
    conf t 
    interface fastEthernet 0/0
    ip nat outside 
    exit  
    interface fastEthernet 1/0
    ip nat inside 
    exit
    ip nat inside source list 1 interface fastEthernet 0/0 overload
    access-list 1 permit 10.0.0.0 0.255.255.255
    end
    wr
    ~~~


# Switch side configuration 


    ~~~
    sh ip int br
    ~~~

    ~~~
    Interface                  IP-Address      OK? Method Status                Protocol
    FastEthernet0/0            unassigned      YES NVRAM  administratively down down    
    FastEthernet1/0            unassigned      YES unset  up                    up      
    FastEthernet1/1            unassigned      YES unset  up                    up      
    FastEthernet1/2            unassigned      YES unset  up                    down    
    FastEthernet1/3            unassigned      YES unset  up                    down    
    FastEthernet1/4            unassigned      YES unset  up                    down    
    FastEthernet1/5            unassigned      YES unset  up                    up      
    FastEthernet1/6            unassigned      YES unset  up                    up      
    FastEthernet1/7            unassigned      YES unset  up                    up      
    FastEthernet1/8            unassigned      YES unset  up                    down    
    FastEthernet1/9            unassigned      YES unset  up                    down    
    FastEthernet1/10           unassigned      YES unset  up                    up      
    FastEthernet1/11           unassigned      YES unset  up                    up      
    FastEthernet1/12           unassigned      YES unset  up                    down    
    FastEthernet1/13           unassigned      YES unset  up                    down    
    FastEthernet1/14           unassigned      YES unset  up                    up      
    FastEthernet1/15           unassigned      YES unset  up                    up      
    Vlan1                      unassigned      YES NVRAM  administratively down down    
    ~~~



## vlan configuration

    ~~~
    vlan database 
    vlan 10 name ExternalNetworkVlanID
    vlan 20 name StorageNetworkVlanID
    vlan 30 name InternalApiNetworkVlanID
    vlan 40 name StorageMgmtNetworkVlanID
    vlan 50 name TenantNetworkVlanID
    exit
    wr
    ~~~


## Check details 


    ~~~
    ESW1#sh vlan-switch 

    VLAN Name                             Status    Ports
    ---- -------------------------------- --------- -------------------------------
    1    default                          active    Fa1/0, Fa1/1, Fa1/2, Fa1/3
                                                    Fa1/4, Fa1/5, Fa1/6, Fa1/7
                                                    Fa1/8, Fa1/9, Fa1/12, Fa1/13
                                                    Fa1/14, Fa1/15
    10   ExternalNetworkVlanID            active    
    20   StorageNetworkVlanID             active    
    30   InternalApiNetworkVlanID         active    
    40   StorageMgmtNetworkVlanID         active    
    50   TenantNetworkVlanID              active    
    1002 fddi-default                     active    
    1003 token-ring-default               active    
    1004 fddinet-default                  active    
    1005 trnet-default                    active    

    VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
    ---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
    1    enet  100001     1500  -      -      -        -    -        1002   1003
    10   enet  100010     1500  -      -      -        -    -        0      0   
    20   enet  100020     1500  -      -      -        -    -        0      0   
    30   enet  100030     1500  -      -      -        -    -        0      0   
              
    VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
    ---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
    40   enet  100040     1500  -      -      -        -    -        0      0   
    50   enet  100050     1500  -      -      -        -    -        0      0   
    1002 fddi  101002     1500  -      -      -        -    -        1      1003
    1003 tr    101003     1500  1005   0      -        -    srb      1      1002
    1004 fdnet 101004     1500  -      -      1        ibm  -        0      0   
    1005 trnet 101005     1500  -      -      1        ibm  -        0      0   
    ESW1#
    ~~~

## VTP CONFIGURATION


    ~~~
    vlan database 
    vtp server 
    vtp domain OPENSTACKLAB
    exit
    wr
    ~~~

## TRUNK PORT CONFIGURATION FOR VLAN RANGE 10-50 


    ~~~
    configure terminal 
    interface range fastEthernet 1/10 - 11
    switchport trunk encapsulation dot1q
    switchport mode trunk
    switchport trunk allowed vlan 1-4,10-50,1002-1005
    exit
    exit
    wr
    ~~~


## Trunking 

    ~~~
    conf t
    interface range fastEthernet 1/10 - 11
    switchport mode trunk
    spanning-tree portfast
    exit
    exit
    wr
    ~~~

	
## Check details 

    ~~~
    ESW1#sh interfaces trunk 

    Port      Mode         Encapsulation  Status        Native vlan
    Fa1/10    on           802.1q         trunking      1
    Fa1/11    on           802.1q         trunking      1

    Port      Vlans allowed on trunk
    Fa1/10    1-4,10-50,1002-1005
    Fa1/11    1-4,10-50,1002-1005

    Port      Vlans allowed and active in management domain
    Fa1/10    1,10,20,30,40,50
    Fa1/11    1,10,20,30,40,50

    Port      Vlans in spanning tree forwarding state and not pruned
    Fa1/10    1,10,20,30,40,50
    Fa1/11    1,10,20,30,40,50
    ESW1#
    ~~~

# End Device (redhat nodes)  Configuration 

## Check ip details after vlan configuration on rhel server (7.5)

    ~~~
    [root@test1020 network-scripts]# ls
    ifcfg-eth0     ifcfg-eth1.20  ifdown-bnep  ifdown-ipv6  ifdown-ppp     ifdown-Team      ifup          ifup-eth   ifup-isdn   ifup-post    ifup-sit       ifup-tunnel       network-functions
    ifcfg-eth1     ifcfg-lo       ifdown-eth   ifdown-isdn  ifdown-routes  ifdown-TeamPort  ifup-aliases  ifup-ippp  ifup-plip   ifup-ppp     ifup-Team      ifup-wireless     network-functions-ipv6
    ifcfg-eth1.10  ifdown         ifdown-ippp  ifdown-post  ifdown-sit     ifdown-tunnel    ifup-bnep     ifup-ipv6  ifup-plusb  ifup-routes  ifup-TeamPort  init.ipv6-global
    [root@test1020 network-scripts]# 
    ~~~

## Configure vlan interface ``eth1.10``
    ~~~
    [root@test1020 network-scripts]# cat ifcfg-eth1.10
    TYPE=vlan  <====================================================== Type should be "VLAN"
    BOOTPROTO=none
    NAME=eth1.10
    DEVICE=eth1.10
    ONBOOT=yes
    VLAN=yes
    IPADDR=10.1.1.2
    PREFIX=24
    NETWORK=10.1.1.0
    [root@test1020 network-scripts]#
    ~~~

## Configure vlan interface ``eth1.20``

    ~~~
    [root@test1020 network-scripts]# cat ifcfg-eth1.20
    TYPE=vlan   <====================================================== Type should be "VLAN"
    BOOTPROTO=none
    NAME=eth1.20
    DEVICE=eth1.20
    ONBOOT=yes
    VLAN=yes
    IPADDR=20.20.20.2
    PREFIX=24
    NETWORK=20.20.20.0
    [root@test1020 network-scripts]# 
    ~~~

## ``ip a`` details 

    ~~~
    [root@test1020 network-scripts]# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host 
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 08:00:27:f2:da:83 brd ff:ff:ff:ff:ff:ff
        inet 10.10.10.20/24 brd 10.10.10.255 scope global noprefixroute eth0
           valid_lft forever preferred_lft forever
        inet6 fe80::a00:27ff:fef2:da83/64 scope link 
           valid_lft forever preferred_lft forever
    3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 08:00:27:45:69:00 brd ff:ff:ff:ff:ff:ff
        inet6 fe80::a00:27ff:fe45:6900/64 scope link 
           valid_lft forever preferred_lft forever
    5: eth1.20@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
        link/ether 08:00:27:45:69:00 brd ff:ff:ff:ff:ff:ff
        inet 20.20.20.2/24 brd 20.20.20.255 scope global noprefixroute eth1.20
           valid_lft forever preferred_lft forever
        inet6 fe80::a00:27ff:fe45:6900/64 scope link 
           valid_lft forever preferred_lft forever
    6: eth1.10@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
        link/ether 08:00:27:45:69:00 brd ff:ff:ff:ff:ff:ff
        inet 10.1.1.2/24 brd 10.1.1.255 scope global noprefixroute eth1.10
           valid_lft forever preferred_lft forever
        inet6 fe80::a00:27ff:fe45:6900/64 scope link 
           valid_lft forever preferred_lft forever
    [root@test1020 network-scripts]# 
    ~~~

## Follow the above process for redhat node B as well. 


## IP and VLAN Configuration 

![Image ipa](https://github.com/NileshChandekar/tagged_vlan_between_switch_and_end_device/blob/master/ipa.png)

## Ping results 

![Image ping](https://github.com/NileshChandekar/tagged_vlan_between_switch_and_end_device/blob/master/ping.png)

## TCPDUMP from Server B (20.20.20.2)  to Server A (20.20.20.1) to verify 802.1 encapsulation (tagged vlan)

![Image tcpdump](https://github.com/NileshChandekar/tagged_vlan_between_switch_and_end_device/blob/master/tcpdump.png)
