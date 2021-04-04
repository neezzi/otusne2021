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
Router(config)#hostname R1
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


