# Instalación y Configuración del Servicio FTP en Raspberry Pi (vsftpd)

---

## Índice

1. Introducción
2. Objetivo del informe
3. Desarrollo teórico
4. Desarrollo práctico (paso a paso)
   4.1. Preparación del sistema
   
   4.2. Instalación de vsftpd
   
   4.3. Estructura de directorios y permisos
   
   4.4. Configuración básica y segura de vsftpd
   
   4.5. Pruebas de conectividad y transferencia
   
5. Conclusión

---

## 1. Introducción

En esta etapa del proyecto, la Raspberry Pi continúa su evolución como servidor dentro de la red local del aula. Previamente se realizó la instalación limpia de **Raspberry Pi OS Lite**, se habilitó **SSH** para administración remota, se preparó el entorno gráfico mínimo con **Xorg** para pruebas de reenvío X y, en la entrega más reciente, se configuró la Raspberry como **servidor DHCP** de la red del grupo **192.168.5.0/24** (interfaz **eth0** conectada al switch del grupo) para asignar IPs dinámicamente a los clientes.

Sobre esa base, en este informe se documenta la instalación y configuración del **servicio FTP** utilizando **vsftpd** para permitir la transferencia de archivos entre los equipos de la LAN y la Raspberry, manteniendo buenas prácticas de seguridad y dejando evidencia de funcionamiento con clientes gráficos como **FileZilla** y **WinSCP**.

---

## 2. Objetivo del informe

* **Instalar** el servidor **vsftpd** en la Raspberry Pi.
* **Probar** el acceso y transferencias utilizando **FileZilla** y **WinSCP**.
* **Registrar** los pasos, comandos y resultados, con explicaciones claras para su defensa oral.

---

## 3. Desarrollo teórico

### 3.1. ¿Qué es FTP?

**FTP (File Transfer Protocol)** es un protocolo de aplicación clásico para **transferir archivos** entre un cliente y un servidor a través de una red (Internet o LAN). Permite **listar directorios, subir, descargar y gestionar archivos** de forma remota. Por diseño, el FTP tradicional **no cifra** la comunicación; por eso se recomienda usarlo en **redes confiables** o añadir **TLS** (FTPS) cuando sea posible.

### 3.2. Servidor vsftpd

**vsftpd** (Very Secure FTP Daemon) es un servidor FTP conocido por su **seguridad, rendimiento y simplicidad**. Soporta usuarios locales del sistema, modo chroot (encerrar a cada usuario en su directorio), **modo pasivo**, **listas de control de acceso** y **TLS** para FTPS. Es la opción estándar en Debian/Raspberry Pi OS.

### 3.3. Clientes FTP: FileZilla y WinSCP

* **FileZilla** (multiplataforma): interfaz gráfica para conectarse a servidores FTP/FTPS, arrastrar y soltar archivos, y gestionar sitios guardados.
* **WinSCP** (Windows): similar a FileZilla; además soporta SFTP/FTP/FTPS. Útil en laboratorios donde se trabaja con Windows.

---

## 4. Desarrollo práctico (paso a paso)

> **Contexto de red**: la Raspberry Pi actúa dentro de la LAN del grupo **192.168.5.0/24**. Su IP típica en este montaje es **192.168.5.1** sobre **eth0**, conectada al **switch** del grupo. Todas las pruebas de cliente se realizan desde notebooks conectadas al mismo switch.

### 4.1. Preparación del sistema

**Comandos:**

```bash
sudo apt update && sudo apt -y upgrade
```

**Explicación:**

* `apt update` refresca los índices de paquetes.
* `apt upgrade` aplica actualizaciones disponibles; `-y` confirma automáticamente.

### 4.2. Instalación de vsftpd

**Comandos:**

```bash
sudo apt -y install vsftpd
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
sudo systemctl status vsftpd --no-pager
```

**Explicación:**

* `apt install vsftpd` instala el servicio.
* `systemctl enable` lo habilita al arranque.
* `systemctl start` lo inicia ahora.
* `status` muestra si está activo y sin errores.

### 4.3. Estructura de directorios y permisos

**Comandos:**

```bash
sudo mkdir -p /home/grupo5/ftp
sudo chown root:root /home/grupo5
sudo chmod 755 /home/grupo5
sudo chown -R grupo5:grupo5 /home/grupo5
```

**Explicación:**

* `mkdir -p` crea la ruta si no existe.
* `chown` asigna propiedad a `ftpuser`.
* `chmod 755` (rwxr-xr-x) da lectura/ejecución global y escritura al dueño.

### 4.4. Configuración básica y segura de vsftpd

**Editar archivo principal:**

```bash
sudo nano /etc/vsftpd.conf
```
![09e0acdc-175f-4743-94ce-15c251150479](https://github.com/user-attachments/assets/9ef779fd-73f9-4a82-931e-9a9713e2f453)

**Parámetros clave (añadir/ajustar):**

```conf
local_enable=YES
write_enable=YES

```

**Explicación:**

* **local_enable=YES**: permite usuarios del sistema.
* **write_enable=YES**: habilita subir/borrar/renombrar.

**Reiniciar servicio:**

```bash
sudo systemctl restart vsftpd
```


### 4.5. Pruebas de conectividad y transferencia

**Verificar IP de la Raspberry (control):**

```bash
ip -4 a show dev eth0
```

**Explicación:**

* Lista direcciones IPv4 de `eth0`; esperamos IP **192.168.5.1** en este escenario.

**Desde una PC del mismo switch:**

```bash
ping 192.168.5.1
```

* Si responde, hay conectividad de red.

![5bb0d367-bb7f-41ed-88f9-89bd567ee3ef](https://github.com/user-attachments/assets/f8f78cf8-8b62-483e-a3bf-a5142f0b90f6)
---

## 5. Conclusión

Se instaló y configuró el servicio FTP en la Raspberry Pi utilizando vsftpd, con usuarios locales y pruebas de conectividad y transferencia mediante FileZilla y WinSCP. Durante las pruebas iniciales se detectó un inconveniente: FileZilla no permitía subir archivos porque en /etc/vsftpd.conf no estaba habilitada la opción de escritura (write_enable=YES). Tras habilitar write_enable=YES y reiniciar el servicio, las cargas funcionaron correctamente. 

Con esta entrega, la Raspberry Pi cumple el rol de servidor de archivos dentro de la red del grupo 192.168.5.0/24, complementando los servicios ya implementados (SSH, Xorg para pruebas y DHCP) y dejando una base clara para prácticas futuras en el laboratorio.
