---
author: admin
comments: true
date: 2009-11-23 10:02:20+00:00
layout: post
slug: jquery-gallery

permalink: /2009/11/jquery-gallery

title: Галерея на jQuery
wordpress_id: 36
categories:
- CSS &amp; HTML
- jQuery
- программирование
tags:
- jQuery
- галерея
---

Часто, создавая сайты, приходится сталкиваться с всевозможными галереями, некоторые из них все еще представляют собой обычную таблицу с картинками. Но этим уже никого не удивить, я расскажу вам как внедрить на свой сайт именно галерею: сверху будет находиться картинка, чуть ниже ее описание и кнопки "Следующая" и "Предыдущая", завершать все будет миниатюрные копии остальных изображений, при наведении на которые, будут становиться менее прозрачными. ![fireworks](http://vredniy.ru/wp-content/uploads/2009/11/fireworks.jpg)<!-- more -->Это рабочий пример, который находится на сайте, посвященному Датским елкам и сопутствующим товарам, а именно фейерверкам. Здесь кнопок, отвечающих за перелистывание изображений нет, тут они ни к чему.

Для начала встраиваем в <head> файл jquery.js, который можно скачать с официального сайта [jQuery](http://jquery.com).
`` Дальше нужно скачать с официального сайта галереи 2 файла: плагин для jQuery и файл стилей для этой галереи. Их можно скачать [тут](http://devkick.com/lab/galleria/jquery.galleria.js) и [тут](http://devkick.com/lab/galleria/galleria.css). Затем прикрепляем их к проекту. ``
`
`
`

`
И немного измененный код css.
`
.caption{font-style:italic;color:#887;}
.demo{position:relative;margin-top:2em;}
.gallery_demo{width:400px;margin:0 auto;}
.gallery_demo li{width:68px;height:70px;border:none;margin: 15px; float: left;}
.gallery_demo li div{left:40px}
.gallery_demo li div .caption{font:italic 0.7em/1.4 georgia,serif;}

#main_image{margin:0 auto 60px auto;height:400px;width:400px;}
#main_image img{margin-bottom:10px;}

.nav{padding-top:15px;clear:both;font:80% 'helvetica neue',sans-serif;letter-spacing:3px;text-transform:uppercase;}

.info{text-align:left;width:700px;margin:30px auto;border-top:1px dotted #221;padding-top:30px;}
.info p{margin-top:1.6em;}`
Вот почти и все, осталось только подготовить изображения и написать небольшой html-код.
`





Предыдущий | Следующий






	
  * ![100 пудов](images/stories/boomz/100%20pudov.jpg)

	
  * ![Фейерверки](images/boomz/bolshoykush.jpg)

	
  * ![Фейерверки](images/boomz/boss.jpg)

	
  * ![Фейерверки](images/boomz/bylina.jpg)

	
  * ![Фейерверки](images/boomz/discoteka.jpg)




`
Вот и все. Если кому понадобятся кнопки "листания" то для этого достаточно, убрать в `

`
display:none.

display:none.
