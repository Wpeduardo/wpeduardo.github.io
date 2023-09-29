---
layout: single
title: Steel Mountain - TryHackMe
excerpt: "Debemos encontrar dos banderas en la maquina Stell Mountain. Donde a partir del escaneo de puertos de Nmap pude saber que hay un servicio RPC, SMB, RDP, IIS, y HFS en su versión 2.3, y levantado en el puerto 8080. Luego, accedí a la interfaz web del servidor HFS a través de mi navegador web, logrando saber que su parámetro GET Search, que es utilizado para buscar archivos compartidos por el servidor, era vulnerable a RCE o Command Injection (CVE-2014-6287) asociada a la versión 2.3 del servidor HFS. Luego, encontré un exploit, que logré analizar con el fin de crear mi propio payload a través de Burp Suite y explotar la vulnerabilidad manualmente. Luego, para la escalada de privilegios vertical utilice la vulnerabilidad Unquoted Path asociada al archivo ejecutable de un servicio, logrando ejecutar un archivo malicioso (creado con msfvenom) con los privilegios de LocalSystem."
date: 2023-09-27	
classes: wide
header:
  teaser: /assets/images/Steel/image002.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Windows  
  - OpenVPN
  - Nmap
  - Commnad Injection
  - HFS
  - Unquoted Path
  - Burp Suite
  - msfvenom
  - Powershell
  - IIS
  - WinPEAS
---

![](/assets/images/Steel/image001.png)

## SUMMARY

Debemos encontrar dos banderas en la maquina Stell Mountain. Donde a partir del escaneo de puertos de Nmap pude saber que hay un servicio RPC, SMB, RDP, IIS, y HFS en su versión 2.3, y levantado en el puerto 8080. Luego, accedí a la interfaz web del servidor HFS a través de mi navegador web, logrando saber que su parámetro GET Search, que es utilizado para buscar archivos compartidos por el servidor, era vulnerable a RCE o Command Injection (CVE-2014-6287) asociada a la versión 2.3 del servidor HFS. Luego, encontré un exploit, que logré analizar con el fin de crear mi propio payload a través de Burp Suite y explotar la vulnerabilidad manualmente. Luego, para la escalada de privilegios vertical utilice la vulnerabilidad Unquoted Path asociada al archivo ejecutable de un servicio, logrando ejecutar un archivo malicioso (creado con msfvenom) con los privilegios de LocalSystem.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener las banderas. Para ello empezaremos utilizando el comando openvpn con el fin de establecer una conexión VPN con la red virtual dónde está la máquina Steel Mountain. Para ello utilizaremos el archivo de configuración, que lo podemos descargar en la plataforma de Tryhackme, que puede incluir información como la dirección del servidor VPN, los certificados y claves de seguridad, la configuración de encriptación, etc.

![](/assets/images/Steel/image003.png)

Luego de que se establece la conexión VPN se crea una interfaz virtual de red en nuestra máquina. Donde se enruta todo el tráfico de red a través de esa interfaz.

![](/assets/images/Steel/image004.png)

Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina Steel Mountain.

![](/assets/images/Steel/image005.png)

## FASE ESCANEO Y ENUMERACION

Luego pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del nodo. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo de la máquina Steel Mountain. Para ello utilizaremos `Nmap`. Donde le pasaremos los siguientes parámetros:

- El parámetro -sC o –script=”default” para utilizar todos los scripts de la categoría default con el fin de realizar un escaneo y detección de los puertos de manera avanzada.
- El parámetros -sV con el fin de conocer las versiones de los servicios levantados en los puertos abiertos.
- El parámetro -n para evitar la resolución DNS, y el parámetro –max-rate para indicarle el número máximo de paquetes por segundo que va a utilizar Nmap para el escaneo con el fin de evitar sobrecargar la red con el tráfico generado por la herramienta.
- El parámetro -Pn para evitar que Nmap, que realice ningún tipo de sondeo o ping hacia el sistema destino con el fin de determinar si el host está vivo o activo, y realice el escaneo de puertos directamente.
- El parámetro -p- para realizar un escaneo de los 65535 puertos del nodo y el parámetro –open con el fin de que nos muestre información solo de los puertos abiertos.

![](/assets/images/Steel/image006.png)

![](/assets/images/Steel/image007.png)

![](/assets/images/Steel/image008.png)

Los resultados que obtuvimos del escaneo vienen a ser:
- Un programa servidor web Microsoft IIS levantado en el puerto 80, que va a permitir alojar páginas y aplicaciones web.
- Microsoft HTTPAPI levantado en los puertos 47001 y 5985 que va a permitir que el sistema Windows ofrezca servicios web a través de servidores web.
- Un programa servidor RDP levantado en el puerto 3389, que va a permitir conectarme al sistema Windows remotamente y acceder a su escritorio y aplicaciones.
- El servicio RPC levantado en el puerto 135 y una serie de puertos dinámicos que pueden utilizarse para la comunicación RPC en sistemas Windows.
- Un servicio SMB en el puerto 445, que va a permitir compartir recursos a través de la red local., y al parecer se utiliza NetBios Session Service, levantado en el puerto 139, para establecer y la mantener la sesión de comunicación entre los sistemas
- Un programa servidor web HFS levantado en el puerto 8080, que proveer una interfaz web donde los usuarios pueden compartir y acceder archivos a través de su navegador web.

Ahora accederemos al directorio raíz del servidor web IIS a través de mi navegador web para saber si tiene alojada alguna aplicación o sitio web.

![](/assets/images/Steel/image009.png)

![](/assets/images/Steel/image010.png)

Podemos observar una pagina web con la imagen de un usuario llamado Bill.

Ahora accederemos a la interfaz web del servidor web HFS.

![](/assets/images/Steel/image011.png)

Podemos observar que la versión del programa servidor HFS es 2.3.

Ahora buscaremos vulnerabilidades asociadas a la versión 2.3 del servidor HFS.

![](/assets/images/Steel/image012.png)

![](/assets/images/Steel/image013.png)

![](/assets/images/Steel/image014.png)

![](/assets/images/Steel/image015.png)

![](/assets/images/Steel/image016.png)

## FASE EXPLOTACION O GANAR ACCESO O HACKING

Podemos observar que la versión 2.3 del programa servidor HFS es vulnerable a RCE o Command Injection debido a que no se realiza una correcta sanitización de los valores ingresados al parámetro GET search, que es utilizado para buscar archivos compartidos en el servidor. Además, llegamos a encontrar un exploit. Donde si analizamos su payload, nos damos cuenta de las siguientes cosas:

1. Usa bytes nulls %00 para omitir el filtro
2. Usa el scripting command exec, que te permite ejecutar un archivo alojado en el servidor HFS con parámetros, con el fin de ejecutar el archivo  ejecutable de la consola Powershell con el parámetro -e con el fin de ingresar el comando codificado en base64 y el que va generar la reverse Shell  en el sistema del atacante.

Ahora que hemos entendido la forma como trabaja el exploit, realizaremos la explotación manual creando nuestro propio payload, pero primero haremos un payload de prueba con el fin de saber si trata de un Command Injection Blind o Verbose. Para ello crearemos un payload, que va a estar codificado con la codificación URL, y que va a ejecutar el comando whoami con el fin de saber si el resultado se muestra en algún elemento HTML de su interfaz web.

![](/assets/images/Steel/image017.png)

![](/assets/images/Steel/image018.png)

![](/assets/images/Steel/image019.png)

Podemos observar que no se llego a mostrar el resultado del comando, por lo tanto, viene a ser un Command Injection Blind. Para demostrar esto crearemos un payload que ejecutara un ping hacia nuestro sistema, y utilizaremos tcpdump para saber si es que están llegando los mensajes ICMP response a nuestro sistema.

![](/assets/images/Steel/image020.png)

![](/assets/images/Steel/image021.png)

![](/assets/images/Steel/image022.png)

Podemos observar los resultados de ejecución de nuestro payload.

Ahora crearemos nuestro payload de una formas legible y entendible. Para ello utilizaremos el parámetro -e de powershell para pasarle el comando, que nos generara la reverse Shell, codificación en base64. Además, nuestro payload va a ser ingresado en la URL codificado en URL con el fin de que sea interpretado correctamente por nuestro navegador web.

![](/assets/images/Steel/image023.png)

![](/assets/images/Steel/image024.png)

Ahora antes de ejecutar nuestro payload debemos habilitar el puerto 1500 en nuestro sistema para que este en modo listening y esperando la conexión entrante de nuestra reverse Shell.

![](/assets/images/Steel/image025.png)

![](/assets/images/Steel/image026.png)

Podemos observar que logramos acceder al sistema Windows destino siendo el usuario Bill. Además, logramos encontrar la primera bandera en archivo user.txt localizado en el directorio Desktop del directorio de Bill.

## FASE ESCALADA PRIVILEGIOS

Ahora debemos buscar un vector de escalada privilegios para ser un usuario con mayores privilegios. Para ello utilizaremos WinPeas con el fin de automatizar el proceso de búsqueda.

![](/assets/images/Steel/image027.png)

Podemos observar que la herramienta llego a encontrar que la ruta del archivo ejecutable del servicio Advanced SystemCare Service 9 contiene espacios y no esta entre comillas. Esto genera que sea un potencial vector de escalada privilegios ya que el sistema operativo del sistema objetivo cuando intente ejecutar el archivo ejecutable buscara primero el archivo ejecutable C:\Programa.exe, y si no lo encuentra, buscara el archivo ejecutable C:\Program Files.exe, y si no lo encuentra, buscara el archivo ejecutable C:\Program Files (x86).exe, y si no lo encuentra, buscara el archivo ejecutable C:\Program Files (x86)\IObit\Advanced.exe, y así sucesivamente hasta encontrar el archivo ejecutable correcto.

Ahora buscaremos bajo que cuenta o usuario se ejecutara el archivo ejecutable del servicio.

![](/assets/images/Steel/image028.png)

Podemos observar que bajo la cuenta o usuario LocalSystem se ejecutara el archivo ejecutable del servicio, y viene a ser la cuenta con privilegios más elevados que un Administrador.

Ahora crearemos con msfvenom un archivo ejecutable que funcione como servicio de Windows, y que va a contener nuestro payload malicioso que nos generara una reverse Shell cuando se ejecute.

![](/assets/images/Steel/image029.png)

Ahora transferiremos nuestro archivo ejecutable malicioso hacia el sistema destino con el fin moverlo a la ruta C:\Program Files (x86) \IObit\ con el nombre de Advanced.exe. Además, le asignamos el privilegio F o Full Control al grupo Everyone, es decir, a todos los usuarios con el fin de que el sistema operativo pueda ejecutarlo cuando reiniciemos el servicio Advanced SystemCare Service 9.

![](/assets/images/Steel/image030.png)

![](/assets/images/Steel/image031.png)

Ahora reiniciaremos el servicio Advanced SystemCare Service 9 con el fin de que se ejecute nuestro archivo ejecutable malicioso con los privilegios del usuario LocalSystem o NT Authority\System, y nos genere una reverse Shell, pero antes no olvidemos habilitar el puerto 4445 en nuestro sistema para que este en modo listening y esperando la conexión entrante.

![](/assets/images/Steel/image032.png)

![](/assets/images/Steel/image033.png)

![](/assets/images/Steel/image034.png)

Podemos observar que logramos acceder al sistema Windows destino siendo el usuario NT Authority \Sytem o LocalSystem.

Ahora buscaremos la segunda bandera en el directorio Desktop del usuario Administrador.
 
![](/assets/images/Steel/image035.png) 
 
 
 
 
 
 
 
 
 
 
 
 
 



































