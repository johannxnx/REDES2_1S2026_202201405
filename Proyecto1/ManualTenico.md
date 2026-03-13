# Proyecto 1 - Chapin Red
## Redes de Computadoras 2 - USAC
**Carnet:** 202201405  
**Nombre:** Johan Moises Cardona Rosales  
**Repositorio:** REDES2_1S2026_202201405  

---

## Índice
1. [Topología](#topología)
2. [Direccionamiento IP y Subnetting](#direccionamiento-ip-y-subnetting)
3. [VLANs](#vlans)
4. [Configuración VTP](#configuración-vtp)
5. [EtherChannel LACP - Edificio Izquierdo](#etherchannel-lacp---edificio-izquierdo)
6. [EtherChannel PAgP - Edificio Derecho](#etherchannel-pagp---edificio-derecho)
7. [Enrutamiento Dinámico EIGRP](#enrutamiento-dinámico-eigrp)
8. [Configuración DHCP](#configuración-dhcp)
9. [ACLs - Control de Acceso](#acls---control-de-acceso)
10. [Spanning Tree Protocol](#spanning-tree-protocol)

---

## Topología

### Topología General
![alt text](topologia.png)

### Red MAN (Metropolitan Area Network)
Conexión entre edificios mediante switches multicapa Cisco 3650 con fibra óptica (módulos GLC-LH-SMD).

| Enlace | Switch A | Puerto A | Switch B | Puerto B |
|--------|----------|----------|----------|----------|
| MS1 ↔ MS7 | MS1 | Gi1/1/1 | MS7 | Gi1/1/2 |
| MS1 ↔ MS2 | MS1 | Gi1/1/2 | MS2 | Gi1/1/1 |
| MS7 ↔ MS2 | MS7 | Gi1/1/3 | MS2 | Gi1/1/3 |
| MS7 ↔ MS6 | MS7 | Gi1/1/1 | MS6 | Gi1/1/2 |
| MS2 ↔ MS6 | MS2 | Gi1/1/2 | MS6 | Gi1/1/1 |
| MS1 ↔ DHCP1 | MS1 | Gi1/0/1 | DHCP1 | Fa0 |
| MS1 ↔ DHCP2 | MS1 | Gi1/0/2 | DHCP2 | Fa0 |

### Edificio Izquierdo (LACP)
| Enlace | Switch A | Puertos A | Switch B | Puertos B | Port-Channel |
|--------|----------|-----------|----------|-----------|--------------|
| MS7 ↔ MS8 | MS7 | Gi1/0/1-3 | MS8 | Fa0/1-3 | Po1 |
| MS7 ↔ MS9 | MS7 | Gi1/0/7-9 | MS9 | Fa0/7-9 | Po2 |
| MS9 ↔ MS8 | MS9 | Fa0/4-6 | MS8 | Fa0/4-6 | Po3 |

### Edificio Derecho (PAgP)
| Enlace | Switch A | Puertos A | Switch B | Puertos B | Port-Channel |
|--------|----------|-----------|----------|-----------|--------------|
| MS2 ↔ MS3 | MS2 | Gi1/0/2,4,5 | MS3 | Fa0/2,4,5 | Po1 |
| MS3 ↔ MS4 | MS3 | Fa0/3,6,7,8 | MS4 | Fa0/3,6,7,8 | Po2 |
| MS4 ↔ MS5 | MS4 | Fa0/4,5 | MS5 | Fa0/5,6 | Po3 |

---

## Direccionamiento IP y Subnetting

### Redes Base
- **VLANs:** 192.188.5.0/24
- **Enlaces:** 10.4.5.0/24

### Subnetting VLANs (192.188.5.0/24 → /27)

| VLAN | Red | Máscara | Gateway | Rango Hosts | Broadcast |
|------|-----|---------|---------|-------------|-----------|
| 55 - Naranja IZQ | 192.188.5.0/27 | 255.255.255.224 | 192.188.5.1 | 192.188.5.2 - .30 | 192.188.5.31 |
| 65 - Verde IZQ | 192.188.5.32/27 | 255.255.255.224 | 192.188.5.33 | 192.188.5.34 - .62 | 192.188.5.63 |
| 75 - Naranja DER | 192.188.5.64/27 | 255.255.255.224 | 192.188.5.65 | 192.188.5.66 - .94 | 192.188.5.95 |
| 85 - Verde DER | 192.188.5.96/27 | 255.255.255.224 | 192.188.5.97 | 192.188.5.98 - .126 | 192.188.5.127 |
| 95 - ADMIN | 192.188.5.128/27 | 255.255.255.224 | 192.188.5.129 | 192.188.5.130 - .158 | 192.188.5.159 |

### Subnetting Enlaces (10.4.5.0/24 → /30)

| Enlace | Red | IP Switch A | IP Switch B |
|--------|-----|-------------|-------------|
| MS1 Gi1/1/1 ↔ MS7 Gi1/1/2 | 10.4.5.0/30 | MS1: 10.4.5.1 | MS7: 10.4.5.2 |
| MS1 Gi1/1/2 ↔ MS2 Gi1/1/1 | 10.4.5.4/30 | MS1: 10.4.5.5 | MS2: 10.4.5.6 |
| MS7 Gi1/1/3 ↔ MS2 Gi1/1/3 | 10.4.5.8/30 | MS7: 10.4.5.9 | MS2: 10.4.5.10 |
| MS7 Gi1/1/1 ↔ MS6 Gi1/1/2 | 10.4.5.12/30 | MS7: 10.4.5.13 | MS6: 10.4.5.14 |
| MS2 Gi1/1/2 ↔ MS6 Gi1/1/1 | 10.4.5.16/30 | MS2: 10.4.5.17 | MS6: 10.4.5.18 |
| MS1 Gi1/0/1 ↔ DHCP1 | 10.4.5.20/30 | MS1: 10.4.5.21 | DHCP1: 10.4.5.22 |
| MS1 Gi1/0/2 ↔ DHCP2 | 10.4.5.24/30 | MS1: 10.4.5.25 | DHCP2: 10.4.5.26 |

---

## VLANs

| ID | Nombre | Edificio | Color |
|----|--------|----------|-------|
| 55 | VLAN_Naranja_EdificioIZQ_202201405 | Izquierdo | Naranja |
| 65 | VLAN_Verde_EdificioIZQ_202201405 | Izquierdo | Verde |
| 75 | VLAN_Naranja_EdificioDER_202201405 | Derecho | Naranja |
| 85 | VLAN_Verde_EdificioDER_202201405 | Derecho | Verde |
| 95 | VLAN_ADMIN_202201405 | Administración | Azul |

### Dispositivos Finales por VLAN

| Dispositivo | Switch | Puerto | VLAN | IP Asignada |
|-------------|--------|--------|------|-------------|
| PC1 | SW1 | Fa0/10 | 55 | 192.188.5.3 |
| PC2 | SW1 | Fa0/11 | 65 | 192.188.5.35 |
| Laptop0 | SW2 | Fa0/10 | 55 | 192.188.5.2 |
| Laptop1 | SW2 | Fa0/11 | 65 | 192.188.5.34 |
| Laptop2 | SW3 | Fa0/2 | 85 | 192.188.5.99 |
| PC3 | SW3 | Fa0/3 | 75 | 192.188.5.66 |
| Laptop3 | SW4 | Fa0/3 | 85 | 192.188.5.98 |
| PC4 | SW4 | Fa0/4 | 75 | 192.188.5.67 |
| PC0 | MS6 | Gi1/0/1 | 95 | 192.188.5.130 |

---

## Configuración VTP

**Dominio:** chapinred  
**Contraseña:** chapinred123  
**Versión:** 2

| Switch | Rol VTP |
|--------|---------|
| MS1 | Server |
| MS2, MS3, MS4, MS5, MS6, MS7, MS8, MS9, SW1, SW2, SW3, SW4 | Client |

### Comandos VTP Server (MS1)
```
configure terminal
vtp mode server
vtp domain chapinred
vtp password chapinred123
vtp version 2

vlan 55
 name VLAN_Naranja_EdificioIZQ_202201405
vlan 65
 name VLAN_Verde_EdificioIZQ_202201405
vlan 75
 name VLAN_Naranja_EdificioDER_202201405
vlan 85
 name VLAN_Verde_EdificioDER_202201405
vlan 95
 name VLAN_ADMIN_202201405
exit
write memory
```

### Comandos VTP Client (todos los demás switches)
```
configure terminal
vtp mode client
vtp domain chapinred
vtp password chapinred123
exit
write memory
```

### Verificación VTP
```
show vtp status
show vlan brief
```

---

## EtherChannel LACP - Edificio Izquierdo

### MS7 (Switch 3650 - Core)
```
configure terminal

interface range GigabitEthernet1/0/1-3
 switchport mode trunk
 channel-protocol lacp
 channel-group 1 mode active
exit

interface range GigabitEthernet1/0/7-9
 switchport mode trunk
 channel-protocol lacp
 channel-group 2 mode active
exit

interface port-channel 1
 switchport mode trunk
exit

interface port-channel 2
 switchport mode trunk
exit

write memory
```

### MS8 (Switch 3560 - Distribución)
```
configure terminal

interface range FastEthernet0/1-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 1 mode passive
exit

interface range FastEthernet0/4-6
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 3 mode passive
exit

interface port-channel 1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

interface port-channel 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

write memory
```

### MS9 (Switch 3560 - Distribución)
```
configure terminal

interface range FastEthernet0/7-9
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 2 mode passive
exit

interface range FastEthernet0/4-6
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 3 mode active
exit

interface port-channel 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

interface port-channel 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

write memory
```

### Verificación LACP
```
show etherchannel summary
show interfaces trunk
show spanning-tree vlan 55
```

---

## EtherChannel PAgP - Edificio Derecho

### MS2 (Switch 3650 - Core)
```
configure terminal

interface GigabitEthernet1/0/2
 switchport mode trunk
 channel-protocol pagp
 channel-group 1 mode desirable
exit

interface GigabitEthernet1/0/4
 switchport mode trunk
 channel-protocol pagp
 channel-group 1 mode desirable
exit

interface GigabitEthernet1/0/5
 switchport mode trunk
 channel-protocol pagp
 channel-group 1 mode desirable
exit

interface port-channel 1
 switchport mode trunk
exit

write memory
```

### MS3 (Switch 3560)
```
configure terminal

interface range FastEthernet0/2,FastEthernet0/4-5
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol pagp
 channel-group 1 mode auto
exit

interface range FastEthernet0/3,FastEthernet0/6-8
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol pagp
 channel-group 2 mode desirable
exit

write memory
```

### MS4 (Switch 3560)
```
configure terminal

interface range FastEthernet0/3,FastEthernet0/6-8
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol pagp
 channel-group 2 mode auto
exit

interface range FastEthernet0/4-5
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol pagp
 channel-group 3 mode desirable
exit

write memory
```

### MS5 (Switch 3560)
```
configure terminal

interface range FastEthernet0/5-6
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol pagp
 channel-group 3 mode auto
exit

write memory
```

### Verificación PAgP
```
show etherchannel summary
show interfaces trunk
```

## Pruebas de Tolerancia a Fallos

### LACP - Edificio Izquierdo

**Objetivo:** Verificar que el EtherChannel LACP mantiene conectividad al desconectar un puerto.

**Paso 1:** Iniciar ping continuo desde PC1 hacia PC3:
```
ping 192.188.5.66 repeat 100
```

**Paso 2:** Mientras corre el ping, apagar un puerto del Po1 en MS7:
```
configure terminal
interface GigabitEthernet1/0/1
shutdown
exit
```

**Paso 3:** Restaurar el puerto:
```
configure terminal
interface GigabitEthernet1/0/1
no shutdown
exit
write memory
```

---

### PAgP - Edificio Derecho

**Objetivo:** Verificar que el EtherChannel PAgP mantiene conectividad al desconectar un puerto.

**Paso 1:** Iniciar ping continuo desde PC3 hacia PC1:
```
ping 192.188.5.3 repeat 100
```

**Paso 2:** Mientras corre el ping, apagar un puerto del Po1 en MS2:
```
configure terminal
interface GigabitEthernet1/0/2
shutdown
exit
```

**Paso 3:** Restaurar el puerto:
```
configure terminal
interface GigabitEthernet1/0/2
no shutdown
exit
write memory
```



---

## Enrutamiento Dinámico EIGRP

**Protocolo:** EIGRP (carnet impar - 202201405)  
**AS Number:** 1

### MS1
```
configure terminal
ip routing

interface GigabitEthernet1/1/1
 no switchport
 ip address 10.4.5.1 255.255.255.252
exit

interface GigabitEthernet1/1/2
 no switchport
 ip address 10.4.5.5 255.255.255.252
exit

interface GigabitEthernet1/0/1
 no switchport
 ip address 10.4.5.21 255.255.255.252
exit

interface GigabitEthernet1/0/2
 no switchport
 ip address 10.4.5.25 255.255.255.252
exit

router eigrp 1
 network 10.4.5.0 0.0.0.3
 network 10.4.5.4 0.0.0.3
 network 10.4.5.20 0.0.0.3
 network 10.4.5.24 0.0.0.3
exit

write memory
```

### MS7
```
configure terminal
ip routing

interface GigabitEthernet1/1/1
 no switchport
 ip address 10.4.5.13 255.255.255.252
exit

interface GigabitEthernet1/1/2
 no switchport
 ip address 10.4.5.2 255.255.255.252
exit

interface GigabitEthernet1/1/3
 no switchport
 ip address 10.4.5.9 255.255.255.252
exit

interface Vlan55
 mac-address 0060.7024.9a01
 ip address 192.188.5.1 255.255.255.224
 ip helper-address 10.4.5.22
exit

interface Vlan65
 mac-address 0060.7024.9a02
 ip address 192.188.5.33 255.255.255.224
 ip helper-address 10.4.5.22
exit

router eigrp 1
 network 192.188.5.0 0.0.0.31
 network 192.188.5.32 0.0.0.31
 network 10.4.5.0 0.0.0.3
 network 10.4.5.8 0.0.0.3
 network 10.4.5.12 0.0.0.3
exit

write memory
```

### MS2
```
configure terminal
ip routing

interface GigabitEthernet1/1/1
 no switchport
 ip address 10.4.5.6 255.255.255.252
exit

interface GigabitEthernet1/1/2
 no switchport
 ip address 10.4.5.17 255.255.255.252
exit

interface GigabitEthernet1/1/3
 no switchport
 ip address 10.4.5.10 255.255.255.252
exit

interface Vlan75
 mac-address 000b.be12.1201
 ip address 192.188.5.65 255.255.255.224
 ip helper-address 10.4.5.26
exit

interface Vlan85
 mac-address 000b.be12.1202
 ip address 192.188.5.97 255.255.255.224
 ip helper-address 10.4.5.26
exit

router eigrp 1
 network 192.188.5.64 0.0.0.31
 network 192.188.5.96 0.0.0.31
 network 10.4.5.4 0.0.0.3
 network 10.4.5.8 0.0.0.3
 network 10.4.5.16 0.0.0.3
exit

write memory
```

### MS6
```
configure terminal
ip routing

interface GigabitEthernet1/1/1
 no switchport
 ip address 10.4.5.18 255.255.255.252
exit

interface GigabitEthernet1/1/2
 no switchport
 ip address 10.4.5.14 255.255.255.252
exit

interface Vlan95
 mac-address 0001.c9a1.5001
 ip address 192.188.5.129 255.255.255.224
 ip helper-address 10.4.5.22
exit

router eigrp 1
 network 192.188.5.128 0.0.0.31
 network 10.4.5.12 0.0.0.3
 network 10.4.5.16 0.0.0.3
exit

write memory
```

### Verificación EIGRP
```
show ip eigrp neighbors
show ip route
show ip eigrp topology
```

---

## Configuración DHCP

### DHCP1 (Servidor Edificio Izquierdo y ADMIN)
**IP:** 10.4.5.22 | **Gateway:** 10.4.5.21

| Pool | Default Gateway | Start IP | Subnet Mask | Max Users |
|------|----------------|----------|-------------|-----------|
| VLAN55 | 192.188.5.1 | 192.188.5.2 | 255.255.255.224 | 29 |
| VLAN65 | 192.188.5.33 | 192.188.5.34 | 255.255.255.224 | 29 |
| VLAN95 | 192.188.5.129 | 192.188.5.130 | 255.255.255.224 | 29 |

### DHCP2 (Servidor Edificio Derecho)
**IP:** 10.4.5.26 | **Gateway:** 10.4.5.25

| Pool | Default Gateway | Start IP | Subnet Mask | Max Users |
|------|----------------|----------|-------------|-----------|
| VLAN75 | 192.188.5.65 | 192.188.5.66 | 255.255.255.224 | 29 |
| VLAN85 | 192.188.5.97 | 192.188.5.98 | 255.255.255.224 | 29 |

### DHCP Relay (IP Helper)

#### MS7 - Edificio Izquierdo
```
interface Vlan55
 ip helper-address 10.4.5.22
exit

interface Vlan65
 ip helper-address 10.4.5.22
exit
```

#### MS2 - Edificio Derecho
```
interface Vlan75
 ip helper-address 10.4.5.26
exit

interface Vlan85
 ip helper-address 10.4.5.26
exit
```

#### MS6 - Edificio Admin
```
interface Vlan95
 ip helper-address 10.4.5.22
exit
```

### Verificación DHCP
```
show ip interface brief
```
En dispositivos finales: `ipconfig`

---

## ACLs - Control de Acceso

### Políticas implementadas
| Origen | Destino | Acción |
|--------|---------|--------|
| VLAN Naranja IZQ | VLAN Naranja DER | ✅ Permitido |
| VLAN Verde IZQ | VLAN Verde DER | ✅ Permitido |
| VLAN Naranja | VLAN Verde | ❌ Bloqueado |
| VLAN Verde | VLAN Naranja | ❌ Bloqueado |
| VLAN ADMIN | Todas las VLANs | ✅ Permitido |
| Otras VLANs | VLAN ADMIN | ❌ Bloqueado |

### ACL en MS7 (interfaces Gi1/1/1, Gi1/1/2, Gi1/1/3)
```
ip access-list extended ACL_VLAN55_65
 permit eigrp any any
 permit ip 192.188.5.0 0.0.0.31 192.188.5.0 0.0.0.31
 permit ip 192.188.5.0 0.0.0.31 192.188.5.64 0.0.0.31
 permit ip 192.188.5.64 0.0.0.31 192.188.5.0 0.0.0.31
 permit ip 192.188.5.32 0.0.0.31 192.188.5.32 0.0.0.31
 permit ip 192.188.5.32 0.0.0.31 192.188.5.96 0.0.0.31
 permit ip 192.188.5.96 0.0.0.31 192.188.5.32 0.0.0.31
 permit ip 192.188.5.128 0.0.0.31 192.188.5.0 0.0.0.63
 permit ip 192.188.5.0 0.0.0.63 192.188.5.128 0.0.0.31
 deny ip any any

interface GigabitEthernet1/1/1
 ip access-group ACL_VLAN55_65 in
interface GigabitEthernet1/1/2
 ip access-group ACL_VLAN55_65 in
interface GigabitEthernet1/1/3
 ip access-group ACL_VLAN55_65 in
```

### ACL en MS2 (interfaces Gi1/1/1, Gi1/1/2, Gi1/1/3)
```
ip access-list extended ACL_VLAN75_85
 permit eigrp any any
 permit ip 192.188.5.64 0.0.0.31 192.188.5.64 0.0.0.31
 permit ip 192.188.5.64 0.0.0.31 192.188.5.0 0.0.0.31
 permit ip 192.188.5.0 0.0.0.31 192.188.5.64 0.0.0.31
 permit ip 192.188.5.96 0.0.0.31 192.188.5.96 0.0.0.31
 permit ip 192.188.5.96 0.0.0.31 192.188.5.32 0.0.0.31
 permit ip 192.188.5.32 0.0.0.31 192.188.5.96 0.0.0.31
 permit ip 192.188.5.128 0.0.0.31 any
 permit ip 192.188.5.64 0.0.0.63 192.188.5.128 0.0.0.31
 deny ip any any

interface GigabitEthernet1/1/1
 ip access-group ACL_VLAN75_85 in
interface GigabitEthernet1/1/2
 ip access-group ACL_VLAN75_85 in
interface GigabitEthernet1/1/3
 ip access-group ACL_VLAN75_85 in
```

### ACL en MS6 (protección ADMIN)
```
ip access-list extended PROTECT_ADMIN
 permit eigrp any any
 deny ip 192.188.5.0 0.0.0.127 192.188.5.128 0.0.0.31
 permit ip any any

interface GigabitEthernet1/1/1
 ip access-group PROTECT_ADMIN in
interface GigabitEthernet1/1/2
 ip access-group PROTECT_ADMIN in
```

### Verificación ACLs
```
show ip access-lists
show ip interface GigabitEthernet1/1/1
```

---

## Spanning Tree Protocol

### Configuración Root Bridge
MS7 es el Root Bridge para VLANs del edificio izquierdo:
```
configure terminal
spanning-tree vlan 55 root primary
spanning-tree vlan 65 root primary
exit
```

MS2 es el Root Bridge para VLANs del edificio derecho:
```
configure terminal
spanning-tree vlan 75 root primary
spanning-tree vlan 85 root primary
exit
```

### Verificación STP
```
show spanning-tree vlan 55
show spanning-tree vlan 65
show spanning-tree vlan 75
show spanning-tree vlan 85
```

---

## Comandos de Verificación General

```bash
# Verificar interfaces
show ip interface brief

# Verificar rutas
show ip route

# Verificar vecinos EIGRP
show ip eigrp neighbors

# Verificar VLANs
show vlan brief

# Verificar trunks
show interfaces trunk

# Verificar EtherChannel
show etherchannel summary

# Verificar ARP
show arp

# Verificar VTP
show vtp status

# Verificar ACLs
show ip access-lists

# Verificar STP
show spanning-tree vlan 55
```

---
