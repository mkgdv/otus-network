# Поиск и устранение неполадок в работе EtherChannel

### Построение сети и загрузка настроек устройств

Построим сеть согласно заданной топологии. Настроим ПК и коммутаторы согласно условиям.

![topology](topology.jpg)	

### Поиск и устранение неисправностей в работе маршрутизатора S1

Используем команду show interfaces trunk, чтобы убедиться что агрегированные каналы работают как транковые порты.

``
S1#show interfaces trunk

Port        Mode         Encapsulation  Status        Native vlan
Po2         on           802.1q         trunking      99

Port        Vlans allowed on trunk
Po2         1-1005

Port        Vlans allowed and active in management domain
Po2         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Po2         1,10,99
``
Как мы видим аггрегированные порты Po2 (fa0/1, fa0/2) отображаются как транковые.
Используемоманду show etherchannel summary, чтобы убедиться в том, что интерфейсы входят в состав соответствующего агрегированного канала, применен правильный протокол и интерфейсы задействованы.
``
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

1      Po1(SD)           LACP   Fa0/1(I) Fa0/2(I) 
2      Po2(SU)           PAgP   Fa0/3(P) Fa0/4(P) 
``
Видим что на Port-channel 1 (Po1) настроен протокол LACP, также видим что порты имеюют флаг (I), что означает что они не объединены в Port-channel. И есть флаг D что означает что интерфейс не задействован (Down)

Используем команду show run | begin interface Port-channel для просмотра текущей конфигурации, начиная с первого интерфейса агрегированного канала.
``
S1#show run | begin interface Port-channel 
interface Port-channel1
 switchport trunk native vlan 99
 switchport mode trunk
!
interface Port-channel2
 switchport trunk native vlan 99
 switchport mode access
!
interface FastEthernet0/1
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode active
!
interface FastEthernet0/2
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode active
!
interface FastEthernet0/3
 switchport trunk native vlan 99
 switchport mode access
 channel-group 2 mode desirable
!
interface FastEthernet0/4
 switchport trunk native vlan 99
 switchport mode access
 channel-group 2 mode desirable
``

Устраним ошибки, найденные в выходных данных из предыдущих команд show. Команды, используемые для исправления конфигураций:
``
S1(config)#interface range f0/1-2
S1(config-if-range)#no channel-group 1 mode active
S1(config-if-range)#channel-group 1 mode desirable
S1(config-if-range)#exit
S1(config)#interface port-channel 2
S1(config-if)#switchport mode trunk
``

Используем команду show interfaces trunk для проверки настроек транковой связи.
``
S1#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Po1         on           802.1q         trunking      99
Po2         on           802.1q         trunking      99

Port        Vlans allowed on trunk
Po1         1-1005
Po2         1-1005

Port        Vlans allowed and active in management domain
Po1         1,10,99
Po2         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Po1         1,10,99
Po2         1,10,99
``

Используем команду show etherchannel summary, чтобы убедиться в том, что агрегированные каналы работают и задействованы.
``
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

1      Po1(SU)           LACP   Fa0/1(P) Fa0/2(P) 
2      Po2(SU)           PAgP   Fa0/3(P) Fa0/4(P) 
``
Видим что аггрегированные каналы работают и задействованы.

### Поиск и устранение неполадок в работе маршрутизатора S2

Выполним команду для того, чтобы убедиться, что агрегированные каналы работают в качестве транковых портов. Ниже запишите команду, которую вы использовали.
``
S2#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Po1         on           802.1q         trunking      99

Port        Vlans allowed on trunk
Po1         1,99

Port        Vlans allowed and active in management domain
Po1         1,99

Port        Vlans in spanning tree forwarding state and not pruned
Po1         1,99
``
Видим что работает только Port-channel 1. Также видим что на данном интерфейсе не разрешён 10 Vlan.

Выполним команду, чтобы убедиться в том, что интерфейсы настроены в правильном агрегированном канале и настроен соответствующий протокол.
``
S2# show etherchannel summary
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

1      Po1(SU)           PAgP   Fa0/1(P) Fa0/2(P) 
3      Po3(SD)           PAgP   Fa0/3(D) Fa0/4(D) 
``
Видим что протокол на обоих Port-channel настроен PAgP, но порты Fa0/3 и Fa0/4 в состоянии Down, то есть незадействованы.

Используем команду show run | begin interface Port-channel для просмотра текущей конфигурации, начиная с первого интерфейса канала порта.
``
S2#show run | begin interface Port-channel 
interface Port-channel1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,99
 switchport mode trunk
!
interface Port-channel3
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,99
 switchport mode trunk
!
interface FastEthernet0/1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,99
 switchport mode trunk
 channel-group 1 mode desirable
!
interface FastEthernet0/2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,99
 switchport mode trunk
 channel-group 1 mode desirable
!
interface FastEthernet0/3
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,99
 switchport mode trunk
 channel-group 3 mode desirable
 shutdown
!
interface FastEthernet0/4
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,99
 switchport mode trunk
 channel-group 3 mode desirable
 shutdown
``
Приступим к устранению ошибок, найденные в выходных данных из предыдущих команд show. Команды, используемые для исправления конфигураций:
``
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface range fa0/3-4
S2(config-if-range)#no shutdown

%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to down

%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to down
S2(config-if-range)#interface port-channel 1
S2(config-if)#switchport trunk allowed vlan 1,10,99
``
Проверим параметры транковой связи
``
S2#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Po1         on           802.1q         trunking      99

Port        Vlans allowed on trunk
Po1         1,10,99

Port        Vlans allowed and active in management domain
Po1         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Po1         1,10,99
``

Проверим правильное функционирование агрегированных каналов
``
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

1      Po1(SU)           PAgP   Fa0/1(P) Fa0/2(P) 
3      Po3(SD)           PAgP   Fa0/3(D) Fa0/4(D) 
``

### Поиск и устранение неполадок в работе маршрутизатора S3

Убедимся, что агрегированные каналы работают в качестве транковых портов.
``
S3#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Po3         on           802.1q         trunking      99

Port        Vlans allowed on trunk
Po3         1-1005

Port        Vlans allowed and active in management domain
Po3         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Po3         1,10,99
``
Не выводится Po2.

Убедимся в том, что интерфейсы настроены в правильном агрегированном канале и применен соответствующий протокол.
``
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

3      Po3(SU)           PAgP   Fa0/3(P) Fa0/4(P)
``
В соответствии с нашей топологией Po3 должен включать в себя Fa0/1 и Fa0/2.

Используем команду show run | begin interface Port-channel для просмотра текущей конфигурации
``
S3#show run | begin interface Port-channel
interface Port-channel3
 switchport trunk native vlan 99
 switchport mode trunk
!
interface FastEthernet0/1
 shutdown
!
interface FastEthernet0/2
 shutdown
!
interface FastEthernet0/3
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 3 mode desirable
!
interface FastEthernet0/4
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 3 mode desirable
!
``
Видим что интерфейсы FastEthernet0/1 и FastEthernet0/2 не настроены. Также не задан и не скофигурирован Port-channel 2

Устраним все обнаруженные неполадки. Команды, использованные для исправления конфигурации:
``
S3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S3(config)#interface range fa0/3-4
S3(config-if-range)#channel-group 2 mode desirable 
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up

%LINK-3-UPDOWN: Interface Port-channel3, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel3, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up

%LINK-5-CHANGED: Interface Port-channel2, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel2, changed state to up

S3(config-if-range)#interface range fa0/1-2
S3(config-if-range)#switchport mode trunk
S3(config-if-range)#switchport trunk native vlan 99
S3(config-if-range)#channel-group 3 mode desirable
S3(config-if-range)#no shutdown
%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up

%LINK-5-CHANGED: Interface Port-channel3, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel3, changed state to up
``
Выполним проверку параметров транковой связи
``
S3#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Po2         on           802.1q         trunking      99
Po3         on           802.1q         trunking      99

Port        Vlans allowed on trunk
Po2         1-1005
Po3         1-1005

Port        Vlans allowed and active in management domain
Po2         1,10,99
Po3         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Po2         none
Po3         1,10,99
``
Выполним проверку правильного функционирования агрегированных каналов
``
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

2      Po2(SU)           PAgP   Fa0/3(P) Fa0/4(P) 
3      Po3(SU)           PAgP   Fa0/1(P) Fa0/2(P) 
``
Видим что агрегированные каналы настроены и работают.

### Проверка EtherChannel и подключения

Используем команду show interfaces etherchannel для проверки работоспособности агрегированных каналов.
``

S1>show interfaces etherchannel 

FastEthernet0/1:
Port state    = 1
Channel group = 1           Mode = Desirable-S1    Gcchange = 0
Port-channel  = Po1         GC   = 0x00000000      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/1     d     U1/S1   H30s    1        0        128        Any      0

Partner's information:
          Partner              Partner          Partner         Partner Group
Port      Name                 Device ID        Port       Age  Flags   Cap.
Fa0/1     S1                   0001.427C.DA39   Fa0/1      0    SC     10001    

Age of the port in the current state:  00d:00h:42m:43s

----
FastEthernet0/2:
Port state    = 1
Channel group = 1           Mode = Desirable-S1    Gcchange = 0
Port-channel  = Po1         GC   = 0x00000000      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/2     d     U1/S1   H30s    1        0        128        Any      0

Partner's information:
          Partner              Partner          Partner         Partner Group
Port      Name                 Device ID        Port       Age  Flags   Cap.
Fa0/2     S1                   0001.427C.DA39   Fa0/2      0    SC     10001    

Age of the port in the current state:  00d:00h:42m:43s

----
FastEthernet0/3:
Port state    = 1
Channel group = 2           Mode = Desirable-S1    Gcchange = 0
Port-channel  = Po2         GC   = 0x00000000      Pseudo port-channel = Po2
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/3     d     U1/S1   H30s    1        0        128        Any      0

Partner's information:
          Partner              Partner          Partner         Partner Group
Port      Name                 Device ID        Port       Age  Flags   Cap.
Fa0/3     S1                   0060.3EE6.1320   Fa0/3      0    SC     10001    

Age of the port in the current state:  00d:00h:08m:30s

----
FastEthernet0/4:
Port state    = 1
Channel group = 2           Mode = Desirable-S1    Gcchange = 0
Port-channel  = Po2         GC   = 0x00000000      Pseudo port-channel = Po2
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/4     d     U1/S1   H30s    1        0        128        Any      0

Partner's information:
          Partner              Partner          Partner         Partner Group
Port      Name                 Device ID        Port       Age  Flags   Cap.
Fa0/4     S1                   0060.3EE6.1320   Fa0/4      0    SC     10001    

Age of the port in the current state:  00d:00h:08m:30s

----
Port-channel1:Port-channel1   (Primary aggregator)
Age of the Port-channel   = 00d:01h:02m:58s
Logical slot/port   = 2/1             Number of ports = 2
HotStandBy port = null
Port state          = 
Protocol            =   1
Port Security       = Disabled

Ports in the Port-channel:

Index   Load   Port     EC state        No of bits
------+------+------+------------------+-----------
  0     00     Fa0/1    Desirable-Sl       0
  0     00     Fa0/2    Desirable-Sl       0

Time since last port bundled:    00d:00h:42m:43s    Fa0/2

Port-channel2:Port-channel2
Age of the Port-channel   = 00d:01h:02m:58s
Logical slot/port   = 2/2             Number of ports = 2
GC                  = 0x00000000      HotStandBy port = null
Port state          = 
Protocol            =   2
Port Security       = Disabled

Ports in the Port-channel:

Index   Load   Port     EC state        No of bits
------+------+------+------------------+-----------
  0     00     Fa0/3    Desirable-Sl       0
  0     00     Fa0/4    Desirable-Sl       0
Time since last port bundled:    00d:00h:08m:30s    Fa0/4
``

Проверим подключение сети VLAN Management
Пинг от S1 до S2
``
S1>ping 192.168.1.12

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.12, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/2 ms
``
Пинг от S1 до S3
``
S1>ping 192.168.1.13

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.13, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
``
Пинг от S2 до S3
``
S2>ping 192.168.1.13

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.13, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
``
Проверим связность между PC-A и PC-C
``
C:\>ping 192.168.0.3

Pinging 192.168.0.3 with 32 bytes of data:

Reply from 192.168.0.3: bytes=32 time=1ms TTL=128
Reply from 192.168.0.3: bytes=32 time<1ms TTL=128
Reply from 192.168.0.3: bytes=32 time<1ms TTL=128
Reply from 192.168.0.3: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.0.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
``

Как видим все запросы успешно выполнены, значит мы выполнили поставленную задачу

Лабораторная работа выполнена с использованием Cisco Paket Tracer 7.3.0

1. ["Поиск и устранение неполадок в работе EtherChannel"](3.2.troubleshooting_etherchannel).