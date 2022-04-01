**STP-Лабораторная работа №2**

***Развертывание коммутируемой сети с резервными каналами***

***Задание***

Часть 1. Создание сети и настройка основных параметров устройства
Часть 2. Выбор корневого моста
Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

**Часть 1**

Подключаем устройства согласно схеме:

![Topology](https://github.com/Farix01/Otus-kurs/blob/main/%D0%9B%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%B8%D0%B8/STP/Topology.png)

Выполняем настройку коммутаторов, как показано в таблице:

```
|**Устройство**|	|**Интерфейс**|	|**IP-адрес**|	|**Маска подсети**|
| :-: | :-: | :-: | :-: |
|S1|	|VLAN 1|	|192.168.1.1|	|255.255.255.0|
|S2|	|VLAN 1|	|192.168.1.2|	|255.255.255.0|
|S3|	|VLAN 1|	|192.168.1.3|	|255.255.255.0|

```
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

