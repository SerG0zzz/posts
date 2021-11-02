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

1 раздел 31М type - BIOS boot

2 разддел 300-500M type - EFI System (для ядер)

3 раздел подкачки  type - linux swap (для слабых компьютеров, но можно сделать просто файл подкачки как в windows на основном разделе с данными)

4 раздел с данными
