## Лабораторная работа. Развертывание коммутируемой сети с резервными каналами

<details>

<summary>Топология</summary>
  
![image](https://user-images.githubusercontent.com/74641903/110851105-aac0e780-82c1-11eb-9493-a3febd95e2ef.png)


</details>

<details>

<summary>Таблица адресации</summary>
  
| Устройство | Интерфейс | IP адрес | Маска подсести |
| --------------- | --------------- | --------------- |--------------- |
| S1 | VLAN 1 | 192.168.1.1 | 255.255.255.0 |
| S2 | VLAN 1 | 192.168.1.2 | 255.255.255.0 |
| S3 | VLAN 1 | 192.168.1.3 | 255.255.255.0 |

</details>

<details>

<summary>Цели</summary>
  
Часть 1. Создание сети и настройка основных параметров устройства <br />
Часть 2. Выбор корневого моста <br />
Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов <br />
Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов <br />


</details>

### Часть 1:	Создание сети и настройка основных параметров устройства
#### Шаг 1:	Организовать сеть согласно топологии

<details>

<summary>Сеть в Packet Tracer</summary>
  
![Топология](https://user-images.githubusercontent.com/74641903/110851785-903b3e00-82c2-11eb-826f-0997d37a2b7a.JPG)

</details>

#### Шаг 2:	Провести базовую настройку коммутаторов

<details>

<summary>Базовая настройка</summary>

```
Switch(config)#hostname S1
S1(config)#no ip domain-lookup
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#logging synchronous 
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#line vty 0 15
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#banner motd $ For autoruzed users only $ 
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.1 255.255.255.0

Switch(config)#hostname S2
S2(config)#no ip domain-lookup
S2(config)#enable secret class
S2(config)#line console 0
S2(config-line)#logging synchronous 
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#line vty 0 15
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#banner motd $ For autoruzed users only $ 
S2(config)#interface vlan 1
S2(config-if)#ip address 192.168.1.2 255.255.255.0

Switch(config)#hostname S3
S3(config)#no ip domain-lookup
S3(config)#enable secret class
S3(config)#line console 0
S3(config-line)#logging synchronous 
S3(config-line)#password cisco
S3(config-line)#login
S3(config-line)#exit
S3(config)#line vty 0 15
S3(config-line)#password cisco
S3(config-line)#login
S3(config-line)#exit
S3(config)#banner motd $ For autoruzed users only $ 
S3(config)#interface vlan 1
S3(config-if)#ip address 192.168.1.3 255.255.255.0

```
  
</details>

#### Шаг 3:	Проверить связность

<details>

<summary>ping S1 to S2</summary>
  
![ping S1 to S2](https://user-images.githubusercontent.com/74641903/110852501-82d28380-82c3-11eb-8eae-681b4bad6838.JPG)

</details>

<details>

<summary>ping S1 to S3</summary>
  
![ping S1 to S3](https://user-images.githubusercontent.com/74641903/110852521-88c86480-82c3-11eb-9c56-eeef48090029.JPG)

</details>

<details>

<summary>ping S2 to S3</summary>
  
![ping S2 to S3](https://user-images.githubusercontent.com/74641903/110852543-8d8d1880-82c3-11eb-838c-8cffbb7e17c5.JPG)

</details>

### Часть 2:	Определение root bridge
#### Шаг 1:	Выключить все порты на коммутаторах
<details>

<summary>shutdown</summary>

```
S1(config)#interface range fa0/1-24, gi0/1-2
S1(config-if-range)#shutdown

S2(config)#interface range fa0/1-24, gi0/1-2
S2(config-if-range)#shutdown

S3(config)#interface range fa0/1-24, gi0/1-2
S3(config-if-range)#shutdown

```
  
</details>

#### Шаг 2:	Настроить подключенные, согласно топологии порты в trunk

<details>

<summary>switchport mode trunk</summary>

```
S1(config)#interface range fa0/1-4
S1(config-if-range)#switchport mode trunk

S2(config)#interface range fa0/1-4
S2(config-if-range)#switchport mode trunk

S3(config)#interface range fa0/1-4
S3(config-if-range)#switchport mode trunk
```

</details>

