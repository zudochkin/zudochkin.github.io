---
author: admin
comments: true
date: 2010-02-22 21:05:11+00:00
layout: post
slug: power-gallery-php-and-jquery
title: Продвинутая галерея на PHP и jQuery
wordpress_id: 288
categories:
- jQuery
- php
- программирование
tags:
- AJAX
- css
- gd
- jQuery
- php
---

[![Галерея на PHP и jQuery](http://vredniy.ru/wp-content/uploads/2010/02/screenshot-150x150.jpg)](http://vredniy.ru/wp-content/uploads/2010/02/screenshot.jpg)

Сегодня мы с вами разработаем небольшую галерею, у которой будет подобие админки с возможностью добавления новых и удалением имеющихся фотографий. Самое интересное в этой галерее будет возможность задать размеры миниатюры в реальном времени, сразу наблюдая за изменениями. Код писался на скорую руку, поэтому ни о какой безопасности и юзабилити речь не идет. Но, если кому покажется интересным создания галереи с нуля, легко сможет добавить недостающий функционал и расширить уже имеющийся. 


Также вы можете ознакомиться с заметкой [Создание миниатюр на PHP](/2010/02/11/thumbnails-ph-gd/)


### Создание админки




Для начала создадим файл **config.php**, в котором будем хранить некоторые конфигурационные данные. В первую очередь это секретной слово, которое поможет нам обезопаситься от посторонних глаз. У меня галерея располагается в папке **http://localhost/resizilka**. Исходя из этого и будет организован конфиг.
[cc lang="php"]

[/cc]


**$galleryFolder** - папка на сервере, где будут располагаться изображения.


**$thumbsFolder** - папка, в которой будут храниться миниатюры.


**$host** - для вывода картинок в галерее.


**$secretKey** - своего рода пароль, при вводе которого мы попадаем в админку.


Теперь о админке, которая будет состоять из одного файла. Конечно, если бы их было больше, можно было упростить структуру.
В начале файла нам понадобится подключить конфигурационный файл и открыть сессию (главное, чтобы до открытия сессии ничего не выводилось, а то вылезет ошибка). Файл **admin.php**
[cc lang="php"]
< ?php require_once 'config.php';
session_start();
if ($_SESSION['secretkey'] != 'passed') {
    if ($_GET['secret'] != $secretKey) {
        die('access denied');
    } else {
        $_SESSION['secretkey'] = 'passed';
    }
}
?>
[/cc]


Для того, чтобы войти в админку нам нужно проследовать по адресу **http://localhost/resizilka/admin.php?secret=fsdfs432fxD** (в конце адреса ключ из конфигурационного файла)


Дальше вставим html-заголовки и подключим нужные стили и скрипты (помимо jQuery используется плагин Resizable). Скачать их и все файлы проекта, вы можете по ссылке в конце заметки.
[cc lang="html"]
< !DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
   "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

    
    
[/cc]


Так как у нас все располагается в одном файле, нам нужна проверка не загружен ли файл, если да, то создаем миниатюру, если нет, то выводим только форму для добавления новых и таблицу с крестиком для удаления имеющихся.
[cc lang="php"]
< ?php
if(!isset($_FILES['filename'])) {?>


## Загрузка файлов на сервер



    
    



## Имеющиеся фотки


< ?php
$images = scandir($thumbsFolder);
echo "

";
unset ($images[0]);
unset ($images[1]);
foreach ($images as $image) {
    echo "

![](\"")
\n";
    echo "
[x]($host/erase.php?ss=$image)
";
}
echo "";
?>
[/cc]


**unset**'ами мы исключаем вывод **.** и **..**


Таблица представляет из себя строки из картинки и крестика, при нажатии на который мы удаляем картинку, отдавая ее в руки скрипту **erase.php** Текст файла для удаления:
[cc lang="php"]
< ?php
require_once 'config.php';
unlink($thumbsFolder . '/' . $_GET['ss']);
header("Location: $host/admin.php");
?>
[/cc]


После удаления переходим на главную страницу админки. За это отвечает редирект.


Теперь ветка алгоритма, если мы что-то загрузили на сервер в файле **admin.php**
[cc lang="php"]
< ?php } else {
    echo $_FILES['filename']['type'];
    $picFilename = $_FILES['filename']['name'];
    if (copy($_FILES['filename']['tmp_name'],
            $galleryFolder . '/' . $picFilename)) {
        $picPath = $galleryFolder . '/' . $picFilename;


    $picData = getimagesize($picPath);
    $picWidth = $picData[0]; $picHeight = $picData[1];
    echo $picWidth . ' x ' . $picHeight;
?>
[/cc]


Т.к. галерея делалась для себя, здесь много debug-информации. Например, тут выводятся размеры картинки, которые мы будет использовать далее.


Сразу подготовим функцию для ресайзинга, которая в качестве параметров принимает картинку-источник и картинку-назначение, координаты левого верхнего угла и размеры. Это и будет наш файл **cutting.php**, к которому мы будем обращаться посредством AJAX-запроса с помощью jQuery.
[cc lang="php"]
< ?php
require_once 'config.php';
$src = $galleryFolder . '/' . $_GET['src'];
$dest = $thumbsFolder . '/' . time() . $_GET['src'];
$x = $_GET['x'];   $y = $_GET['y'];
$width = $_GET['width'];    $height = $_GET['height'];
imageCutting($src, $dest, $x, $y, $width, $height);
function imageCutting($src, $dest, $x, $y, $width, $height, $rgb = 0xffffff)
{
    if(!file_exists($src)) {
        echo "$ src not exist";
        return FALSE;
    }
    $size = getimagesize($src);
    if ($size === FALSE) {
        echo "cannot get image size";
        return FALSE;
    }

    $format = strtolower(substr($size['mime'], strpos($size['mime'], '/') + 1));
    $icFunc = 'imagecreatefrom' . $format;
    if (!function_exists($icFunc)) {
        echo "function does not exist";
        return FALSE;
    }

    $isrc = $icFunc($src);
    $idest = imagecreatetruecolor($width, $height);

    imagefill($idest, 0, 0, $rgb);
    imagecopyresampled($idest, $isrc, 0, 0, $x, $y, $width, $height, $width, $height);

    imagejpeg($idest, $dest, 80);

    imagedestroy($isrc);
    imagedestroy($idest);
    echo "all done   
**" . $dest . '**  
was created';
    return TRUE;
}
?>
[/cc]


Теперь перейдем к разметке и программированию на стороне клиента. Т.к. почти весь функционал у нас находится в одном файле, нам будет намного удобнее передавать значения некоторых переменных в JavaScript.


Продолжим кропотать над **index.php**. Добавим немного разметки и само изображение, чтобы видно было из чего мы хотим сделать миниатюру.
[cc lang="html"]



![](<?php echo $picPath; ?/>)



    


    


    


    


    


    


    


    








  




    **Позиция**
      

    x: 
     
    y: 
      

    Закончить
    





[/cc]


`id="resizeMe"` и все вложенные в него элементы отвечают за квадратик с рамкой и маркерами для ресайза. Добавим немного уличной магии в виде JavaScript.
[cc lang="javascript"]

[/cc]


Во второй строчке которого мы объявляем переменные, чтобы хранить в них координаты левого верхнего угла миниатюры. При нажатии на кнопку в 6 строке мы собираем параметры в одну строку и отправляем с помощью jQuery скрипту **cutting.php**, получая при этом какой-либо ответ без перезагрузки страницы. Если вы хотите, чтобы имелась возможность ресайзить квадратик, нужно задать отличные величины в строчках 20-23, раскомментировать строки 42-48 и в строках 14 и 15 передавать ширину и высоту соответственно.


### Главная страница галереи




Осталось только сделать вывод картинок, которые находятся в папке миниатюр (в **config.php** за эту папку отвечает переменная **$thumbsFolder**).


Это и будет наш **index.php**
[cc lang="php"]



< ?php
require_once 'config.php';
$images = scandir($thumbsFolder);
echo "

";
unset ($images[0]);
unset ($images[1]);
foreach ($images as $image) {
if (($i % 3 == 0) && ($i!=0)) echo "

";
if ($i == 0) echo "

";
echo "
![](\"")
\n";
$i++;
}
if ($i % 3 != 0) echo "";
?>


[/cc]


В этом файле мы всего лишь выводим наши миниатюры в таблице в три столбца.


Эта несложная в реализации галерея имеет право на существование, если ее довести до ума и продумать дизайн. Но это уже все на ваше усмотрение.




[Скачать исходники](/files/resizilka.rar)
