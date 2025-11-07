# Instalación y Configuración del Servicio FTP en Raspberry Pi (vsftpd)

---

## Índice

1. Introducción
2. Objetivo del informe
3. Desarrollo teórico
   3.1. ¿Qué es FTP?
   3.2. Servidor vsftpd
   3.3. Modos de transferencia (activo/pasivo)
   3.4. FTPS (FTP sobre TLS)
   3.5. Clientes FTP: FileZilla y WinSCP
4. Desarrollo práctico (paso a paso)
   4.1. Preparación del sistema
   4.2. Instalación de vsftpd
   4.3. Estructura de directorios y permisos
   4.4. Configuración básica y segura de vsftpd
   4.5. Habilitar modo pasivo
   4.6. (Opcional) Habilitar FTPS con certificado autofirmado
   4.7. Pruebas de conectividad y transferencia
   4.8. Verificación de logs
5. Conclusión

---

## 1. Introducción

En esta etapa del proyecto, la Raspberry Pi continúa su evolución como servidor dentro de la red local del aula. Previamente se realizó la instalación limpia de **Raspberry Pi OS Lite**, se habilitó **SSH** para administración remota, se preparó el entorno gráfico mínimo con **Xorg** para pruebas de reenvío X y, en la entrega más reciente, se configuró la Raspberry como **servidor DHCP** de la red del grupo **192.168.5.0/24** (interfaz **eth0** conectada al switch del grupo) para asignar IPs dinámicamente a los clientes.

Sobre esa base, en este informe se documenta la instalación y configuración del **servicio FTP** utilizando **vsftpd** para permitir la transferencia de archivos entre los equipos de la LAN y la Raspberry, manteniendo buenas prácticas de seguridad y dejando evidencia de funcionamiento con clientes gráficos como **FileZilla** y **WinSCP**.

---

## 2. Objetivo del informe

* **Instalar** el servidor **vsftpd** en la Raspberry Pi.
* **Configurar** FTP en modo seguro para la **LAN del grupo** (192.168.5.0/24), con usuarios locales y “jaula” (chroot) por usuario.
* **Habilitar** el modo **pasivo** para compatibilidad con clientes detrás de NAT/firewall.
* **Probar** el acceso y transferencias utilizando **FileZilla** y **WinSCP**.
* (Opcional) **Implementar FTPS** (FTP explícito sobre TLS) con certificado autofirmado.
* **Registrar** los pasos, comandos y resultados, con explicaciones claras para su defensa oral.

---

## 3. Desarrollo teórico

### 3.1. ¿Qué es FTP?

**FTP (File Transfer Protocol)** es un protocolo de aplicación clásico para **transferir archivos** entre un cliente y un servidor a través de una red (Internet o LAN). Permite **listar directorios, subir, descargar y gestionar archivos** de forma remota. Por diseño, el FTP tradicional **no cifra** la comunicación; por eso se recomienda usarlo en **redes confiables** o añadir **TLS** (FTPS) cuando sea posible.

### 3.2. Servidor vsftpd

**vsftpd** (Very Secure FTP Daemon) es un servidor FTP conocido por su **seguridad, rendimiento y simplicidad**. Soporta usuarios locales del sistema, modo chroot (encerrar a cada usuario en su directorio), **modo pasivo**, **listas de control de acceso** y **TLS** para FTPS. Es la opción estándar en Debian/Raspberry Pi OS.

### 3.3. Modos de transferencia (activo/pasivo)

* **Activo**: el cliente abre conexión al puerto 21 del servidor y **el servidor** inicia la conexión de datos hacia un puerto aleatorio del **cliente**. Puede fallar si el cliente está detrás de un firewall.
* **Pasivo**: el cliente abre la conexión de control (21) y **también** abre la conexión de datos hacia un **rango de puertos** que el servidor publica. Es el modo recomendado en entornos con firewall/NAT y el más usado por clientes GUI.

### 3.4. FTPS (FTP sobre TLS)

**FTPS** agrega **cifrado TLS** al FTP. En modalidad **Explícita**, la sesión empieza en 21 y se negocia TLS. Aporta **confidencialidad e integridad** a credenciales y datos. En entornos educativos, un **certificado autofirmado** es suficiente para practicar, aunque los clientes mostrarán una advertencia la primera vez.

### 3.5. Clientes FTP: FileZilla y WinSCP

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

**Crear usuario dedicado (recomendado):**

```bash
sudo adduser --disabled-password --gecos "" ftpuser
sudo passwd ftpuser
```

**Explicación:**

* `adduser` crea un usuario del sistema; aquí **sin contraseña inicial** (se asigna luego) y sin datos interactivos (`--gecos ""`).
* `passwd ftpuser` define la contraseña que se usará para autenticarse por FTP.

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
sudo mkdir -p /srv/ftp/ftpuser
sudo chown -R ftpuser:ftpuser /srv/ftp/ftpuser
sudo chmod -R 755 /srv/ftp
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

**Parámetros clave (añadir/ajustar):**

```conf
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
xferlog_enable=YES
xferlog_std_format=YES

chroot_local_user=YES
allow_writeable_chroot=YES

user_sub_token=$USER
local_root=/srv/ftp/$USER

userlist_enable=YES
userlist_file=/etc/vsftpd.user_list
userlist_deny=NO
```

**Explicación:**

* **anonymous_enable=NO**: desactiva acceso anónimo.
* **local_enable=YES**: permite usuarios del sistema.
* **write_enable=YES**: habilita subir/borrar/renombrar.
* **local_umask=022**: permisos por defecto 755 para directorios/archivos nuevos.
* **xferlog_***: activa logs de transferencia.
* **chroot_local_user=YES**: encierra a cada usuario en su "home FTP".
* **allow_writeable_chroot=YES**: permite chroot con directorio escribible.
* **local_root** y **user_sub_token**: raíz FTP por usuario en `/srv/ftp/<usuario>`.
* **userlist_enable / userlist_deny=NO**: sólo usuarios listados podrán entrar.

**Autorizar usuarios:**

```bash
echo "ftpuser" | sudo tee -a /etc/vsftpd.user_list
```

**Reiniciar servicio:**

```bash
sudo systemctl restart vsftpd
```

### 4.5. Habilitar modo pasivo

**Añadir a `/etc/vsftpd.conf`:**

```conf
pasv_enable=YES
pasv_min_port=30000
pasv_max_port=30100
```

**Explicación:**

* Define un **rango acotado** de puertos de datos para el modo pasivo (útil para abrir en firewall si hace falta). En la LAN del laboratorio, suele funcionar sin cambios adicionales.

**Reiniciar:**

```bash
sudo systemctl restart vsftpd
```

### 4.6. (Opcional) Habilitar FTPS con certificado autofirmado

**Generar certificado:**

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
 -keyout /etc/ssl/private/vsftpd.pem \
 -out /etc/ssl/certs/vsftpd.pem
sudo chmod 600 /etc/ssl/private/vsftpd.pem
```

**Configurar TLS en `/etc/vsftpd.conf`:**

```conf
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
rsa_cert_file=/etc/ssl/certs/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.pem
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
```

**Reiniciar:**

```bash
sudo systemctl restart vsftpd
```

**Explicación:**

* Habilita **FTPS explícito** (TLS). Las credenciales y datos viajan cifrados. Con certificados autofirmados, los clientes mostrarán una advertencia inicial.

### 4.7. Pruebas de conectividad y transferencia

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

**Prueba rápida por consola (opcional):**

```bash
ftp 192.168.5.1
# Usuario: ftpuser
# Pass: (la definida)
```

**Transferencia:**

* En el cliente FTP: `put archivo.txt` para subir; `get archivo.txt` para descargar.

**Prueba con FileZilla (FTP simple o FTPS):**

* **Host**: `192.168.5.1`, **Puerto**: `21`.
* **Protocolo**: FTP; **Cifrado**: *Usar solo FTP simple (inseguro)* o **Requerir FTP explícito sobre TLS** (si se configuró FTPS).
* **Modo de acceso**: Normal; **Usuario**: `ftpuser`; **Contraseña**: (…)
* Conectar, crear carpeta de prueba y transferir un archivo.

**Prueba con WinSCP:**

* Protocolo: FTP; Servidor: `192.168.5.1`; Puerto: `21`; Usuario/Contraseña: (…)
* Elegir **TLS/Explicito** si se habilitó FTPS.

### 4.8. Verificación de logs

**Ubicaciones útiles:**

* Transferencias: `/var/log/vsftpd.log`
* Autenticación: `/var/log/auth.log`

**Comandos:**

```bash
sudo tail -n 50 /var/log/vsftpd.log
sudo tail -n 50 /var/log/auth.log
```

**Explicación:**

* `tail -n 50` muestra las últimas 50 líneas para confirmar inicio de sesión y archivos transferidos.

---

## 5. Conclusión

Se instaló y configuró el servicio **FTP** en la Raspberry Pi utilizando **vsftpd**, asegurando el acceso con **usuarios locales**, directorios "enjaulados" (**chroot**) y **modo pasivo** para compatibilidad en la LAN. Se documentaron las pruebas de conectividad y transferencia con **FileZilla** y **WinSCP**, y se dejaron rutas de **logs** para validar la operación. De manera opcional, se añadió **FTPS** mediante **TLS** para cifrar credenciales y datos, fortaleciendo la seguridad del servicio.

Con esta entrega, la Raspberry Pi consolida su rol como **servidor de archivos** dentro de la red del grupo **192.168.5.0/24**, complementando los servicios ya implementados (SSH, Xorg para pruebas y DHCP) y dejando una base sólida para despliegues y prácticas futuras en el laboratorio.
