# STP
## Topology
![topology](https://github.com/viktorkomarov/net/blob/main/stp/topology)

### Addressing Table

| Device | Interface | IP Address   | Subnet Mask |
|--------|-----------|--------------|-------------|
| S1     | VLAN 1    | 192.168.1.1  | /24         |
| S2     | VLAN 1    | 192.168.1.2  | /24         |
| S3     | VLAN 1    | 192.168.1.3  | /24         |

## Базовая конфигурация

Общая для всех свитчей, пример команд показан на S1
```console
enable
conf t
hostname S1
```
```console
no ip domain-lookup
```
```console
logging synchronous
```
```console
enable secret class
```
```console
line con 0
password cisco
login
```
```console
line vty 0 4
password cisco
login
```
```console
service password-encryption
```
```console
banner motd #Unauthorized access to this device is prohibited!#
```
```console
int vlan 1
ip addrress 192.168.1.1 255.255.255.0
no shut
```
```console
write
```

### Проверка связности
От S1

![ping1](https://github.com/viktorkomarov/net/blob/main/stp/ping1)

От S2

![ping2](https://github.com/viktorkomarov/net/blob/main/stp/ping2)

## Определение корневого свитча

Показано на пример S1

```console
int range fa0/1-4
shut
switchport mode trunk
exit
int range fa0/2,fa0/4
no shut
```

## Результаты show spanning-tree
### S1
![s1](https://github.com/viktorkomarov/net/blob/main/stp/s1)
### S2
![s2](https://github.com/viktorkomarov/net/blob/main/stp/s2)
### S3
![s3](https://github.com/viktorkomarov/net/blob/main/stp/s3)

### Анализ команд

Какой коммутатор является корневым мостом?
> S1

Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста? 
> Обладает наименьшим BRIDGE ID

Какие порты на коммутаторе являются корневыми портами? 
> S2: Fa0/2
> S3: Fa0/4

Какие порты на коммутаторе являются назначенными портами?
> S1: Fa0/2,Fa0/4
> S3: Fa0/2

Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?
> S2: Fa0/4

Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?
> У свитча S3 MAC адрес меньше и его интерфейс Fa0/2 - назначенный

## Изменяем cost

S2
```console
int fa0/2
spanning-tree vlan 1 cost 18
```

![s2_2](https://github.com/viktorkomarov/net/blob/main/stp/s2_2)

```console
int fa0/2
no spanning-tree vlan 1 cost 18
```

## Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

На всех свитчах
```console
int range fa0/1,fa0/,3
no shut
```

### S2
![s2_3](https://github.com/viktorkomarov/net/blob/main/stp/s2_3)

### S3
![3_2](https://github.com/viktorkomarov/net/blob/main/stp/s3_2)

Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?
> S2: Fa0/1
> S3: Fa0/3

Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?
> Они имеют меньший порядковый номер


## Вопросы для повторения
1. Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?
> COST

2. Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?
> BRIDGE ID

3. Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?
> PORT ID