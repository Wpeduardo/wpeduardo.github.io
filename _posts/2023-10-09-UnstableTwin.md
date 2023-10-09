---
layout: single
title: Unstable Twin - TryHackMe
excerpt: "Debemos encontrar dos banderas. Donde a partir de un escaneo de puertos pude saber que hay un servicio Web y SSH. Luego, realicé fuerza bruta de directorios en el directorio raíz del servidor web, y encontré un endpoint de una API utilizada para la autenticación y otra para recuperar imágenes almacenadas en el servidor web. Luego, con el comando curl realice un SQLi a través de un parámetro vulnerable utilizada por el endpoint de una API para la autenticación, logrando extraer un hash de un usuario de los registros de la base de datos. Luego, cracke el hash con John The Ripper, logrando acceder al sistema mediante el servicio SSH, logrando encontrar la primera bandera en el sistema de archivos del usuario mary_ann, y utilizando steghide logre extraer unos strings de las imágenes, que también se podían obtener con el endpoint de una API creada para recuperar imágenes. Luego, concatene los strings y lo decodifico para obtener la ultima bandera."
date: 2023-10-09	
classes: wide
header:
  teaser: /assets/images/Twin/image002.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - Curl
  - SQLite
  - SQLi
  - HTTP
  - Gobuster
  - Python
  - John The Ripper
---

![](/assets/images/Twin/image001.png)

## SUMMARY

Debemos encontrar dos banderas. Donde a partir de un escaneo de puertos pude saber que hay un servicio Web y SSH. Luego, realicé fuerza bruta de directorios en el directorio raíz del servidor web, y encontré un endpoint de una API utilizada para la autenticación y otra para recuperar imágenes almacenadas en el servidor web. Luego, con el comando curl realice un SQLi a través de un parámetro vulnerable utilizada por el endpoint de una API para la autenticación, logrando extraer un hash de un usuario de los registros de la base de datos. Luego, cracke el hash con John The Ripper, logrando acceder al sistema mediante el servicio SSH, logrando encontrar la primera bandera en el sistema de archivos del usuario mary_ann, y utilizando steghide logre extraer unos strings de las imágenes, que también se podían obtener con el endpoint de una API creada para recuperar imágenes. Luego, concatene los strings y lo decodifico para obtener la ultima bandera.

## FASE RECONOCMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener las banderas. Para ello empezaremos utilizando el comando openvpn con el fin de establecer una conexión VPN con la red virtual dónde está la máquina Unstable Twin. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/Twin/image003.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/Twin/image004.png)

Además, la plataforma de Tryhackme nos muestra la dirección ip de la máquina Unstable Twin

![](/assets/images/Twin/image005.png)

## FASE ESCANEO Y ENUMERACION

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina Madeye Castle. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:

- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El parámetros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -Pn para evitar que Nmap, que realice ningún tipo de sondeo o ping hacia el sistema destino con el fin de determinar si el host está vivo o activo, y realice el escaneo de puertos directamente.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/Twin/image006.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Un programa servidor web Ngnix levantado en el puerto 80. Además, gracias al script http-tittle podemos observar que la pagina web que aloja no presenta la etiqueta title.
- Un programa servidor SSH levantado en el puerto 22.

Ahora accederemos a la página web, que no tiene la etiqueta title, alojada en el directorio raíz del servidor web a través de mi navegador web.

![](/assets/images/Twin/image007.png)

Podemos observar que es una página web en blanco sin ningún elemento HTML.

Ahora realizaremos fuerza bruta de directorios con Gobuster sobre los directorio raíz del servidor web.

![](/assets/images/Twin/image008.png)

Podemos observar que la herramienta llego a encontrar dos recursos llamados info y get_image. Donde uno ellos presentan el código de estado HTTP 500 en la respuesta HTTP, esto se debe a que el servidor web no ha podida procesar correctamente la solicitud HTTP GET realizada por la herramienta al acceder al directorio. Además, es probable que debemos especificar algún parámetro GET en la URL para que el servidor web pueda interpretar correctamente nuestra solicitud.

![](/assets/images/Twin/image009.png)

Ahora accederemos solo al recurso info a través de mi navegador web.

![](/assets/images/Twin/image010.png)

Podemos observar que la respuesta HTTP, que obtenemos de nuestra solicitud HTTP GET hacia el recurso info, contiene datos en formato JSON. Donde nos indican que se ha implementado un endpoint API que va a ser utilizado como login para autenticar y consta de dos parámetros: username and password

Ahora si accedemos al recurso api a través del navegador web, nos toparemos con una página web vacía que existe ya que no recibimos un código de estado HTTP 404.

![](/assets/images/Twin/image011.png)

A partir de esto realizaremos fuerza bruta de directorios con Gobuster dentro del directorio api.

![](/assets/images/Twin/image012.png)

Podemos observar que la herramienta llego a encontrar un recurso llamado login y al momento de que la herramienta quiso acceder al recurso login, recibe un código de estado HTTP 405 en la respuesta HTTP del servidor web, esto nos indica que no está utilizando el método de solicitud correcto para acceder al recurso. Esto tiene sentido ya que se trata de endpoint para la autenticación por lo que el único método que debería permitir es POST con los parámetros username y password como se indico en el mensaje anterior.

Ahora utilizaremos la herramienta curl para realizar una solicitud POST HTTP a través del endpoint API hacia el servidor web y con los parametros username y password.

![](/assets/images/Twin/image013.png)

Podemos observar que la respuesta HTTP de nuestra solicitud POST viene a ser un mensaje en formato JSON.

## FASE GANAR ACCESO O HACKING O EXPLOTACION

Ahora testearemos si el parámetro username es vulnerable a SQLi para ello utilizaremos la siguiente consulta SQL maliciosa básica para omitir la autenticación.

![](/assets/images/Twin/image014.png)

Podemos observar que si logramos obtener datos en formato JSON que parecen ser arreglos de dos columnas donde uno ellos parecen ser el identificador o id y otro el username. Por lo tanto, es probable que la tabla consultada por el API conste de dos columnas.

Ahora utilizaremos la siguiente consulta SQL maliciosa para comprobar nuestra hipótesis.

![](/assets/images/Twin/image015.png)

Podemos observar que nuestra hipótesis estaba en lo correcto.

Ahora enumeraremos las bases de datos gestionadas por el DBMS instalada en el sistema destino con la siguiente consulta SQL maliciosa, suponiendo que el DBMS es MySQL.

![](/assets/images/Twin/image016.png)

Podemos observar que nos topamos con un código de estado HTTP 500 en nuestra respuesta HTTP. Esto quiere decir que el servidor web no ha logrado interpretar nuestra solicitud HTTP ya que nuestra consulta SQL no es la correcta. Es probable que el DBMS no sea MySQL y sea SQLite debido a que mucho más ligero y fácil de integrar a las aplicaciones web.

Ahora utilizaremos otra consulta SQL maliciosa para saber la versión del DBMS SQlite, suponiendo que sea un SQLite.

![](/assets/images/Twin/image017.png)

Podemos observar que logramos obtener la versión del DBMS SQLite.

Ahora que sabemos el tipo de DBMS, utilizaremos la siguiente consulta SQL maliciosa para enumerar las tablas de la base de datos consultada por el API para la autenticación.

![](/assets/images/Twin/image018.png)

Podemos observar que la base de datos consta de dos tablas: users y notes.

Ahora enumeraremos las columnas de ambas tablas.

- users
![](/assets/images/Twin/image019.png)

- notes
![](/assets/images/Twin/image020.png)

Ahora volcaremos los registros de las columnas de ambas tablas.

- users
![](/assets/images/Twin/image021.png)

- notes
![](/assets/images/Twin/image022.png)

Podemos observar que logramos encontrar usernames en los registros de la tabla users que corresponden a los nombres de los personajes de la película Twins. Además, en los registros de la tabla notes nos topamos con un hash de una contraseña cuyo propietario lo mas probable es que sea de mary_ann ya que, según la trama de la película, ella busca recuperar a sus gemelos.

Ahora crackearemos el hash con John The Ripper, pero antes identificaremos el tipo de algoritmo hash que utiliza.

![](/assets/images/Twin/image023.png)

![](/assets/images/Twin/image024.png)

Podemos observar que la herramienta logro crackear el hash.

Ahora utilizaremos estas credenciales para iniicar sesion en el sistema destino mediante el servicio SSH.

![](/assets/images/Twin/image025.png)

![](/assets/images/Twin/image026.png)

Podemos observar que logramos acceder al sistema destino. Además, logramos obtener la primera bandera y una nota que hace referencia a obtener unas imágenes de su familia que vienen a ser sus hijos y sus esposas a través de un parámetro GET name. Esto nos hace pensar en el recurso get_image que nos arrojó un código de estado HTTP 500 y que tal vez si incluimos el parámetro GET name en la URL ya no aparecerá ese error y podremos recuperar las imágenes, pero para evitar estar haciendo esto podemos encontrar el directorio de la aplicación web, enumerando los procesos que se están ejecutando en el sistema destino.

![](/assets/images/Twin/image027.png)

![](/assets/images/Twin/image028.png)

![](/assets/images/Twin/image029.png)

![](/assets/images/Twin/image030.png)

Podemos observar que el API para la autenticación, la funcionalidad para recuperar las imágenes, y otros detalles han sido implementado a través de la biblioteca Flask y están siendo ejecutados bajo la cuenta del usuario root. Además, las imágenes que se logran recuperar a través del parámetro GET name están en el directorio de la aplicación web.

Ahora trasladaremos estas imágenes a mi sistema con el comando scp, y aplicaremos la herramienta steghide, que viene a ser una herramienta de esteganografía, para extraer datos ocultos dentro de las imágenes.

![](/assets/images/Twin/image031.png)

![](/assets/images/Twin/image032.png)

![](/assets/images/Twin/image033.png)

![](/assets/images/Twin/image034.png)

![](/assets/images/Twin/image035.png)

Podemos observar que logre extraer 5 archivos ocultos en las imágenes, donde la matriarca de la familia nos indica que los ordenemos en el orden de los
colores que aparecen en arcoíris. Para conocer utilizaremos el motor de búsqueda Google.

![](/assets/images/Twin/image036.png)

Ahora agruparemos los números que aparecen en cada archivo de acuerdo al color que tienen asociados.

![](/assets/images/Twin/image037.png)

![](/assets/images/Twin/image038.png)
Podemos observar que el numero que obtenemos del arreglo no viene a ser un hash, por lo tanto, debe ser alguna forma de codificación. Para ello si
probamos con las codificaciones Base en CyberChef, nos toparemos con la sorpresa que ha sido codificado con Base62, y nos encontramos con la última
bandera solicitada.
Ahora he intentado encontrar algún vector de escalada privilegios para llegar a ser el usuario root, pero no lo he logrado aun así te animo a intentarlo.
 
 

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































