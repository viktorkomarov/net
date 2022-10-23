# Goals
- Настроить политику маршрутизации в офисе Чокурдах
- Распределить трафик между 2 линками

# Tasks
- Настроите политику маршрутизации для сетей офиса.
- Распределите трафик между двумя линками с провайдером.
- Настроите отслеживание линка через технологию IP SLA.(только для IPv4)
- Настройте для офиса Лабытнанги маршрут по-умолчанию.

## Настройте для офиса Лабытнанги маршрут по-умолчанию

R27

```console
ip route 0.0.0.0 0.0.0.0 70.1.0.2
ipv6 route ::/0 2001:0:1::2
```


## Настроить политику маршрутизации в офисе Чокурдах (распределить трафик между 2 линками)

Офис Чокурдах

![topology](https://github.com/viktorkomarov/net/blob/main/pbr/img/pbr_base.jpeg)

Пусть трафик с VLAN30 (192.168.0.0 /20) пойдет через R28(e0/0), а трафик с VLAN40 (192.168.16.0 /20) пойдет через R28(e0/1)

R28:
```console 
ip access-list extended vlan30
permit ip 192.168.0.0 0.0.15.255 any
exit
```

```console 
ip access-list extended vlan40
permit ip 192.168.16.0 0.0.15.255 any
exit
```

```console
route-map REBALANCE permit 10
match ip address vlan30
set ip next-hop 70.1.2.193
exit
```

```console
route-map REBALANCE permit 15
match ip address vlan40
set ip next-hop 70.1.2.65
exit
```

```console
interface e0/2.30
ip policy route-map REBALANCE
interface e0/2.40
ip policy route-map REBALANCE
exit
```


### VPC30

Before

![vpc30_before](https://github.com/viktorkomarov/net/blob/main/pbr/img/vpcs30_before.jpeg)

After

![vpc30](https://github.com/viktorkomarov/net/blob/main/pbr/img/vpcs30.jpeg)

### VPC31

Before

![vpc31_before](https://github.com/viktorkomarov/net/blob/main/pbr/img/vpcs31_before.jpeg)

After

![vpc31](https://github.com/viktorkomarov/net/blob/main/pbr/img/vpcs31.jpeg)

## Настроите отслеживание линка через технологию IP SLA.(только для IPv4)

R28

```console
ip sla 1
 icmp-echo 70.1.2.65 source-ip 70.1.2.66
 frequency 5
exit
ip sla schedule 1 life forever start-time now
ip sla 2
 icmp-echo 70.1.2.193 source-ip 70.1.2.194
 frequency 5
exit
ip sla schedule 2 life forever start-time now
track 1 ip sla 1 reachability
track 2 ip sla 2 reachability
```

### IP SLA && PBR

```console
route-map REBALANCE permit 10
 match ip address vlan30
 set ip next-hop verify-availability 70.1.2.65 10 track 1
 set ip next-hop verify-availability 70.1.2.193 20 track 2
exit
route-map REBALANCE permit 20
 match ip address vlan32
 set ip next-hop verify-availability 70.1.2.193 10 track 2
 set ip next-hop verify-availability 70.1.2.65 20 track 1
exit
```