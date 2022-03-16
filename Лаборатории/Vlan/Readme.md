**Vlan-Лабораторная работа №1**

**Задание**

- Построение сети и настройка основных параметров устройства
- Создание виртуальных локальных сетей и назначение портов коммутатора
- Настройка магистрали 802.1Q между коммутаторами
- Настройка маршрутизации между VLAN на маршрутизаторе
- Проверка работы маршрутизации между VLAN


**1. Построение сети и настройка основных параметров на устройствах**

Подключаем устройства, как показано на схеме топологии.
![Topologiya](https://github.com/Farix01/Otus-kurs/blob/main/%D0%9B%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%B8%D0%B8/Vlan/Topologiya.png)

Настроим основные параметры для маршрутизатора:

- Настройка имени
- Установка паролей для привелигированного режима, консоли и VTY
- Отключение поиска DNS
- Создание баннера
 
Конфиг:
```
hostname R1

!

enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0

enable password 7 0822404F1A0A

no ip domain-lookup

!

banner motd "Attention!!! Unauthorized access is prohibited!!!"

line con 0

` `password 7 0822455D0A16

` `login

!

line vty 0 4

` `password 7 0822455D0A16

` `login

line vty 5 15

` `password 7 0822455D0A16

` `login

!
```

Настроим основные параметры для коммутатора:

- Настройка имени
- Установка паролей для привелигированного режима, консоли и VTY
- Отключение поиска DNS
- Создание баннера
 
Конфиг:
```
hostname S2

!

enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0

enable password 7 0822455D0A16

no ip domain-lookup

!

banner motd "Attention!!! Unauthorized access is prohibited!!!"

line con 0

` `password 7 0822455D0A16

` `login

!

line vty 0 4

` `password 7 0822455D0A16

` `login

line vty 5 15

` `password 7 0822455D0A16

` `login

!
```
Настройка хостов ПК:
Настраиваем по таблице
```
PC-A    NIC 192.168.3.3 255.255.255.0   192.168.3.1

PC-B    NIC 192.168.4.3 255.255.255.0   192.168.4.1
```
**2. Создание виртуальных локальных сетей и назначение портов коммутатора**

- Создаем VLAN, как указано в таблице на обоих коммутаторах.
- Назначаем VLAN соответствующему интерфейсу.
- Устанавливаем шлюз по умолчанию
- Проверяем, что Vlan назначены правильным интерфейсам

|**VLAN**|**Name**|**Interface Assigned**|
| :-: | :-: | :-: |
|3|Management|S1: VLAN 3, S2: VLAN 3, S1: F0/6|
|4|Operations|S2: F0/18|
|7|ParkingLot|S1: F0/2-4, F0/7-24, G0/1-2, S2: F0/2-17, F0/19-24, G0/1-2|
|8|Native|N/A|


Конфиг S2:
```
interface FastEthernet0/23
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface FastEthernet0/24
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface GigabitEthernet0/1
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface GigabitEthernet0/2
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
interface Vlan3
 ip address 192.168.3.12 255.255.255.0
!
interface Vlan4
 no ip address
!
interface Vlan7
 no ip address
!
banner motd "Attention!!! Unauthorized access is prohibited!!!"
!

```
**3. Настройка магистрали 802.1Q между коммутаторами**

- Настраиваем интерфейс F0/1 в качестве магистрали на обоих коммутаторах
- Проверяем порты магистрали
  Конфиг S1:

!

interface FastEthernet0/1

` `switchport trunk native vlan 8

` `switchport trunk allowed vlan 3-4,8

` `switchport mode trunk

Осуществляем проверку командой “show interfaces trunk”.

**4. Настройка маршрутизации между VLAN на маршрутизаторе**

- Активируем интерфейс G0/0/1 на маршрутизаторе
- Настраиваем подинтерфейсы Vlan, как в таблице
- Проверяем работоспособность подинтерфейсов

Таблица:

|**Device**|**Interface**|**IP Address**|**Subnet Mask**|**Default Gateway**|
| :-: | :-: | :-: | :-: | :-: |
|R1|G0/0/1.3|192.168.3.1|255.255.255.0|N/A|
|*R1*|G0/0/1.4|192.168.4.1|255.255.255.0|*N/A*|
|*R1*|G0/0/1.8|N/A|N/A|*N/A*|


Конфиг R1:
```
!

interface GigabitEthernet0/0/1.3

` `encapsulation dot1Q 3

` `ip address 192.168.3.1 255.255.255.0

!

interface GigabitEthernet0/0/1.4

` `encapsulation dot1Q 4

` `ip address 192.168.4.1 255.255.255.0

!

interface GigabitEthernet0/0/1.8

` `no ip address

!
```
Проверяем командой “show ip interface brief”

**5. Проверка работы маршрутизации между VLAN**

- Проверка связи с PC-A на шлюз по умолчанию.
- Пинг с PC-A на PC-B
- Пинг с PC-A на S2

![Ping](https://github.com/Farix01/Otus-kurs/blob/main/Лаборатории/Vlan/Ping.jpg)

Итоговая схема подключения:
<br>
![Shema](https://github.com/Farix01/Otus-kurs/blob/main/Лаборатории/Vlan/Shema.jpg)
