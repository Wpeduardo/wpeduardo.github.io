---
layout: single
title: Wonderland - TryHackMe
excerpt: "En este caso debemos encontrar dos banderas ubicadas en la maquina Wonderland. Donde a partir del script http-title de Nmap pude saber el titulo de la pagina web que se aloja en el servidor web de la maquina Wonderland. Luego utilizo el script http-enum de Nmap y wfuzz para encontrar recursos ocultos a partir dell directorio raiz del servidor web Llegando a encontrar el recurso t que viene a ser una pagina web cuyo codigo HTML contenia unas credenciales.Luego con estas credenciales logre autenticarme mediante el servicio SSH al sistema objetivo. Luego para la escalada de privilegios, aproveche que podia ejecutar un script python con los privilegios del usuario rabbit para crear un script en python y llegar a ser el usuario rabibt. Luego utilize un archivo que tenia los permisos SUID y SGID habilitados para llegar ser el usuario hatter. Luego utilize el archivo binario perl que tenia asignado un capability con ciertos permisos para llegar ser el usuario root."
date: 2023-06-09
classes: wide
header:
  teaser: /assets/images/Wonderland/image003.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - HTTP
  - Wfuzz
  - Bash
  - SSH
  - Capability
  - Perl
  - Python
  - Sudo
---

![](/assets/images/Wonderland/image001.png)

## Summary

En este caso debemos encontrar dos banderas ubicadas en la maquina Wonderland. Donde a partir del script `http-title` de `Nmap` pude saber el titulo de la pagina web que se aloja en el servidor web de la maquina Wonderland.Luego, utilizo el script `http-enum` de `Nmap` y `wfuzz` para encontrar recursos ocultos a partir del directorio raiz del servidor web.Llegando a encontrar el recurso `t` que viene a ser una pagina web cuyo codigo `HTML` contenia unas credenciales.Luego, con estas credenciales logre autenticarme mediante el servicio `SSH` al sistema objetivo.Luego, para la escalada de privilegios aproveche que podia ejecutar un script en `Python` con los privilegios del usuario `rabbit` para crear un script en python y llegar a ser el usuario rabbit.Luego, utilize un archivo que tenia los permisos `SUID` y `SGID` habilitados para llegar ser el usuario `hatter`. Luego utilize el archivo binario `perl` que tenia asignado un `capability` con ciertos permisos para llegar a ser el usuario `root`.

## Fase Reconocimiento

Para resolver este ejercicio empezaremos utilizando el comando `openvpn` con el fin de establecer una conexion VPN con la red virtual donde esta la maquina Wonderland. Para ello utilizaremos el archivo de configuracion, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir informacion como la direccion del servidor VPN, los certificados y claves de seguridad, la configuracion de encriptacion, etc.

![](/assets/images/Wonderland/image005.png)

Luego de que se establece la conexion VPN se crea una interfaz virtual de red en nuestra maquina. Donde se enruta todo el trafico de red a traves de esa interfaz.

![](/assets/images/Wonderland/image007.png)

Ademas, la plataforma de Tryhackme nos muestra la direccion ip de la maquina Wonderland.

![](/assets/images/Wonderland/image009.png)

## Fase Ecaneo y Enumeracion

Luego pasaremos a la fase de Escaneo y Enumeracion con el fin de poder escanear los puertos del nodo. Ademas, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios levantados en los puertos abiertos de la maquina Wonderland.Para ello utilizaremos `Nmap`.Donde le pasaremos los siguientes parametros:
- El parametro -sC o -script="default" para utilizar todos los scripts de la categoria default con el fin de realizar un escaneo y deteccion de los puertos de manera avanzada.
- El parametros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parametro -n para evitar la resolucion DNS y el parametro --min-rate para indicarle el numero de paquetes por segundo que va utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el trafico generado por Nmap.
- El parametro -p- para realizar un escaneo de los 65535 puertos del nodo y el parametro --open con el fin de que nos muestre informacion solo de los puertos abiertos.

![](/assets/images/Wonderland/image011.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Hay un servicio HTTP que se esta levantado en el puerto 80, y el  programa servidor HTTP que se esta corriendo.Ademas, la version del programa servidor HTTP.
- A partir del script http-tittle.nse de Nmap podimos obtener el titulo de una pagina HTML que se aloja en el servidor web de la maquina Wonderland . Este resultado tambien lo podimos haber obtenido al realizar una solicitud GET HTTP al servidor web con el comando curl para obtener el cuerpo de la respuesta HTML del servidor web.

![](/assets/images/Wonderland/image013.png)

Observamos que el cuerpo de la respuesta HTML viene a ser el codigo HTML de la pagina web.Donde en la etiqueta title podemos observar el titulo de la pagina HTML que se obtuvo del script http-title.nse.
- Hay un servicio SSH que se esta levantado en el puerto 22, y el programa servidor SSH se esta corriendo.Ademas, la version del programa servidor SSH.

Ahora accederemos a la pagina web a traves de mi navegador web para observar su contenido.

![](/assets/images/Wonderland/image015.png)

Donde podemos verificar el titulo que encontramos anteriormente.

Ahora ejecutaremos los scripts de nmap de la categoria vuln con el fin de conocer las vulnerabilidades de los servicios que estan levantados en los puertos abiertos.

![](/assets/images/Wonderland/image017.png)

![](/assets/images/Wonderland/image019.png)

Podemos notar que se ha el utilizado script http-enum.nse de Nmap, que realiza un fuerza bruta de directorios con una lista de nombres posibles de directorios y archivos con el fin de encontrar archivos y directorios escondidos, llegandose a obtener el recurso r, que esta almacenado en el directorio raiz del servidor web.

Ahora accederemos al recurso `r` a traves de mi navegador web para observar su contenido.

![](/assets/images/Wonderland/image021.png)

![](/assets/images/Wonderland/image023.png)

Nos damos cuenta que el recurso viene a ser una pagina web, cuyo codigo HTML no contiene ninguna informacion relevante, pero el mensaje `Keep Going` nos incita a realizar fuerza bruta de directorios desde la ruta del recurso `r`  para encontrar recursos escondidos.Para ello utilizaremos la herramienta wfuzz ,que tambien realiza fuerza bruta de directorios utilizando un lista de posibles nombre de directorios y archivos. Ademas, le pasaremos los siguientes parametros a la herramienta:
- El parametro -hc con el fin de filtrar los recursos que tenga un codigo de estado HTTP 403,404,400.
- El parametro -w con el fin de indicar la ruta de la lista de posibles nombres de archivos y directorios,que utilizara la herramienta.
- El parametro -u con el fin de indicar la ruta donde hara el ataque de fuerza bruta de directorios.

![](/assets/images/Wonderland/image025.png)

![](/assets/images/Wonderland/image027.png)

Llegamos a obtener un nuevo recurso llamado `a`.
Ahora accederemos al recurso a a traves de mi navegador web para observar su contenido.

![](/assets/images/Wonderland/image029.png)

![](/assets/images/Wonderland/image031.png)

Nos damos cuenta que el recurso viene a ser una pagina web, cuyo codigo HTML no contiene ninguna informacion relevante, pero el mensaje `Keep Going` nos incita a seguir realizando fuerza bruta de directorios ahora desde la ruta del recurso `a`.Para ello seguiremos utilizando la herramienta wfuzz.

![](/assets/images/Wonderland/image033.png)

![](/assets/images/Wonderland/image035.png)

Llegamos a obtener un nuevo recurso llamado `b`.
Ahora accederemos al recurso a traves de mi navegador web para observar su contenido.
	
![](/assets/images/Wonderland/image037.png)

![](/assets/images/Wonderland/image039.png)

Nos damos cuenta que el recurso viene a ser una pagina web, cuyo codigo HTML no contiene ninguna informacion relevante, pero el mensaje `Keep Going` nos incita a seguir realizando fuerza bruta de directorios ahora desde la ruta del recurso `b`.Para ello seguiremos utilizando la herramienta wfuzz. 

![](/assets/images/Wonderland/image041.png)

![](/assets/images/Wonderland/image043.png)

Llegamos a obtener un nuevo recurso llamado `b`.
Ahora accederemos al recurso a traves de mi navegador web para observar su contenido.

![](/assets/images/Wonderland/image045.png)

![](/assets/images/Wonderland/image047.png)

Nos damos cuenta que el recurso viene a ser una pagina web, cuyo codigo HTML no contiene ninguna informacion relevante, pero el mensaje `Keep Going` nos incita a seguir realizando fuerza bruta de directorios ahora desde la ruta del recurso `b`.Para ello seguiremos utilizando la herramienta wfuzz.

![](/assets/images/Wonderland/image049.png)

Llegamos a obtener un nuevo recurso llamado `i`.
Ahora accederemos al recurso a traves de mi navegador web para observar su contenido.

![](/assets/images/Wonderland/image051.png)

![](/assets/images/Wonderland/image053.png)

Nos damos cuenta que el recurso viene a ser una pagina web, cuyo codigo HTML no contiene ninguna informacion relevante, pero el mensaje `Keep Going` nos incita a seguir realizando fuerza bruta de directorios ahora desde la ruta del recurso `i`.Para ello seguiremos utilizando la herramienta wfuzz.

![](/assets/images/Wonderland/image055.png)

Llegamos a obtener un nuevo recurso llamado `t`.
Ahora accederemos al recurso a traves de mi navegador web para observar su contenido.

![](/assets/images/Wonderland/image057.png)

![](/assets/images/Wonderland/image059.png)

Nos damos cuenta que el recurso viene a ser una pagina web, cuyo codigo HTML contiene un username y su password.

## Fase Hacking o Ganar Acceso o Explotacion

Ahora intentare autenticarme con estas credenciales mediante el servicio SSH que tiene el sistema objetivo.

![](/assets/images/Wonderland/image061.png)

![](/assets/images/Wonderland/image063.png)

Nos damos cuenta que hemos obtenido acceso a la maquina Wonderland mediante el servicio SSH y siendo el usuario alice. 

Ahora debemos buscar una vector de escalada de privilegios con el fin de llegar a ser el usuario root.

## Fase Escalada de Privilegios

Ahora utilizamos el siguiente comando para observar que programas puedo ejecutar con el comando sudo, y con los privilegios del usuario root.

![](/assets/images/Wonderland/image065.png)

Observamos que podemos ejecutar un script en python llamado walrus_and_the_carpenter con el archivo binario python3.6 pero teniendo los privilegios del usuario rabbit.

Ahora, observaremos el contenido del script con el editor de texto nano, y los permisos que tenemos sobre este script. 

![](/assets/images/Wonderland/image067.png)

![](/assets/images/Wonderland/image069.png)

![](/assets/images/Wonderland/image071.png)

Observamos que solo tenemos el permiso de lectura sobre el script, y que ademas utiliza la libreria random.

Nosotros debemos saber que Python antes de importar la libreria estandar random busca en el directorio, donde esta almacenado el script, con el fin encontrar un archivo llamado random.py y si no llegase a encontrar un archivo, entonces, utilizaria la libreria estandar. Aprovecharemos esto para crear un script en python que se llame random.py con el fin de que ejecute nuestro script en ves de la librearia estandar random.

Ahora crearemos nuestro script en python para que nos genere una nueva sesion una shell bash en el sistema.

![](/assets/images/Wonderland/image073.png)

![](/assets/images/Wonderland/image075.png)

Ahora ejecutaremos el script walrus_and_the_carpenter.py con el comando sudo y con su parametro -u para especificar que se ejecute el script siendo el usuario rabbit. De esta manera se ejecutara nuestro script random.py que nos generara una nueva sesion en el sistema siendo el usuario rabbit y utilizando la shell bash.

![](/assets/images/Wonderland/image077.png)

De esta manera llegamos a ser el usuario rabbit.

![](/assets/images/Wonderland/image079.png)

Luego,en el directorio home del usuario rabbit llegamos a encontrar un  archivo llamado teaParty. Ademas, observamos que el archivo tiene habilitado los permisos SUID y SGID. Por lo tanto podemos ejecutar este archivo teniendo los privilegios del propietario que viene a ser el usuario root.

Ahora ejecutaremos este archivo para ver que obtenemos.

![](/assets/images/Wonderland/image081.png)

Observamos que obtenemos un mensaje de poca importancia. 

Ahora observaremos los comandos que se ejecutan en segundo plano cuando ejecutamos el archivo. Para ello utilizaremos el comando cat.

![](/assets/images/Wonderland/image083.png)

Observamos que hay muchos cadenas de caracters que no son legibles para los seres humanos. Por lo tanto utilizaremos el comando strings para observar solo las cadenas de caracters que son legibles para los seres humanos.

![](/assets/images/Wonderland/image085.png)

Observamos que no tenemos el comando strings en el sistema remoto. 

Ahora vamos a transferir este archivo a mi sistema utilizando el comando scp que me permite transferir archivos utilizando el servicio SSH, pero primero debemos habilitar el servicio SSH en nuestro sistema. 

![](/assets/images/Wonderland/image087.png)

Luego, utilizamos el comando scp especificando la ruta donde esta nuestro archivo que queremos transferir. Ademas, debemos especificar un usuario y la dirrecion ip  de nuestro sistema con la ruta donde se almacenara el archivo que vamos a transferir. 

![](/assets/images/Wonderland/image089.png)

![](/assets/images/Wonderland/image091.png)

De esta manera obtengo el archivo teaParty en mi sistema.

Ahora utilizaremos el comando strings para observar solo las cadenas de caracteres que son legibles para los seres humanos.

![](/assets/images/Wonderland/image093.png)

Observamos que se utiliza el comando echo con su argumento -n para evitar el salto de linea despues de que se imprima el mensaje Probably by. Ademas, observamos que se utiliza el operador && para que se ejecute el comando date despues del comando echo, pero observamos que no se especifica la ruta completa del archivo binario ejecutable de date. Por lo tanto, podemos aprovechar como un vector de escalada de privilegios.

Ahora crearemos un script en bash que tenga de nombre date en el directorio tmp ya que es el unico donde tengo permiso de escritura es decir puedo crear,modificar o eliminar archivos o subdirectorios de ese directorio.

![](/assets/images/Wonderland/image095.png)

Ahora crearemos nuestro script date en bash que nos generara una nueva sesion en el sistema utilizando la shell bash.Ademas,tenemos que darle el permiso de ejecutar.

![](/assets/images/Wonderland/image097.png)

Ahora modificaremos la variable de entorno PATH. Donde le agregaremos la ruta del directorio tmp en la primera posicion de su lista de directorios que tiene la variable de entorno con el fin de cuando se quiera ejecutar el comando date en el archivo teaParty, el sistema busque primero en el directorio tmp el archivo binario ejecutable date, que vendria a ser nuestro script. 

![](/assets/images/Wonderland/image099.png)

Ahora ejecutaremos el archivo teaParty que tiene los permisos SUID y SGID para que se nos genere una nueva sesion siendo el usuario propietario del archivo y con una shell bash.

![](/assets/images/Wonderland/image101.png)

De esta manera llegamos a ser el usuario hatter.

Ahora debemos encontrar otro vector de escalada de privilegios que nos lleve a ser el usuario root. 

![](/assets/images/Wonderland/image103.png)

En el directorio home del usuario hatter llegamos a encontrar un archivo llamado password que posiblemente contenga la contraseña del usuario hatter.

Ahora utilizamos el siguiente comando para observar que programas puedo ejecutar con el comando sudo, y con los privielgios del usuario root.

![](/assets/images/Wonderland/image105.png)

Observamos que se ha configurado, que el usuario hatter, no ejecute ningun programa o comando con el comando sudo. 

Ahora utilizamos el siguiente comando para que nos enumere los archivos binarios a los cuales se le han asignado capabilities.

![](/assets/images/Wonderland/image107.png)

Observamos que a los archivos binarios perl y perl5.26.1 se le han asignado el capability cap_setuid con los permisos p y e, que le otoroga la capacidad a los archivos binarios de ejecutarse con los privilegios del usuario propietario del archivo.

Ademas, observamos que ambos archivos binarios tienen habilitado el permiso de ejecutar para los usuarios que pertenecen al grupo hatter. Por lo tanto, si podemos ejecutar cualquier de los dos archivos.
Aprovechando esto utilizaremos el siguiente comando que intentara ejecutar un shell interactivo ("/bin/sh") con los privilegios de root, al utilizar la funcion "setuid(0)" para cambiar el ID de usuario antes de ejecutar el shell. Esto permitira el acceso y la ejecucion de comandos con privilegios de superusuario. 

![](/assets/images/Wonderland/image109.png)

De esta manera llegamos a ser el usuario root.

Ahora buscaremos las dos banderas contenida en los archivos user.txt y root.txt. Para localizar los archivos utilizaremos el comando find para buscar desde el directorio /, archivos con nombre user.txt y root.txt.

![](/assets/images/Wonderland/image111.png)

![](/assets/images/Wonderland/image113.png)




