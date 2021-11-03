---
title: Установка Arch Linux
description: Разворачивание на системе с UEFI BIOS
modified: 2021-11-03
tags: Linux
---
```bash
ping google.ru
```
```bash
ip a
```
## для wi-fi
```bash
rfkill unblock wifi
```
```bash
ip link set wlan0 up
```
```bash
iwctl
```
```bash
station wlan0 connect <имя точки> <пароль>
```
```bash
exit
```
```bash
ping google.com 
```
если не пингуется

```bash
nano /etc/resolv.conf
```
дописать в файл:

```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```

## разметка диска
вывести список разделов
```bash
fdisk -l
```
инициализировать диск в gpt
```bash
fdisk /dev/sda
```
g

w

создать разделы на диске
```bash
cfdisk /dev/sda
```

1) раздел 31М type - **BIOS boot**

2) раздел 300-500M type - **EFI System** (для ядер)

3) раздел подкачки  type - **linux swap** (для слабых компьютеров, но можно сделать просто файл подкачки как в windows на основном разделе с данными)

4) раздел с данными type - **linux filesystem**

записываем все созданные разделы

проверяем созданные разделы `fdisk -l`

## Форматирование разделов

1) раздел пока не трогаем

2) раздел форматирование в **FAT**

```bash
mkfs.vfat /dev/sda2
```
3) Если есть **swap** раздел
```bash
mkswap /dev/sda3
```
и подключение **swap**
```bash
swapon /dev/sda3
```
4 форматирование раздела с данными в **btrfs**
```bash
mkfs.btrfs /dev/sda4
```

## Монтирование разделов

монтирование раздела с данными
```bash
mount /dev/sda4 /mnt
```
создание папки для загрузчика
```bash
mkdir /mnt/boot
mkdir /mnt/boot/EFI
```
монтирование раздела с будущем загрузчиком
```bash
mount /dev/sda2 /mnt/boot/EFI
```

## Установка системы с помощью **pacstrap**

Установка нужного набора пакетов

**linux-zen** - самое свежее и быстрое ядро, более стабильное **linux**, и самое стабильное **linux-lts**, headers дописываются

соответственно выбранному ядру, те для lts будет **linux-lts-headers**

**linux-firmware** - набор драйверов-прошивок

**dosfstools btrfs-progs** - набор утилит для ФС

**amd-ucode** - микрокод для процессоров AMD, для intel - **intel-ucode iucode-tool**

**nano** - простой консольный редактор, для редактирования конфигов
```bash
pacstrap -i /mnt base base-devel linux-zen linux-zen-headers linux-firmware dosfstools btrfs-progs amd-ucode nano
```
генерирование файла конфигурации дисков в системе
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
теперь проверка того что сгенерировалось
```bash
cat /mnt/etc/fstab
```

## Переход в установленную систему и её настройка

```bash
arch-chroot /mnt
```
конфигурирование времени и даты
```bash
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc
```
русификация системы, выбрать нужные языки, путём уберания у нужных строк символа **#**
```bash
nano /etc/locale.gen
locale-gen
```
отредактировать файл, добавив параметр языка системы **LANG=ru_RU.UTF-8**
```bash
nano /etc/locale.conf
```
настройка консоли на родной язык
```bash
nano /etc/vconsole.conf
```
прописать в файл:
```bash
KEYMAP=ru
FONT=cyr-sun16
```
после задать имя компьютера в файле
```bash
nano /etc/hostname
```
добавление доменных имён компьютера
```bash
nano /etc/hosts
```
прописать туда:
```bash
127.0.0.1	localhost
::		localhost
127.0.0.1	<имя компьтера>.localdomain	<имя компьтера>
```
создание образа оперативной памяти для ядра, можно если одно ядро и так **mkinitcpio -P**
```bash
mkinitcpio -p linux-zen
```
установка пароля для root
```bash
passwd
```
## Скачивание загрузчика и его настройка

за одно и сетевые утилиты
```bash
pacman -Syu
pacman -S grub efibootmgr dhcpcd dhclient networkmanager
```
Установка загрузчика
```bash
grub-install /edv/sda
```
если не получилось, для EFI BIOS:
```bash
grub-install --boot-directory=/boot/EFI
```
конфигурирование загрузчика
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
и выход из chroot
```bash
exit
```
## Размонтирование разделов и перезагрузка
```bash
umount -R /mnt
reboot
```
## Добавление пользователей
```bash
nano /etc/sudoers
```
раскоментировать строку убрав **#** в строке **# %wheel ALL=(ALL) ALL**

создание пользователя
```bash
useradd -m -G wheel -s /bin/bash <имя пользователя с маленькой буквы>
```
пароль для пальзователя, обязательно отличный от **root**
```bash
passwd <имя пользователя>
exit
```
проверка root - доступа
```bash
sudo su
```

## Настройка сети
```bash
systemctl enable NetworkManager
reboot
```
проверка сети
```bash
ping google.com
```
если нет проволной сети
```bash
nmcli d wifi connect <имя точки длступа> password xxxxxxxx
```
## Уставнавливаем видеодрайвера

для amd
```bash
sudo pacman -Syu lib32-mesa vulkan-radeon lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader
```
для intel
```bash
sudo pacman -Syu lib32-mesa vulkan-intel lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader libva-media-driver xf86-video-intel
```
для Nvidia
```bash
sudo pacman -Syu nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader lib32-opencl-nvidia opencl-nvidia libxnvctrl
```
для Nvidia+Intel
```bash
sudo pacman -Syu nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader lib32-opencl-nvidia opencl-nvidia libxnvctrl
sudo pacman -Syu lib32-mesa vulkan-intel lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader libva-intel-driver xf86-video-intel
```

установим апплет networkmanager для графической оболочки
```bash
sudo pacman -S network-manager-applet
```
ещё перезагрузка
```bash
reboot
```

## Установка графической оболочки

для KDE
```bash
sudo pacman -S xorg xorg-server plasma plasma-wayland-session kde-applications sddm sddm-kcm packagekit-qt5
```
Включаем дисплей менеджер
```bash
systemctl enable sddm
```

для Gnome
```bash
pacman -S  xorg xorg-server gnome gnome-extra gdm
```
Включаем дисплей менеджер
```bash
systemctl enable gdm
```

для XFCE
```bash
pacman -S xorg xorg-server xfce4 xfce4-goodies lightdm lightdm-gtk-greeter
```
Включает дисплей менеджер
```bash
systemctl enable lightdm
```

для Сinnamon
```bash
pacman -S xorg xorg-server cinnamon
```
Включаем дисплей менеджер
```bash
systemctl enable gdm
```

для Deepin
```bash
pacman -S xorg xorg-server deepin deepin-extra lightdm lightdm-deepin-greeter
```
Включаем дисплей менеджер
```bash
systemctl enable lightdm
```

для Enlightenment
```bash
pacman -S xorg xorg-server enlightenment lightdm lightdm-gtk-greeter
```
Включаем дисплей менеджер
```bash
systemctl enable lightdm
```

для Mate
```bash
pacman -S xorg xorg-server mate mate-extra mate-panel mate-session-manager
```
Включаем дисплей менеджер
```bash
systemctl enable mdm
```

для LXDE
```bash
pacman -S xorg xorg-server lxde-common  lxsession openbox lxde lxdm
```
Включаем дисплей менеджер
```bash
systemctl enable lxdm
```
