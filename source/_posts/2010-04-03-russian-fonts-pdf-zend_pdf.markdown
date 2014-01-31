---
author: admin
comments: true
date: 2010-04-03 15:20:30+00:00
layout: post
slug: russian-fonts-pdf-zend_pdf

permalink: /2010/04/russian-fonts-pdf-zend_pdf

title: Русские шрифты в Pdf (Zend_Pdf)
wordpress_id: 435
categories:
- php
- Zend Framework
- программирование
tags:
- pdf
- Zend Framework
- Zend_Pdf
- кодировка
---

[![zend_pdf и русский текст](http://vredniy.ru/wp-content/uploads/2010/04/zend_pdf-150x150.gif)](http://vredniy.ru/wp-content/uploads/2010/04/zend_pdf.gif)
Всем доброго времени суток. Вчера стояла передо мной задача экспорта данных, с использованием Zend Framework дело оказалось не очень сложным, благо что документация по Zend_Pdf хватает с лихвой, да и примеров в сети можно найти очень много. Но проблемы с кодировкой нигде не рассматриваются. Решил я этот вопрос методом перебора...<!-- more -->
Перебирал я шрифты из папки fonts (Windows 7) и отрисовывал каждым русский текст. Что из этого вышло вы можете посмотреть на картинке. Исходный код представлен ниже:
[cc lang="php"]
newPage(Zend_Pdf_Page::SIZE_A4);
$path = 'C:\Windows\Fonts';
$fonts = opendir($path);
$i = 0;
$j = 0;
while($a = readdir($fonts)) {
    if (('.' == $a) or ('..' == $a)) continue;
    if (preg_match('@(.*)\.(ttf)$@', $a)) {
        //echo $a . '

* * *

';
        if($j++ > 30) break;
        $page->setFont(Zend_Pdf_Font::fontWithPath($path .
                "\\" . $a), 12);
        $page->drawText('По-русски ' . $a, $j * 2, $i, 'utf-8');
        $i+=15;
    }
}
$pdf->pages[] = $page;
//$pdf->save('../public/users/1.pdf');
header('Content-type: application/pdf');
echo $pdf->render();
?>
[/cc]
В цикле (8-18 строки) проверяется каждый шрифт из папки шрифтов на соответствие TrueType шрифту (*.ttf), делается это не более 30 раз, мне показалось это достаточным, чтобы отыскать хотя бы один подходящий шрифт. Если убрать комментарии с 20 строки, то файл сохранится на жестком диске, иначе будет появляться стандартное окно "Сохранить/Открыть"
Из списка можно выбрать наиболее подходящий именно вам шрифт и использовать его в своих проектах.
