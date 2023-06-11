---
layout: single
title: BountyHacker - TryHackMe
excerpt: "En este caso debemos encontrar dos banderas ubicadas en la maquina Bounty Hacker. Donde a partir de un escaneo de puertos utilizando los scripts de la categoria default de Nmap, descubri que el servicio FTP esta levantado en el puerto 21 del sistema objetivo.Ademas, a parti del script ftp-anom.nse de Nmap llegue a saber que el servidor FTP permitia conexiones anonimas. Por lo tanto, accedi a los recursos compartidos por el servidor FTP sin proveer credenciales. Luego, llegue a descargar dos recursos cuyo contenido me permitio realizar un cracking de contraseñas online hacia el servicio SSH del sistema objetivo utilizando Hydra. Llegando a encontrar unas credenciales validas. Luego logre acceso al sistema objetivo mediante el servicio SSH.Luego para la escalada de privilegios, aproveche que podia ejecutar el archivo binario tar con los privilegios del usuario root para ejecutar un comando que me permito ser el usuario root."
date: 2023-06-11
classes: wide
header:
  teaser: /assets/images/BountyHacker/image003.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - FTP
  - HTTP 
  - Hydra
  - SSH
  - Tar
  - Sudo
---

![](/assets/images/BountyHacker/image001.png)

## Summary

En este caso debemos encontrar dos banderas ubicadas en la maquina Bounty Hacker. Donde a partir de un escaneo de puertos utilizando los scripts de la categoria `default` de `Nmap`, descubri que el servicio FTP esta levantado en el puerto 21 del sistema objetivo.Ademas, a partir del script `ftp-anom.nse` de Nmap llegue a saber que el servidor FTP permitia conexiones anonimas. Por lo tanto, accedi a los recursos compartidos por el servidor FTP sin prover credenciales. Luego, llegue a descargar dos recursos cuyo contenido me permitio realizar un cracking de contraseñas online hacia el servicio `SSH` del sistema objetivo utilizando `Hydra`. Llegando a encontrar unas credenciales validas. Luego logre acceso al sistema objetivo mediante el servicio SSH.Luego para la escalada de privilegios, aproveche que podia ejecutar el archivo binario `tar` con los privilegios del usuario root para ejecutar un comando que me permito ser el usuario `root`.

## Fase Reconocimiento

Para resolver este ejercicio empezaremos utilizando el comando `openvpn` con el fin de establecer una conexion VPN con la red virtual donde esta la maquina Bounty Hacker. Para ello utilizaremos el archivo de configuracion, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir informacion como la direccion del servidor VPN, los certificados y claves de seguridad, la configuracion de encriptacion, etc. 

![](/assets/images/BountyHacker/image005.png)

Luego de que se establece la conexion VPN se crea una interfaz virtual de red en nuestra maquina. Donde se enruta todo el trafico de red a traves de esa interfaz.

![](/assets/images/BountyHacker/image007.png)

Ademas, la plataforma de Tryhackme nos muestra la direccion de la maquina Bounty Hacker.

![](/assets/images/BountyHacker/image009.png)

## Fase Ecaneo y Enumeracion

Luego pasaremos a la fase de Escaneo y Enumeracion con el fin de poder escanear los puertos del nodo. Ademas, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios levantados en los puertos abiertos de la maquina Bounty Hacker.Para ello utilizaremos `Nmap`.Donde le pasaremos los siguientes parametros:
- El parametro -sC o -script="default" para utilizar todos los scripts de la categoria default con el fin de realizar un escaneo y deteccion de los puertos de manera avanzada.
- El parametros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parametro -n para evitar la resolucion DNS, y el parametro --min-rate para indicarle el numero de paquetes por segundo que va utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el trafico generado por Nmap.
- El parametro -p- para realizar un escaneo de los 65535 puertos del nodo y el parametro --open con el fin de que nos muestre informacion solo de los puertos abiertos.

![](/assets/images/BountyHacker/image011.png)

![](/assets/images/BountyHacker/image013.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Hay un servicio `HTTP` que se esta levantado en el puerto 80, y el programa servidor HTTP que se esta corriendo.Ademas, la version del programa servidor HTTP.
- Hay un servicio `SSH` que se esta levantado en el puerto 22, y el programa servidor SSH se esta corriendo.Ademas, la version del programa servidor SSH.
- Hay un servicio `FTP`, que esta levantado en el puerto 21, y el programa servidor FTP que se esta corriendo.Ademas, la version del programa servidor FTP.
- Utilizando el script `ftp-anom.nse` de `Nmap` se llego a comprobar que el servidor FTP admite conexiones anonimas, es decir, permite que usuarios anonimos pueden acceder al servidor FTP, y sus recursos compartidos sin proporcionar credenciales de autenticacion. Ademas, nos lista o enumera los recursos compartidos del servidor.

Ahora accederemos al servidor FTP a traves de una conexion anonima con el fin de descargar los recursos compartidos `locks.txt` e `task.txt.`

![](/assets/images/BountyHacker/image015.png)

![](/assets/images/BountyHacker/image017.png)

![](/assets/images/BountyHacker/image019.png)

Ahora observaremos el contenido del recurso `locks.txt`.

![](/assets/images/BountyHacker/image021.png)

Vemos que se trata de una lista de palabras que podriamos utilizar para realizar un cracking de contraseñas online mediante ataque de fuerza bruta hacia el servicio SSH que se esta ejecutando en el puerto 22 de la maquina Bounty Hacker.

Ahora observaremos el contenido del recurso `task.txt`.

![](/assets/images/BountyHacker/image023.png)

Llegamos a obtener el autor o propietario de este recurso.Ademas, este nombre lo podriamos utilizar como username en el crackeo de contraseñas online mediante ataque de fuerza bruta hacia el servicio SSH que se esta ejecutando en el puerto 22 de la maquina Bounty Hacker.

## Fase Hacking o Ganar Acceso o Explotacion

Ahora realizare el cracking de contraseñas online sobre el servicio SSH utilizando la herramienta `Hydra`. Para ello le tengo ciertos parametros a la herramienta:
- El parametro -l para indicar el username que supuestamente conocemos.
- El parametro -P para indicar la lista de palabras que se utilizara para el ataque de fuerza bruta.
- La dirrecion ip de la maquina Bounty Hacker, y especificamos el servicio.

![](/assets/images/BountyHacker/image025.png)

Observamos que la herramienta llego a encontrar unas credenciales validas para el servicio SSH.

Ahora accederemos de manera remoto al sistema destino mediante el servicio SSH, y utilizando las credenciales(obtenidas anteriormente) en la autenticacion. 

![](/assets/images/BountyHacker/image027.png)

De esta manera llegamos obtener acceso al sistema objetivo siendo el usuario `lin`. 

## Fase Escalada de Privilegios

Ahora debemos buscar un vector de escalada de privilegios con el fin de llegar a ser el usuario root. Para ello utilizaremos el siguiente comando para observar que programas puedo ejecutar con el comando `sudo`, y con los privielgios del usuario root.

![](/assets/images/BountyHacker/image029.png)

Observamos que podemos ejecutar el archivo binario `tar` teniendo los privilegios del usuario root.

Ahora buscaremos en GTOBins una forma de poder aprovechar esto con el fin de escalar privilegios. 

![](/assets/images/BountyHacker/image031.png)

Llegue a encontrar un comando que nos permite escalar privilegios cuando el archivo binario `tar` se puede ejecutar con los privilegios del superusuario.

Ahora, ejecutaremos este comando en la maquina Bounty Hacker.

![](/assets/images/BountyHacker/image033.png)

De esta manera llegamos a ser el usuario root o superusuario.

Ahora buscaremos las dos banderas contenida en los archivos user.txt y root.txt. Para localizar los archivos utilizaremos el comando find para buscar desde el directorio /, archivos con nombre user.txt y root.txt.

![](/assets/images/BountyHacker/image035.png)

![](/assets/images/BountyHacker/image037.png)


















