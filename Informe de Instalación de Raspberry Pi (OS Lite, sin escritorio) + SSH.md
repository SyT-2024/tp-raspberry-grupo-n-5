

**Alumnos:** _Benavidez, Nuñez, Del Gesso, Pedone  y Alvarez_

**Curso y Docentes:** _6C Nicolas Falco y Teo Reyna_

**Fecha:** _12/09/2025_

## Lista de verificación

-  Descargar e instalar **Raspberry Pi Imager** en la PC.
    
- Seleccionamos el dispositivo *RASPBERRY PI 3*.
- Grabar **Raspberry Pi OS Lite (32-bit)** en la microSD.
- Seleccionar la *SDHC CARD* como almacenamiento
    
- Configurar la instalación del SO (*Raspberry Pi OS Lite 32-bits*): 
    
-  Primer arranque y **descubrir la IP** asignada por el router.
    
-  Conexión por **SSH** desde otra máquina.

## 1) Pasos especificados

**Teoría.** Una **Raspberry Pi** es una computadora de placa única (SBC) de bajo costo. Para servidores caseros/educativos conviene usar **Raspberry Pi OS Lite**, que es una distribución basada en Debian **sin entorno gráfico**, más ligera, con menor consumo de RAM/CPU y más estable para servicios.

**Práctica.**

1. Descargar **Raspberry Pi Imager**
    
2. Instalarlo en la PC.
    
3. Preparar: 
	- Nombre del anfitrión: Grupo5
	- Establecemos nombre de usuario y contraseña 
	- Establecemos los ajustes regionales
	- Activamos el SSH y ponemos la autenticación por contraseña
	- Desactivamos la telemetría 
	
4. Primer arranque y descubrir IP:
	- Para descubrir la IP escribimos en la terminal: ip a ó hostname -I
	- IP: 192.168.60.114

5. Conexión por SSH: 
	- Escribimos en la terminal: ssh grupo5@192.168.60.114 después ponemos la contraseña del grupo que se definió en el punto 3 *grupo52025*
## Fotos de cada punto:

1. ![[Pasted image 20250912113705.png]]
2.  ![[Pasted image 20250912113623.png]]
3. ![[Blank 5 Grids Collage.png]]
4. ![[Pasted image 20250912114325.jpg]]
5. ![[Pasted image 20250912114346.jpg]]
## Conclusión:

Con esta actividad aprendimos a instalar y poner en funcionamiento una Raspberry Pi desde cero. Pudimos seguir los pasos básicos: preparar la tarjeta de memoria, configurar el usuario y la contraseña, y finalmente conectarnos desde otra computadora usando SSH.

En la parte teórica entendimos que la Raspberry Pi es una computadora de bajo costo que, con un sistema operativo liviano, se puede usar como servidor o para distintos proyectos educativos y caseros.

Al principio tuvimos algunas confusiones, como equivocarnos en la versión del sistema que había que grabar, pero esos errores nos ayudaron a entender mejor el proceso y a corregirnos en grupo. En general, la práctica nos sirvió para conocer cómo se prepara y maneja una Raspberry Pi sin necesidad de monitor o teclado, y nos dio una primera experiencia de cómo se trabaja con estas herramientas en la vida real.


