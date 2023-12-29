---
layout: single
title: VulnNet:Internal - TryHackMe
excerpt: "Debemos encontrar 3 banderas. Primero, a partir de un escaneo de puertos pude saber que había un servidor NFS que alojaba un directorio compartido que contenía el archivo de configuración del programa Redis. Luego, con el valor del parámetro “requirepasss” logre autenticarme y volcar los datos de las bases de datos de Redis, logrando encontrar un mensaje codificado en Base64, cuyo contenido decodificado contenía unas credenciales para acceder al módulo compartido del Rsync, que contenía el directorio home de un usuario local, logrando cargar una clave publica SSH con el fin de acceder al sistema destino mediante la clave privada SSH del servicio SSH. Luego, en la escalada de privilegio vertical, pude saber que la herramienta TeamCity estaba siendo ejecutado por el usuario root, y en el puerto 8111 y que solo admitía conexiones provenientes del mismo sistema, por tal razón realice un Local Port Forwarding con el fin de poder acceder a su interfaz web, logrando autenticarme mediante una token de autenticación encontrado en los logs, y logrando ejecutar un comando que generara una reverse Shell con los privilegios de root."
date: 2023-12-29	
classes: wide
header:
  teaser: /assets/images/Pivoting/image003.jpg
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - TeamCity
  - Linpeas
  - Rsync
  - Python
  - Samba
  - SSH
  - NFS
  - Redis
  - Local Port Forwarding
---

![](/assets/images/Pivoting/image001.png)

## SUMMARY

Debemos encontrar 3 banderas. Primero, a partir de un escaneo de puertos pude saber que había un servidor NFS que alojaba un directorio compartido que contenía el archivo de configuración del programa Redis. Luego, con el valor del parámetro “requirepasss” logre autenticarme y volcar los datos de las bases de datos de Redis, logrando encontrar un mensaje codificado en Base64, cuyo contenido decodificado contenía unas credenciales para acceder al módulo compartido del Rsync, que contenía el directorio home de un usuario local, logrando cargar una clave publica SSH con el fin de acceder al sistema destino mediante la clave privada SSH del servicio SSH. Luego, en la escalada de privilegio vertical, pude saber que la herramienta TeamCity estaba siendo ejecutado por el usuario root, y en el puerto 8111 y que solo admitía conexiones provenientes del mismo sistema, por tal razón realice un Local Port Forwarding con el fin de poder acceder a su interfaz web, logrando autenticarme mediante una token de autenticación encontrado en los logs, y logrando ejecutar un comando que generara una reverse Shell con los privilegios de root.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener tres banderas. Primero, empezaremos utilizando el comando openvpn y el archivo de configuración OpenVPN con el fin de establecer una conexión VPN y poder acceder a los laboratorios donde están las máquinas virtuales y vulnerables.

![](/assets/images/Pivoting/image005.png)

Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina VulnNet Internal.

![](/assets/images/Pivoting/image007.png)

## FASE ESCANEO Y ENUMERACION

Ahora pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del sistema destino. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo del sistema destino.

![](/assets/images/Pivoting/image009.png)

![](/assets/images/Pivoting/image011.png)

![](/assets/images/Pivoting/image013.png)

Los resultados que obtenemos son:
- Un programa servidor SSH que se ejecuta en el puerto 22.
- Un servidor NFS que se ejecuta en el puerto 2049.
- El servicio rpcbind que se ejecuta en el puerto 111 y que ha asignado los puertos 38351,45025,59709,60651,2049 a diversos programas que se comunican con el protocolo RPC.
- El servicio SMB que se ejecuta en el puerto 139(donde el protocolo de transport utilizado es NetBT) y 445(donde el protocolo de transporte utilizado solamente es TCP)

Ahora enumeraremos los directorios compartidos por el servidor NFS.

![](/assets/images/Pivoting/image015.png)

Podemos observar que el directorio compartido es /opt/conf. Ademas, es sabido que el directorio opt viene a contener subdirectorios con todas las dependencias de aplicaciones y sofitwares de terceros instalados en el sistema, por tal razon, en el subdirectorio conf deberiamos encontrar archhivos de configuracion de aplicaciones o softwares de terceros.

Ahora montaremos este directorio compartdio en algun directorio de nuestro sistema de archivos.

![](/assets/images/Pivoting/image017.png)

Podemos obsevar que logramos montar el directorio compartido en un directorio de nuestro sistema. Ademas, se logro encontrar un archivo de configuracion que le corresponde al programa Redis, que puede ser util en el caso de que se haya establecido autenticacion para acceder al servidor Redis.

Ahora intentaremos acceder al servidor Redis, y enumeraremos sus bases de datos.

![](/assets/images/Pivoting/image019.png)

![](/assets/images/Pivoting/image021.png)

Podemos observar que se ha configuracion la autenticacion para acceder al servidor Redis. Ademas, si buscamos informacion en Hacktricks con respecto a como configurar la autenticacion de este servicio, nos indican que se puede establecer la autenticacion de dos formas, mediante el parametro requirepass para solicitar una contraseña, y con el parametro masteruser para solicitar un username y una contraseña. Ademas, ambos parametros se establecen en el archivo de configuracion, por tal motivo aprovecharemos en buscar esos parametros en nuestro archivo de configuracion.

![](/assets/images/Pivoting/image023.png)

![](/assets/images/Pivoting/image025.png)

Podemos observar que logramos encontrar el parametro `requirepass` con su valor configurado. Luego, logramos autenticarnos exitosamente, y logramos enumerar las bases de datos en el servidor Redis, logrando encontrar una base de datos con 5 claves, donde tres de esas claves almacena un valor del tipo string y los otros almacenan un valor tipo lista.

Ahora volcaremos los datos que almacenan estas claves.

![](/assets/images/Pivoting/image027.png)

![](/assets/images/Pivoting/image029.png)

Podemos observar que una de las clave del tipo string almacenaba una Flag. Ademas, una de las clave del tipo lista almacenaba varios strings iguales codificados en Base64, cuyo mensaje decodificado contenia unas credenciales correspondientes al programa Rsync. Ademas, debemos saber que este programa es utilizado para sincronizar y transferir archivos entre sistemas Linux/Unix.

Ahora para saber que archivos o directorios estan listos para la sincronizacion debemos enumerar los modulos que vienen a ser un conjunto de archivos y directorios que se pueden acceder y sincronizar por separado. Ademas, estos modulos pueden estar protegia por un mecanismo de autenticacion que solicita una contraseña.

![](/assets/images/Pivoting/image031.png)

Podemos observar que se tiene un modulo llamado files. Ademas, si intentamos enumerar los directorios o archivos que pueda contener, nos encontramos con la sorpresa que tiene habilitado un mecanismo de autenticacion. Por tal razon, ahora enumerarmos los recursos del modulo utilizando las credenciales encontradas anteriormente.

![](/assets/images/Pivoting/image033.png)

![](/assets/images/Pivoting/image035.png)

Podemos observar que los archivos y directorios que contiene el modulo files, viene a ser contenedor del directorio home de un usuario local lamado sys-internal. Por tal razon descargaremos el contenido del modulo files en un directorio local llamado `sys-internal_home`.

![](/assets/images/Pivoting/image037.png)

![](/assets/images/Pivoting/image039.png)

![](/assets/images/Pivoting/image041.png)

Podemos observar que logramos descargar todo los archivos y subdirectorios del directorio home del usario sys-internal. Ademas, dentro de el directorios, nos topamos con la segunda bandera, y con el directorio oculto .ssh, que contiene ninguna clave privada SSH. Ademas, nos encontramos con el directorio ocutlo .mozilla, e intentamos extraer las contraseñas almacenadas en los perfiles de usuarios en Firefox, pero no logro extraer nada.

## FASE GANAR ACCESO O EXPLOTACION O HACKING

Ahora aprovecharemos que podemos cargar archivos o directorios en los recurso del modulo, para cargar el archivo authorized_keys, que va contener una clave publica SSH que generaremos en nuestro sistema, y lo cargaremos en el subidrectorio .ssh del sistema destino. Todo ello con el fin de acceder al sistema destino mediante el servicio SSH y la clave privada SSH que generaremos.

![](/assets/images/Pivoting/image043.png)

![](/assets/images/Pivoting/image045.png)

![](/assets/images/Pivoting/image047.png)

De esta manera logramos obtener acceso al sistema destino siendo el usuario sys-internal y mediante la clave privada del servicio SSH.

## FASE ESCALADA PRIVILEGIOS

Ahora buscaremos vectores de escalada de privilegios de manera manual y automatizada con Linpeas en el sistema de archivos.

### Enumeracion de archivos donde el propietario pertenece al grupo root y que tenga el permiso de escritura y lectura habilitado para sys-internal, y archivos con el bit SUID habilitado.

![](/assets/images/Pivoting/image049.png)

### Enumeracion de tareas cron en el archivo crontab general.

![](/assets/images/Pivoting/image051.png)

### Enumeracion de puertos que estan esperando y recibiendo conexiones entrantes por todas las interfaces de red o solo provenientes mismo sistema.

![](/assets/images/Pivoting/image053.png)

![](/assets/images/Pivoting/image055.png)

![](/assets/images/Pivoting/image057.png)

Podemos observar que hay un subdirectorio inisual llamado TeamCity en el directorio raiz que le corresponde a una herramienta muy conocido que es utilizado por desarrolladores de software para constuir, probar y desplegar su codigo de maner automatica, y que viene a ser una plataforma para la integracion continua y entrega conitnua. Ademas, es sabido que esta herramienta consta de una interfaz web que utiliza el puerto 8111 por defecto para su comunicación, pero el problema esta en que solo admite conexiones entrantes provenientes del mismo sistema destino. Por tal razon realizremos un Local Port Forwarding con SSH.

![](/assets/images/Pivoting/image059.png)

![](/assets/images/Pivoting/image061.png)

Podemos observar que nos topamos con un login que nos solicita unas credenciales para acceder al portal de administracion de la herramienta. Debido a que podemos visualizar el directorio de la herramienta con todas sus dependencias y archivos, buscaremos en Internet donde se suele encontrar las credenciales de un usuario valido.

![](/assets/images/Pivoting/image063.png)

![](/assets/images/Pivoting/image065.png)

![](/assets/images/Pivoting/image067.png)

![](/assets/images/Pivoting/image069.png)

![](/assets/images/Pivoting/image071.png)

Podemos observar que logre encontrar un articulo(`https://exploit-notes.hdks.org/exploit/web/teamcity-pentesting/`) donde nos habla de autenticarnos a partir de un token de autenticacion de un Super user . A partir de ello logre encontrar 5 tokens de autenticacion del Super user en los logs de la herramienta, pero solamente con el ultimo token de autenticacion me logre autenticar exitosamente en el login. Ademas, en el articulo se especifica los pasos de como ejeuctar un comando que nos generara una reverse shell. Ademas, este reverse shell tendra los privilegios de root ya que la herramienta ha sido ejecutado bajo la cuenta del usuario root.

![](/assets/images/Pivoting/image073.png)

![](/assets/images/Pivoting/image075.png)

![](/assets/images/Pivoting/image077.png)

![](/assets/images/Pivoting/image079.png)

![](/assets/images/Pivoting/image081.png)

![](/assets/images/Pivoting/image083.png)

![](/assets/images/Pivoting/image085.png)

De esta manera logramos realizar una escalada privilegio vertical siendo el usuario root en el sistema destino. Ademas, se logro encontrar la ultima bandera.













