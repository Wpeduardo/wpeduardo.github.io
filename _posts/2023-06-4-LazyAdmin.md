---
layout: single
title: LazyAdmin - TryHackMe
excerpt: "En este caso debemos encontrar dos banderas ubicadas en la maquina LazyAdminFinal. Donde a partir del script http-enum de Nmap encontré el recurso content, que viene a ser la pagina de inicio predeterminada del CMS SweetRice. Luego utilizo wfuzz para encontrar recursos ocultos en el directorio raíz de la aplicación CMS. Llegando a encontrar el recurso as que viene a ser un login o formulario para entrar a la plataforma del CMS. Además, encontré el recurso inc que viene a ser un directory list que contenía un archivo de respaldo de una base de datos MySQL. Donde encontré unas credenciales que las utilice para autenticarme en el formulario. Luego aprovechándome que la aplicación CMS permite subir plugins, themes, o modificar el archivo del código fuente de un theme es que puede subir un script PHP que me genere el acceso a la maquina objetivo. Luego para la escalada de privilegios, modifique un comando de un archivo perl que podía ejecutar teniendo los privilegios del usuario root."
date: 2023-06-04
classes: wide
header:
  teaser: /assets/images/LazyAdmin/image003.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - linux  
  - openvpn
  - nmap
  - http
  - ssh
  - wfuzz
  - MySQL
  - cmssweetrice
  - hashidentifier
  - johntheripper
  - php
  - perl
  - sudo
---

![](/assets/images/LazyAdmin/image001.png)

## Summary

En este caso debemos encontrar dos banderas ubicadas en la maquina `LazyAdminFinal`. Donde a partir del script `http-enum.nse` de Nmap  encontré el recurso `content`, que viene a ser la pagina de inicio predeterminada del `CMS SweetRice`. Luego utilizo `wfuzz` para encontrar recursos ocultos en el directorio raíz de la aplicación CMS. Llegando a encontrar el recurso `as` que viene a ser un login o formulario para entrar a la plataforma del CMS. Además, encontré el recurso `inc` que viene a ser un directory list que contenía un archivo de respaldo de una base de datos `MySQL`. Donde encontré unas credenciales que las utilice para autenticarme en el formulario. Luego aprovechándome que la aplicación CMS permite subir `plugins`, `themes`, o modificar el archivo del código fuente de un theme es que puede subir un script PHP que me genere el acceso a la maquina objetivo. Luego para la escalada de privilegios, modifique un comando de un archivo `perl` que podía ejecutar teniendo los privilegios del usuario root.

## Fase Reconocimiento

Para resolver esta maquina empezaremos utilizando el comando `openvpn` con el fin de establecer una conexión VPN con la red virtual dónde está la máquina LazyAdminFinal. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/LazyAdmin/image005.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/LazyAdmin/image007.png)

Además, la plataforma de Tryhackme nos muestra la dirección de la máquina LazyAdminFinal.

![](/assets/images/LazyAdmin/image009.png)

## Fase Escaneo y Enumeracion

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios levantados en los puertos abiertos de la máquina LazyAdminFinal.Para ello utilizaremos `Nmap`.Donde le pasaremos los siguientes parametros:

- El parametro `-sC`  o `–script=”default”` para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El `-sV` con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parametro `-n` para evitar la resolución DNS, y el parametro `–min-rate` para indicarle el número de paquetes por segundo que va utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por Nmap.
- El parametro `-p-` para realizar un escaneo de los 65535 puertos del nodo y el parametro `–open` con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/LazyAdmin/image011.png)

![](/assets/images/LazyAdmin/image013.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Hay un servicio `HTTP` que se esta levantado en el puerto 80, y el  programa servidor HTTP que se esta corriendo.Además, la versión del programa servidor HTTP.
- Hay un servicio `SSH` que se esta levantado en el puerto 22, y el  programa servidor SSH se está corriendo.Además, la versión del programa servidor SSH.

Ahora ejecutaremos los scripts de Nmap de la categoría vuln con el fin de conocer las vulnerabilidades de los servicios que están levantados en los puertos abiertos.

![](/assets/images/LazyAdmin/image015.png)

![](/assets/images/LazyAdmin/image017.png)

Podemos notar que se ha el utilizado script `http-enum.nse` de Nmap, que realiza un fuerza bruta de directorios con una lista de nombres posibles de directorios y archivos con el fin de encontrar archivos y directorios escondidos, llegandose a obtener el recurso content, que está almacenado en el directorio raíz del servidor web.

Ahora accederemos al recurso `content` a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image019.png)

Nos damos cuenta que el recurso viene a ser la pagina de inicio predeterminada del CMS `SweetRice`,que viene a ser un sistema de gestion de contenido. y que nos permite crear y gestionar contenido en linea, que puede ser sitios o aplicaciones web.Por lo tanto,el servidor web esta alojando la aplicacion web SweetRice.

Ahora utilizaremos la herramienta `wfuzz` ,que también realiza fuerza bruta de directorios utilizando un lista de posibles nombre de directorios y archivos, pero desde la ruta del recurso content para que la herramienta encuentre recursos escondidos en el directorio raiz de la aplicacion web SweetRice. Además, le pasaremos los siguientes parametros a la herramienta:
- El parametro `–hc` con el fin de filtrar los recursos que tenga un código de estado HTTP 403,404,400.
- El parametro `-w` con el fin de indicar la ruta de la lista de posibles nombres de archivos y directorios,  que utilizara la herramienta.
- El parametro `-u` con el fin de indicar la ruta donde hara el ataque de fuerza bruta de directorios.

![](/assets/images/LazyAdmin/image021.png)

![](/assets/images/LazyAdmin/image023.png)

Llegamos a obtener nuevos recursos escondidos en el directorio raiz de la aplicacion CMS SweetRice.

Ahora accederemos al recurso `js` a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image025.png)

Nos damos cuenta que el recurso `js` viene a ser un directory list que contiene archivos con extension .js. Por lo tanto, su codigo esta escrito en JavaScript.

Ahora accederemos al recurso `_themes` a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image027.png)

Nos damos cuenta que el `recurso _themes` viene a ser un directory list.Donde lo mas probable es que se almacenen los `themes` que vayan a utilizar los sitios o aplicaciones web que se vayan a crear mediante el CMS.Por lo tanto, el subdirectorio default vendria a cotener todos los archivos del codigo fuente del theme que viene por defecto en el CMS.

Ademas, los `themes` vienen a ser plantillas prediseñadas que determinan la apariencia visual y el diseño de un sitio o aplicacion web.

Ahora accederemos al recurso `attachment` a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image029.png)

Nos damos cuenta que el recurso viene a ser un directory list vacio.

Ahora accederemos al recurso `as` a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image031.png)

Nos damos cuenta que el recurso viene a ser un formulario o login para acceder a la plataforma del CMS SweetRice.

Ahora accederemos al recurso `inc` a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image033.png)

![](/assets/images/LazyAdmin/image035.png)

![](/assets/images/LazyAdmin/image037.png)

Nos damos cuenta que el recurso `inc` viene a ser un directory list que contiene varios archivos php, ya que el codigo fuente de la aplicacion web esta escrito en `PHP`. 

Ademas, encontramos un subidrectorio, que hace referencia a un respaldo de una base de datos `MySQL`. Donde encontramos un archivo de respaldo que se ha hecho de una base de datos `MySQL`. 

Ademas, analizando el contenido de este archivo de respaldo. Llegamos a encontrar un username llamado `manager`,que al parecer su rol es admin, y su password, que esta en forma hash como usualmente suelen estar almacenado en la base de datos.

Ahora utilizaremos la herramienta `hash-identifier` para saber que funcion hash esta utilizando este hash

![](/assets/images/LazyAdmin/image039.png)

El resultado de la herramienta nos indica que lo mas probable es que sea la funcion hash `MD5`.

Ahora utilizaremos `john the ripper` para realizar el crackeo del hash. Donde le especificamos:
- La funcion hash que utiliza el hash.
- Un archivo que contiene el hash que queremos crackear.
- Un diccionario que sera utilizado por la herramienta para el crackeo.

![](/assets/images/LazyAdmin/image041.png)

Llegamos a encontrar la credencial `Password123`.

Otra manera de encontrar la credencial seria a traves del sitio web `Crackstation` que consta de una base de datos de hashes de contraseñas. Por lo tanto, al momento de ingresar nuestro hash, el sitio web lo va comparar con los hashes de su base de datos,y si coinciden con algun hash nos va mostrar la contraseña y la funcion o algoritmo hash que utilizaba el hash.

![](/assets/images/LazyAdmin/image1.png)

Ahora probaremos estas credenciales en el formulario que encontramos anteriormente.

![](/assets/images/LazyAdmin/image043.png)

![](/assets/images/LazyAdmin/image045.png)

Llegamos a acceder a la plataforma del CMS SweetRice. Ademas, observamos que tiene una seccion llamada `Plugin list`, que nos permite subir `plugins` en formato zip.(Los plugins viene a ser piezas de software que nos permiten agregar funcionalidades adicionales o personalizar la apariencia o el comportamiento del sitio o aplicacion web que se vaya a crear utilizando el CMS).

El archivo zip, que subiremos como plugin, va contener un script php, que cuando se ejecute nos generara una conexion reverse shell hacia nuestra maquina, 

![](/assets/images/LazyAdmin/image047.png)

![](/assets/images/LazyAdmin/image049.png)

![](/assets/images/LazyAdmin/image051.png)

Ahora debemos encontrar la ruta de donde se esten almacenando los plugins o archivos zip que hemos subido. Para ello debemos guiarnos del nombre que toma el recurso `_themes`, que viene a ser el directory list que almacena los themes,y que se van a utilizar en los sitios o aplicaciones web que se vayan a crear con el CMS. Por lo tanto, lo mas probable es que el recurso  que almacene los plugins.Tambien se llame `_plugin` o algo parecido a ello.

![](/assets/images/LazyAdmin/image053.png)

De esta manera verificamos nuestra hipotesis.

Ahora habilitamos el puerto 1500 en nuestra máquina para que esté en escucha y esperando la conexión entrante. Para ello utilizamos la herramienta netcat. Además utilizamos el comando rlwrap para que la shell generada en nuestra máquina presenta funcionalidades como el autocompletado de comandos, historial de comandos, etc.

![](/assets/images/LazyAdmin/image055.png)

Ahora, accederemos al recurso `_plugin`. Donde daremos clic al script php para que se ejecute la conexión reverse shell hacia nuestra máquina.

![](/assets/images/LazyAdmin/image057.png)

De esta manera llegamos obtener acceso a la máquina LazyAdminFinal.

Otra manera de obtener acceso a la maquina objetivo seria aprovechando de que tenemos la opcion de poder subir `themes` en formato `zip`. 

El archivo zip, que subiremos como theme, va contener el mismo script php que utilizamos anteriormente, que cuando se ejecute nos generara una conexion reverse shell hacia nuestra maquina. 

![](/assets/images/LazyAdmin/image059.png)

Ahora accederemos al recurso `_themes` a través de mi navegador web para verificar que se haya subido nuestro archivo `zip`.

![](/assets/images/LazyAdmin/image061.png)

Ahora habilitamos el puerto 1500 en nuestra máquina para que esté en escucha y esperando la conexión entrante.

![](/assets/images/LazyAdmin/image055.png)

Luego, accederemos al recurso `_themes`. Donde daremos clic al script php contenido en el archivo zip, que subi anteriormente, para que se ejecute la conexión reverse shell hacia nuestra máquina.

![](/assets/images/LazyAdmin/image057.png)

Otra manera de obtener acceso a la maquina objetivo seria alterando unos de los archivos php del codigo fuente del theme `default`, que viene por defecto en el CMS.

![](/assets/images/LazyAdmin/image067.png)

En este caso modificaremos el contenido del archivo `comment_form.php` con el script php, que utilizamos anteriormente.

Ahora habilitamos el puerto 1500 en nuestra máquina para que esté en escucha y esperando la conexión entrante.

![](/assets/images/LazyAdmin/image055.png)

Ahora, accederemos subdirectorio `default`, que contiene los archivos de codigo fuente del theme default. Donde daremos clic al `comment_form.php` para que se ejecute la conexión reverse shell hacia nuestra máquina.

![](/assets/images/LazyAdmin/image065.png)

![](/assets/images/LazyAdmin/image057.png)

![](/assets/images/LazyAdmin/image071.png)

## Fase Escalada de privilegios

Nos damos cuenta que hemos obtenido acceso siendo el usuario `www-data`, y observando el contenido del archivo `/etc/passwd` nos percatamos que al usuario www-data se le he configurado una shell nologin que viene a ser una shell no interactiva para limitar al usuario en la ejecución de ciertos comandos que requieren una tty. Por lo tanto, debemos buscar una vector de escalada de privilegios. 

Ahora utilizamos el siguiente comando para observar que programas puedo ejecutar con el comando `sudo`, y con los privielgios del usuario root.

![](/assets/images/LazyAdmin/image073.png)

Observamos que podemos ejecutar el archivo Perl `backup.pl` con el archivo binario `Perl` teniendo los privilegios del usuario root. 

Ahora, observaremos el contenido del archivo `backup.pl`.

![](/assets/images/LazyAdmin/image075.png)

Vemos que el archivo perl contiene un script perl con los siguientes comandos:
- #!/usr/bin/perl: Esta línea es conocida como shebang y se utiliza para indicar al sistema operativo que debe usar el intérprete de Perl para ejecutar este script.
- system("sh", "/etc/copy.sh");: La función system() se utiliza para ejecutar comandos en el sistema operativo. En este caso, se está ejecutando el comando sh /etc/copy.sh. Esto significa que se está ejecutando el script copy.sh utilizando el intérprete de shell (sh).

Ahora observaremos el contenido del script `copy.sh`. 

![](/assets/images/LazyAdmin/image077.png)

![](/assets/images/LazyAdmin/image079.png)

Donde observamos un comando que crea un FIFO, que es un mecanismo de comunicación entre procesos que permite enviar datos de manera unidireccional, para establecer la conexión de shell inversa y utiliza el comando nc para redirigir los datos de salida de la shell hacia un host remoto.

Ahora, aprovechando que el script `copy.sh` tiene el permiso de ser editable para los usuarios que pertenecen a un grupo diferente al grupo primario del usuario root, modificaremos la dirrecion ip de ese host remoto a la nuestra para que se genera la conexion reverse shell con nuestra maquina.

![](/assets/images/LazyAdmin/image081.png)

Nos damos cuenta que no podemos modificar el contenido del script con el editor de texto `nano` ya que requiere de una shell interactiva. Por lo tanto, utilizaremos el comando `echo` con el fin de redirigir la cadena de caracteres, que digitemos, hacia el script.

![](/assets/images/LazyAdmin/image083.png)

Ahora habilitamos el puerto 1700 en nuestra máquina para que esté en escucha y esperando la conexión entrante

![](/assets/images/LazyAdmin/image085.png)

Ahora ejecutaremos el archivo `backup.pl` con el archivo binario perl y con el archivo binario sudo para ejecutarlo con los privilegios del usuario root.

![](/assets/images/LazyAdmin/image087.png)

![](/assets/images/LazyAdmin/image089.png)

De esta manera llegamos a ser el usuario root o superusuario.

Luego, con el comando `cat` podremos observar el contenido de ambas banderas. 

![](/assets/images/LazyAdmin/image091.png)

