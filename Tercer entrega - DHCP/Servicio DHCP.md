# SERVICIO DHCP – Raspberry Pi (LAN 192.168.5.0/24)


---

## Índice

* [Introducción](#introducción)
* [Objetivo](#objetivo)
* [Desarrollo](#desarrollo)

  * [Plan de direccionamiento](#plan-de-direccionamiento)
  * [Concepto teórico: DHCP en nuestra LAN](#concepto-teórico-dhcp-en-nuestra-lan)
  * [Configurar IP por NMCLI (Grupo 5)](#configurar-ip-por-nmcli-grupo-5)
  * [Servidor DHCP — Opción A (ISC-DHCP-Server)](#servidor-dhcp--opción-a-isc-dhcp-server)
  * [Pruebas y verificación](#pruebas-y-verificación)
  * [Archivo Packet Tracer (.pkt) en el repo](#archivo-packet-tracer-pkt-en-el-repo)
  * [Glosario de comandos utilizados (desglose token-por-token)](#glosario-de-comandos-utilizados-desglose-token-por-token)
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

### Concepto teórico: DHCP en nuestra LAN

**DHCP (Dynamic Host Configuration Protocol)** es el protocolo que **asigna automáticamente** a cada dispositivo de la red su **configuración IP**: dirección IP, máscara, gateway y DNS.
Sin DHCP, cada equipo requeriría configuración **manual**, lo que es **lento**, **propenso a errores** (IPs duplicadas, máscaras mal puestas) y **difícil de mantener**.
En `192.168.5.0/24`, DHCP garantiza:

* **IP válidas** y **únicas** dentro de `192.168.5.100–200`.
* **Gateway único** `192.168.5.1` y **DNS coherentes**.
* Una experiencia **plug-and-play**: conectar y trabajar.

---

### Configurar IP por NMCLI (Grupo 5)

> Objetivo: Dejar **eth0** con IP estática, gateway y DNS usando **NetworkManager** (`nmcli`).

```bash
sudo nmcli con add type ethernet ifname eth0 con-name lan-grupo5 ipv4.method manual ipv4.addresses 192.168.5.1/24
sudo nmcli con mod lan-grupo5 ipv4.gateway 192.168.5.1
sudo nmcli con mod lan-grupo5 ipv4.dns "1.1.1.1 8.8.8.8"
sudo nmcli con up lan-grupo5
```
![275554d9-4c01-4842-a94e-e36e981f1884](https://github.com/user-attachments/assets/d2103c02-e7f8-4be3-8726-921e445a41ec)

---

### Servidor DHCP — (ISC-DHCP-Server)

**Instalar:**

```bash
sudo apt update
sudo apt install -y isc-dhcp-server
```
![671e8002-df89-448e-a76e-55c9bddb7153](https://github.com/user-attachments/assets/179f910a-4462-460f-ad9f-dc8564ded880)

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
![41c7f262-00b7-452c-8dd8-8598aa739958](https://github.com/user-attachments/assets/937fc19f-1212-45bc-bf82-ccea60a6e3a3)

**Levantar y chequear:**

```bash
sudo systemctl enable --now isc-dhcp-server
sudo systemctl status isc-dhcp-server
sudo journalctl -u isc-dhcp-server -f
sudo tail -n 50 /var/lib/dhcp/dhcpd.leases
```
![3414cb03-6d91-4c1d-a3f0-426da8c9176b](https://github.com/user-attachments/assets/e42fc4a6-8af9-4c9a-bda5-8ef4e70fe3dd)

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
![5740c2e2-be9e-4854-bf1f-a8dddc53e5fb](https://github.com/user-attachments/assets/581321fd-52f9-4f9b-8b2f-69f1e981fdb8)

* **Conectividad con la Raspberry:**

```bash
ping -c 3 192.168.5.1
```

---

### Archivo Packet Tracer (.pkt) en este archivo ''Tercer entrega - DHCP''
El archivo de **Packet Tracer** ya está cargado en eéste archivo y se llama **`Grupo5.pkt`**.
Sirve como evidencia del diseño y para observar el flujo **DORA** (Discover, Offer, Request, Ack) de forma controlada.

---

## Glosario de comandos utilizados (desglose token-por-token)

### Gestión de paquetes (APT)

**Comando:** `sudo apt update`

* `sudo`: ejecuta con privilegios de **superusuario** (root).
* `apt`: gestor de **paquetes** de Debian/Ubuntu/Raspberry Pi OS.
* `update`: **actualiza el índice** local de paquetes disponibles desde los repositorios.

**Comando:** `sudo apt install -y isc-dhcp-server`

* `sudo`: privilegios de superusuario.
* `apt`: gestor de paquetes.
* `install`: **instala** paquetes.
* `-y`: responde **“yes” automáticamente** a las confirmaciones.
* `isc-dhcp-server`: nombre del **paquete** a instalar.

---

### Edición de archivos

**Comando:** `sudo nano /etc/default/isc-dhcp-server`

* `sudo`: privilegios de superusuario (archivo del sistema).
* `nano`: editor de texto **en consola**.
* `/etc/default/isc-dhcp-server`: **ruta** del archivo a editar (define interfaz del servicio).

**Comando:** `sudo nano /etc/dhcp/dhcpd.conf`

* `sudo`: privilegios de superusuario.
* `nano`: editor de texto.
* `/etc/dhcp/dhcpd.conf`: archivo principal de **configuración** del servidor ISC DHCP.

---

### Servicios y logs (systemd / journal)

**Comando:** `sudo systemctl enable --now isc-dhcp-server`

* `sudo`: privilegios de superusuario.
* `systemctl`: herramienta para **administrar servicios** (systemd).
* `enable`: **habilita** el servicio en el **inicio** del sistema.
* `--now`: además de habilitar, **inicia inmediatamente**.
* `isc-dhcp-server`: nombre del **servicio**.

**Comando:** `sudo systemctl status isc-dhcp-server`

* `sudo`: privilegios de superusuario (para ver detalles completos).
* `systemctl`: administración de servicios.
* `status`: muestra **estado**, **PID**, **logs recientes**, etc.
* `isc-dhcp-server`: servicio a consultar.

**Comando:** `sudo journalctl -u isc-dhcp-server -f`

* `sudo`: privilegios de superusuario.
* `journalctl`: visor de **logs** de systemd.
* `-u isc-dhcp-server`: filtra logs por **unidad/servicio**.
* `-f`: sigue los logs **en vivo** (“follow”).

**Comando:** `sudo tail -n 50 /var/lib/dhcp/dhcpd.leases`

* `sudo`: privilegios de superusuario (archivo de sistema).
* `tail`: muestra el **final** de un archivo.
* `-n 50`: últimas **50** líneas.
* `/var/lib/dhcp/dhcpd.leases`: archivo de **concesiones** (leases) otorgadas.

---

### Red (iproute2, ping)

**Comando:** `ip -4 a show dev eth0`

* `ip`: herramienta de **iproute2** para red.
* `-4`: limita salida a **IPv4**.
* `a` (abreviatura de `addr`): muestra **direcciones**.
* `show`: **muestra** información.
* `dev`: especifica **dispositivo** de red.
* `eth0`: interfaz **Ethernet** principal.

**Comando:** `ping -c 3 192.168.5.1`

* `ping`: prueba de **conectividad** ICMP.
* `-c 3`: envía **3** paquetes y termina.
* `192.168.5.1`: **destino** (nuestra Raspberry en LAN).

---

### NetworkManager (nmcli)

**Comando (crear conexión):**
`sudo nmcli con add type ethernet ifname eth0 con-name lan-grupo5 ipv4.method manual ipv4.addresses 192.168.5.1/24`

* `sudo`: privilegios de superusuario.
* `nmcli`: interfaz de **línea de comandos** de NetworkManager.
* `con`: abreviatura de **connection**.
* `add`: **crea** una conexión nueva.
* `type ethernet`: tipo de conexión **Ethernet**.
* `ifname eth0`: interfaz **física** donde aplica (eth0).
* `con-name lan-grupo5`: **nombre** identificador de la conexión.
* `ipv4.method manual`: método IPv4 **manual** (estático).
* `ipv4.addresses 192.168.5.1/24`: dirección **IP/máscara** que asigna a eth0.

**Comando (modificar gateway):**
`sudo nmcli con mod lan-grupo5 ipv4.gateway 192.168.5.1`

* `mod`: **modifica** una conexión existente.
* `lan-grupo5`: nombre de la conexión a modificar.
* `ipv4.gateway 192.168.5.1`: define **gateway** IPv4.

**Comando (modificar DNS):**
`sudo nmcli con mod lan-grupo5 ipv4.dns "1.1.1.1 8.8.8.8"`

* `ipv4.dns`: lista de **servidores DNS** (separados por espacio).
* `"1.1.1.1 8.8.8.8"`: **Cloudflare** y **Google**.

**Comando (activar conexión):**
`sudo nmcli con up lan-grupo5`

* `up`: **levanta/activa** la conexión indicada.

---

## Conclusión

Este TP fue, hasta ahora, el desafío más exigente del módulo de redes. No se trató solo de poner a punto la Raspberry como servidor DHCP para 192.168.5.0/24, sino de comprender cómo funciona de verdad una LAN con switch, separar el dominio de broadcast y dejar todo bien documentado en el repositorio. El proceso completo —diseño, implementación y verificación (DORA, leases, ping y parámetros IP)— nos llevó a trabajar con una lógica “de producción”: pasos claros, un único DHCP activo y control permanente de estados y registros.

Lo que más nos costó fue lograr que todo conviva sin conflictos. Aparecieron errores y, en particular, un choque con la red del colegio (otro servidor DHCP en el mismo dominio de broadcast). Con la ayuda del profesor pudimos identificar el problema y resolverlo: aislamos el switch del grupo, confirmamos que solo nuestra Raspberry ofreciera DHCP por eth0 y validamos con los logs y los leases que los clientes recibían la configuración correcta.

Como aprendizaje clave, adoptamos un checklist fijo: IP estática de la Raspberry en 192.168.5.1/24 (configurada con NMCLI), un solo servidor DHCP, rango 192.168.5.100–200, gateway 192.168.5.1, DNS coherentes y verificación en clientes con ipconfig /renew o nmcli. El archivo Grupo5.pkt del repositorio acompaña la documentación y permite replicar la topología y el flujo DORA de manera controlada.

---

## Fuentes

* Debian – Servidor DHCP (ISC): [https://servidordebian.org/es/buster/intranet/dhcp/server](https://servidordebian.org/es/buster/intranet/dhcp/server)
* Sobrebits – Configurar DHCP en Raspberry (referencia general): [https://sobrebits.com/montar-un-servidor-casero-con-raspberry-pi-parte-3-configurar-servidor-dhcp/](https://sobrebits.com/montar-un-servidor-casero-con-raspberry-pi-parte-3-configurar-servidor-dhcp/)
