---
layout: single
title: hackerNote - TryHackMe
excerpt: "Debemos encontrar dos banderas. Donde a partir de un escaneo de puertos pude saber que hay un servidor web que aloja una aplicación web, que está compuesta de un login, y de un registro. Además, lograremos realizar una enumeración de usernames mediante un script en Python, aprovechando que hay una diferencia entre el tiempo de respuesta cuando ingresamos un username valido y cuando ingresamos un username incorrecto. Luego, realizamos fuerza bruta de contraseñas mediante un script en Python y un wordlist, que fue creado teniendo en cuenta la pista configurada por el propietario de la cuenta cuando se registró. Luego, de autenticarnos exitosamente, llegamos a obtener las credenciales para acceder al sistema destino mediante el servicio SSH. Luego, hubo dos formas para realizar la escalada privilegio vertical, la primera fue explotando la vulnerabilidad CVE-2021-3156, la segunda fue explotando la vulnerabilidad CVE-2019-18634, ambas vulnerabilidad relacionadas con sudo."
date: 2023-11-13	
classes: wide
header:
  teaser: /assets/images/hackerNote/image002.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - Enumeration Usernmaes
  - Force Brute Passwords
  - HTTP
  - HTML
  - Sudo
  - Python
  - SSH
  - Baron Samedit
  - CVE-2019-18634
---

![](/assets/images/hackerNote/image001.png)

## SUMMARY

Debemos encontrar dos banderas. Donde a partir de un escaneo de puertos pude saber que hay un servidor web que aloja una aplicación web, que está compuesta de un login, y de un registro. Además, lograremos realizar una enumeración de usernames mediante un script en Python, aprovechando que hay una diferencia entre el tiempo de respuesta cuando ingresamos un username valido y cuando ingresamos un username incorrecto. Luego, realizamos fuerza bruta de contraseñas mediante un script en Python y un wordlist, que fue creado teniendo en cuenta la pista configurada por el propietario de la cuenta cuando se registró. Luego, de autenticarnos exitosamente, llegamos a obtener las credenciales para acceder al sistema destino mediante el servicio SSH. Luego, hubo dos formas para realizar la escalada privilegio vertical, la primera fue explotando la vulnerabilidad CVE-2021-3156, la segunda fue explotando la vulnerabilidad CVE-2019-18634, ambas vulnerabilidad relacionadas con sudo.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener las banderas. Para ello empezaremos utilizando el comando openvpn con el fin de establecer una conexión VPN con la red virtual dónde está la máquina hackerNote. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/hackerNote/image003.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/hackerNote/image004.png)

Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina hackerNote.

![](/assets/images/hackerNote/image005.png)

## FASE ESCANEO Y ENUMERACION

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina hackerNote. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:

- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El parámetros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -Pn para evitar que Nmap, que realice ningún tipo de sondeo o ping hacia el sistema destino con el fin de determinar si el host está vivo o activo, y realice el escaneo de puertos directamente.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/hackerNote/image006.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Un programa servidor Web en el puerto 80 y 8080. Además, observamos que el servidor esta implementado en el lenguaje de programación Go o también conocido como Goolang y esta utilizando el paquete net/http para gestionar las solicitudes HTTP.
- Un programa servidor SSH en el puerto 22.

Ahora accederemos al servidor web a traves de mi navegador web para observar si está sirviendo una aplicación web o algún sitio web.

![](/assets/images/hackerNote/image007.png)

![](/assets/images/hackerNote/image008.png)

Podemos observar que hay una aplicación web alojada en el servidor web. Además, una de sus páginas web consta de un registro y un login. Luego, cree una cuenta en el registro con el username eduardo y con la pista igual a Dina. Además, esta pista va a ser mostrada cuando ingresemos el username eduardo y demos clic al botón I forgot my contraseña del login.

![](/assets/images/hackerNote/image009.png)

Otra cosa de que nos damos cuenta es que el mensaje de error obtenido de la repuesta HTML es bastante genérico ya que no varia si ingresamos un username valido y una contraseña incorrecta o si ingresamos ambos incorrectos.

![](/assets/images/hackerNote/image010.png)

![](/assets/images/hackerNote/image011.png)

Otra cosa que nos damos cuenta son los tiempos de respuesta ya que, si ingresamos un username valido y una contraseña incorrecta, obtendremos un tiempo de respuesta mayor que cuando ingresamos un username invalido y una contraseña incorrecta.

## FASE GANAR ACCESSO O EXPLOTACION O HACKING

Ahora teniendo en cuenta los tiempos de respuesta entre un username valido y uno incorrecto. Podemos elaborar un script que nos permita realizar una enumeración de usernames de manera eficiente. Además, utilizando las herramientas integradas en el navegador web, podemos observar que la solicitud POST para la autenticación va dirigida hacia el endpoint de una API, y los datos del cuerpo de la solicitud POST HTTP van en formato JSON.

![](/assets/images/hackerNote/image012.png)

![](/assets/images/hackerNote/image013.png)

![](/assets/images/hackerNote/image014.png)

![](/assets/images/hackerNote/image015.png)

Podemos observar que mi script en Python primero calcula el tiempo de respuesta de una solicitud HTTP con el username eduardo que es válido. Luego, imprimirá los usernames con un tiempo de respuesta cercano al tiempo de respuesta del username eduardo. De esta manera logramos obtener dos usernames válidos, donde uno es del mi usuario creado.

Ahora obtendremos la pista o hint del username james a traves del login.

![](/assets/images/hackerNote/image016.png)

Podemos observar que la pista nos dice que la contraseña esta compuesta por el nombre de un color y de un numero.

Ahora teniendo en cuenta estas especificaciones, crearemos un wordlist para realizar fuerza bruta de contraseñas contra el login con el fin de obtener las credenciales del usuario james. Para la creación del wordlist, utilizaremos el siguiente script en Python.

![](/assets/images/hackerNote/image017.png)

![](/assets/images/hackerNote/image018.png)

De esta manera creamos nuestro worlidst listo para ser usado en nuestro ataque de fuerza bruta de contraseña contra el login.

Ahora crearemos otro script para realizar la fuerza bruta de contraseñas.

![](/assets/images/hackerNote/image019.png)

![](/assets/images/hackerNote/image020.png)

![](/assets/images/hackerNote/image021.png)

De esta manera logramos encontrar unas credenciales validas, y logramos autenticamos en el login de la aplicación web. Luego, observamos una nota que contiene un password para iniciar sesión en el sistema destino mediante el servicio SSH.

![](/assets/images/hackerNote/image022.png)

De esta manera ganamos acceso al sistema destino.

## FASE ESCALADA PRIVILEGIOS

Ahora debemos buscar vectores de escalada privilegios. Primero enumeraremos la versión instalada del programa sudo en el sistema destino.

![](/assets/images/hackerNote/image023.png)

Debido a que la versión del programa sudo esta desactualizada, utilizaremos el siguiente crash PoC para saber si la versión del sudo es vulnerable a la vulnerabilidad Baron Samedit (https://github.com/lockedbyte/CVE-Exploits/tree/master/CVE-2021-3156).

![](/assets/images/hackerNote/image024.png)

![](/assets/images/hackerNote/image025.png)

De esta manera comprobamos que la versión del sudo si es vulnerable a Baron Samedit.

Ahora utilizaremos el siguiente exploit para automatizar la explotación. Además, el exploit lo puedes encontrar en: https://github.com/blasty/CVE-2021-3156.

![](/assets/images/hackerNote/image026.png)

De esta manera realizamos una escalada privilegio vertical llegando a ser el usuario root.

Otra manera de realizar la escalada de privilegio vertical es cuando ingresamos la contraseña del usuario james para enumerar los comandos o programas que podemos ejecutar con sudo, y nos aparece asteriscos. Por lo tanto, la configuración pwdfeedback está habilitada en el archivo /etc/sudoers. Además, debido a que la versión de sudo es inferior a 1.8.26. Entonces se cumpla las condiciones de la vulnerabilidad CVE-2019-18634.

![](/assets/images/hackerNote/image027.png)

Ahora utilizaremos el siguiente exploit para automatizar la explotación (https://github.com/saleemrashid/sudo-cve-2019-18634).

![](/assets/images/hackerNote/image028.png)

Esta viene a ser otra manera de realizar una escalada de privilegio vertical.

Ahora buscaremos las dos banderas en el sistema de archivos del sistema destino.

![](/assets/images/hackerNote/image029.png) 

 
 

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































