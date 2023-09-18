---
layout: single
title: CyberCrafted - TryHackMe
excerpt: "Debemos encontrar 4 banderas en la maquina CyberCrafted. Donde a partir del escaneo de puertos pude saber que hay un servidor Web, Minecraft, y SSH. Además, a partir de un script de Nmap pude saber de una página web con cierto nombre de dominio que lo utilicé para realizar una enumeración de subdominios, llegando a encontrar un subdominio de una página web, que contenía un login, y un subdominio de una tienda en línea, que no podía acceder, pero realice fuerza bruta de directorios sobre el directorio raíz de la aplicación web, encontrando un recurso PHP que era vulnerable a SQLi Blind, permitiéndome volcar los registros de la base de datos, y obtener acceso al sistema destino. Luego, realice doble escalda privilegio horizontal a partir de una clave privada SSH encriptada y una credencial encontrada en un log a partir de una tarea cron. Luego, para ser root, utilice el comando screen que podía ejecutarlo con los privilegios de root."
date: 2023-09-18	
classes: wide
header:
  teaser: /assets/images/Cyber/image002.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - SSH
  - Minecraft
  - Hashcat
  - John The Ripper
  - SQLi
  - Screen
  - Sudo
  - Crontab
  - Gobuster
  - Wfuzz
  - PHP
  - Nmap
---

![](/assets/images/Cyber/image001.png)

## SUMMARY

Debemos encontrar 4 banderas en la maquina CyberCrafted. Donde a partir del escaneo de puertos pude saber que hay un servidor Web, Minecraft, y SSH. Además, a partir de un script de Nmap pude saber de una página web con cierto nombre de dominio que lo utilicé para realizar una enumeración de subdominios, llegando a encontrar un subdominio de una página web, que contenía un login, y un subdominio de una tienda en línea, que no podía acceder, pero realice fuerza bruta de directorios sobre el directorio raíz de la aplicación web, encontrando un recurso PHP que era vulnerable a SQLi Blind, permitiéndome volcar los registros de la base de datos, y obtener acceso al sistema destino. Luego, realice doble escalda privilegio horizontal a partir de una clave privada SSH encriptada y una credencial encontrada en un log a partir de una tarea cron. Luego, para ser root, utilice el comando screen que podía ejecutarlo con los privilegios de root.

## RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener las banderas. Para ello empezaremos utilizando el comando openvpn con el fin de establecer una conexión VPN con la red virtual dónde está la máquina CyberCrafted. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/Cyber/image003.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/Cyber/image004.png)

Además, la plataforma de Tryhackme nos muestra la dirección ip de la máquina CyberCrafted.

![](/assets/images/Cyber/image005.png)

## FASE ESCANEO Y ENUMERACION

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina CyberCrafted. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:

- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El parámetros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/Cyber/image006.png)

Los resultados que obtuvimos del escaneo vienen a ser:

- Un servicio HTTP o web levantado en el puerto 80, y el programa servidor web que está corriendo, y la versión del programa servidor. Además, gracias al script http-title se puedo obtener el valor de la etiqueta title de una pagina web almacenada en el servidor web, que contiene un nombre de dominio. Además, al parecer cuando intentemos ingresar a la página web a través de la dirección IP del servidor web, se nos redireccionara a ese  nombre de dominio.
- Un servidor Minecraft levantado en el puerto 25565, y que está usando la versión 1.7.2.
- Un servicio SSH levantado en el puerto 22, y el programa servidor web que esta corriendo, y la versión del programa servidor.

Ahora accederemos a la página web desde mi navegador web.

![](/assets/images/Cyber/image007.png)

![](/assets/images/Cyber/image008.png)

Podemos observar que la pagina web nos dice que el servidor Minecraft y una tienda en línea están aun en desarrollo. Además, si observamos el código fuente de la página web, nos topamos un mensaje de un desarrollador que nos incita a realizar una enumeración de subdominios.

Ahora realizaremos una enumeración de subdominios utilizando Wfuzz y sobre el nombre de dominio cybercrafted.thm.

![](/assets/images/Cyber/image009.png)

![](/assets/images/Cyber/image010.png)

Podemos observar que nos topamos con muchos falsos positivos, es decir con muchas respuestas HTTP con un código de estado 302 HTTP que supuestamente podemos acceder, pero no es así.

Ahora vamos a filtrar todas estas respuestas HTTP teniendo en cuenta que todas estas respuestas HTTP presenta 0 caracteres, es decir, 0 Ch.

![](/assets/images/Cyber/image011.png)

Podemos observar que la herramienta llego a encontrar varios subdominios, aunque no estamos autorizados para acceder al subdominio store.

Ahora accederemos a estos subdominios a través de mi navegador web, pero antes debemos asociarlo con la dirección IP del servidor web en el archivo /etc/hosts con el fin de que nuestro sistema evite realizar una consulta DNS a los servidores DNS recursivos proveídos por nuestra ISP.

![](/assets/images/Cyber/image012.png)

admin.cybercrafted.thm :

![](/assets/images/Cyber/image013.png)

![](/assets/images/Cyber/image014.png)

![](/assets/images/Cyber/image015.png)

Podemos observar que este subdominio viene a ser una pagina web que consta de un formulario HTML o login. Además, si realizamos una autenticación fallida en el login, obtendremos el mensaje de error y aparecerá un parámetro GET en nuestra URL, que controlará el mensaje de error. Debido a que el valor del parámetro GET error se reflejada en el código fuente de la pagina web, nos incita a probar si es vulnerable a XSS, y llegamos a determinar que, si es vulnerable a través de un payload XSS simple, pero debido a que ningún usuario va a interactuar con nuestro payload, es inútil seguir por esta vía.

Ahora realizaremos una fuerza bruta de directorios sobre el subdominio store con el fin de encontrar algún recurso escondido en el directorio raíz de la aplicación web que va a ser una tienda en línea.

![](/assets/images/Cyber/image016.png)

Podemos observar que la herramienta llego a encontrar un recurso escondido.

Ahora accederemos al recurso a través de mi navegador web.

![](/assets/images/Cyber/image017.png)

Podemos observar que el recurso se trata de una aplicación web que nos permite realizar búsquedas sobre ciertos ítems, y nos mostrara información sobre ellos. Además, para que la aplicación web nos muestre información sobre los ítems, este debe estar realizando consultas SQL hacia su base de datos.

## FASE GANAR ACCESO O EXPLOTACION O HACKING

Ahora probaremos si la aplicación web es vulnerable a SQLi. Para ello insertaremos símbolos en el cuadro de búsqueda con el fin de obtener errores generados por el DBMS.

![](/assets/images/Cyber/image018.png)

![](/assets/images/Cyber/image019.png)

Podemos observar que la aplicación web presenta la vulnerabilidad SQLi Blind ya que no se llegan a mostrar mensaje de errores por parte del DBMS, pero aun así nuestras consultas SQLi maliciosas se concatenan directamente con las consultas SQL que realiza la aplicación web para mostrar información en los cuadros de los ítems solicitados.

Ahora realizaremos una enumeración de todas las bases de datos gestionadas y creadas por el DBMS instalada en el sistema destino.

![](/assets/images/Cyber/image020.png)

Podemos observar la base de datos webapp, que viene a ser la utilizada por la aplicación web, ya que las demás bases de datos se crean por defecto al instalar el DBMS.

Ahora enumeraremos las tablas de webapp.

![](/assets/images/Cyber/image021.png)

Podemos observar que webapp consta de dos tablas.

Ahora enumeraremos las columnas de la tabla admin.

Tabla admin:

![](/assets/images/Cyber/image022.png)

Podemos que la tabla admin consta de tres columnas.

Ahora volcaremos los registros de esta tabla.

![](/assets/images/Cyber/image023.png)

Podemos observar que nos topamos con la primera bandera, y con un username, que le pertenece al usuario admin, y un hash.

Ahora utilizaremos Hashcat para crackear el hash, pero antes debemos saber cuál es la función o algoritmo Hash que utiliza.

![](/assets/images/Cyber/image024.png)

![](/assets/images/Cyber/image025.png)

![](/assets/images/Cyber/image026.png)

Podemos observar que llegamos a saber que la función Hash utilizada es SHA-1, y la herramienta llego a crackear el hash.

Ahora utilizaremos las credenciales encontradas para autenticarnos en el formulario HTML que encontramos en el subdominio admin.

![](/assets/images/Cyber/image027.png)

![](/assets/images/Cyber/image028.png)

Podemos observar que logramos autenticarnos exitosamente. Además, accedimos a una aplicación web que nos permite ejecutar comandos en el sistema operativo del sistema destino remotamente, algo parecido a un RCE o Command Injection.

Ahora ejecutaremos un comando que nos generara una reverse Shell, pero antes debemos habilitar el puerto 1500 en nuestro sistema para que este en modo listening y aceptando la conexión entrante de la reverse Shell.

![](/assets/images/Cyber/image029.png)

![](/assets/images/Cyber/image030.png)

De esta manera obtenemos acceso al sistema destino mediante la explotación de la vulnerabilidad SQLI y RCE, y siendo el usuario www-data.

## FASE ESCALADA DE PRIVILEGIOS 

Ahora debemos buscar un vector de escalada de privilegios, para ello realizaremos una enumeración manual sobre los directorios del sistema de archivos del sistema destino.

![](/assets/images/Cyber/image031.png)

![](/assets/images/Cyber/image032.png)

Podemos observar que se han generado unas claves SSH para el usuario local xxultimatecreeperxx con el fin de que pueda iniciarse sesión mediante su clave privada SSH, que esta encriptada.

Ahora copiaremos el contenido de la clave privada en nuestro sistema, e intentaremos crackearla con John The Ripper con el fin de obtener su clave.

![](/assets/images/Cyber/image033.png)

Podemos observar que la herramienta llego a obtener la clave para descifrar la clave privada SSH.

Ahora iniciaremos sesión en el sistema destino mediante la clave privada SSH y siendo el usuario xxultimatecreeperxx.

![](/assets/images/Cyber/image034.png)

De esta manera realizamos una escalada de privilegio horizontal siendo el usuario local xxultimatecreeperxx.

Ahora debemos buscar un vector de escalada de privilegios, para ello observaremos el contenido del archivo crontab general.

![](/assets/images/Cyber/image035.png)

Podemos observar que el usuario cybercrafted ha programado una tarea cron para que se ejecute un comando de manera periódica. Adema, si analizamos el comando, nos damos cuenta que utiliza el comando tar para comprimir todos los archivos del directorio /opt/minecraft/cybercrafted/world/ en un archivo tar, y luego se utilizara el algoritmo de compresión gzip sobre el archivo tar para reducir su tamaño. Además, debido a que se utiliza el wilcard o comodín, nos incitar a realizar una Wilcard Injection.

![](/assets/images/Cyber/image036.png)

Pero si enumeramos los permisos del directorio world, nos damos cuenta de que no tenemos los permisos para crear archivos dentro del directorio, por lo tanto, no podemos realizar el Wilcard Injection.

Ahora realizaremos una enumeración sobre los directorios del sistema de archivos del sistema destino.

![](/assets/images/Cyber/image037.png)

Podemos observar que nos encontramos con la segunda bandera, y con una nota dejada por el usuario local cybercrafted, donde nos hablan sobre la instalación de un plugin dentro del servidor Minecraft para permitir el acceso de usuarios que no tienen una cuenta premium.

Ahora realizaremos una enumeración sobre los archivos del plugin instalado.

![](/assets/images/Cyber/image038.png)

Podemos observar que el usuario local cybercrafted creo un plugin personalizable a través de la API proporcionada por el proyecto Bukkit. Además, nos encontramos con un registro de eventos dentro de los archivos del plugin creado, donde podemos observar las credenciales insertadas por el usuario local cybercrafted para autenticarse en el servidor Minecraft, que ha sido ejecutado por el usuario root como lo podemos observar si enumeramos los procesos que se ejecutan en el sistema destino.

![](/assets/images/Cyber/image039.png)

Podemos observar varias cosas en el proceso donde se está ejecutando el servidor Minecraft cuyo identificador es el 988:

- Se ha ejecutado un servidor Minecraft personalizado por la comunidad a partir de la versión 1.7.2 del servidor oficial lanzado en la plataforma Vanilla. Además, este servidor modificado se ha descargado a partir de la plataforma Craftbukkit.
- Se ha utilizado el parámetro nogui con el fin de que se acceda al servidor Minecraft no se provea una interfaz grafica de usuario sino se proveerá una interfaz de línea de comandos para administrar el servidor.
- Se ha utilizado el comando screen con el fin de ejecutar el servidor Minecraft en una sesión de terminal aparte de la ventana de terminal donde se ejecutó el comando, y que consta con sus propio prompt de comando y ambiente de ejecución.
- Se está utilizando el parámetro -m de screen con el fin de que otros usuarios locales puedan ingresar a la sesión donde se ejecutar el servidor, y el parámetro -S es para especificar el nombre de la sesión donde se ejecutar el servidor Minecraft.

Ahora supondremos que el usuario local cybercrafted ha reutilizado sus credenciales para iniciar sesión en el sistema destino, por ello, probaremos sus credenciales para iniciar sesión.

![](/assets/images/Cyber/image040.png)

De esta manera realizamos una escalada de privilegio horizontal siendo el usuario local cybercrafted.

Ahora debemos buscar un vector de escalada de privilegios, para ello enumeraremos los comandos que el usuario local cybercrafted puede ejecutar con los privilegios de root.

![](/assets/images/Cyber/image041.png)

Podemos observar que el usuario local cybercrafted puede ejecutar ese comando con los privilegios de root y con el fin de recuperar la sesión de terminal donde se ejecuto el servidor Minecraft.

Ahora ejecutaremos este comando para recuperar la sesión de terminal cybercrafted donde se ejecuto el servidor Minecraft.

![](/assets/images/Cyber/image042.png)

Podemos observar que se obtuvo como una interfaz de línea de comandos para administrar el servidor Minecraft debido a que cuando se ejecutó el servidor en la sesión de terminal cybercrafted se utilizó el parámetro nogui con el fin de no se provee una interfaz grafica de usuario o la interfaz grafica del juego cuando se accede al servidor Minecraft.

Ahora aprovecharemos de que se puede ejecutar una nueva ventana de terminal dentro de la sesión de terminal cybercrafted a través del comando Ctlr+a+c. Además, esta ventana de terminal va a ser una Shell bash con los privilegios del usuario root.

![](/assets/images/Cyber/image043.png)

De esta manera realizamos una escalada de privilegio vertical siendo root.

Ahora buscaremos las dos banderas restantes en el archivo root.txt y user.txt.

![](/assets/images/Cyber/image044.png) 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































