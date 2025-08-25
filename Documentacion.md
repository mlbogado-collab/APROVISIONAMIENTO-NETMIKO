# Documentación de Configuración de Red -- BOGADO MERELES, Micaela L.

## Configuraciones

### Servidor Linux (Debian 12)
```bash
# Configuración IP estática
sudo ip addr add 10.10.5.150/24 dev enp0s3
sudo ip link set enp0s3 up

# Gateway por defecto
sudo ip route add default via 10.10.5.159
```

### SW2 (Switch Remoto - Cisco)
```bash
conf t
hostname SwitchREMOTO
!
vlan 150
name VLAN_GESTION
!
interface vlan 150
ip address 10.10.5.152 255.255.255.0
no shutdown
!
ip default-gateway 10.10.5.151
!
username admin privilege 15 secret Admin123
!
line vty 0 4
transport input ssh
login local
!
interface Ethernet0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 150
no shutdown
!
interface Ethernet0/1
switchport mode access
switchport access vlan 150
no shutdown
!
do wr
```

### R2 (Router Remoto - MikroTik)
```bash
# VLANs
/interface vlan
add name=VLAN_GESTION_REMOTA vlan-id=150 interface=ether2
add name=VLAN_P2P vlan-id=599 interface=ether1

# IPs
/ip address
add address=10.10.5.599/30 interface=VLAN_P2P
add address=10.10.5.152/24 interface=VLAN_GESTION_REMOTA

# NAT para Internet
/ip firewall nat
add chain=srcnat out-interface=VLAN_P2P action=masquerade
```

### SW1 (Switch Local - Cisco)
```bash
conf t
hostname SwitchLOCAL
!
vlan 150
name VLAN_GESTION
!
interface vlan 150
ip address 10.10.5.151 255.255.255.0
no shutdown
!
ip default-gateway 10.10.5.159
!
username admin privilege 15 secret Admin123
!
line vty 0 4
transport input ssh
login local
!
interface ethernet0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 150
!
# Puertos de acceso en VLAN 150
interface range ethernet0/1 - 3, ethernet1/0 - 1
switchport mode access
switchport access vlan 150
!
do wr
```

### R1 (Router Principal - MikroTik)
```bash
# VLANs
/interface vlan
add name=VLAN_GESTION vlan-id=150 interface=ether3
add name=VLAN_P2P vlan-id=599 interface=ether2

# IPs
/ip address
add address=10.10.5.159/24 interface=VLAN_GESTION
add address=10.10.5.151/30 interface=VLAN_P2P

# NAT para Internet
/ip firewall nat
add chain=srcnat out-interface=ether1 action=masquerade

# Ruta estática a red remota
/ip route
add dst-address=10.10.5.0/24 gateway=10.10.5.599
```

---

## Pruebas de Conectividad

### Desde Switches:
```bash
# Test conectividad básica
ping 10.10.5.159
ping 10.10.5.151
```

### Desde Servidor Linux:
```bash
# Test gateway local
ping 10.10.5.159

# Test switch local
ping 10.10.5.151

# Test router remoto
ping 10.10.5.599

# Test switch remoto
ping 10.10.5.152

# Test internet
ping 8.8.8.8
```

---

## Seguridad

### Credenciales:
- **Usuario**: admin
- **Password**: Admin123
- **Privilegio**: Level 15 (máximo)

---

## Esquema de IPs

### Red Principal (Sede Local)
- **Subnet**: 10.10.5.0/24
- **Rango usable**: 10.10.5.150 - 10.10.5.159
- **Broadcast**: 10.10.5.255

### Enlace P2P (R1-R2)
- **VLAN P2P**: 599
- **R1**: 10.10.5.159
- **R2**: 10.10.5.599

### Red Remota (Sede Remota)
- **Subnet**: 10.10.5.0/24
- **SW1-LOCAL**: 10.10.5.151
- **SW2-REMOTO**: 10.10.5.152

---

## Comandos útiles
```bash
# MikroTik
/interface vlan print
/ip address print
/ip route print

# Cisco
show vlan
show ip interface brief
show running-config
```

---

## Notas Importantes

1. **VLAN 150**: Gestión de dispositivos de red
2. **VLAN 599**: Enlace punto a punto entre routers
3. **MTU**: Ajustar si hay problemas de fragmentación
4. **Firewall**: Considerar reglas adicionales en producción
5. **Backup**: Guardar configuraciones regularmente

---

## Topología de Red

```
 dejar igual
```

---

## Descripción General

Implementación de red corporativa con:
- **2 sedes** conectadas vía enlace P2P
- **VLAN de gestión** (VLAN 150) para dispositivos de red
- **VLAN P2P** (VLAN 599) para enlace entre routers
- **Conectividad a Internet** desde ambas locaciones
- **Administración via SSH** segura

---

### Verificar:
- ✅ Interfaces up/up
- ✅ Tablas de routing correctas
- ✅ NAT funcionando
- ✅ Conectividad VLAN trunking

---

**Última actualización**: 2025-08-25
