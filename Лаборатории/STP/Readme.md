# STP-Лабораторная работа №2

***Развертывание коммутируемой сети с резервными каналами***

***Задание***

Часть 1. Создание сети и настройка основных параметров устройства

Часть 2. Выбор корневого моста

Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов

Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

# Часть 1

Подключаем устройства согласно схеме:

![Topology](https://github.com/Farix01/Otus-kurs/blob/main/%D0%9B%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%B8%D0%B8/STP/Topology.png)

Выполняем настройку коммутаторов, как показано в таблице:

|**Устройство**|**Интерфейс**|**IP-адрес**|**Маска подсети**|
| :-: | :-: | :-: | :-: |
|S1|VLAN 1|192.168.1.1|255.255.255.0|
|S2|VLAN 1|192.168.1.2|255.255.255.0|
|S3|VLAN 1|192.168.1.3|255.255.255.0|


Конфиг:

```
hostname S1
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
enable password cisco
!
no ip domain-lookup
!
interface Vlan1
 ip address 192.168.1.1 255.255.255.0
!
banner motd ^CAttention!!! Unauthorized access is prohibited!!!^C
!
line con 0
 password cisco
 logging synchronous
 login
 exec-timeout 0 0
!
line vty 0 4
 exec-timeout 0 0
 logging synchronous
 login
line vty 5 15
 exec-timeout 0 0
 logging synchronous
 login
```
***После этих настроек все коммутаторы успешно пингуют друг друга.***

# Часть 2

***Согласно методичке оставляем включенные порты F0/2 и F0/4 и переводим их в Транк***
 ```
interface FastEthernet0/1
 switchport mode trunk
 shutdown
!
interface FastEthernet0/2
 switchport mode trunk
!
interface FastEthernet0/3
 switchport mode trunk
 shutdown
!
interface FastEthernet0/4
 switchport mode trunk
!
 ```
***Отображаем данные протокола STP***

Конфиг:
```
S1#show span
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.116E.7750
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0010.116E.7750
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Desg FWD 19        128.4    P2p
Fa0/2            Desg FWD 19        128.2    P2p
```

```
S2#show span
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.116E.7750
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.21EE.D503
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Desg FWD 19        128.4    P2p
Fa0/2            Root FWD 19        128.2    P2p
```

```
S3#show span
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.116E.7750
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00E0.B0C1.4E4D
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

**Отобразим на схеме роль активных портов на каждом коммутаторе**

![Shema](https://github.com/Farix01/Otus-kurs/blob/main/%D0%9B%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%B8%D0%B8/STP/Shema.png)

**Ответы на вопросы**

 Приоритет у трех коммутаторов одинаковый, поэтому root будет выбираться по наименьшему BID, в нашем случае - это коммутатор S1 за счёт меньшего мак адреса.
 
 Корневыми портами назначаются те порты, у которых стоимость пути до корневого моста меньше. По нашей топологии стоимость портов одинакова до root (0+19). S2: FA0/2, S3: FA0/2.
 ```
S2#show spanning-tree interface fa0/2
VLAN0001 Root FWD 19
 ```
На корневом мосте все порты являются назначенными, так же если на одном конце сегмента находится корневой порт, на другом будет назначенный порт. На коммутаторе, не являющимся корневым, назначенный - считается порт с наименьшей стоимостью пути к корневому мосту. В нашем примере - это  S2: FA0/4. 

Порт F0/2 на S3 назначен в качестве альтернативного и заблокирован для предотвращения петли, т.к. его BID больше за счёт большего мак-адреса.

# Часть 3

***Определим коммутатор с заблокированным портом***
```
S3#show span
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.116E.7750
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00E0.B0C1.4E4D
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```
***Настроим приоритетность порта***
На коммутаторе S3 уменьшаем приоритет активного порта.
Команда:
```
S3(config)# interface f0/4
S3(config-if)# spanning-tree cost 18
```
После чего видим такие значения:

```
S3#show span
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.116E.7750
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00E0.B0C1.4E4D
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Root FWD 18        128.4    P2p
```
***Заблокированный порт на S3 стал назначенным, так как его стоимость стала ниже*** 
```
S2#show span
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.116E.7750
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.21EE.D503
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Desg FWD 19        128.4    P2p
Fa0/2            Root BLK 19        128.2    P2p

```
***На коммутаторе S2 заблокировался порт fa0/2, так как его стоимость осталась 19, и эта самая большая стоимость до root.***
***Отменим приоритет 18 на порту, вернём всё на свои места***
```
S3(config)#interface fa0/4
S3(config-if)#no spanning-tree cost 18
S3(config-if)#end
```
После этого снова заблокировался порт на свиче S3.

# Часть 4
***Включим ранее отключенные порты на свичах***
```
S3(config)#int range fa0/1, fa0/3
S3(config-if-range)#no shutdown
```
***Проведём наблюдение***
```
S3#show span
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.116E.7750
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00E0.B0C1.4E4D
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Altn BLK 19        128.1    P2p
Fa0/3            Root FWD 19        128.3    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Altn BLK 19        128.4    P2p
```
```
S2#show span
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.116E.7750
             Cost        19
             Port        1(FastEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.21EE.D503
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Desg FWD 19        128.4    P2p
Fa0/1            Root FWD 19        128.1    P2p
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/2            Altn BLK 19        128.2    P2p
```

После чего, мы можем подвести итог:

1) В качестве корневых портов выбраны: S3:Fa0/3, S2:Fa0/1.
2) STP выбрал данные порты, так как их BID меньше, за счёт номера порта в сторону корневого моста.
