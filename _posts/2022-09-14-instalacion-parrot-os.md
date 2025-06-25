---
layout: single
title: Instalación ParrotOS
author: Juan Antonio Soto
date: 2022-09-12
classes: wide
excerpt: "Vamos a tratar de explicar y documentar la instalación de Parrot OS es un entorno virtual VMWare. Intentaré detallar los pasos y poner enlaces de todos las fuentes que he ido consultando para la instalación del entorno. No soy ningún experto, ni tengo experiencia previa en la gestion, ni instalación de sistemas operativos linux, simplemente hago esto para intentar mejorar, aprender conceptos..."
category:
  - Instalación linux
  - Guía
tags:
  - linux
  - GitHubPages
  - VMWare
  - ParrosOS
  - Ciberseguridad
---
# Instalación Parrot OS

## Descripción

[Parrot OS](https://es.wikipedia.org/wiki/Parrot_Security_OS) es una distribución de Linux enfocada principalmente a la seguridad informática. 

Dispone de varias versiones, en mi caso, voy a instalar la *Security edition* ya que cuenta con bastantes herramientras preinstaladas relacionadas con la ciberseguridad.

[Link de descarga](https://www.parrotsec.org/download/)

He visto que tienen una documentación muy detallada sobre la instalación.

https://www.parrotsec.org/docs/installation.html

## Características de la máquina virtual

En el post de instalación de ArchLinux detallo más este proceso, haciendo una breve descripción de algunas de las opciones

* Instalación típica
* Instalador de disco archivo imagen ISO - *Asignandole la ISO de Parrot OS previamente descargada*
* 60GB de tamaño de disco - **Store virtual disk as a single file**
* 8GB de RAM - *Con 4Gb sería suficiente según he leido*
* 2 Procesadores
* Adaptador de red Bridged con el check marcado

## Arrancando la máquina e instalando el SO

Esta distribución realiza muchos de los pasos automáticamente, a diferencia del post de ArchLinux donde se detalla la configuración de todo a mano mediante comandos.

Arrancamos la máquina, y nos mostrará el siguiente menú, seleccionamos la primera opción.

![install menu](../assets/images/inslatar-parrot-os/install-menu.png)

Se nos ejecutará el SO, pero sin instalar, aunque ya podemos hacernos una idea del contenido del SO que además viene con las VMTools preinstaladas

![fist SO](../assets/images/inslatar-parrot-os/first-so-vmtools.png)

El comando ejecutado en la terminal muestra la versión de VMTools que tenemos "instalada", para comprobarlo usamos el siguiente comando.

````console
foo@bar:~$ vmware-toolbox-cmd -v
````

En el escritorio debemos hacer doble clic en **Install Parrot** para que comience el asistente de instalación mediante GUI.

En mi caso, toda la instalación por defecto hasta el menú de particiones.
Elegimos la opción de borrar el disco, esta vez, sin swap.  

![disk partition](../assets/images/inslatar-parrot-os/disk-partition.png)  

He checkeado la opción de cifrar el sistema, añadiendo una contraseña. **No es la contraseña de ningún ususario, si no la del crifrado del sistema, nos la pedirá al arrancar, antes de llegar a poder logear con cualquier usuario**.

Si no ciframos el disco, cabe la posibilidad de que se pueda modificar la contraseña de un usuario existente, arrancando la consola en el arranque, ya que esta consola se incializa con el usuario root, cifrando el disco podríamos evitar eso.

La siguiente opción de instalación será la de configurar el usuario del sistema.
Y nada más que configurar en el proceso de instalación. Ya simplemente lo dejamos que termine de instalar.

![installprogress](../assets/images/inslatar-parrot-os/installing-parrot.png)

Y poco más que configurar en el proceso de instalación

## Bibliografía, enlaces de interés
[Link de descarga Parrot OS](https://www.parrotsec.org/download/)
[Parrot instalation doc](https://www.parrotsec.org/docs/installation.html)
[Video instalación s4vitar](https://hack4u.io/cursos/personalizacion-de-entorno-en-linux/13843/)