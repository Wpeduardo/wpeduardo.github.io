---
layout: single
title: CyberHeroes - TryHackMe
excerpt: "En este caso debemos omitir el mecanismo de autenticación de un login de un sitio web para obtener la flag. Primero, realicé un escaneo de puertos con Nmap para saber el número de puerto donde se esta ejecutando el servidor web que aloja el sitio web, Luego, accedí al sitio web para encontrar la ruta del login, que constaba de un formulario HTML con dos parámetros asociados al username y password. Luego, revise el código fuente de la pagina web donde estaba el login, dándome cuenta que la lógica de este mecanismo de autenticación se basaba en un conjunto de comandos dentro de una función JavaScript, que podía ser observaba en el mismo código fuente de la pagina web, Luego, analizando el código de esta función, pude obtener la composición de los username y password correcto, logrando omitir fácilmente la autenticación sin la necesidad de realizar fuerza bruta."
date: 2023-12-05	
classes: wide
header:
  teaser: /assets/images/Cyber1/image002.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - JavaScript
  - Nmap
  - HTTP
  - HTML
  - Bypass Authenticacion
---

![](/assets/images/Cyber1/image001.png)

## SUMMARY

En este caso debemos omitir el mecanismo de autenticación de un login de un sitio web para obtener la flag. Primero, realicé un escaneo de puertos con Nmap para saber el número de puerto donde se esta ejecutando el servidor web que aloja el sitio web, Luego, accedí al sitio web para encontrar la ruta del login, que constaba de un formulario HTML con dos parámetros asociados al username y password. Luego, revise el código fuente de la pagina web donde estaba el login, dándome cuenta que la lógica de este mecanismo de autenticación se basaba en un conjunto de comandos dentro de una función JavaScript, que podía ser observaba en el mismo código fuente de la pagina web, Luego, analizando el código de esta función, pude obtener la composición de los username y password correcto, logrando omitir fácilmente la autenticación sin la necesidad de realizar fuerza bruta.

## FASE RECONOCIMIENTO ACTIVO

En este caso debemos acceder al sistema destino y omitir el mecanismo de autenticación de un login de un sitio web. Primero, empezaremos utilizando el comando openvpn y el archivo de configuración OpenVPN con el fin de establecer una conexión VPN y poder acceder a los laboratorios donde están las máquinas virtuales y vulnerables.

![](/assets/images/Cyber1/image003.png)

Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina Different CTF.

![](/assets/images/Cyber1/image004.png)

## FASE ESCANEO Y ENUMERACION

Ahora pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del sistema destino. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios, y el sistema operativo del sistema destino.

![](/assets/images/Cyber1/image005.png)

Los resultados que obtenemos son:
-  Un programa servidor SSH que se ejecuta en el puerto 22.
-  Un programa servidor HTTP que se ejecuta en el puerto 80. Ademas, podemos observar que hay un sitio web alojado en el servidor web.

Ahora accederemos al sitio web a traves de mi navegador web.

![](/assets/images/Cyber1/image006.png)

![](/assets/images/Cyber1/image007.png)

Podemos observar que en la pagina de bienvenida de este stio web consta de un mensaje bastante bonito: “Heros are not born, They work hard and become one”. Ademas, nos dicen que intentemoso omitir el mecanismo de autenticacion del login del sitio web para formar parte de ellos.

![](/assets/images/Cyber1/image008.png)

Podemos observar que el login consta de dos campos relacionados con el Username y Password. Ademas, si observamos el codigo fuente de la pagina web donde esta el login, nos damos cuenta que al momento de dar clic al boto de Login para autenticarnos, se ejecutara la funcion authenticate para verificar si las credenciales ingresadas son correctas. Pero la vulnerabilidad en este login radica en que su logica, que se basa en esta funcion JavaScript llamada authenticate, puede ser observada y analizada por cualquiera..

Ahora analizaremos esta funcion JavaScript para poder encontrar las credenciales correctas.

* Primero se obtiene los elementos HTML cuyso atributos id son uname y pass, y que vienen a ser el username y password ingresados. Luego, se almacenan en dos variables llamadas a y b.
* Luego se declara una funcion llamada ReverseString que va aceptar un argumento string. Ademas, esta funcion va convertir el string en un array donde sus elementos va ser cada carácter del string mediante el operador de propagacion. Ademas, utilizara la funcion reverse para invertir el orden de los elementos del array. Ademas, utilizar la funcion join para concatenar cada elemento del array en un string.
* Luego, se utilizara el condicional if para comparar el valor almacenado en los atributos value elementos HTML( cuyos atributos id son uname y pass, y que vienen a ser el username y password ingresados) con el valor h3ck3rBoi y el valor que se retorna de esta ejecucion RevereString("54321@terceSrepuS"), que vendria a ser SuperSecret@12345.
* Si ambos valores comparados son iguales, se realizar una solicitud GET con el fin de observar el contenido de un archivo de texto .txt, que va contener la flag. Caso contrario, aparecera una ventana emergente con el mensaje Incorrect Password, try again.. you got this hacker !.

Ahora que conocemos la logica del mecanismo de autenticacion del login, facilmente podemos deducir que el username y password correcto son h3ck3rBoi: SuperSecret@12345. Comprobaremos esto en el login, y para observar la flag.

![](/assets/images/Cyber1/image009.png)

 
 
 
 
 
 
 
 
 



































