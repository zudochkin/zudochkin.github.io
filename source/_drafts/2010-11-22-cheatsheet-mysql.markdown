---
author: admin
comments: true
date: 2010-11-22 17:07:16+00:00
layout: post
slug: cheatsheet-mysql

permalink: /2010/11/cheatsheet-mysql

title: Шпаргалка по MySQL
wordpress_id: 616
categories:
- db
- программирование
tags:
- db
- innodb
- mysql
- mysqldump
- mysqlshow
---

[![mysqldump](http://vredniy.ru/wp-content/uploads/2010/11/rustem_logoMysql2-150x150.gif)](http://vredniy.ru/wp-content/uploads/2010/11/rustem_logoMysql2.gif)Вам, наверное, также как и мне очень часто приходится сохранять бекапы. Эта заметка будет короткой и посвящена бекапу баз данных и таблиц и в конце немного может не всем известным фактам.
<!-- more -->
Создаем бекап:
[cc]mysqldump -u USER -pPASSWORD DATABASE > /path/to/dump/dump.sql[/cc]

Порой нужно сохранить лишь структуру базы данных без данных в ней содержащихся, для этого:
[cc]mysqldump --no-data -u USER -pPASSWORD DATABASE > /path/to/dump/schema.sql[/cc]

Дамп для одной или нескольких таблиц:
[cc]mysqldump -u USER -pPASSWORD DATABASE TABLE1 TABLE2 > /path/to/dump/tables.sql[/cc]

Чтобы бекап не занимал слишком много места (для некоторых это критично), сразу заархивируем его на лету:
[cc]mysqldump -u USER -pPASSWORD DATABASE | gzip > /path/to/dump/dump.sql.gz[/cc]

Если мы не хотим забыть когда же создавали этот бекап (не считая времени создания), воспользуемся следующей консольной командой:
[cc]mysqldump -u USER -pPASSWORD DATABASE | gzip > `date +/path/to/dump/backup.sql.%Y-%m-%d.%H-%M-%S.gz`[/cc]
Название бекапа будет примерно таким **backup.sql.2010-11-22.19-52-05.gz**

Восстановление из бекапа:
[cc]gunzip < /path/to/dump/dump.sql.gz | mysql -u USER -pPASSWORD DATABASE[/cc]
или 
[cc]zcat /path/to/dump/dump.sql.gz | mysql -u USER -pPASSWORD DATABASE[/cc]

Порой могут понадобится некоторые опции при создании бекапа
[cc]mysqldump -Q -c -e -u USER -pPASSWORD DATABASE > /path/to/dump/dump.sql[/cc]
где
**-Q** имена оборачиваются обратными кавычками
**-c** делает полную вставку, включая имена полей
**-e** расширенная вставка, дамп получается меньше и делается быстрее.

Для просмотра списка баз данных используем:
[cc]mysqlshow -u USER -p[/cc]
или всех таблиц заданной базы:
[cc]mysqlshow -u USER -p DATABASE[/cc]

И еще одна маленькая плюшка:
если в вашем проекте используются таблицы InnoDB, то вам следует добавлять параметр **--single-transaction**, что гарантирует целостность бекапа. Т.к. MyISAM не поддерживает транзакции, для них не актуально. 

Надеюсь, данная информация пригодится вам.

