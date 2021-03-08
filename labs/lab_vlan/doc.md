# Lab - Configure Router-on-a-Stick Inter-VLAN Routing
<details>
<summary>Схема</summary>

![image](https://user-images.githubusercontent.com/74641903/110258222-9673a680-7fb2-11eb-8dca-feb313fd12d4.png)

</details>

<details>
<summary>Таблица адресации</summary>

| Device        | Interface     | IP adress        | Subnet Mask     | Default Gateway |
| ------------- |:-------------:| :---------------:|:---------------:|----------------:|
| R1            | G0/0/1.3      |   192.168.3.1    | 255.255.255.0   | N/A             |
|               | G0/0/1.4      |   192.168.4.1    | 255.255.255.0   | N/A             |
|               | G0/0/1.8      |    N/A           | N/A             | N/A             |
| S1            | VLAN 3        | 192.168.3.11     | 255.255.255.0   | 192.168.3.1     |
| S2            | VLAN 3        | 192.168.3.12     | 255.255.255.0   | 192.168.3.1     | 
| PC0           | NIC           |   192.168.3.3    | 255.255.255.0   | 192.168.3.1     |
| PC1           | NIC           |   192.168.4.3    | 255.255.255.0   | 192.168.4.1     |

</details>

<details>
<summary>Таблица ВЛАН</summary>

| VLAN          | Name          | Interface Assigned    |
| ------------- |:-------------:| :---------------:     |
| 3             | Management    |   S1: VLAN 3 <br /> S2: VLAN 3 <br /> S1: F0/6  |
| 4             | Operations    |   S2: F0/18   |
| 7             | ParkingLot    |   S1: F0/2-4, F0/7-24, G0/1-2 <br /> S2: F0/2-17, F0/19-24, G0/1-2            |
| 8             | Native        | N/A     |

</details>
<details>
<summary>Задание</summary>
  
1. Организовать сеть и провести базовую настройку устройств
2. Создать ВЛАНы и назначить на порту коммутаторов
3. Настроить 802.1Q транк между коммутаторами
4. Настроить маршрутизацию между ВЛАН на маршрутизаторе
5. Убедиться что все работает

</details>

## Выполнение
### 1 Организовать сеть и провести базовую настройку устройств <br/>
##### 1.1 Подключить устройства согласно имеющейся схемы
![Схема](https://user-images.githubusercontent.com/74641903/110259808-ad1dfb80-7fba-11eb-8057-c1c2a9cf0340.jpg)

##### 1.2 Провести первичную настройку маршрутизатора

<details>
<summary>R1</summary>

```
Router>en
Router>enable                                           
Router#conf t                                           
Router(config)#hostname R1                              
R1(config)#no ip domain-lookup                          
R1(config)#enable secret class                          
R1(config)#line console 0
R1(config-line)#password cisco                          
R1(config-line)#login                                   
R1(config)#line vty 0 4
R1(config-line)#password cisco                          
R1(config-line)#login                                   
R1(config)#service password-encryption                  
R1(config)#banner motd $ For autoruzed users only $     
R1#copy running-config startup-config                  
Destination filename [startup-config]? 
R1#clock set 0:12:00 08 March 2021                      
```

</details>

##### 1.3 Провести базовую настройку коммутаторов

<details>
<summary>S1</summary>

```
Switch#enable
Switch#configure terminal 
Enter configuration commands, one per line. End with CNTL/Z.
Switch(config)#hostname S1
S1(config)#no ip domain-lookup 
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#line vty 0 15
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption 
S1(config)#banner motd $ For autoruzed users only $ 
S1#clock set 0:20:00 08 March 2021
S1#copy running-config startup-config 
```

</details>

<details>
<summary>S2</summary>

```
Switch#enable
Switch#configure terminal 
Enter configuration commands, one per line. End with CNTL/Z.
Switch(config)#hostname S1
S2(config)#no ip domain-lookup 
S2(config)#enable secret class
S2(config)#line console 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#line vty 0 15
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#service password-encryption 
S2(config)#banner motd $ For autoruzed users only $ 
S2#clock set 0:20:00 08 March 2021
S2#copy running-config startup-config 
```

</details>

##### 1.4 Назначить компьютерам IP Адреса согласно схемы

<details>
<summary>PC0</summary>
  
  ![PC0](https://user-images.githubusercontent.com/74641903/110260420-38988c00-7fbd-11eb-912b-b762f393f578.jpg)

  </details>
  
  
<details>
<summary>PC1</summary>
  
  
  ![PC1](https://user-images.githubusercontent.com/74641903/110260439-4fd77980-7fbd-11eb-8c9c-7a5d89d32276.jpg)

  </details>
  
### 2 Создать VLANs и назначить их на порты коммутаторов <br/>
##### 2.1 Создать VLANS на обоих коммутаторах

<details>
<summary>S1</summary>
 
```
S1(config)#vlan 3
S1(config-vlan)#name Management
S1(config-vlan)#vlan 4
S1(config-vlan)#name Operations
S1(config-vlan)#vlan 7
S1(config-vlan)#name ParkingLot
S1(config-vlan)#vlan 8
S1(config-vlan)#name Native
```

</details>
  
  <details>
<summary>S2</summary>

```  
S2(config)#vlan 3
S2(config-vlan)#name Management
S2(config-vlan)#vlan 4
S2(config-vlan)#name Operations
S2(config-vlan)#vlan 7
S2(config-vlan)#name ParingLot
S2(config-vlan)#vlan 8
S2(config-vlan)#name Native
```

  </details>
  
  ##### 2.1.1 Задать интерфейс управления и шлюз по-умолчанию согласно информации из таблицы
  
  <details>
<summary>S1</summary>

```  
S1(config)#interface vlan 3
S1(config-if)#ip address 192.168.3.11 255.255.255.0
S1(config-if)#no shutdown 
S1(config-if)#exit
S1(config)#ip default-gateway 192.168.3.1
```

  </details>
  
   <details>
<summary>S2</summary>

```  
S2(config)#interface vlan 3
S2(config-if)#ip address 192.168.3.12 255.255.255.0
S2(config-if)#no shutdown 
S2(config-if)#exit
S2(config)#ip default-gateway 192.168.3.1
```

  </details>
  
  ##### 2.1.2 Назначить на все не используемые порты vlan ParkingLot, перевести в access и отключить их
  
   <details>
<summary>S1</summary>

```  
S1(config)#interface range fa0/2 - 4 , fa0/7 - 24, gi0/1 - 2
S1(config-if-range)#switchport mode access 
S1(config-if-range)#switchport access vlan 7
S1(config-if-range)#shutdown 
```

  </details>
  
   <details>
<summary>S2</summary>

```  
S2(config)#interface range fa0/2 - 17, fa0/19 - 24, gi0/1 -2
S2(config-if-range)#switchport mode access 
S2(config-if-range)#switchport access vlan 7
S2(config-if-range)#shutdown  
```

  </details>
  
  ##### 2.2 Назначить VLANs на нужные порты согласно таблицы, проверить результат командой show vlan brief
  
   <details>
<summary>S1</summary>

```  
S1(config)#interface fa0/6
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 3

S1#show vlan brief 
VLAN Name Status Ports
---- -------------------------------- --------- -------------------------------
1 default active Fa0/1, Fa0/5
3 Management active Fa0/6
4 Operations active 
7 ParkingLot active Fa0/2, Fa0/3, Fa0/4, Fa0/7
Fa0/8, Fa0/9, Fa0/10, Fa0/11
Fa0/12, Fa0/13, Fa0/14, Fa0/15
Fa0/16, Fa0/17, Fa0/18, Fa0/19
Fa0/20, Fa0/21, Fa0/22, Fa0/23
Fa0/24, Gig0/1, Gig0/2
8 Native active 
1002 fddi-default active 
1003 token-ring-default active 
1004 fddinet-default active 
1005 trnet-default active 

```

  </details>
  
   <details>
<summary>S2</summary>

```  
S2(config)#interface fa0/18
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 4

S2#show vlan brief 
VLAN Name Status Ports
---- -------------------------------- --------- -------------------------------
1 default active Fa0/1
3 Management active 
4 Operations active Fa0/18
7 ParingLot active Fa0/2, Fa0/3, Fa0/4, Fa0/5
Fa0/6, Fa0/7, Fa0/8, Fa0/9
Fa0/10, Fa0/11, Fa0/12, Fa0/13
Fa0/14, Fa0/15, Fa0/16, Fa0/17
Fa0/19, Fa0/20, Fa0/21, Fa0/22
Fa0/23, Fa0/24, Gig0/1, Gig0/2
8 Native active 
1002 fddi-default active 
1003 token-ring-default active 
1004 fddinet-default active 
1005 trnet-default active 
```

  </details>
  
### 3 Настройка 802.1q транк на обоих коммутаторах
##### 3.1.a Настроить порты fa0/1 на обоих свичах в транк

   <details>
<summary>S1</summary>

```  
S1(config)#interface fa0/1
S1(config-if)#switchport mode trunk
```

  </details>
  
  ##### 3.1.b Установить vlan 8 как native на обоих портах, разрешить прохождение vlan 3,4,8 в trunk
  
   <details>
<summary>S1</summary>

```  
S1(config-if)#switchport trunk native vlan 8
S1(config-if)#switchport trunk allowed vlan 3,4,8
```

  </details>
  
   <details>
<summary>S2</summary>

```  
S2(config-if)#switchport trunk native vlan 8
S2(config-if)#switchport trunk allowed vlan 3,4,8
```

  </details>
  
##### 3.1.c Проверить, используя команду show interfaces trunk

   <details>
<summary>S1</summary>

```  
S1#show interfaces trunk
Port Mode Encapsulation Status Native vlan
Fa0/1 on 802.1q trunking 8

Port Vlans allowed on trunk
Fa0/1 3-4,8

Port Vlans allowed and active in management domain
Fa0/1 3,4,8

Port Vlans in spanning tree forwarding state and not pruned
Fa0/1 3,4,8

```

  </details>
  
   <details>
<summary>S2</summary>

```  
S2#show interfaces trunk
Port Mode Encapsulation Status Native vlan
Fa0/1 on 802.1q trunking 8

Port Vlans allowed on trunk
Fa0/1 3-4,8

Port Vlans allowed and active in management domain
Fa0/1 3,4,8

Port Vlans in spanning tree forwarding state and not pruned
Fa0/1 3,4,8

```

  </details>
  
  ##### 3.2 Настроить fa0/5 S1 по аналогии с fa0/1, сохранить конфигурацию, проверим командой show interfaces trunk 
  
   <details>
<summary>S1</summary>

```  
S1(config)#interface fa0/5
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 8
S1(config-if)#switchport trunk allowed vlan 3,4,8

S1#copy running-config startup-config
S2#copy running-config startup-config 

S1#show interfaces trunk
Port Mode Encapsulation Status Native vlan
Fa0/1 on 802.1q trunking 8
Port Vlans allowed on trunk
Fa0/1 3-4,8
Port Vlans allowed and active in management domain
Fa0/1 3,4,8
Port Vlans in spanning tree forwarding state and not pruned
Fa0/1 3,4,8

```

  </details>
  
 Fa0/5 не отображается в выводе команды, так как интерфейс gi0/0/1 на R1 в состоянии down
 
 ### 4 Настройка межвлановой маршрутизации на R1
 ##### 4.1 Поднять gi0/0/1, настроить сабинтерфейсы согласно таблицы
 
   <details>
<summary>R1</summary>

```  
R1(config)#interface gigabitEthernet 0/0/1
R1(config-if)#no shut

R1(config)#interface gi0/0/1.3
R1(config-subif)#description Management
R1(config-subif)#encapsulation dot1Q 3
R1(config-subif)#ip address 192.168.3.1 255.255.255.0

R1(config-subif)#interface gi0/0/1.4
R1(config-subif)#description Operations
R1(config-subif)#encapsulation dot1Q 4
R1(config-subif)#ip address 192.168.4.1 255.255.255.0

R1(config-subif)#interface gi0/0/1.8
R1(config-subif)#description Native
R1(config-subif)#encapsulation dot1Q 8 Native
```

  </details>
 
 ##### 4.2 Проверить командой show ip interfaces brief
 
   <details>
<summary>R1</summary>

```  
R1#show ip interface brief 
Interface IP-Address OK? Method Status Protocol 
GigabitEthernet0/0/0 unassigned YES unset administratively down down 
GigabitEthernet0/0/1 unassigned YES unset up up 
GigabitEthernet0/0/1.3 192.168.3.1 YES manual up up 
GigabitEthernet0/0/1.4 192.168.4.1 YES manual up up 
GigabitEthernet0/0/1.8 unassigned YES unset up up 
Vlan1 unassigned YES unset administratively down down
```

</details>

### 5 Проверить работу межвлановой маршрутизации
##### 5.1 Проверка командой ping с PC0

   <details>
<summary>PC0 ping to gateway</summary>
 
![gateway](https://user-images.githubusercontent.com/74641903/110294323-c2b61400-8000-11eb-86e2-e17b998c70e2.JPG)

</details>

   <details>
<summary>PC0 ping to PC1</summary>

![pc2](https://user-images.githubusercontent.com/74641903/110294334-c77ac800-8000-11eb-9720-5dce9cce6f3a.JPG)

</details>

   <details>
<summary>PC0 ping to S2</summary>
 
![S2](https://user-images.githubusercontent.com/74641903/110294352-cba6e580-8000-11eb-9924-8e4c40dca27c.JPG)

</details>

##### 5.2 Проверка командой tracert с PC1

   <details>
<summary>tracert PC1 to PC0</summary>
 
![tracert](https://user-images.githubusercontent.com/74641903/110294692-3fe18900-8001-11eb-8156-d273cd4371a1.JPG)

</details>


## Заключение 
Финальная схема с активными линками:
![финал](https://user-images.githubusercontent.com/74641903/110295691-74097980-8002-11eb-8495-76a700df8abd.JPG)
