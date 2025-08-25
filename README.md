## ¿Qué es?
Este proyecto permite automatizar la configuración de una red corporativa distribuida en varias sedes, utilizando Python y la librería Netmiko. El objetivo principal es facilitar la gestión remota de switches Cisco y routers MikroTik, aplicando configuraciones de VLAN, enlaces punto a punto y acceso seguro por SSH.

## Características
- Automatización de comandos para dispositivos Cisco y MikroTik.
- Configuración de VLAN de gestión y VLAN P2P.
- Asignación de direcciones IP y rutas estáticas.
- Habilitación de NAT para acceso a Internet.
- Pruebas de conectividad y verificación de estado.
- Seguridad mediante credenciales y SSH.

## Uso
1. Configura la IP de la PC SYSADMIN en la VLAN de gestión.
2. Instala Python y Netmiko siguiendo las instrucciones del documento.
3. Ejecuta el script principal para aplicar las configuraciones en todos los dispositivos.

## Resultado Esperado
- Red operativa con conectividad entre sedes y acceso a Internet.
- Dispositivos configurados automáticamente y listos para administración remota.

## Requisitos
- Python 3
- Netmiko
- Acceso SSH a los dispositivos de red
