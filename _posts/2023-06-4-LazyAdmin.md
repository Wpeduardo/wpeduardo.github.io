---
layout: single
title: LazyAdmin - TryHackMe
excerpt: "En este caso debemos encontrar dos banderas ubicadas en la maquina LazyAdminFinal. Donde a partir del script http-enum de Nmap encontré el recurso content, que viene a ser la pagina de inicio predeterminada del CMS SweetRice. Luego utilizo wfuzz para encontrar recursos ocultos en el directorio raíz de la aplicación CMS. Llegando a encontrar el recurso as que viene a ser un login o formulario para entrar a la plataforma del CMS. Además, encontré el recurso inc que viene a ser un directory list que contenía un archivo de respaldo de una base de datos MySQL. Donde encontré unas credenciales que las utilice para autenticarme en el formulario. Luego aprovechándome que la aplicación CMS permite subir plugins, themes, o modificar el archivo del código fuente de un theme es que puede subir un script PHP que me genere el acceso a la maquina objetivo. Luego para la escalada de privilegios, modifique un comando de un archivo perl que podía ejecutar teniendo los privilegios del usuario root."
date: 2023-06-04
classes: wide
header:
  teaser: /assets/images/LazyAdmin/image003.png
  teaser_home_page: true
  icon: /assets/images/decarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - linux  
  - openvpn
  - nmap
  - ftp
  - http
  - wfuzz
  - cmsmadesimple
  - metasploit
  - burpsuite
  - php
  - phtml
  - polkit
  - pkexec
  - ssh
  - sudo
---

![](/assets/images/LazyAdmin/image001.png)

## Summary

En este caso debemos encontrar dos banderas ubicadas en la maquina LazyAdminFinal. Donde a partir del script http-enum de Nmap encontré el recurso content, que viene a ser la pagina de inicio predeterminada del CMS SweetRice. Luego utilizo wfuzz para encontrar recursos ocultos en el directorio raíz de la aplicación CMS. Llegando a encontrar el recurso as que viene a ser un login o formulario para entrar a la plataforma del CMS. Además, encontré el recurso inc que viene a ser un directory list que contenía un archivo de respaldo de una base de datos MySQL. Donde encontré unas credenciales que las utilice para autenticarme en el formulario. Luego aprovechándome que la aplicación CMS permite subir plugins, themes, o modificar el archivo del código fuente de un theme es que puede subir un script PHP que me genere el acceso a la maquina objetivo. Luego para la escalada de privilegios, modifique un comando de un archivo perl que podía ejecutar teniendo los privilegios del usuario root.

## Fase Reconocimiento

Para resolver esta maquina empezaremos utilizando el comando `openvpn` con el fin de establecer una conexión VPN con la red virtual dónde está la máquina EasyCTF. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/LazyAdmin/image005.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/LazyAdmin/image007.png)

Además, la plataforma de Tryhackme nos muestra la dirección de la máquina LazyAdminFinal.

![](/assets/images/LazyAdmin/image009.png)

## Fase Escaneo y Enumeracion

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina LazyAdminFinal.Para ello utilizaremos `Nmap`.Donde le pasaremos los siguientes parametros:

- El parametro -sC  o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El -sV con el fin de conocer el sistemas operativos del nodo y las versiones de los servicios levantados.
- El parametro -n para evitar la resolución DNS, y el parametro –min-rate para indicarle el número de paquetes por segundo que va utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por Nmap.
- El parametro -p- para realizar un escaneo de los 65535 puertos del nodo y el parametro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/LazyAdmin/image011.png)

![](/assets/images/LazyAdmin/image013.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Hay un servicio `HTTP` que se esta levantado en el puerto 80, y el  programa servidor HTTP que se esta corriendo.Además, la versión del programa servidor HTTP.
- Hay un servicio `SSH` que se esta levantado en el puerto 22, y el  programa servidor SSH se está corriendo.Además, la versión del programa servidor SSH.

Ahora ejecutaremos los scripts de Nmap de la categoría vuln con el fin de conocer las vulnerabilidades de los servicios que están levantados en los puertos abiertos.

![](/assets/images/LazyAdmin/image015.png)

![](/assets/images/LazyAdmin/image017.png)

Podemos notar que se ha el utilizado script http-enum.nse de Nmap, que realiza un fuerza bruta de directorios con una lista de nombres posibles de directorios y archivos con el fin de encontrar archivos y directorios escondidos, llegandose a obtener el recurso content, que está almacenado en el directorio raíz del servidor web.

Ahora accederemos al recurso content a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image019.png)

Nos damos cuenta que el recurso viene a ser la pagina de inicio predeterminada de la aplicacion web `SweetRice`,que viene a ser un sistema de gestion de contenido. y que nos permite crear y gestionar contenido en linea, que puede ser sitios o aplicaciones web.Por lo tanto, el servidor web esta alojando el CMS SweetRice.

Ahora utilizaremos la herramienta wfuzz ,que también realiza fuerza bruta de directorios utilizando un lista de posibles nombre de directorios y archivos, pero desde la ruta del recurso content para que la herramienta encuentre recursos escondidos en el directorio raiz de la aplicacion web SweetRice. Además, le pasaremos los siguientes parametros a la herramienta:

![](/assets/images/LazyAdmin/image021.png)

![](/assets/images/LazyAdmin/image023.png)

Llegamos a obtener nuevos recursos escondidos en el directorio raiz de la aplicacion CMS SweetRice.

Ahora accederemos al recurso js a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image025.png)

Nos damos cuenta que el recurso js viene a ser un directory list que contiene archivos con extension .js. Por lo tanto, su codigo esta escrito en JavaScript.

Ahora accederemos al recurso _themes a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image027.png)

Nos damos cuenta que el recurso _themes viene a ser un directory list.Donde lo mas probable es que se almacenen los `themes` que vayan a utilizar los sitios o aplicaciones web que se vayan a crear mediante el CMS.Por lo tanto, el subdirectorio default vendria a cotener todos los archivos del codigo fuente del theme que viene por defecto en el CMS.

Ademas, los themes vienen a ser plantillas prediseñadas que determinan la apariencia visual y el diseño de un sitio o aplicacion web.

Ahora accederemos al recurso attachment a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image029.png)

Nos damos cuenta que el recurso viene a ser un directory list vacio.

Ahora accederemos al recurso as a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image031.png)

Nos damos cuenta que el recurso viene a ser un formulario o login para acceder a la plataforma del CMS SweetRice.

Ahora accederemos al recurso inc a través de mi navegador web para observar su contenido.

![](/assets/images/LazyAdmin/image033.png)

![](/assets/images/LazyAdmin/image035.png)

![](/assets/images/LazyAdmin/image037.png)

Nos damos cuenta que el recurso inc viene a ser un directory list que contiene varios archivos php, ya que el codigo fuente de la aplicacion web esta escrito en `PHP`. 

Ademas, encontramos un subidrectorio, que hace referencia a un respaldo de una base de datos `MySQL`. Donde encontramos un archivo de respaldo que se ha hecho de una base de datos MySQL. 

Ademas, analizando el contenido de este archivo de respaldo. Llegamos a encontrar un username llamado manager,que al parecer su rol es admin, y su password, que esta en forma hash como usualmente suelen estar  almacenada en la base de datos.

Ahora utilizaremos la herramienta `hash-identifier` para saber que funcion hash esta utilizando este hash

![](/assets/images/LazyAdmin/image039.png)

El resultado de la herramienta nos indica que lo mas probable es que sea la funcion hash MD5.

Ahora utilizaremos `john the ripper` para realizar el crackeo del hash. Donde le especificamos:
- La funcion hash que utiliza el hash.
- Un archivo que contiene el hash que queremos crackear.
- Un diccionario que sea utilizado por la herramienta para el crackeo.

![](/assets/images/LazyAdmin/image041.png)

Llegamos a encontrar la credencial Password123.

Otra manera de encontrar la credencial seria a traves del sitio web `Crackstation` que consta de una base de datos de hashes de contraseñas. Por lo tanto, al momento de ingresar nuestro hash, el sitio web lo va comparar con los hashes de su base de datos,y si coinciden con algun hash nos va mostrar la contraseña y la funcion o algoritmo hash que utilizaba el hash.

![](/assets/images/LazyAdmin/image1.png)





Otra manera que p

Ahora buscaremos un exploit para la versión 2.2.8 de CMS Made simple en la base de datos de `Metasploit` a través del comando searchsploit.

![](/assets/images/EasyCTF/image041.png)

![](/assets/images/EasyCTF/image043.png)

Llegamos a encontrar un exploit, que viene a ser un script cuyo código fuente es python, y que se basa en SQL injection. Además, el exploit ha sido probado exitosamente en las versiones menores a 2.2.10 de CMS Made simple. 

Ahora le daremos el parametro -m a searchsploit con el fin de que nos haga una copia del exploit en nuestro directorio actual.

![](/assets/images/EasyCTF/image045.png)

Ahora utilizaremos el parametro -h en el script para observar qué parámetros se le tiene que pasar para que funcione correctamente.

![](/assets/images/EasyCTF/image047.png)

Llegamos a notar que los parámetros que debemos pasarle al script vienen a ser: 
- EL parametro -u que viene a ser el url donde está almacenada el CMS.
- EL parametro -w que viene a ser el  wordlist con el fin de que pueda crackear el hash que vaya a encontrar el script
- El parametro -c para  habilitar el crackeo del hash que encuentre.

![](/assets/images/EasyCTF/image049.png)

![](/assets/images/EasyCTF/image051.png)

![](/assets/images/EasyCTF/image053.png)

Vemos que el script llegó a encontrar la salt, que se le agregó a la contraseña antes de que se aplicara un algoritmo o función hash. 
- Además, encontró un username, que viene a ser igual a las suposiciones que habíamos hecho en pasos anteriores.
- Además, encontró el email asociado al usuario mitch. 
- Además, encontró un hash. 
- Por último, llegó a crackear el hash con su salt utilizando el wordlist rockyou.txt, que le pasamos.

De esta manera, el script llegó a encontrar la contraseña secret asociada al usuario mitch.

Otra manera de crackear el hash y el salt encontrado sería utilizando `hashcat`, pero primero debemos saber que función o algoritmo hash está utilizando el hash encontrado. Para ello utilizare hash-identifier con el fin de encontrar los posibles algoritmos hash, que está utilizando el hash encontrado.

![](/assets/images/EasyCTF/image055.png)

Llegamos a obtener una posible función hash MD5. 
Luego utilizamos el siguiente comando. Donde digitamos los siguientes parámetros:
- El archivo ejecutable de Hashcat
- EL parametro -a para especificar el tipo de ataque que se realizará durante el proceso del crackeo del hash.En nuestro caso toma el valor de 0 ya que se va realizar fuerza bruta.
- El parametro -m para especificar el tipo de algoritmo o función de hash que tiene el hash ingresado. En nuestro caso toma el valor de 20 ya que viene a ser MD5 con su salt.
- Luego digitamos el hash, que se va crackear, y su salt separados por un :.
- Luego digitamos el diccionario que se va utilizar para el crackeo del hash.En nuestro caso viene a ser el rockyou.

![](/assets/images/EasyCTF/image057.png)

![](/assets/images/EasyCTF/image059.png)

Observamos que mediante esta  manera también llegamos a obtener la contraseña secret del usuario mitch.
Ahora con estas credenciales vamos a logearnos en el formulario del CMS.

![](/assets/images/EasyCTF/image061.png)

Logramos autenticarnos exitosamente. Además, observamos que hay una sección llamada File Manager que nos permite cargar archivos al servidor web.
Teniendo en cuenta esto vamos a testear si la aplicación web CMS Made simple presenta la vulnerabilidad File Upload, es decir, si es que no realiza una correcta validación de los archivos que se suben al servidor web. 
Para ello intentaremos subir un archivo PHP cuyo código genera una conexión reverse shell hacia nuestra máquina mediante el puerto 1500.

![](/assets/images/EasyCTF/image063.png)

Nos damos cuenta que al subir el archivo PHP no aparece el archivo en el directorio uploads. La primera hipótesis que podemos hacer es que la aplicación web está filtrando los archivos PHP.
Para verificar esta hipótesis, utilizamos `BurpSuite` para capturar la solicitud POST HTML que hacemos al servidor web, para que cargue nuestro archivo PHP. Luego enviaremos la solicitud capturada a la ventana Repeater para poder observar la respuesta HTML que recibimos del servidor web.

![](/assets/images/EasyCTF/image065.png)

Observando la respuesta HTTP del servidor, podemos concluir que el error está en el filename, que viene a ser el nombre de mi archivo. Ahora intentaremos cambiar la extensión, que aparece en el nombre de archivo, por un .PHTML, que viene a ser un archivo que contiene código HTML y PHP, pero en nuestro caso va contener el mismo código PHP que tenía el anterior archivo PHP.

![](/assets/images/EasyCTF/image067.png)

Vemos que con este cambio en la extensión del archivo se nos permite cargar el archivo `PHTML`.Por lo tanto, el servidor web filtra los archivos PHP que se suben a través de la aplicación web.
Teniendo en cuenta ello, vamos a subir un archivo PHTML con el mismo contenido del archivo `PHP`, que intentamos subir inicialmente.

![](/assets/images/EasyCTF/image069.png)

En esta ocasión, la aplicación web nos dejó subir el archivo phtml, y se encuentra en el directorio Uploads. Por lo tanto, la aplicación web si presenta la vulnerabilidad File Upload ya que no valida adecuadamente los archivos que cargan los usuarios al servidor a través de la aplicación CMS.
Ahora daremos clic al archivo phtml para que se ejecute el código que tiene, pero antes habilitamos el puerto 1500 en nuestra máquina para que esté en escucha y esperando la conexión entrante. Para ello utilizamos la herramienta netcat.

![](/assets/images/EasyCTF/image071.png)

![](/assets/images/EasyCTF/image073.png)

Podemos observar que hemos ganado acceso a la máquina EasyCTF, siendo el usuario www-data.Además, el mensaje can’t access tty nos indica que al usuario www-data se le he configurado con una shell nologin, que viene a ser una shell no interactiva, cada vez que inicie sesión en el sistema. Esto lo podemos verificar accediendo al contenido del archivo /etc/passwd.

![](/assets/images/EasyCTF/image075.png)

La desventaja de tener una shell no interactiva viene a ser que no vamos a poder ejecutar varios comandos que requieren de un tty.

## Fase Escalada de privilegios 
Ahora pasaremos a la fase de escalada de privilegios.Para ello buscaremos archivos binarios con permiso `SUID`, que nos va permitir tener los privilegios del propietario, que venga a ser root, del archivo cuando lo ejecutamos.

![](/assets/images/EasyCTF/image077.png)

![](/assets/images/EasyCTF/image079.png)

Nos damos cuenta que el el archivo binario `pkexec` tiene permiso SUID. Por lo tanto, a través de este archivo binario ejecutable puede ejecutar comandos teniendo el privilegios del usuario root. 
Ahora para explotar este archivo binario vamos a utilizar un script, que utiliza el marco de autorización `Polkit`(o PolicyKit) y lo podemos encontrar en el repositorio https://github.com/berdav/CVE-2021-4034

Polkit en sí viene a ser un componente utilizado para controlar los privilegios de todo el sistema en sistemas operativos similares a Unix.Además, proporciona una forma organizada para que los procesos no privilegiados se comuniquen con los privilegiados.
Además, es posible usar Polkit para ejecutar comandos con privilegios elevados usando el comando pkexec seguido del comando que se pretende ejecutar (con permiso de root).

Ahora vamos a clonar el repositorio en mi maquina. 

![](/assets/images/EasyCTF/image081.png)

Luego vamos a levantar un servidor web en mi maquina.Ademas, lo voy a levantar en el directorio que contiene el directorio CVE-2021-4034 con el fin de poder descargar el directorio CVE-2021-4034 desde la shell nologin que tenemos en la máquina EasyCTF.

![](/assets/images/EasyCTF/image083.png)

![](/assets/images/EasyCTF/image085.png)

Ahora accedemos al directorio CVE-2021-4034. 

![](/assets/images/EasyCTF/image087.png)

Donde observamos un archivo llamado Makefile, que contiene un conjunto de instrucciones. Para ejecutar las instrucciones de este archivo utilizó el comando make.

![](/assets/images/EasyCTF/image089.png)

Luego observamos que se me genera un archivo ejecutable CVE-2021-4034, que viene a ser el script que usaremos para explotar el archivo binario pkexec.

![](/assets/images/EasyCTF/image091.png)

Luego ejecutamos el script.

![](/assets/images/EasyCTF/image093.png)

De esta manera llegamos a ser el usuario root. Ahora vamos a buscar las dos banderas que nos solicitan.

![](/assets/images/EasyCTF/image095.png)

![](/assets/images/EasyCTF/image097.png)

De esta manera llegamos a encontrar el contenido de las dos banderas.
Otra manera que pudimos haber obtenido acceso a la máquina EasyCTF es utilizando la suposición que hicimos anteriormente(Que es muy probable que el administrador del sistema utilice las mismas credenciales en todo los servicios). Para verificar esto intentaremos establecer una conexión remota con el servidor SSH que tiene la maquina EasyCTF utilizando las credenciales encontradas en la aplicación web CMS Made simple.

![](/assets/images/EasyCTF/image099.png)

![](/assets/images/EasyCTF/image101.png)

También llegamos obtener acceso a la máquina EasyCTF utilizando el servicio `SSH` que está levantado en el puerto 2222 de la máquina.
Ahora, otra forma de escalar privilegios serían aprovechando que se tiene el archivo binario sudo.Además, al usuario mitch le han configurado con una shell interactiva(o bash) para cuando inicia sesión el sistema EasyCTF.

![](/assets/images/EasyCTF/image103.png)

Ahora que ya hemos verificado que existe el archivo `sudo`. Utilizaremos la bandera -l con el fin de listar los comandos que puede ejecutar el usuario actual con los privilegios del superusuario.
![](/assets/images/EasyCTF/image105.png)

Del resultado del comando, podemos concluir que podemos utilizar el archivo binario vim con el fin de  ejecutar comando teniendo los privilegios del superusuario.
Ahora, utilizaremos el siguiente comando para que nos genere una shell sh siendo el usuario root.

![](/assets/images/EasyCTF/image107.png)

Esto viene a ser otra forma de llegar a ser el usuario root.


