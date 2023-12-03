---
layout: single
title: Different CTF - TryHackMe
excerpt: "Debemos encontrar dos banderas. Primero, a partir de un escaneo de puertos pude saber que un servidor web alojado un sitio web creado WordPress. Luego, realicé fuerza bruta de directorios con el fin de encontrar recursos escondidos en el directorio raíz del sitio web, encontrando un wordlist y una imagen, luego extraje un archivo de texto oculto en la imagen, pero antes utilicé stegcracker para encontrar la palabra clave requerida. Luego, encontré unas credenciales FTP de un usuario en el archivo de texto, logrando acceder a los recursos compartidos, que consista en el directorio raíz de otro sitio web creado con el mismo CMS, logrando cargar un archivo PHP en los recursos compartidos y que nos genere una reverse Shell cuando lo observemos a traves del navegador web. Luego, en la escalada privilegio vertical, utilice sucrack para crackear la contraseña de un usuario local, luego analice el código fuente de un archivo binario con diversas tools de ingeniería inversa, logrando obtener el password de root."
date: 2023-12-03	
classes: wide
header:
  teaser: /assets/images/Different/image002.jpg
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
  - Base85
  - Base64
  - Python
  - Gobuster
  - Ghidra
  - hexeditor	
  - ltrace
  - WpScan
  - Steghide
  - Stegcracker
  - CMS Wordpress
  - phpMyAdmin
  - John THe Ripper
  - sucrack
  - su
---

![](/assets/images/Different/image001.png)

## SUMMARY

Debemos encontrar dos banderas. Primero, a partir de un escaneo de puertos pude saber que un servidor web alojado un sitio web creado WordPress. Luego, realicé fuerza bruta de directorios con el fin de encontrar recursos escondidos en el directorio raíz del sitio web, encontrando un wordlist y una imagen, luego extraje un archivo de texto oculto en la imagen, pero antes utilicé stegcracker para encontrar la palabra clave requerida. Luego, encontré unas credenciales FTP de un usuario en el archivo de texto, logrando acceder a los recursos compartidos, que consista en el directorio raíz de otro sitio web creado con el mismo CMS, logrando cargar un archivo PHP en los recursos compartidos y que nos genere una reverse Shell cuando lo observemos a traves del navegador web. Luego, en la escalada privilegio vertical, utilice sucrack para crackear la contraseña de un usuario local, luego analice el código fuente de un archivo binario con diversas tools de ingeniería inversa, logrando obtener el password de root.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener dos banderas. Primero, empezaremos utilizando el comando openvpn y el archivo de configuración OpenVPN con el fin de establecer una conexión VPN y poder acceder a los laboratorios donde están las máquinas virtuales y vulnerables.

![](/assets/images/Different/image003.png)

Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina Different CTF.

![](/assets/images/Different/image004.png)

## FASE ESCANEO Y ENUMERACION

Ahora pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del sistema destino. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo del sistema destino.

![](/assets/images/Different/image005.png)

Los resultados que obtenemos son:
- Un programa servidor FTP que se ejecuta en el puerto 21.
- Un programa servidor HTTP que se ejecuta en el puerto 80. Ademas, podemos observar que hay un sitio web alojado en el servidor, que ha sido creado a partir del CMS Wordpress.

Ahora accederemos al sitio web a traves de mi navegador web.

![](/assets/images/Different/image006.png)

![](/assets/images/Different/image007.png)

Podemos observar en el codigo fuente de la pagina inicio del sitio web, unos enlaces que hacen refencia a los diferentes recursos de la estructura de directorios de un sitio web creado con el CMS WordPress. Ademas, en los enlaces observamos un nombre de dominio donde lo mas probable es que le corresponda al sitio web.

Ahora agregaremos este nombre de dominio asociado con la dirrecion IP del servidor web en nuestro archivo /etc/hosts.

![](/assets/images/Different/image008.png)

Ahora utilizaremos WPScan para enumerar los usernames que pueda encontrar en las paginas web del sitio web.Luego, utilizaremos WPScan para realizar ataque de fuerza bruta de contraseñas contra el login del CMS WordPress con los usernames encontrados y con el wordlist rockyou.txt.

![](/assets/images/Different/image009.png)

![](/assets/images/Different/image010.png)

Podemos observar que la herramienta logro enecontrar un username valido, pero luego de 1 hora de espera ya me di cuenta que la herramienta no lograra encontrar ningun password valido en el wordlist rockyou.txt.

Ahora realizaremos fuerza bruta de directorios con el fin de encontrar recursos escondidos en el directorio raiz del sitio web.

![](/assets/images/Different/image011.png)

Podemos observar que algunos de los recursos encontrado son los tipicos directorios que pertenecen a la estructura de directorios de un sitio web creado con el CMS WordPress. Pero, el recurso announcements es atipico. Por lo que observaremos su contenido a traves del navegador web.

![](/assets/images/Different/image012.png)

Podemos observar que el recurso viene a ser un directory listing que contiene dos recursos: un wordlist, y una imagen jpg.

Ahora intentare realizar fuerza bruta de contraseñas utilizando WPScan contra el login del CMS WordPress pero utilizando este nuevo wordlist.

![](/assets/images/Different/image013.png)

Luego de esperar mas de media hora podemos deducir que no encontraremos un password valido para el username hakanbey01.

Ahora utilizaremos steghide para extraer posibles datos ocultos en el imagen jpg que enncontramos en el directory listing.

![](/assets/images/Different/image014.png)

![](/assets/images/Different/image015.png)

![](/assets/images/Different/image016.png)

Podemos observar que nos solicitan una palabra clave para extraer datos ocultos en la imagen JPG. Por tal razon utilice stegcracker con el fin de realizar fuerza bruta para encontrar esa palabra clave utilizando el wordlist encontrado anteriormente. Luego, logramos extraer un archivo de texto que contenia un texto que estaba codificado en base64, logrando decodificar el texto, nos encontramos unas credenciales FTP.

Ahora accederemos al servidor FTP con estas credenciales para enumerar los recursos compartidos por este usuario.

![](/assets/images/Different/image017.png)

Podemos observar que los recurso compartidos por el usuario hakanftp vienen a ser archivos y subdirectorios que corresponden a la estructura de directorios de un sitio o aplicación web construido con el CMS WordPress. Ademas, todos estos recursos compartidos parecen estar almacenado en el directorio home de algun usuario local debido a que uno de los recursos compartido viene a ser el archivo oculto .bash_history.

## FASE GANAR ACCESSO O EXPLOTACION

Ahora intentaremos subir un archivo PHP en los recursos compartidos con el fin de observarlo a traves de nuestro navegador web y sea ejecutado por el servidor web, y nos genere una reverse shell. Ademas, no nos olvidemos de asignarle el permiso de ejecucion para que pueda ser ejecutado.

![](/assets/images/Different/image018.png)

![](/assets/images/Different/image019.png)

![](/assets/images/Different/image020.png)

Podemos obsevar que nos topamos con alguno confuso ya que al parecer los recursos compartidos no le pertenecen al directorio raiz de esta sitio web. Por esta razon nos sale el codigo de estado HTTP 404, es decir, no se ha encontrado el recurso en el directorio raiz del sitio web. Por lo que podemos deducir que se ha implementado la tecnica virtual hosts, y los recursos compartidos le corresponden a otro directorio raiz de otro sitio web que esta alojado en el mismo servidor web. Ademas, este sitio web del que desconocemos su nombre de dominio, tambien ha sido creado usando el CMS WordPress.

Ahora teniendo la experiencia de haberme topado con varios sitios web creados con el CMS WordPress, podemos saber que casi siempre el archivo wp-config.php contiene credenciales de la base de datos que utilizara el CMS para almacenar las credenciales de los usuarios que pueden acceder a su portal de administracion o gestion.

![](/assets/images/Different/image021.png)

Podemos observar que logramos encontrar unas credenciales para acceder a la aplicación phpMyAdmin que viene a ser comumente utilizado por el CMS WordPress con el fin de poder gestionar sus bases de datos MySQL a traves de una interfaz web.

Ahora accederemos a la interfaz web de la aplicación phpMyAdmin con el fin de interactuar con la base de datos phpmyadmin1 donde se alojara cosas del sitio o aplicación web.

![](/assets/images/Different/image022.png)

Dentro del portal de gestion de la aplicación, nos topamos con dos bases de datos llamadas phpmyadmin y phpmyadmin1. Con esto podemos asegurar que si hay dos sitios web que han sido creados con el CMS WordPress. Luego, gracias a la experiencia auditando sitios web creados con este CMS, sabemos que en la bases de datos utilzados por sitios web, suele haber una columna llamada option_value en la tabla wp-options que contiene la URL del sitio web.

![](/assets/images/Different/image023.png)

Podemos observar que la URL de este stio web creado por el CMS WordPress, es un subdominio del dominio del otro sitio web.

Ahora agregaremos este subdominio asociada a la dirrecion IP del servidor web en nuestro archivo /etc/hosts, y luego intentaremos observar el archivo PHP, que subimos anterioemente, para que sea ejecutado por el servidor web y nos genere nuestra reverse shell.

![](/assets/images/Different/image024.png)

![](/assets/images/Different/image025.png)

Ademas, tambien podemos intentar crackear los hashes del usuario hakanbey01 ya que cada sitio web consta de ese mismo usuario pero difiere en los hashes, por lo que intentaremos almenos crackear un hash con el fin de poder usar sus contraseñas en los siguientes escenarios que nos topemos.

![](/assets/images/Different/image026.png)

![](/assets/images/Different/image027.png)

![](/assets/images/Different/image028.png)

Podemos observar la funcion hash utilizada en los hahses, es phpass y solo logramos crackear el primer hash. Ademas, el hash crackeado le corresponde al usuario hakanbey01 en el sitio web subdomain.adana.thm.

## FASE ESCALADA PRIVILEGIOS
Ahora volviendo al escenario anterior, intentaremos buscar vectores de escalada de privilegios con el fin de escalar privilegios, pero antes realizaremos un tratamiento de nuestra reverse shell.

![](/assets/images/Different/image029.png)

![](/assets/images/Different/image030.png)

![](/assets/images/Different/image031.png)

Podemos observar que hay un usuario local llamdo hakanftp y su direcftorio home viene a ser el directorio raiz de uno de los sitios web creado con el CMS WordPress. Ademas, si utilizamos las mismas credenciales utilizada para acceder al servidor FTP, pero ahora para iniciar sesion en el sistema destino, nos damos cuenta que si funciono. De esta manera realizamos una escala de privilegio horizontal.

Ahora en el sistema de archivos de este usuario local, no logre encontrar otro vector de escalada de privielgios, pero note que el password de hakanftp y de la palabra clave, utilizada para extraer los dato ocultos de la imagen, comienzan con la palabra 123adana. Por lo que intentare realizar fuerza bruta sobre la cuenta del usuario local hakanbey mediante la herramineta sucrack, usando el wordlist encontrado anteriormente, pero le agregare la palabra 123adana a todas las palabras del wordlist.

![](/assets/images/Different/image032.png)

![](/assets/images/Different/image033.png)

![](/assets/images/Different/image034.png)

![](/assets/images/Different/image035.png)

![](/assets/images/Different/image036.png)

Podemos observar que logramos encontrar el password del usuario local hakanbey.

Ahora iniicaremos sesion en el sistema destino con las credenciales del usuario hakanbey.

![](/assets/images/Different/image037.png)

De esta manera realizamos una escalada de privilegio vertical. Ahora, seguiremos buscando vectores de escalada de privilegio para ser el usuario root.

![](/assets/images/Different/image038.png)

![](/assets/images/Different/image039.png)

![](/assets/images/Different/image040.png)

![](/assets/images/Different/image041.png)

Podemos observar que al enumerar los archivos binarios con el bit SUID habilitado, nos topamos con dos archivos binarios bastante atipicos donde tenemos el permiso de escritura habilitado en ambos. Ademas, si ejecutamos el primer archivo binario, nos topamos con que nos solicitan un string, y si no le proveemos el correcto string, se nos cierra la sesion con hakanbey. Luego, utilizamos el comando ltrace con el fin de observar las funciones strcat que se ejecutan en segundo plano al momento de ejecutar el archivo binario, asi como sus valores de retorno y los argumentos que le pasan.

De esta manera podemos deducir que el string solicitado puede ser warzoneinadana. Ahora, pondremos a prueba nuestra hipotesis.

![](/assets/images/Different/image042.png)

![](/assets/images/Different/image043.png)

Podemos observar en el resultado de la ejecucion del binario, el comando copy para copiar una imagen del directorio root al directorio home de hakanbey. Ademas, nos dan una pista de usar la herramienta hexeditor para observar los bytes localizados en la posicion 00000020. Luego, debemos usar CyberChef para decodificar esos bytes de esa posicion.

![](/assets/images/Different/image044.png)

![](/assets/images/Different/image045.png)

![](/assets/images/Different/image046.png)

Podemos observar que luego decodificar los bytes mediante la decodificacion Hex y la codificacion Base85, obtenemos las credenciales de root. De esta menra logramos ser el usuario root.

Ahora buscaremos las dos banderas en los archivos root.txt y user.txt.

![](/assets/images/Different/image047.png)

Otra manera que pudimos haber obtado por analizar el funcionamiento interno del archivo binario, seria utilizando Ghidra con el fin de observar su codigo fuente en C.

![](/assets/images/Different/image048.png)

![](/assets/images/Different/image049.png)

Donde podemos observar el uso de la funcion strcat para concatenar los valores establecidos en las variables local_1d8, local_1c8, local_1b8, local_1a8, en la variable local_1e8. Luego, se utiliza la funcion scanf para leer los caracteres que ingresamos cuando nos solicitan ingresar el string correcto, y luego la funcion los almacena en la variable local_138. Luego, con la funcion strcmp se realiza una comparacion de los valores almacenados en la variable local_1e8 y local_138, y si el resultado del la funcion es 0, es decir, si ambas variables tienen el mismo valor almacenado, se nos muestra el contenido del archivo hint.txt y se ejecuta el comando copy para copiar una imagen del directorio root al directorio home del usuario hakanbey.

De esta manera tambien hubieramos podido saber el string correcto que deberiamos ingresar teniendo en cuenta que los valores de las variables predfinidas en el progama, se almacenan en la memoria siguiendo el formato llitle-endian.



































