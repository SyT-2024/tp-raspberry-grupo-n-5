# SERVICIO DHCP – Raspberry Pi (LAN 192.168.5.0/24)
---

## Índice

* [Introducción](#introducción)
* [Objetivo](#objetivo)
* [Concepto teórico: ¿Qué es DHCP y por qué lo usamos?](#concepto-teórico-qué-es-dhcp-y-por-qué-lo-usamos)
* [Desarrollo](#desarrollo)

  * [Plan de direccionamiento](#plan-de-direccionamiento)
  * [Configurar IP por NMCLI (Grupo 5)](#configurar-ip-por-nmcli-grupo-5)
  * [Servidor DHCP — Opción A (ISC-DHCP-Server)](#servidor-dhcp--opción-a-isc-dhcp-server)
  * [Pruebas y verificación](#pruebas-y-verificación)
  * [Archivo Packet Tracer (.pkt) en el repo](#archivo-packet-tracer-pkt-en-el-repo)
  * [Glosario de comandos utilizados (desglose)](#glosario-de-comandos-utilizados-desglose)
* [Conclusión](#conclusión)
* [Fuentes](#fuentes)

---

## Introducción

La **Raspberry Pi** funcionará como **servidor DHCP** de la LAN del grupo `192.168.5.0/24`.
La Raspberry (por **eth0**) y las notebooks se conectan al **switch** del grupo. La entrega incluye la documentación de implementación y verificación del servicio.

---

## Objetivo

Diseñar, implementar y **documentar** una LAN por grupo donde la **Raspberry Pi** actúe como **servidor DHCP** para `192.168.5.0/24`. La solución debe:

* Asignar **IP dinámicas** (rango sugerido `192.168.5.100–200`, gateway `192.168.5.1`, máscara `/24`, DNS públicos).
* Integrar **Raspberry + switch + notebooks** en una topología simple y funcional.
* **Verificarse** con pruebas (ping, leases) y dejar evidencia en el repositorio.

---

## Concepto teórico: ¿Qué es DHCP y por qué lo usamos?

**DHCP (Dynamic Host Configuration Protocol)** es un protocolo que **asigna automáticamente** a cada dispositivo de la red su **configuración IP**: dirección IP, máscara, gateway y DNS.
Sin DHCP, cada notebook tendría que configurarse **a mano**, lo que es **lento**, **propenso a errores** (IPs duplicadas, máscaras incorrectas) y **difícil de mantener**.
Con DHCP:

* Los equipos del grupo reciben **IP válidas** dentro del rango elegido (`192.168.5.100–200`).
* Se garantiza un **único gateway** (`192.168.5.1`) y **DNS coherentes**.
* La red es **más ordenada y escalable**: enchufar y usar (*plug and play*).

---

## Desarrollo

### Plan de direccionamiento

| Componente            | Valor                           |
| --------------------- | ------------------------------- |
| Red LAN               | `192.168.5.0/24`                |
| Raspberry (eth0)      | `192.168.5.1`                   |
| Rango DHCP (clientes) | `192.168.5.100 – 192.168.5.200` |
| Máscara               | `255.255.255.0`                 |
| Broadcast             | `192.168.5.255`                 |
| DNS sugeridos         | `1.1.1.1`, `8.8.8.8`            |

**Conexiones físicas**

* Raspberry (eth0) ↔ **Switch** del grupo
* Notebooks ↔ **Switch** (en *Obtener IP automáticamente*)

---

### Configurar IP por NMCLI (Grupo 5)

> **Objetivo:** Dejar **eth0** de la Raspberry con IP estática, gateway y DNS, usando **NetworkManager** (`nmcli`).
> Útil para administrar la interfaz sin tocar `dhcpcd.conf`.

```bash
sudo nmcli con add type ethernet ifname eth0 con-name lan-grupo5 ipv4.method manual ipv4.addresses 192.168.5.1/24
sudo nmcli con mod lan-grupo5 ipv4.gateway 192.168.5.1
sudo nmcli con mod lan-grupo5 ipv4.dns "1.1.1.1 8.8.8.8"
sudo nmcli con up lan-grupo5
```

**¿Qué hace cada comando?**

* `nmcli con add type ethernet ifname eth0 con-name lan-grupo5 ipv4.method manual ipv4.addresses 192.168.5.1/24`
  Crea una **conexión** llamada `lan-grupo5` para la interfaz **eth0**, con método IPv4 **manual** (IP estática) y dirección `192.168.5.1/24`.
* `nmcli con mod lan-grupo5 ipv4.gateway 192.168.5.1`
  Define el **gateway** (puerta de enlace) de esa conexión (aquí la propia Pi).
* `nmcli con mod lan-grupo5 ipv4.dns "1.1.1.1 8.8.8.8"`
  Asigna los **DNS** Cloudflare y Google a la conexión.
* `nmcli con up lan-grupo5`
  **Levanta/activa** la conexión recién configurada.

> **Nota:** Si gestionás **eth0** con `nmcli`, evitá que otros servicios reconfiguren la misma interfaz para no duplicar ajustes.

---

### Servidor DHCP — Opción A (ISC-DHCP-Server)

**Instalar:**

```bash
sudo apt update
sudo apt install -y isc-dhcp-server
```

**Seleccionar interfaz:**

```bash
sudo nano /etc/default/isc-dhcp-server
```

```bash
INTERFACESv4="eth0"
```

**Configurar pool:**

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

```conf
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.5.0 netmask 255.255.255.0 {
  range 192.168.5.100 192.168.5.200;
  option routers 192.168.5.1;
  option subnet-mask 255.255.255.0;
  option broadcast-address 192.168.5.255;
  option domain-name-servers 1.1.1.1, 8.8.8.8;
}
```

**Levantar y chequear:**

```bash
sudo systemctl enable --now isc-dhcp-server
sudo systemctl status isc-dhcp-server
sudo journalctl -u isc-dhcp-server -f
sudo tail -n 50 /var/lib/dhcp/dhcpd.leases
```

---

### Pruebas y verificación

En cada notebook (conectada al switch y en DHCP):

* **Renovar IP en cliente**

  * Windows: `ipconfig /release` → `ipconfig /renew`
  * Linux/macOS: desactivar/activar NIC o `nmcli dev reapply <nic>`

* **Comprobar parámetros recibidos:**

  * IP en `192.168.5.100–200`
  * Gateway `192.168.5.1`
  * DNS `1.1.1.1 / 8.8.8.8`

* **Conectividad con la Raspberry:**

```bash
ping -c 3 192.168.5.1
```

---

### Archivo Packet Tracer (.pkt) en el repo

El archivo de **Packet Tracer** ya está cargado en el repositorio y se llama **`Grupo5.pkt`**.
Sirve como evidencia del diseño y para observar el flujo **DORA** (Discover, Offer, Request, Ack) de forma controlada.

---

## Glosario de comandos utilizados (desglose)

**Edición y archivos**

* `nano RUTA/ARCHIVO`: abre el archivo en el editor de texto **nano**.

  * Guardar: `Ctrl+O` → Enter. Salir: `Ctrl+X`.

**Gestión de paquetes**

* `sudo apt update`: actualiza el índice de paquetes disponibles.
* `sudo apt install -y PAQUETE`: instala el paquete indicado sin pedir confirmación interactiva.

**Servicios (systemd)**

* `sudo systemctl enable --now SERVICIO`: habilita el servicio al arranque y lo inicia ya.
* `sudo systemctl status SERVICIO`: muestra el estado del servicio.
* `sudo systemctl restart SERVICIO`: reinicia el servicio.

**Logs**

* `sudo journalctl -u SERVICIO -f`: muestra los logs **en vivo** de ese servicio.

**Red**

* `ip -4 a show dev eth0`: muestra las direcciones IPv4 asignadas a `eth0`.
* `ping -c 3 IP`: envía 3 paquetes ICMP para probar conectividad.

**NetworkManager (nmcli)**

* `nmcli con add ...`: crea una **conexión** con parámetros (tipo, interfaz, método IPv4, IP/máscara).
* `nmcli con mod ...`: **modifica** parámetros de la conexión (gateway, DNS, etc.).
* `nmcli con up NOMBRE`: **activa** la conexión.

**DHCP (ISC)**

* Archivo `**/etc/default/isc-dhcp-server**`: define **en qué interfaz** escucha (p. ej., `eth0`).
* Archivo `**/etc/dhcp/dhcpd.conf**`: define el **pool** (rango, gateway, máscara, broadcast, DNS).
* `**/var/lib/dhcp/dhcpd.leases**`: lista de **concesiones** otorgadas (útil para verificar clientes).

---

## Conclusión

Este TP fue, hasta ahora, el desafío **más complejo** del módulo de redes. No sólo implicó configurar correctamente la Raspberry como **servidor DHCP** para `192.168.5.0/24`, sino también **entender la dinámica real** de una LAN con switch, aislar el dominio de broadcast y dejar evidencia clara en el repositorio. La combinación de diseño, implementación y verificación (DORA, leases, pruebas de ping y parámetros IP) nos obligó a trabajar con criterios de producción: pasos claros, un **único** DHCP activo y controles de estado/logs.

**Lo más difícil** fue lograr que **todo el entorno conviva sin conflictos**. Tuvimos errores durante el proceso; el más relevante fue un **conflicto con la red del colegio** (otro servidor DHCP en el mismo dominio de broadcast). **Gracias al profesor** pudimos diagnosticar y resolverlo: aislamos el switch del grupo, verificamos que sólo nuestra Raspberry ofreciera DHCP en `eth0` y confirmamos con logs y *leases* que los clientes recibían la configuración correcta.

Como medida central, adoptamos un **checklist** permanente: IP estática de la Raspberry en `192.168.5.1/24` (configurada con **NMCLI**), **un único servidor DHCP**, rango `192.168.5.100–200`, gateway `192.168.5.1`, DNS coherentes y verificación en clientes con `ipconfig /renew` o `nmcli`. El archivo **`Grupo5.pkt`** en el repo complementa la documentación técnica y permite replicar la topología y el proceso DORA de forma controlada.

En síntesis, superamos el obstáculo más difícil —la **convivencia con la infraestructura del colegio**— aprendiendo a **aislar, medir y verificar** con criterios técnicos. Hoy contamos con una **LAN estable**, con **asignación automática de IP** y documentación clara; y dejamos sentadas buenas prácticas (log de eventos, un solo DHCP por dominio, pruebas de conectividad y *leases*) que nos preparan para futuras extensiones del trabajo.

---

## Fuentes

* Debian – Servidor DHCP (ISC): [https://servidordebian.org/es/buster/intranet/dhcp/server](https://servidordebian.org/es/buster/intranet/dhcp/server)
* Sobrebits – Configurar DHCP en Raspberry (como referencia general): [https://sobrebits.com/montar-un-servidor-casero-con-raspberry-pi-parte-3-configurar-servidor-dhcp/](https://sobrebits.com/montar-un-servidor-casero-con-raspberry-pi-parte-3-configurar-servidor-dhcp/)
