---
author: admin
comments: true
date: 2010-05-29 07:24:51+00:00
layout: post
slug: zend-framework-project-creation-netbeans
title: Быстрое создание Zend Framework проекта
wordpress_id: 486
categories:
- Zend Framework
tags:
- IDE
- NetBeans
- Zend Framework
---

Сегодня я расскажу вам как быстро создать полностью рабочий проект с минимальным функционалом. Для этого нам потребуется: 1) Среда разработки NetBeans 6.9 Beta, в предыдущей версии не поддерживается Zend Framework 2) Сам Zend Framework, можно скачать с официального сайта, на текущей момент последней является версия 1.10.5; 3) И конечно же установленный веб-сервер, я использую xampp на Windows машине с установленной Windows 7.<!-- more -->
Я складываю все версии фреймворков в одно место, пусть это будет c:\Zend. Веб сервер располагается в папке c:\xampp. Куда установлен NetBeans не важно. Перво-наперво нужно добавить пути к php.exe и zf.bat в переменную Windows %PATH%. Для этого проследуем в Панель управления - Система - Дополнительные Параметры системы - снизу Переменные среды - (см. картинку чуть ниже) выбрать из нижнего списка Path и нажать изменить - добавляем в конце ";c:\xampp\php;C:\Zend\ZendFramework-1.10.5\bin" без кавычек. Перезагружаемся, чтобы изменения вступили в силу. 
[![zend framework проект](http://vredniy.ru/wp-content/uploads/2010/05/path-changer-150x150.png)](http://vredniy.ru/wp-content/uploads/2010/05/path-changer.png)
Теперь все готово, осталось только настроить среду разработки. Для этого выбираем из меню Сервис - Настройки, открываем вкладку PHP [![netbeans-settings](http://vredniy.ru/wp-content/uploads/2010/05/netbeans-settings-150x150.png)](http://vredniy.ru/wp-content/uploads/2010/05/netbeans-settings.png) В поле PHP 5 Interpreter нажимаем кнопку Search..., если все предыдущие шаги были проделаны правильно, то появится путь к файлу php.exe. Далее во вкладке Zend, которая появилась только в 6.9 версии, напротив строки Zend Script также выбираем Search... и NetBeans должен сам найти путь к zf.bat. Осталось теперь только нажать на чуть ниже расположенную кнопку Register Provider и после нескольких секунд ожидания вы увидите сообщение либо об ошибке, либо об успешном выполнении операции.
Вот сейчас начинается самое интересное: создания первого проекта Zend Framework (Файл - Создать проект...) [![netbeans-project-creation](http://vredniy.ru/wp-content/uploads/2010/05/netbeans-project-creation1-300x206.jpg)](http://vredniy.ru/wp-content/uploads/2010/05/netbeans-project-creation1.jpg)  
  


[![netbeans-zf](http://vredniy.ru/wp-content/uploads/2010/05/netbeans-zf-300x206.jpg)](http://vredniy.ru/wp-content/uploads/2010/05/netbeans-zf.jpg)  
  
[![zf-project](http://vredniy.ru/wp-content/uploads/2010/05/zf-project1-300x185.jpg)](http://vredniy.ru/wp-content/uploads/2010/05/zf-project1.jpg)  
  
[![netbeans-zf-project](http://vredniy.ru/wp-content/uploads/2010/05/netbeans-zf-project-300x197.jpg)](http://vredniy.ru/wp-content/uploads/2010/05/netbeans-zf-project.jpg) Наш проект создан, осталось лишь добавить папку Library из Zend Framework в php include_path, чтобы проделать это нужно нажать правой кнопкой на строке Include Path, которая находится в самом низу всех файлов проекта, выбрать Properties и добавить туда папку c:\Zend\ZendFramework( номер версии )\Library для того, чтобы работал Auto complete для всех классов из этого фреймворка. Спустя некоторое время, когда NetBeans просканирует все файлы фремворка, можно приступать к работе. 
В завершении мы создадим простой контроллер средствами NetBeans (см. картинки ниже) и включим поддержку слоев (layouts). 
[![zend-command](http://vredniy.ru/wp-content/uploads/2010/05/zend-command-300x293.jpg)](http://vredniy.ru/wp-content/uploads/2010/05/zend-command.jpg)
  
  

[![create-controller](http://vredniy.ru/wp-content/uploads/2010/05/create-controller-300x236.jpg)](http://vredniy.ru/wp-content/uploads/2010/05/create-controller.jpg)
Контроллер создан, как и созданы шаблоны для его отображения всего одной командой. Дело за слоями.
[![enable-layout](http://vredniy.ru/wp-content/uploads/2010/05/enable-layout-300x236.jpg)](http://vredniy.ru/wp-content/uploads/2010/05/enable-layout.jpg)
После каждой команды нажимаем кнопку Run. Слои созданы и прописаны к конфигурационном файле. 
Почему я выбираю NetBeans?






	
  * Подсветка синтаксиса, автодополнение


	
  * Форматирование исходных текстов


	
  * Удобное средство документирования кода


	
  * Встроенные средства дебагинга


	
  * Поддержка SVN


	
  * Разнообразные плагины, которые расширяют функционал и без того неплохой IDE





