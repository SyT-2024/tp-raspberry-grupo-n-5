# Puerta de Enlace Wi-Fi→LAN en Raspberry Pi (NAT + DNS + DHCP activo)

## Índice
1. [Introducción](#introducción)  
2. [Objetivo del Informe](#objetivo-del-informe)  
3. [Desarrollo Teórico](#desarrollo-teórico)  
   - [Rol de la Raspberry como router](#rol-de-la-raspberry-como-router)  
   - [NAT/masquerade y DNS](#natmasquerade-y-dns)  
   - [Topología del grupo](#topología-del-grupo)  
4. [Desarrollo Práctico (con glosario en cada paso)](#desarrollo-práctico-con-glosario-en-cada-paso)  
   - [4.1 Conectar la Raspberry al Wi-Fi del WAP (wlan0)](#41-conectar-la-raspberry-al-wi-fi-del-wap-wlan0)  
   - [4.2 Verificar IP estática en LAN (eth0) y DHCP activo](#42-verificar-ip-estática-en-lan-eth0-y-dhcp-activo)  
   - [4.3 Habilitar el reenvío IP en el kernel](#43-habilitar-el-reenvío-ip-en-el-kernel)  
   - [4.4 Configurar NAT (masquerade) LAN→Wi-Fi](#44-configurar-nat-masquerade-lanwi-fi)  
   - [4.5 Ajustar/validar DNS para clientes](#45-ajustarvalidar-dns-para-clientes)  
   - [4.6 Pruebas de conectividad y navegación](#46-pruebas-de-conectividad-y-navegación)  
   - [4.7 Persistencia tras reinicio](#47-persistencia-tras-reinicio)  
   - [4.8 Alternativa: hotspot de teléfono](#48-alternativa-hotspot-de-teléfono)  
5. [Conclusión](#conclusión)

---

## Introducción
En esta etapa, la **Raspberry Pi** consolida su rol como **servidor central** de la LAN del aula. Ya contamos con **Raspberry Pi OS Lite** + **SSH**, **Xorg** mínimo para reenvío X y **servidor DHCP** operativo en la subred del grupo **192.168.5.0/24** sobre `eth0` (conectada al **switch**).  

En esta entrega sumamos **salida a Internet** para los clientes de la LAN, usando `wlan0` conectada al **Wireless AP** del laboratorio (o un **hotspot** de celular). La Raspberry actuará como **puerta de enlace** con **NAT** y **reenvío IP**, manteniendo la configuración de **DHCP** de la entrega anterior.

## Objetivo del Informe
- Conectar `wlan0` al WAP (**SSID:** `SyT_2023`, **PWD:** `sistTele2023`) o a un hotspot.  
- Mantener `eth0` en **192.168.5.1/24** y **DHCP** activo (entrega anterior).  
- Habilitar **IP forwarding** y **NAT (masquerade)** para tráfico LAN→Internet.  
- Asegurar **DNS funcional** para clientes.  
- Dejar **evidencia de pruebas** (ping, traceroute, navegación).

---

## Desarrollo Teórico

### Rol de la Raspberry como router
Un **router** conecta redes distintas y **reenvía** paquetes entre interfaces. Aquí:  
- **LAN:** `eth0` en **192.168.5.0/24** (Raspberry `192.168.5.1`, servidor DHCP).  
- **Salida:** `wlan0` asociada al **WAP**/hotspot (IP por DHCP del AP).

### NAT/masquerade y DNS
- **IP forwarding:** el kernel permite que un paquete entre por una interfaz y salga por otra.  
- **NAT (masquerade):** traduce IP privadas de la LAN a la IP externa de `wlan0` para que Internet pueda responder.  
- **DNS:** los clientes resuelven nombres (p. ej. `google.com`) vía servidores DNS (Cloudflare/Google o los que indique la cátedra).

### Topología del grupo
```
[ Notebooks ] --(switch)-- eth0 [ Raspberry Pi ] wlan0 )))) [ WAP/Hotspot ] ---> Internet
       LAN 192.168.5.0/24            (router NAT)
```

---

## Desarrollo Práctico (con glosario en cada paso)

### 4.1 Conectar la Raspberry al Wi-Fi del WAP (wlan0)

**Comandos**
```bash
nmcli radio wifi on
nmcli dev wifi list
sudo nmcli dev wifi connect "SyT_2023" password "sistTele2023" ifname wlan0

ip addr show wlan0
ip route
ping -c 3 1.1.1.1
ping -c 3 google.com
```

**Glosario (este paso)**
- `nmcli radio wifi on`: enciende la radio Wi-Fi.  
- `nmcli dev wifi list`: escanea y lista redes disponibles (verificar `SyT_2023`).  
- `nmcli dev wifi connect`: asocia **wlan0** al SSID y solicita IP por **DHCP** al AP.  
- `ip addr show wlan0`: verifica que **wlan0** está **UP** y con IP.  
- `ip route`: debe existir **default route** por **wlan0**.  
- `ping 1.1.1.1`: prueba de conectividad IP a Internet desde la Raspberry.  
- `ping google.com`: prueba de **DNS** + conectividad.

---

### 4.2 Verificar IP estática en LAN (eth0) y DHCP activo

**Comandos**
```bash
ip addr show eth0
sudo systemctl status isc-dhcp-server --no-pager

# (solo si hiciera falta reponer el perfil de red de la entrega anterior)
sudo nmcli con add type ethernet ifname eth0 con-name lan-grupo5 ipv4.method manual ipv4.addresses 192.168.5.1/24
sudo nmcli con mod lan-grupo5 ipv4.gateway 192.168.5.1
sudo nmcli con mod lan-grupo5 ipv4.dns "1.1.1.1 8.8.8.8"
sudo nmcli con up lan-grupo5
```

**Glosario (este paso)**
- `ip addr show eth0`: confirma IP **192.168.5.1/24** en `eth0`.  
- `systemctl status isc-dhcp-server`: verifica que el **DHCP** esté activo.  
- `nmcli con add/mod/up`: crea/modifica/levanta perfil estático para `eth0`.

---

### 4.3 Habilitar el reenvío IP en el kernel

**Comandos**
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-ipforward.conf
sudo sysctl --system
```

**Glosario (este paso)**
- `ip_forward=1`: habilita **IP forwarding** (reenvío entre interfaces).  
- `/proc/sys/...`: cambio **temporal** hasta reinicio.  
- `/etc/sysctl.d/...` + `sysctl --system`: cambio **persistente**.

---

### 4.4 Configurar NAT (masquerade) LAN→Wi-Fi

**Comandos**
```bash
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
```

**Glosario (este paso)**
- `-t nat -A POSTROUTING ... MASQUERADE`: traduce IPs privadas LAN a la IP de **wlan0**.  
- `FORWARD` con `NEW,ESTABLISHED,RELATED`: permite tráfico de ida y retorno entre `eth0` y `wlan0` con control de estado.

---

### 4.5 Ajustar/validar DNS para clientes

**Ejemplo en `dhcpd.conf` (si usaste *isc-dhcp-server*)**
```conf
option domain-name-servers 1.1.1.1, 8.8.8.8;
```

**Comandos**
```bash
sudo systemctl restart isc-dhcp-server

# En un cliente Linux:
nmcli con down <perfil> && nmcli con up <perfil>
# o
ip addr flush dev <iface> && sudo dhclient -r && sudo dhclient
```

**Glosario (este paso)**
- `option domain-name-servers`: DNS entregados por DHCP a los clientes.  
- `systemctl restart`: reinicia el servicio DHCP para aplicar cambios.  
- `nmcli con down/up` o `dhclient`: renueva concesión DHCP en el cliente.

---

### 4.6 Pruebas de conectividad y navegación (desde un cliente de la LAN)

**Comandos**
```bash
ip addr
ip route
ping -c 3 192.168.5.1
ping -c 3 1.1.1.1
ping -c 3 google.com
traceroute 8.8.8.8
```

**Glosario (este paso)**
- `ip addr`: confirma IP **192.168.5.x/24** recibida por DHCP.  
- `ip route`: **default gateway** debe ser **192.168.5.1**.  
- `ping 192.168.5.1`: prueba puerta de enlace (Raspberry).  
- `ping 1.1.1.1`: prueba salida a Internet por **IP**.  
- `ping google.com`: prueba **DNS** + salida.  
- `traceroute`: primer salto **192.168.5.1**; luego red del WAP/hotspot.

---

### 4.7 Persistencia tras reinicio (iptables y sysctl)

**Comandos**
```bash
sudo apt install -y iptables-persistent
sudo netfilter-persistent save
sudo systemctl enable netfilter-persistent
```

**Glosario (este paso)**
- `iptables-persistent` / `netfilter-persistent`: guardan y restauran reglas **iptables** al iniciar.  
- `enable`: activa restauración automática al boot.  
- (Recordatorio) `99-ipforward.conf`: asegura **ip_forward=1** permanente.

---

### 4.8 Alternativa: hotspot de teléfono

**Comandos (ejemplo)**
```bash
nmcli radio wifi on
sudo nmcli dev wifi connect "MiHotspot" password "ClaveDelCel" ifname wlan0
# El resto (IP forwarding, NAT, DNS, pruebas) es idéntico a los pasos anteriores.
```

**Glosario (este paso)**
- Igual que el paso **4.1**, solo cambia el **SSID/clave** del hotspot.

---

## Conclusión
Se habilitó **acceso a Internet** para la LAN **192.168.5.0/24** manteniendo el **DHCP** previo. La Raspberry, conectada por **wlan0** al **WAP** (o hotspot), opera como **router** gracias a **IP forwarding** y **NAT**. Se ajustó **DNS** para clientes y se verificó conectividad con **ping** y **traceroute**. Finalmente, se dejó **persistencia** de reglas para que la solución sobreviva a reinicios.  

**Resultado:** los equipos del grupo navegan a Internet a través de la Raspberry, cumpliendo los requisitos de la consigna.
