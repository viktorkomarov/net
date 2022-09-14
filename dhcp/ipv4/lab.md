# DHCPv4
## Topology
![topology](https://github.com/viktorkomarov/net/blob/main/dhcp/ipv4/img/topology)

### Addressing Table

| Device | Interface   | IP Address | Subnet Mask     | Default Gateway |
|--------|-------------|------------|-----------------|-----------------|
| R1     | G0/0/0      | 10.0.0.1   | 255.255.255.252 | N/A             |
|        | G0/0/1      |            |                 |                 |
|        | G0/0/1.100  |            |                 |                 |
|        | G0/0/1.200  |            |                 |                 |
|        | G0/0/1.1000 |            |                 |                 |
| R2     | G0/0/0      | 10.0.0.2   | 255.255.255.252 | N/A             |
|        | G0/0/1      |            |                 |                 |
| S1     | VLAN 200    |            |                 |                 |
| S2     | VLAN 1      |            |                 |                 |
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