---
title: "Proxy сервер в качестве шлюза"
date: 2023-08-15T12:56:03+03:00
draft: false
tags: ["proxy"]
categories: ["network"]
cover:
    image: /img/tun2socks.png

---

Прокси-сервер, прокси — это промежуточный сервер между пользователем интернета и серверами, откуда запрашивается информация. По сути, прокси — это посредник, фильтр или шлюз, который стоит между человеком и огромными (и не всегда безопасными) данными в сети.

#### HTTP proxy

Даже с введением SOCKS5, HTTP и HTTPS остаются самыми популярными прокси. HTTP — это сокращение от Hypertext Transfer Protocol.

Основная цель использования HTTP-прокси — организовать работу браузеров и других программ, использующих протокол TCP. Другими словами, это программы, которые используют стандартные порты 80, 8080, 3128. Прокси HTTP и HTTPS работают довольно просто. Браузер отправляет запросы прокси-серверу на открытие определенного ресурса (URL). Затем сервер получает данные и отправляет их браузеру.

Разница между HTTP и HTTPS заключается в том, что первый является незащищенным прокси, а второй — защищенным. Прокси-серверы HTTPS, также называемые HTTP через TLS или SSL, используются для безопасных соединений, например, при отправке данных вашей кредитной карты и других конфиденциальных данных

#### SOCKS proxy

SOCKS — это сокращение от Socket Secure, сетевого протокола, который направляет сетевой трафик через брандмауэр, тем самым облегчая связь с серверами. SOCKS, в отличие от HTTP / HTTPS, не модерирует HTTP-заголовок — серверы передают данные через себя, ничего не меняя.

Сегодня SOCKS — это самый продвинутый протокол передачи данных, специально разработанный для программ, которые не поддерживают использование прокси напрямую. Они работают со стандартными портами 1080 и 1081. Прокси-сервер SOCKS эволюционировал от исходного SOCKS до SOCKS4 и последнего усовершенствования SOCKS5.

Прокси-серверы SOCKS4 поддерживают только TCP-соединения, в то время как более новые серверы SOCKS5 поддерживают UDP, TCP, авторизацию по логину и паролю, а также удаленный DNS-запрос. Для справки, SOCKS — полностью анонимный прокси.

#### tun2socks

Это открытый проект, написанный китайскими разработчиками на языке Golang и распространяющийся по лицензии GPL-3.0. Данный проект, позволяет поднять туннель, который заворачивает весь трафик через SOCKS-прокси.

Суть работы сервиса tun2socks заключается в том, что в ОС создается виртуальный интерфейс, который операционная система воспринимает как обычный сетевой интерфейс. Далее настроив маршрутизацию можно отправить весь трафик в туннель.

Более детально про работу, настройку описано в официальной документации данного проекта: [https://github.com/xjasonlyu/tun2socks/wiki](https://github.com/xjasonlyu/tun2socks/wiki)

Мы же опишем пример настройки сервиса tun2socks на ubuntu 22.04

![Схема](/img/tun2socks.png#center)

```bash
    apt update 
    apt upgrade 
    apt install golang-go gccgo-go bind9 
    
    cd /opt 
    
    # скачиваем нужный нам релиз сервиса с ресурса https://github.com/xjasonlyu/tun2socks/releases 
    
    wget https://github.com/xjasonlyu/tun2socks/releases/download/v2.5.0/tun2socks-linux-amd64.zip unzip tun2socks-linux-amd64.zip 
    chmod +x /opt/tun2socks-linux-amd64 
    
    # оборачиваем все это в сервис 
    
    nano /etc/systemd/system/tun2socks.service  
    
    [Unit] 
    Description=Tun2Socks gateway 
    After=network.target  
    
    [Service] 
    User=root 
    Type=idle  
    ExecStart=/usr/sbin/tun2socks-linux-amd64 -device tun://gatewaytun -proxy socks5://10.10.0.50:1080 & sleep 3; ip link set gatewaytun up Restart=on-failure 
    [Install] 
    WantedBy=multi-user.target 
    
    # настраиваем сеть nano /etc/netplan/$$$$$$$$$$$$$$.yaml 
    
    network:  
	    ethernets:  
		    ens160: 
			    optional: true 
			    adresses: [192.168.1.10/24] 
			    routes: - to: 10.10.0.50 via: 192.168.1.1 
		    gatewaytun: 
			    dhcp4: no dhcp-identifier: mac 
			    optional: true 
			    addresses: [10.10.10.10/24] 
			    nameservers: addresses: [8.8.8.8] 
			    routes: - to: 0.0.0.0/0 via: 10.10.10.10 
	    version: 2 
    
    # включаем маршрутизацию на сервере 
    
    nano /etc/sysctl.conf net.ipv4.ip_forward=1
    

```