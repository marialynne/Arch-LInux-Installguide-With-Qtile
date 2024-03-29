# Arch LInux installguide with Qtile
***
## Preinstalación

### Teclado latino
```
# Teclado Latino:
loadkeys la-latin1
```
### Wifi temporal
```
# Activar tarjeta 
rfkill unblock all
# Conectar a internet temporalmente 
iwctl
station wlan0 connect zafira
exit 

# Actualizar el reloj del sistema
timedatectl set-ntp true
```
### Particiones 
```
# Ver particiones
lsblk

# Crear particiones
fdisk /dev/nvme0n1 
```
```
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
    
``` 
### Formatear particiones
```
# Formatear particiones

# mkfs.fat -F 32 /dev/efi_system_partition
mkfs.fat -F 32 /dev/nvme0n1p1

# mkswap /dev/swap_partition
mkswap /dev/nvme0n1p2

# mkfs.ext4 /dev/root_partition
mkfs.ext4 /dev/nvme0n1p3
```
### Monturas
Cuidado aquí y asegurate de hacer las monturas en el orden correcto o al instalar el grub este no detectará el efi
```
# Hacer monturas

# Montar el sistema raíz
mount /dev/nvme0n1p3 /mnt

# swapon /dev/swap_partition
swapon /dev/nvme0n1p2

# Creando las carpetas para montar el boot y EFI
mkdir /mnt/boot
mkdir /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi
```
### Instalar Linux 
```
# Instalar Linux
pacstrap /mnt base linux linux-firmware nano networkmanager network-manager-applet wireless_tools wpa_supplicant dosfstools base-devel

# Generar FSTAB
genfstab -U /mnt >> /mnt/etc/fstab
```
### Modo instalación live
```
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
```
***
## Post instalación

### Activar servicios de internet
```
# Activando NetworkManager
systemctl start NetworkManager
systemctl enable NetworkManager

systemctl start wpa_supplicant.service
systemctl enable wpa_supplicant.service
```
### Conectarse a wifi
```
# Conectar a red wifi con nmcli
# Asegúrate de que la radio WiFi está activada (por defecto): 
nmcli radio wifi on

# Actualizar la lista de conexiones Wi-Fi disponibles
nmcli device wifi rescan

# Ver los puntos de acceso Wi-Fi disponibles
nmcli dev wifi list

# Conectarse a una conexión Wi-Fi mediante nmcli
nmcli dev wifi connect SSID-Name password wireless-password
```
### Creación de usuario
```
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
```
### Sudo
```
# Instalar sudo
pacman -S sudo

# Editamos el archivo sudoers
nano /etc/sudoers

# Descomentamos y guardamos
%wheel ALL=(ALL:ALL) ALL
```
### Drivers de Nvidia
```
# Instalar Driver de video para Nvidia
pacman -S nvidia nvidia-utils nvidia-settings 
```
### Neofetch
```
# Instalamos neofetch
pacman -S neofetch
```
### Git
```
# Instalamos git
pacman -S git
```
### Xorg
```
# Instalar interfaz gráfica
pacman -S xorg xorg-server
```
### Directorios 
```
sudo pacman -S xdg-user-dirs
```
### Qtile
```
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

### Cambiar idioma de teclado
```
# Configurar de manera temporal el idioma de teclado a latam
setxkbmap latam
```
### Kitty terminal
```
# Installar kitty como terminal 
sudo pacman -S kitty

# En ~/.config/qtile/config.py podremos editar la configuración de qtile
code ~/.config/qtile/config.py
```
### Rofi
```
# Instalar programa ejecutador de programas
sudo pacman -S rofi

# Para cambiar el tema de rofi instala which
sudo pacman -S which
rofi-theme-selector

Key([mod], "m", lazy.spawn("rofi -show run")),
Key([mod, 'shift'], "m", lazy.spawn("rofi -show")),

```
### Brillo y audio
```
# Para controllar brillo de pantalla instala brightnessctl
sudo pacman -S brightnessctl

# Brightness
Key([], "XF86MonBrightnessUp", lazy.spawn("brightnessctl set +10%")),
Key([], "XF86MonBrightnessDown", lazy.spawn("brightnessctl set 10%-")),

    #Audio
    sudo pacman -Sy alsa-utils pulseaudio

    # Audio controller with pamixer
    sudo pacman -S pamixer
    
    # Esto es sagrado para el audio de la omen
    sudo pacman -S sof-firmware sof-tools
    reboot
    
    # Configurar el audio en amixer
    alsamixer
    # FN F6 --> Selecciona tu tarjeta de audio y desmuteala

# Volume
Key([], "XF86AudioLowerVolume", lazy.spawn("pamixer --decrease 5")),
Key([], "XF86AudioRaiseVolume", lazy.spawn("pamixer --increase 5")),
Key([], "XF86AudioMute", lazy.spawn("pamixer --toggle-mute")),
```
### OhMyZSH y ZSH
Hacerlo como usuario y root
```
# Zsh y OhMyZSH
# Realizar tanto en usuario como en root
sudo pacman -S zsh zsh-completions
chsh -s /bin/zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
### Ranger y thunar
```
sudo pacman -S thunar ranger
```
### Gwet
```
sudo pacman -S wget
```
### Unzip
```
sudo pacman -S unzip
```
### SSH
```
sudo pacman -S --needed base-devel git wget yajl
cd /tmp
git clone https://aur.archlinux.org/package-query.git
cd package-query/
makepkg -si && cd /tmp/
git clone https://aur.archlinux.org/yaourt.git
cd yaourt/
makepkg -si
yaourt -S openssh
mkdir -p ~/.config/systemd/user/
nano ~/.config/systemd/user/ssh-agent.service
```
Pegar esto
```
[Unit]
Description=SSH key agent

[Service]
Type=forking
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
ExecStart=/usr/bin/ssh-agent -a $SSH_AUTH_SOCK

[Install]
WantedBy=default.target
```
Exportar variables
```
#.bash_profile
export SSH_AUTH_SOCK="$XDG_RUNTIME_DIR/ssh-agent.socket"

# ~/.config/fish/config.fish
set -x SSH_AUTH_SOCK $XDG_RUNTIME_DIR/ssh-agent.socket

reboot
```
Activar servicio ssh
```
systemctl --user enable --now ssh-agent
```
Verificar versión
```
ssh -V
```
Creación de llaves
```

ssh-keygen

# Activar agente
eval $(ssh-agent -s)

# Registrar 
ssh-add ~/.ssh/id_rsa

# Pegar llave pública en Github
cd .ssh && cat id_rsa.pub
```

***
### Repos
(De manera manuel)
```
# Desde manu320 (usuario)
# Creamos carpeta para nuestros repositorios del sistema
mkdir -p Repos

# Accedenos a la carpeta
cd !$
```
### Paru 
(De manera manual)
```
# Descargamos paru
git clone https://aur.archlinux.org/paru-bin.git

# Accedemos a paru-bin
cd paru-bin/

# Instalamos paru-bin
makepkg -si

```
### BlackArch
(De manera manual)
```
# Instalación de repositorios BlackArch en carpeta para BlackArch en Repos
mkdir blackarch
cd !$
curl -O https://blackarch.org/strap.sh
chmod +x strap.sh
sudo ./strap.sh

# Sincronizar y actualizar paquetes 
sudo pacman -Syyu
```
### Snap 
(De manera manual)
```
# Instalación de snap
cd repos
git clone https://aur.archlinux.org/snapd.git
cd snapd
makepkg -si

sudo systemctl enable snapd
reboot
```
### Spotify snap
```
sudo snap install spotify
```
### Postman
```
git clone https://aur.archlinux.org/postman-bin.git
# pasos siguientes a makepkg -si
```
### Brave broweser
```
cd Repos 
git clone https://aur.archlinux.org/brave-bin.git
cd brave-bin
makepkg -si
```
### Dunst notifications
```
sudo pacman -S dunst
```
### Media control
```
sudo pacman -S playerctl 
```
### Bluetooth
```
sudo pacman -S bluez blueman bluez-utils pulseaudio-bluetooth pavucontrol
sudo systemctl start bluetooth.service
sudo systemctl enable bluetooth.service
reboot

# Config file
sudo vim /etc/bluetooth/main.conf
```
### Wallpaper feh
```
sudo pacman -S feh
feh --bg-scale path/to/wallpaper
```
### Sensores de hardware
(No necesario)
```
sudo pacman -S htop
```
Necesario
```
sudo pacman -S zip unzip tree which redshift qt5ct lxappearance python-psutil
# nerdfonts
cd Repos
git clone https://aur.archlinux.org/nerd-fonts-complete.git 
cd nerd-fonts-complete
makepg -si

# comic-code
# Instalalo en zip dentro de /usr/share/fonts
mkdir comic-code
cd comic-code
unzip comic-code.zip

# Actualizamos cache
fc-cache

# Para saber las posibles fuentes dispoibles desde kitty
kitty +list-fonts --psnames | grep comic
# o también, pero el de arriaba es mejor por el filtrado grep
kitty +list-fonts --psnames 

```
# Para los warnings:
```
sudo pacman -Syu linux-firmware-qlogic

```



## Linux

### Ver porcentaje de bateria 

`cat /sys/class/power_supply/BAT1/capacity`


### Conectar a Internet

* List all available networks

`nmcli device wifi list`
* Connect to your network

`nmcli device wifi connect YOUR_SSID password YOUR_PASSWORD`

### Aplicaciones en segundo plano

* Comando para ejecutar aplicaciones en segundo plano:
Se agrega `&` al final, ejemplo:

`firefox &`

***

## Qtile
* Comando para cambiar idioma de teclado en ArchLinux Qtile:

`setxkbmap latam`

***

## Pacman 

### Información

* Comando para obtener información de una aplicación en ArchLInux:

`pacman -Qi software-name`

* Comando para saber si tenemos un software instalado:

`pacman -Q software-name`

> Esto devuelte el nombre del programa y su versión

### Actualizar

> No es prudente actualizar si, por ejemplo, se necesitará tener el sistema estable por motivos de trabajo. Más bien, actualizar durante el tiempo libre y estar preparados para hacer frente a cualquier problema que pueda surgir.

* Comando para buscar una aplicación oficial:

`sudo pacman -S software-name`

* Comando para sincronizar paquetes:

`sudo pacman -Sy`

* Comando para actualizar paquetes:

`sudo yay -Syu`

* **Comando para sincronizar y actualizar paquetes al mismo tiempo:**

`sudo pacman -Syyu`

### Eliminar 

* Comando para borrar aplicaciones instaladas:

`sudo pacman -R software-name`

* Comando para borrar aplicaciones y dependencias:

`sudo pacman -Rs software-name`

* Comando para borrar aplicaciones y dependencias junto a archivos de configuración:

`sudo pacman -Rsn software-name`

* Comando para eliminar un software sin importar dependencias:

`pacman -Rdd software-name`

* Comando para eliminar todos los paquetes de que depende un software:

**Podría eliminar paquetes importantes**
`pacman -Rc software-name`

### Ayuda
* Comando para obtener lista de opciones con pacman:

`pacman -help`

* Comando para obtener lista de opciones más especifico:

`pacman -S -help`

* Comando para obtener manual de pacman:

`pacman man`

