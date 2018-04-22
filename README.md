# INSTALACIÓN DE ARCHLINUX CON SOPORTE UEFI #
Ingrese a https://gtronick.github.io/ALIG/ para ver la versión web.

----
*Use el comando:*

    elinks https://gtronick.github.io/ALIG/
    
*Para acceder a esta guía desde el Live system de ArchLinux.*

----
Sitio web de **GTRONICK**: [http://gtronick.com](http://gtronick.com)    
Autor: Jaime Quiroga  
Editado por última vez: **16/04/2018 09:49 AM**

El presente documento no pretende ser una guía completa para la instalación de ArchLinux. Es una guía rápida para acelerar el proceso de instalación. Para más detalles, consultar la [**Wiki**](https://wiki.archlinux.org/index.php/Installation_guide) de ArchLinux, y su guía de instalación.


1. Configurar la BIOS de tu equipo para permitir el arranque desde un dispositivo USB, y el arranque EFI. Si la instalación se está haciendo en VirtualBox, configurar la máquina virtual para permitir el arranque con EFI. Seleccionar la máquina virtual, propiedades, System, Enable EFI.

2. Iniciar la máquina y seleccionar el disco de instalación

3. Seleccionar:

        Arch Linux Arch ISO x86_64 UEFI USB

4. Para verificar que estamos en modo UEFI, ejecutar el siguiente comando: 

        ls /sys/firmware/efi/efivars

    *Si se muestra contenido en la carpeta efivars, quiere decir que arrancamos el sistema correctamente en modo UEFI.*


5. Verificar la conexión a Internet haciendo ping a: archlinux.org (o cualquier otra página o IP)

        ping archlinux.org

6. En caso de tener sólo wifi, usar:

        ip link (Para listar las interfaces. Ubicar la de Wifi, generalmente es wlp2s0)
        wifi-menu -o wlp2s0

    *Seleccionar la red, e ingresar contraseña.*

7. Activar la sincronización del reloj del sistema con Internet: 

        timedatectl set-ntp true

8. Verificar: (opcional)

        timedatectl status

9. Identificar los discos: 

        lsblk

10. Crear una nueva tabla de particiones GPT en /dev/sda:

        gdisk /dev/sda

        w (Para escribir los cambios)
        Y (Para aceptar los cambios)

11. Verificar nuevamente: 

        gdisk /dev/sda

    *Se debe listar "GPT Present" al final de la lista.*

12. Crear partición /boot :

        n (Crea una nueva partición)
        (Dejar número de la partición por defecto, presionando ENTER)
        (Dejar por defecto el sector inicial, presionando ENTER)
        (Para el sector final, escribir +512M y presionar ENTER)
        (Escribir EF00 cuando se pida código de partición y luego ENTER)
        w (Para escribir los cambios y luego ENTER)
        y (Para aceptar los cambios y luego ENTER)

13. Crear particion swap :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +2G
        8200
        W
        Y
        
14. Crear particion / :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +10G
        8304
        W
        Y

15. Crear partición /home :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        ENTER
        8302
        W
        Y

16. Verificar:

        lsblk

17. Formatear partición /boot :

        mkfs.fat -F32 /dev/sda1

18. Formatear particion swap :

        mkswap /dev/sda2

19. Activar swap :

        swapon /dev/sda2

20. Formatear particion / :

        mkfs.ext4 /dev/sda3

21. Formatear partición /home :

        mkfs.ext4 /dev/sda4

22. Montar particion / en /mnt :
        
        mount /dev/sda3 /mnt

23. Crear directorio para /boot :

        mkdir -p /mnt/boot

24. Montar partición /boot :

        mount /dev/sda1 /mnt/boot

25. Crear directorio para /home :

        mkdir -p /mnt/home

26. Montar partición /home :

        mount /dev/sda4 /mnt/home

27. Instalar los paquetes base:

        pacstrap /mnt

    *Esto iniciará la instalación de los paquetes base (191.35 MiB aprox.)*

28. Generar fstab:

        genfstab -U /mnt >> /mnt/etc/fstab

29. Verificar:

        cat /mnt/etc/fstab

30. Iniciar sesión como root en la instalación:

        arch-chroot /mnt /bin/bash

31. Generar locales:

        nano /etc/locale.gen

    *Descomentar las líneas de interés quitando el símbolo #, en este caso:*

        en_US.UTF-8 UTF-8

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*
        
32. Construir el soporte de idioma: 

        locale-gen

33. Crear el archivo de configuración correspondiente:

        nano /etc/locale.conf

    *Agregar el siguiente contenido:*

      LANG=en_US.UTF-8

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

34. Ajustar zona horaria:

        tzselect
        2 
        ENTER
        14 (Número correspondiente a la zona)
        ENTER
        1 (Número correspondiente a la subzona)
        ENTER

35. Crear el link simbólico para hacer el cambio permanente:

        ln -s /usr/share/zoneinfo/<ZONA>/<SUB_ZONA> /etc/localtime

    *donde < ZONA > puede ser America y < SUB_ZONA > puede ser Bogota.*
    
36. Instalar **systemd-boot** (Sólo si no se va a usar GRUB, de lo contrario saltar al paso **40**):

        bootctl --path=/boot install

37. Generar archivo de configuración de systemd-boot:
        
        nano /boot/loader/loader.conf

    Agregar el siguiente contenido:

        default arch
        timeout 0
        editor 0

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

38. Generar el archivo de la entrada por defecto para systemd-boot:

        echo $(blkid -s PARTUUID -o value /dev/sda3) > /boot/loader/entries/arch.conf

    Esto generará un archivo de nombre arch.conf en la ruta especificada, con un contenido similar a:

        14420948-2cea-4de7-b042-40f67c618660

39. Abrir el archivo generado:

        nano /boot/loader/entries/arch.conf

    Se debe agregar lo siguiente, de manera que el serial generado, quede después de PARTUUID y antes de rw, como sigue:

        title ArchLinux
        linux /vmlinuz-linux
        initrd /initramfs-linux.img
        options root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

40. Instalar **GRUB** (sólo si no instaló systemd-boot, de lo contrario saltar al paso **42**):
        
        grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub

    *Si se reporta un error, que indica que /boot no parece ser una partición EFI, verificar que esté correctamente montada en /mnt/boot. Para hacer esto, escribir exit para acceder a la consola del live system. Luego, ejecutar:*
        
        mkdir -p /mnt/boot
        mount /dev/sda1 /mnt/boot
        arch-chroot /mnt/ /bin/bash

    *Repetir el comando de instalación grub-install....*

41. Generar archivo de configuración de grub:
        
        grub-mkconfig -o /boot/grub/grub.cfg

42. Configuración de red:

    *Agregar el nombre del host a /etc/hostname, por ejemplo con:*

        echo gtronick > /etc/hostname

43. Agregar el hostname a /etc/hosts, por ejemplo:
        
        127.0.0.1        localhost.localdomain        localhost
        ::1              localhost.localdomain        localhost
        127.0.1.1        gtronick.localdomain	      gtronick

44. Instalar paquetes para el controlador WiFi:

        pacman -S iw wpa_supplicant dialog

45. Ajustar contraseña para  root:

        passwd

    *Ingresar nueva contraseña*   
    *Repetir la contraseña*


46. Salir de la sesión, desmontar particiones:

        exit
        umount -R /mnt
        umount -R /mnt/boot #si existe o aún está montado

47. Antes de reiniciar, verificar que se hayan desmontado todas las particiones de /dev/sda:

        lsblk

48. Por último reiniciar con:

        reboot

