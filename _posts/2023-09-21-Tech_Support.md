---
layout: single
title: Tech_Supp0rt:1 - TryHackMe
excerpt: "Debemos encontrar 1 bandera en la maquina Tech_Supp0rt: 1. Donde a partir del escaneo de puertos, pude saber que hay un servidor HTTP, SSH, Samba. Luego, utilice un script de Nmap para enumerar los recursos compartidos del servidor SMB, encontrando un directorio que contenía un archivo con unas credenciales para acceder al panel de administración del CMS Subrion, donde pude explotar la vulnerabilidad File Upload para cargar un archivo PHP y generar una reverse Shell, que me permitió acceder al sistema destino. Luego, para la escalada privilegio horizontal, utilice Linpeas, encontrando una credencial en un archivo de configuración del CMS Wordpress. Luego, para ser root, utilice el comando iconv, que podía ejecutarlos con los privilegios de root, permitiéndome modificar el contenido la línea correspondiente a root en el archivo /etc/shadows."
date: 2023-09-21	
classes: wide
header:
  teaser: /assets/images/Tech/image002.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - CyberChef
  - File Upload
  - HTTP
  - SSH
  - Samba
  - Burp Suite
  - Wfuzz
  - PHP
  - CMS Subrion
  - Sudo
  - icovn
---

![](/assets/images/Tech/image001.png)

## SUMMARY

Debemos encontrar 1 bandera en la maquina Tech_Supp0rt: 1. Donde a partir del escaneo de puertos, pude saber que hay un servidor HTTP, SSH, Samba. Luego, utilice un script de Nmap para enumerar los recursos compartidos del servidor SMB, encontrando un directorio que contenía un archivo con unas credenciales para acceder al panel de administración del CMS Subrion, donde pude explotar la vulnerabilidad File Upload para cargar un archivo PHP y generar una reverse Shell, que me permitió acceder al sistema destino. Luego, para la escalada privilegio horizontal, utilice Linpeas, encontrando una credencial en un archivo de configuración del CMS Wordpress. Luego, para ser root, utilice el comando iconv, que podía ejecutarlos con los privilegios de root, permitiéndome modificar el contenido la línea correspondiente a root en el archivo /etc/shadows.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener las banderas. Para ello empezaremos utilizando el comando openvpn con el fin de establecer una conexión VPN con la red virtual dónde está la máquina Tech_Supp0rt: 1. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/Tech/image003.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/Tech/image004.png)

Además, la plataforma de Tryhackme nos muestra la dirección ip de la máquina Tech_Supp0rt: 1.

![](/assets/images/Tech/image005.png)

## FASE ESCANEO Y ENUMERACION

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina Tech_Supp0rt: 1. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:

- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El parámetros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/Tech/image006.png)

Los resultados que obtuvimos del escaneo vienen a ser:

- Un servicio HTTP o web levantado en el puerto 80, y el programa servidor web que está corriendo, y la versión del programa servidor. Además, gracias al script http-title se puedo obtener el valor de la etiqueta title de la página web que viene por defecto al instalar el programa servidor.
- Un servicio SSH levantado en el puerto 22, y el programa servidor que está corriendo, y la versión del programa servidor.
- Un servidor Samba levando en el puerto 445 y 139, y la versión del programa servidor. Además, podemos observar el hostname y el sistema operativo del sistema destino.

Ahora enumeraremos los recursos compartidos por el servidor Samba. Para ello utilizaremos el script smb-enum-shares de Nmap.

![](/assets/images/Tech/image007.png)

![](/assets/images/Tech/image008.png)

Podemos observar que el script logro enumerar tres recursos compartidos siendo el usuario gutest, donde uno de ellos es un directorio llamado websvr. Además, podemos observar que un usuario anonimo tiene los permisos de escritura y lectura sobre el directorio websvr.

Ahora accederemos al directorio websvr mediante una conexión anónima, es decir, sin introducir credenciales de inicio de sesión.

![](/assets/images/Tech/image009.png)

![](/assets/images/Tech/image010.png)

Podemos observar que el directorio websvr contenía un archivo llamado enter.txt que contiene unas credenciales, que posiblemente son para acceder al portal de administración del CMS Subrion. Además, en el archivo se indica que un sitio web, que ha sido desarrollado utilizando el CMS WordPress, y otro sitio web, que ha sido desarrollado utilizando el CMS Subrion pero hay problemas para acceder a él.

Ahora intentaremos acceder al sitio web, que ha sido creado utilizando el CMS Subrion, a través de mi navegador web.

![](/assets/images/Tech/image011.png)

![](/assets/images/Tech/image012.png)

Podemos observar que cuando intentamos acceder al sitio web, nuestra URL cambio, es decir, se nos redirige a otra ruta distintica a la que especificamos inicialmente y que contiene una dirección IP distinta al servidor web y que pertenece probablemente a otro sistema que no está en la subred a la cual nos correctamente con la VPN de TryHackMe.

Esto lo podemos entender de mejorar manera si capturamos la solicitud GET, que realiza nuestro navegador web hacia el servidor web para acceder al sitio web, con Burp Suite.

![](/assets/images/Tech/image013.png)

Podemos observar que la respuesta HTTP del servidor web viene a contener el código de estado 302 HTTP con el fin de indicar que el recurso solicitado ha sido movido temporalmente a otra ruta y utiliza la cabecera HTTP Location para indicarle la ruta a la cual mi navegador web va a ser redirigido para que acceda al recurso, pero he aquí el problema debido a que el sistema 10.0.2.15 no esta en nuestra misma sudred, por lo tanto, no podemos conectarnos a él y tampoco acceder al recurso, pero el mensaje del archivo enter.txt nos indica que arreglemos el problema, accediendo al panel de administración del CMS. Por lo tanto, esto da suponer que solo el sitio web es que el esta mal configurado su ruta.

Ahora accederemos al panel de administración del CMS.

![](/assets/images/Tech/image014.png)

Podemos observar que se requiere autenticación para acceder al portal de administración del CMS. Para ello utilizaremos las credenciales proveídas anteriormente, pero antes debemos decodificar la contraseña que nos han proporcionado, teniendo en cuneta que nos han dicho que ha sido decodificado utilizando una fórmula mágica.

Ahora debido a que no tenemos idea de que tipo de codificación ha sido utilizado sobre la contraseña, utilizaremos la operación Magic de CyberChef, para que nos detecte las operaciones que se han utilizado para codificar nuestros datos y que operaciones podemos utilizar para decodificarlos.

![](/assets/images/Tech/image015.png)

![](/assets/images/Tech/image016.png)

Podemos observar que la operación Magic logro decirnos los tipos de codificación que se habían realizado sobre nuestra contraseña, y gracias a ello logramos decodificarlo fácilmente.

Ahora nos autenticaremos en el formulario del CMS Subrion para acceder a su portal de administración.

![](/assets/images/Tech/image017.png)

![](/assets/images/Tech/image018.png)

Podemos observar que logramos acceder al portal de administración exitosamente. Además, dentro del portal de administración tenemos la opción de cargar archivos para agregar contenido al sitio o aplicación web que queremos crear.

## FASE GANAR ACCESO O HACKING O EXPLOTACION

Ahora aprovecharemos la vulnerabilidad File Upload, para cargar un archivo PHP, ya que el lenguaje back-end del CMS y del servidor web es PHP, con el fin de ejecutar una reverse Shell cuando se ejecute nuestro archivo PHP. Además, también podemos observar un exploit en ExploitDB, desarrollados para explotar esta vulnerabilidad de manera automatizada, y con el fin de analizar su código fuente y poder saber que tipo de archivos permite el CMS cargar.

![](/assets/images/Tech/image019.png)

![](/assets/images/Tech/image020.png)

![](/assets/images/Tech/image021.png)

Podemos observar en el código fuene del exploit que el payload que utiliza, es decir, el archivo que va a cargar va a tener la extensión phar, que viene a ser otro tipo de extensión utilizado en archivos PHP.

Ahora cargaremos nuestro archivo PHP con la extensión phar.

![](/assets/images/Tech/image022.png)

Podemos observar que el CMS nos permitió cargar nuestro archivo PHP.

Ahora accederemos a la ruta donde se ha cargado nuestro archivo con el fin de que el servidor web destino ejecute la reverse Shell, pero antes debemos habilitar el puerto 1500 en nuestro sistema para que este en modo listening y esperando la conexión entrante.

![](/assets/images/Tech/image023.png)

![](/assets/images/Tech/image024.png)

Podemos observar que logramos obtener acceso al sistema destino siendo el usuario www-data.

![](/assets/images/Tech/image025.png)

Además, podemos observar que al usuario www-data se le ha configurado para que utilice la Shell nologin cuando inicie sesión en el sistema destino. Esta Shell viene a ser no interactiva e inestable. Por lo tanto, realizaremos un tratamiento de la tty para tener una Shell estable e interactiva.

![](/assets/images/Tech/image026.png)

## FASE ESCALADA PRIVILEGIOS

Ahora debemos buscar un vector de escalada de privilegios con el fin de ser un usuario con privilegios más elevados. Para ello utilizaremos linpeas, que nos ayudara a encontrar vectores de escalada privilegios de manera automatizada.

![](/assets/images/Tech/image027.png)

Podemos observar que la herramienta llego a encontrar unas credenciales para acceder a la base de datos wpdb en el archivo wp-config.php del CMS Wordpress. Por lo tanto, la base de datos wpdb va ser utilizado por el CMS Wordpress para almacenar información sobre los usuarios que pueden acceder al portal de administración del CMS.

Ahora vamos a volcar los registros de la tabla de la base de datos wpdb que contiene información a través del cliente mysql.

![](/assets/images/Tech/image028.png)

![](/assets/images/Tech/image029.png)

Podemos observar que logramos volcar todos los registros de la tabla wp_users. Donde encontramos las credenciales del mismo usuario, pero la contraseña esta en forma de hash.

Ahora crackearemos este hash con Hashcat con el fin de afirmar que el usuario support tiende a reutilizar sus contraseñas.

![](/assets/images/Tech/image030.png)

![](/assets/images/Tech/image031.png)

![](/assets/images/Tech/image032.png)

Podemos observar que nuestra hipótesis estaba en lo cierto.

Ahora supondremos que el usuario support viene a ser el usuario local scamsite en el sistema destino, y utilizaremos la credencial encontrada anteriormente para autenticarnos en el sistema destino.

![](/assets/images/Tech/image033.png)

Podemos observar que nuestra hipótesis estaba en lo cierto, y logramos realizar una escalada privilegio horizontal.

Ahora debemos buscar un vector de escalada de privilegios con el fin de ser un usuario con privilegios más elevados. Para ello enumeraremos todos los comandos que el usuario scamsite puede ejecutar con los privilegios de otro usuario con mayores privilegios.

![](/assets/images/Tech/image034.png)

Podemos observar que el usuario scamsite puede ejecutar el archivo binario ejecutable iconv con los privilegios de root. Además, debemos saber que este comando es utilizado para convertir la codificación de caracteres de un archivo de un formato a otro en sistemas Linux. Además, la sintaxis básica de este comando es:

![](/assets/images/Tech/image035.png)

Ahora que conocemos la sintaxis básica, podemos aprovecharnos de el para no especificar los parametros f y t y tampoco utilizar el redirector de salida con el archivo de salida con el fin de observar solo el contenido de un archivo como /etc/shadow, que normalmente requiere los privilegios de root para observar su contenido.

![](/assets/images/Tech/image036.png)

Ahora que podemos observar el contenido del /etc/shadow, copiaremos su contenido en un archivo de nuestro sistema, y modificaremos la línea con respecto al usuario, agregando un hash conocido. Luego, utilizaremos el comando iconv para sobrescribir su contenido en el archivo /etc/shadow original del sistema destino.

![](/assets/images/Tech/image037.png)

![](/assets/images/Tech/image038.png)

![](/assets/images/Tech/image039.png)

![](/assets/images/Tech/image040.png)

Podemos observar que logramos modificar exitosamente la línea asociado con root en el archivo /etc/shadow del sistema destino, permitiéndonos acceder al sistema destino mediante la contraseña nueva que hemos establecido.

Ahora buscaremos la bandera en el directorio root.

![](/assets/images/Tech/image041.png) 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































