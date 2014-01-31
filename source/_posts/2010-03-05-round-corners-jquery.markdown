---
author: admin
comments: true
date: 2010-03-05 09:34:34+00:00
layout: post
slug: round-corners-jquery
title: Закругленные углы jQuery
wordpress_id: 380
categories:
- CSS &amp; HTML
- jQuery
- программирование
tags:
- jQuery
- закругленные углы
---

[![закругленные углы jQuery](http://vredniy.ru/wp-content/uploads/2010/03/roundcorners-150x150.gif)](http://vredniy.ru/wp-content/uploads/2010/03/roundcorners.gif)
В сегодняшней заметке мы научимся закруглять углы у элементов с обычными углами по средствам jQuery. Пример очень простой, поэтому легко подойдет для начинающих. <!-- more -->
Нам понадобится всего одна картинка (вы можете ее увидеть справа отсюда)![закругленные углы](http://vredniy.ru/wp-content/uploads/2010/03/rndcorn.png) для всех четырех углов. Идея заключается в следующем. Мы создаем блок DIV с определенным классом, который и будем закруглять вставляя туда 4 пустых DIV'а с разными классами (углами), абсолютно позиционируя каждый. Картинка для всех углов, как я и сказал, будет одна и та же. Эту технику легко можно применять и без jQuery, просто добавляя 4 DIV'а в блок, который хотим закруглить, но динамичное закругление углов выглядит намного приятнее и проще. Работающий пример вы можете просмотреть, проядя по ссылке: [закругленные углы jQuery](/examples/roundcorners/) В конце заметки вас ждет полностью рабочий архив с полностью рабочим исходником =).
Первым делом создадим файл стилей CSS **style.css**
[cc lang="css"]
html, * {
	padding: 0;
	margin: 0;
}

div.rndblock {
	margin: 20px;
	width: 200px;
	height: auto;
	background:none repeat scroll 0 0 white;
	color:#6D6D6D;
	position:relative;	
	border:1px solid #E5E2E2;
	float: left;
}

.rndblock h2 {
	margin: 20px;
	text-decoration: underline;
}

.tl {
	background-position: top left;
	left: -1px;
	top: -1px;
}

.tr {
	background-position: top right;
	right: -1px !important;
	top: -1px;
}

.br {
	background-position: bottom right;
	bottom: -1px !important;
	right: -1px !important;
}

.bl {
	background-position: bottom left;
	bottom: -1px !important;
	left: -1px;
}

.tl, .tr, .bl, .br {
	background-image: url("../images/rndcorn.png");
	position: absolute;
	height: 10px;
	width: 10px;
	overflow: hidden;
}

button {
	background-color: lime;
	border: dashed 1ps yellow;
	padding: 3px;
	margin: 5px;
}
.rndblock p {
	padding: 10px;
}[/cc]
Теперь немного пояснений:



	
  * строки **6-15** декорируют этот самый закругляемый блок, в **8**й строке можно задать относительные единицы измерения, тогда блок будет "тянуться", **10**я и **13**я строки соответственно задают фон и границу

	
  * **22-44** мы задаем позиции фонового изображения для углов и позиционируем их относительно основного блока

	
  * **46-52** задаем общие для всех углов параметры: ведь картинка для углов у нас одна и та же и позиционирование для всех углов происходит абсолютное.

	
  * **остальные строки** лишь декор остальных элементов на странице


Теперь непосредственно разметка и jQuery код.
[cc lang="html"]


	
	
		


			

## Блок номер 1


			

Lorem ipsum dolor sit amet, consectetur adipiscing elit. 	Morbi elementum nisl nec ipsum vehicula hendrerit. 


			Round this corners

		


		
		


			

## Блок номер 2


			

Lorem ipsum dolor sit amet, consectetur adipiscing elit. 	Morbi elementum nisl nec ipsum vehicula hendrerit. 


			

Lorem ipsum dolor sit amet, consectetur adipiscing elit. 	Morbi elementum nisl nec ipsum vehicula hendrerit. 


			Round this corners

		


		
		


			

## Блок номер 3


			

Lorem ipsum dolor sit amet, consectetur adipiscing elit. 	Morbi elementum nisl nec ipsum vehicula hendrerit. 


			

Lorem ipsum dolor sit amet, consectetur adipiscing elit. 	Morbi elementum nisl nec ipsum vehicula hendrerit. 


			

Lorem ipsum dolor sit amet, consectetur adipiscing elit. 	Morbi elementum nisl nec ipsum vehicula hendrerit. 


			Round this corners

		


	

[/cc]



	
  * В 6 строке мы подключаем jQuery через Google

	
  * строки 9-12 создают переменную с 4 DIV'ами, которые и являются углами, которые мы и будем добавлять к блоку, который хотим закруглить

	
  * 13-15 добавляем углы к блоку после нажатия на кнопку, чтобы успеть заметить разницу.

	
  * 21-26, как видите в этих строках нет ни одного упоминания о углах, просто блок с абзацем, заголовком и кнопкой


Работающий исходник: [здесь](/files/roundcorners.zip)
Если убрать закругление углов по нажатию на кнопку и делать это при загрузке на страницу, то будет очень легко закруглять углы, не усложняя код. Единственный минус этого метода: люди с отключенным JavaScript не увидят наших стараний, хоть их и немного, но их тоже надо учитывать. 



