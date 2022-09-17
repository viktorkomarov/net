# DHCPv6
## Topology
![topology](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv6/img/topology)

### Addressing Table

| Device | Interface | IPv6                   |
|--------|-----------|------------------------|
| R1     | G0/0      | 2001:db8:acad:2::1 /64 |
|        |           | fe80::1                |
|        | G0/1      | 2001:db8:acad:1::1 /64 |
|        |           | fe80::1                |
| R2     | G0/0      | 2001:db8:acad:2::2 /64 |
|        |           | fe80::1                |
|        | G0/1      | 2001:db8:acad:3::1 /64 |
|        |           | fe80::1                |
| PC_A   | NIC       | DHCP                   |
| PC_B   | NIC       | DHCP                   |


## Basic config for switch

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
write
```

## Configure basic settings for each router

```console
enable
conf t
hostname R1
```
```console
no ip domain-lookup
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
ipv6 unicast-routing
```
```console
write
```

## Configure interfaces and routing for both routers.

R1:
```console
int e0/0
ipv6 address 2001:db8:acad:2::1/64
ipv6 address fe80::1 link-local
no shut
int g0/1
ipv6 address 2001:db8:acad:1::1/64
ipv6 address fe80::1 link-local
no shut
ipv6 route ::/0 2001:db8:acad:2::2
```

![ping](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv6/img/ping)


## Verify SLAAC Address Assignment from R1

![SLAAC](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv6/img/slaac)

Where did the host-id portion of the address come from?
> EUI-64


## Configure R1 to provide stateless DHCPv6 for PC-A.

R1:
```console
ipv6 dhcp pool R1-STATELESS
dns-server 2001:db8:acad::254
domain-name STATELESS.com
int e0/1
ipv6 nd other-config-flag
ipv6 dhcp server R1-STATELESS
```

![stateless_ping](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv6/img/stateless_ping)

## Configure a stateful DHCPv6 server on R1
R1:
```console
ipv6 dhcp pool R2-STATEFUL
address prefix 2001:db8:acad:3:aaa::/80
dns-server 2001:db8:acad::254
domain-name STATEFUL.com
int e0/0
ipv6 dhcp server R2-STATEFUL
```
## Configure R2 as a DHCP relay agent for the LAN
```console
int e0/1
ipv6 nd managed-config-flag
ipv6 dhcp relay destination 2001:db8:acad:2::1 e0/0
```

![pc_b](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv6/img/pc_b)