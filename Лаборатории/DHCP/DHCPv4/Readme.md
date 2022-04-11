# DHCPv4 - Лабораторная работа №3
**Цели**

Часть 1: Построение сети и настройка основных параметров устройств

Часть 2: Настройка и проверка двух DHCPv4-серверов на R1

Часть 3: Настройка и проверка DHCP-ретранслятора на R2

# Часть 1
**Подключаем устройства по топологии**
![Topology](https://github.com/Farix01/Otus-kurs/blob/main/%D0%9B%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%B8%D0%B8/DHCP/DHCPv4/Topology.png)
***Заполним таблицу адресации***
| **Устройство**  | **Интерфейс**   | **IP-адрес**  | **Маска подсети**  |  **Шлюз по умолчанию** |
|---|---|---|---|---|
| R1  |  G0/0/0 | 10.0.0.1  | 255.255.255.252  |  N/A |
|   |  G0/0/1 |  N/A |  N/A |  N/A |
|   | G0/0/1.100  |  192.168.1.1 | 255.255.255.192  | N/A  |
|   | G0/0/1.200  |  192.168.1.65 |  255.255.255.224 |  N/A |
|   |  G0/0/1.1000 | N/A  | N/A  |  N/A |
|  R2 |  G0/0/0 | 10.0.0.2  |  255.255.255.252 |  N/A |
|     |  G0/0/1 | 192.168.1.97  | 255.255.255.240  |  N/A |
|  S1 |  VLAN 200 | 192.168.1.66  | 255.255.255.224  | 192.168.1.65  |
|  S2 |  VLAN 1 | 192.168.1.98  | 255.255.255.240   | 192.168.1.97  |
|  PC-A |  NIC | DHCP  | DHCP  | DHCP  |
| PC-B  |  NIC |  DHCP | DHCP  | DHCP  |

**Таблица VLAN**
| VLAN	| Name	| Interface Assigned |
| -----	| -----	| ------------------ |
| 1	| N/A	| S2: F0/18 |
| 100	| Clients	| S1: F0/6 |
| 200	| Management	| S1: VLAN 200  |
| 999	| Parking_Lot	| S1: F0/1-4, F0/7-24, G0/1-2 |
| 1000	| Native	| N/A |

**Создадим схемы адресации**

Subnet the network 192.168.1.0/24 to meet the following requirements:
a.	One subnet, “Subnet A”, supporting 58 hosts (the client VLAN at R1).
Subnet A:
Type your answers here.
Record the first IP address in the Addressing Table for R1 G0/0/1.100. 

b.	One subnet, “Subnet B”, supporting 28 hosts (the management VLAN at R1). 
Subnet B:
Type your answers here.
Record the first IP address in the Addressing Table for R1 G0/0/1.200. Record the second IP address in the Address Table for S1 VLAN 200 and enter the associated default gateway.

c.	One subnet, “Subnet C”, supporting 12 hosts (the client network at R2).
Subnet C:
Type your answers here.
Record the first IP address in the Addressing Table for R2 G0/0/1.

**Схема сети**

![Shema](https://github.com/Farix01/Otus-kurs/blob/main/%D0%9B%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%B8%D0%B8/DHCP/DHCPv4/Shema.png)

**Настраиваем основные параметры на маршрутизаторах**
```
hostname R1,R2
!
enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0
enable password 7 0822404F1A0A
no ip domain-lookup
!
banner motd "Attention!!! Access is denied!"
!
line con 0
 exec-timeout 0 0
 password 7 0822455D0A16
 logging synchronous
 login
!
line aux 0
 password 7 0822455D0A16
 logging synchronous
 login
!
line vty 0 4
 password 7 0822455D0A16
 logging synchronous
 login
line vty 5 15
 password 7 0822455D0A16
 logging synchronous
 login
 !
 clock set 12:00:00 11 APRIL 2022
```
**Настройка маршрутизации на R1**
```
interface GigabitEthernet0/0/0
 ip address 10.0.0.1 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/0/1
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/0/1.100
 description Clients
 encapsulation dot1Q 100
 ip address 192.168.1.1 255.255.255.192
!
interface GigabitEthernet0/0/1.200
 description Management
 encapsulation dot1Q 200
 ip address 192.168.1.65 255.255.255.224
!
interface GigabitEthernet0/0/1.1000
 description Native
 encapsulation dot1Q 1000 native
 no ip address
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
ip route 0.0.0.0 0.0.0.0 10.0.0.2 
```
**Настройка g0/0/1 на R2, затем G0/0/0 и статической маршрутизации**
```
interface GigabitEthernet0/0/0
 ip address 10.0.0.2 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/0/1
 ip address 192.168.1.97 255.255.255.240
 duplex auto
 speed auto
!
ip classless
ip route 0.0.0.0 0.0.0.0 10.0.0.1 
!
```
**Проверяем статический маршрут между R1 и R2**
```
R1#ping 10.0.0.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```
**Настроим основные параметры для каждого коммутатора**
```
hostname S1,S2
!
enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0
enable password 7 0822404F1A0A
!
!
!
no ip domain-lookup
!
banner motd "Attention!!! Access is denied!!!"
!
!
!
line con 0
 password 7 0822455D0A16
 logging synchronous
 login
 exec-timeout 0 0
!
line vty 0 4
 exec-timeout 0 0
 password 7 0822455D0A16
 logging synchronous
 login
line vty 5 15
 exec-timeout 0 0
 password 7 0822455D0A16
 logging synchronous
 login
 !
 clock set 12:00:00 11 APRIL 2022
```
**Создадим Vlan и назначим правильным интерфейсам на S1**
```
!
!
interface FastEthernet0/5
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 100,200,1000
 switchport mode trunk
!
interface FastEthernet0/1
 switchport access vlan 999
 switchport mode access
 shutdown
!
!
interface FastEthernet0/6
 switchport access vlan 100
 switchport mode access
!
!
interface Vlan1
 no ip address
 shutdown
!
interface Vlan200
 ip address 192.168.1.66 255.255.255.224
!
interface Vlan999
 no ip address
!
interface Vlan1000
 no ip address
!
ip default-gateway 192.168.1.65
!
S1#show vlan
1    default                          active    
100  Clients                          active    Fa0/6
200  Managemetr                       active    
999  Parking_Lot                      active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
```
**Создадим Vlan и шлюз по умолчанию на S2**
```
!
interface Vlan1
 ip address 192.168.1.98 255.255.255.240
!
ip default-gateway 192.168.1.97
!
```
# Часть 2
**Настроим и проверим два DHCP сервера на R1**
```
ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp pool R1_Client
network 192.168.1.0 255.255.255.192
default-router 192.168.1.97 192.168.1.101
domain-name ccna-lab.com
lease 7 12 30
exit
ip dhcp excluded-address 192.168.1.97
ip dhcp pool R2_Client_LAN 
network 192.168.1.96 255.255.255.240
default-router 192.168.1.97
domain-name ccna-lab.com
lease 7 12 30
```
