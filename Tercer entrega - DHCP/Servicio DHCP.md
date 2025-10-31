

[!IMPORTANT] Objetivo

> **Objetivo del TP:** diseñar, implementar y **documentar** una LAN por grupo donde la **Raspberry Pi** actúe como **servidor DHCP** para `192.168.{{GRUPO}}.0/24`.  

> La solución debe:

> - Asignar **IP dinámicas** (rango sugerido `192.168.{{GRUPO}}.100–200`, gateway `192.168.{{GRUPO}}.1`, máscara `/24`, DNS públicos).

> - Integrar **Raspberry + switch + notebooks** en topología simple y funcional.

> - **Documentarse** con un diagrama en **Packet Tracer** (DORA observable).

> - **Verificarse** con pruebas (ping, leases).

> - (Opcional) **NAT** si la Pi tiene upstream por `wlan0`.

  

## Índice

- [[#Introducción|Introducción]]

- [[#Desarrollo|Desarrollo]]

  - [[#Plan-de-direccionamiento|Plan de direccionamiento]]

  - [[#Configurar-IP-estática-en-la-Raspberry|Configurar IP estática en la Raspberry]]

  - [[#Servidor-DHCP—opción-A-ISC-DHCP-Server|Servidor DHCP — opción A (ISC)]]

  - [[#Servidor-DHCP—opción-B-dnsmasq|Servidor DHCP — opción B (dnsmasq)]]

  - [[#Pruebas-y-verificación|Pruebas y verificación]]

  - [[#Diagrama-en-Packet-Tracer|Diagrama en Packet Tracer]]

  - [[#Extra-opcional-Salida-a-Internet-por-NAT|Extra (opcional): Salida a Internet por NAT]]

  - [[#Solución-de-problemas|Solución de problemas]]

- [[#Conclusión|Conclusión]]

  

---
## Introducción

La **Raspberry Pi** funcionará como **servidor DHCP** de la LAN del grupo (`192.168.{{GRUPO}}.0/24`). La Raspberry (por **eth0**) y las notebooks se conectan al **switch** del grupo. La entrega incluye un **diagrama en Packet Tracer** que refleje el diseño implementado.

---
## Desarrollo

### Plan de direccionamiento

- **Red LAN:** `192.168.{{GRUPO}}.0/24`  

- **Raspberry (eth0):** `192.168.{{GRUPO}}.1`  

- **Rango DHCP (clientes):** `192.168.{{GRUPO}}.100–192.168.{{GRUPO}}.200`  

- **Máscara:** `255.255.255.0`  

- **Broadcast:** `192.168.{{GRUPO}}.255`  

- **DNS sugeridos:** `1.1.1.1`, `8.8.8.8`

  

> [!tip] Conexiones físicas

> - Raspberry (eth0) ↔ **Switch** del grupo  

> - Notebooks ↔ **Switch** (modo *Obtener IP automáticamente*)

---
### Configurar IP estática en la Raspberry

> [!note] Raspberry Pi OS / Debian con **dhcpcd**

Editar `dhcpcd`:

```bash

sudo nano /etc/dhcpcd.conf

```

Agregar al final:

```ini

interface eth0

static ip_address=192.168.{{GRUPO}}.1/24

static domain_name_servers=1.1.1.1 8.8.8.8

```

Aplicar y verificar:

```bash

sudo systemctl restart dhcpcd

ip -4 a show dev eth0

```

---
### Servidor DHCP—opción A (ISC-DHCP-Server)

Instalar:

```bash

sudo apt update

sudo apt install -y isc-dhcp-server

```

Fijar interfaz:

```bash

sudo nano /etc/default/isc-dhcp-server

```

```bash

INTERFACESv4="eth0"

```

Configurar pool:

```bash

sudo nano /etc/dhcp/dhcpd.conf

```

```conf

default-lease-time 600;

max-lease-time 7200;

authoritative;

  

subnet 192.168.{{GRUPO}}.0 netmask 255.255.255.0 {

  range 192.168.{{GRUPO}}.100 192.168.{{GRUPO}}.200;

  option routers 192.168.{{GRUPO}}.1;

  option subnet-mask 255.255.255.0;

  option broadcast-address 192.168.{{GRUPO}}.255;

  option domain-name-servers 1.1.1.1, 8.8.8.8;

}

```

Levantar y chequear:

```bash

sudo systemctl enable --now isc-dhcp-server

sudo systemctl status isc-dhcp-server

sudo journalctl -u isc-dhcp-server -f

sudo tail -n 50 /var/lib/dhcp/dhcpd.leases

```

---
### Servidor DHCP—opción B (dnsmasq)

> [!warning] No usar junto con ISC al mismo tiempo.

Instalar:

```bash

sudo apt update

sudo apt install -y dnsmasq

```

Crear config:

```bash

sudo nano /etc/dnsmasq.d/lan.conf

```

```conf

interface=eth0

bind-interfaces

dhcp-range=192.168.{{GRUPO}}.100,192.168.{{GRUPO}}.200,255.255.255.0,12h

dhcp-option=3,192.168.{{GRUPO}}.1

dhcp-option=6,1.1.1.1,8.8.8.8

```

Habilitar y chequear:

```bash

sudo systemctl enable --now dnsmasq

sudo systemctl status dnsmasq

sudo tail -n 50 /var/lib/misc/dnsmasq.leases

```

---
### Pruebas y verificación

- **Renovar IP en cliente**  

  - Windows: `ipconfig /release` → `ipconfig /renew`  

  - Linux/macOS: desactivar/activar NIC o `nmcli dev reapply <nic>`

- **Comprobar**: IP `192.168.{{GRUPO}}.100–200`, GW `192.168.{{GRUPO}}.1`, DNS `1.1.1.1/8.8.8.8`

- **Ping a la Raspberry**:

```bash

ping -c 3 192.168.{{GRUPO}}.1

```

---
### Diagrama en Packet Tracer

**Debe incluir:**

 - 1 **Switch** (ej. 2960)

 - 1 **Server** (simula la Raspberry) con IP `192.168.{{GRUPO}}.1`

 - 2–4 **PCs** en **DHCP**

 - Cables **straight-through** al switch

**Sugerencia de config del Server (Packet Tracer):**

  - **Config → FastEthernet**: IP `192.168.{{GRUPO}}.1`, máscara `255.255.255.0`

  - **Services → DHCP**:

  - Network: `192.168.{{GRUPO}}.0`

  - Mask: `255.255.255.0`

  - Default Gateway: `192.168.{{GRUPO}}.1`

  - DNS: `1.1.1.1`

  - Start IP / Max Users: rango equivalente `100–200`

  
**Guardar**: `DHCP_Grupo{{GRUPO}}_ApellidoNombre.pkt`  

**Simulation Mode**: observar **DORA** (Discover, Offer, Request, ACK).

---

### Extra (opcional): Salida a Internet por NAT

Si `wlan0` tiene Internet y querés que **eth0** tenga salida:

**IP forward**:

```bash

echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-ipforward.conf

sudo sysctl -p /etc/sysctl.d/99-ipforward.conf

```

**NAT con nftables**:

```bash

sudo apt install -y nftables

sudo nft add table ip nat

sudo nft 'add chain ip nat postrouting { type nat hook postrouting priority 100 ; }'

sudo nft add rule ip nat postrouting oif "wlan0" masquerade

sudo systemctl enable --now nftables

sudo nft list ruleset | sed -n '1,120p'

```

---
### Solución de problemas

> [!bug] Checklist rápido

> - ¿Hay **otro** DHCP en el mismo dominio de broadcast? (Aislá el switch del grupo).

> - ¿La Raspberry **eth0** tiene `192.168.{{GRUPO}}.1/24`?

> - ¿El servicio corre en **eth0** y no en `wlan0`?

**Logs útiles**:

```bash

# ISC

sudo journalctl -u isc-dhcp-server -f

sudo tail -n +1 /var/lib/dhcp/dhcpd.leases

  

# dnsmasq

sudo journalctl -u dnsmasq -f

sudo cat /var/lib/misc/dnsmasq.leases

```

**Reservas por MAC (opcional)**  

ISC:

```conf

host notebook_juan {

  hardware ethernet AA:BB:CC:DD:EE:FF;

  fixed-address 192.168.{{GRUPO}}.50;

}

```

dnsmasq:

```conf

dhcp-host=AA:BB:CC:DD:EE:FF,192.168.{{GRUPO}}.50,12h

```

**Fuentes**  

- Debian – Servidor DHCP (ISC): https://servidordebian.org/es/buster/intranet/dhcp/server  

- Raspberry + dnsmasq: https://sobrebits.com/montar-un-servidor-casero-con-raspberry-pi-parte-3-configurar-servidor-dhcp/

---
## Conclusión


> Este TP fue, hasta ahora, el desafío **más complejo** del módulo de redes. No sólo implicó configurar correctamente la Raspberry como **servidor DHCP** para el segmento `192.168.{{GRUPO}}.0/24`, sino también **entender la dinámica real** de una LAN con switch, aislar el dominio de broadcast y documentarlo en Packet Tracer. La combinación de diseño, implementación y verificación (DORA, leases, pruebas de ping y parámetros IP) nos obligó a trabajar con criterios de producción: pasos claros, un único DHCP activo y controles de estado/logs.

**Lo más difícil** fue lograr que **todo el entorno conviva sin conflictos**. Tuvimos **muchos errores** en el camino; el más relevante fue un **conflicto con la red del colegio** (otro servidor DHCP presente en el mismo dominio de broadcast, interferencias con el router/AP institucional y rutas por defecto no deseadas). Ese choque generó síntomas confusos: clientes que obtenían IP fuera del rango, gateways incorrectos y pérdidas intermitentes de conectividad. **Gracias a la intervención del profesor**, pudimos diagnosticar y resolver el problema: aislamos físicamente el switch del grupo, verificamos que sólo nuestra Raspberry ofreciera DHCP en `eth0`, y confirmamos con los logs (_journalctl_ y archivos de _leases_) que los clientes recibían la configuración correcta.

Como medida central, adoptamos un **checklist** permanente: confirmar IP estática de la Raspberry en `192.168.{{GRUPO}}.1/24`, mantener **un único servidor DHCP** activo, usar el rango `192.168.{{GRUPO}}.100–200`, gateway `192.168.{{GRUPO}}.1`, DNS coherentes y verificar en clientes con `ipconfig /renew` o `nmcli`. Además, el diagrama en **Packet Tracer** se volvió clave para **comunicar y validar** la arquitectura y para observar DORA de forma controlada.

En síntesis, superamos el obstáculo más difícil la **convivencia con la infraestructura del colegio** aprendiendo a **aislar, medir y verificar** con criterios técnicos. Hoy contamos con una **LAN estable**, con **asignación automática de IP** y documentación clara; y dejamos sentadas buenas prácticas (log de eventos, un solo DHCP por dominio, pruebas de conectividad y _leases_) que nos preparan para futuras extensiones, como **reservas por MAC** o **NAT** hacia Internet cuando sea necesario.

>[!Resultados logrados]

- **Disponibilidad**: notebooks obtienen IP dentro del rango `192.168.{{GRUPO}}.100–200`.  

-  **Aislamiento por grupo**: cada mesa gestiona su propio segmento sin conflictos de DHCP.  

-  **Mantenibilidad**: configuraciones simples, versionables y fáciles de restaurar.  

-  **Escalabilidad**: sumar PCs al switch no requiere cambios en los clientes.
