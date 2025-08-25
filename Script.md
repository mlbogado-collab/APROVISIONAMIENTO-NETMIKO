# Guía de Automatización para Configuración de Red -- BOGADO MERELES, Micaela L.

## Introducción
Este documento presenta un script en Python para automatizar la configuración de equipos de red (switches Cisco y routers MikroTik) en una red corporativa con varias sedes. Se detalla cómo preparar la PC del administrador para conectarse por SSH, asignar la IP en la VLAN de gestión, instalar Python y Netmiko, y ejecutar el script de automatización.

## 1. Comprobar interfaz de red disponible
Ejecuta el siguiente comando para identificar la interfaz activa (descarta `lo`):
```bash
ip a
```
La interfaz que encuentres será la que uses para la configuración IP.

## 2. Configurar IP fija y ruta predeterminada
Utiliza estos comandos (sustituye `enp0s3` si tu interfaz tiene otro nombre):
```bash
sudo ip addr add 10.10.5.150/24 dev enp0s3
sudo ip link set enp0s3 up
sudo ip route add default via 10.10.5.159
```
- `10.10.5.150/24`: IP para la PC del administrador
- `10.10.5.159`: IP del MikroTik como puerta de enlace

## 3. Validar conectividad
Verifica la comunicación con el gateway y con Internet:
```bash
ping 10.10.5.159     # MikroTik
ping 8.8.8.8         # Google
```

## 4. Instalar Python y pip
Actualiza los repositorios e instala los paquetes necesarios:
```bash
sudo apt update
sudo apt install python3 python3-pip
```
Confirma la instalación:
```bash
python3 --version
pip3 --version
```

## 5. Ejecutar el script de automatización
Corre el script Python:
```bash
python3 Script.py
```

### Código del Script
```python
from netmiko import ConnectHandler

# ==================
# Equipos
# ==================
devices = {
    'SW1-LOCAL':{
        'device_type': 'cisco_ios',
        'host': '10.10.5.151',  # IP de gestión del SW1
        'username': 'admin',
        'password': 'admin',
        'secret': 'admin'
    },
    'SW2-REMOTO':{
        'device_type': 'cisco_ios',
        'host': '10.10.5.152',  # IP de gestión del SW2
        'username': 'admin',
        'password': 'admin',
        'secret': 'admin'
    },
    'R1-LOCAL':{
        'device_type': 'mikrotik_routeros',
        'host': '10.10.5.159',  # IP de gestión del R1
        'username': 'admin',
        'password': 'admin',
        'port': 22
    },
    'R2-REMOTO':{
        'device_type': 'mikrotik_routeros',
        'host': '10.10.5.599',  # IP de gestión del R2
        'username': 'admin',
        'password': 'admin',
        'port': 22
    }
}

# ==================
# Configuración de R1 (MikroTik)
# ==================
def r1_local():
    """
    Aplica la configuración al router R1-LOCAL (MikroTik):
    - VLAN de gestión (150) en ether3
    - VLAN P2P (599) en ether2
    - Asignación de IPs
    - NAT para acceso a Internet
    - Ruta estática hacia la sede remota
    """
    r1 = ConnectHandler(**devices['R1-LOCAL'])
    commands = [
        "/interface vlan add name=VLAN_GESTION vlan-id=150 interface=ether3",
        "/interface vlan add name=VLAN_P2P vlan-id=599 interface=ether2",
        "/ip address add address=10.10.5.159/24 interface=VLAN_GESTION",
        "/ip address add address=10.10.5.151/30 interface=VLAN_P2P",
        "/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade",
        "/ip route add dst-address=10.10.5.0/24 gateway=10.10.5.599"
    ]
    output = r1.send_config_set(commands)
    print(output)
    r1.disconnect()

# ==================
# Configuración de R2 (MikroTik)
# ==================
def r2_remoto():
    """
    Aplica la configuración al router R2-REMOTO (MikroTik):
    - VLAN de gestión remota (150) en ether2
    - VLAN P2P (599) en ether1
    - Asignación de IPs
    - NAT para acceso a Internet
    """
    r2 = ConnectHandler(**devices['R2-REMOTO'])
    commands = [
        "/interface vlan add name=VLAN_GESTION_REMOTA vlan-id=150 interface=ether2",
        "/interface vlan add name=VLAN_P2P vlan-id=599 interface=ether1",
        "/ip address add address=10.10.5.599/30 interface=VLAN_P2P",
        "/ip address add address=10.10.5.152/24 interface=VLAN_GESTION_REMOTA",
        "/ip firewall nat add chain=srcnat out-interface=VLAN_P2P action=masquerade"
    ]
    output = r2.send_config_set(commands)
    print(output)
    r2.disconnect()

# ==================
# Configuración de SW1 (Cisco)
# ==================
def sw1_local():
    """
    Aplica la configuración al switch SW1-LOCAL (Cisco):
    - Hostname y VLAN de gestión
    - Interface VLAN con IP
    - Gateway por defecto
    - Usuario admin y SSH
    - Puertos trunk y access
    """
    sw1 = ConnectHandler(**devices['SW1-LOCAL'])
    sw1.enable()
    commands = [
        "hostname SwitchLOCAL",
        "vlan 150",
        " name VLAN_GESTION",
        "exit",
        "interface vlan 150",
        " ip address 10.10.5.151 255.255.255.0",
        " no shutdown",
        "exit",
        "ip default-gateway 10.10.5.159",
        "username admin privilege 15 secret Admin123",
        "line vty 0 4",
        " transport input ssh",
        " login local",
        "exit",
        "ip domain-name redes.local",
        "crypto key generate rsa modulus 1024",
        "ip ssh version 2",
        # Trunk hacia R1
        "interface ethernet0/0",
        " switchport trunk encapsulation dot1q",
        " switchport mode trunk",
        " switchport trunk allowed vlan 150",
        "exit",
        # Access ports
        "interface ethernet0/1",
        " switchport mode access",
        " switchport access vlan 150",
        "exit",
        "interface ethernet0/2",
        " switchport mode access",
        " switchport access vlan 150",
        "exit",
        "interface ethernet0/3",
        " switchport mode access",
        " switchport access vlan 150",
        "exit",
        "interface ethernet1/0",
        " switchport mode access",
        " switchport access vlan 150",
        "exit",
        "interface ethernet1/1",
        " switchport mode access",
        " switchport access vlan 150",
        "exit",
        "do wr"
    ]
    output = sw1.send_config_set(commands)
    print(output)
    sw1.disconnect()

# ==================
# Configuración de SW2 (Cisco)
# ==================
def sw2_remoto():
    """
    Aplica la configuración al switch SW2-REMOTO (Cisco):
    - Hostname y VLAN de gestión
    - Interface VLAN con IP
    - Gateway por defecto
    - Usuario admin y SSH
    - Puertos trunk y access
    """
    sw2 = ConnectHandler(**devices['SW2-REMOTO'])
    sw2.enable()
    commands = [
        "hostname SwitchREMOTO",
        "vlan 150",
        " name VLAN_GESTION",
        "exit",
        "interface vlan 150",
        " ip address 10.10.5.152 255.255.255.0",
        " no shutdown",
        "exit",
        "ip default-gateway 10.10.5.151",
        "username admin privilege 15 secret Admin123",
        "line vty 0 4",
        " transport input ssh",
        " login local",
        "exit",
        "ip domain-name redes.local",
        "crypto key generate rsa modulus 1024",
        "ip ssh version 2",
        # Trunk hacia R2
        "interface Ethernet0/0",
        " switchport trunk encapsulation dot1q",
        " switchport mode trunk",
        " switchport trunk allowed vlan 150",
        " no shutdown",
        "exit",
        # Access port
        "interface Ethernet0/1",
        " switchport mode access",
        " switchport access vlan 150",
        " no shutdown",
        "exit",
        "do wr"
    ]
    output = sw2.send_config_set(commands)
    print(output)
    sw2.disconnect()

# ==================
# Ejecución principal
# ==================
if __name__ == "__main__":
    """
    Secuencia recomendada:
    1. Configurar los switches
    2. Configurar los routers
    """
    print("Comenzando la automatización de la red...")
    print("\nConfigurando SW1-LOCAL...")
    sw1_local()
    print("\nConfigurando SW2-REMOTO...")
    sw2_remoto()
    print("\nConfigurando R1-LOCAL...")
    r1_local()
    print("\nConfigurando R2-REMOTO...")
    r2_remoto()
    print("\n¡Todos los equipos han sido configurados correctamente!")
```

## ¿Cómo usar el script?
1. Guarda el archivo como `network_automation.py`
2. Verifica la conectividad con las IPs de gestión
3. Ejecuta: `python network_automation.py`

## Resultado esperado
El script realiza automáticamente:
- ✅ Configuración de 2 switches Cisco con VLAN de gestión
- ✅ Configuración de 2 routers MikroTik con enlace P2P
- ✅ Conectividad entre sedes
- ✅ Acceso a Internet
- ✅ SSH seguro habilitado

**Importante**: Revisa que los puertos, credenciales y direcciones IP coincidan con tu entorno antes de ejecutar.

---
