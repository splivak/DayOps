---
title: "NTP"
date: 2024-07-08T15:32:31+03:00
draft: false
tags: ["service"]
categories: ["linux"]
cover:
    image: /img/linux.png
---

NTP — протокол для синхронизации системного времени с эталонным, предоставляемым специальными серверами. В статье рассмотрим, как настроить протокол NTP в различных ОС и на некоторых устройствах, а начнем с большой инструкции по настройке NTP Server Linux.

## Установка и настройка NTP сервера

```bash
sudo apt-get update
sudo apt-get upgrade

sudo apt-get install ntp

sntp --version

sudo nano /etc/ntp.conf

sudo service ntp restart

sudo service ntp status

sudo ufw allow from any to any port 123 proto udp #если у нас включен фаервол, не забываем разешить принимать UDP пакеты по 123 порту 
```

## Настраиваеем синхронизацию NTP

```bash
sudo apt-get install ntpdate

sudo nano /etc/hosts 
#ntp1.ntp-servers.net ntp-server

sudo ntpdate ntp-server

sudo timedatectl set-ntp off

sudo apt-get install ntp

server NTP-server-host prefer iburst

sudo service ntp restart

ntpq -ps
```