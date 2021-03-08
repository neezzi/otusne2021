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
##### 1.1 Подключаем устройства согласно имеющейся схемы
![Схема](https://user-images.githubusercontent.com/74641903/110259808-ad1dfb80-7fba-11eb-8057-c1c2a9cf0340.jpg)

##### 1.2 Проводим первичную настройку маршрутизатора

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

##### 1.3 Проводим базовую настройку коммутаторов

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

##### 1.4 Назначаем компьютерам IP Адреса

<details>
<summary>PC0</summary>
  
  ![PC0](https://user-images.githubusercontent.com/74641903/110260420-38988c00-7fbd-11eb-912b-b762f393f578.jpg)

  </details>
  
  
<details>
<summary>PC1</summary>
  
  
  ![PC1](https://user-images.githubusercontent.com/74641903/110260439-4fd77980-7fbd-11eb-8c9c-7a5d89d32276.jpg)

  </details>
  
### 2 Создаем ВЛАНы и назначаем их на порты коммутаторов <br/>
##### 2.1 Создаем ВЛАНы на обоих коммутаторах

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
  
  ##### 2.1.1 Зададим интерфейс управления и шлюз по-умолчанию согласно информации из таблицы
  
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
  
  ##### 2.1.2 Назначим на все не используемые порты vlan ParkingLot, переведем в access и отключим их
  
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
  
  ##### 2.2 Назначим вланы на нужные порты согласно таблицы, проверим результат командой show vlan brief
  
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
  
  ### 3 Настройка транка между коммутаторами, проверка командой show interface trunk
  
  
