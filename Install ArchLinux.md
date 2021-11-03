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
