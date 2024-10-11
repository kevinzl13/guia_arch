# Guia de instalación de Arch Linux

- [Instalar Arch](#instalar-arch)
- [Controladores](#controladores)
- [Entonros Gráficos - VM](#window-manager)

# Instalar Arch

## Primeros Pasos

> Cambiar teclado

- loadkeys la-latin1

> Cambiar fuente

- setfont ter-v14b

> Sincronizar paquetes

- pacman -Sy

> Cambiar Idioma

- echo "es_EC.UTF-8 UTF-8" > /etc/locale.gen
- locale-gen
- export LANG=es_EC.UTF-8

> Comprobar el cambio

pacman -Sy

## Keyring

- pacman-key --init
- pacman-key --populate archlinux

## Comprobar UEFI o BIOS

### BIOS LEGACY

- ls /sys/firmware/efi/efivars

> [!NOTE]
> no deberia salir nada en la ruta si es BIOS

### UEFI

- ls /sys/firmware/efi/efivars

> [!NOTE]
> Deberia salir un listado de archivos si es UEFI

## Comprobar internet

ping -c 3 www.google.com

> Da 3 resultados de apquetes enviados

### Conectar al Internet

ls /sys/class/net | grep w

#### WIFI nmcli

- systemctl start --now NetworkManager.service
- nmcli dev wifi list
- nmcli device wifi connect 'NombreDeRed' password 'contraseña'
- nmcli dev status



#### WIFI iwctl

- systemctl start --now iwd.service
- iwctl --passphrase 'contraseña' station wlo1 connect 'NombreDeRed'


#### Wifi - Red oculta (opcional)

- nano /etc/wpa_supplicant/wpa_supplicant.conf

> network={<br>
    ssid="Familia Hernández"<br>
    scan_ssid=1<br>
    key_mgmt=WPA-PSK<br>
    psk="password"<br>
}

- wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf

#### Ethernet

Se conecta automaticamente


## Discos y Particiones

> Listar discos

- fdisk -l

### Memoria Swap

| Cantidad de RAM en le sistema | Espacio de Intercambio Recomendado    | Espacio de Intercambio si permite Hibernacion |
|-------------------------------|---------------------------------------|-----------------------------------------------|
| 1 GB                          | 2 veces la cantidad de RAM            | 3 veces la cantida de RAM                     |
| 2 GB - 8 GB                   | Igual a la cantidad de RAM            | 2 veces la cantidad de RAM                    |
| 8 GB - 64 GB                  | Al menos 4 GB                         | 1.5 veces la cantidad de RAM                  |
| 64 GB - más                   | Al menos 4 GB                         | No se recomienda Hibernacion                  |

- ● Menos de 1GB RAM física    =  2GB de SWAP
- ● Entre 2GB a 4GB RAM física =  2GB a 4GB de SWAP
- ● 8GB de RAM física          =  4GB de SWAP
- ● Más de 8GB de RAM física   =  2GB a 4GB de SWAP
- ● 8 GB si se encesita Hibernacion

### Crear particiones

#### BIOS LEGACY - MBR DOS

|               | MBR           |
|---------------|---------------|
| **Dialog**            |fdisk <br> parted  |
| **Pseudo-gráficos**   |cfdisk             |
| **No-interactivo**    |sfdisk <br> parted |
| **Gráficamente**      |Gparted <br> gnome-disk-utility <br> Partionmanager|

> **cfdisk**

> [!NOTE]
> sgdisk --zap-all /dev/sda
> **Borra Todo**

> Ingresar al particionado

- cfdisk /dev/sda
- Seleccionar opcion de **dos** para BIOS

> Seleccionar Tamaño Particiones

| **Particionado**  | **Tamaño**    | **Tipo**  | **Inicio**|
|-------------------|---------------|-----------|-----------|
| /dev/sda1         | 300M          | Linux                 | **Marcar como inicio**    |
| /dev/sda2         | 4G            | Linux swap / Solaris  |                           |
| /dev/sda3         | Resto del espacio asignar |Linux      |                           |

- Escribir cambios
- Salir

> Formatear Particiones
> -L se utiliza para dar etiqueta (LABEL)

- mkfs.ext4 -L "Bios" /dev/sda1
- mkswap /dev/sda2
- swapon /dev/sda2
- mkfs.ext4 -L "Root" /dev/sda3

> Revisar Disco

- fdisk -l
- lsblk

> Montar Particiones

- mount /dev/sda3 /mnt/
- mkdir -p /mnt/boot
- mount /dev/sda1 /mnt/boot


#### UEFI

|               | GPT           |
|---------------|---------------|
| **Dialog**            |fdisk <br> parted <br> gdisk   |
| **Pseudo-gráficos**   |cfdisk  <br> cgdisk            |
| **No-interactivo**    |sfdisk <br> parted <br> sgdisk |
| **Gráficamente**      |Gparted <br> gnome-disk-utility <br> partionmanager|

> **cfdisk**

> [!NOTE]
> sgdisk --zap-all /dev/sda
> **Borra Todo**

> Ingresar al particionado

- cfdisk /dev/sda
- Seleccionar opcion de **gpt** para UEFI

> Seleccionar Tamaño Particiones

| **Particionado**  | **Tamaño**    | **Tipo**  |
|-------------------|---------------|-----------|
| /dev/sda1         | 300M          | Sistema EFI   |
| /dev/sda2         | 4G            | Linux swap    |
| /dev/sda3         | Resto del espacio asignar     | Sistema de ficheros Linux|

- Escribir cambios
- Salir

> Formatear Particiones
> -n se utiliza para dar etiqueta (LABEL)

- mkfs.fat -F32 -n "UEFI" /dev/sda1
- mkswap /dev/sda2
- swapon /dev/sda2
- mkfs.ext4 /dev/sda3

> Revisar Disco

- fdisk -l
- lsblk

> Montar Particiones

- mount /dev/sda3 /mnt
- mkdir -p /mnt/boot/efi
- mount /dev/sda1 /mnt/boot/efi


## Instalar sistema base

- pacstrap /mnt base base-devel nano
- ls /mnt/

## Instalar Kernel

- pacstrap /mnt linux linux-headers mkinitcpio linux-firmware

> Listado de Kernels

#### ● linux

> Stable
> Núcleo y módulos Vanilla Linux, con algunos parches aplicados.

- sudo pacman -S linux linux-headers mkinitcpio linux-firmware

#### ● linux-hardened

> Hardened
> Un kernel de Linux centrado en la seguridad que aplica un conjunto de parches de refuerzo para mitigar las vulnerabilidades del kernel y del espacio de usuario.

- sudo pacman -S linux-hardened linux-hardened-headers mkinitcpio linux-firmware

#### ● linux-lts

> Longterm LTS
> Kernel y módulos de Linux con soporte a largo Soporte.

- sudo pacman -S linux-lts linux-lts-headers mkinitcpio linux-firmware

#### ● linux-rt

> Zen Kernel
> Resultado de un esfuerzo de colaboración de los piratas informáticos del kernel para proporcionar el mejor kernel de Linux posible para los sistemas cotidianos.

- sudo pacman -S linux-zen linux-zen-headers mkinitcpio linux-firmware

#### ● linux-zen

> Realtime kernel
> Mantenido por un pequeño grupo de desarrolladores principales liderados por Ingo Molnar. Este parche permite adelantarse a casi todo el kernel, con la excepción de unas pocas regiones de código muy pequeñas.

- sudo pacman -S linux-rt linux-rt-headers mkinitcpio linux-firmware

> [!IMPORTAT]
> Es importante saber que para instalar drivers para la grafica, ethernet y otros componentes,
cambia los nombres de los drivers, si tienes el kernel lts va driver-lts para otros kernel como ZEN o RT o HARDENED driver-dkms

### Montar discos automaticamente

- genfstab -p /mnt >> /mnt/etc/fstab

## Entrar al sistema Arch

- arch-chroot /mnt

### Configuraciones generales

> Cambiar Idioma

- nano /etc/locale.gen **descomentar el idioma que se quiere** es_EC.UTF-8 UTF-8 y guardar
- locale-gen
- export LANG="es_EC.UTF-8"
- echo "LANG=es_EC.UTF-8" > /etc/locale.conf

> Zona horaria

- ls /usr/share/zoneinfo/ **listado de ciudades**
- https://ipapi.co/timezone **tu zona horaria**
- ln -sf /usr/share/zoneinfo/**tuciudad**/**pais** /etc/localtime
- pacman -S curl
- ln -sf -/usr/share/zoneinfo/$(curl https://ipapi.co/timezone) /etc/localtime

> Sincronizar las horas

- hwclock -w

> Configurar teclado

- echo KEYMAP=**tu telcado** > /etc/vconsole.conf
- echo KEYMAP=latam > /etc/vconsole.conf

> Configurar Equipo

- echo **nombre PC** > /etc/hostname
- passwd **password de root**
- useradd -m -g users -G wheel -s /bin/bash **usuario**
- passwd **usuario**

> añadir sudo a usuario

- nano /etc/sudoers

> buscar user privilege abajo de root poner

- **usuario** ALL=(ALL:ALL) ALL

### Servicios de internet

- pacman -S dhcp dhcpcd networkmanager iwd
- systemctl enable dhcpcd
    - systemctl status dhcpcd
    - systemctl start dhcpcd
- systemctl enable NetworkManager
    - systemctl status NetworkManager
    - systemctl start NetworkManager
- systemctl enable iwd
    - systemctl status iwd
    - systemctl start iwd

### Repositorios

- pacman -S reflector
- reflector --verbose --latest 10 --sort rate --save /etc/pacman.d/mirrorlist

### Modo de Arranque - GRUB

#### BIOS

- ls /boot
- pacman -S grub os-prober
- grub-install /dev/sda **ruta del disco**

#### UEFI

- ls /boot/efi
- pacman -S grub efibootmgr os-prober
- grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=**Arch**
- grub-install --target=x86_64-efi --efi-directory=/boot/efi --removable


### Agregar icono de Arch

- pacman -S plymouth
- **temas** ls /usr/share/plymouth/themes

> modificar el grub

- nano /etc/default/grub
- GRUB_CMDLINE_LINUX_DEFAULT="quiet splash numlock=on"
- GRUB_DISABLE_OS_PROBER=false
- grub-mkconfig -o /boot/grub/grub.cfg

### Configurar Pacman (opcional)

- nano /etc/pacman.conf

> Descomentar

- Color
- VerbosePkgLists
- ParalleDownloads=5

> Añadir
- ILoveCandy

> Descomentar

- [multilib]
- Include

### Carpetas personales (Desktop, Downloads, etc)

- pacman -S xdg-user-dirs
- xdg-user-dirs-update
- su **usuario** -c "xdg-user-dirs-update"

### Tipografias Basicas

- pacman -S gnu-free-fonts ttf-hack ttf-inconsolata noto-fonts-emoji ttf-dejavu

### Adicionales

- pacman -S neofetch lsb-release git wget

## Salir del sistema

- exit
- umount -R /mnt
- swapoff /dev/sda2 **/dev/tu swap**
- reboot

# Controladores

## Video

### Genericos

- sudo pacman -S xf86-video-vesa xf86-video-fbdev

### AMD

### Intel

### Nvidia

## Audio

- Alsa
- Jack
- PulseAudio
- Pipewire

### Alsa

- sudo pacman -S alsa alsa-firmware alsa-utils alsa-plugins

### Jack Audio

- sudo pacman -S jack2 jack-example-tools a2jmidid realtime-privileges

> Interfaz para controlar JACK

- sudo pacman -S cadence
- sudo pacman -S carla

### PulseAudio

- sudo pacman -S pulseaudio pulseaudio-alsa pamixer pavucontrol

> Pulse Audio con soporte de Bluetooh

- sudo pacman -S pulseaudio-bluetooth bluez bluez-utils
- sudo systemctl enable --now bluetooth

### Pipewire

- sudo pacman -S pipewire pipewire-audio gst-plugin-pipewire pipewire-alsa pipewire-jack pipewire-pulse pipewire-roc wireplumber realtime-privileges
- systemctl --user enable  pipewire.service
- systemctl --user enable  pipewire.socket



# Gestores de ventana

## Display Manager

- ● lightdm
- ● ggd
- ● sddm
- ● slim
- ● Más

## Desktop

- ● GNOME
- ● KDE Plasma
- ● Xfce
- ● Deepin
- ● Mate
- ● Lxde
- ● Lxqt
- ● Más

### Gnome

> Minimalista

- sudo pacman -S gnome-shell gdm gnome-control-center gnome-tweaks gnome-shell-extensions gnome-menus gnome-terminal nautilus gnome-calculator gnome-keyring gnome-system-monitor gedit gnome-backgrounds gnome-calendar gnome-clocks gnome-color-manager gnome-disk-utility sushi totem gnome-user-docs dconf-editor gnome-icon-theme-extras gnome-themes-extra

> Completo

- sudo pacman -S gnome gnome-extra

> Activar servicio GDM

- sudo systemctl enable gdm.service

### KDE Plasma

> Minimalista

- sudo pacman -S plasma konsole dolphin dolphin-plugins kfind kdialog spectacle kontact kmix kalarm kalendar kate kdeconnect ffmpegthumbs kdegraphics-thumbnailers kdesdk-thumbnailers svgpart

> Completo

- sudo pacman -S plasma kde-applications

> Kde Plasma con Wayland

- sudo pacman -S plasma-wayland-session egl-wayland

> Activar servicio SDDM

- sudo systemctl enable sddm.service

### XFCE4

> Completo

- sudo pacman -S xfce4 xfce4-goodies network-manager-applet lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings light-locker accountsservice

> Activar servicio LightDM

- sudo systemctl enable lightdm.service

### Deepin

> Completo

- sudo pacman -S deepin deepin-extra lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings light-locker accountsservice

> Activar servicio LightDM

- sudo systemctl enable lightdm.service


### Mate

> Completo

- sudo pacman -S mate mate-extra mate-applet-dock network-manager-applet mate-media mate-power-manager lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings light-locker accountsservice mugshot

> Activar servicio LightDM

- sudo systemctl enable lightdm.service

### LXDE

> Compelto

- sudo pacman -S lxde lxmed slock lxdm accountsservice mugshot

> Activar servicio LXDM

- sudo systemctl enable lxdm.service

### LXQT

> Compelto

- sudo pacman -S lxqt papirus-icon-theme arc-gtk-theme xscreensaver connman lxqt-connman-applet connman-gtk sddm light-locker accountsservice mugshot

> Activar servicio SSDM

- sudo systemctl enable sddm.service

> Desactivamos NetworkManaget usara Connman

- sudo systemctl disable NetworkManager && sudo systemctl enable connman

## Window Manager

- ● Qtile
- ● i3
- ● bspwm
- ● xmonad
- ● AwesomeWm
- ● dwm
- ● Spectrwm
- ● Más


# Repositorios

## Yay

- git clone https://aur.archlinux.org/yay.git
- cd yay
- makepkg -si

## Paru

- git clone https://aur.archlinux.org/paru.git
- cd paru
- makepkg -si
