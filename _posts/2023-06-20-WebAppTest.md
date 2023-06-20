---
layout: single
title: WebAppTest - TryHackMe
excerpt: "En este caso debemos encontrar una bandera ubicada en la maquina `WebAppTest`. Donde a partir del script `smb-enum-shares` de `Nmap` pudimos encontrar un directorio compartido por el servidor `Samba`.Ademas, en este directorio compartido encontramos un archivo, que contenia los usernames de los administradores del sistema objetivo.Luego, utilizando el script `http-enum` de `Nmap` pudimos encontrar un recurso escondido en el directorio raiz del servidor web.Ademas, este recurso era un directory list que contenia archivos, que contenian registros de las actividades de los administradores del sistema. Luego, utilizamos `hydra` para realizar un cracking de contraseñas onlinea contra el servicio `SSH`. Llegando a encontrar unas credenciales validas, y obteniendo acceso al sistema obejtivo.Luego, realizamos una escalada de privilegios para ser el usuario root aprovechando que el archivo binario `vim.basic` tiene el permiso `SUID` habilitado."
date: 2023-06-20
classes: wide
header:
  teaser: /assets/images/WebAppTest/image003.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - Samba
  - HTTP
  - Python
  - SSH
  - Vim.basic
  - Hydra
---

![](/assets/images/WebAppTest/image001.png)

## Summary

En este caso debemos encontrar una bandera ubicada en la maquina `WebAppTest`. Donde a partir del script `smb-enum-shares` de `Nmap` pudimos encontrar un directorio compartido por el servidor `Samba`.Ademas, en este directorio compartido encontramos un archivo, que contenia los usernames de los administradores del sistema objetivo.Luego, utilizando el script `http-enum` de `Nmap` pudimos encontrar un recurso escondido en el directorio raiz del servidor web.Ademas, este recurso era un directory list que contenia archivos, que contenian registros de las actividades de los administradores del sistema. Luego, utilizamos `hydra` para realizar un cracking de contraseñas onlinea contra el servicio `SSH`. Llegando a encontrar unas credenciales validas, y obteniendo acceso al sistema obejtivo.Luego, realizamos una escalada de privilegios para ser el usuario root aprovechando que el archivo binario `vim.basic` tiene el permiso `SUID` habilitado.

## Fase Reconocimiento

Para resolver este ejercicio empezaremos utilizando el comando `openvpn` con el fin de establecer una conexion VPN con la red virtual donde esta la maquina `WebAppTest`.Para ello utilizaremos el archivo de configuracion, que lo podemos descargar en la plataforma de `Tryhackme`, que puede incluir informacion como la direccion del servidor VPN, los certificados y claves de seguridad, la configuracion de encriptacion, etc. 

![](/assets/images/WebAppTest/image005.png)

Luego de que se establece la conexion VPN se crea una interfaz virtual de red en nuestra maquina. Donde se enruta todo el trafico de red a traves de esa interfaz.

![](/assets/images/WebAppTest/image007.png)

Ademas, la plataforma de Tryhackme nos muestra la direccion de la maquina `WebAppTest`.

![](/assets/images/WebAppTest/image009.png)

## Fase Escaneo y Enumeracion

Ahora escanearemos los puertos del sistema objetivo. Ademas, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios que se estan ejecutando en los puertos abiertos en el sistema `WebAppTest`. Para ello utilizaremos `Nmap`.Donde le pasaremos los siguientes parametros:
- El parametro -sC o -script="default" para utilizar todos los scripts de la categoria default con el fin de realizar un escaneo de los puertos de manera avanzada.
- El parametro -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parametro -n para evitar la resolucion DNS, y -max-rate para indicar numero de paquetes maximo por segundo que va utilizar Nmap para el escaneo de los puertos con el fin de evitar sobrecargar la red con el trafico generado por Nmap.
- El parametro -p- para realizar un escaneo de los 65535 puertos del sistema y el parametro -open con el fin de que la salida solo muestra informacion de los puertos abiertos.

![](/assets/images/WebAppTest/image011.png)

![](/assets/images/WebAppTest/image013.png)

![](/assets/images/WebAppTest/image015.png)

Los resultados,que obtuvimos del escaneo,son:
- Hay un servicio `HTTP` que se esta levantado en el puerto 80,8080,8009 y el  programa servidor `HTTP` que se esta corriendo.Ademas, la version del programa servidor `HTTP`.
- Hay un servicio `SSH` que se esta levantado en el puerto 22, y el  programa servidor `SSH` se esta corriendo.Ademas, la version del programa servidor `SSH`.
- Hay un servicio `Samba`, que esta levantado en el puerto 445.
- A partir del script `smb-security-mode.nse` de `Nmap` podimos saber que es posible autenticarnos al servidor `SMB` con el usuario `guest`.  

Ahora utilizare el script `smb-enum-shares.nse` de `Nmap` con el fin de poder saber si puedo enumerar los recursos compartidos del servidor SMB.

![](/assets/images/WebAppTest/image017.png)

Se llego a enumerar un par de recursos de compartidos del servidor SMB utilizando el usuario `guest` en la autenticacion.Ademas, observando el tipo del primer recurso compartido, deducimos que viene a ser un directorio. Ademas, los permisos de control de acceso que tenemos sobre el recurso compartido siendo el usuario guest son de `lectura` y `escritura`. 

Ahora, accederemos al recurso compartido para poder interactuar de manera interactiva con su contenido a traves de una interfaz de linea de comandos. 

![](/assets/images/WebAppTest/image019.png)

![](/assets/images/WebAppTest/image021.png)

Llegamos a observar que el directorio contiene un archivo llamado `staff.txt`. Ademas, con el comando `get` descargaremos este recurso en nuestro sistema.Observando el contenido del archivo. Podemos obtener dos `usernames` que los podemos usar en un cracking de contraseñas online usando ataques de fuerza bruta sobre un servicio, que se ejecuta en el sistema `WebAppTest`.

Ahora utilizare el script `http-enum.nse` de `Nmap` con el fin de realizar fuerza bruta de directorios con una lista de nombres posibles de directorios y archivos con el fin de encontrar archivos y directorios escondidos almacenados en el directorio raiz del servidor web, que se esta ejecutando en el puerto 80 del sistema objetivo.

![](/assets/images/WebAppTest/image023.png)

Llegamos a obtener un recurso escondido en el directorio raiz del servidor web.

Ahora accederemos al recurso `development` a traves de mi navegador web para observar su contenido

![](/assets/images/WebAppTest/image025.png)

![](/assets/images/WebAppTest/image027.png)

![](/assets/images/WebAppTest/image029.png)

Observamos que el recurso `development` viene a ser un directoy list que contiene dos archivos. El contenido del archivo `dev.txt` viene a ser un registro de los avances de los dos administradores, cuyas iniciales son `J` y `K`. El contenido del archivo `j.txt` viene a ser un mensaje de recomendacion hecha por el administrador `K` hacia el administrador `J` con el fin de que mejore la robustez de sus credenciales de inico de sesion hacia el sistema destino.Teniendo en cuenta el contenido de los dos archivos podemos deducir que los administradores del sistema destino son `Jan` y `Kay`. 

## Fase Hacking o Explotacion o Ganar Acceso

Ahora, utilizaremos estos dos usernames para realizar un cracking de contraseñas online hacia el servicio `SSH` que se ejecute en el sistema destino.Para ello utilizaremos la herramienta `Hydra` con los siguientes parametros:
- El parametro -L para indicar la lista de palabras que se utilizara para el ataque de fuerza bruta sobre el username.Ademas, esta lista de palabras va contener los nombres de los administradores en mayuscula y minuscula ya que la autenticacion en SSH distingue entre mayusculas y minusculas al ingresar las credenciales.
- El parametro -P para indicar la lista de palabras que se utilizara para el ataque de fuerza bruta sobre el password.
- El parametro -f para que se tenga el ataque de fuerza bruta cuando se encuentre unas credenciales validas.
- La dirrecion ip del sistema objetivo, y especificamos el servicio SSH.

![](/assets/images/WebAppTest/image031.png)

![](/assets/images/WebAppTest/image033.png)

![](/assets/images/WebAppTest/image035.png)


Observamos que la herramienta llego a encontrar unas credenciales validas para el servicio `SSH`.

Ahora accederemos al sistema destino mediante el servicio `SSH`, y utilizando las credenciales en la autenticacion.

![](/assets/images/WebAppTest/image037.png)

Observamos que hemos obtenido acceso al sistema destino mediante el servicio `SSH`, y siendo el usuario jan.

## Fase Escalada de Privilegios

Ahora buscaremos archivos con permisos `SUID` o `SGID` con el fin de escalar privilegios, y llegar a ser el superusuario o `root`. Para ello ejecutamos el comando `find`, indicando que busque desde el directorio /, archivos con permisos `SUID`.

![](/assets/images/WebAppTest/image039.png)

En esta ocasion utilizaremos el archivo binario `vim.basic` para la escalada de privilegios.Ademas, debemos saber que a traves de este archivo binario podemos utilizar el editor de texto de terminal `Vim`.Teniendo en cuento esto se utilizara el parametro `-c` con el fin de ejecutar comandos utilizando el editor de texto `Vim`, y con los privilegios del usuario `root`. 

![](/assets/images/WebAppTest/image041.png)

El comando que ejecutaremos viene a ser un comando de `Python` que nos devolvera una shell `sh` con los privilegios del usuario `root` o superusuario.

![](/assets/images/WebAppTest/image043.png)

De esta manera llegamos a ser el usuario root o superusuario.

Ahora buscaremos el contenido de la primera bandera almacenada en el archivo `pass.bak`.Para localizar el archivo utilizaremos el comando `find` para buscar desde el directorio /, archivos con nombre `pass.bak`.

![](/assets/images/WebAppTest/image045.png)























