---
layout: single
title: BoilerCTF - TryHackMe
excerpt: "Debemos encontrar dos banderas en la maquina BoilCTF. Donde a partir del escaneo de puertos de Nmap pude saber que hay un servicio HTTP, FTP, SSH, y Webmin. Luego, realice fuerza bruta de directorios sobre el directorio raíz del CMS Joomla, llegando a encontrar el CMS Joomla en su versión, 3.9.12.Luego, realice fuerza bruta de directorios sobre el directorio raíz del CMS, llegando a encontrar la herramienta sar2html, que proporciona una interfaz web para presentar los resultados del comando sar. Además, esta herramienta presentaba la vulnerabilidad Command Injection, que nos permitió observar el contenido de un registro, llegando a obtener las credenciales SSH de un usuario. Luego, para la escalada privilegios horizontal, utilice unas credenciales encontradas en un script, Luego, para la escalada vertical, utilice un archivo binario ejecutable find con el permiso SUID."
date: 2023-09-01	
classes: wide
header:
  teaser: /assets/images/Boil/image002.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - Webmin
  - HTTP
  - FTP
  - SSH
  - sar2html
  - Command Injection
  - Wfuzz
  - Suid
  - CMS Joomla
  - find
---

![](/assets/images/Boil/image001.png)

## SUMMARY

Debemos encontrar dos banderas en la maquina BoilCTF. Donde a partir del escaneo de puertos de Nmap pude saber que hay un servicio HTTP, FTP, SSH, y Webmin. Luego, realice fuerza bruta de directorios sobre el directorio raíz del CMS Joomla, llegando a encontrar el CMS Joomla en su versión, 3.9.12.Luego, realice fuerza bruta de directorios sobre el directorio raíz del CMS, llegando a encontrar la herramienta sar2html, que proporciona una interfaz web para presentar los resultados del comando sar. Además, esta herramienta presentaba la vulnerabilidad Command Injection, que nos permitió observar el contenido de un registro, llegando a obtener las credenciales SSH de un usuario. Luego, para la escalada privilegios horizontal, utilice unas credenciales encontradas en un script, Luego, para la escalada vertical, utilice un archivo binario ejecutable find con el permiso SUID.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener las banderas. Para ello empezaremos utilizando el comando openvpn con el fin de establecer una conexión VPN con la red virtual dónde está la máquina BoilCTF. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/Boil/image003.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/Boil/image004.png)

Además, la plataforma de Tryhackme nos muestra la dirección ip de la máquina BoilCTF.

![](/assets/images/Boil/image005.png)

## FASE ESCANEO Y ENUMERACION

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina BoilCTF. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:

- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El parámetros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/Boil/image006.png)

Los resultados que obtuvimos del escaneo vienen a ser:

- Un servicio SSH levantado en el puerto 55007 y el programa servidor SSH que se está corriendo, y la versión del programa servidor.
- Un servicio Webmin levantado en el puerto 10000, y que utiliza un servidor web liviano llamado MiniServ con el fin de proveer una interfaz web en que el administrador del sistema destino pude acceder desde su navegador web para administrar el sistema destino. Además, la versión del servicio Webmin es 1.930.
- Un servidor FTP levando en el puerto 21, y el programa servidor FTP que se está corriendo, y la versión del programa servidor. Además, a partir del script ftp-anon de Nmap se llego a saber que admite conexiones anónimas.
- Un servicio HTTP o web levantado en el puerto 80, y el programa servidor web que esta corriendo, y la versión del programa servidor.

Ahora accederemos al servidor FTP mediante una conexión anónima con el fin de observar los recursos compartidos.

![](/assets/images/Boil/image007.png)

![](/assets/images/Boil/image008.png)

Podemos observar que llegamos a encontrar un archivo oculto en el servidor FTP, que contenía un mensaje cifrado, que utilizaba el cifrado ROT13, y si lo desciframos, nos encontramos con un mensaje troll.

Ahora accederemos a la interfaz web, servida por el servidor web del servicio Webmin, desde mi navegador web.

![](/assets/images/Boil/image009.png)

![](/assets/images/Boil/image010.png)

![](/assets/images/Boil/image011.png)

Podemos observar que nos topamos con un login o formulario HTML para acceder al portal de administración del servicio Webmin. Además, observamos que los parámetros o campos del formulario no son vulnerables a SQLi. Por lo tanto, no podemos omitir la autenticación. Además, si buscamos en Internet, vulnerabilidades relacionadas con la versión 1.930 del servicio Webmin para usuarios no autenticados, no encontraremos ninguna vulnerabilidad.

Ahora realizaremos fuerza bruta de directorios con Wfuzz sobre el directorio raíz del servidor web con el fin de encontrar recursos escondidos.

![](/assets/images/Boil/image012.png)

Podemos observar que la herramienta llego a encontrar tres recursos.

Ahora observaremos el recurso robots.txt desde mi navegador web.

![](/assets/images/Boil/image013.png)

Podemos observar varias rutas de recursos que van los motores de búsqueda no deben mostrar en sus resultados de búsqueda, y si intentamos acceder a algunas de ellas, nos encontramos con el caso de que son falsos positivos. Además, observamos un mensaje que está en código ASCII.

Ahora decodificaremos el mensaje.

![](/assets/images/Boil/image014.png)

![](/assets/images/Boil/image015.png)

![](/assets/images/Boil/image016.png)

Podemos observar que luego de convertir el código ASCII a texto, obtenemos un texto codificado en base64, luego de decodificar el texto, obtenemos un hash MD5, y luego de crackear el has MD5, obtenemos otro mensaje troll.

Ahora observaremos el recurso joomla desde mi navegador web.

![](/assets/images/Boil/image017.png)

![](/assets/images/Boil/image018.png)

Podemos observar un sitio web, que parece ser uno que viene por defecto al instalar el sistema de gestión de contenido (CMS) Joomla. Además, este sitio web consta de un formulario HTML o login, y si intentamos omitir la autenticación a través de consultas SQL maliciosas, no llegaremos a obtener nada.

Ahora buscaremos la versión del CMS Joomla instalado en el sistema destino mediante algunos recursos que vienen en su instalación. 

![](/assets/images/Boil/image019.png)

Podemos observar que la versión del CMS Joomla instalada en el sistema destino es 3.9.12. Además, si buscamos en Internet, vulnerabilidades relacionadas con la versión 1.912 del servicio Joomla para usuarios no autenticados, no encontraremos ninguna vulnerabilidad. 

Ahora, sabiendo que el CMS Joomla nos permite crear y gestionar contenido en línea, es decir, sitios o aplicaciones web. Por lo tanto, va tener un portal de administración que es accesible solo para los desarrolladores de los sitios o aplicaciones, y comúnmente esta en el recurso administrator.

![](/assets/images/Boil/image020.png)

Podemos observar que para acceder al portal de administración debemos autenticarnos a través de un login, y si intentamos probar credenciales básicas como admin:password o root:password, no obtendremos nada. Además, si intentamos omitir la autenticación a través de consultas SQL maliciosas, no llegaremos a obtener nada.

Ahora realizaremos fuerza bruta de directorios sobre el directorio raíz del CMS Joomla con el fin de encontrar recursos escondidos.

![](/assets/images/Boil/image021.png)

![](/assets/images/Boil/image022.png)

![](/assets/images/Boil/image023.png)

![](/assets/images/Boil/image024.png)

![](/assets/images/Boil/image025.png)

Podemos observar que llegamos a encontrar muchos recursos escondidos dentro del directorio raíz del CMS, pero algunos de ellos contienen mensajes troll, y otros forman parte de la instalación del CMS, aunque el recurso _test, viene a ser la herramienta sar2HTML, que convierte los resultados del comando sar en páginas HTML con el fin de que los puedes observarlos a través de su interfaz web y de un navegador web. Además, el comando sar nos ayuda a recopilar datos del rendimiento del sistema como estadísticas del uso de CPU, memoria, disco, y red en intervalos regular. 

## FASE HACKING O GANAR ACCESSO O EXPLOTACION

Con todo lo sabido sobre la herramienta sar2HTML, nos da una idea de que se puede realizar un Command Injection ya que se esta ejecutando un comando en el sistema operativo del objetivo. Por lo tanto, podemos utilizar; para ejecutar otro comando en la misma línea que se ejecute el comando sar en segundo plano.

![](/assets/images/Boil/image026.png)

![](/assets/images/Boil/image027.png)

Podemos observar que nuestra suposición llego a estar en lo cierto. Por lo tanto, la herramienta sar2html presenta la vulnerabilidad Command Injection o también conocida como RCE. Ademas, si llegamos a enumerar los subdirectorios y archivos del directorio _test, nos encontramos con un archivo llamado log.txt

Ahora observaremos su contenido a través de nuestro navegador web.

![](/assets/images/Boil/image028.png)

Podemos observar que el archivo contiene unos registros, donde se observa las credenciales SSH del usuario basterd, y obtener el username de otro usuario local, llamado pentest, que tiene los privilegios del usuario root.

Ahora accederemos con estas credenciales al sistema destino mediante el servicio SSH.

![](/assets/images/Boil/image029.png)

De esta manera, obtenemos acceso al sistema destino mediante el usuario basterd.

## FASE ESCALADA PRIVILEGIOS
Ahora debemos buscar un vector de escalada privilegios. Para ello realizaremos una enumeración manual sobre los archivos del usuario basted.

![](/assets/images/Boil/image030.png)

Podemos observar que encontramos un script en el directorio home del usuario basterd, y que contiene unas posibles credenciales del usuario local Stoner.

Ahora intentaremos autenticarnos con estas credenciales siendo el usuario Stoner.

![](/assets/images/Boil/image031.png)

De esta manera, obtenemos acceso al sistema destino mediante el usuario stoner.
Ahora debemos buscar un vector de escalada privilegios. Para ello realizaremos una enumeración manual sobre los archivos con permiso SUID.

![](/assets/images/Boil/image032.png)

Podemos observar que el archivo binario ejecutable find tiene asignado el permiso SUID.

Ahora aprovecharemos el argumento exec del comando find, para ejecutar un Shell bash con los privilegios del usuario root.

![](/assets/images/Boil/image033.png)

De esta manera realizamos la escalada privilegios vertical siendo el usuario root.
Ahora buscaremos las dos banderas en los archivos. secret y root.txt

![](/assets/images/Boil/image034.png)
 
 
 
 
 
 
 
 
 
 
 
 
 
 



































