---
author: admin
comments: true
date: 2010-03-13 13:22:10+00:00
layout: post
slug: jquery-for-beginners-part4

permalink: /2010/03/jquery-for-beginners-part4

title: 'jQuery для начинающих: углубляемся в реализацию'
wordpress_id: 411
categories:
- javascript
- jQuery
- программирование
tags:
- jQuery
- анимация
- туториал
---

### Понимание jQuery


Привет всем, кто хочет освоить данный замечательный JavaScript фреймворк. В этой заметке мы немного углубимся в реализацию jQuery. Узнаем как происходят те или иные события, как это реализовано в jQuery. Дальше будет интересно.<!-- more -->
Что происходит, когда мы вызываем 'jQuery'?

jQuery функция по себе очень простая

[cc lang="javascript"]
jQuery = function (selector, context) {
	return new jQuery.fn.init(selector, context);
}
[/cc]

Что мы имеем внутри? jQuery функция (объявленная как "обертка") возвращает инициализированный jQuery объект - экземпляр 'jQuery.fn.init' конструктора.

Полезно знать то, что когда мы вызываем функцию 'jQuery' мы фактически создаем полностью уникальный объект с определенным набором свойств. jQuery в этом плане очень умный, т.к. дает нам объект, который можно расценивать как массив. Каждый из элементов (все вместе их еще называют "коллекциями") вызывается числовым индексом, точно также как массив. И jQuery "приделывает" к объекту свойство 'length', как и следовало ожидать от массива. Это открывает перед нами безграничные возможности. К примеру, это значит, что мы можем пользоваться некоторой функциональностью 'Array.prototype'. Метод jQuery 'slice' хороший этому пример :

[cc lang="javascript"]
/* ... */
slice: function() {
	return this.pushStack(
	Array.prototype.slice.apple(this, arguments),
	'slice',
	Array.prototype.slice.call(arguments).join(',')
	);
},
/* ... */
[/cc]

Родной метод jQuery 'slice' не заботится о том, что 'this' указывает на реальный массив - это и есть получение 'length' (длины), как [0], [1] и т.д.

Еще интересные свойства у объекта jQuery - это '.selector' и '.context', которые отражают аргументы, который мы передаем, вызывая 'jQuery(...)'.

[cc lang="javascript"]
var jqObject = jQuery('a');
jqObject.selector; // => 'a'
[/cc]

Одна вещь, которую важно не упустить из виду, jQuery иногда возвращает нам для манипуляций новый объект jQuery. Если мы запускаем метод, который как-либо меняет коллекцию, к примеру как '.parents()', то jQuery не будет изменять объект, он просто вернет новый экземпляр.

[cc lang="javascript"]
var oroginalObject = jQuery('a');
var anotherObject = originalObject.parents();

original === anotherObject; // => false
[/cc]

Все методы, которые как-либо изменяют коллекцию возвращают новый jQuery объект, вы также можете обратиться к старому объекту, вызвав '.end()' или более расширенный '.prevObject'.



### Легкое создание элементов


Главная возможность объектной модели документа или попросту DOM jQuery - это создание элементов. Версия 1.4 изменила методику их создания, сделав ее намного более простой и лаконичной.

[cc lang="javascript"]
var myDiv = jQuery('

', {
	id: 'myNewElement', 
	class: 'someClass', 
	css: {
		color: 'blue',
		backgroundColor: '#fff', 
		border: '1px dashed red'
	},
	click: function() {
		alert('Ты на меня нажал!');
	},
	html: jQuery('', {
		href: '#',
		click: function() {
			//сделать что-нибудь
			return false;
		}
	})
});
[/cc]

Чтобы разузнать больше о соответствии методам jQuery, напишите в консоле 'jQuery.attrFn'.



### Сериализация


jQuery предоставляет методы для сериализации input-полей одной или нескольких форм. Это полезно когда отсылаются данные по средствам XMLHttpRequest объекта (AJAX). Сериализация была реализовано очень давно и очень немногие разработчики пользовались ей. Теперь же отправление целой формы через AJAX, используя jQuery, просто не может быть проще. Взгляните на пример.

[cc lang="javascript"]
var myForm = $('#myForm');
jQuery.post('submit.php', myForm.serialize(), function() {
	alert('Форма отправлена');
});
[/cc]



### Заставьте двигаться что угодно на своей странице



Метод jQuery 'animate', наверное, самый гибкий из имеющихся. Он может быть использован для анимации чего угодно, не только манипулируя CSS свойствами и не только DOM элементами. Теперь о том, как мы его можем использовать. 

[cc lang="javascript"]
jQuery('#animateMe').animate({
	left: 300, 
	top: 300
});
[/cc]

Когда вы узазываете свойство, которое следует анимировать (к примеру, 'left'), jQuery проверят является ли это свойство свойством стиля ('element.style'), затем проверяет является ли оно подсвойством 'style', - если нет, то jQuery просто обновляет свойство 'left'. Маленький пример, чтобы это закрепить. 

[cc lang="javascript"]
jQuery('#animateMe').animate({
	top: 123, 
	hello: 111
});
[/cc]

'top' - это действительно свойство CSS, поэтому jQuery обновит 'element.style.top', а про свойство 'hello' такого сказать нельзя, поэтому jQuery просто обновит свойство 'element.hello'.

Мы легко можем использовать это преимущество. Сказать, к примеру, что мы хотим анимировать квадрат на холсте (canvas). Сначала объявим простой конструктор и метод 'draw', который будет вызываться на каждом шаге анимации.

[cc lang="javascript"]
function Square(cnvs, width, height, color) {
	this.x = 0; this.y = 0;
	this.width = width; this.height = height;
	this.color = color;

	this.cHeight = cnvs.height;
	this.cWidth = cnvs.width;
	this.cntxt = cnvs.getContext('2d');
}

Square.prototype.draw = function() {
	this.cntxt.clearRect(0, 0, this.cWidth, this.cHeight);
	this.cntxt.fillStyle = this.color;
	this.cntxt.fillRect(this.x, this.y, this.width, this.height);
};
[/cc]

Здесь мы содаем наш 'Square' конструктор и один из его методов. Создание холста (canvas) и последующая анимация тоже абсолютно прозрачна и проста.

[cc lang="javascript"]
var canvas = $('').appendTo('body')[0];
canvas.height = 500;
canvas.width = 600;

var square = new Square(canvas, 50, 50, 'rgb(0, 0, 200)');

jQuery(square).animate({
	x: 400,
	y: 200
}, {
	//'draw' должен вызываться на каждом шаге анимации
	step: jQuery.proxy(square, 'draw'),
	duration: 1000
	});
});
[/cc]

Это очень просто эффект, который понятно демонстрирует возможности. [Наглядный пример](/examples/jquery/square.html)



### jQuery.ajax


AJAX методы jQuery ('.ajax', '.post', '.get') возвращают 'XMLHttpRequest' объект, который может быть использован для выполнения соответствующих операциях при запросах. К примеру:

[cc lang="javascript"]
var cRequest;

jQuery('button.doRequest').click(function() {
	cRequest = jQuery.get('getting.php', function(response) {
		alert('Данные: ' + response.responseText);
	});
});

jQuery('button.cancelRequest').click(function() {
	if (cRequest) {
		cRequest.abort(); // метод XMLHttpRequest
	}
});
[/cc]

Здесь мы отправляем запрос, нажимая на кнопку '.doRequest', прирываем активный запрос нажатием '.cancelRequest.

Другой потенциальный способ использования - синхронные запросы:

[cc lang="javascript"]
var myRequest = jQuery.ajax({
	url: 'someurl.php',
	async: false
});

console.log(myRequest.responseText);
[/cc]



### Несколько слов об очередях


jQuery имеет встроенный механизм для работы с очередями, который используется всеми методами анимацию. Мы сейчас посмотрим на это на простом примере:

[cc lang="javascript"]
jQuery('a').hover(
function() {
	jQuery(this).animate({paddingLeft:'+=15px'});
}, function() {
	jQuery(this).animate({paddingLeft:'-=15px'});
});
[/cc]

Быстрый обход всей группы ссылок и при наведении на какую-либо она выдвигается, что очень хорошо подходит для организации несложного меню. [Пример](/examples/jquery/menu.html)

Метод 'queue' очень похож на вызов метода 'each'. Мы передаем в него функцию, которая в итоге будет вызвана для каждого элемента в коллекции.

[cc lang="javascript"]
jQuery('a').queue(function() {
	jQuery(this).addClass('all-done').dequeue();
});
[/cc]


