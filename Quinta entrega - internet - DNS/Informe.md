# Puerta de Enlace Wi-Fi → LAN en Raspberry Pi (NAT + DHCP activo)

## Índice
1. Introducción  
2. Objetivo  
3. Desarrollo Teórico  
4. Desarrollo Práctico (con glosario)  
5. Pruebas  
6. Conclusión  

---

## 2. Introducción

A lo largo de las entregas anteriores, la Raspberry Pi fue configurada progresivamente para cumplir distintos roles dentro de la red del aula. En la primera etapa se instaló **Raspberry Pi OS Lite** y se habilitó el acceso remoto mediante **SSH**, permitiendo la administración sin entorno gráfico. Posteriormente, se incorporó **Xorg** para habilitar el reenvío de aplicaciones gráficas a través de SSH.

En una siguiente entrega, la Raspberry asumió el rol de **servidor DHCP**, asignando direcciones IP dentro de la red del grupo **192.168.5.0/24** a través de la interfaz `eth0`. Luego, se instaló y configuró el servicio **FTP** utilizando vsftpd, permitiendo la transferencia de archivos entre los equipos del laboratorio y la Raspberry.

Con estas bases ya implementadas, en esta entrega la Raspberry Pi se expandirá para actuar además como **puerta de enlace** hacia Internet para los dispositivos conectados al switch del grupo.

---

## 2. Objetivo
- Conectar la Raspberry al Wi-Fi externo mediante `wlan0`.  
- Mantener `eth0` como LAN del grupo (192.168.5.1/24 + DHCP).  
- Activar **IP forwarding**.  
- Configurar **NAT**.  
- Verificar la navegación desde la LAN.

---

## 3. Desarrollo Teórico

### Raspberry como Router
La Raspberry conecta dos redes:
- `eth0` → LAN 192.168.5.0/24  
- `wlan0` → Internet  
Y reenvía paquetes entre ambas.

### IP Forwarding
Permite que un paquete entre por `eth0` y salga por `wlan0`.  
Es obligatorio para que funcione el enrutamiento.

### NAT (Masquerade)
Traduce las IP privadas de la LAN (192.168.5.x) a la IP pública de `wlan0`, permitiendo que Internet responda correctamente.

### DHCP
Asigna IPs del rango 192.168.5.x a los clientes conectados al switch.

---

## 4. Desarrollo Práctico (con glosario)

### 4.1 Conectar la Raspberry al Wi-Fi (wlan0)

**Archivo**
```bash
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

**Contenido**
```
network={
    ssid="NOMBRE_DEL_HOTSPOT"
    psk="CONTRASEÑA_DEL_HOTSPOT"
    key_mgmt=WPA-PSK
}
```

**Reinicio**
```bash
sudo reboot
```

**Verificación**
```bash
ip a
ip route
ping 8.8.8.8
```

**Corrección de rutas**
```bash
sudo ip route del default
sudo ip route add default via 192.168.137.1 dev wlan0
```

**Glosario**
- `wpa_supplicant.conf`: archivo donde se define el Wi-Fi.  
- `ip a`: muestra interfaces e IP.  
- `ip route`: muestra rutas de salida.  
- `ping`: prueba conectividad.  
- `route del/add`: corrige rutas por defecto.

---

### 4.2 Activar IP Forwarding
```bash
sudo nano /etc/sysctl.conf
```

Agregar:
```
net.ipv4.ip_forward=1
```

Aplicar:
```bash
sudo sysctl -p
```

**Glosario**
- `ip_forward`: habilita que la Raspberry reenvíe paquetes.
- `sysctl -p`: aplica cambios en el kernel.

---

### 4.3 Configurar NAT (masquerade)
```bash
sudo iptables -t nat -F
sudo iptables -t nat -A POSTROUTING -s 192.168.5.0/24 -o wlan0 -j MASQUERADE
sudo iptables -t nat -L
```

**Glosario**
- `iptables`: firewall para reglas de red.
- `POSTROUTING`: etapa donde se modifica la IP origen.
- `MASQUERADE`: usa la IP externa de wlan0 como IP pública.
- `-F`: limpia reglas previas.
- `-L`: lista reglas activas.

---

### 4.4 Verificar DHCP
```bash
sudo systemctl status isc-dhcp-server --no-pager
```

**Glosario**
- `isc-dhcp-server`: asigna IPs a los clientes de la LAN.

---

## 5. Pruebas

### En la Raspberry
```bash
ping 8.8.8.8
ping google.com
```

### En un cliente conectado al switch
```bash
ip a                 # debe tener 192.168.5.x
ip route             # default via 192.168.5.1
ping 192.168.5.1     # prueba gateway
ping 8.8.8.8         # salida a Internet
ping google.com      # DNS funcionando
```

---

## 6. Conclusión

Durante esta entrega se logró completar la configuración de la Raspberry Pi como puerta de enlace para la red del grupo, permitiendo que los equipos conectados al switch accedan a Internet a través de la interfaz inalámbrica `wlan0`. El proceso requirió no solo aplicar los conceptos de NAT, IP forwarding y administración de rutas, sino también resolver una serie de dificultades que surgieron durante la implementación práctica.

Uno de los principales inconvenientes fue la conexión inicial de `wlan0` al punto de acceso: la Raspberry se asociaba al Wi-Fi, pero no obtenía una dirección IP válida, lo que impedía cualquier comunicación. En varios intentos, la interfaz aparecía como *off/any*, lo que obligó a revisar cuidadosamente la configuración del archivo `wpa_supplicant.conf` y asegurarse de utilizar el SSID y la contraseña correctos del hotspot. Más adelante, se presentaron problemas al realizar pruebas de conectividad: la Raspberry tenía IP en `wlan0`, pero no podía hacer ping hacia Internet. Esto se debía a la presencia de dos rutas por defecto simultáneas (`eth0` y `wlan0`), lo que provocaba que el tráfico saliera por la interfaz equivocada. Fue necesario eliminar la ruta incorrecta y establecer manualmente la ruta por defecto hacia el gateway del hotspot.

Otro desafío importante fue la configuración de NAT. En los primeros intentos, la regla de iptables no incluía correctamente la subred 192.168.5.0/24, lo que impedía que la LAN pudiera acceder a Internet incluso cuando la Raspberry sí tenía salida. Además, surgió un conflicto con el servicio `dnsmasq`, que se activó de manera no intencional y mostraba errores al reiniciar; esto requería deshabilitarlo para evitar interferencias con el servicio DHCP configurado previamente.

Finalmente, hubo cálidas pruebas de funcionamiento donde el hotspot de Windows aparecía como conectado, pero sin compartir realmente Internet. Recién tras activar correctamente la función de “Zona con cobertura inalámbrica móvil” se pudo establecer una conexión estable que permitiera a la Raspberry convertirse en un router funcional.

A pesar de estas dificultades, cada problema fue identificado y resuelto, consolidando no solo el funcionamiento del sistema, sino también la comprensión de conceptos fundamentales de redes. Como resultado final, toda la LAN del grupo obtiene salida a Internet a través de la Raspberry Pi, cumpliendo plenamente con los objetivos de la entrega.
