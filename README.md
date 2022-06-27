# Arch-LInux-installguide-with-Qtile
```
# Teclado Latino:
loadkeys la-latin1

# Conectar a internet temporalmente 
iwctl
station wlan0 connect zafira
exit 

# Actualizar el reloj del sistema
timedatectl set-ntp true

# Ver particiones
lsblk

# Crear particiones
fdisk /dev/nvme0n1 

g --> Formato GPT
n --> nueva partición
Partition number --> enter
Firs sector --> enter
Last sector --> +512M

n --> nueva partición 
Partition number --> enter
Firs sector --> enter 
Last sector --> +16G

n --> nueva partición 
Partition number --> enter
Firs sector --> enter 
Last sector --> enter para disco entero

w  --> Guardar cambios

Debe de verse así:
    nvme0n1     259:0    0 476.9G  0 disk
    ├─nvme0n1p1 259:1    0   512M  0 part /mnt/boot/EFI
    ├─nvme0n1p2 259:2    0    16G  0 part [SWAP]
    └─nvme0n1p3 259:3    0 460.4G  0 part /mnt
    nvme1n1     259:4    0  27.3G  0 disk
    
    
# Formatear particiones

# mkfs.fat -F 32 /dev/efi_system_partition
mkfs.fat -F 32 /dev/nvme0n1p1

# mkswap /dev/swap_partition
mkswap /dev/nvme0n1p2

# mkfs.ext4 /dev/root_partition
mkfs.ext4 /dev/nvme0n1p3


# Hacer monturas

# Creando las carpetas para montar el boot y EFI
mkdir /mnt/boot
mkdir /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi

# swapon /dev/swap_partition
swapon /dev/nvme0n1p2

# Montar el sistema raíz
mount /dev/nvme0n1p3 /mnt


# Instalar Linux
pacstrap /mnt base linux linux-firmware nano networkmanager network-manager-applet wireless_tools wpa_supplicant os-prober mtools dosfstools base-devel linux-headers

# Generar FSTAB
genfstab -U /mnt >> /mnt/etc/fstab


# Vamos a ingresar al modo instalación, actualmente estamos en la versión Live.
arch-chroot /mnt

# Configurar Timezone
ln -sf /usr/share/zoneinfo/America/Mexico_City /etc/localtime

# Sincronizar reloj
hwclock --systohc

# Configurar el idioma
nano /etc/locale.gen

# Tenemos que descomentar la línea de nuestro idioma
es_MX.UTF-8
en_US.UTF-8

# Generamos el archivo locale.gen con el comando
locale-gen

# Agregamos el lenguaje a locale.conf
nano /etc/locale.conf
LANG=es_MX.UTF-8
LANG=en_US.UTF-8

# Configurar nombre (hostname), es el nombre del equipo, en mi caso se llamará «omen-laptop»
echo omen > /etc/hostname

# Configurar el dns local
nano /etc/hosts

# Pegamos lo siguiente:
127.0.0.1 localhost
::1 localhost
127.0.0.1 omen.localdomain omen

# Configurar distribución de teclado
echo KEYMAP=la-latin1 > /etc/vconsole.conf

# Establecer contraseña del Administrador (root)
passwd

# Instalación de Grub
pacman -S grub efibootmgr

# Instalación 
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

# Crear configuración
grub-mkconfig -o /boot/grub/grub.cfg

# Salimos 
exit

# Desmontar las particiones y reiniciar
umount -a
reboot

# Activando NetworkManager
systemctl start NetworkManager
systemctl enable NetworkManager

systemctl start wpa_supplicant.service
systemctl enable wpa_supplicant.service

# Crear usuario
useradd -m -g users -G audio,lp,optical,storage,video,wheel,games,power,scanner,http,adm -s /bin/bash nksistemas

    audio – Privilegios para configurar el audio.
    lp – Privilegios para configurar impresoras.
    optical – Configurar y manejo de unidades ópticas, CD, DVD, etc.
    storage – Manejo de almacenamiento, Pendrives, Discos USB, etc.
    video – Manejo de hardware de vídeo.
    wheel – Imprescindible para usar el comando sudo.
    games – Permisos para instalar juegos.
    power – Apagar, encender o suspender la máquina.
    scanner – Instalación y manejo de escáneres.
    http – Acceso a archivos servidos por servidores HTTP.
    adm - Admin
    
# Contraseña para usuario 
passwd manu320

# Agregar grupo al usuario
usermod -aG wheel,adm manu320

# Ver los grupos que tiene el usuario
groups manu320


# Conectar a red wifi con nmcli
# Asegúrate de que la radio WiFi está activada (por defecto): 
nmcli radio wifi on

# Actualizar la lista de conexiones Wi-Fi disponibles
nmcli device wifi rescan

# Ver los puntos de acceso Wi-Fi disponibles
nmcli dev wifi list

# Conectarse a una conexión Wi-Fi mediante nmcli
nmcli dev wifi connect SSID-Name password wireless-password


# Instalar sudo
pacman -S sudo

# Editamos el archivo sudoers
nano /etc/sudoers

# Descomentamos y guardamos
%wheel ALL=(ALL:ALL) ALL

# Instalar Driver de video para Nvidia
pacman -S nvidia nvidia-utils

# Instalamos neofetch
pacman -S neofetch

# Instalamos git
pacman -S git

# Desde manu320 (usuario)
# Creamos carpeta para nuestros repositorios del sistema
mkdir -p Repos

# Accedenos a la carpeta
cd !$

# Descargamos paru
git clone https://aur.archlinux.org/paru-bin.git

# Accedemos a paru-bin
cd paru-bin/

# Instalamos paru-bin
makepkg -si


# Instalación de repositorios BlackArch en carpeta para BlackArch en Repos
mkdir blackarch
cd !$
curl -O https://blackarch.org/strap.sh
chmod +x strap.sh
sudo ./strap.sh

# Sincronizar y actualizar paquetes 
sudo pacman -Syyu

# Instalar interfaz gráfica
pacman -S xorg xorg-server

# Instalar lo necesario para Qtile
sudo pacman -S lightdm lightdm-gtk-greeter qtile xterm code firefox
sudo systemctl enable lightdm
reboot
```

## Qtile commands

Key | Action
--- | ---
mod + return |	launch xterm
mod + k |	next window
mod + j |	previous window
mod + w |	kill window
mod + [asdfuiop] |go to workspace [asdfuiop]
mod + ctrl + r | restart qtile
mod + ctrl + q | logout


**Recuerda actualizar Qtile con cada cambio que se haga a la configuración**


```

# Configurar de manera temporal el idioma de teclado a latam
setxkbmap latam

# Installar kitty como terminal 
sudo pacman -S kitty

# En ~/.config/qtile/config.py podremos editar la configuración de qtile
code ~/.config/qtile/config.py

# Instalar programa ejecutador de programas
sudo pacman -S rofi

# Para cambiar el tema de rofi instala which
sudo pacman -S which
rofi-theme-selector

# Wallpaper
sudo pacman -S feh
feh --bg-scale path/to/wallpaper

# Para controllar brillo de pantalla instala brightnessctl
sudo pacman -S brightnessctl

# Brightness
Key([], "XF86MonBrightnessUp", lazy.spawn("brightnessctl set +10%")),
Key([], "XF86MonBrightnessDown", lazy.spawn("brightnessctl set 10%-")),

    # Software util para reproducir audio
    pacman -S --noconfirm gstreamer gst-plugins-bad gst-plugins-base  gst-plugins-base-libs gst-plugins-good gst-plugins-ugly xine-lib libdvdcss libdvdread dvd+rw-tools vlc lame

    #Audio
    sudo pacman -Sy alsa-utils pulseaudio

    # Audio controller with pamixer
    sudo pacman -S pamixer
    
    # Esto es sagrado para el audio de la omen
    sudo pacman -S sof-firmware sof-tools
    reboot

# Volume
Key([], "XF86AudioLowerVolume", lazy.spawn("pamixer --decrease 5")),
Key([], "XF86AudioRaiseVolume", lazy.spawn("pamixer --increase 5")),
Key([], "XF86AudioMute", lazy.spawn("pamixer --toggle-mute")),


# Zsh y OhMyZSH
# Realizar tanto en usuario como en root
sudo pacman -S zsh zsh-completions
chsh -s /bin/zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"


```
