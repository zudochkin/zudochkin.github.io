---
author: admin
comments: true
date: 2011-03-17 20:09:11+00:00
layout: post
slug: jpeg2text-ascii-art

permalink: /2011/03/jpeg2text-ascii-art

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

[![ascii-art-php](http://vredniy.ru/wp-content/uploads/2011/03/main-150x150.png)](http://vredniy.ru/wp-content/uploads/2011/03/main.png)Привет, разработчики или просто тем, кто зашел на мой блог. Сегодня не напрягающая мозги заметка, которая создана лишь с целью позабавиться. Из картинки мы будем делать цветной ASCII ART. Пригодится для поздравлений или просто шутки ради.<!-- more -->
Нам понадобится любая картинка в формате jpeg (кто хочет поддержку других форматов, может легко это доделать) и немного кода на php + расширение gd (хотя сейчас сложно найти хостинг без поддержки этой замечательной графической библиотеки).

Алгоритм до безумия прост:
 - Ресайзим картинку до размеров по максимальному измерению до 100 пикселей (падает точность, но скорость заметно возрастает).

 - В цикле по всем пиксилям полученного изображения выводим любой символ (я предпочел звездочку "*"), закрашивая его соответствующим цветом исходного изображения.

Еще допишем стили, чтобы наш ASCII ART красиво отображался (проверял в FF и Chrome):
[cc lang="css"]
	div#picture span {
		display: block;
		float: left;
		width: 5px;
		height: 5px;
	}
[/cc]

И исходный код трансформатора на php:

[cc lang="php"]
$y) {
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

$message = '

';
for($h = 0; $h < $ny; $h++) {
	$message .= '

';
	for($w = 0; $w < ceil($nx); $w++) {
		$rgb = imagecolorat($ni, $w, $h);
		$r = ($rgb >> 16) & 0xFF;
		$g = ($rgb >> 8) & 0xFF;
		$b = $rgb & 0xFF;
		$message .= sprintf('*', $r, $g, $b);
	}
}
$message .= '

';
echo $message;
?>
[/cc]
Первое изображение в заметке - это результат трансформации изображения чуть ниже. Удачи вам и кода без ошибок :) [![ascii-art-php](http://vredniy.ru/wp-content/uploads/2011/03/homer6-300x225.jpg)](http://vredniy.ru/wp-content/uploads/2011/03/homer6.jpg)

