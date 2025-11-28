# Puerta de Enlace Wi-Fi → LAN en Raspberry Pi (NAT + DHCP activo)

## Índice
1. Introducción  
2. Objetivo  
3. Desarrollo Teórico  
4. Desarrollo Práctico (con glosario)  
5. Pruebas  
6. Conclusión  

---

## 1. Introducción
En esta entrega configuramos la **Raspberry Pi** para que funcione como **puerta de enlace a Internet** de la red del grupo (**192.168.5.0/24**), manteniendo activo el **DHCP** configurado previamente en `eth0`.

Para lograr la salida a Internet:
- Conectamos **wlan0** a un Wi-Fi externo (hotspot de Windows).
- Activamos **IP forwarding** en la Raspberry.
- Configuramos **NAT (masquerade)**.
- Probamos la navegación desde un cliente conectado al switch.

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
La Raspberry Pi quedó configurada como **router NAT**, conectando la LAN del grupo a Internet mediante `wlan0`.  
Se activó el **IP forwarding**, se configuró **NAT**, se mantuvo el **DHCP** existente y se verificó la conectividad desde múltiples clientes.

**Resultado:**  
Los equipos de la LAN navegan a Internet correctamente a través de la Raspberry Pi, cumpliendo con los requisitos de la entrega final.
