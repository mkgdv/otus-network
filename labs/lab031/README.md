# Настройка EtherChannel

![topology](topology.jpg)

### Настроим базовые параметры каждого коммутатора.

```
Switch(config)#no ip domain-lookup
Switch(config)#hostname S1
S1(config)#banner motd #Unauthorized access denied#
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#logging synchronous
S1(config)#exit
S1(config)#interface range f0/1 - f0/24
S1(config-if-range)#shutdown
S1(config-if-range)#interface range gi0/1 - gi0/2
S1(config-if-range)#shutdown
S1(config)#interface f0/6
S1(config-if)#no shutdown
S1(config-if)#exit
S1(config)#vlan 99
S1(config-vlan)#name Management
S1(config-vlan)#vlan 10
S1(config-vlan)#name Staff
S1(config-vlan)#exit
S1(config)#interface range f0/1-f0/4
S1(config-if-range)#switchport mode access
S1(config-if-range)#switchport access vlan 10
S1(config-if-range)#exit
S1(config)#interface vlan 99
S1(config-if)#ip address 192.168.99.11 255.255.255.0
S1(config-if)#exit
S1(config)#exit
S1#copy running-config startup-config

Switch(config)#no ip domain-lookup
Switch(config)#hostname S2
S2(config)#banner motd #Unauthorized access denied#
S2(config)#enable secret class
S2(config)#line console 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#logging synchronous
S2(config)#exit
S2(config)#interface range f0/1 - f0/24
S2(config-if-range)#shutdown
S2(config-if-range)#interface range gi0/1 - gi0/2
S2(config-if-range)#shutdown
S2(config)#interface f0/18
S2(config-if)#no shutdown
S2(config-if)#exit
S2(config)#vlan 99
S2(config-vlan)#name Management
S2(config-vlan)#vlan 10
S2(config-vlan)#name Staff
S2(config-vlan)#exit
S2(config)#interface range f0/1-f0/4
S2(config-if-range)#switchport mode access
S2(config-if-range)#switchport access vlan 10
S2(config-if-range)#exit
S2(config)#interface vlan 99
S2(config-if)#ip address 192.168.99.12 255.255.255.0
S2(config-if)#exit
S2(config)#exit
S2#copy running-config startup-config
```

### Настройка протокола PAgP

Настроим PAgP на S1 и S3
```
S1(config)#interface range f0/3-4
S1(config-if-range)#channel-group 1 mode desirable
Creating a port-channel interface Port-channel 1
S1(config-if-range)#no shutdown
%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to down
%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to down

S3(config)#interface range f0/3-f0/4
S3(config-if-range)#channel-group 1 mode auto
Creating a port-channel interface Port-channel 1
S3(config-if-range)#no shutdown
%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up
%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up
%LINK-5-CHANGED: Interface Port-channel1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
```

Проверим:

```
S1#show running-config
Building configuration...

Current configuration : 1892 bytes
!
version 12.2
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname S1
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
!
!
!
no ip domain-lookup
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface Port-channel1
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
 shutdown
!
interface FastEthernet0/2
 switchport access vlan 10
 switchport mode access
 shutdown
!
interface FastEthernet0/3
 switchport access vlan 10
 switchport mode access
 channel-group 1 mode desirable
!
interface FastEthernet0/4
 switchport access vlan 10
 switchport mode access
 channel-group 1 mode desirable
!
...


S1#show interfaces f0/3 switchport 
Name: Fa0/3
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (Staff)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk private VLANs: none
Operational private-vlan: none
Trunking VLANs Enabled: All
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL
Protected: false
Unknown unicast blocked: disabled
Unknown multicast blocked: disabled
Appliance trust: none

S3#show running-config
Building configuration...

Current configuration : 1882 bytes
!
version 12.2
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname S3
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
!
!
!
no ip domain-lookup
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface Port-channel1
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
 shutdown
!
interface FastEthernet0/2
 switchport access vlan 10
 switchport mode access
 shutdown
!
interface FastEthernet0/3
 switchport access vlan 10
 switchport mode access
 channel-group 1 mode auto
!
interface FastEthernet0/4
 switchport access vlan 10
 switchport mode access
 channel-group 1 mode auto
!
...

S3#show interfaces f0/3 switchport
Name: Fa0/3
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (Staff)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk private VLANs: none
Operational private-vlan: none
Trunking VLANs Enabled: All
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL
Protected: false
Unknown unicast blocked: disabled
Unknown multicast blocked: disabled
Appliance trust: none
```

Убедимся что порты объединены
```
S1#show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port


Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           PAgP   Fa0/3(P) Fa0/4(P)

S3#show etherchannel summary 
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port


Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           PAgP   Fa0/3(P) Fa0/4(P) 
```

Что означают флаги «SU» и «P» в сводных данных по Ethernet?
Как видно из описания флаг SU означает что аггрегация портов происходит на втором уровне, флаг P - что объеденены в Port-channel.

### Настроим транковые порты

```
S3(config)#interface port-channel 1
S3(config-if)#switchport mode trunk
S3(config-if)#switchport trunk native vlan 99
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/3 (99), with S1 FastEthernet0/3 (10).

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/4 (99), with S1 FastEthernet0/3 (10).

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/3 (99), with S1 Port-channel1 (10).

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/4 (99), with S1 Port-channel1 (10).

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/3 (99), with S1 FastEthernet0/4 (10).

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/4 (99), with S1 FastEthernet0/4 (10).

S1(config)#interface Port-channel 1
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/3 (10), with S3 FastEthernet0/3 (99).

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/4 (10), with S3 FastEthernet0/3 (99).

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/3 (10), with S3 FastEthernet0/4 (99).

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/4 (10), with S3 FastEthernet0/4 (99).

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/3 (10), with S3 Port-channel1 (99).

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/4 (10), with S3 Port-channel1 (99).

S1(config-if)#switchport mode trunk 
S1(config-if)#switchport trunk native vlan 99
```

Проверим какие комманды включены в список на интерфейсах F0/3 и F0/4
```
S1#show running-config 

...

!
interface Port-channel1
 switchport access vlan 10
 switchport trunk native vlan 99
 switchport mode trunk
!

...

interface FastEthernet0/3
 switchport access vlan 10
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode desirable
!
interface FastEthernet0/4
 switchport access vlan 10
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode desirable
!

S3#show running-config

...

!
interface Port-channel1
 switchport access vlan 10
 switchport trunk native vlan 99
 switchport mode trunk
!

...

!
interface FastEthernet0/3
 switchport access vlan 10
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode auto
!
interface FastEthernet0/4
 switchport access vlan 10
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode auto
!
```
b.

```
S1# show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Po1         on           802.1q         trunking      99

Port        Vlans allowed on trunk
Po1         1-1005

Port        Vlans allowed and active in management domain
Po1         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Po1         none

S1#show spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0090.2120.90AD
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.2120.90AD
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/6            Desg FWD 19        128.6    P2p

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    32778
             Address     0090.2120.90AD
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     0090.2120.90AD
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Po1              Desg BKN*9         128.28   Shr *TYPE_Inc

S3#show interfaces trunk 
Port        Mode         Encapsulation  Status        Native vlan
Po1         on           802.1q         trunking      99

Port        Vlans allowed on trunk
Po1         1-1005

Port        Vlans allowed and active in management domain
Po1         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Po1         10

S3#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0090.21CA.D19C
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.21CA.D19C
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/18           Desg FWD 19        128.18   P2p

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    32778
             Address     0090.2120.90AD
             Cost        9
             Port        28(Port-channel1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     0090.21CA.D19C
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Po1              Root FWD 9         128.28   Shr
```

# Настройка протокола LACP

### Настроим LACP между S1 и S2

```
S1(config)#interface range f0/1-2
S1(config-if-range)#switchport mode trunk
S1(config-if-range)#switchport trunk native vlan 99
S1(config-if-range)#channel-group 2 mode active
S1(config-if-range)#

Creating a port-channel interface Port-channel 2

S1(config-if-range)#no shutdown

%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to down

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to down


S2(config)#interface range f0/1-2
S2(config-if-range)#switchport mode trunk
S2(config-if-range)#switchport trunk native vlan 99
S2(config-if-range)#channel-group 2 mode passive
Creating a port-channel interface Port-channel 2

S2(config-if-range)#no shutdown

%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan99, changed state to up

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up

%LINK-5-CHANGED: Interface Port-channel2, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel2, changed state to up
```

Убедимся что порты объединены

```
S1#show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           PAgP   Fa0/3(P) Fa0/4(P)
2      Po2(SU)           LACP   Fa0/1(P) Fa0/2(P)

S2#show etherchannel summary 
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port


Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

2      Po2(SU)           LACP   Fa0/1(P) Fa0/2(P) 
```

Видим что для агрегирования используется протокол LACP. Агрегированными портами на обоих свичах являются Fa0/1 и Fa0/2

### Настроим LACP между S2 и S3

```
S2(config)#interface range f0/3-4
S2(config-if-range)#switchport mode trunk
S2(config-if-range)#switchport trunk native vlan 99
S2(config-if-range)#channel-group 3 mode active
S2(config-if-range)#
Creating a port-channel interface Port-channel 3

S2(config-if-range)#no shutdown

%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to down

%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to down

S3(config)#interface range f0/1-2
S3(config-if-range)#switchport mode trunk
S3(config-if-range)#switchport trunk native vlan 99
S3(config-if-range)#channel-group 3 mode passive
S3(config-if-range)#
Creating a port-channel interface Port-channel 3

S3(config-if-range)#no shutdown

%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan99, changed state to up

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up

%LINK-5-CHANGED: Interface Port-channel3, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel3, changed state to up
```

Убедимся что канал EtherChannel образован

```
S2#show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

2      Po2(SU)           LACP   Fa0/1(P) Fa0/2(P) 
3      Po3(SU)           LACP   Fa0/3(P) Fa0/4(P) 


S3#show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           PAgP   Fa0/3(P) Fa0/4(P)
3      Po3(SU)           LACP   Fa0/1(P) Fa0/2(P)
```

### Проверим наличие сквозного соединения

Проверим эхо-запросами доступность устройств

```
S1>ping 192.168.99.12

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.99.12, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms

S1>ping 192.168.99.13

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.99.13, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms

S2>ping 192.168.99.11

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.99.11, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms

S2>ping 192.168.99.13

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.99.13, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms

S3>ping 192.168.99.11

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.99.11, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms

S3>ping 192.168.99.12

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.99.12, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```
Как мы могли убедится все устройства отвечают на запросы друг друга.

### Вопросы для повторения

В EtherChannel нельзя объединить порты с разной пропускной способностью.

Лабораторная работа выполнена с использованием Cisco Paket Tracer 7.3.0
1. ["Настройка EtherChannel"](3.etherchannel.pkt).
