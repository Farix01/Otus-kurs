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

