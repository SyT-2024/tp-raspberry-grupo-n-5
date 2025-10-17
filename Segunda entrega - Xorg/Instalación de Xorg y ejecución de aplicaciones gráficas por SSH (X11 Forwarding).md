## Instalación de Xorg y ejecución de aplicaciones gráficas por SSH (X11 Forwarding)

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

<img width="1280" height="960" alt="image" src="https://github.com/user-attachments/assets/160e200a-1ab4-4f5a-8625-a4e08d0169d9" />


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

<img width="1280" height="960" alt="image" src="https://github.com/user-attachments/assets/9af549b7-fff7-459d-bf84-693739e587c6" />


### 5. Probar con una aplicación gráfica simple

Ejemplo con `xclock` (reloj gráfico, puede instalarse si no está):

`sudo apt install x11-apps -y xclock`

<img width="1280" height="960" alt="image" src="https://github.com/user-attachments/assets/08a92faa-7f01-4c5e-9599-365c7559275f" />


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

La instalación y configuración del servidor gráfico Xorg en la Raspberry Pi permitió comprender de forma práctica el funcionamiento del sistema de ventanas X11 y su integración con el protocolo SSH mediante el reenvío gráfico (X11 Forwarding).
Durante el proceso se comprobó que las aplicaciones gráficas ejecutadas desde la Raspberry mediante ssh -X se mostraban correctamente en el equipo cliente, utilizando los recursos gráficos de este último. Sin embargo, se observó que al ejecutarlas aparecía un gestor de ventanas en el entorno del cliente, a pesar de que la consigna especificaba que el servidor debía funcionar sin uno. Esto se debe a que el propio entorno gráfico del cliente es quien gestiona la visualización de las ventanas, incluso cuando el procesamiento principal ocurre en la Raspberry Pi.

A nivel técnico, se logró establecer una conexión remota segura y funcional, demostrando la capacidad de Xorg para separar el procesamiento y la representación gráfica. Esta arquitectura permite optimizar los recursos del servidor, reducir la carga sobre el hardware de la Raspberry y aprovechar la potencia gráfica del cliente.

En conjunto, la práctica permitió consolidar conocimientos sobre la configuración de servicios en Linux, el manejo del protocolo SSH, y la estructura modular del sistema X11.
El trabajo evidenció cómo Xorg actúa como una pieza clave dentro del modelo cliente-servidor para la ejecución de aplicaciones gráficas distribuidas, garantizando eficiencia y flexibilidad en entornos remotos.
