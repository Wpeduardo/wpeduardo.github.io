---
layout: single
title: Minotaur's Labyrinth - TryHackMe
excerpt: "Debemos encontrar tres banderas en la maquina Labyrinth Fast. Donde a partir del escaneo de puertos de Nmap pude saber que hay un servicio HTTP, SSH. FTP, Además, con scripts de Nmap supe que había una página web, que contenía un login, y el servidor FTP admitía conexiones anónimas. Luego, pude acceder a los recursos compartido del servidor FTP. Donde encontré unas pistas para realizar fuerza bruta de directorios con Wfuzz, llegando a encontrar un registro que contenía unas credenciales, que me permitió autenticarme en el login accediendo al portal de administración de una aplicación web vulnerable a SQLi. Luego, obtuve los registros de su base de datos, y obtuve las credenciales del admin de la aplicación web, y logré autenticarme en la aplicación web, encontrando una sección nueva de la aplicación web que era vulnerable a Command Injection, logrando acceder al sistema. Luego, la escala privilegios vertical lo realiza a partir de un script, ejecutada por el usuario root."
date: 2023-08-31	
classes: wide
header:
  teaser: /assets/images/Centauro/image002.png
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
  - FTP
  - Linpeas
  - SQLi
  - Wfuzz
  - PHP
  - John The Ripper
  - Hash-Identifier
---

![](/assets/images/Centauro/image001.png)

## SUMMARY

Debemos encontrar tres banderas en la maquina Labyrinth Fast. Donde a partir del escaneo de puertos de Nmap pude saber que hay un servicio HTTP, SSH. FTP, Además, con scripts de Nmap supe que había una página web, que contenía un login, y el servidor FTP admitía conexiones anónimas. Luego, pude acceder a los recursos compartido del servidor FTP. Donde encontré unas pistas para realizar fuerza bruta de directorios con Wfuzz, llegando a encontrar un registro que contenía unas credenciales, que me permitió autenticarme en el login accediendo al portal de administración de una aplicación web vulnerable a SQLi. Luego, a partir de volcar los registros de la base de datos de la aplicación web, obtuve las credenciales del admin de la aplicación web, y logré autenticarme en la aplicación web, encontrando una sección nueva de la aplicación web que era vulnerable a Command Injection, logrando acceder al sistema. Luego, la escala privilegios vertical lo realiza a partir de un script, que era una tarea que se ejecutaba de manera periódica y automatizada por el usuario root.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener las banderas. Para ello empezaremos utilizando el comando openvpn con el fin de establecer una conexión VPN con la red virtual dónde está la máquina Labyrinth Fast. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/Centauro/image003.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/Centauro/image004.png)

Además, la plataforma de Tryhackme nos muestra la dirección ip de la máquina Labyrinth Fast.

![](/assets/images/Centauro/image005.png)

## FASE ESCANEO Y ENUMERACION

Ahora pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina Labyrinth Fast. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:

- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El parámetros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/Centauro/image006.png)

Los resultados que obtuvimos del escaneo vienen a ser:

- Hay un servicio HTTP levantado en el puerto 80, y el programa servidor HTTP que se está corriendo, y la versión del programa servidor. Además, a partir del script http-title.nse de Nmap se pudo saber que hay una página web almacenado en el servidor web, cuya etiqueta tittle es Login.
- Además, supuestamente el servidor HTTP o web esta utilizando el protocolo SSL, ya que esta levantado en el puerto 443 también, aunque parece un falso positivo por parte de Nmap.
- Hay un servicio FTP levantado en el puerto 21, y el programa servidor FTP que se está corriendo. Además, la versión del programa servidor. Además, a partir del script ftp-anom.nse de Nmap, se pudo saber que el servidor FTP admite conexiones anónimas, es decir, que nos podemos conectar al servidor FTP sin proveer unas credenciales de inicio de sesión.
- El puerto 3306 está abierto, aunque Nmap no logra determinar con exactitud el servicio que esta levantando en el puerto.

Ahora nos conectaremos al servidor FTP a través de una conexión anónimos con el fin de observar el recurso publico pub, que viene a ser un directorio.

![](/assets/images/Centauro/image007.png)

![](/assets/images/Centauro/image008.png)

Podemos observar que el directorio compartido pub:

1. Contiene un archivo, que viene a ser un mensaje redactado por el usuario Minotaur, y que hace referencia hacia otro usuario llamado Daedalus. Por  lo tanto, ya conocemos dos posibles usuarios locales del sistema destino.
2. Contiene un subdirectorio, que contiene la primera bandera. Además, contiene un archivo, que viene a ser un mensaje redactado por el usuario Minotaur, y que hace referencia a una posible tarea cron o una tarea automatizada que se ejecuta de manera periódica debido a que el usuario tiende a olvidar mucho las cosas.

Ahora observaremos el contenido de la página web, almacenada en el servidor, desde mi navegador web.

![](/assets/images/Centauro/image009.png)

![](/assets/images/Centauro/image010.png)

![](/assets/images/Centauro/image011.png)

Podemos observar que la página web consta de un formulario, pero ninguno de los parámetros o campos del formulario es vulnerable a SQLi.

Ahora teniendo en cuenta lo dicho en el mensaje compartido en el servidor FTP, realizaremos fuerza bruta de directorios con la herramienta Wfuzz sobre el directorio raíz del servidor web con el fin de encontrar recursos escondidos.

![](/assets/images/Centauro/image012.png)

![](/assets/images/Centauro/image013.png)

Podemos observar que nos encontramos con muchos recursos con código HTTP 302, pero si intentamos acceder a algunos de ellos a través de mi navegador web, nos topamos con la sorpresa de que son falsos positivos, pero nos damos cuenta de que estos falsos positivos tienen en común el valor 3562 en la columna chars, que viene a ser la cantidad de caracteres contenida en la respuesta HTTP emitida por el servidor web en cada solicitud HTTP.

Ahora filtraremos los falsos positivos utilizando el valor 3652 de la columna chars.

![](/assets/images/Centauro/image014.png)

Podemos observar que llegamos a encontrar 5 recursos escondidos en el directorio raíz del servidor web.

Ahora observaremos el recurso logs, que viene a ser el mas interesante ya que puede contener información sensible, desde mi navegador web.

![](/assets/images/Centauro/image015.png)

![](/assets/images/Centauro/image016.png)

![](/assets/images/Centauro/image017.png)

Podemos observar que el recurso logs, viene a ser un directorio, que contiene un archivo registro, que viene a ser una solicitud POST HTTP. Donde podemos observar unas posibles credenciales que le pueden corresponder al usuario Daedalus.

Ahora probaremos estas credenciales en el formulario HTML, que encontramos anteriormente.

![](/assets/images/Centauro/image018.png)

![](/assets/images/Centauro/image019.png)

![](/assets/images/Centauro/image020.png)

Podemos observar que llegamos acceder al portal de administración de una aplicación web, que consta de una funcionalidad que nos permite hacer consultas sobre otros usuarios que se han autenticado en la aplicación web, y obtener el hash de su contraseña. Además, el formato de como se presentar los resultados de las consultas, nos da a intuir que el servidor web utiliza el valor, que ingresamos como nombre de la persona o criatura, para realizar sus consultas SQL hacia su base de datos, y poder mostrar la información que observamos.

## FASE GANAR ACCESO O EXPLOTACION O HACKING

Ahora probaremos si la aplicación web desinfecta o sanatiza adecuadamente los valores que ingresamos en ese campo.

![](/assets/images/Centauro/image021.png)

![](/assets/images/Centauro/image022.png)

Podemos observar que la aplicación no sanatiza adecuadamente las entradas del usuario en ese campo. Por lo tanto, es vulnerable aSQLi. Además, llegamos a enumerar todas las bases de datos gestionadas por el DBMS en el sistema destino.

Ahora enumeraremos las tablas de la base de datos labyrinth ya que tiene el mismo nombre que la aplicación, por lo tanto, es la que debe almacenar la información de los usuarios autenticados en la aplicación web.

![](/assets/images/Centauro/image023.png)

Podemos observar que la base de datos consta de dos tablas.

Ahora enumeraremos las columnas de la tabla people, y volcaremos sus registros.

![](/assets/images/Centauro/image024.png)

Registros de la columna namePeople:

![](/assets/images/Centauro/image025.png)

Registros de la columna passwordPeople:

![](/assets/images/Centauro/image026.png)

Registros de la columna permissionPeople:

![](/assets/images/Centauro/image027.png)

A partir de los resultados, podemos concluir que el usuario M!n0taur viene a ser el administrador de la aplicación web ya que tiene el permiso admin asignado.

Ahora crackearemos el hash de la contraseña del usuario M!n0taur, pero antes debemos saber que función o algoritmo hash utiliza.

![](/assets/images/Centauro/image028.png)

![](/assets/images/Centauro/image029.png)

Podemos observar que con la herramienta hash-identifier pudimos saber que función hash posiblemente utiliza, y luego utilizando John The Ripper, pudimos crackearla.

Ahora nos autenticaremos en la aplicación web con estas credenciales.

![](/assets/images/Centauro/image030.png)

![](/assets/images/Centauro/image031.png)

Podemos observar que al ingresar al portal de administración de la aplicación siendo el administrador, nos aparece la segunda bandera y una nueva sección llamada Secret_Stuff, y al momento de hacer clic sobre él, nos redirige hacia una página web, que presenta la funcionalidad de ejecutar el comando echo en el sistema operativo del sistema destino, y mostrar su resultado en la página web.

Ahora probaremos si la aplicación web sanatiza o desinfecta adecuadamente la entrada que ingresaremos, y que será imprimida por el comando echo.

![](/assets/images/Centauro/image032.png)

![](/assets/images/Centauro/image033.png)

![](/assets/images/Centauro/image034.png)

![](/assets/images/Centauro/image035.png)

![](/assets/images/Centauro/image036.png)

![](/assets/images/Centauro/image037.png)

Podemos observar que hay un filtro configurado por el administrador de la aplicación web, que limita el valor de nuestras entradas, pero luego de varios intentos, llegamos a dar con el payload indicado. Donde pudimos enumerar los archivos y subdirectorios del directorio raíz del servidor web en el sistema destino. Por lo tanto, la aplicación web es vulnerable a Command Injection.

Ahora observaremos el contenido del archivo echo.php con el fin de saber que caracteres no podemos ingresar en el campo, pero vamos a tener que codificar su contenido antes de poder ver su contenido sino el servidor web lo va a ejecutar.

![](/assets/images/Centauro/image038.png)

![](/assets/images/Centauro/image039.png)

![](/assets/images/Centauro/image040.png)

Podemos observar que llegamos a conseguir el contenido del archivo echo.php, y podemos observar los caracteres que no podemos observar, que son: `#!@%^&*()$_=\[\]\';,{}:>?~\\\\.`, y el comando que está ejecutando en segundo plano la aplicación web con nuestra entrada.

Ahora teniendo en cuenta los caracteres que no podemos ingresar, utilizaremos el siguiente comando para que nos generara una reverse Shell cuando se ejecute, pero lo utilizaremos en su forma codificada para evitar ser detectado por el filtro. Además, utilizaremos el símbolo de la tubería | con el fin de redirigir la salida de los comandos hacia la entrada de otros comandos.

![](/assets/images/Centauro/image041.png)

![](/assets/images/Centauro/image042.png)

![](/assets/images/Centauro/image043.png)

Podemos observar que llegamos obtener acceso al sistema destino siendo el usuario daemon y explotando la vulnerabilidad Command Injection.

![](/assets/images/Centauro/image044.png)

Ademas, Podemos observar que al usuario Daemon se le ha configurado para que utilice la Shell nologin cuando inicie sesión en el sistema destino. Esta Shell viene a ser no interactiva e inestable. Por lo tanto, realizaremos un tratamiento de la tty para tener una Shell estable e interactiva.

![](/assets/images/Centauro/image045.png)

## FASE ESCALADA PRIVILEGIOS

Ahora debemos buscar un vector de escalada de privilegios con el fin de ser un usuario con privilegios más elevados. Para ello utilizaremos la herramienta linpeas, que nos automatizara el proceso de buscar vectores de escalada privilegios. Esta herramienta lo puedes descargar en el siguiente enlace: https://github.com/carlospolop/PEASS-ng/releases/tag/20230413-7f846812.

![](/assets/images/Centauro/image046.png)

![](/assets/images/Centauro/image047.png)

Podemos observar que la herramienta llego a encontrar unos directorios inusuales en el sistema de archivo del sistema destino o en el directorio raíz de este.

Ahora observaremos el contenido del directorio timers.

![](/assets/images/Centauro/image048.png)

![](/assets/images/Centauro/image049.png)

Podemos observar que hay un script llamador timer.sh, que va digitar el mensaje dont fo…..forge….ttt en el archivo dontforget sin sobrescribir su contenido. Además, este archivo está dentro de uno de los directorios inusuales.

Ahora observaremos el contenido del directorio reminders.

![](/assets/images/Centauro/image050.png)

Podemos observar que el directorio reminders contiene solo el archivo dontforget. Además podemos notar en el contenido del archivo varias líneas con el mensaje dont fo…..forge….ttt. Esto nos da a deducir que el script timer.sh viene a ser como una tarea cron, que se ejecuta de manera automatizada y periódica, ejecutada con los privilegios de algún usuario del sistema local. Además, debido a que el archivo dontforget y el script timer son de propiedad del usuario root, lo mas probable es que el script timer se ejecute con sus privilegios.

Ahora debido a que tenemos el permiso de escritura sobre el scrip timer.sh, es que podemos modificar su contenido. Para ello ingresaremos un comando que nos generara una reverse Shell. Además, habilitaremos el puerto 1900 en modo listening para que este esperando la conexión entrante de la reverse Shell.

![](/assets/images/Centauro/image051.png)

![](/assets/images/Centauro/image052.png)

Podemos observar, luego de esperar unos minutos para que se ejecute el script timer según lo supuesto anteriormente, obtuvimos acceso al sistema siendo el usuario root. Además, logramos obtener la última bandera.

 
 

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































