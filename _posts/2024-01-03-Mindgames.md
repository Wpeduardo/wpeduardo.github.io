---
layout: single
title: Mindgames - TryHackMe
excerpt: "Debemos encontrar 2 bandera. Primero, a partir de un escaneo de puertos pude saber que había un servidor web que alojaba una aplicación web que contaba de un formulario con un campo donde podías ingresar código en BrainFuck que luego iba a ser traducido a código en Python e iba a ser ejecutado, por lo tanto, aprovechando que la librería os y sus métodos logro ejecutar comandos en el sistema operativo del sistema destino con el fin de ejecutar una reverse Shell. Luego, para la escalada de privilegio vertical, aproveche que el archivo binario openssl tenía asignado el capability `cap_setuid`, logrando obtener los privilegios de root. Además, el procedimiento de la escalada de privilegio vertical a partir del uso del binario openssl esta mejor detallada en el articulo `https://chaudhary1337.github.io/p/how-to-openssl-cap_setuid-ep-privesc-exploit/`."
date: 2024-01-03	
classes: wide
header:
  teaser: /assets/images/MindGame/image003.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - RCE
  - Brainfuck
  - Python
  - openssl
  - HTML
  - HTTP
  - Capability
  - JavaScript
---

![](/assets/images/MindGame/image001.png)

## SUMMARY

Debemos encontrar 2 bandera. Primero, a partir de un escaneo de puertos pude saber que había un servidor web que alojaba una aplicación web que contaba de un formulario con un campo donde podías ingresar código en BrainFuck que luego iba a ser traducido a código en Python e iba a ser ejecutado, por lo tanto, aprovechando que la librería os y sus métodos logro ejecutar comandos en el sistema operativo del sistema destino con el fin de ejecutar una reverse Shell. Luego, para la escalada de privilegio vertical, aproveche que el archivo binario openssl tenía asignado el capability “cap_setuid”, logrando obtener los privilegios de root. Además, el procedimiento de la escalada de privilegio vertical a partir del uso del binario openssl esta mejor detallada en el articulo https://chaudhary1337.github.io/p/how-to-openssl-cap_setuid-ep-privesc-exploit/.

## FASE RECONOCIMIENTO

En este caso debemos acceder al sistema destino y obtener la bandera. Primero, empezaremos utilizando el comando openvpn y el archivo de configuración OpenVPN con el fin de establecer una conexión VPN y poder acceder a los laboratorios donde están las máquinas virtuales y vulnerables.

![](/assets/images/MindGame/image005.png)

Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina Mindgames.

![](/assets/images/MindGame/image007.png)

## FASE ESCANEO Y ENUMERACION

Ahora pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del sistema destino. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo del sistema destino.

![](/assets/images/MindGame/image009.png)

Los resultados que obtenemos son:
- Un programa servidor SSH que se ejecuta en el puerto 22.
- Un programa servidor HTTP, que ha sido implementado en el lenguaje de programacion Go para manejar las operaciones HTTP, se ejecuta en el puerto 80, y que aloja un sitio web.

Ahora observaremos las caracteristicas y funcionalidades que pueda presentar el sitio web desde mi navegador web.

![](/assets/images/MindGame/image011.png)

Podemos observar que la unica pagina web de la aplicación web consta de varios mensajes que han sido construidos con el lenguaje de programacion esoterico Brainfuck, esto lo podemos comprobar utilizando esta herramienta en linea:

![](/assets/images/MindGame/image013.png)

Ademas, si converitmos el mensaje escrito en Brainkfuck y que hace referencia a la serie de Fibonacci en su forma original, obtendremos el codigo escrito Python, como se puede observar en la siguiente imagen.

![](/assets/images/MindGame/image015.png)

Ademas, si introducimos la serie de Fibonacci en su forma Brainfuck en el campo “Try before you buy”, obtenemos el resultado de la ejecucion de su codigo Python equivalnete debajo del mensaje “Program Output”

![](/assets/images/MindGame/image017.png)

Ademas, si analizamos el codigo fuente de esta pagina web, nos toparemos con un archivo JavaScript externo incrustado donde se tiene una funcion que realiza una solicitud POST hacia un endpoint de una API, que se encarga de ejecutar el codigo Python equivalente del codigo BrainFuck, y que devuelve la respuesta de la ejecucion en el cuerpo de la respuesta HTTP.

![](/assets/images/MindGame/image019.png)

![](/assets/images/MindGame/image021.png)

## FASE GANAR ACCESO O EXPLOTACION O HACKING

Ahora que conocemos la logica de la ejecucion de codigo BrainFuck realizada por la aplicación, utilizaremos el metodo “system” de la librería “os” para ejecutar comandos en el sistema operativo del sistema destino. Para ello lo traduciremos en codigo BrainFuck nuestro codigo Python.

![](/assets/images/MindGame/image023.png)

![](/assets/images/MindGame/image025.png)

Ahora que hemos confirmado que tenemos un RCE, cargaremos un script Shell en el sistema destino que me generara una reverse shell cuando lo ejecute.

### 1Paso: Cargar el script en el sistema destino.

![](/assets/images/MindGame/image027.png)

![](/assets/images/MindGame/image029.png)

### 2Paso: Asignacion del permiso de ejecucion al script Shell.

![](/assets/images/MindGame/image031.png)

![](/assets/images/MindGame/image033.png)

### 3Paso: Ejecucion del script Shell y la obtenecion de la reverse shell

![](/assets/images/MindGame/image035.png)

![](/assets/images/MindGame/image037.png)

De esta manera ganamos acceso al sistema destino. Ahora realizaremos un tratamiento de nuestra reverse shell para que sea mas interactiva y estable.

![](/assets/images/MindGame/image039.png)

## FASE ESCALADA PRIVILEGIOS

Ahora buscaremos vectores de escalada de privilegios en el sistema de archivos de manera manual.

![](/assets/images/MindGame/image041.png)

Podemos observar que el archivo binario openssl tiene asignado el capability cap_setuid, por lo tanto, se convierte en un vector de escalada de privilegio vertical que nos permite tener los privilegios de root. El procedimiento que seguiremos lo puedes encontrar en este articulo: `https://chaudhary1337.github.io/p/how-to-openssl-cap_setuid-ep-privesc-exploit/`.

![](/assets/images/MindGame/image043.png)

![](/assets/images/MindGame/image045.png)

De esta manera logramos realizar la escalada de privilegio vertical siendo el usuario root, y logre enecontrar las dos banderas en el sistema de archivos.










