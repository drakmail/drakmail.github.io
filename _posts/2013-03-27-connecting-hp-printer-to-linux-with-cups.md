---
layout: post
title: Подключение HP принтера с использованием SAMBA в Linux через CUPS
tags:
- linux
- cups
- принтеры
---

Принтеры HP меня приятно удивили. Всё оказалось просто и понятно. Ниже краткая заметка, чтобы не гуглить лишний раз команду просмотра списка SAMBA ресурсов.

Смотрим список принтеров на SAMBA-сервере:

```bash
$ smbclient -L 192.168.0.227 -U admin
```
    
Пример:
    
    $ smbclient -L 192.168.0.227 -U drakmail
    Enter drakmail's password: 
    Domain=[SERVDPSROOT] OS=[Windows Server 2003 3790 Service Pack 2] Server=[Windows Server 2003 5.2]
    
        Sharename       Type      Comment
    	---------       ----      -------
    	print$          Disk      Драйверы принтеров
        ....
    	HPLaserJ        Printer   HP LaserJet M1522 MFP Series PCL 6
        
    session request to 192.168.0.227 failed (Called name not present)
    session request to 192 failed (Called name not present)
    Domain=[SERVDPSROOT] OS=[Windows Server 2003 3790 Service Pack 2] Server=[Windows Server 2003 5.2]
    
    	Server               Comment
    	---------            -------
    
    	Workgroup            Master
    	---------            -------

Собственно, в CUPS дальше всё как обычно:

![Управление принтерами](/images/2013-03-27/1.png)

![Добавление принтера](/images/2013-03-27/2.png)

![Ввод данных нового принтера](/images/2013-03-27/3.png)

![Продолжение ввода данных нового принтера](/images/2013-03-27/4.png)

![Выбор .ppd файла для принтера](/images/2013-03-27/5.png)

![Указание стандратных опций печати](/images/2013-03-27/6.png)

![Информационная страница](/images/2013-03-27/7.png)

![Печать тестовой страницы](/images/2013-03-27/8.png)

![ИНформационная страница](/images/2013-03-27/9.png)
