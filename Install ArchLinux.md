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
```
