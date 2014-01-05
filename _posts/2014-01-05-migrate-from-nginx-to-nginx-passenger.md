---
layout: post
title: Миграция с обычного nginx на passenger в debian 7
tags:
- nginx
- passenger
- ruby
- python
---

## Дано

Настроенный и работающий сервер с nginx, который сейчас проксирует апач и пачку thin'ов с uwsgi'ми. Nginx обычный, из репозитория, конфиг тоже вполне обычный.

**Задача:** избавиться от кучи thin'ов, сделать процесс деплоя гораздо проще и быстрее.

## Подготовка

Как всегда, первым делом стоит развернуть вагрант (как это делать - см. [предыдущий пост про redmine](http://drakmail.ru/2013/11/24/migrate-from-chilliproject-to-redmine.html)) и обновить систему:

```bash
$ vagrant up
$ vagrant ssh
$ sudo apt-get update
$ sudo apt-get upgrade
```

Далее, поставим nginx из репозитория и проверим, работает ли тестовый сайт (хотя бы так):

```bash
$ sudo apt-get install nginx
$ sudo /etc/init.d/nginx restart
$ curl 127.0.0.1
```

Если в консоль вывелось Welcome to nginx - то всё ок.

## Собственно, сама смена  nginx на nginx-passenger

Собственно, самая важная часть - замена обычного nginx на хитрый nginx-passenger.

Пробуем делать всё по [мануалу](http://blog.phusion.nl/2013/09/11/debian-and-ubuntu-packages-for-phusion-passenger/).

Шаг 1: Добавление репозитория

```bash
$ gpg --keyserver keyserver.ubuntu.com --recv-keys 561F9B9CAC40B2F7
$ gpg --armor --export 561F9B9CAC40B2F7 | sudo apt-key add -
$ sudo apt-get install apt-transport-https
$ sudo su -c "echo 'deb https://oss-binaries.phusionpassenger.com/apt/passenger wheezy main' > /etc/apt/sources.list.d/passenger.list"
$ sudo chown root: /etc/apt/sources.list.d/passenger.list
$ sudo chmod 600 /etc/apt/sources.list.d/passenger.list
```

Шаг 2: Установка пакетов

```bash
$ sudo apt-get update
$ sudo apt-get install nginx-full passenger
$ vim /etc/nginx/nginx.conf
# тут раскомментируем 2 строчки с passenger:
# passenger_root /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini;
# passenger_ruby /usr/bin/ruby;
$ sudo /etc/init.d/nginx restart
$ curl 127.0.0.1 # всё по прежнему работает :)
```

Шаг 3: Пробуем что-то настроить

От оставшегося с прошлого раза redmine можно снова получить пользу - натравить для проверки "в боевых условиях" passenger на него:

```bash
sudo vim /etc/nginx/sites-enabled/redmine
```

Туда кидаем стандартный конфиг:

```
server {
	listen 80;
	server_name redmine;
	root /var/www/delta-zet.com/data/redmine/public;   # <--- be sure to point to 'public'!
	passenger_enabled on;
	allow all;
	passenger_min_instances 1;
	client_max_body_size      15m;
}
```

Перезапускаем...

```bash
$ sudo /etc/init.d/nginx restart
```

Пробуем получить страничку...

```bash
$ curl -H 'Host: redmine' 127.0.0.1
```

Шаг 4: Подводные камни

Куда же без них. После тестового прогона redmine (опять же из прошлого поста :)) оказалось, что почему-то сегфолтится гем json'а. Не сложно догадаться, что redmine у нас был настроен через rvm, а passenger использует системный ruby.

Поверхностно гуглим, понимаем, что должно спасти [явное указание пути к нужному бинарнику `ruby`](http://www.modrails.com/documentation/Users%20guide%20Nginx.html#PassengerRuby) - просто добавляем одну строчку в конфиг:

```
passenger_ruby /var/www/delta-zet.com/data/.rvm/rubies/ruby-2.0.0-p353/bin/ruby;
```

Пробуем... и это срабатывает! Всё готово для повторения процедуры на продакшене - как видно, это совсем не страшно :)
