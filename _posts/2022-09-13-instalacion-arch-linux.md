---
layout: single
title: Instalación ArchLinux
author: Juan Antonio Soto
date: 2022-09-12
classes: wide
excerpt: "En éste articulo explicaré los pasos que he seguido para instalar la distro ArcLinux en una máquina virtual. Intentaré detallar los pasos realizados, lo máximo posible, ya que me estoy introduciendo en Linux y toda esta documentación me sirve para interiorizar y comprender todos los pasos realizados."
category:
  - Personalización linux
  - Guía
tags:
  - linux
  - GitHubPages
  - VMWare
  - ArchLinux
---
# Instalación ArchLinux

## Descripción

En éste articulo explicaré los pasos que he seguido para instalar la distro ArcLinux en una máquina virtual. Intentaré detallar los pasos realizados, lo máximo posible, ya que me estoy introduciendo en Linux y toda esta documentación me sirve para interiorizar y comprender todos los pasos realizados.  

Debido a que todavía no tengo muchos conocimientos en relación a Linux, me estoy apoyando en varias guías/videos/documentación para realizarlo.  

En mi máquina tengo instalado windows, mi intención es instalar ArchLinux en una máquina virtual, evitando así el arranque dual boot, ya que no me interesa.

Temas que vamos a tratar en el post.

* Creación máquina virtual
  * Configuración del SO
  * Configurar teclado
* Creación de particiones
  * Asignación de tamaño
  * Configurar partición SWAP
  * Dar formator a las particiones
  * Montar particiones
  * Fstab y genfstab
* Configurar acceso al sistema
  * Configurar usuarios
  * Configurar grupos sudo su
* Instalar y configurar bootloader
* Configurar el hostname
* Comprobar la configuración
* Configurar acceso a internet
* Instalación de paquetes
* Instalación black arch
  * Instalación interfaz gráfica
* Arreglar proporciones de pantalla
* Bibliografía, enlaces de interés

## Creación máquina virtual 

Usando VMWare crearé una nueva máquina virtual con la configuración típica.  

[Descargamos la ISO](https://archlinux.org/download/) y seleccionamos la opción instalar disc image file, añadiendo la ruta de descarga.  

![installer](/assets/images/instalar-archlinux/installer-disc-option.png)  

Seleccionamos el gues operating system.  

![os](/assets/images/instalar-archlinux/guest-operating-system.png)  

El siguiente paso será asignarle la máxima capacidad al disco virtual, en mi caso he añadido 60GB y he seleccionado la opción de almacenar el disco duro como un solo fichero.
Se recomienda usar la segunda opción, de almacenarlo en varios ficheros cuando se esta utilizando un sistema de ficheros con límite de capacidad.  

![diskspace](/assets/images/instalar-archlinux/disk_space.png)

En el siguiente paso le daremos al botón de **customize Hardware** yo, siguiendo las recomendaciones de otros usuarios que tienen más conocimientos, he asignado 60GB de disco, 2 CPU, 4GB de memoria RAM y he pueso el adaptador de red como Briged, para usar el adaptador físico de mi máquina y poder tener salida a internet.  

![configuration](/assets/images/instalar-archlinux/hardware-configuration.png)

Hacemos clic en **Finish**.

## Configuración del SO
Ahora vamos a configurar el SO, a diferencia de otras distribuciones, en ArchLinux tenemos que realizar al configuración a mano.  
Arrancamos la máquina y seleccionamos la primera opción de arranque **Arch Linux install nodiun (x86_64, BIOS)**, después nos abrirá una consola.  

## Configurar teclado
Por defecto viene con la distribución de teclados EU para teclados ANSI, si queres cambiar a una [distribución de teclado española](https://man.archlinux.org/man/loadkeys.1) usad el siguiente comando para modificar las keys en la **sesión actual**.
```console
foo@bar:~$ loadkeys es
foo
```  
Para hacerlo fijo, tendremos que modificar el fichero **/etc/vconsole.conf**
```console
/etc/vconsole.conf
------------------

KEYMAP=es
```  
Este es el fichero que **systemd** lee al principio para cargar las keymaps.

## Creación de particiones

### Asignación de tamaño

Para ello usaremos [cfdisk](https://es.wikipedia.org/wiki/Cfdisk) y seleccionaremos el label **dos** (ver [MBR](https://en.wikipedia.org/wiki/Master_boot_record)).  
Ahora vamos a crear las particiones, crearemos 3 particiones, una de 512M PRIMARIA, otra de 54.5G PRIMARIA y otra de 5G PRIMARIA para la memoria SWAP.

![partitions](/assets/images/instalar-archlinux/all-partitions.png)  


Nota: los tamaños los he escogido basandome en guías. **Los tamaños se asignan con M y G** Ej. 512M o 5G

### Configurar partición SWAP

En la partición SWAP seleccionamos **Type**   

![swap](/assets/images/instalar-archlinux/partition-type.png)

Seleccionamos la opción 82 Linux swap / solarias y cuando salgamos, seleccionamos la opción write para confirmar la configuración.  

Así deberían quedar   

![partitionresult](/assets/images/instalar-archlinux/final-disk.png)

### Dar formator a las particiones

En este punto vamos a darle el formato de archivos correspondiente a cada partición, usando el comando [mkfs](https://elmanualdelmundo.blogspot.com/2019/10/como-usar-el-comando-mkfs-en-linux.html)

```console
# -F para asignarle el Fat Size, 12,16 o 32
foo@bar:~$ mkfs.vfat -F 32 /dev/sda1

# a la partición 2 le ponemos el fs ext4
foo@bar:~$ mkfs.ext4 /dev/sda2

# con mkswap le asignamos el area swap a la partición 3
foo@bar:~$ mkswap /dev/sda3

# por último swapon para habilitar la partición como swapping
foo@bar:~$ swapon
``` 

### Montar particiones

Ahora vamos a montar las particiones, es decir, asignarle un directorio en el sistema a cada una de las particiones.  
Lo montaremos en el directorio /mnt. [ver estructura de directorios linux](https://geekland.eu/estructura-de-directorios-en-linux/)
```console
# montamos la partición 2 en el directorio /mnt
foo@bar:~$ mount /dev/sda2 /mnt
foo@bar:~$ mkdir /mnt/boot
foo@bar:~$ mount /dev/sda1 /mnt/boot

# hacemos un pacstrap para realizar la instalación del sistema bajo el punto de montaje
foo@bar:~$ pacstrap /mnt linux linux-firmware networkmanager grub wpa_supplicant base base-devel
``` 
Nota: Los paquetes que he instalado, los he seleccionado en base a la guía de s4vitar, la cual estoy siguiendo para en un futuro realizar prácticas con este entorno.

### Fstab y genfstab

Una vez tenemos montadas las particiones, ahora debemos configurar el fstab, que es el fichero que contiene la información de configuración que va a ser leida por los comando mount y unmount.

```console
# Generamos el fichero de configuración fstab usando la utilidad genfstab, y la guardamos en el directorio correspondiente
foo@bar:~$ genfstab -U /mnt > /mnt/etc/fstab
``` 

## Configurar acceso al sistema 

Nuestro sistema va a residir en al partición sda2, a la cual le hemos asignado 54G anteriormente, esta partición la hemos montado en el directorio /mnt
```console
# usaremos el comando arch-chroot pasando la ruta /mnt, si no, por defecto nos pillará la ruta /bin/bash
foo@bar:~$ arch-chroot /mnt
```
Una vez ejecutado el comando arch-chroot nos abrirá una nueva consola, esta consola nos asegura que varias funcionalidades y sistemas de archivos esten disponibles.

### **Configurar usuarios**

Ahora configuraremos una contraseña para el usuario root. 


![rootpassword](/assets/images/instalar-archlinux/arch-chroot.png).  

También aprovecharemos para crear un usuario 

```console
# -m para que cree el directorio personal de usuario 
foo@bar:~$ useradd -m juan

# también podemos asignarle una contraseña
foo@bar:~$ passwd juan
```
### **Configurar grupos sudo su**]

Vamos a añadir al usuario creado anteriormente al grupo correspondiente para que pueda tener acceso a las utilidades de **sudo** y **su**, en archlinux para eso, tenemos que añadirlo al grupo **wheel**

```console
# -a para añadir al grupo, debe usarse con G para especificar el nombre del grupo
foo@bar:~$ usermod -aG wheel juan
```

Ahora que juan pertenece al grupo wheel, vamos a asegurar la instalación de algunos paquetes para poder terminar el proceso de asignación de permisos sudo. 

```console
# aseguramos la instalación del paquete sudo con el administrador de paquetes de archlinux pacman. 
foo@bar:~$ pacman -S sudo

# instalamos los editores de ficheros
foo@bar:~$ pacman -S vim nano
```
Y ahora vamos a editar el fichero /etc/sudoers  
Tenemos que descomentar la línea marcado en verde, para que todos los usuarios pertenecientes al grupo wheel puedan ejecutar cualquier comando. ¡OJO! no descomentar la línea marcada en rojo, o NO pedirá el password para hacer un sudo su.  

![sudoers](/assets/images/instalar-archlinux/edit-sudoers.png)

## Instalar y configurar bootloader

```console
# instalamos el grub en el dispositivo
foo@bar:~$ grub-install /dev/sda

# el siguiente comando nos genera el fichero de configuración grub.cfg
foo@bar:~$ grub-mkconfig -o /boot/grub/grub.cfg
```

## Configurar el hostname

En este punto no existirá el fichero hostname, el cual registra el hostname en el sistema operativo, para ello ejecutamos el siguiente comando.
 
```console
# lo he creado con un eco y redirigiendo la salida, la manera más rapida a mi entender
foo@bar:~$ echo juan > /etc/hostname
```

## Comprobar la configuración

Ahora debemos reiniciar, para ello hacemos un **exit** para salir de la consola de arch-chroot y ejecutamos un **reboot now**.  
Nos aparecerá el login, lo introducimos.  

![login](/assets/images/instalar-archlinux/login.png)  

Y comprobamos que hemos accedido con el usuario juan.  

![juanacces](/assets/images/instalar-archlinux/juan-access.png)

## Configurar acceso a internet

Vamos a configurar el acceso a internet de nuestra máquina, para ello haremos uso del paquete que instalamaos al hacer el **pacstrap, NetworkManager**, haremos uso de **systemctl** que se usa para controlar y administrar los servicios systemd.

```console
# necesitaremos permisos en este punto
foo@bar:~$ sudo su

# arrancamos el servicio
foo@bar:~$ systemctl start NetworkManager.service

# habilitamos el serivicio para que arranca cuando se inicie el sistema a partir de ahora
foo@bar:~$ systemctl enable NetworkManager

# arrancamos y habilitamos el servicio wpa_ supplicant
foo@bar:~$ systemctl enable --now wpa_supplicant

```
Nota: con **systemctl enable --now** aparte de habilitar el servicio, tambien lo arrancamos.

Ya deberiamos tener acceso a internet, podemos comprobarlo haciendo un ping.

## Instalación de paquetes

Vamos a instalar distintos paquetes, usando el comando **pacman**, gestor de paquetes de Arch Linux.

```console
# como root instalaremos los paquetes
foo@bar:~$ pacman -S git
```
Ahora, vamos a clonarnos un repositorio para poder usar la utilidad **AUR**, el cual está manejado por la comunidad, se usa también en otras distribuciones.

Para clonarlo.

```console
# OJO atended sobre que ruta ejecutais el git clone, pues ahí es donde se descargará el repositorio.
foo@bar:~$ git clone https://aur.archlinux.org/paru-bin.git
```
[Mas información sobre AUR](https://es.wikipedia.org/wiki/AUR)

Ahora que tenemos el repositorio clonado, vamos a hacer uso del comando **makepkg** para crear el compilado en base a un repositorio, el comando debemos usarlo en la ruta en la que tenemos el repositorio clonado.
Nota: makepkg tira del paquete base-devel que nosotros instalamos anteriormente.

```console
# creamos el compilado, -s para instalar dependencias si faltan, -i para instalar si se ha ejecutado todo correctamente

foo@bar:~$ makepkg -si

# ejecutar desde la carpeta del repositorio clonado, si se han seguido los pasos, dentro de paru-bin
```

Esta es la pinta que tiene que tener despues de compilar/instalar AUR.  

![aur](/assets/images/instalar-archlinux/aur-ok.png)

## Instalación black arch

Voy a instalar black arch ya estoy cacharreando con el mundo de la ciberseguridad, y esta distro esta dirigida para ello, sobre todo para pentesting.

Yo me he creado un directorio bajo la carpeta personal donde estoy alojando tanto el AUR como el BlackArch.

En esa carpeta vamos a hacer un **curl** para descargarnos lo siguiente.

```console
# -O para nombre al fichero local como al fichero remoto

foo@bar:~$ curl -O https://blackarch.org/strap.sh
```
El comando anterior simplemente nos descarga el fichero, ahora modificamos los permisos para poder ejecutarlo.  

![strappermission](/assets/images/instalar-archlinux/strap-permissions.png)

Ahora ejecutamos el .sh como usuario root.

```console
foo@bar:~$ sudo su

# suponiendo que estamos en la ruta donde esta el fichero
foo@bar:~$ /assets/images/instalar-archlinux/strap.sh
```
  
![blackarchready](/assets/images/instalar-archlinux/blackarch-ready.png)  

Como vemos, ya tenemos black arch instalado, mediante el comando **pacman** podríamos instalar todas las herramientas necesarios para hacer nuestras pruebas de pentesting. 

### Instalación interfaz gráfica

instalamos el sistema gráfico [xorg y xorg-server](https://es.wikipedia.org/wiki/X.Org_Server)

```console
foo@bar:~$ pacman -S xorg xorg-server
foo@bar:~$ pacman -S gnome
```

Hemos instalado el entorno de escritorio [gnome](https://es.wikipedia.org/wiki/GNOME) despues de xorg y xorg server.

Para empezar a ver nuestra interfaz gráfica, solo nos faltaría arrancar el servicio **Gnome Display Manager o GDM** 
```console
# arranca y habilita el servicio

foo@bar:~$ systemctl enable --now gdm.service
```
Esperamos, y ya tenemos nuestra interfaz gráfica  

![gui](/assets/images/instalar-archlinux/interfaz-grafica.png)  

![gui2](/assets/images/instalar-archlinux/gui-2.png)

## Arreglar proporciones de pantalla

Vamos a arreglar las proporciones de pantalla con las vmware tools.

Como root vamos a instalarnos los siguientes paquetes.

```console
foo@bar:~$ pacman -S gtkmm
foo@bar:~$ pacman -S open-vm-tools
foo@bar:~$ pacman -S xf86-video-vmware xf86-input-vmmouse
```

Una vez instalados los paquetes vamos a instalar un daemon de las vmtools

```console
foo@bar:~$ systemctl enable vmtoolsd
```
Reiniciamos la máquina, y si todo ha ido correctamente debería aparecernos con las dimensiones correctas y todo debería funcionar bien.

## Bibliografía, enlaces de interés

[Link descarga ArchLinux](https://archlinux.org/download/)  

[**Video Guía de S4vitar**](https://www.youtube.com/watch?v=fshLf6u8B-w&t=984s)  

[Documentación VMWare](https://docs.vmware.com/en/VMware-Workstation-Pro/index.html)  

[Configuración network adapter](https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/16.0/com.vmware.player.win.using.doc/GUID-C82DCB68-2EFA-460A-A765-37225883337D.html)  

[Cambiar distribución de teclado](https://man.archlinux.org/man/loadkeys.1)  

[systemd](https://wiki.archlinux.org/title/Systemd_(Espa%C3%B1ol))  

[MBR](https://en.wikipedia.org/wiki/Master_boot_record)  

[MBR vs GPT](https://hardzone.es/2018/09/29/mbr-vs-gpt-mejor-disco-duro-ssd/)  

[mkswap](https://man7.org/linux/man-pages/man8/mkswap.8.html)  

[swapon](https://www.computerhope.com/unix/swapon.htm)  

[pacstrap](https://man.archlinux.org/man/extra/arch-install-scripts/pacstrap.8.en)  

[fstab](https://es.wikipedia.org/wiki/Fstab)

[genfstab](https://man.archlinux.org/man/extra/arch-install-scripts/genfstab.8.en)

[arch-chroot](https://man.archlinux.org/man/arch-chroot.8)

[grub](https://linux.die.net/man/8/grub-install)

[neofetch](https://es.wikipedia.org/wiki/Neofetch)  

[gtkmm](https://en.wikipedia.org/wiki/Gtkmm)