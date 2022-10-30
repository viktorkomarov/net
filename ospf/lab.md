# Goals
- Настроить OSPF офисе Москва
- Разделить сеть на зоны
- Настроить фильтрацию между зонами

# Tasks
- Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.
- Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.
- Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию.
- Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101.
- Настройка для IPv6 повторяет логику IPv4.

## Topology

![topology](https://github.com/viktorkomarov/net/blob/main/ospf/img/ospf_topology.jpeg)


## area 0

R14

```console
router ospf 1
router-id 14.14.14
area 101 stub no-summary
network 192.168.240.0 0.0.3.255 area 101
network 192.168.232.0 0.0.3.255 area 0
network 192.168.228.0 0.0.3.255 area 0
default-information originate
```

R15

```console
router ospf 1
router-id 15.15.15
network 192.168.244.0 0.0.3.255 area 102
network 192.168.236.0 0.0.3.255 area 0
network 192.168.224.0 0.0.3.255 area 0
default-information originate
distribute-list prefix PL101 in
ip prefix-list PL101 seq 5 deny 192.168.240.0/22
ip prefix-list PL101 seq 10 permit 0.0.0.0/0 le 32
```

## R12 - R13

R12

```console
router ospf 1
router-id 12.12.12.12
network 192.168.228.0 0.0.3.255 area 0
network 192.168.224.0 0.0.3.255 area 0
network 192.168.128.0 0.0.31.255 area 10
```

Same for R13

## R19

```console
router ospf 1
router-id 19.19.19.19
area 101 stub
network 192.168.240.0 0.0.3.255 area 101
```

## R20

```console
router ospf 1
router-id 20.20.20.20
network 192.168.244.0 0.0.3.255 area 102
```


### Summarize

R12

![topology](https://github.com/viktorkomarov/net/blob/main/ospf/img/r12.jpeg)

R19

![topology](https://github.com/viktorkomarov/net/blob/main/ospf/img/r19.jpeg)

R20

![topology](https://github.com/viktorkomarov/net/blob/main/ospf/img/r20.jpeg)

R14

![topology](https://github.com/viktorkomarov/net/blob/main/ospf/img/r14.jpeg)