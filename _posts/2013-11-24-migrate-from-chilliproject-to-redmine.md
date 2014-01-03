---
layout: post
title: Миграция с ChilliProject на Redmine
tags:
- redmine
- chilliproject
- debian
- postgresql
- linux
---

## Вступление

Всё началось примерно год назад. Redmine в то время находился в периоде стагнации в связи с тем, что часть разработчиков устроила драму и ушла пилить форк под названием ChilliProject. В тот момент я принял решение, которое и стало поводом написать этот пост - я решил перейти с Redmine на ChilliProject.

Сейчас ситуация повторилась, только ChilliProject стал стагнировать, в то время как Redmine'овцы уже перешли на 3-и рельсы, есть планы по переходу на 4-е, большинство плагинов пилятся тоже с оглядкой только лишь на Redmine. Поэтому настало время для второго судьбоносного решения - перехода с ChilliProject обратно на Redmine.

## Процесс

Собственно, с лирической частью закончено. Теперь о самом процессе. Так как ChilliProject у нас работает на продакшене, с достаточно большой базой тикетов, страниц wiki, файлов и документов, делать перенос сразу на продакшене несколько страшно. Поэтому проще всего использовать vagrant для миграции на локальной машине, а после - проверить, всё ли смигрировало нормально и просто закинуть базу и файлы на продакшен.

### Шаг 1. Инициализация vagrant

Так как на продакшене используется debian, образ vagrant'а лучше всего использовать тоже debian'овский. Это позволит избежать различных косяков при перекидывании. Добавление 64-х битного debian 7.2 делается очень просто:

```
vagrant box add debian7.2 https://dl.dropboxusercontent.com/u/197673519/debian-7.2.0.box
```

Далее требуется инициализировать новую виртуалку с debian:

```
vagrant init debian7.2
```

И профорвардить 3000-й порт для тестирования всего этого. Для этого надо раскомментировать и чуть изменить строчку Vagrantfile:

```
# config.vm.network :forwarded_port, guest: 80, host: 8080
```

заменить на

```
config.vm.network :forwarded_port, guest: 3000, host: 3030
```

### Шаг 2. Воссоздание среды

Ничего сложного, просто ставим Postgres, rvm (по хорошему, надо бы сменить его на chruby, но лень), imagemagick и bundler:

```
vagrant up
vagrant ssh
sudo su
dpkg-reconfigure locales
exit
sudo su -l # refresh locale settings
apt-get update && apt-get upgrade
apt-get install postgresql-9.1 postgresql-server-dev-9.1 libmagickwand-dev vim
adduser --force-badname --home /var/www/delta-zet.com/data/ delta-zet.com # add user as on the primary server
su -l delta-zet.com
\curl -L https://get.rvm.io | bash -s stable
source ~/.bashrc
source ~/.bash_profile
rvm autolibs disable # to disable sudo
rvm install 2.0.0
gem install bundler
```

### Шаг 3. Установка Redmine

Тут вообще всё просто:

```
sudo su -l postgres
psql
create user "delta-zet.com" with password '123456';
create database redmine with owner "delta-zet.com";
\q
exit
su -l delta-zet.com
wget -O redmine.tar.gz http://www.redmine.org/releases/redmine-2.4.1.tar.gz
tar -xvf redmine.tar.gz
ln -s redmine-2.4.1/ redmine
cd redmine
cp config/database.yml.example config/database.yml
vim config/database.yml
bundle install --without development test
rake generate_secret_token
RAILS_ENV=production rake db:migrate
RAILS_ENV=production REDMINE_LANG=ru rake redmine:load_default_data
mkdir -p tmp tmp/pdf public/plugin_assets
chown -R delta-zet.com:delta-zet.com files log tmp public/plugin_assets
chmod -R 755 files log tmp public/plugin_assets
ruby script/rails server webrick -e production
```

После этого можно убедиться, что всё работает. Теперь начинается самая интересная часть - перенос.

### Шаг 4. Миграция

Сначала задампим БД ChilliProject'а:

```
pg_dump redmine > redmine.sql
```

И импортируем его в vagrant, предварительно очистив тестовую таблицу:

```
sudo su -l postgres
psql
drop database redmine;
create database redmine with owner "delta-zet.com";
\q
exit
sudo su -l delta-zet.com
psql redmine < redmine.sql
```

Правки структуры...

```
psql redmine
ALTER TABLE journals RENAME COLUMN created_at TO created_on;
ALTER TABLE journals RENAME COLUMN activity_type TO journalized_type;
ALTER TABLE journals RENAME COLUMN journaled_id TO journalized_id;
UPDATE journals SET journalized_type='Issue' WHERE journalized_type='issues';
ALTER TABLE wiki_contents ADD comments VARCHAR(250) NULL;
ALTER TABLE journals RENAME COLUMN changes TO changes_chilli;
ALTER TABLE journals DROP COLUMN type;
ALTER TABLE journals DROP COLUMN version;
```

Затем запускаем миграции...

```
RAILS_ENV=production rake db:migrate
```

Потом просто затариваем директорию, переносим files, прописываем заново workflow'ы, так как они почему-то не смигрировали и радуемся :)

## Заключение

TODO: Добавить заключение :D
