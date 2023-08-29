---
layout: single
title: UltraTech - TryHackMe
excerpt: "Debemos encontrar 1 bandera en la maquina Ultratech-latest. Donde a partir del escaneo de puertos de Nmap pude saber que hay un servicio HTTP, SSH, FTP, y una API. Además, con un script de Nmap supe que había una página web, que pertenecía a una aplicación web. Luego, analizando el código fuente de la aplicación web me encontré con la ruta de un directorio, que contenía 3 archivos JavaScript. Donde uno de ellos era utilizado para la creación de una API. Además, la API estaba levantado en uno de los puertos, y ofrecía la funcionalidad de ejecutar el comando ping en el sistema operativo del sistema destino a través de un parámetro GET que era vulnerable a Command Injection.Luego, en la escalada privilegio horizontal me encontré con un database SQLite. Donde encontré el hash de r00t, y logré crackearlo con John the Ripper. Luego, para la escalada vertical, la primera manera es aprovechando que el usuario r00t pertenece al grupo Docker, y la segunda manera es explotando la vulnerabilidad Sudo Baron Samedit."
date: 2023-08-29	
classes: wide
header:
  teaser: /assets/images/Ultratech/image002.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - Command Injection
  - HTTP
  - JavaScript
  - Node.js
  - API
  - Docker
  - SSH
  - FTP
  - SQLite
  - Baron Samedit
  - John The Ripper
  - PHP
---

![](/assets/images/Ultratech/image001.png)

## SUMMARY

Debemos encontrar 1 bandera en la maquina Ultratech-latest. Donde a partir del escaneo de puertos de Nmap pude saber que hay un servicio HTTP, SSH, FTP, y una API. Además, con un script de Nmap supe que había una página web, que pertenecía a una aplicación web. Luego, analizando el código fuente de la aplicación web me encontré con la ruta de un directorio, que contenía 3 archivos JavaScript. Donde uno de ellos era utilizado para la creación de una API. Además, la API estaba levantado en uno de los puertos, y ofrecía la funcionalidad de ejecutar el comando ping en el sistema operativo del sistema destino a través de un parámetro GET que era vulnerable a Command Injection.Luego, en la escalada privilegio horizontal me encontré con un database SQLite. Donde encontré el hash de r00t, y logré crackearlo con John the Ripper. Luego, para la escalada vertical, la primera manera es aprovechando que el usuario r00t pertenece al grupo Docker, por lo tanto, puede ejecutar y administrar contenedores Docker, y la segunda manera es explotando la vulnerabilidad Sudo Baron Samedit presente.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener las banderas. Para ello empezaremos utilizando el comando openvpn con el fin de establecer una conexión VPN con la red virtual dónde está la máquina Ultratech-latest. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/Ultratech/image003.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/Ultratech/image004.png)

Además, la plataforma de Tryhackme nos muestra la dirección ip de la máquina Ultratech-latest.

![](/assets/images/Ultratech/image005.png)

## FASE ESCANEO Y ENUMERACION
Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina Ultratech-latest. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:

- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El parámetros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/Ultratech/image006.png)

Los resultados que obtuvimos del escaneo vienen a ser:

- Hay un servicio HTTP levantado en el puerto 31331, y el programa servidor HTTP que se está corriendo. Además, la versión del programa servidor.
- Además, a partir del script http-title.nse de Nmap se pudo saber que hay una página web almacenado en el servidor web, cuya etiqueta tittle es UltraTech - The best of technology (AI, FinTech, Big Data).
- Hay un servicio SSH que se está levantado en el puerto 22, y el programa servidor SSH que se está corriendo. Además, la versión del programa servidor SSH.
- Hay un servicio FTP levantado en el puerto 21, y el programa servidor FTP que se está corriendo. Además, la versión del programa servidor.
- Hay un posible servidor web o API levantado en el puerto 8081. Además, el posible servidor web o API ha sido creado a partir de código Javascript utilizando el entorno de tiempo de ejecución Node.js.

Ahora observaremos la página web almacenada en el servidor web levantada en el puerto 31331 desde nuestro navegador web.

![](/assets/images/Ultratech/image007.png)

![](/assets/images/Ultratech/image008.png)

Podemos observar que la página web viene a ser la página de inicio de una aplicación web almacenada en el servidor web. Además, cuando analizamos el código fuente de la aplicación, encontrar la ruta de un archivo JavaScript, que al parecer es utilizado para crear la aplicación web. Además, nos percatamos que el archivo JavaScript esta almacenada en el directorio js.

Ahora accederemos al directorio js con el fin de observar que otros archivos JavaScript han sido creados para posibles aplicaciones web o APIs o servidores escondidos.

![](/assets/images/Ultratech/image009.png)

![](/assets/images/Ultratech/image010.png)

![](/assets/images/Ultratech/image011.png)

Podemos observar que en el directorio js nos encontramos con un archivo JavaScript cuyo código hace referencia a un API levantada en el puerto 8081. Por lo tanto, con esto podemos afirmar que en el puerto 8081 se esta ejecutando una API que permite la comunicación entre el sistema destino, siendo mas especifico, permite que se pueda ejecutar al parecer un comando ping desde el sistema operativo del sistema destino.

Ahora accederemos al API desde mi navegador web con el fin de probar la funcionalidad que se ha programado en su código.

![](/assets/images/Ultratech/image012.png)

![](/assets/images/Ultratech/image013.png)

Podemos observar que pudimos probar la funcionalidad de la API, que viene a ser ejecutar el comando ping desde el sistema operativo del sistema destino hacia nuestro sistema con un solo paquete ICMP Request. Además, los resultados de la ejecución del comando ping se pueden observar debido a la solicitud GET HTTP que se realiza.

Ahora probaremos si el parámetro GET ip es vulnerable a Command Injection, para ello utilizaremos; al lado del valor de la dirección ip con el fin de que se ejecutar otro comando utilizando la misma línea de comandos del comando ping.

![](/assets/images/Ultratech/image014.png)

Podemos observar solo que le comando ping se ejecuto y nuestro comando ls se imprime en pantalla, pero no se ejecuta. Para solucionar esto, usaremos la sustitución de comandos, que nos permite ejecutar comandos dentro de la línea de otro comando, utilizando las comillas invertidas o $().

![](/assets/images/Ultratech/image015.png)

![](/assets/images/Ultratech/image016.png)

Podemos observar que llegamos a realiza enumerar recursos del sistema destino a partir de la ejecución de un comando en el sistema operativo del sistema destino. Por lo tanto, el parámetro ip es vulnerable a Command Injection.

## FASE HACKING O GANAR ACCESO O EXPLOTACION

Ahora aprovechando esta vulnerabilidad cargaremos un archivo PHP en el sistema destino a través del comando wget y levantando un servidor web local en mi sistema utilizando el módulo http.server de Python.

![](/assets/images/Ultratech/image017.png)

![](/assets/images/Ultratech/image018.png)

Ahora ejecutaremos nuestro archivo PHP, que nos generara una reverse Shell, utilizando el archivo binario ejecutable PHP instalado en el sistema destino.

![](/assets/images/Ultratech/image019.png)

![](/assets/images/Ultratech/image020.png)

Podemos observar que llegamos acceder al sistema destino siendo el usuario www explotando la vulnerabilidad Command Injection.

Ahora realizaremos un tratamiento de la tty para tener una Shell más interactiva.

![](/assets/images/Ultratech/image021.png)

## FASE ESCALADA PRIVILEGIOS

Ahora debemos buscar un vector de escalada de privilegios con el fin de ser un usuario con privilegios más elevados. Para ello revisaremos la versión del archivo binario ejecutable sudo con el fin de saber si su versión cae dentro del rango de versiones vulnerables a ña vulnerabilidad CVE-2021-3156 o Sudo Baron Samedit.

![](/assets/images/Ultratech/image022.png)

![](/assets/images/Ultratech/image023.png)

Podemos observar que la versión de sudo en el sistema destino si caen dentro de las versiones vulnerables, pero para estar seguro utilizaremos los siguientes comandos obtenidas de una PoC(o Prueba de Concepto): https://github.com/lockedbyte/CVE-Exploits/tree/master/CVE-2021-3156

![](/assets/images/Ultratech/image024.png)

![](/assets/images/Ultratech/image025.png)

Podemos observar que la versión del sudo si es vulnerable.

Ahora utilizaremos el siguiente exploit https://github.com/blasty/CVE-2021-3156 para explotar la vulnerabilidad y llegar a obtener a una Shell con los privilegios del usuario root.

![](/assets/images/Ultratech/image026.png)

![](/assets/images/Ultratech/image027.png)

De esta manera podemos realizar una escalada de privilegios vertical, explotando la vulnerabilidad Sudo Baron Samedit.

Otra manera que pudimos haber optado para realizar una escalada privilegios, seria realizar una enumeración manual sobre los archivos del usuario local www.

![](/assets/images/Ultratech/image028.png)

Podemos observar que se llegó a encontrar un archivo con extensión sqlite. Este tipo de archivos vienen a ser base de datos SQLite creadas por aplicaciones web cuando se le integran el DBMS SQLite para que puedan crear, modificar, y consultar sus bases de datos dentro del contexto de la propia aplicación.

Ahora utilizaremos sqlite3 para poder interactuar con esta base de datos SQLite, pero antes utilizaremos netcat para transferir el archivo a mi sistema. Donde tengo instalado el cliente de línea de comando sqlite3.

![](/assets/images/Ultratech/image029.png)

![](/assets/images/Ultratech/image030.png)

![](/assets/images/Ultratech/image031.png)

![](/assets/images/Ultratech/image032.png)

![](/assets/images/Ultratech/image033.png)

Podemos observar que la base de datos SQLite consta de una tabla llamada users. Además, esta tabla consta de tres columnas y pudimos enumerar los tipos de datos que almacenan cada una. Además, pudimos volcar los registros de la tabla users. Donde nos encontramos con el hash del usuario local r00t del sistema destino.

Ahora utilizare la herramienta hash-identifier para determinar la función o algoritmo hash que se ha aplicado al hash.

![](/assets/images/Ultratech/image034.png)

Podemos observar que la función hash más probable es MD5.

Ahora utilizare John the Ripper para crackear el hash.

![](/assets/images/Ultratech/image035.png)

Podemos observar que la herramienta llego a crackear el hash fácilmente, obteniendo el password del usuario r00t.

Ahora supondremos que el usuario local r00t ha reutilizado estas credenciales en su inicio de sesión hacia el sistema destino. Por lo tanto, intentaremos autenticarnos con estas credenciales.

![](/assets/images/Ultratech/image036.png)

Nuestra suposición fue correcta. De esta manera realizamos una escalada privilegio horizontal.

Ahora debemos buscar otro vector de escalada de privilegios para ser un usuario con privilegios más elevados. Para ello utilizaremos el siguiente comando con el fin de obtener información sobre la identidad del usuario r00t.

![](/assets/images/Ultratech/image037.png)

Podemos observar que el usuario pertenece al grupo secundario Docker. Por lo tanto, el usuario puede ejecutar y administrar contenedores Dockers en el sistema sin la necesidad de tener los privilegios del usuario root. Por ejemplo, enumeraremos todas las imágenes descargadas en el sistema destino.

![](/assets/images/Ultratech/image038.png)

Podemos observar que se ha descargado una sola imagen llamada bash.

Ahora aprovechando que podemos ejecutar contenedores Docker, utilizaremos el siguiente comando con el fin de iniciar un contenedor utilizando la imagen bash, y montando el sistema de archivos del sistema destino en el directorio /mnt del contenedor. Luego, utilizaremos el parámetro chroot con el fin de convertir el directorio raíz del contenedor al directorio mnt y ejecutaremos una shell interactiva dentro del contenedor con el fin de poder acceder a todos los archivos y directorios del sistema destino con los privilegios del usuario root.

![](/assets/images/Ultratech/image039.png)

De esta manera encontramos otra manera de realizar una escalada privilegio vertical para ser el usuario root.

Ahora encontraremos la bandera que viene a ser la clave privada SSH del usuario root.

![](/assets/images/Ultratech/image040.png) 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































