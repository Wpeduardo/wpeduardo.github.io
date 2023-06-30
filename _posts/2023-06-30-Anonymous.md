---
layout: single
title: Anonymous - TryHackMe
excerpt: "En este caso debemos encontrar dos banderas ubicadas en la maquina `Anonymousv6`. Donde a partir del escaneo de puertos de `Nmap` pude saber que hay un servicio `FTP` y un servicio `SMB`. Ademas, a partir del script `ftp-anom.nse` pude saber que el servidor FTP admite conexiones anonimas y se enumero un directorio compartido. Luego, debido a que el directorio compartido tenia configurado el permiso de escritura para usuarios que pertenecen a un grupo diferente al grupo primario del propietario del directorio, se llego a subir un script modificado que contenia un codigo, que generaba una conexion reverse Shell hacia nuestro sistema. Ademas, el script tenia el mismo nombre que el script original por lo que su contenido se sobrescribira en el contenido del script original. Ademas, a partir de la suposicion de que este script es una tarea `cron` es que se obtuvo acceso al sistema objetivo. Luego, realizamos una escalada de privilegios para ser el usuario root aprovechando que el archivo binario `env` tiene el permiso `SUID` habilitado."
date: 2023-06-30
classes: wide
header:
  teaser: /assets/images/Anonymous/image003.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - SSH
  - FTP
  - Samba
  - SMBClient
  - Crontab
  - env
  - SUID
---

![](/assets/images/Anonymous/image001.png)

## SUMMARY

En este caso debemos encontrar dos banderas ubicadas en la maquina `Anonymousv6`. Donde a partir del escaneo de puertos de `Nmap` pude saber que hay un servicio `FTP` utilizando el puerto 21, un servicio `SMB` utilizando los puertos 139 y 445. Ademas, a partir del script `ftp-anom.nse` de `Nmap` pude saber que el servidor FTP admite conexiones anonimas y se enumero un directorio compartido, que contenia un script. Luego, debido a que el directorio compartido tenia configurado el permiso de escritura para usuarios que pertenecen a un grupo diferente al grupo primario del propietario del directorio, se llego a subir un script modificado que contenia un codigo, que generaba una conexion reverse Shell hacia nuestro sistema. Ademas, el script tenia el mismo nombre que el script original por lo que su contenido se sobrescribira en el contenido del script original. Ademas, a partir de la suposicion de que este script es una tarea `cron` es que se obtuvo acceso al sistema objetivo. Luego, realizamos una escalada de privilegios para ser el usuario root aprovechando que el archivo binario `env` tiene el permiso `SUID` habilitado.

## FASE RECONOCIMIENTO

Para resolver este ejercicio empezaremos utilizando el comando `openvpn` con el fin de establecer una conexion VPN con la red virtual donde esta la maquina `Anonymousv6`. Para ello utilizaremos el archivo de configuracion, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir informacion como la direccion del servidor VPN, los certificados y claves de seguridad, la configuracion de encriptacion, etc.

![](/assets/images/Anonymous/image005.png)

Luego de que se establece la conexion VPN se crea una interfaz virtual de red en nuestra maquina. Donde se enruta todo el trafico de red a traves de esa interfaz. 

![](/assets/images/Anonymous/image007.png)

Ademas, la plataforma de Tryhackme nos muestra la direccion ip de la maquina `Anonymousv6`.

![](/assets/images/Anonymous/image009.png)
## FASE ESCANEO Y ENUMERACION
Luego pasaremos a la fase de Escaneo y Enumeracion con el fin de poder escanear los puertos del nodo. Ademas, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la maquina `Anonymousv6`. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parametros:
- El parametro -sC o -script="default" para utilizar todos los scripts de la categoria default con el fin de realizar un escaneo y deteccion de los puertos de manera avanzada.
- El parametro -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parametro -n para evitar la resolucion DNS, y el parametro -max-rate para indicarle el numero maximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el trafico generado por la herramienta.
- El parametro -p- para realizar un escaneo de los 65535 puertos del nodo y el parametro -open con el fin de que nos muestre informacion solo de los puertos abiertos.

![](/assets/images/Anonymous/image011.png)

Los resultados que obtuvimos del escaneo vienen a ser:
* Hay un servicio `FTP` que se esta levantado en el puerto 21, y el programa servidor FTP que se esta corriendo. Ademas, la version del programa servidor FTP.
* A partir del script `ftp-anom.nse` de Nmap se llego a comprobar que el servidor FTP admite conexiones anonimas, es decir, permite que usuarios anonimos pueden acceder al servidor FTP, y sus recursos compartidos sin proporcionar credenciales de autenticacion. Ademas, nos lista o enumera los recursos compartidos del servidor al cual podemos acceder sin proveer credenciales en la autenticacion.
* Hay un servicio `SMB` que se esta levantado en el puerto 445 y 139.
* Hay un servicio `SSH`, que esta levantado en el puerto 22, y el programa servidor SSH que se esta corriendo. Ademas, la version del programa servidor SSH.

Ahora utilizaremos el script `smb-enum-shares.nse` de Nmap para poder enumerar los recursos compartidos del servidor SMB.

![](/assets/images/Anonymous/image013.png)

Podemos observar que el script enumero el recurso compartido `pics`, y se pudo acceder al recurso a traves de una sesion siendo el usuario guest. Ademas, el tipo del recurso nos indica que es un directorio en el cual esta configurado los permisos de lectura para los usuarios anonimos. Ademas, la ruta de donde se ha colocado este recurso compartido viene a ser el directorio home del usuario local `namelessone` en el sistema destino.

Ahora accederemos al recuso compartido `pics` a traves del programa cliente SMB `SMBClient`.

![](/assets/images/Anonymous/image015.png)

![](/assets/images/Anonymous/image017.png)

![](/assets/images/Anonymous/image019.png)

Podemos observar que el recurso compartido `pics` viene a ser un directorio que contiene dos imagenes de dos perros que no tienen importancia.

Ahora accederemos al servidor FTP a traves de una conexion anonima con el fin de observar el directorio compartido scripts. 

![](/assets/images/Anonymous/image021.png)

Podemos observar que el directorio scripts contiene archivo llamado `to_do.txt`. Donde se indica que au no esta deshabilita la configuracion de conexiones anonimas. Ademas, el directorio compartido contiene un script llamado `clean.sh`. Donde podemos observar su contenido a traves del comando less. Ademas, observamos que este script parece ser una tarea `cron` ya que va a eliminar los archivos del directorio tmp del sistema objetivo de una manera periodica. Ademas, los archivos, que eliminara, lo registraran en el registro `removed_files`, y en el caso de que no haiga archivos por eliminar en el directorio tmp se escribira el mensaje `Running cleanup script:  nothing to delete` en el registro.

Ahora suponiendo que este script es una tarea `cron` configurada por el usuario namelessone en el sistema objetivo. Crearemos otro script que me genera una conexion reverse Shell desde el sistema destino hacia nuestro sistema. Ademas, este script tendra el mismo nombre que el script `clean.sh`.

![](/assets/images/Anonymous/image023.png)

Ahora teniendo en cuenta que el directorio compartido `scripts` tiene habilitado el permiso escritura para los usuarios que pertenecen a un grupo diferente al grupo primario del propietario del directorio. Entonces, podemos cargar archivos en el directorio compartido. 

Ahora cargaremos nuestro script personalizado con el fin de que su contenido sobrescriba al contenido del script `clean.sh` original.

![](/assets/images/Anonymous/image025.png)

Ahora habilitaremos el puerto 1900 en nuestra maquina para que este en escucha y esperando la conexion entrante que se generara en el caso de que nuestra suposicion sea correcta. Para ello utilizamos la herramienta `netcat`. Ademas, utilizamos el comando `rlwrap` para que la Shell generada en nuestra maquina presente funcionalidades como el autocompletado de comandos, historial de comandos, etc.

![](/assets/images/Anonymous/image027.png)

![](/assets/images/Anonymous/image029.png)

De esta manera obtenemos acceso al sistema objetivo siendo el usuario `namelessone`. 

Ahora debemos buscar una vector de escalada de privilegios con el fin de ser un usuario con privilegios mas elevados. Para ello buscaremos archivos con permisos `SUID` con el fin de escalar privilegios, y llegar a ser el superusuario o root. Para ello ejecutamos el comando `find`, indicando que busque desde el directorio /, archivos con permisos `SUID`.

![](/assets/images/Anonymous/image031.png)

![](/assets/images/Anonymous/image035.png)

En esta ocasion utilizaremos el archivo binario `env` para la escalada de privilegios. Para ello debemos saber que este archivo binario es utilizado para ejecutar un script o comandos y sus parametros, con un interprete de comandos especifico como shell, bash. Por lo tanto, ejecutaremos el siguiente comando para ejecutar una Shell sh con los privilegios del usuario root a traves del parametro -p.

![](/assets/images/Anonymous/image037.png)

De esta obtenemos acceso al sistema objetivo siendo el usuario root.
 
Ahora buscaremos las dos banderas contenidas en los archivos `user.txt` y `root.txt`. Para localizar los archivos utilizaremos el comando find para buscar desde el directorio /, archivos 
con sus nombres.

![](/assets/images/Anonymous/image039.png)

![](/assets/images/Anonymous/image041.png)





 















 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 

 
 

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































