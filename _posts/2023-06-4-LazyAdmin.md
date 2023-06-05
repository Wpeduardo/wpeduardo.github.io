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

Ahora probaremos estas credenciales en el formulario que encontramos anteriormente.

![](/assets/images/LazyAdmin/image043.png)

![](/assets/images/LazyAdmin/image045.png)

Llegamos a acceder a la plataforma del CMS SweetRice. Ademas, observamos que tiene una seccion llamada Plugin list, que nos permite subir `plugins` en formato zip.(Los plugins viene a ser piezas de software que nos permiten agregar funcionalidades adicionales o personalizar la apariencia o el comportamiento del sitio o aplicacion web que se vaya a crear utilizando el CMS).

El archivo zip, que subiremos como plugin, va contener un script php, que cuando se ejecute nos generara una conexion reverse shell hacia nuestra maquina, 

![](/assets/images/LazyAdmin/image047.png)

![](/assets/images/LazyAdmin/image049.png)

![](/assets/images/LazyAdmin/image051.png)

Ahora debemos encontrar la ruta de donde se esten almacenando los plugins o archivos zip que hemos subido. Para ello debemos guiarnos del nombre que toma el directoy list _themes, que almacena los themes, que se vayan a utilizar en los sitios o aplicaciones web que se vayan a crear con el CMS. Por lo tanto, lo mas probable es que el directory list que almacene los plugins.Tambien se llame _plugin.

![](/assets/images/LazyAdmin/image053.png)

De esta manera verificamos nuestra hipotesis.

Ahora habilitamos el puerto 1500 en nuestra máquina para que esté en escucha y esperando la conexión entrante. Para ello utilizamos la herramienta netcat. Además utilizamos el comando rlwrap para que la shell generada en nuestra máquina presenta funcionalidades como el autocompletado de comandos, historial de comandos, etc.

![](/assets/images/LazyAdmin/image055.png)

Ahora, accederemos al recurso _plugin. Donde daremos clic al script php para que se ejecute la conexión reverse shell hacia nuestra máquina.

![](/assets/images/LazyAdmin/image057.png)

De esta manera llegamos obtener acceso a la máquina LazyAdminFinal.

Otra manera de obtener acceso a la maquina objetivo seria aprovechando de que tenemos la opcion de poder subir themes en formato zip. 

El archivo zip, que subiremos como theme, va contener el mismo script php que utilizamos anteriormente, que cuando se ejecute nos generara una conexion reverse shell hacia nuestra maquina. 

![](/assets/images/LazyAdmin/image059.png)

Ahora accederemos al recurso _themes a través de mi navegador web para verificar que se haya subido nuestro archivo zip.

![](/assets/images/LazyAdmin/image061.png)

Ahora habilitamos el puerto 1500 en nuestra máquina para que esté en escucha y esperando la conexión entrante.

![](/assets/images/LazyAdmin/image063.png)

Ahora, accederemos al recurso _themes. Donde daremos clic al script php para que se ejecute la conexión reverse shell hacia nuestra máquina.

![](/assets/images/LazyAdmin/image065.png)

Otra manera de obtener acceso a la maquina objetivo seria alterando unos de los archivos php del codigo fuente del theme default, que viene por defecto en el CMS.

![](/assets/images/LazyAdmin/image067.png)

En este caso modificaremos el contenido del archivo comment_form.php con el script php, que utilizamos anteriormente.

Ahora habilitamos el puerto 1500 en nuestra máquina para que esté en escucha y esperando la conexión entrante.

![](/assets/images/LazyAdmin/image069.png)

Ahora, accederemos subdirectorio default, que contiene los archivos de codigo fuente del theme. Donde daremos clic al comment_form.php para que se ejecute la conexión reverse shell hacia nuestra máquina.

![](/assets/images/LazyAdmin/image071.png)



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


