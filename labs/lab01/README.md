# Настройка расширенных сетей VLAN, VTP и DTP.

Таблица адресации

| Заголовок таблицы | Интерфейс | IP-адрес      | Маска подсети          |
|-------------------|-----------|---------------|------------------------|
| S1   | VLAN 99 | 192.168.99.1 | 255.255.255.0 |
| S2   | VLAN 99 | 192.168.99.2 | 255.255.255.0 |  
| S3   | VLAN 99 | 192.168.99.3 | 255.255.255.0 |
| PC-A | NIC | 192.168.10.1     | 255.255.255.0 |
| PC-B | NIC | 192.168.20.1     | 255.255.255.0 |
| PC-C | NIC | 192.168.10.2     | 255.255.255.0 |

Настроим S1 и S3 в качестве клиентов VTP в домене CCNA  
Проверим настроенные конфигурации VTP:
```
S1#show vtp status
VTP Version                     : 2
Configuration Revision          : 0
Maximum VLANs supported locally : 255
Number of existing VLANs        : 9
VTP Operating Mode              : Client
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP V2 Mode                     : Disabled
VTP Traps Generation            : Disabled
MD5 digest                      : 0x2C 0x37 0x65 0x0B 0x17 0x6F 0x80 0xED
Configuration last modified by 0.0.0.0 at 3-1-93 00:57:10

S2#show vtp status
VTP Version                     : 2
Configuration Revision          : 8
Maximum VLANs supported locally : 255
Number of existing VLANs        : 9
VTP Operating Mode              : Server
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP V2 Mode                     : Disabled
VTP Traps Generation            : Disabled
MD5 digest                      : 0x6A 0xAC 0x73 0x30 0x7D 0x27 0xCC 0x62 
Configuration last modified by 0.0.0.0 at 3-1-93 00:57:10
Local updater ID is 192.168.99.2 on interface Vl20 (lowest numbered VLAN interface found)

S3#show vtp status
VTP Version                     : 2
Configuration Revision          : 8
Maximum VLANs supported locally : 255
Number of existing VLANs        : 9
VTP Operating Mode              : Client
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP V2 Mode                     : Disabled
VTP Traps Generation            : Disabled
MD5 digest                      : 0x6A 0xAC 0x73 0x30 0x7D 0x27 0xCC 0x62
Configuration last modified by 0.0.0.0 at 3-1-93 00:57:10
```
### Настроим динамические магистральные каналы между S1 и S2
Проверим административный и оперативный режим у коммутационного порта f0/1 на S1 и S2

```
S1#show interfaces f0/1 switchport 
Name: Fa0/1
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
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

S2#show interfaces f0/1 switchport 
Name: Fa0/1
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
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
Проверим магистральный канал между коммутаторами S1 и S2
```
S1#show interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Fa0/1       desirable        802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/1       1-1005

Port        Vlans allowed and active in management domain
Fa0/1       1

S2#show interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Fa0/1       auto             802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/1       1-1005

Port        Vlans allowed and active in management domain
Fa0/1       1

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1


```

Настроим статический магистральный канал между S1 и S3 и снова проверим его командой show interfaces trunk
```
S1#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       desirable    n-802.1q       trunking      1
Fa0/3       on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/1       1-1005
Fa0/3       1-1005

Port        Vlans allowed and active in management domain
Fa0/1       1
Fa0/3       1

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       none
Fa0/3       none

S2#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       auto         n-802.1q       trunking      1
Fa0/3       on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/1       1-1005
Fa0/3       1-1005

Port        Vlans allowed and active in management domain
Fa0/1       1
Fa0/3       1

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       none
Fa0/3       none
```
### Добавление VLAN'ов и назначение портов

При попытке добавить VLAN на S1 мы получаем сообщение что VLAN можно добавлять только на сервере VTP домена.
Добавим на S2 VLAN'ы из условия:
```
S2(config)# vlan 10
S2(config-vlan)# name Red
S2(config-vlan)# vlan 20
S2(config-vlan)# name Blue
S2(config-vlan)# vlan 30
S2(config-vlan)# name Yellow
S2(config-vlan)# vlan 99
S2(config-vlan)# name Management
S2(config-vlan)# end
```
Проверяем:
```
S2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
10   Red                              active
20   Blue                             active    Fa0/18
30   Yellow                           active
99   Managment                        active
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
1005 trnet-default                    active

S1>show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/4, Fa0/5, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
10   Red                              active    Fa0/6
20   Blue                             active
30   Yellow                           active
99   Managment                        active
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
1005 trnet-default                    active

S3>show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/4, Fa0/5, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
10   Red                              active    Fa0/6, Fa0/18
20   Blue                             active
30   Yellow                           active
99   Managment                        active
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
1005 trnet-default                    active
```

### Назначение портов сетям VLAN и настройка IP-адресов на коммутаторах
На коммутаторе S1 переведем порт F0/6 в режим доступа и назначим его сети VLAN 10. Настроим IP-адрес на интерфейсе SVI для VLAN 99 на коммутаторах в соответствии с условиями. На коммутаторах S2 и S3 повторим данную процедуру для порта F0/18
```
S1(config)# interface f0/6  
S1(config-if)# switchport mode access  
S1(config-if)# switchport access vlan 10
S1(config-if)# exit
S1(config)# interface vlan 99
S1(config-if)# ip address 192.168.99.1 255.255.255.0
S1(config-fi)# no shutdown

S2(config)# interface f0/18
S2(config-if)# switchport mode access
S2(config-if)# switchport access vlan 20
S2(config-if)# exit
S2(config)# interface vlan 99
S2(config-if)# ip address 192.168.99.2 255.255.255.0
S2(config-fi)# no shutdown

S3(config)# interface f0/18
S3(config-if)# switchport mode access
S3(config-if)# switchport access vlan 10
S3(config-if)# exit
S3(config)# interface vlan 99
S3(config-if)# ip address 192.168.99.3 255.255.255.0
S3(config-fi)# no shutdown
```
Проверка наличия сквозного соединения
Отправим ping-запрос с PC-B на PC-A:
```
C:\>ping 192.168.10.1

Pinging 192.168.10.1 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.10.1:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```
Ответ на echo-запросы мы не получим, т. к. данные ПК находятся в разных VLAN и могут обмениваться трафиком только через L3.

Отправим ping-запрос с PC-A на PC-C
```
C:\>ping 192.168.10.2

Pinging 192.168.10.2 with 32 bytes of data:

Reply from 192.168.10.2: bytes=32 time=20ms TTL=128
Reply from 192.168.10.2: bytes=32 time=1ms TTL=128
Reply from 192.168.10.2: bytes=32 time=1ms TTL=128
Reply from 192.168.10.2: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.10.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 20ms, Average = 5ms
```
Мы получаем ответы на echo-запросы, т. к. оба ПК находятся в одном VLAN 10.

Пинг с коммутатора S1 на компьютер PC-A
```
S1>ping 192.168.10.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.10.1, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```
Ответ мы не получим, т. к. на интерфейсе к которому подключен ПК не настроен IP-адрес (Я считаю что поэтому. Ведь в echo-request должен быть Source IP чтобы понять куда отправлять echo-reply)

Отправка ping-запрос с коммутатора S2 на коммутатор S1
```
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.99.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
```
Проходит успешно, т. к. оба интерфейса находятся в VLAN 99

### Настройка сети VLAN расширенного диапазона
Переводим VTP коммутатора S1 в режим Transparent и проверям.
```
S1#show vtp status
VTP Version                     : 2
Configuration Revision          : 0
Maximum VLANs supported locally : 255
Number of existing VLANs        : 10
VTP Operating Mode              : Transparent
VTP Domain Name                 : CCNA
VTP Pruning Mode                : Disabled
VTP V2 Mode                     : Disabled
VTP Traps Generation            : Disabled
MD5 digest                      : 0x2C 0x37 0x65 0x0B 0x17 0x6F 0x80 0xED
Configuration last modified by 0.0.0.0 at 3-1-93 00:57:10
```

Добавим сеть VLAN из расширенного диапазона и проверим
```
S1# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/4, Fa0/5, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
10   Red                              active    Fa0/6
20   Blue                             active
30   Yellow                           active
99   Managment                        active
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
1005 trnet-default                    active
2000 VLAN2000                         active
```
Как видим VLAN успешно добавлен.

### Ответы на вопросы
Каковы преимущества и недостатки использования VTP?  
К преимуществам можно отности сегментирование сети на уровне ПО, большая степень административного контроля, уменьшение полосы пропускания, снижение затрат на оборудование. Снижение нагрузки на оборудование из-за сигментирования широковещательных штормов.  
К недостаткам можно отнести проприетарность технологии (необходмо оборудование поддерживающих технологию вендоров). При добавлении в сеть нового устройства всегда необходимо учитывать существующую конфигурацию VTP, чтобы случайно не затереть её (в следствии более высокой версии ревизии, коммутатор может стать сервером в домене VTP и затереть существующие в нём VLAN'ы). Ограниченность в 1005 VLAN'ов в основном диапазоне. Потребности корпоративных сетей могут выходить за этот предел, а для поддержки расширенного диапазона нужны будут более современные устройства.  

### Лабораторная работа выполнена с использованием Cisco Paket Tracer 7.3.0

1. [Лабораторная работа по теме "Масштабирование сетей VLAN"](1.vtp_dtp.pkt).

