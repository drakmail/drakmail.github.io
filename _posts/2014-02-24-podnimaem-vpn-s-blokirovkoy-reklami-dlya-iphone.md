---
layout: post
title: Поднимаем VPN с блокировкой рекламы для iOS устройств
tags:
- ios
- vpn
---

## Зачем это всё?

Если вам надоела реклама, вечные всплывающие баннеры и тому подобное, но вы не хотите делать джейлбрек - то единственным адекватным вариантом является поднятие своего сервера с VPN и блокировщиком рекламы. Как это сделать будет описано ниже.

## Что нужно?

Любая недорогая VPS, расположенная недалеко от вашей страны (вполне неплохой вариант - VPS за $5 от DigitalOcean в Нидерландах), Debian/Ubuntu GNU/Linux (в этой статье будет использоваться 32-х битная Ubuntu 12.04 LTS), прямые руки.

## Приступим!

### VPS

Для начала требуется создать VPS и закинуть туда публичный ключ.

```
ssh-copy-id root@IP
```

После этого неплохо обновить систему и перезагрузиться.

```
ssh root@IP
apt-get update
apt-get upgrade -y
reboot
```

### VPN

Затем самое важное - установка VPN сервера. Это совсем не сложно. Если всё делать по мануалу - никаких проблем возникнуть не должно. Я настраивал по [этой статье](https://pleasefeedthegeek.wordpress.com/2012/04/21/l2tp-ubuntu-server-setup-for-ios-clients/).

### Privoxy

Для блокирования рекламы необходимо просто установить privoxy и скопировать ему правила блокирования. Установка privoxy ну очень простая, описана [здесь](http://kimondo.co.uk/raspberry-pi-as-an-adblock-server-for-ipad-iphone-android-and-anything-else-on-your-network/). Далее, чтобы скормить списки AdBlock в privoxy, осталось просто скачать и запустить на сервере [этот скрипт](https://raw.github.com/evaryont/bin/master/adblock-to-privoxy) (я заменил там некоторые списки на российские). Важно:

1. Не забыть в конфиге privoxy заменить listen_address на IP сервера в VPN.
2. Следить за тем, чтобы в правилах privoxy было не более 10 списков правил, иначе не запустится.

## Минусы

Из минусов стоит отметить:

1. iOS отключается от VPN, стоит ей перейти в режим бездействия. Очень напрягает. На текущий момент без джейлбрека способов решения этой проблемы не нашел.
2. Скорость загрузки страниц может немного упасть, решаемо за счёт размещения сервера ближе к стране проживания :)
3. Списки рекламы надо обновлять руками или написать дополнительный скрипт.

## Финал

В итоге, мы получаем надежный и достаточно удобный способ смотреть интернеты без рекламы, с некоторыми недостатками, но и не лишенный преимуществ.

