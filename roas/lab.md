# Configure Router-on-a-Stick Inter-VLAN Routing
## Topology
![topology](https://github.com/viktorkomarov/net/blob/main/roas/topology)

### Addressing Table

| Device | Interface | IP Address   | Subnet Mask | Default Gateway |
|--------|-----------|--------------|-------------|-----------------|
| R1     | G0/0/1.3  | 192.168.3.1  | /24         |                 |
|        | G0/0/1.4  | 192.168.4.1  | /24         |                 |
|        | G0/0/1.8  |              |             |                 |
| S1     | VLAN 3    | 192.168.3.11 | /24         | 192.168.3.1     |
| S2     | VLAN 3    | 192.168.3.12 | /24         | 192.168.3.1     |
| PC-A   | NIC       | 192.168.3.3  | /24         | 192.168.3.1     |
| PC-B   | NIC       | 192.168.4.3  | /24         | 192.168.4.1     |

### VLAN Table

| VLAN | Name       | Interface Range                                            |
|------|------------|------------------------------------------------------------|
| 3    | Management | S1: VLAN 3 S2: VLAN 3 S1: F0/6                             |
| 4    | Operations | S2: F0/18                                                  |
| 7    | ParkingLot | S1: F0/2-4, F0/7-24, G0/1-2  S2: F0/2-17, F0/19-24, G0/1-2 |
| 8    | Native     |                                                            |

## Basic Router And Switch Configuration

a. Console into the router and enable privileged EXEC mode.
```console
enable
```
b. Enter configuration mode.
```console
conf t
```
c. Assign a device name to the router.
```console
hostname R1
```
d. Disable DNS lookup to prevent the router from attempting to translate incorrectly entered commands as though they were host names.
```console
no ip domain-lookup
```
e. Assign class as the privileged EXEC encrypted password.
```console
enable secret class
```
f. Assign cisco as the console password and enable login.
```console
line con 0
password cisco
login
```
g. Assign cisco as the VTY password and enable login.
```console
line vty 0 4
password cisco
login
```
h. Encrypt the plaintext passwords.
```console
service password-encryption
```
i. Create a banner that warns anyone accessing the device that unauthorized access is prohibited.
```console
banner motd #Unauthorized access to this device is prohibited!#
```
j. Save the running configuration to the startup configuration file.
```console
write
```
k. Set the clock on the router.
```console
clock set 18:50:00 Sep 4 2022
```
Аналогично конфигурируем S1/S2 

## Basic PC Configuration
### PC-A
![pca](https://github.com/viktorkomarov/net/blob/main/roas/pca)
### PC-B
![pcb](https://github.com/viktorkomarov/net/blob/main/roas/pcb)

## Create VLANs and Assign Switch Ports
S1 && S2:
a. Create and name the required VLANs on each switch from the table above.
```console
vlan 3
name Management
```
b. Configure the management interface and default gateway on each switch using the IP address information in the Addressing Table. 
```console
int vlan 3
ip address 192.168.3.11 255.255.255.0
no shutdown
exit
ip default-gateway 192.168.3.1
```
c. Assign all unused ports on both switches to the ParkingLot VLAN, configure them for static access mode, and administratively deactivate them

unused:
```console
int range f0/7-24
switchport mode access
switchport access vlan 8
shutdown
```

## Assign VLANs to the correct switch interfaces
a. Assign used ports to the appropriate VLAN (specified in the VLAN table above) and configure them for static access mode. Be sure to do this on both switches
```console
int f0/6
switchport mode access
switchport access vlan 3
```
b. Issue the show vlan brief command and verify that the VLANs are assigned to the correct interfaces.
![vlan](https://github.com/viktorkomarov/net/blob/main/roas/vlan_brief)

## Configure an 802.1Q Trunk Between the Switches
### Manually configure trunk interface F0/1.
a. Change the switchport mode on interface F0/1 to force trunking. Make sure to do this on both switches.
```console
int f0/1
switchport mode trunk
swtichport trunk enc dot1q
```
b. As a part of the trunk configuration, set the native VLAN to 8 on both switches. You may see error messages temporarily while the two interfaces are configured for different native VLANs.
```console
switchport trunk native vlan 8
```
c. As another part of trunk configuration, specify that VLANs 3, 4, and 8 are only allowed to cross the trunk.
```console
switchport trunk allow vlan 3,4,8
```
d. Issue the show interfaces trunk command to verify trunking ports, the Native VLAN and allowed VLANs across the trunk.
![trunk](https://github.com/viktorkomarov/net/blob/main/roas/trunk)

### Manually configure S1’s trunk interface F0/5

Аналогичная конфигурация для F0/5 S1
Порт сначала не покажется как trunk, так как с другой стороны роутер (порт по умолчанию выключен)

## Configure Inter-VLAN Routing on the Router
a. Activate interface G0/0/1 on the router.
```console
int g0/0/1
no shut
```
b. Configure sub-interfaces for each VLAN as specified in the IP addressing table. All sub-interfaces use 802.1Q encapsulation. Ensure the sub-interface for the native VLAN does not have an IP address assigned. Include a description for each sub-interface.
```console
int g0/0/1.3
enc dot1q 3
ip address 192.168.3.1 255.255.255.0
```
c. Use the show ip interface brief command to verify the sub-interfaces are operational.
![ip_brief](https://github.com/viktorkomarov/net/blob/main/roas/ip_brief)

### Complete the following tests from PC-A. All should be successful.
![ping_check](https://github.com/viktorkomarov/net/blob/main/roas/ping_check)