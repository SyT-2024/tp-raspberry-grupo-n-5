
**Grupo 5 – Integrantes:** Benavidez, Nuñez, Del Gesso, Pedone y Alvarez  
**Curso / Docentes:** 6C – Nicolás Falco y Teo Reyna  
**Fecha:** 19/09/2025

---

## Índice
1. Introducción  
2. Objetivo del Informe  
3. Desarrollo Teórico  
   - ¿Qué es Xorg?  
   - ¿Qué es el X11 Forwarding por SSH?  
   - Diferencia entre servidor gráfico, gestor de ventanas y escritorio  
4. Procedimiento paso a paso  
   - Requisitos del cliente (PC)  
   - Instalación mínima en Raspberry Pi  
   - Configuración de SSH para X11  
   - Prueba con una aplicación gráfica remota  
5. Comandos utilizados y explicación  
6. Conclusión  
7. Evidencias (capturas)

---

## 1) Introducción
En esta entrega instalamos **Xorg** (servidor gráfico) en una **Raspberry Pi OS Lite** sin entorno de escritorio ni gestor de ventanas. Configuramos **SSH con X11 Forwarding** para ejecutar aplicaciones gráficas en la Raspberry y **mostrarlas en el equipo cliente**, evitando que la Raspberry renderice video localmente.

---

## 2) Objetivo del Informe
- Instalar **Xorg** en la Raspberry Pi **sin** gestor de ventanas/escritorio.  
- Habilitar **X11 Forwarding** en SSH.  
- Ejecutar una app gráfica en la Raspberry y visualizarla **remotamente** en el cliente.

---

## 3) Desarrollo Teórico

### ¿Qué es Xorg?
**Xorg** (X.Org Server) implementa el sistema de ventanas **X11**. Proporciona la capa que permite a los programas gráficos dibujar ventanas y recibir eventos de teclado/mouse. No es un escritorio: es el “motor” gráfico.

### ¿Qué es el X11 Forwarding por SSH?
**SSH** puede tunelizar tráfico X11. Con **`-X`**/`-Y` el cliente levanta un **X server** local y la app remota (en la Raspberry) envía sus instrucciones gráficas por la conexión SSH para mostrarse **en la pantalla del cliente**.

### Diferencias clave
- **Servidor gráfico (Xorg)**: capa base que habla X11.  
- **Gestor de ventanas**: acomoda, mueve y decora ventanas (bordes/botones).  
- **Escritorio**: paneles, menú, iconos, apps preinstaladas.  
> En esta práctica instalamos **solo el servidor Xorg**.

---

## 4) Procedimiento paso a paso

### A. Requisitos del cliente (PC)
- **Linux/macOS**: ya traen cliente SSH y X server (en Linux).  
- **Windows**: instalar un X server como **VcXsrv** o **Xming**.  
  - VcXsrv recomendado (modo *Multiple windows*, “Disable access control” marcado).  
  - Luego, abrir VcXsrv antes de conectarse por SSH.

### B. Instalación mínima en Raspberry Pi
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y xserver-xorg xauth x11-apps
