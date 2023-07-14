---
layout: single
title: Archangel - TryHackMe
excerpt: "Debemos encontrar dos banderas en la maquina `ARcHanG3l`. Donde a partir del escaneo de puertos de `Nmap` pude saber que hay un servicio `HTTP` y `SSH`.Ademas, con el script `http-title.nse` de Nmap supe que hay una página web almacenada en el servidor web. Donde encontré un nombre de dominio el cual estuvo asociado con otro sitio web. Luego, utilice `wfuzz` para realizar fuerza bruta de directorios sobre sitio web, llegando a encontrar una página web, que presentaba la vulnerabilidad `Path Trasnversal`. Luego, se observó el contenido del archivo `/etc/passwd` Además, utilizando `Burp Suite` pude observar el contenido de los archivos de registros del servicio web. Luego, modificando el valor del encabezado `User-Agent` con la extensión `User-Agent Switcher and Manager`, y utilizando la función `include` pudimos ejecutar comandos remotos en el sistema destino a través del parámetro GET `cmd`. Luego, realizamos una escalada de privilegios horizontal para ser el usuario `archangel` mediante una tarea cron. Luego, realizamos una escalada vertical para ser el usuario `root` mediante una manipulación de la variable `PATH`."
date: 2023-07-14	
classes: wide
header:
  teaser: /assets/images/Archangel/image003.png
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
  - HTTP
  - strings
  - User-Agent Switcher and Manager
  - Burp Suite
  - Crontab
  - Manipulacion Path
  - Wfuzz
  - Path Traversal
---

![](/assets/images/Archangel/image001.png)

## SUMMARY

Debemos encontrar dos banderas en la maquina `ARcHanG3l`. Donde a partir del escaneo de puertos de `Nmap` pude saber que hay un servicio `HTTP` y `SSH`.Ademas, con el script `http-title.nse` de Nmap supe que hay una página web almacenada en el servidor web. Donde encontré un nombre de dominio el cual estuvo asociado con otro sitio web. Luego, utilice `wfuzz` para realizar fuerza bruta de directorios sobre sitio web, llegando a encontrar una página web, que presentaba la vulnerabilidad `Path Trasnversal`. Luego, se observó el contenido del archivo `/etc/passwd` Además, utilizando `Burp Suite` pude observar el contenido de los archivos de registros del servicio web. Luego, modificando el valor del encabezado `User-Agent` con la extensión `User-Agent Switcher and Manager`, y utilizando la función `include` pudimos ejecutar comandos remotos en el sistema destino a través del parámetro GET `cmd`. Luego, realizamos una escalada de privilegios horizontal para ser el usuario `archangel` mediante una tarea cron. Luego, realizamos una escalada vertical para ser el usuario `root` mediante una manipulación de la variable `PATH`.

## FASE RECONOCIMIENTO

Para resolver este ejercicio empezaremos utilizando el comando `openvpn` con el fin de establecer una conexion VPN con la red virtual donde esta la maquina `ARcHanG3l`. Para ello utilizaremos el archivo de configuracion, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir informacion como la direccion del servidor VPN, los certificados y claves de seguridad, la configuracion de encriptacion, etc.

![](/assets/images/Archangel/image005.png)

Luego de que se establece la conexion VPN se crea una interfaz virtual de red en nuestra maquina. Donde se enruta todo el trafico de red a traves de esa interfaz. 

![](/assets/images/Archangel/image007.png)

Ademas, la plataforma de Tryhackme nos muestra la direccion ip de la maquina `ARcHanG3l`.

![](/assets/images/Archangel/image009.png)

## FASE ESCANEO Y ENUMERACION

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina `ARcHanG3l`. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:
- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de
manera avanzada.
- El parámetro -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap
para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos
abiertos.

![](/assets/images/Archangel/image011.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Hay un servicio HTTP que se está levantado en el puerto 80, y el programa servidor HTTP que se está corriendo. Además, la versión del programa servidor
- A partir del script http-tittle.nse de Nmap llegamos a saber que hay una página web que se aloja en el servidor web.
- Hay un servicio SSH, que esta levantado en el puerto 22, y el programa servidor SSH que se está corriendo. Además, la versión del programa servidor SSH

Ahora observaremos la pagina web alojada en el servidor web del sistema destino, que llegamos a encontrar con el script de Nmap, desde mi navegador web.

![](/assets/images/Archangel/image013.png)

![](/assets/images/Archangel/image015.png)

Podemos observar que la pagina web viene a ser la página de inicio de un sitio web, Además, revisando el código fuente de la página web, encontramos varias etiquetas de anclaje cuyos atributos href contienen enlaces de páginas web del sitio web, que no tienen mucha importancia. Además, en la pagina de
inicio del sitio web podemos encontrar un correo electrónico asociado a un nombre dominio llamado mafialive.com.

Ahora intentaremos acceder al sitio web asociado a ese nombre de dominio, pero para evitar que mi navegador web haga una consulta DNS hacia mi servidor DNS recursivo con el fin de saber la dirección IP asociada a ese nombre de dominio, debemos configurar este nombre de dominio asociada con la dirección IP del sistema destino en el archivo /etc/hosts.

![](/assets/images/Archangel/image017.png)

![](/assets/images/Archangel/image019.png)

Podemos observar que al acceder al nombre de dominio mafialive.thm obtenemos una pagina de inicio que nos indica que el sitio web está en desarrollo.

Ahora realizaremos fuerza bruta de directorios sobre el directorio del sitio web con el fin de encontrar un recurso escondido. Para ello utilizaremos wfuzz que viene a ser una herramienta automatizada que nos permite realizar fuerza bruta de directorios.

![](/assets/images/Archangel/image021.png)

Podemos observar que la herramienta llego a encontrar un recurso escondido llamado test.php

Ahora observaremos el contenido del recurso a través de mi navegador web.

![](/assets/images/Archangel/image023.png)

![](/assets/images/Archangel/image025.png)

![](/assets/images/Archangel/image027.png)

Podemos observar que el código fuente de la página web, contiene una etiqueta button. que deﬁne un botón. Además, cuando hacemos clic al boton se ejecuta el contenido del archivo mrrobot.php y su salida HTML se muestra en el navegador web. Además, podemos observar que se utiliza un archivo PHP llamado test. Donde lo más probable es que este archivo PHP utilice alguna función como include,require,include_once y require_once, para ejecutar contenido de los archivos almacenados en la ruta del directorio /var/www/html/development_testing y mostrar su salida HTMl a través del navegador web.Además, a través del valor del parámetro GET view vamos a indicarle el nombre del archivo, que queremos observar, a la función contenida en el archivo test.php.

Ahora probaremos si en el directorio development_testing esta almacenado el archivo test.php con el fin de poder observar su contenido a través del navegador web, pero antes debemos saber que no es posible observar el contenido de un archivo PHP sin utilizar el filtro PHP convert.base64-encode ya que el contenido de un archivo PHP es interpretado por el servidor web y su resultado HTML es mostrado en navegador web mas no su código fuente. Por lo tanto, al utilizar este filtro PHP vamos a codificar el contenido del archivo PHP y luego mostrarlo en el navegador como una cadena de texto codificado en base64.

![](/assets/images/Archangel/image029.png)

![](/assets/images/Archangel/image031.png)

Podemos observar que llegamos a obtener una cadena de caracteres codificado en base64, que viene a ser el contenido del archivotest.php

Ahora decodificaremos la cadena de caracteres para observar el contenido del archivo test.php.

![](/assets/images/Archangel/image033.png)

Podemos observar en el contenido del archivo, que el desarrollado del sitio web ha programado un filtro para evitar que digitemos puntos dobles(o ../) como entrada en el valor del parámetro GET view. Además, ha configurado otro filtro para que la función include solo pueda mostrar el contenido de los archivos ubicados en el directorio /var/www/html/development_testing.

## FASE EXPLOTACION O GANAR ACCESO O HACKING 
Ahora teniendo en cuenta estos filtros podemos utilizar ..// en ves de los puntos dobles para movernos al directorio raíz / del sistema objetivo. Luego,digitaremos las rutas de archivos importantes del sistema objetivo como /etc/passwd, /etc/shadow, /proc/version con el fin de observar su contenido a través del navegador web.

![](/assets/images/Archangel/image035.png)

![](/assets/images/Archangel/image037.png)

Podemos observar el contenido del archivo /etc/passwd del sistema objetivo mediante la vulnerabilidad Path Transversal.

Ahora utilizaremos el servidor Proxy de Burp Suite para interceptar la solicitud GET realizada hacia al servidor web para obtener el contenido del archivo /etc/passwd. Además, enviaremos nuestra solicitud hacia el módulo Intruder de Burp Suite con el fin de realizar un ataque Sniper sobre la ruta /etc/passwd con el fin de probar con diferentes rutas de archivos importantes que puede haber en un sistema.

![](/assets/images/Archangel/image039.png)

![](/assets/images/Archangel/image041.png)

![](/assets/images/Archangel/image043.png)

![](/assets/images/Archangel/image045.png)

![](/assets/images/Archangel/image047.png)

Observamos que a través del ataque Sniper de Burp Suite sabemos que podemos acceder al contenido del archivo /etc/crontab, /var/log/apache2/error.log,/var/log/apache2/access.log del sistema objetivo a través de la vulnerabilidad Path Transversal. Además, en el contenido del registro de acceso se muestran el valor del encabezado User-Agent de las diferentes solicitudes GET que realizamos hacia el sitio web a través del parámetro GET view.

Ahora teniendo en cuenta que podemos acceder a los registros de acceso hacia el servidor web que se esta ejecutando en el sistema destino, podemos modificar el valor del encabezado User-Agent a un código PHP, que me permita ejecutar comando en el sistema objetivo a través del parámetro GET cmd.Este código PHP se ejecutará cuando a través del parámetro GET view realizamos una solicitud GET, con el encabezado User-Agent modificado, le pidamos a la función include, del archivo test.php, que nos ejecute el contenido del registro de acceso y nos muestre su salida HTML a través del navegador web.

![](/assets/images/Archangel/image049.png)

![](/assets/images/Archangel/image051.png)

![](/assets/images/Archangel/image053.png)

![](/assets/images/Archangel/image055.png)

Podemos observar que hemos modificado el contenido del encabezado User-Agent de nuestras solicitudes HTTP utilizando la extensión User. Luego, actualice la URL que contiene la ruta del archivo de registros de acceso para que se realiza una solicitud GET HTTP y quede registrada en el archivo de registros de acceso. Además, la función include de test.php ejecutara el contenido del código PHP, que insertamos a User-Agent. Por lo tanto, ya podremos ejecutar comandos remotos en el sistema objetivo utilizando el parámetro GET cmd.

Ahora levantaremos un servidor web local en mi sistema con el fin de descargar un código PHP, que me genere una conexión reverse Shell desde el sistema objetivo hacia mi sistema, a través del parámetro GET cmd y el comando get.

![](/assets/images/Archangel/image057.png)

![](/assets/images/Archangel/image059.png)

![](/assets/images/Archangel/image061.png)

Podemos observar que se descarga el archivo reverse.php, que nos generara la conexión reverse Shell, en el directorio development_testing.

Ahora ejecutaremos el archivo reverse.php utilizando la función Include del archivo test.php, pero antes debemos habilitar el puerto 1500 en el sistema para que este escuchando las conexiones entrantes.

![](/assets/images/Archangel/image065.png)

![](/assets/images/Archangel/image067.png)

![](/assets/images/Archangel/image069.png)

De esta manera llegamos obtener acceso al sistema objetivo siendo el usuario www-data.

## FASE ESCALADA DE PRIVILEGIOS 

Ahora debemos buscar un vector de escalada de privilegios con el fin de ser un usuario con privilegios más elevados. Para ello observamos detalladamente la tarea cron, que observamos en el contenido del archivo crontab del sistema anteriormente.

![](/assets/images/Archangel/image071.png)

Podemos observar que el usuario archangel ha configurado una tarea cron, que viene a ser un script sh que se ejecutara cada minuto de cada día de cada mes. Además, podemos observar que tenemos el permiso para modificar el contenido del script sh ya que este habilitado el permiso de escritura para usuarios que pertenecen a otro grupo diferente al grupo primario del usuario archangel.

Ahora modificaremos el contenido del script sh para que nos genere una conexión reverse Shell hacia nuestro sistema con el fin de ser el usuario archangel.

![](/assets/images/Archangel/image073.png)

Ahora habilitamos el puerto 1900 en el sistema para que este escuchando las conexiones entrantes, y esperamos que el sistema destino ejecute la tarea cron siendo el usuario archangel.

![](/assets/images/Archangel/image075.png)

De esta manera realizamos la escalada horizontal llegando a ser el usuario archangel.

Ahora debemos buscar un vector de escalada de privilegios con el fin de ser un usuario con privilegios más elevados. Para ello utilizaremos el comando find para buscar los archivos binarios que tienen asignado el permiso SUID.

![](/assets/images/Archangel/image077.png)

Podemos observar que el archivo backup ubicado en el directorio home del usuario archangel tiene habilitado el permiso SUID. Además, tenemos habilitado el permiso de ejecución en el archivo backup.

Ahora ejecutaremos el archivo SUID backup para poder deducir que comandos se ejecutan en segundo plano.

![](/assets/images/Archangel/image079.png)

Podemos observar que se ejecuta el comando cp para copiar todos los archivos del directorio /home/user/archangel/myfiles hacia una ruta, pero este directorio no existe en el sistema destino es por ello que sale un mensaje de error.

Ahora observaremos el contenido del archivo SUID, pero con el comando strings para que me extraigan solo las cadenas de caracteres legibles.

![](/assets/images/Archangel/image081.png)

Podemos observar que no se ha configurado la ruta completa del archivo binario cp en el comando que se ejecuta en segundo plano al momento de ejecutar el archivo SUID. Aprovechando esto podemos manipular la variable de entorno PATH del sistema objetivo para que ejecute un archivo binario malicioso de nuestra propiedad en ves del archivo binario cp original. Para ello crearemos el archivo malicioso, que nos genera una conexión reverse Shell hacia mi sistema siendo el usuario root, y estará ubicado en el directorio tmp. Además, este archivo tendrá el mismo nombre que el archivo binario cp original.

Ahora modificaremos el valor de la variable PATH con el fin de que el sistema busque primero en el directorio tmp y pueda ejecutar nuestro archivo malicioso cp.

![](/assets/images/Archangel/image083.png)

Ahora habilitamos el puerto 1800 en el sistema para que este escuchando las conexiones entrantes, y ejecutamos el archivo binario SUID.

![](/assets/images/Archangel/image085.png)

![](/assets/images/Archangel/image087.png)

De esta manera llegamos a ser el usuario root.

Ahora buscaremos las dos banderas contenidas en los archivos user.txt y root.txt. Para localizar los archivos utilizaremos el comando find para buscar desde el directorio /, archivos con sus nombres.

![](/assets/images/Archangel/image089.png)
 
![](/assets/images/Archangel/image091.png) 
 
 
 
 
 
 
 
 
 
 

 
 

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































