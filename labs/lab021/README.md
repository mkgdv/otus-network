# Настройка HSRP

### Построение сети и проверка связи

Создадим сеть согласно заданной топологии. Топология сети:

![topology](topology.jpg)

Произведём базовую настройку всех узлов. Начнём с настройки маршрутизатора R1:

```
Router>enable
Router#configure t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R1
R1(config)#no ip domain lookup
R1(config)#interface GigabitEthernet 0/1
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown

%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
R1(config-if)#exit
R1(config)#interface serial 0/0/0
R1(config-if)#ip address 10.1.1.1 255.255.255.252
R1(config-if)#clock rate 128000
R1(config-if)#exit
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#logging synchronous
R1(config-line)#exit
R1(config)#line vty 0 15
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#

%SYS-5-CONFIG_I: Configured from console by console

R1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```

Настройка R2:

```
Router>enable
Router#configure t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#no ip domain lookup
Router(config)#hostname R2
R2(config)#interface serial 0/0/0
R2(config-if)#ip address 10.1.1.2 255.255.255.252
R2(config-if)#no shutdown

%LINK-5-CHANGED: Interface Serial0/0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Serial0/0/0, changed state to up

R2(config-if)#exit
R2(config)#interface serial 0/0/1
R2(config-if)#ip address 10.2.2.2 255.255.255.252
R2(config-if)#clock rate 128000
R2(config-if)#no shutdown

%LINK-5-CHANGED: Interface Serial0/0/1, changed state to down

R2(config-if)#exit
R2(config)#interface Lo1

%LINK-5-CHANGED: Interface Loopback1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback1, changed state to up

R2(config-if)#ip address 209.165.200.225 255.255.255.224
R2(config-if)#exit
R2(config)#enable secret class
R2(config)#line console 0
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#logging synchronous
R2(config-line)#exit
R2(config)#line vty 0 15
R2(config-line)#password cisco
R2(config-line)#login
R2#
%SYS-5-CONFIG_I: Configured from console by console

R2#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
```

Настройка маршрутизатора R3:

```
Router>enable
Router#configure t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R3
R3(config)#no ip domain lookup
R3(config)#interface serial 0/0/1
R3(config-if)#ip address 10.2.2.1 255.255.255.252
R3(config-if)#no shutdown

%LINK-5-CHANGED: Interface Serial0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Serial0/0/1, changed state to up
R3(config)#interface Gi0/1
R3(config-if)#ip address 192.168.1.3 255.255.255.0
R3(config-if)#no shutdown

%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up

R3(config)#enable secret class
R3(config)#line console 0
R3(config-line)#password cisco
R3(config-line)#login
R3(config-line)#logging synchronous
R3(config-line)#exit
R3(config)#line vty 0 15
R3(config-line)#password cisco
R3(config-line)#login

%SYS-5-CONFIG_I: Configured from console by console

R3#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
```

Переходим к базовой настройке коммутаторов. Начнём с S1:

```
S1>enable
S1#configure t
S1(config)#no ip domain lookup
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#loggin synchronous 
S1(config-line)#exit
S1(config)#line vty 0 15
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.11 255.255.255.0
S1(config-if)#no shutdown

%LINK-5-CHANGED: Interface Vlan1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up

S1(config-if)#exit
S1(config)#ip default-gateway 192.168.1.1
S1(config)#exit

%SYS-5-CONFIG_I: Configured from console by console

S1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```

Настройка S3:

```
S3>enable
S3#confi
S3#configure t
Enter configuration commands, one per line.  End with CNTL/Z.
S3(config)#no ip domain lookup
S3(config)#enable secret class
S3(config)#line console 0
S3(config-line)#password cisco
S3(config-line)#login
S3(config-line)#logging synchronous
S3(config-line)#exit
S3(config)#line vty 0 15
S3(config-line)#password cisco
S3(config-line)#login
S3(config-line)#exit
S3(config)#interface vlan 1
S3(config-if)#ip address 192.168.1.13 255.255.255.0
S3(config-if)#no shutdown

%LINK-5-CHANGED: Interface Vlan1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up

S3(config-if)#exit
S3(config)#ip default-gateway 192.168.1.1
S3(config)#exit

%SYS-5-CONFIG_I: Configured from console by console

S3#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
```

Базовая настройка оборудования произведена. Проверим подключение между компьютерами, выполним ping с PC-A на PC-C:

```
C:\>ping 192.168.1.33

Pinging 192.168.1.33 with 32 bytes of data:

Reply from 192.168.1.33: bytes=32 time=1ms TTL=128
Reply from 192.168.1.33: bytes=32 time<1ms TTL=128
Reply from 192.168.1.33: bytes=32 time<1ms TTL=128
Reply from 192.168.1.33: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.1.33:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

Настроим протокол RIP на маршрутизаторах. Начнём с R1:

```
R1#configure
Configuring from terminal, memory, or network [terminal]?
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#router rip
R1(config-router)#version 2
R1(config-router)#no auto-summary
R1(config-router)#do sh ip route con

 C   10.1.1.0/30  is directly connected, Serial0/0/0
 C   192.168.1.0/24  is directly connected, GigabitEthernet0/1

R1(config-router)#network 10.0.0.0
R1(config-router)#network 192.168.1.0

%SYS-5-CONFIG_I: Configured from console by console

R1#copy running-config startup-config
```

Далее настраиваем на R2:

```
R2#configure t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#router rip
R2(config-router)#version 2
R2(config-router)#no auto-summary
R2(config-router)#do sh ip route conn

 C   10.1.1.0/30  is directly connected, Serial0/0/0
 C   10.2.2.0/30  is directly connected, Serial0/0/1
 C   209.165.200.224/27  is directly connected, Loopback1

R2(config-router)#network 10.0.0.0
```

Настройка R3:

Настройка обеспечения избыточности на первом хопе с помощью HSRP

```
R3#configure t
Enter configuration commands, one per line.  End with CNTL/Z.
R3(config)#router rip
R3(config-router)#version 2
R3(config-router)#no auto-summary 
R3(config-router)#do sh ip route conn

 C   10.2.2.0/30  is directly connected, Serial0/0/1
 C   192.168.1.0/24  is directly connected, GigabitEthernet0/1

R3(config-router)#network 10.0.0.0
R3(config-router)#network 192.168.1.0
```

Далее настроим маршрут по умолчанию на маршрутизаторе R2 с использованием Lo1 в качестве интерфейса выхода в сеть 209.165.200.224/27:

```
R2(config)#ip route 0.0.0.0 0.0.0.0 loopback 1
%Default route without gateway, if not a point-to-point interface, may impact performance
```
На R2 используем следующие команды для перераспределения маршрута по умолчанию в процесс RIP: 

```
R2(config)#router rip
R2(config-router)#default-information originate
```

Проверим подключение. Отправим с PC-A echo-запросы на все сконфигурированые интерфейсы:

```
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time=1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

C:\>ping 10.1.1.1

Pinging 10.1.1.1 with 32 bytes of data:

Reply from 10.1.1.1: bytes=32 time<1ms TTL=255
Reply from 10.1.1.1: bytes=32 time<1ms TTL=255
Reply from 10.1.1.1: bytes=32 time=2ms TTL=255
Reply from 10.1.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 10.1.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 2ms, Average = 0ms

C:\>ping 10.1.1.2

Pinging 10.1.1.2 with 32 bytes of data:

Reply from 10.1.1.2: bytes=32 time=1ms TTL=254
Request timed out.
Reply from 10.1.1.2: bytes=32 time=1ms TTL=254
Reply from 10.1.1.2: bytes=32 time=1ms TTL=254

Ping statistics for 10.1.1.2:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 1ms, Average = 1ms

C:\>ping 10.2.2.2

Pinging 10.2.2.2 with 32 bytes of data:

Reply from 10.2.2.2: bytes=32 time=1ms TTL=254
Reply from 10.2.2.2: bytes=32 time=1ms TTL=254
Reply from 10.2.2.2: bytes=32 time=1ms TTL=254
Reply from 10.2.2.2: bytes=32 time=1ms TTL=254

Ping statistics for 10.2.2.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 1ms, Average = 1ms

C:\>ping 192.168.1.3

Pinging 192.168.1.3 with 32 bytes of data:

Reply from 192.168.1.3: bytes=32 time<1ms TTL=255
Reply from 192.168.1.3: bytes=32 time<1ms TTL=255
Reply from 192.168.1.3: bytes=32 time<1ms TTL=255
Reply from 192.168.1.3: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 10.2.2.1

Pinging 10.2.2.1 with 32 bytes of data:

Reply from 10.2.2.1: bytes=32 time=1ms TTL=255
Reply from 10.2.2.1: bytes=32 time=1ms TTL=255
Reply from 10.2.2.1: bytes=32 time=1ms TTL=255
Reply from 10.2.2.1: bytes=32 time<1ms TTL=255

Ping statistics for 10.2.2.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

C:\>ping 209.165.200.225

Pinging 209.165.200.225 with 32 bytes of data:

Reply from 209.165.200.225: bytes=32 time=2ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254

Ping statistics for 209.165.200.225:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 2ms, Average = 1ms
```

Как видим во всех случаях ответы приходят. Сделаем такую же проверку с PC-C:

```
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 10.1.1.1

Pinging 10.1.1.1 with 32 bytes of data:

Reply from 10.1.1.1: bytes=32 time=2ms TTL=255
Reply from 10.1.1.1: bytes=32 time<1ms TTL=255
Reply from 10.1.1.1: bytes=32 time=1ms TTL=255
Reply from 10.1.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 10.1.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 2ms, Average = 0ms

C:\>ping 10.1.1.2

Pinging 10.1.1.2 with 32 bytes of data:

Reply from 10.1.1.2: bytes=32 time=1ms TTL=254
Reply from 10.1.1.2: bytes=32 time=1ms TTL=254
Reply from 10.1.1.2: bytes=32 time=1ms TTL=254
Reply from 10.1.1.2: bytes=32 time=1ms TTL=254

Ping statistics for 10.1.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 1ms, Average = 1ms

C:\>ping 10.2.2.2

Pinging 10.2.2.2 with 32 bytes of data:

Reply from 10.2.2.2: bytes=32 time=1ms TTL=254
Reply from 10.2.2.2: bytes=32 time=6ms TTL=254
Reply from 10.2.2.2: bytes=32 time=1ms TTL=254
Reply from 10.2.2.2: bytes=32 time=10ms TTL=254

Ping statistics for 10.2.2.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 10ms, Average = 4ms

C:\>ping 192.168.1.3

Pinging 192.168.1.3 with 32 bytes of data:

Reply from 192.168.1.3: bytes=32 time<1ms TTL=255
Reply from 192.168.1.3: bytes=32 time=5ms TTL=255
Reply from 192.168.1.3: bytes=32 time<1ms TTL=255
Reply from 192.168.1.3: bytes=32 time=1ms TTL=255

Ping statistics for 192.168.1.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 5ms, Average = 1ms

C:\>ping 10.2.2.1

Pinging 10.2.2.1 with 32 bytes of data:

Reply from 10.2.2.1: bytes=32 time=1ms TTL=255
Reply from 10.2.2.1: bytes=32 time<1ms TTL=255
Reply from 10.2.2.1: bytes=32 time<1ms TTL=255
Reply from 10.2.2.1: bytes=32 time<1ms TTL=255

Ping statistics for 10.2.2.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

C:\>ping 209.165.200.225

Pinging 209.165.200.225 with 32 bytes of data:

Reply from 209.165.200.225: bytes=32 time=2ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254

Ping statistics for 209.165.200.225:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 2ms, Average = 1ms

C:\>ping 192.168.1.11

Pinging 192.168.1.11 with 32 bytes of data:

Reply from 192.168.1.11: bytes=32 time<1ms TTL=255
Reply from 192.168.1.11: bytes=32 time=1ms TTL=255
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.11:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

C:\>ping 192.168.1.13

Pinging 192.168.1.13 with 32 bytes of data:

Reply from 192.168.1.13: bytes=32 time<1ms TTL=255
Reply from 192.168.1.13: bytes=32 time=4ms TTL=255
Reply from 192.168.1.13: bytes=32 time<1ms TTL=255
Reply from 192.168.1.13: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.13:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 4ms, Average = 1ms
```

Как видим все устройства отвечают на запросы.

### Настройка обеспечения избыточности на первом хопе с помощью HSRP

Определим путь интернет-трафика для PC-A и PC-C. Проведём трассировку с PC-A до нашего loopback-адреса 209.165.200.225 на маршрутизаторе R2:

```
C:\>tracert 209.165.200.225

Tracing route to 209.165.200.225 over a maximum of 30 hops: 

  1   1 ms      0 ms      0 ms      192.168.1.1
  2   0 ms      2 ms      0 ms      209.165.200.225

Trace complete.
```

Видим что пакеты прошли через маршрутизатор R1 и далее на loopback-интерфейс вышестоящего маршрутизатора R2.

Проведём такую же проверку с PC-C до loopback-адреса 209.165.200.225:

```
C:\>tracert 209.165.200.225

Tracing route to 209.165.200.225 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.1.3
  2   0 ms      1 ms      1 ms      209.165.200.225

Trace complete.
```

Теперь чтобы достигнуть loopback-интерфейса на R2 пакеты прошли через маршрутизатор R3.

Запустим сеанс эхо-тестирования на PC-A и разорвём соединение между S1 и R1.

```
C:\>ping -t 209.165.200.225

Pinging 209.165.200.225 with 32 bytes of data:

Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=6ms TTL=254
Reply from 209.165.200.225: bytes=32 time=2ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
...
Request timed out.
Request timed out.

Ping statistics for 209.165.200.225:
    Packets: Sent = 37, Received = 34, Lost = 3 (9% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 12ms, Average = 1ms
```

Как видим трафик между этими двумя интерфейсами перестал проходить. Восстановим соединение между S1 и R1.
При повторении данных шагов на комьютере PC-C и коммутаторе S3 мы можем наблюдать такую же картину:

```
C:\>ping -t 209.165.200.225

Pinging 209.165.200.225 with 32 bytes of data:

Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=5ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 209.165.200.225:
    Packets: Sent = 8, Received = 4, Lost = 4 (50% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 5ms, Average = 2ms
```

Восстановим соединение и проверим прохождение пакетов с обоих ПК ещё раз.
PC-A:

```
C:\>ping -t 209.165.200.225

Pinging 209.165.200.225 with 32 bytes of data:

Reply from 209.165.200.225: bytes=32 time=2ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254

Ping statistics for 209.165.200.225:
    Packets: Sent = 7, Received = 7, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 2ms, Average = 1ms
```

PC-C:

```
C:\>ping -t 209.165.200.225

Pinging 209.165.200.225 with 32 bytes of data:

Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254

Ping statistics for 209.165.200.225:
    Packets: Sent = 6, Received = 6, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 1ms, Average = 1ms
```

### Настроим HSRP на R1 и R3

Сначала настроим протокол HSRP на маршрутизаторе R1:

```
R1(config)#interface G0/1
R1(config-if)#standby version 2
R1(config-if)#standby 1 ip 192.168.1.254
R1(config-if)#

%HSRP-6-STATECHANGE: GigabitEthernet0/1 Grp 1 state Init -> Init

R1(config-if)#standby 1 priority 150
R1(config-if)#standby 1 preempt

%HSRP-6-STATECHANGE: GigabitEthernet0/1 Grp 1 state Speak -> Standby
%HSRP-6-STATECHANGE: GigabitEthernet0/1 Grp 1 state Standby -> Active
```
Настроим HSRP на маршрутизаторе R3

```
R3(config)#interface g0/1
R3(config-if)#standby version 2
R3(config-if)#standby 1 ip 192.168.1.254

%HSRP-6-STATECHANGE: GigabitEthernet0/1 Grp 1 state Init -> Init
%HSRP-6-STATECHANGE: GigabitEthernet0/1 Grp 1 state Speak -> Standby

```

Проверим работу HSRP на R1 и R3:

```
R1#show standby
GigabitEthernet0/1 - Group 1 (version 2)
  State is Active
    7 state changes, last state change 21:00:36
  Virtual IP address is 192.168.1.254
  Active virtual MAC address is 0000.0C9F.F001
    Local virtual MAC address is 0000.0C9F.F001 (v2 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.285 secs
  Preemption enabled
  Active router is local
  Standby router is 192.168.1.3
  Priority 150 (configured 150)
  Group name is hsrp-Gig0/1-1 (default)

R3#show standby 
GigabitEthernet0/1 - Group 1 (version 2)
  State is Standby
    5 state changes, last state change 20:41:44
  Virtual IP address is 192.168.1.254
  Active virtual MAC address is 0000.0C9F.F001
    Local virtual MAC address is 0000.0C9F.F001 (v2 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 2.478 secs
  Preemption disabled
  Active router is 192.168.1.1
  Standby router is local
  Priority 100 (default 100)
  Group name is hsrp-Gig0/1-1 (default)
```
Исходя из выходных данных, видим что HSRP поднят, активным является маршрутизатор R1, MAC-адрес виртуального маршрутизатора 0000.0C9F.F001. Для резервного маршрутизатора используется IP-адрес: 192.168.1.3 и выбран приоритет 100.

Посмотрим сводку состоянии HSRP на R1 и R3:

```
R1#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/1      1    150 P Active   local           192.168.1.3     192.168.1.254

R3#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/1      1    100   Standby  192.168.1.1     local           192.168.1.254 

```

Изменим адрес шлюза по умолчанию для PC-A, PC-C, S1 и S3. Используем IP-адрес нашего виртуального роутера.
Проверим новые настройки. Отправим эхо-запрос с PC-A и с PC-C на loopback-адрес маршрутизатора R2. PC-A:

```
C:\>ping -t 209.165.200.225

Pinging 209.165.200.225 with 32 bytes of data:

Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=3ms TTL=254

Ping statistics for 209.165.200.225:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 3ms, Average = 1ms

Control-C
^C
C:\>tracert 209.165.200.225

Tracing route to 209.165.200.225 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.1.1
  2   4294967295 ms5 ms      1 ms      209.165.200.225

Trace complete.
```

PC-C:

```
C:\>ping -t 209.165.200.225

Pinging 209.165.200.225 with 32 bytes of data:

Reply from 209.165.200.225: bytes=32 time=2ms TTL=254
Reply from 209.165.200.225: bytes=32 time=3ms TTL=254
Reply from 209.165.200.225: bytes=32 time=3ms TTL=254
Reply from 209.165.200.225: bytes=32 time=2ms TTL=254

Ping statistics for 209.165.200.225:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 2ms, Maximum = 3ms, Average = 2ms

C:\>tracert 209.165.200.225

Tracing route to 209.165.200.225 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.1.1
  2   0 ms      0 ms      0 ms      209.165.200.225

Trace complete.

```

Как видим все эхо-запросы успешно выполняются и пакеты идут через установленный активным в HSRP маршрутизатор.

Запустим сеанс эхо-тестирования на PC-A и разорвём соединение с коммутатором, подключенным к активному маршрутизатору HSRP (R1).

```
C:\>ping -t 209.165.200.225

Pinging 209.165.200.225 with 32 bytes of data:

Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=6ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=5ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Request timed out.
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254

Ping statistics for 209.165.200.225:
    Packets: Sent = 13, Received = 12, Lost = 1 (8% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 6ms, Average = 1ms
```

Как видим, после разрыва соединения между S1 и R1 связь на какое-то время пропала (один эхо-запрос остался без ответа). Но благодаря настроенному протоколу HSRP трафик был направлен через резервный маршрутизатор и сеть продолжила работу.

Проверим настройки HSRP на маршрутизаторах R1 и R3

```
R1#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/1      1    150 P Init     unknown         unknown         192.168.1.254 

R3#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/1      1    100   Active   local           unknown         192.168.1.254
```

Активным теперь является маршрутизатор R3.
Восстановим соединение между S1 и R1 и ещё раз посмотрим вывод команды

```
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/1      1    150 P Listen   unknown         unknown         192.168.1.254
R1#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/1      1    150 P Speak    unknown         unknown         192.168.1.254

%HSRP-6-STATECHANGE: GigabitEthernet0/1 Grp 1 state Speak -> Standby
%HSRP-6-STATECHANGE: GigabitEthernet0/1 Grp 1 state Standby -> Active

R1#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/1      1    150 P Active   local           unknown         192.168.1.254
```

Видим что после восстановления соединения маршрутизатор R1 вновь стал активным, т. к. мы принудительно настроили ему в HSRP приоритет 150, тогда как у R3 стоит значение по умолчанию (100).

#### Изменение приоритетов HSRP

Изменим приоритет HSRP на 200 на маршрутизаторе R3. 

```
R3(config)#interface g0/1
R3(config-if)#standby 1 priority 200
```
Видим что активным роутером по-прежнему является R1. Используем команду standby preempt для изменения активного роутера.

```
R3(config-if)#standby 1 preempt

 %HSRP-6-STATECHANGE: GigabitEthernet0/1 Grp 1 state Standby -> Active

```
Видим что теперь активным роутером является R3:

```
R3#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/1      1    200 P Active   local           192.168.1.1     192.168.1.254
```

### Вопросы для повторения
Для чего в локальной сети может потребоваться избыточность?

Избыточность используется для резервирования узлов в сети и для продолжения функционирования сети после выхода из строя каких-либо ключевых хостов.

Лабораторная работа выполнена с использованием Cisco Paket Tracer 7.3.0

1. ["Настройка HSRP"](2.1.hsrp).
