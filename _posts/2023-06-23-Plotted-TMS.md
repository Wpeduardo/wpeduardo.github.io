---
layout: single
title: Plotted-TMS - TryHackMe
excerpt: "En este caso debemos encontrar dos banderas en la maquina Plotted-TMS v3. Donde a partir del escaneo de puertos de Nmap pude saber que hay dos servicios HTTP utilizando los puertos 80 y 445. Luego, utilizo wfuzz para encontrar recursos ocultos en los directorios raiz de los servidores web. Donde  encontré recursos ocultos en el directorio raíz del servidor web que utiliza el puerto 445. Llegando a encontrar el recurso management que viene a ser formulario de una aplicación web, el cual es vulnerable a SQLi blind permitiéndonos autenticarnos sin proveer credenciales. Luego, en el portal de administración de la aplicación web subimos un archivo PHP en lugar de la imagen del avatar, que nos permitio ejecutar una conexión reverse shell hacia nuestra máquina. Luego, realizamos una escalada de privilegios para ser el usuario plot_admin aprovechando una tarea cron. Luego, realizamos otra escalada de privilegios para ser el usuario root utilizando el archivo binario `doas`, que tiene el permiso `SUID` habilitado."
date: 2023-06-23
classes: wide
header:
  teaser: /assets/images/Plotted-TMS/image003.png
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
  - Python
  - SSH
  - PHP
  - SQLi
  - doas
  - C
  - Base64
  - wfuzz
---

![](/assets/images/Plotted-TMS/image001.png)

## Summary

En este caso debemos encontrar dos banderas ubicadas en la maquina Plotted-TMS v3 Donde a partir del escaneo de puertos de Nmap pude saber que hay dos servicios HTTP utilizando los puertos 80 y 445. Luego, utilizo wfuzz para encontrar recursos ocultos en el directorio raiz del servidor web que utiliza el puerto 80. Donde solo encontré recomendaciones de seguir con la fase de enumeración. Luego, utilizo wfuzz para encontrar recursos ocultos en el directorio raíz del servidor web que utiliza el puerto 445. Llegando a encontrar el recurso management que viene a ser formulario de una aplicación web, el cual es vulnerable a inyección SQL blind permitiéndonos autenticarnos sin proveer credenciales. Luego, en el portal de administración de la aplicación web subimos un archivo PHP en lugar de la imagen del avatar, que nos permitir ejecutar una conexión reverse shell hacia nuestra máquina. Llegando obtener acceso al sistema objetivo. Luego, realizamos una escalada de privilegios para ser el usuario plot_admin aprovechando una tarea cron. Luego, realizamos otra escalada de privilegios para ser el usuario root utilizando el archivo binario `doas`, que tiene el permiso `SUID` habilitado.

## Fase Reconocimiento

Para resolver este ejercicio empezaremos utilizando el comando `openvpn` con el fin de establecer una conexion VPN con la red virtual donde esta la maquina `Plotted-TMS v3`. Para ello utilizaremos el archivo de configuracion, que lo podemos descargar en la plataforma de `Tryhackme`, que puede incluir informacion como la direcciin del servidor VPN, los certificados y claves de seguridad, la configuracion de encriptacion, etc.

![](/assets/images/Plotted-TMS/image005.png)

Luego de que se establece la conexion VPN se crea una interfaz virtual de red en nuestra maquina. Donde se enruta todo el trafico de red a traves de esa interfaz. Ademas, la plataforma de `Tryhackme` nos muestra la direccion ip de la maquina `Plotted-TMS-v3`.

![](/assets/images/Plotted-TMS/image007.png)

![](/assets/images/Plotted-TMS/image009.png)

## Fase Escaneo y Enumeracion

Luego pasaremos a la fase de Escaneo y Enumeracion con el fin de poder escanear los puertos del nodo. Ademas, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la maquina `Plotted-TMS-v3`. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parametros:
- El parametro -sC o -script="default" para utilizar todos los scripts de la categoria default con el fin de realizar un escaneo y deteccion de los puertos de manera avanzada.
- El parametro -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parametro -n para evitar la resolucion DNS, y el parametro -max-rate para indicarle el numero maximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el trafico generado por la herramienta.
- El parametro -p- para realizar un escaneo de los 65535 puertos del nodo y el parametro -open con el fin de que nos muestre informacion solo de los puertos abiertos.

![](/assets/images/Plotted-TMS/image011.png)

Los resultados que obtuvimos del escaneo vienen a ser:
* Hay un servicio HTTP que se esta levantado en el puerto 80, y el programa servidor HTTP que se esta corriendo. Ademas, la version del programa servidor HTTP.
* Hay un servicio HTTP que se esta levantado en el puerto 445, y el programa servidor HTTP que se esta corriendo. Ademas, la version del programa servidor HTTP.
* Hay un servicio SSH, que esta levantado en el puerto 22, y el programa servidor SSH que se esta corriendo. Ademas, la version del programa servidor SSH.

Ahora utilizaremos la herramienta `wfuzz`, que viene a ser una forma automatizada de realizar fuerza bruta de directorios en el directorio raiz del servidor web, que esta utilizando el puerto 80, utilizando una lista de posibles nombres de directorios y archivos.
* El parametro -hc con el fin de filtrar los recursos que tenga un codigo de estado HTTP 403,404,400.
* El parametro -w con el fin de indicar la ruta de la lista de posibles nombres de archivos y directorios, que utilizara la herramienta.
* El parametro -u con el fin de indicar la ruta donde hara el ataque de fuerza bruta de directorios.

![](/assets/images/Plotted-TMS/image013.png)

Llegamos a obtener tres recursos nuevos llamados `admin`, `shadow`, `passwd`.
 
Ahora accederemos al recurso `admin` a traves de mi navegador web para observar su contenido

![](/assets/images/Plotted-TMS/image015.png)

![](/assets/images/Plotted-TMS/image017.png)

Nos damos cuenta de que el recurso `admin` viene a ser un directory list que contiene archivo, cuyo nombre hace referencia a una clave privada SSH, pero su contenido viene a ser un mensaje que parece estar codificado en `base64`.

Ahora decodificaremos el mensaje con el comando `base64` y su parametro -d para la decodificacion.

![](/assets/images/Plotted-TMS/image019.png)

Observamos que el mensaje se trata de una recomendacion de que sigamos enumerando mas recursos escondidos.

Ahora accederemos a los recursos `shadow` y `passwd` a traves de mi navegador web para observar su contenido

![](/assets/images/Plotted-TMS/image021.png)

![](/assets/images/Plotted-TMS/image023.png)

Observamos que ambos recursos tienen el mismo contenido, que viene a ser un mensaje codificado en base64.

Ahora decodificaremos el mensaje con el comando base64 y su parametro -d para la decodificacion.

![](/assets/images/Plotted-TMS/image025.png)

Observamos que el mensaje se trata de una recomendacion de que sigamos enumerando mas recursos escondidos.

Ahora utilizaremos la herramienta wfuzz para realizar fuerza bruta de directorios en el directorio raiz del servidor web, que esta utilizando el puerto 445.

![](/assets/images/Plotted-TMS/image027.png)

Llegamos a obtener un nuevo recurso llamado `management`. 

Ahora accederemos al recurso `management` a traves de mi navegador web para observar su contenido

![](/assets/images/Plotted-TMS/image029.png)

![](/assets/images/Plotted-TMS/image031.png)

Observamos que se trata de la pagina web de presentacion de una aplicacion web, y presenta un boton llamado `Login`, que nos redirige hacia un formulario. Debido a que no hemos llegado a encontrar ningunas credenciales validas y tampoco tenemos pistas de algun username ni password es mejor evitar realizar un cracking de contraseñas online mediante fuerza bruta ya que nos tomara mucho tiempo encontrar unas credenciales validas dependiendo de la robustez de las contraseñas de los usuarios. 

## Fase Hacking o Gannar Acceso o Explotacion
En esta ocasion intentaremos realizar una inyeccion SQL o `SQLi`, pero primero debemos visualizar la estructura de lal consultal SQL, que realiza la aplicacion web hacia su servidor de base de datos, para permitirnos el acceso.  Para ello utilizaremos `Burp Suite`. Donde utilizaremos su servidor Proxy para recuperar la solicitud HTTP POST, que se realiza desde nuestro navegador web al momento de rellenar el formulario y solicitar la autenticacion, luego enviaremos la solicitud hacia la ventana Repeater. Para poder modificar los valores de los parametros de la solicitud y observar las diferentes respuestas HTTP enviadas por el servidor web. 

![](/assets/images/Plotted-TMS/image033.png)

![](/assets/images/Plotted-TMS/image035.png)

![](/assets/images/Plotted-TMS/image037.png)

Observamos que el valor del parametro `password` se le aplica una funcion hash md5, que evita que el valor del parametro se concatene con la consulta SQL, pero el parametro `username` no tiene ninguna medida de seguridad que evite que los valores que le pasemos no se concatenen con la consulta SQL. Por lo tanto, la aplicacion web presenta la vulnerabilidad `SQLi blind` que nos permite realizar `Autenticacion Bypass` al momento de ingresar 'or 1=1;# como valor al parametro `username` con el fin de que la aplicacion web nos permita autenticarnos sin proveer ninguna credencial valida. 

![](/assets/images/Plotted-TMS/image039.png)

![](/assets/images/Plotted-TMS/image041.png)

Observamos que logramos acceder al portal de administracion de la aplicacion web siendo el usuario admin.

Ahora, debemos buscar una manera de obtener acceso al sistema objetivo. Para ello iremos al perfil del usuario admin. 

![](/assets/images/Plotted-TMS/image043.png)

Donde podemos observar que nos permite subir archivos para modificar la foto de nuestro Avatar. Pondremos a prueba si la aplicacion web presenta la vulnerabilidad `File Upload`, subiendo un archivo llamado reverse.php que contiene un codigo `PHP` que nos generara una conexion reverse Shell a traves del puerto 1500 en nuestra maquina. 

![](/assets/images/Plotted-TMS/image045.png)

Observamos que si nos permitio subir el archivo php. Por lo tanto, la aplicacion web si presenta la vulnerabilidad File Upload. 

Ahora daremos clic al boton Update para que se ejecute el codigo PHP de nuestro archivo, pero antes habilitamos el puerto 1500 en nuestra maquina para que esta en escucha y esperando la conexion entrante. Para ello utilizamos la herramienta `netcat`. Ademas, utilizamos el comando rlwrap para que la Shell generada en nuestra maquina presente funcionalidades como el autocompletado de comandos, historial de comandos, etc.

![](/assets/images/Plotted-TMS/image047.png)

Ahora si damos clic al boton Update. 

![](/assets/images/Plotted-TMS/image049.png)

![](/assets/images/Plotted-TMS/image051.png)

De esta manera obtenemos acceso al sistema objetivo, pero nos damos cuenta de que somos el usuario `www-data`, y observando el contenido del archivo /etc/passwd nos percatamos que al usuario www-data se le he configurado una shell nologin que viene a ser una shell no interactiva para limitar al usuario en la ejecucion de ciertos comandos que requieren una tty. Por lo tanto, debemos buscar una vector de escalada de privilegios con el fin de ser un usuario con privilegios mas elevados.

## Fase Escala de Privilegios
Ahora buscaremos archivos con permisos `SUID` con el fin de escalar privilegios, y llegar a ser el superusuario o root. Para ello ejecutamos el comando find, indicando que busque desde el directorio /, archivos con permisos SUID.

![](/assets/images/Plotted-TMS/image053.png)

![](/assets/images/Plotted-TMS/image055.png)

En esta ocasion utilizaremos el archivo binario `doas` para la escalada de privilegios. Ademas, debemos saber que a traves de este archivo binario podemos ejecutar ciertos comandos o programas con privilegios elevados. Para poder saber que comandos o programas se pueden ejecutar con el archivo binario doas debemos acceder a su archivo de configuracion llamado `doas.conf`. 

![](/assets/images/Plotted-TMS/image057.png)

Podemos observar que el contenido del archivo `doas.conf` nos indica que el usuario `plot_admin` puede ejecutar comandos del programa `openssl` con los privilegios del usuario root sin tener que ingresar sus credenciales.Por lo tanto, debemos buscar un vector de escalada de privilegios que nos permite ser el usuario `plot_admin`.

Ahora observaremos el contenido del archivo `/etc/crontab` que viene a ser le archivo crontab del sistema, que va a contener tareas cron que se ejecutaran en todo el sistema.

![](/assets/images/Plotted-TMS/image059.png)

![](/assets/images/Plotted-TMS/image061.png)

![](/assets/images/Plotted-TMS/image063.png)

Observamos que el usuario `plot_admin` ha establecido que el script `backup.sh` se ejecute de manera periodica cada minuto de cada hora de cada dia de cada semana de cada mes. Ademas, podemos observar que los permisos de control de acceso sobre el directorio scripts, que contiene el script `backup.sh`, nos indican que el usuario www-data es el propietario. Por lo tanto, podemos modificar, eliminar, crear archivos en el directorio, pero podemos observar que los permisos de control de acceso sobre el script no nos permiten modificar su contenido. Por lo tanto, lo eliminaremos y crearemos otro script con el mismo nombre que nos genera una conexion reverse shell haciendo nuestra maquina. Ademas, le asignaremos el permiso de ejecucion a otros usuarios para que el usuario `plot_admin` pueda ejecutarlo.

![](/assets/images/Plotted-TMS/image065.png)

Ahora habilitamos el puerto 1700 en nuestra maquina para que esta en escucha y esperando la conexion entrante, y esperemos un minuto para que el sistema siendo el usuario `plot_admin` ejecute nuestro script modificado.

![](/assets/images/Plotted-TMS/image067.png)

![](/assets/images/Plotted-TMS/image069.png)

De esta obtenemos acceso al sistema objetivo siendo el usuario `plot_admin`. 

Ahora, aprovecharemos que podemos correr comandos con el programa `OpenSSL` y con los privilegios del usuario root a traves del archivo binario `doas`. Para ello utilizaremos el parametro -engine para cargar una biblioteca crypto , que va contener un codigo que nos generara un Shell siendo el usuario root cuando se ejecute. Ademas, utilizaremos el parametro req para que el programa OpenSSL pueda procesarlo.

![](/assets/images/Plotted-TMS/image071.png)

Ahora para crear la biblioteca compartida cypto debemos primero crear un archivo con codigo `C` que nos generara la shell. Luego, se generara un archivo objeto utilizando el compilador GCC. Ademas, utilizamos el parametro -o para renombrar a nuestro archivo con su extension de objeto.

![](/assets/images/Plotted-TMS/image073.png)

![](/assets/images/Plotted-TMS/image075.png)

Ahora, utilizaremos el compilador GCC y el parametro -shared para generar un archivo de biblioteca compartida `exploit.so`. Ademas, utilizaremos el parametro `-lcrypto` para indicar que se debe enlazar con la biblioteca "crypto" de OpenSSL para utilizar funciones criptograficas. Ademas, utilizamos el parametro -o para renombrar a nuestro archivo con su extension de biblioteca compartida.

![](/assets/images/Plotted-TMS/image077.png)

Ahora, utilizaremos el modulo `HTTPServer` de `Python` que nos permite ejecutar un servidor web en la red local con el fin servir nuestros archivos y subdirectorios del directorio actual donde estamos. De esta manera desde el sistema objetivo podemos descargar con el comando `wget` la biblioteca compartida que hemos creado.

![](/assets/images/Plotted-TMS/image079.png)

![](/assets/images/Plotted-TMS/image081.png)

Ahora si ejecutaremos el comando que describimos anteriormente para ser el usuario root.

![](/assets/images/Plotted-TMS/image083.png)

Ahora buscaremos las dos banderas contenidas en los archivos `user.txt` y `root.txt`. Para localizar los archivos utilizaremos el comando find para buscar desde el directorio /, archivos con sus nombres.

![](/assets/images/Plotted-TMS/image085.png)




 



































