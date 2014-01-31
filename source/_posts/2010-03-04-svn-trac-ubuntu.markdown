---
author: admin
comments: true
date: 2010-03-04 20:16:18+00:00
layout: post
slug: svn-trac-ubuntu
title: Работа над большими проектами (svn & trac)
wordpress_id: 372
categories:
- Linux
- php
- программирование
tags:
- svn
- trac
- ubuntu
- система контроля за версиями
---

[![система контроля за версиями](http://vredniy.ru/wp-content/uploads/2010/03/trac-150x150.jpg)](http://vredniy.ru/wp-content/uploads/2010/03/trac.jpg)
Очень часто, когда проект выходит за рамки нескольких файлов и работа над ним ведется более чем одним программистом, приходится применять систему контроля за версиями, чтобы все изменения, вносимые в проект, были видны каждому, кто над этим проектом трудится. И чтобы имелась возможность откатиться до нужного состояния. Процесс установки взят с Хабра.
Первым делом нужна машина под управлением *nix системы, у меня работает Ubuntu на локальной машине через виртуальную машину, есть еще и VDS, в которой тоже предоставлен root доступ, без которого не установишь систему контроля за версиями. Но обеих машинах был настроен стандартный набор LAMP<!-- more -->, если у вас он еще не установлен, то вы можете прочитать как его установить [тут](/2010/02/setup-lamp-linux-apache-mysql-php/).
А теперь по порядку.
1. Устанавливаем необходимые пакеты
**apt-get install trac libapache2-mod-python libapache2-svn subversion python-subversion**
2. Включаем модуль Python'а
**a2enmod mod_python**
Теперь настроим svn и trac.



	
  1. Создаем группу пользователей, под которой будет работать svn:
**groupadd svn**

	
  2. Добавляем пользователя, под которым работаем в Ubuntu (себя):
**usermod -a -G svn свой_логин**

	
  3. Добавляем Апач в эту же группу:
**usermod -a -G svn www-data**

	
  4. Создаем папку, где будут храниться будущие исходники:
**mkdir /var/svn**

	
  5. Создаем хранилище:
**svnadmin create /var/svn**

	
  6. Меняем права доступа и разрешаем запись и группе, и владельцу:
**chown -R www-data:svn /var/svn  
chmod -R g+ws /var/svn**

	
  7. Создаем пароль к хранилищу, который мы будем вводить, чтобы войти в репозиторий:
**htpasswd -c -m /etc/apache2/svn.htpasswd имя_пользователя**

	
  8. Создаем правило для Апача для доступа к svn-хранилищу:
**nano /etc/apache2/conf.d/svn**
и записываем туда
[cc lang="bash"]

DAV svn
SVNPath /var/svn
AuthType Basic
AuthName "SVN Repo"
AuthUserFile /etc/apache2/svn.htpasswd
Require valid-user

[/cc]



SVN установлен, осталось только перезагрузить Апач **/etc/init.d/apache2 restart** И можно заходить по адресу **http://localhost/svn**
Перейдем к установке trac'а. Здесь тоже не будет ничего сложного



	
  1. Создаем папку:
**mkdir /var/trac**

	
  2. Создаем окружение для работы Trac'а с SVN:
**trac-admin /var/trac initenv** Там ответим на пару несложных вопросов, одним из которых будет с какой системой контроля версиями мы будем работать, отвечаем смело **SVN**

	
  3. Меняем права доступа:
**chown -R www-data:svn /var/trac  
chmod -R g+ws /var/trac**

	
  4. Создаем пароль для Trac'а, при вводе которого Апач нас будет пускать в хранилище:
**htpasswd -c -m /etc/apache2/trac.htpasswd имя_пользователя**

	
  5. Создадим правило для Апача для доступа к Trac'у:
**nano /etc/apache2/conf.d/trac**и допишем туда следующее 
[cc lang="bash"]

AuthType Basic
AuthName "Projects"
AuthUserFile /etc/apache2/trac.htpasswd
Require valid-user



SetHandler mod_python
PythonInterpreter main_interpreter
PythonHandler trac.web.modpython_frontend
PythonOption TracEnv /var/trac
PythonOption TracUriRoot /trac

[/cc]



Trac установлен и увидеть его мы можем по адресу: **http://localhost/trac**

Вот теперь намного проще будет создавать большие проекты и будет намного меньше путаницы.
Все популярные IDE позволяют работать с SVN, поэтому у вас не должно возникнуть проблем.



