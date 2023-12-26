---
layout: single
title: For Business Reasons - TryHackMe
excerpt: "Debemos encontrar 2 banderas. Primero, a partir de un escaneo de puertos pude saber que un servidor web que alojaba un sitio web creado con el CMS WordPress. Luego, utilizando WPScan logre obtener una credencial válida para acceder al panel de gestión del CMS. Luego, logre obtener acceso a un contenedor Docker del sistema destino mediante tres formas (modificando el código fuente de un archivo de un Theme, Plugin o instalando y activando un Plugin malicioso). Luego, cree un script en Bash para escanear la subred de los contenedores Docker y el sistema anfitrión, logrando saber que el sistema anfitrión tenia un puerto abierto con el servicio SSH ejecutado. Luego, mediante un Reverse Port Forwarding con chisel logre acceder al sistema anfitrión mediante el servicio SSH y reutilizando las credenciales del usuario del CMS. Luego, en la escalada de privilegio vertical, encontré dos formas aprovechando que el usuario local pertenecía al grupo secundario Docker y lxd."
date: 2023-12-25	
classes: wide
header:
  teaser: /assets/images/Pivot/image002.jpg
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - WPScan
  - CMS Wordpress
  - Chisel
  - Linpeas
  - PHP
  - HTML
  - HTTP
  - SSH
  - Remote Port Forwarding
  - Docker
  - lxd
  - Bash
---

![](/assets/images/Pivot/image001.png)

## SUMMARY

Debemos encontrar 2 banderas. Primero, a partir de un escaneo de puertos pude saber que un servidor web que alojaba un sitio web creado con el CMS WordPress. Luego, utilizando WPScan logre obtener una credencial válida para acceder al panel de gestión del CMS. Luego, logre obtener acceso a un contenedor Docker del sistema destino mediante tres formas (modificando el código fuente de un archivo de un Theme, Plugin o instalando y activando un Plugin malicioso). Luego, cree un script en Bash para escanear la subred de los contenedores Docker y el sistema anfitrión, logrando saber que el sistema anfitrión tenia un puerto abierto con el servicio SSH ejecutado. Luego, mediante un Reverse Port Forwarding con chisel logre acceder al sistema anfitrión mediante el servicio SSH y reutilizando las credenciales del usuario del CMS. Luego, en la escalada de privilegio vertical, encontré dos formas aprovechando que el usuario local pertenecía al grupo secundario Docker y lxd.

## FASE RECONOCIMIENTO 

En este caso debemos acceder al sistema destino y obtener dos banderas. Primero, empezaremos utilizando el comando openvpn y el archivo de configuración OpenVPN con el fin de establecer una conexión VPN y poder acceder a los laboratorios donde están las máquinas virtuales y vulnerables.

![](/assets/images/Pivot/image003.png)

Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina Business4.

![](/assets/images/Pivot/image004.png)

## FASE ESCANEO Y ENUMERACION
Ahora pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del sistema destino. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo del sistema destino.

![](/assets/images/Pivot/image005.png)

Los resultados que obtenemos son:
- Tres puertos cerrados.
- Un programa servidor Apache que se ejecuta en el puerto 80 y que aloja un sitio web que ha sido desarollado utilizando el CMS WordPress.

Ahora utilizaremos WPScan para realizar una enumeracion de themes, plugins, usernames del sitio web. Ademas, la version del CMS WordPress instalado.

![](/assets/images/Pivot/image006.png)

Podemos observar que la herramienta logro encontrar un username valido para la autenticacion en el login del CMS WordPress. Ahora, realizaremos un ataque de fuerza bruta de contraseñas contre el login utilizando WPScan.

![](/assets/images/Pivot/image007.png)

Podemos observar que se logro encontrar un password valido para el username sysadmin. Ahora, accederemos al panel de gestion del CMS, con el fin de saber que rol tiene asignado este usuario. Ademas, es bastante conocido que la ruta del login es /wp-admin.

![](/assets/images/Pivot/image008.png)

Podemos observar que tenemos el rol de Administrator, por lo que podemos modificar el codigo fuente de algun archivo de algun Plugin o Theme instalado, o podemos cargar un Plugin con el fin de ganara cceso al sistema destino.

## FASE GANAR ACCESO O EXPLOTACION O HACKING
### Primer Metodo: Cargar Plugin
Ahora, cargare un Plugin para ello necesito subir un archivo Zip que contendra una archivo PHP que me generara una reverse shell cuando activemos el Plugin cargado.

![](/assets/images/Pivot/image009.png)

![](/assets/images/Pivot/image010.png)

![](/assets/images/Pivot/image011.png)

![](/assets/images/Pivot/image012.png)

### Segundo Metodo: Modificar codigo fuente de un Plugin

En este caso se tiene instalado dos Plugin(como se puede observar en la siguiente imagen), pero no han sido activados, por lo que aprovecharemos en agregar nuestro Payload PHP, que nos generara una reverse shell cuando activemos nuestro Plugin.

![](/assets/images/Pivot/image013.png)

![](/assets/images/Pivot/image014.png)

### Tercer Metodo: Modificar codigo fuente de un Theme

Este metodo consiste en modificar el codigo fuente de un archivo de un theme instalado y que no este siendo utilizado por el sitio web. Luego, ejecutaremos nuestro Payload, observando el archivo modificado a traves de nuestro navegador web.

![](/assets/images/Pivot/image015.png)

![](/assets/images/Pivot/image016.png)

![](/assets/images/Pivot/image017.png)

![](/assets/images/Pivot/image018.png)

De esta manera ganamos acceso al sistema destino. Ahora realizaremos una enumeracion manual en busca de vectores de esalada de privilegios en el sistema de archivos.

![](/assets/images/Pivot/image019.png)

![](/assets/images/Pivot/image020.png)

Podemos observar que nos topamos con dos pruebas que evidencian que estamos en un contenedor Docker. Primero, el numero de proceso escasos que se ejecutan en el sistema, y segundo el archivo oculto dockerenv en el directorio raiz. Ademas, este contenedor Docker ha sido ejecutado para levantar el servidor web Apache incluido con el CMS WordPress y el sitio web.

Ahora realizaremos una escalada de privilegios automatizado con Linpeas pero el problema esta en que no se tiene ejecutado archivos binarios como wget o curl para poder descargarlo en la contenedor. Para ello implementaremos una funcionalidad de Carga de Archivos y lo desplegaremos en una de las rutas del CMS Wordpress con el fin de cargar el ejecutable de Linpeas.

```
echo '<?php
if(isset($_FILES["uploaded_file"])) {
    $path = "/tmp/";
    $path = $path . basename($_FILES["uploaded_file"]["name"]);


    if(move_uploaded_file($_FILES["uploaded_file"]["tmp_name"], $path)) {
        echo "El archivo ". basename($_FILES["uploaded_file"]["name"]). " ha sido cargado en el directorio /tmp";
    } else {
        echo "Hubo un error en la carga del archivo";
    }
}
?>' > upload.php
```

```
echo '<!DOCTYPE html><html><head><title>Funcionalidad Carga Archivos</title></head><body><form enctype='multipart/form-data'
action='upload.php' method='POST'><p>Carga Tus Archivos</p><input type='file' name='uploaded_file'></input><input type='submit'
value='Upload'></input></form></body></html>' > carga.html

```
![](/assets/images/Pivot/image021.png)

![](/assets/images/Pivot/image022.png)

![](/assets/images/Pivot/image023.png)

Podemos observar que se logro cargar el ejecutable Linpeas exitosamente mediante nuestra funcionalidad de carga de archivos. Ademas, solo se logro obtener las credenciales para acceder al DBMS MySQL utilizado para almacenar infomarcion del CMS WordPress. Aunque, este DBMS solo admite conexiones proveninetes del mismo sistema ya que no logramos escanearlo mediante Nmap, y lo mas probable es que se este ejecutando en otro contenedor Docker o en el mismo sistema anfitrion.

## PIVOTING

![](/assets/images/Pivot/image024.png)

Podemos observar todas las dirreciones IP de las interfaces de red del contenedor Docker. Donde la segunda dirrecion IP viene a estar asignada la interfaz de red por donde nos comunicaremos con los demas contenedores Docker y con el sistema anfitrion.

Ahora aprovecharemos que la subred de los contenedores Docker es 172.18.0.0/16 para crear un script en Bash que nos ayude a automatizar el escaneo del puerto 22 en los otros contenedores Docker que puedieran existir en la subred. Hemos escogido el puerto 22 con el fin de realizar un pivoting hacia tal sistema. Ademas, cargaremos el binario de nc de nuestro sistema hacia el contenedor Docker mediante la funcionalidad de Carga de Archivos.

![](/assets/images/Pivot/image025.png)

![](/assets/images/Pivot/image026.png)

Logramos saber que el sistema 172.18.0.1 tiene el puerto 22 abierto, y este numero de puerto viene a ser el predeterminado del servicio SSH. Ademas, es sabido que la dirrecion IP 172.18.0.1 corresponde a la interfaz de puente de Docker que le corresponde al sistema anfitrion. Por tal motivo podemos asegurar que se ejecuta el servicio SSH en el sistema anfitrion, pero al parecer hay un dispositivo de seguridad que anda bloqueando las conexiónes entrantes no locales ya que cuando realizamos nuestro primer escaneo con Nmap al sistema destino, obtuvimos el puerto 22 como cerrado.

Para solucionar esto utilizaremos Remote Port Forwarding con chisel con el fin de pode acceder al sistema anfitrion mediante el servicio SSH como si este servicio se estuviera ejecutando en nuestro sistema, y, aprovecharemos que tenemos acceso al contenedor Docker. Primero, cargaremos el ejecutable de chisel en el contenedor Docker mediante la funcionalidad de Carga de Archivos.

![](/assets/images/Pivot/image027.png)

![](/assets/images/Pivot/image028.png)

Ahora que hemos establecido el Remote Port Forwarding, accederemos al servicio SSH del sistema destino como si se tratase de un servicio SSH que se estuviera ejecutando en mi sistema. Ademas, reutilizaremos las credenciales del usuario sysadmin utilizadas para acceder al panel de gestion del CMS WordPress.

![](/assets/images/Pivot/image029.png)

De esta manera logramos acceder al sistema anfitrion, logrando salir del contenedor Docker.

## FASE ESCALADA PRIVILEGIOS

Ahora buscaremos vectores de escalada de privilegios de manera manual en el sistema de archivos del usuario sysadmin.

![](/assets/images/Pivot/image030.png)

Podemos observar que el usuario sysadmin pertenece al grupo secundario lxd por lo puede gestionar contenedores Linux. Debido a ello crearemos un contenedor Linux con la imagen alpine en su version 3.19. Ademas, este contenedor se ejecutara con los privilegios de root, y montaremos el directorio raiz con todo sus sudbidrectorios y archivos del sistema anfitrion en el directrorio /mnt/root del contenedor. Ademas, se ejectuara una shell sh con el fin de poder navegar por el sistema de archivos del contenedor.

![](/assets/images/Pivot/image031.png)

De esta manera vamos a poder tener los privilegios de root en el contenedor, y poder navegar por el sistema de archivos sin ningun inconveniente, pero en si no se trataria de una escalada de privilegio vertical en el sistema anfitrion, pero aprovecharemos que el directorio raiz se monta en el directiorio /mnt/root para asignarle el bit SUID habilitado al archivo binario sh con el fin de poder ejecutar una shell sh con los privilegios de root en el sistema anfitrion.

![](/assets/images/Pivot/image032.png)

![](/assets/images/Pivot/image033.png)

![](/assets/images/Pivot/image034.png)

De esta manera logramos realizar la escalada de privilegio vertical en el sistema anfitrion, y logramos obtener las dos banderas.

### METODO ALTERNATIVO DE ESCALADA PRIVILEGIO VERTICAL

Otra manera de escalar privilegios seria aprovechar que tambien el usuario local pertenece al grupo secundario Docker por lo que puede gestioanr contenedores Docker. De esta manera aprovechareriamos en ejecutar un contenedor Docker con las imágenes que ya tiene descargado el sistema anfitrion, para montar el directorio raiz del sistema anfitrion en algun directorio dentro del contenedor Docker. Ademas, con el parametro it ejecutaremos una shell  dentro del contenedor para poder navegar en el sistema archivos.

![](/assets/images/Pivot/image035.png)

![](/assets/images/Pivot/image036.png)







