## Lab - Implement DHCPv4

<details>

<summary>Топология</summary>
  
![Топология](https://user-images.githubusercontent.com/74641903/113519062-a1f6c500-9592-11eb-9748-2bdaf3e46485.JPG)


</details>

<details>

<summary>Таблица адресации</summary>
  
| Устройство | Интерфейс | IP адрес | Маска подсести | Шлюз по-умолчанию |
| ---------- | --------------- | --------------- |--------------- | --------------- |
| R1         | e0/0      | 10.0.0.1 | 255.255.255.252 | N/A |
|            | e0/1      | N/A | N/A |  N/A |
|            | e0/1.100  | 192.168.1.1 | 255.255.255.192 | N/A |
|            | e0/1.200  | 192.168.1.65 | 255.255.255.224 | N/A |
|            | e0/1.1000 | N/A | N/A | N/A |
| R2         | e0/0 | 10.0.0.2 | 255.255.255.252 | N/A |
|            | e0/1 | 192.168.1.97 | 255.255.255.240 | N/A |
| S1         | VLAN 200 | 192.168.1.66 | 255.255.255.224 | 192.168.1.65 |
| S2         | VLAN 1 | 192.168.1.98 | 255.255.255.240 | 192.168.1.97 |
| PC-A       | eth0 | DHCP | DHCP | DHCP |
| PC-B       | eth0 | DHCP | DHCP | DHCP |

</details>

<details>

<summary>VLAN таблица</summary>
  
| VLAN | Имя | Интерфейс |
| ---------- | --------------- | --------------- |
| 1         |  N/A     | S2: e0/0 |
| 100           | Clients     | S1: e0/0 |
| 200           | Management | S1: VLAN 200 |
| 999           | Parking_Lot  | S1: e0/2-3 |
| 1000           | Native | N/A |


</details>

<details>

<summary>Задачи</summary>
  

Часть 1: Организовать сеть и провести базовую настройку устройств <br />
Часть 2: Настроить и проверить работу двух DHCPv4 серверов на  R1 <br />
Часть 3: Настроить и проверить работу  DHCP Relay на R2 


</details>

### Часть 1: Организовать сеть и провести базовую настройку устройств

#### Шаг 1: Организовать схему адресации

<details> 

<summary>Условие</summary>
Разбить 192.168.1.0/24 на 3 подсети по следующим параметрам: <br />
a. “Подсеть A”, поддерживающая 58 хостов (the client VLAN at R1). <br />
Подсеть A: 192.168.1.0/26 <br /> 
<br />

b. “Подсеть B”, поддерживающая 28 хостов (the management VLAN at R1). <br />
Подсеть B: 192.168.1.64/27 <br />
<br />

c.“Подсеть C”, поддерживающая 12 хостов (the client network at R2). <br />
Подсеть C: 192.168.1.96/28 <br />

</details>

#### Шаг 2: Подключить устройства согласно топологии
#### Шаг 3: Провести базовую настройку R1 и R2

<details>
<summary>R1</summary>

```
Router(config)#hostname R1
R1(config)#no ip domain lookup
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config)#service password-encryption
R1(config)#banner motd $ Only for authorized users $
R1#clock set 16:01:00 04 April 2021
R1#copy running-config startup-config

```

</details>

<details>
<summary>R2</summary>

```
Router(config)#hostname R2
R2(config)#no ip domain lookup
R2(config)#enable secret class
R2(config)#line console 0
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#login
R2(config)#service password-encryption
R2(config)#banner motd $ Only for authorized users $
R2#clock set 16:01:00 04 April 2021
R2#copy running-config startup-config

```

</details>

#### Шаг 4: Настроить межвлановую маршрутизацию на R1

<details>
<summary>R1</summary>

```
R1(config)#interface ethernet 0/1
R1(config-if)#no shutdown

R1(config)#interface e0/1.100
R1(config-subif)#description Clients
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#exit
R1(config)#interface e0/1.200
R1(config-subif)#encapsulation dot1Q 200
R1(config-subif)#description management
R1(config-subif)#ip address 192.168.1.65 255.255.255.224
R1(config-subif)#exit
R1(config)#interface e0/1.1000
R1(config-subif)#encapsulation dot1Q 1000 native
R1(config-subif)#description native
R1(config-subif)#exit


R1#sh ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES NVRAM  administratively down down
Ethernet0/1                unassigned      YES NVRAM  up                    up
Ethernet0/1.100            192.168.1.1     YES manual up                    up
Ethernet0/1.200            192.168.1.65    YES manual up                    up
Ethernet0/1.1000           unassigned      YES unset  up                    up



```

</details>

#### Шаг 5: Настроить e0/1 на R2, затем е0/0 и статическую маршрутизацию на R1 и R2


<details>
<summary>R2</summary>

```
R2(config)#interface e0/1
R2(config-if)#ip address 192.168.1.97 255.255.255.240
R2(config-if)#no shut
R2(config-if)#exit

R2(config)#interface e0/0
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#no shut

R2(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.1
```

</details>

<details>
<summary>R1</summary>

```
R1(config)#int e0/0
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#no shut

R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2

```

</details>
<details>

<summary>Проверка доступности R2: e0/1 c R1</summary>

```
R1#ping 192.168.1.97
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms


```

</details>

#### Шаг 6: Провести базовую настройку коммутаторов S1 и S2

<details>

<summary>S1</summary>

```
Switch(config)#hostname S1
S1(config)#no ip domain lookup
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption
S1(config)#banner motd $ only for autorized users $
S1(config)#exit
S1#copy running-config startup-config
S1#clock set  18:19:00 04 April 2021

```

</details>

<details>

<summary>S2</summary>

```
Switch(config)#hostname S2
S2(config)#no ip domain lookup
S2(config)#enable secret class
S2(config)#line console 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#line vty 0 4
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#service password-encryption
S2(config)#banner motd $ only for autorized users $
S2(config)#exit
S2#copy running-config startup-config
S2#clock set  18:19:00 04 April 2021

```

</details>

#### Шаг 7: Провести настройку VLAN и IP согласно имеющихся таблиц

<details>

<summary>Создать VLANs по таблице на S1</summary>

```
S1(config)#vlan 100
S1(config-vlan)#name Clients
S1(config-vlan)#vlan 200
S1(config-vlan)#name Management
S1(config-vlan)#vlan 999
S1(config-vlan)#name Parking_lot
S1(config-vlan)#vlan 1000
S1(config-vlan)#name Native
S1(config-vlan)#exit

```

</details>

<details>

<summary>Настроить и активировать интерфейс VLAN 200 на S1</summary>

```
S1(config)#interface vlan 200
S1(config-if)#ip address 192.168.1.66 255.255.255.224
S1(config-if)#no shut
S1(config-if)#exit
S1(config)#ip default-gateway 192.168.1.65

```

</details>

<details>

<summary>Настроить и активировать интерфейс VLAN 1 на S2</summary>

```
S2(config)# interface vlan 1
S2(config-if)# ip address 192.168.1.98 255.255.255.240
S2(config-if)# no shutdown
S2(config-if)# exit
S2(config)# ip default-gateway 192.168.1.97

```

</details>
<details>

<summary>Перевести все неиспользуемые порты на S1 во VLAN Parking_Lot, настроить в access, выключить их. На S2 выключить все неиспользуемые порты</summary>

```
S1(config)#int range ethernet 0/2 -3
S1(config-if-range)#switchport mode access
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#shutdown
S1(config-if-range)#exit

S2(config)#int range e0/2-3
S2(config-if-range)#switchport mode access
S2(config-if-range)#shutdown
S2(config-if-range)#exit


```

</details>
