# INSTALACIÓN DE ARCHLINUX CON SOPORTE UEFI #
# DUAL BOOT CON WINDOWS 10 #
Ingrese a https://gtronick.github.io/ALIG-DUAL/ para ver la versión web.

----
*Use el comando:*

    elinks https://gtronick.github.io/ALIG-DUAL/
    
*Para acceder a esta guía desde el Live system de ArchLinux.*

----
Sitio web de **GTRONICK**: [http://gtronick.com](http://gtronick.com)    
Autor: Jaime Quiroga  
Editado por última vez: **11/09/2018 10:51 AM**

El presente documento no pretende ser una guía completa para la instalación de ArchLinux. Es una guía rápida para acelerar el proceso de instalación. Para más detalles, consultar la [**Wiki**](https://wiki.archlinux.org/index.php/Installation_guide) de ArchLinux, y su guía de instalación.

Pasos preliminares en Windows 10:

1. Asegurarse de que Windows está instalado en modo UEFI

    Arranque Windows    
    Pulse la tecla Win y "R" para iniciar la ventana Ejecutar    
    En la ventana Ejecutar teclee "msinfo32" y pulse Enter    
    En la ventana Información del sistema, seleccione Resumen del sistema en la izquierda y verifique el valor de modo de BIOS en la            derecha.    
    Si el valor es UEFI, Windows se inicia en modo UEFI-GPT. Si el valor es Heredado('legacy'), Windows se inicia en modo BIOS-MBR.
    
2. Asegúrese de deshabilitar el fastboot y el secureboot en la configuración de su BIOS
    
3. Libere como mínimo 10 GB de disco duro para la instalación de ArchLinux
    
    Arranque Windows    
    Abra MiPC    
    Clic derecho sobre Este equipo    
    Clic en Administrar    
    En la parte izquierda de la pantalla hacer clic en Administrador de discos    
    Seleccionar el disco a reducir y hacer clic derecho sobre él    
    Seleccionar Reducir Volumen    
    Especificar la cantidad de espacio en MB a reducir    
    Aceptar y reiniciar    

Instalación de ArchLinux:

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

10. Verificar la tabla de particiones: 

        gdisk /dev/sda

    *Se debe listar "GPT Present" al final de la lista.*

11. Crear particion swap :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +2G
        8200
        W
        Y
        
12. Crear particion / :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        +10G
        8304
        W
        Y

13. Crear partición /home :

        gdisk /dev/sda
        n
        ENTER
        ENTER
        ENTER
        8302
        W
        Y

14. Verificar:

        lsblk

15. Formatear particion swap :

        mkswap /dev/sda5

16. Activar swap :

        swapon /dev/sda5

17. Formatear particion / :

        mkfs.ext4 /dev/sda6

18. Formatear partición /home :

        mkfs.ext4 /dev/sda7

19. Montar particion / en /mnt :
        
        mount /dev/sda6 /mnt

20. Crear directorio para /boot :

        mkdir -p /mnt/boot

21. Montar partición /boot :

        mount /dev/sda2 /mnt/boot

22. Crear directorio para /home :

        mkdir -p /mnt/home

23. Montar partición /home :

        mount /dev/sda7 /mnt/home

24. Instalar los paquetes base:

        pacstrap /mnt

    *Esto iniciará la instalación de los paquetes base (191.35 MiB aprox.)*

25. Generar fstab:

        genfstab -U /mnt >> /mnt/etc/fstab

26. Verificar:

        cat /mnt/etc/fstab

27. Iniciar sesión como root en la instalación:

        arch-chroot /mnt /bin/bash

28. Generar locales:

        nano /etc/locale.gen

    *Descomentar las líneas de interés quitando el símbolo #, en este caso:*

        en_US.UTF-8 UTF-8

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*
        
29. Construir el soporte de idioma: 

        locale-gen

30. Crear el archivo de configuración correspondiente:

        nano /etc/locale.conf

    *Agregar el siguiente contenido:*

      LANG=en_US.UTF-8

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

31. Ajustar zona horaria:

        tzselect
        2 
        ENTER
        14 (Número correspondiente a la zona)
        ENTER
        1 (Número correspondiente a la subzona)
        ENTER

32. Crear el link simbólico para hacer el cambio permanente:

        rm /etc/localtime
        ln -s /usr/share/zoneinfo/<ZONA>/<SUB_ZONA> /etc/localtime

    *donde < ZONA > puede ser America y < SUB_ZONA > puede ser Bogota.*
    
33. Instalar **systemd-boot**:

        bootctl --path=/boot install

34. Generar archivo de configuración de systemd-boot:
        
        nano /boot/loader/loader.conf

    Agregar el siguiente contenido:

        default arch
        timeout 3
        editor 0

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

35. Generar el archivo de la entrada por defecto para systemd-boot:

        echo $(blkid -s PARTUUID -o value /dev/sda6) > /boot/loader/entries/arch.conf

    Esto generará un archivo de nombre arch.conf en la ruta especificada, con un contenido similar a:

        14420948-2cea-4de7-b042-40f67c618660

36. Abrir el archivo generado:

        nano /boot/loader/entries/arch.conf

    Se debe agregar lo siguiente, de manera que el serial generado, quede después de PARTUUID y antes de rw, como sigue:

        title ArchLinux
        linux /vmlinuz-linux
        initrd /initramfs-linux.img
        options root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw

    *Guardar presionando Ctrl + X, luego Y y finalmente ENTER*

37. Configuración de red:

    *Agregar el nombre del host a /etc/hostname, por ejemplo:*

        echo gtronick > /etc/hostname

38. Agregar el hostname a /etc/hosts, por ejemplo:

        nano /etc/hosts
        
    *Agregar el siguiente contenido, reemplazando gtronick por tu hostname*
        
        127.0.0.1        localhost.localdomain        localhost
        ::1              localhost.localdomain        localhost
        127.0.1.1        gtronick.localdomain	      gtronick

39. Instalar paquetes para el controlador WiFi:

        pacman -S iw wpa_supplicant dialog

40. Ajustar contraseña para  root:

        passwd

    *Ingresar nueva contraseña*   
    *Repetir la contraseña*


41. Salir de la sesión, desmontar particiones:

        exit
        umount -R /mnt
        umount -R /mnt/boot #si existe o aún está montado

42. Antes de reiniciar, verificar que se hayan desmontado todas las particiones de /dev/sda:

        lsblk

43. Por último reiniciar con:

        reboot
  
44. Después de reiniciar el equipo con ArchLinux instalado, crear un nuevo usuario, por ejemplo:

        useradd -m myUser
        
45. Asignar una contraseña al nuevo usuario creado:

        passwd myUser
        
46. Dar permisos de uso para Sudo al nuevo usuario:

        visudo
        
    *Buscar la línea  ROOT  ALL=(ALL) ALL y justo debajo de esta, agregar nuestro usuario, por ejemplo:*
        
        myUser   ALL=(ALL) ALL
        
    *Presionar : (dos puntos) luego q y finalmente ENTER.*

Visita mi canal de YouTube para ver la instalación en video y otros tutoriales: [GTRONICK](https://www.youtube.com/user/GTRONICK)
