## Índice

1. Introducción
    
2. Objetivo del Informe
    
3. Desarrollo Teórico
    
    - ¿Qué es Xorg?
        
    - ¿Qué es una Raspberry Pi?
        
    - ¿Qué es SSH con reenvío gráfico (X11 Forwarding)?
        
4. Pasos de Instalación y Configuración
    
    - Actualización del sistema
        
    - Instalación de Xorg
        
    - Configuración de SSH para reenvío gráfico
        
    - Verificación de funcionamiento
        
5. Explicación Detallada de los Comandos
    
6. Conclusión
    

---

## Introducción

En esta entrega se busca instalar y configurar el servidor gráfico **Xorg** en una Raspberry Pi sin gestor de ventanas ni entorno de escritorio. La finalidad es que las aplicaciones gráficas puedan ejecutarse de manera remota desde un cliente, utilizando el procesamiento gráfico del cliente y no de la Raspberry.

Esto permitirá trabajar con aplicaciones gráficas ligeras de forma eficiente, aprovechando el protocolo **X11 Forwarding** de **SSH**.

---

## Objetivo del Informe

El objetivo de esta práctica es lograr la instalación y configuración básica del servidor gráfico **Xorg** en la Raspberry Pi para que, mediante la conexión SSH, sea posible abrir aplicaciones gráficas en el cliente.

---

## Desarrollo Teórico

### ¿Qué es Xorg?

Xorg es un **servidor gráfico** que implementa el sistema de ventanas X11. Su función principal es permitir que las aplicaciones gráficas muestren ventanas y elementos gráficos, ya sea en el mismo dispositivo o remotamente en otro equipo.

### ¿Qué es una Raspberry Pi?

La Raspberry Pi es una **computadora de bajo costo y tamaño reducido**, utilizada principalmente en proyectos educativos y de desarrollo. Corre generalmente distribuciones basadas en Linux (como Raspberry Pi OS, derivado de Debian).

### ¿Qué es SSH con reenvío gráfico (X11 Forwarding)?

**SSH (Secure Shell)** es un protocolo que permite conectarse de manera segura a otro equipo.  
El **X11 Forwarding** es una extensión que permite enviar la interfaz gráfica de una aplicación desde el servidor al cliente, para que se ejecute gráficamente en el cliente aunque el proceso corra en el servidor.

---

## Pasos de Instalación y Configuración

### 1. Actualizar el sistema

`sudo apt update sudo apt upgrade -y`

### 2. Instalar Xorg

`sudo apt install xorg -y`

### 3. Habilitar el reenvío gráfico en SSH

Editar el archivo de configuración del servidor SSH:

`sudo nano /etc/ssh/sshd_config`

Dentro del archivo, asegurarse de que estén estas líneas:

`X11Forwarding yes X11DisplayOffset 10`

Reiniciar el servicio SSH:

`sudo systemctl restart ssh`

### 4. Conectarse desde el cliente con reenvío gráfico

Desde el equipo cliente (Linux o WSL con X11 instalado):

`ssh -X usuario@ip_raspberry`

### 5. Probar con una aplicación gráfica simple

Ejemplo con `xclock` (reloj gráfico, puede instalarse si no está):

`sudo apt install x11-apps -y xclock`

---

## Explicación Detallada de los Comandos

- **`sudo apt update`**: Actualiza la lista de paquetes disponibles en los repositorios.
    
- **`sudo apt upgrade -y`**: Instala las versiones más recientes de los paquetes ya instalados.
    
- **`sudo apt install xorg -y`**: Instala el servidor gráfico Xorg.
    
- **`sudo nano /etc/ssh/sshd_config`**: Abre el archivo de configuración del servidor SSH para modificarlo.
    
- **`X11Forwarding yes`**: Habilita el reenvío gráfico mediante X11 en SSH.
    
- **`sudo systemctl restart ssh`**: Reinicia el servicio SSH para aplicar cambios.
    
- **`ssh -X usuario@ip_raspberry`**: Conecta desde el cliente al servidor activando el reenvío gráfico.
    
- **`xclock`**: Ejecuta un programa gráfico de prueba que muestra un reloj.
    

---

## Conclusión

En este informe se instaló y configuró el servidor gráfico **Xorg** en una Raspberry Pi. Se habilitó el reenvío gráfico mediante SSH para que aplicaciones gráficas se ejecuten en el cliente sin necesidad de instalar un entorno de escritorio completo en el servidor.

Esto permite trabajar con aplicaciones gráficas de manera remota, reduciendo el consumo de recursos de la Raspberry Pi y delegando el procesamiento gráfico al equipo cliente.