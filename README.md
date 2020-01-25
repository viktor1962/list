# list
Install commands
Arch Linux installation
Check network connection

0.0) loadkeys ru
0.1) setfont cyr-sun16

First of all, check the internet connection. I recommend you to use a wired connection. To check if your internet works, you need to ping to any server, for example, the Arch Linux website:

1) ping -c 3 archlinux.org


If you are not sure what interfaces are available, use

2) ip link

If you use a wired connection, it is usually picked up automatically. Wi-Fi requires some additional settings. For Wi-Fi run:

3) wifi-menu

You will see a window looking like this where you can choose the available networks:


Type the password and connect to your Wi-Fi network. However, I still recommend using the wired internet connection.
Partition

Next step in our Arch Linux installation guide, is to list the available partitions of our hard drives:

4) fdisk -l

Most probably, you will have only two hard drives: the USB drive with the Arch Linux installation ISO and your computer HDD. When you have several hard drives, look at their size and define which one you want to use for the Arch Linux installation.

If you already have a partition table, skip this step. In the case your hard drive is brand-new as in the case of a virtual machine or you want to re-partition your hard drive, run this command to create a new partition table:

5) cfdisk /dev/sda

Note! Back up all your data, because creating a new partition table will erase everything from a drive.

In the label type window, select GPT.
Avalible partition types
Select GPT partition

Legacy mode! If you do the legacy installation, choose dos partition type and do not create the UEFI partition.

Clicking the New button and create 3 partitions:

    /dev/sda1 # choose 512Mb of space (UEFI)
    /dev/sda2 # choose at least 10 GB of space (root)
    /dev/sda3 # choose all the left space (home)

Write the table to your hard drive and quit.

Now, list the partitions again:

6) lsblk

The /dev/sda disk should have three partitions. We need to format them.

The first partition is a UEFI one. It needs to be formatted in a FAT file system:

7) mkfs.fat -F32 /dev/sda1

Legacy mode! If you do the legacy installation, skip the step of UEFI formatting.

The other two partitions can be formatted in any Linux file system. I recommend using EXT4:

8) mkfs.ext4 /dev/sda2
9) mkfs.ext4 /dev/sda3

Next, mount the root and home partitions:

10) mount /dev/sda2 /mnt 
11) mkdir /mnt/home
12) mount /dev/sda3 /mnt/home

The root mounting point is the folder where the system will be installed. Check mounting points whether they were created successfully:

13) lsblk

lsblk
Install the system

Now, we start the installation process.

14) pacstrap -i /mnt base base-devel

When the system requests to choose the components to install, select all and yes. Wait some time until it completes.
Install the base of Arch Linux

Generate fstab file

Next step in this Arch Linux installation guide is to generate the fstab file:

15) genfstab -U -p /mnt >> /mnt/etc/fstab

To learn what -U and -p mean, type genfstab -help to see the description of these options.
Chroot to the installed system

Next, chroot (change root) to your account that is mounted to /mnt using the BASH environment:

16) arch-chroot /mnt /bin/bash

This way you change your live environment to the root environment of the installed Arch system. This way, you will be able to access the system as a root user. A bit later, you will have to add the regular user.
Set locale

To set the localization
# Добавим русскую локаль в систему
17) echo -e "en_US.UTF-8 UTF-8\nru_RU.UTF-8 UTF-8" >> /etc/locale.gen
 # Обновим текущую локаль системы
18) locale-gen
# Указываем язык системы
19) echo 'LANG="ru_RU.UTF-8"' > /etc/locale.conf

# Указываем keymap для console + прописываем шрифт
20) echo 'KEYMAP=ru' >> /etc/vconsole.conf
21) echo 'FONT=cyr-sun16' >> /etc/vconsole.conf

# Set the time zone

To set the time zone, type:

22) ln -sf /usr/share/zoneinfo/

and press the Tab key to see all the available options. In my case, I need to use Europe. Again, you can press the Tab key, and you will see all the available cities. I will use Stockholm. Save this link to  /etc/localtime. The final command will look as follows:

22.0) ln -sf /usr/share/zoneinfo/Europe/Stockholm /etc/localtime

Instead of Europe and Stockholm, you can select your region and time zone.
Set local time

# To set the time on your PC, run this command:

22,1) hwclock --systohc --utc

# Hostname is the computer’s name. Let’s name it archPC. Use the following command:

23) echo archPC > /etc/hostname

# You also need to add this name to the hosts file. Type:

24) nano /etc/hosts

# In the Nano editor, add this line at the end of the file:

127.0.1.1 localhost.localdomain archPC

# Arch Linux Installation: hostname


# If you use a static IP address, replace 127.0.1.1 with your static IP address given by the Internet provider. Press Ctrl + O to save, Ctrl + X to exit the editor.


# install the network manager:

25) pacman -S networkmanager

# Then enable it:

26) systemctl enable NetworkManager


Now, the system will be able to run a network manager at the system boot and connect to the Internet automatically. Remember, these settings work only for the wired internet connection.
# root password
#Next, set the root password. Type:

27) passwd

and type your password twice. Be attentive, as you will see nothing while typing.

# Install GRUB

Next, install the GRUB bootloader which is the vital component. Without it, your system will never boot. To install all the necessary packages, type the following command:

28) pacman -S grub efibootmgr

grub, efibootmgr installation in Arch Linux

# Installation of GRUB and EFI packagesWhen GRUB and EFI packges are installed, install the bootlader and generate its configuration files by running these commands one by one:

29) mkdir /boot/efi
30) mount /dev/sda1 /boot/efi
#lsblk # to check if everything is mounted correctly
31) grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi --recheck
32) grub-mkconfig -o /boot/grub/grub.cfg


# GRUB installation

Legacy mode! If you do the legacy installation, install the GRUB in this way:
# Старый способ:
pacman -S grub
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg

Basically, the minimal installation of Arch Linux is complete.
# Edit EFI bootloader

Legacy mode! If you do the legacy installation, skip this part.

However, we must make additional configurations with the bootloader. Create a BOOT directory:

33) sudo mkdir /boot/efi/EFI/BOOT

# After that, copy GRUB bootloader to this directory and give it a different name:

34) sudo cp /boot/efi/EFI/GRUB/grubx64.efi /boot/efi/EFI/BOOT/BOOTX64.EFI

# To be even safer, we can also create a startup script for EFI:

35) sudo nano /boot/efi/startup.nsh

# In Nano editor, add these lines:

bcfg boot add 1 fs0:\EFI\GRUB\grubx64.efi "My GRUB bootloader"

To complete editing, press Ctrl + O to save the changes and Ctrl + X to exit the editor. In the end, run exit to exit the chroot account.
# Umount&&Reboot

Next, unmount all mounted partitions and reboot the system:

36) umount -R /mnt
37) reboot

If you did everything correctly, after the reboot, you will see the GRUB welcome screen with Arch Linux installed.
# Заходим под : root
To continue, log in as a root user with a previously setup password.
Arch Linux root login
Log in as a root user
# Add user

After the login, create a user account. It’s not a good idea to constantly work from the root account. Type:

38) useradd -m -g users -G wheel -s /bin/bash USERNAME

Write your own name instead of username.


Also, create a password for the new user:

39) passwd USERNAME

# Next, enable sudo privileges for a newly created user:

39) EDITOR=nano visudo

Using the arrow keys, scroll down the screen and find the line:

#УДАЛИТЬ ЗНАК "#": # %wheel ALL=(ALL) ALL

Uncomment it, by removing the # sign.
Arch Linux Installation: %wheel ALL=(ALL) ALL
Uncomment %wheel ALL=(ALL) ALL

Press Ctrl + O to save and Ctrl + X to exit the editor.

Now, exit the system by running the command:

40) exit

# и войдите в систему как новый пользователь с именем пользователя и паролем, который вы создали.
and log in as a new user with the username and password you created.
# Install Audio, X window system, Xfce desktop, login manager

To make the new system usable, install audio, X Window System, Xfce desktop, and login manager. Type the following command:

41) pacman -S pulseaudio pulseaudio-alsa xorg xorg-xinit xorg-server xfce4 lightdm lightdm-gtk-greeter

If you install the system on VirtualBox, in the end, add virtualbox-guest-utils

When you press Enter, the system will offer to choose the components to install. Just press Enter twice to apply the default settings. After that, the system will request to choose the driver for the video card:
Arch Linux video driver options
# драйвер видео можно по дефолту: 1) libglvnd
После установки системы можно и Ваш драйвер потом уже установить!!!
If you have a discrete video graphic card, select the second option. When you use integrated Intel video card, select the first option. The utility will install many packages. Wait some time until it completes.

By the way, if you prefer the Plasma 5 desktop as I do, I showed how to install and configure Plasma 5 in Arch Linux here.
# Create xinit file

Xinit file allows to start an Xorg display server automatically. You need to create the xinitrc file with the command to launch Xfce desktop:
# Файл Xinit позволяет автоматически запустить сервер отображения Xorg. Вам нужно создать файл xinitrc с командой для запуска рабочего стола Xfce:

42) echo “exec startxfce4” > ~/.xinitrc

# After that, enable the login manager: После этого включите менеджер входа в систему:
43) systemctl enable lightdm
       ***Start GUI***
# To test whether your graphical environment works, run:
44) startx

If you install any other desktop besides Xfce, you will need to use another command and probably another login manager. Below, you will find the xinitrc command and the command to install the desktop:
Если вы устанавливаете любой другой рабочий стол, кроме Xfce, вам нужно будет использовать другую команду и, возможно, другой менеджер входа в систему. Ниже вы найдете команду xinitrc и команду для установки рабочего стола:

GNOME:

"exec gnome-session"
sudo pacman -S gnome

Cinnamon:

"exec cinnamon-session"
sudo pacman -S cinnamon

Mate:

"exec mate-session"
sudo pacman -S mate

Unity:

Unity installation is tricky - see the Arch Linux Wiki.

"exec unity"

Budgie:

"export XDG_CURRENT_DESKTOP=Budgie:GNOME"
"exec budgie-desktop"
sudo pacman -S budgie-desktop

Openbox:

"exec openbox-session"
sudo pacman -S openbox

i3:

"exec i3"
sudo pacman -S i

Awesome:

"exec awesome"
sudo pacman -S awesome

Deepin:

"exec startdde"
sudo pacman -S deepin

LXDE

"exec startlxde"
sudo pacman -S lxde


I would like to point out that I have not tested all these desktops. Some users reported that Plasma 5, for example, doesn’t start with lightDM. So, you need to use SDDM with Plasma 5. For the full list of Login Managers look at Arch Linux Wiki.
Start GUI




