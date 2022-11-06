# Goals
- Настроить IS-IS офисе Триада

# Tasks
- R23 и R25 находятся в зоне 2222
- R24 находится в зоне 24
- R26 находится в зоне 26

## Topology

![isis](https://github.com/viktorkomarov/net/blob/main/isis/img/isis.jpeg)

![isis_with_area](https://github.com/viktorkomarov/net/blob/main/isis/img/isis_with_area.jpeg)

### R23

R23:

```console
router isis
net 49.2222.0023.0023.0023.00
exit
int e0/1
ip router isis
ipv6 router isis
exit
```

### R25

R25:

```console
router isis
net 49.2222.0025.0025.0025.00
exit
int range e0/0,e0/2
ip router isis
ipv6 router isis
exit
```

### R24

R24:

```console
router isis
net 49.0024.0024.0024.0024.00
exit
int range e0/1-2
ip router isis
ipv6 router isis
exit
```

### R26

R26:

```console
router isis
net 49.0026.0026.0026.0026.00
exit
int range e0/0,e0/2
ip router isis
ipv6 router isis
exit
```

### Check settings

R23:

![r23_settings](https://github.com/viktorkomarov/net/blob/main/isis/img/r23_settings.jpeg)

R26:

![r26_settings](https://github.com/viktorkomarov/net/blob/main/isis/img/r26_settings.jpeg)