# DHCPv4
## Topology
![topology](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/topology)

### Addressing Table

| Device | Interface   | IP Address | Subnet Mask     | Default Gateway |
|--------|-------------|------------|-----------------|-----------------|
| R1     | G0/0/0      | 10.0.0.1   | 255.255.255.252 | N/A             |
|        | G0/0/1      |            |                 |                 |
|        | G0/0/1.100  |192.168.1.1 | 255.255.255.192 |                 |
|        | G0/0/1.200  |192.168.1.65| 255.255.255.224 |                 |
|        | G0/0/1.1000 |            |                 |                 |
| R2     | G0/0/0      | 10.0.0.2   | 255.255.255.252 | N/A             |
|        | G0/0/1      |192.168.1.97| 255.255.255.240 |                 |
| S1     | VLAN 200    |192.168.1.66| 255.255.255.192 | 192.168.1.65    |
| S2     | VLAN 1      |192.168.1.98| 255.255.255.240 | 192.168.1.97    |
| PC-A   | NIC         | DHCP       | DHCP            | DHCP            |
| PC-B   | NIC         | DHCP       | DHCP            | DHCP            |

### VLAN Table

| VLAN | Name        | Interface Assigned          |
|------|-------------|-----------------------------|
| 1    | N/A         | S2: F0/18                   |
| 100  | Clients     | S1: F0/6                    |
| 200  | Management  | S1: VLAN 200                |
| 999  | Parking_Lot | S1: F0/1-4, F0/7-24, G0/1-2 |
| 1000 | Native      | N/A                         |

## Build the Network and Configure Basic Device Settings

### Establish an addressing scheme

Network 192.168.1.0/24
Need three subnet: 58, 28, 12 hosts

| Subnet | Need Hosts  | Allocated Hosts | Subnet          |
|--------|-------------|-----------------|-----------------|
| A      | 60 (58 + 2) | 62              | 192.168.1.0/26  |
| B      | 30 (28 + 2) | 30              | 192.168.1.64/27 |
| C      | 14 (12 + 2) | 14              | 192.168.1.96/28 |

Record the first IP address in the Addressing Table for R1 G0/0/1.100.
> 192.168.1.1 255.255.255.192

Record the first IP address in the Addressing Table for R1 G0/0/1.200. Record the second IP address in the Address Table for S1 VLAN 200 and enter the associated default gateway.
> R1 G0/0/1.200: 192.168.1.65 255.255.255.224
> S1 VLAN 200: 192.168.1.66 255.255.255.224

Record the first IP address in the Addressing Table for R2 G0/0/1.
> 192.168.1.97 255.255.255.240

### Configure basic settings for each router

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
clock set 18:50:00 Sep 14 2022
```
```console
write
```

### Configure Inter-VLAN Routing on R1

```console
int g0/0/1
no shut
```
```console
int g0/0/1.100
desc CLIENT
enc dot1q 100
ip addr 192.168.1.1 255.255.255.192
```
![topology](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/ip_int_brief)

### Configure G0/0/1 on R2, then G0/0/0 and static routing for both routers

Дефолтный роут на примере R2, аналогично для R1 c указанием нужного next-hope

```console
int g0/0/1
ip address 192.168.1.97 255.255.255.240
no shut
int g0/0/0
ip address 10.0.0.2 255.255.255.252
no shut
ip route 0.0.0.0 0.0.0.0 10.0.0.1
write
```
![ping](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/ping)

 Configure basic settings for each switch

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

### Create VLANs

```console
vlan 100
name CLIENTS
```
same for other VLANs

S1:
```console
int vlan 200
ip addr 192.168.1.66 255.255.255.224
no shut
exit
ip default-gateway 192.168.1.65
exit
write
```
S2:
```console
int vlan 1
ip addr 192.168.1.98 255.255.255.240
no shutd
exit
ip default-gateway 192.168.1.97
```
S1:
```console
int range f0/1-4,f0/7-24,g0/1-2
switchport mode access
switchport access vlan 999
shutdown
```
S2:
```console
int range fa0/1-4,fa0/6-17,fa0/19-24,g0/1-2
swtichport mode access
shut
```

### Assign VLANs to the correct switch interfaces.
S1:
```console
int fa0/6
switchport mode access
switcport access vlan 100
int fa0/5
switchport mode trunk
switchport trunk vlan native 1000
switchport trunk allow vlan 100,200,1000
```
Изначально fa0/5 находился в дефолтном влане

![vlan_brief](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/vlan_brief)

![trunk_brief](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/trunk_brief)

At this point, what IP address would the PC’s have if they were connected to the network using DHCP?
> DHCP сервис еще не настроен => устройства будут получать дефолтные значения 

## Configure and verify two DHCPv4 Servers on R1

### Configure R1 with DHCPv4 pools for the two supported subnets
```console
ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp pool SUBNET_A
network 192.168.1.0 255.255.255.192
domain-name ccna-lab.com
default-router 192.168.1.1
lease 2 12 30 // недоступно в Cisco PacketTrace
exit
```

![subnet_a](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/subnet_a)

same for SUBNET_C

![subnet_c](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/SUBNET_C)

### Attempt to acquire an IP address from DHCP on PC-A
![renew_a](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/renew)
![ping_pca](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/ping_pca)

### Configure R2 as a DHCP relay agent for the LAN on G0/0/1
```console
int g0/0/1
ip helper-address 10.0.0.1
```

### Attempt to acquire an IP address from DHCP on PC-B

![renew_pcb](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/renew_pcb)
![ping_pcb](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/ping_pcb)

![binding](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/binding)

