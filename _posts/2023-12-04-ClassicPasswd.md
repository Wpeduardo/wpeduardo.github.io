---
layout: single
title: Classic Passwd - TryHackMe
excerpt: "Debemos analizar un archivo binario ejecutable LSB y obtener el username que nos solicitan al momento de que lo ejecutamos. Además, al ser LSB, debemos saber que los bytes se almacenaran en la memoria siguiendo el formato litte-endian. Luego, utilizo la herramienta Ghidra para realizar ingeniería inversa y poder obtener el código fuente del binario en C. Observando que se utiliza la función strcmp para comparar byte a byte, el username que ingresamos con los bytes almacenados desde una dirección de memoria que le corresponde a una variable predefinida en el programa. Además, debemos saber que la comparación se terminara cuando se tope con un byte nulo. Luego, si ambos strings son iguales, la función generará el valor entero 0, que cumplirá con la condición de un if, generando la bandera. Pero debemos tener en cuenta que se han definido otras variables cuyos bytes se han almacenado en direcciones de memoria consecutivas, por lo que debemos tener en cuenta estos bytes en el username que ingresaremos."
date: 2023-12-04	
classes: wide
header:
  teaser: /assets/images/Passwd/image002.jpg
  teaser_home_page: true
  icon: /assets/images/descarga7.webp
categories:
  - TryHackMe
  - ctf
tags:
  - Linux  
  - OpenVPN
  - Ghidra
  - little-endian
  - ltrace
  - C
---

![](/assets/images/Passwd/image001.png)

## SUMMARY

Debemos analizar un archivo binario ejecutable LSB y obtener el username que nos solicitan al momento de que lo ejecutamos. Además, al ser LSB, debemos saber que los bytes se almacenaran en la memoria siguiendo el formato litte-endian. Luego, utilizo la herramienta Ghidra para realizar ingeniería inversa y poder obtener el código fuente del binario en C. Observando que se utiliza la función strcmp para comparar byte a byte, el username que ingresamos con los bytes almacenados desde una dirección de memoria que le corresponde a una variable predefinida en el programa. Además, debemos saber que la comparación se terminara cuando se tope con un byte nulo. Luego, si ambos strings son iguales, la función generará el valor entero 0, que cumplirá con la condición de un if, generando la bandera. Pero debemos tener en cuenta que se han definido otras variables cuyos bytes se han almacenado en direcciones de memoria consecutivas, por lo que debemos tener en cuenta estos bytes en el username que ingresaremos.

## DESARROLLO

En este caso debemos analizar el binario que nos proveen en el enunciado del laboratorio, y descubrir la contraseña solicitada al ejecutar el archivo binario.

![](/assets/images/Passwd/image003.png)

![](/assets/images/Passwd/image004.png)

Podemos observar que el archivo binario ejecutable es LSB, por lo tanto los bytes se almacenaran en memoria siguiendo el formato little-endian.

Ahora utilizaremos la herramienta Ghidra con el fin de observar el codigo fuente del archivo binario.

![](/assets/images/Passwd/image005.png)

Podemos observar en el codigo fuente del archivo binario que se utiliza la funcion scanf para solicitar al usuario que ingrese el username, cuyos caracteres seran almacenados en la variable local_238. Luego, se utilizara la funcion strcpy con el fin de copiar el valor de la variable local_238 en la variable local_2c8. Luego, se utilizara la funcion strcmp para comparar byte a byte, los byte que estan almacenados en la dirrecion de memoria de la variable local_246 con los bytes almacenados en la variable local_2c8, hasta encontrar un byte nulo. Luego, si ambas cadenas de bytes son iguales, se emitira el resultado entero 0, y cumplira con el condicional if, lo que me generara el mensaje Welcome. Caso contrario, se imprimir una mensaje de Authentication Error y se terminara la ejecucion del programa.

A simple vista podriamos pensar que el username requerido deberia ser igual que el valor de la variable local_246. Pero el detalle en esto, radica en el hecho de que se ha indicado una dirrecion de memoria(&local_246), como un argumento de la funcion strcmp, por lo que desde esa dirrecion de memoria se empezara a realizar la comparacion de byte a byte, y debido a que luego de la variable local_246, se han almacenados bytes de las variables local_23e y local_23a en dirreciones de memoria consecutivas. Esto va generar que no solo se comparen los bytes de la variable local local_2c8 con la variable local_238 sino que tambien tomara los bytes de las variables local_23e y local_23a hasta el byte nulo(00).

![](/assets/images/Passwd/image006.png)

Pòr esta razon, los bytes que deberia tener el username que ingresaremos son 0x6435736a36424741,0x476b6439, 0x37. Luego, utilizaremos xxd para pasarlo a formato ASCII. Ademas, debido a que los bytes se almacenan en la memoria en formato little-endian, deberiamos poner los strings en sentido inverso y recien unirlos en un solo.

![](/assets/images/Passwd/image007.png)

Ahora probaremos este string como el username requerido en la ejecucion del archivo binario.

![](/assets/images/Passwd/image008.png)

## SEGUNDO METODO

Otra manera que pudimos haber utilizado es mediante la funcion ltrace, que nos permite observar las funciones que se invocan en segundo plano y los resultados que retornan al momento de la ejecucion del programa.

![](/assets/images/Passwd/image009.png)

Podemos observar que una de las funciones que se invocan es strcmp, donde comparara el string AGB6js5d9dkG7 con el string que ingresemos como username solicitado en el programa. Por lo tanto, de esta manera podremos saber que el string correcto que deberiamos ingresar como username es AGB6js5d9dkG7 con el fin de que el condicional if se cumpla y nos imprima el mensaje Welcome con la bandera.

## TERCER METODO
Otra manera que pudimos haber obtado es dirigirnos a la funcion donde se imprimira la bandera, y observar el valor de ella directamente.

![](/assets/images/Passwd/image010.png)

Podemos observar que se hara uso de la funcion printf para mostrar la flag, y que se utilizaran marcardores de posicion %d que esperan ser reemplezados por los valores enteros de los otros dos argumentos, que viene a ser 0x638a78 y 0x2130, pero tendremos que representarlos en su formato decimal para poder conocer la composicion completa de la flag.

![](/assets/images/Passwd/image011.png)

De esta manera logramos resolver este reto con tres maneras distintas.


























