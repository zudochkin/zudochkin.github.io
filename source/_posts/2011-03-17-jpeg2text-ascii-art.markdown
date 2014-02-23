---
author: admin
comments: true
date: 2011-03-17 20:09:11+00:00
layout: post
slug: jpeg2text-ascii-art

permalink: /2011/03/jpeg2text-ascii-art

description: "Сегодня не напрягающая мозги заметка, которая создана лишь с целью позабавиться. Из картинки мы будем делать цветной ASCII ART. Пригодится для поздравлений или просто шутки ради."

keywords: "ascii art, ascii art php, jpeg2txt, jpeg2text, jpeg2text php,ascii-art,gd,php,шутки юмора"

title: Jpeg2text или немного баловства
wordpress_id: 671
categories:
- php
- шутки юмора
tags:
- ascii-art
- gd
- php
---



Привет, разработчики или просто тем, кто зашел на мой блог. Сегодня не напрягающая мозги заметка, которая создана лишь с целью позабавиться. Из картинки мы будем делать цветной ASCII ART. Пригодится для поздравлений или просто шутки ради.<!-- more -->

[{% img image /images/posts/2011-03-jpeg2text-ascii-art/main-150x150.png %}](/images/posts/2011-03-jpeg2text-ascii-art/main.png)

Нам понадобится любая картинка в формате jpeg (кто хочет поддержку других форматов, может легко это доделать) и немного кода на php + расширение gd (хотя сейчас сложно найти хостинг без поддержки этой замечательной графической библиотеки).

Алгоритм до безумия прост:
 - Ресайзим картинку до размеров по максимальному измерению до 100 пикселей (падает точность, но скорость заметно возрастает).

 - В цикле по всем пиксилям полученного изображения выводим любой символ (я предпочел звездочку "*"), закрашивая его соответствующим цветом исходного изображения.

Еще допишем стили, чтобы наш ASCII ART красиво отображался (проверял в FF и Chrome):

``` css
div#picture span {
  display: block;
  float: left;
  width: 5px;
  height: 5px;
}
```


И исходный код трансформатора на php:


``` php
<?php
/**
 * @author vredniy.ru
 */
$file = "homer6.jpg";

// максимальное измерение (прибавляет скорости)
$mSize = 100;

$im = imagecreatefromjpeg($file);
list($x, $y) = getimagesize($file);
$ratio = $x/$y;
if ($x > $y) {
   $nx = $mSize;
   $ny = round($mSize / $ratio);
} else {
   $ratio = 1/$ratio;
   $ny = $mSize;
   $nx = round($mSize / $ratio);
}
$ni = imagecreatetruecolor($nx, $ny);
// создаем пропорционально уменьшенную копию
imagecopyresampled($ni, $im, 0, 0, 0, 0, $nx, $ny, $x, $y);

$message = '<div id="picture">';
for($h = 0; $h < $ny; $h++) {
   $message .= '<div style="clear:both"></div>';
   for($w = 0; $w < ceil($nx); $w++) {
      $rgb = imagecolorat($ni, $w, $h);
      $r = ($rgb >> 16) & 0xFF;
      $g = ($rgb >> 8) & 0xFF;
      $b = $rgb & 0xFF;
      $message .= sprintf('<span style="color: rgb(%d, %d, %d)">*</span>', $r, $g, $b);
   }
}
$message .= '</div>';
echo $message;
?>
```

Первое изображение в заметке - это результат трансформации изображения чуть ниже. Удачи вам и кода без ошибок :) [{% img image /images/posts/2011-03-jpeg2text-ascii-art/homer6-300x225.jpg %}](/images/posts/2011-03-jpeg2text-ascii-art/homer6.jpg)

