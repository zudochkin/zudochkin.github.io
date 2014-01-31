---
author: admin
comments: true
date: 2009-11-27 08:57:31+00:00
layout: post
slug: drop-down-menu-html-%d0%b8-css
title: Выпадающее меню на HTML и CSS
wordpress_id: 68
categories:
- CSS &amp; HTML
tags:
- css
- html
- выпадающее меню
- меню
---

Привет, сегодня я расскажу вас как сделать несложное, но от этого не менее красивое, раскрывающееся меню на "чистом" HTML и CSS, без усложнения кода скриптами и прочей, утяжеляющей страницу мутью. Применяется это меню может где угодно: и на личных страничках, и на корпоративных сайтах со сложной иерархической структурой. Для начала нам нужно создать HTML-файл, который будет отвечать за пункты меню. Поехали.<!-- more -->
`







	
  * Статьи

	
    * ... о дизайне

	
    * ... о программировании

	
    * ... обо всем на свете




	
  * Контакты

	
  * Категории

	
    * CSS

	
    * PHP и mySQL

	
    * JavaScript и jQuery








`
Сейчас этот список выглядит вот так

[caption id="attachment_71" align="alignnone" width="235" caption="выпадающее меню"][![выпадающее меню](http://vredniy.ru/wp-content/uploads/2009/11/menu.gif)](http://vredniy.ru/wp-content/uploads/2009/11/menu.gif)[/caption]
Для того, чтобы изменить его внешний вид, напишем несколько строк CSS.
`
#mn ul {
list-style: none; /*убираем маркер у элементов списка*/
width: 190px; /*задаем ширину вертикального меню*/
}
#mn ul li {
position: relative; /*для того, чтобы выпадающий блок мы
могли
позиционировать абсолютно*/
}
#mn li ul {
position: absolute;/*вот для этого*/
left:100px;/*отступ слева появляющегося блока*/
top:5px;
display: none;/*сначала невидим*/
}
#mn ul li a {
display: block;
}
/*и самое главное, при наведении показать*/
#mn li:hover ul {
display: block;
}
/*танцы с бубном для ие*/
* html #mn ul li { float: left; height: 1%; }
* html #mn ul li a { height: 1%; }`
Вот что у нас получилось: [Простое выпадающее меню](/examples/simplemenu/index.html)
Остается самое интересное, декорирование этого самого меню, тут полет фантазии не знает предела, поэтому оставляю вам место для творчества. То, что в конечном итоге получилось у меня, можете лицезреть на картинке снизу. Удачи.

[caption id="attachment_77" align="aligncenter" width="279" caption="выпадающее меню (последняя версия)"][![выпадающее меню (последняя версия)](http://vredniy.ru/wp-content/uploads/2009/11/lastversion.jpg)](http://vredniy.ru/wp-content/uploads/2009/11/lastversion.jpg)[/caption]
