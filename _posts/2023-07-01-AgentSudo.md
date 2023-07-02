---
layout: single
title: Agent-sudo - TryHackMe
excerpt: "Debemos encontrar dos banderas en la maquina `Agent-sudo`. Donde a partir del escaneo de puertos de Nmap pude saber que hay un servicio `HTTP`, `FTP`, y `SSH`. Ademas, con un script de `Nmap` supe que hay una pagina web almacenada en el servidor web, y utilizando `Burp Suite` y la extension `User-Agent Switcher and Manger` encontre el valor del encabezado `User-Agent` para obtener el contenido real de la pagina web. Donde encontre un username. Luego, realice cracking de contraseña contra el servicio `SSH` utilizando `Hydra`, encontrando unas credenciales para el servicio `FTP`. Luego, encontre dos imagenes. Donde llegue a extraer archivos ocultos utilizando `Binwalk` y `Steghide`. Ademas, uno de los archivos ocultos contenia las credenciales de inicio de sesion del usuario `james` en el servicio `SSH`. Luego, aprovechandonos de la vulnerabilidad `CVE-2019-14287` de `sudo`, llegamos a ser el usuario root."
date: 2023-07-01
classes: wide
header:
  teaser: /assets/images/User-Agent/image003.png
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
  - Burp Suite
  - Binwalk
  - Steghide
  - Sudo
  - John The Ripper
  - User-Agent Switcher and Manager 
---

![](/assets/images/User-Agent/image001.png)

## SUMMARY

Debemos encontrar dos banderas en la maquina `Agent-sudo`. Donde a partir del escaneo de puertos de Nmap pude saber que hay un servicio `HTTP`, `FTP`, y `SSH`. Ademas, con un script de `Nmap` supe que hay una pagina web almacenada en el servidor web, y utilizando `Burp Suite` y la extension `User-Agent Switcher and Manger` encontre el valor del encabezado `User-Agent` para obtener el contenido real de la pagina web. Donde encontre un username. Luego, realice cracking de contraseña contra el servicio `SSH` utilizando `Hydra`, encontrando unas credenciales para el servicio `FTP`. Luego, encontre dos imagenes. Donde llegue a extraer archivos ocultos utilizando `Binwalk` y `Steghide`. Ademas, uno de los archivos ocultos contenia las credenciales de inicio de sesion del usuario `james` en el servicio `SSH`. Luego, aprovechandonos de la vulnerabilidad `CVE-2019-14287` de `sudo`, llegamos a ser el usuario root.

## FASE RECONOCIMIENTO

Para resolver este ejercicio empezaremos utilizando el comando `openvpn` con el fin de establecer una conexion VPN con la red virtual donde esta la maquina `Agent-sudo`. Para ello utilizaremos el archivo de configuracion, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir informacion como la direccion del servidor VPN, los certificados y claves de seguridad, la configuracion de encriptacion, etc.

![](/assets/images/User-Agent/image005.png)

Luego de que se establece la conexion VPN se crea una interfaz virtual de red en nuestra maquina. Donde se enruta todo el trafico de red a traves de esa interfaz. 

![](/assets/images/User-Agent/image007.png)

Ademas, la plataforma de Tryhackme nos muestra la direccion ip de la maquina `Agent-sudo`.

![](/assets/images/User-Agent/image009.png)

## FASE ESCANEO Y ENUMERACION
Luego pasaremos a la fase de Escaneo y Enumeracion con el fin de poder escanear los puertos del nodo. Ademas, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la maquina `Agent-sudo`. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parametros:
- El parametro -sC o -script="default" para utilizar todos los scripts de la categoria default con el fin de realizar un escaneo y deteccion de los puertos de manera avanzada.
- El parametro -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parametro -n para evitar la resolucion DNS, y el parametro -max-rate para indicarle el numero maximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el trafico generado por la herramienta.
- El parametro -p- para realizar un escaneo de los 65535 puertos del nodo y el parametro -open con el fin de que nos muestre informacion solo de los puertos abiertos.

![](/assets/images/User-Agent/image011.png)

Los resultados que obtuvimos del escaneo vienen a ser:
* Hay un servicio `HTTP` que se esta levantado en el puerto 80, y el programa servidor HTTP que se esta corriendo. Ademas, la version del programa servidor HTTP.
* A partir del script `http-tittle.nse` de `Nmap` llegamos a obtener el valor de la etiqueta tittle de una pagina web que se aloja en el servidor web. 
* Hay un servicio `FTP` que se esta levantado en el puerto 21, y el programa servidor FTP que se esta corriendo. Ademas, la version del programa servidor FTP.
* Hay un servicio `SSH`, que esta levantado en el puerto 22, y el programa servidor SSH que se esta corriendo. Ademas, la version del programa servidor SSH.

Ahora accederemos a la pagina web a traves de mi navegador web para observar su contenido.

![](/assets/images/User-Agent/image013.png)

Observamos que nos dan una indicacion de que modifiquemos el valor del encabezado `User-Agent` de la solicitud GET HTTP, que realizamos al servidor web, para observar el contenido real de la pagina web. Ademas, observando el mensaje del `agente R` podemos notar que el codename se refiere posiblemente a las iniciales en mayuscula de los usuarios. 

Ahora crearemos una lista con las letras del abecedario en mayuscula. 

![](/assets/images/User-Agent/image015.png)

Ahora utilizaremos el servidor Proxy de `Burp Suite` para interceptar la solicitud GET realizada hacia al servidor web para obtener la pagina web, que obtuvimos anteriormente. Ademas, enviaremos nuestra solicitud hacia el modulo `Intruder` de Burp Suite con el fin de realizar un ataque `Sniper` sobre el valor del encabezado `User-Agent` utilizando la lista de letras, que creamos anteriormente.

![](/assets/images/User-Agent/image017.png)

![](/assets/images/User-Agent/image019.png)

![](/assets/images/User-Agent/image021.png)

Podemos observar que llegamos a encontrar un codigo de estado HTTP 302 cuando se realiza una solicitud GET con el encabeza User-Agent igual a C.

Ahora utilizaremos `User-Agent Switcher and Manager`, que viene a ser una extension del navegador web, que nos permite modificar el valor del encabezado User-agent de nuestras solicitudes cuando naveguemos por Internet. 

![](/assets/images/User-Agent/image023.png)

![](/assets/images/User-Agent/image025.png)

Podemos observar que cuando cambiamos el valor del encabezado `User-Agent` a C, el contenido de la pagina web cambia. Ademas, en el mensaje que realiza el `agente R`. Podemos obtener el nombre del usuario `chris`, y que la contraseña que ha configurado en algun inicio de sesion de algun servicio es debil. Por lo tanto, esto nos incentiva a realizar cracking de contraseñas online sobre algun servicio que se ejecute en el sistema objetivo.

## FASE HACKING O EXPLOTACION O GANAR ACCESO
Ahora realizare el cracking de contraseñas sobre el servicio `FTP`, utilizando la herramienta `Hydra`. Teniendo en cuenta que usaremos como username chris, y el diccionario rockyou.txt. 

![](/assets/images/User-Agent/image027.png)

Podemos observar que la herramienta llego a encontrar unas credenciales validas. 

Ahora nos autenticaremos con el servidor FTP del sistema objetivo, siendo el usuario `chris` con el fin de observar sus recursos almacenados. 

![](/assets/images/User-Agent/image029.png)

Podemos observar que se llego a encontrar un archivo llamado `To_agentJ.txt`. Donde se indica que en algunos de las imagenes `cute-alien.jpg` y `cutie.png` se ha ocultado las credenciales de inicie de sesion del `agente J`. Por lo tanto, en este caso se esta utilizando la esteganografia para ocultar datos en una imagen. Ademas, para extraer datos ocultos de las imegenes utilizaremos la herramienta `Binwalk`.

![](/assets/images/User-Agent/image031.png)

![](/assets/images/User-Agent/image033.png)

![](/assets/images/User-Agent/image035.png)

![](/assets/images/User-Agent/image037.png)

Podemos observar que se llego a encontrar varios archivos ocultos en la imagen `cuite.png`, y la herramienta los almaceno en un directorio llamado `_cutie.png.extracted`. Donde observamos un archivo llamado `To_agentR.txt` que esta vacio. Ademas, encontramos un archivo zip que contiene un archivo llamado `To_agentR.txt`, pero cuando queremos descomprimir el archivo zip nos damos cuenta de que esta cifrado.

Ahora utilizaremos `John The Ripper` para encontrar la contraseña requerida para descomprimir el archivo zip.

![](/assets/images/User-Agent/image039.png)

Podemos observar que `John The Ripper` llego a encontrar la contraseña requerida para descomprimir el archivo zip. 

Ahora descomprimimos el archivo zip para obtener el archivo `To_agentR.txt`.

![](/assets/images/User-Agent/image041.png)

![](/assets/images/User-Agent/image043.png)

Podemos observar que el archivo contiene el mensaje `QXJlYTUx` que parece estar codificado en `Base64`.

Ahora decodificaremos el mensaje utilizando el comando base64 y su parametro -d para decodificar.

![](/assets/images/User-Agent/image045.png)

Observamos que el mensaje QXJlYTUx decodificado significa `Area51`. Ademas,debido a que los archivos ocultos en la imagen `cutie.png` no contenian ninguna credencial de inicio de sesion del usuario J. Utilizaremos otra herramienta de esteganografia llamada `Steghide` con el fin de encontrar datos ocultos en la otra imagen falsa llamada `cute-alien.jpg`.

![](/assets/images/User-Agent/image047.png)

Observamos que para acceder a los datos ocultos en la imagen se requiere de una contraseña, que era `Area51`. Luego el archivo oculto en la imagen se llamaba `message.txt`, que contiene las credenciales de inicio de sesion del usuario `James`.

Ahora utilizaremos estas credenciales para acceder al sistema objetivo a traves del servicio `SSH`.

![](/assets/images/User-Agent/image049.png)

De esta manera obtenemos acceso al sistema objetivo siendo el usuario `james`.

## FASE ESCALADA DE PRIVILEGIOS
Ahora debemos buscar un vector de escalada de privilegios con el fin de ser un usuario con privilegios mas elevados. Para ello utilizaremos el siguiente comando para observar que programas puedo ejecutar con el comando `sudo`, y con los privilegios del usuario root.

![](/assets/images/User-Agent/image051.png)

Podemos observar que aparece el termino `(ALL, !root)` al lado del archivo binario `bash`. Esto significa que el usuario james puede ejecutar el archivo binario bash con los privilegios de cualquier usuario del sistema objetivo, pero no con los privilegios del usuario root.Pero podemos aprovechar de la vulnerabilidad `CVE-2019-14287` de `sudo` para ejecutar el siguiente comando con el fin de llegar a ser el usuario root. 

![](/assets/images/User-Agent/image053.png)

Podemos observar que utilizamos el parametro -u para indicar el usuario con el que ejecutaremos el archivo binario bash, pero en vez de digitar el usuario root, que esta prohibido, digitamos el identificador especial #-1, el cual sera tratado incorrectamente como el ID 0, que es el identificador del usuario root. 

De esta manera ejecutamos una Shell bash siendo el usuario root 

Ahora buscaremos las dos banderas contenidas en los archivos user_flag.txt y root.txt. Para localizar los archivos utilizaremos el comando find para buscar desde el directorio /, archivos con sus nombres.

![](/assets/images/User-Agent/image055.png)












 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 

 
 

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































