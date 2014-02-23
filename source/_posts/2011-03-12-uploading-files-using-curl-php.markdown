---
author: admin
comments: true
date: 2011-03-12 20:23:49+00:00
layout: post
slug: uploading-files-using-curl-php

permalink: /2011/03/uploading-files-using-curl-php

description: "cURL - очень важный иструмент для PHP программистов, в этой заметке я расскажу вам как использовать некоторые функции этой интересной библиотеки."
keywords: "curl, php, загрузка файлов php,php"

title: Загрузка файлов с помощью cURL и PHP
wordpress_id: 666
categories:
- php
tags:
- curl
- php
---


cURL - очень важный иструмент для PHP программистов, в этой заметке я расскажу вам как использовать некоторые функции этой интересной библиотеки. Отправление данных, имитация отправления данных через форму - вот неполный список возможностей данного инструмента.<!-- more -->


``` php
<?php
// куда будем отправлять
$url = 'http://localhost/post.php';

// данные формы, наряду с отправляемым файлом
$postData['name'] = 'vredniy.ru';
$postData['image'] = '@c:/xampp/htdocs/test.jpg';

// инициализация cUrl
$ch = curl_init();

// сообщаем куда будет отправлять
curl_setopt($ch, CURLOPT_URL, $url);

// файлы и данные будет отправлены
curl_setopt($ch, CURLOPT_POSTFIELDS, $postData);

// передаем true или 1 если хотим ждать ответа после запроса
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

// включим отладочную информацию
curl_setopt($ch, CURLOPT_VERBOSE, true);

// отсылаем запрос
$response = curl_exec($ch);

// отладка: посмотрим на ответ сервера
echo $response;
?>
```


Отправить несколько файлов также легко, как и отправить один, а именно

``` php
<?php
$postData['image1'] = '@c:/xampp/htdocs/test1.jpg';
$postData['image2'] = '@c:/xampp/htdocs/test2.jpg';
?>
```


Все интересность заключается в предваряемом символе "собачки" ("@"), который указывает на локальный файл

Как всегда ничего сложного, надеюсь, вам заметка станет полезной. И удачи в нелегком программерском деле :).
