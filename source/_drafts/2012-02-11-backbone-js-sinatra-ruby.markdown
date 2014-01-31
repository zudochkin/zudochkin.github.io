---
author: admin
comments: true
date: 2012-02-11 19:58:01+00:00
layout: post
slug: backbone-js-sinatra-ruby

permalink: /2012/02/backbone-js-sinatra-ruby

title: Backbone.js и Sinatra (Ruby)
wordpress_id: 883
categories:
- javascript
- jQuery
- Ruby
tags:
- backbone
- heroku
- jQuery
- ruby
- sinatra
---

Приветствую тебя, товарищ-разработчик, сегодня речь пойдет больше о клиентсом программировании, точнее об одном фреймворке. Backbone js ему название. Что в нем особенного спросите вы. Это первый MVC фреймворк на стороне клиента, попавшийся мне на глаза. Очень долго я с ним воевал, поэтому тут уж дело чести понять как работают 1200 строк кода на javascript. 
<!-- more -->
Данная заметка не прендует на всеобъемлющее описание, да и практической пользы от разрабатываемого в ней приложения (без доработки) практически нет, это больше для души или, как говорится, for fun.

Сначала поставим себе небольшую цель, чего же мы хотим увидеть по окончании: одна страница, без ссылок и переходов, все взаимодействие с сервером происходит в режиме реального времени по средством ajax запросов. В конце мы сможем добавлять новые книги, удалять их и изменять поля. Весь "дизайн" выполнен человеком, то бишь мною, ничего общего с дизайнерами и верстальщиками не имеющего, но, как мне кажется, он [дизайн], получился удобным и не отталкивающим. Обрабатывать наши ajax запросы будет Ruby при помощи замечательного фреймворка Sinatra, но работа им предстоит не сложная, поэтому сосредоточимся на frontend'е.



### Что еще за backbone.js?


Аналог jquery? Нет, у данного фреймворка совсем другая специализация - его основная задача создать основу для модульного приложения на стороне клиента, чтобы богатые на компоненты и модули интерфейсы легко могли взаимодействовать с пользователем. Чтобы добавления новой функциональности не превращало код приложения в спагетти, а наоборот, этому способствовало.



### Из чего состоит backbone.js?


Модель, Представление, Коллекция и Роутер, если не сильно вдаваться в подробности (очень хорошая документация собрана на сайте backbone.js), то модель представляет собой, если проводить аналогию с базами данных, строку в таблице, к любым свойствам которой мы легко можем обращаться, также легко мы можем ее обновлять, сохранять и прочее. Коллекция - кучка моделей или по аналогии - несколькими строками таблицы, объединенных общими полями и свойствами. Представление - вот это, по-моему, самая интересная и в то же время непростая для меня часть. Представление в понимании создателей backbone.js - это совсем иное, нежели мы, backend-разработчики привыкли видеть. Это что-то очень похожее на букву V из аббревиатуры MVC, но с немного большими возможностями и ответственностями. Я думаю на примере станет немного яснее. И последним будет Роутер - в данном приложении я его использовать не буду, оно слишком маленькое, чтобы усложнять его и без того не простую к пониманию новую технологию, по крайней мере, для меня. Как написано на сайте backbone.js роутер предоставляет методы для связывания ссылок на странице со всевозможными событиями. С появлением в нашей жизни History API, появилась возможность использовать наряду с ссылками вида /#about, ссылки обычного вида, /about, к примеру, что намного приятнее для глаза человеческого.
![backbone sinatra](http://vredniy.ru/wp-content/uploads/2012/02/backbone-sinatra.jpeg)
Итак, хватит воды, давайте приступим к делу. Начнем, конечно же с html, который и будет подключать все библиотеки стили на стороне клиента, в нем же будем хранить наш шаблон, что для маленького приложения не будет грубой ошибкой.

[cc lang="html"]










[/cc]

Начнем с подключения библиотеки и стилей: за основу взят, уже не в первый раз css фреймворк bootstrap от twitter, в самом начале подключаем нашу любимую библиотеки [Query](/category/jquery), но backbone.js может работать и без нее, это больше для нас, чтобы вместо 5 строк не писать 50 :). underscore.js - библиотека-утилита, которая предоставляет основу для функционального программирования на JavaScript, без него не работает наш backbone.js, поэтому подключайте underscore.js раньше. Остальные библиотеки не должны вызвать сложностей в понимании: они уже у всех на слуху, отдельно коснусь закомментированной строки. Если ее расскомментировать и внести небольшие изменения в код, то вместо хранения всего в базе данных, мы будем использовать localStorage браузера, который это поддерживает.

Теперь немного разметки,

[cc lang="html"]



  


  	


  		

# Моя библиотека


  	


  


  


    


      


      	
      	

### Добавить


      	
		Название книги
		
		Добавить новую

	


    


    


    	
    


    


    	

### Описание


    


  






[/cc]

Здесь ничего особенного, поэтому не буду комментрировать.

Самое время привнести в наш проект немного магии, которое будет не мало. Первая магия - шаблоны на стороне клиента.
[cc lang="html"]

[/cc]

Казалось бы так похож на erb, но это просто совпадение, или разработчики данного шаблона рубисты. Вместо конструкций вида <%= title %> в конечном html, появятся наши переменные, нам их нужно будет лишь заполнить. А верстка останется неизменной, что очень удобно.

Почти все, кроме одной строки, хотя эта одна строчка подключает сердце нашего приложения 
[cc lang="html"]

[/cc]

Теперь самое время окунуться с головой в клиентское программирование (файл js/application.js).

Начнем, пожалуй, с модели.
[cc lang="javascript"]
var Book = Backbone.Model.extend({
	defaults: {
		title: "Book's title",
		year: 2009,
		author: "Murakami",
		genre: ["horror", "comedy"],
		isbn: "0128127622",
		status: "not read",
		image: "/images/placeholder.png",
		date: ''
	},
});
[/cc]

Все модели нашего приложения должны расширять базовый класс Backbone.Model, свойство defaults, как понятно из названия отвечает за установку свойств модели по умолчанию. Я взял для примеры стандартные книжные свойства, большая часть из которых работает не до конца. 

Теперь перейдем непосредственно к нашей библиотеке, расширив класс коллекции Backbone.Collection

[cc lang="javascript"]
var Library = Backbone.Collection.extend({
	//localStorage: new Store("BackboneCollection"),
	url: '/books',
	model: Book,
});
[/cc]

Закомментированная строка сохраняла бы всю нашу работу в local Storage браузера, мы же будем взаимодействовать с сервером, для этого нам понадобится адрес, по которому и будет производиться все общение (точнее будет основой для производных адресов). Последняя строка указывает коллекция каких объектов нами будет использоваться.

Если для модели нам не нужен объект, то для коллекции пригодится, поэтому создадим его.

[cc lang="javascript"]
var library = new Library();
[/cc]


А сейчас начнется самое интересное - Представление. Будем разбирать его неспеша, чтобы ничего важного не упустить.

[cc lang="javascript"]
var BookView = Backbone.View.extend({
		tagName: 'div',
		className: 'book',

		template: _.template($('#book-template').html()),

		events: {
			"click a.book-edit": "edit",
			"keypress input": "update",
			"click .book-delete": "clear",
			"click .book-save": "close",
		},
[/cc]

Сначала расширяем базовый класс Backbone.View, далее указываем элемент, с которым мы будем работать, в данном случае, это div с классом book. Следующая строка указывает какой шаблон мы будем рендерить и сохраняет его в переменной. В свойстве events мы перечисляем какие события мы будет "прослушивать".
Каждая строка состоит из следующих моментов: сначала идет событие, которое мы будем обрабатывать, дальше - элемент, с которым это событие произойдет, после двоеточия - имя метода, который будет вызываться при наступлении прослушиваемого события.

Первая строка - нажимаем на ссылку - редактирование книги, вторая - нажимаем на кнопку (в указанном методе мы проверям Enter ли это) в текстовом поле - сохранить изменения. И последние две - это удаление и сохранение изменений.

Теперь опишим все методы нашего представления

[cc lang="javascript"]
initialize: function() {
	this.model.bind('change', this.render, this);
	this.model.bind('destroy', this.remove, this);
},

update: function(e) {
	if (e.keyCode == 13) this.close();	
},

close: function() {
	this.model.set(this._get());
	this.model.save(this.model.toJSON());
		$(this.el).removeClass("editing");	
},
[/cc]

Первым не важно для модели, коллекции или представления вызывается метод initialize, поэтому можно расценивать его как конструктор.

this.model будет во время работы приложения указывать на текущую модель, т.е. каждый кусок разметки будет связан с отдельной моделью, изменения в котором коснутся только одной модели, также как и наоборот. Первым делом мы связывем события модели с методами: первое это изменение модели, после которого должно произойти "отрисовка", второе - удаление.

Метод _update_ вызывается каждыйраз, когда пользователь нажимает на клавиатуру, нам нужно отлавливать только нажатие Enter, чтобы сохранить изменения.

Метод _close_ вызывается, когда пользователь нажал Enter или кликнул по кнопке "Сохранить", посмотрите еще раз на последнюю строку из свойства events. В первой строке которого мы обновляем нашей модели свойств из текстовых полей. Дальше мы сохраняем модель и удаляем с себя класс редактирования (нужно будет для "рюшечек").

Следующий метод, наверное, самый значимый, он отрисовывает все изменения модели на экране пользователя. 

[cc lang="javascript"]
render: function() {
	$(this.el).html(this.template(this.model.toJSON()));
	$(this.el).css({'background': ' no-repeat url(' + this.model.get('image') + ')'});
	this.setText();
	return this;
},
[/cc]

_this.el_ ни что иное, как tagName: 'div' и className: 'book'. Первая строка данного метода изменяет html элемента, учитывая новые свойства модели и учитывая разметку из шаблона. (Помните, мы писали недавно template: _.template($('#book-template').html())). Вторая строка данного метода заполняет фоновой картинкой наш кусок разметки.

"Толстые" методы для обновления свойств модели, исходя из значения текстовых полей или наоборот - текстоых полей, исходя их свойств модели.

[cc lang="javascript"]
setText: function() {
	var text = this.model.get('title');
	this.$('.book span.book-title').text(text);
	this.inputTitle = this.$('input.book-title');
	this.inputYear = this.$('input.book-year');
	this.inputAuthor = this.$('input.book-author');
	this.inputGenre = this.$('input.book-genre');
	this.inputIsbn = this.$('input.book-isbn');
	this.inputStatus = this.$('select.book-status');
	this.inputDate = this.$('input.book-date');
},
_fillForm: function() {
	var data = this.model.toJSON();
	this.inputTitle.val(data.title);
	this.inputYear.val(data.year);
	this.inputAuthor.val(data.author);
	this.inputGenre.val(data.genre);
	this.inputIsbn.val(data.isbn);
	this.inputStatus.val(data.status);
	this.inputDate.val(data.date);
},

_get: function() {
	return {
		title: this.inputTitle.val(),
		year: this.inputYear.val(),
		author: this.inputAuthor.val(),
		genre: this.inputGenre.val(),
		isbn: this.inputIsbn.val(),
		status: this.inputStatus.val(),
		date: this.inputDate.val()
	}
},

[/cc]

И последние 3 методя для данного представления:

[cc lang="javascript"]
edit: function() {
	$('div.book').removeClass('editing');
	$(this.el).addClass('editing').find('input, select').fadeIn('slow');
	this._fillForm();
	this.inputTitle.focus();
	return false;
},

clear: function() {
	if (confirm("Вы уверены?")) {
		this.model.destroy();
	}
},

remove: function() {
	$(this.el).fadeOut('slow', function() {$(this).remove()});
}
[/cc]

Первый - уделяет класс редактирования со всех книг и устанавливает на активной и заполняет форму значениями модели (данный метод вызывается, когда пользователь нажмет на ссылку для редактирования).

Второй - задает пользователю вопрос, при утвердительном ответа на который удаляет данную модель.

Третий - вызывается автоматически и удаляет элемент со страницы, а вызывается он из-за того, что мы в конструкторе написали следующее:
[cc lang="javascript"]
this.model.bind('destroy', this.remove, this);
[/cc]


Осталось рассмотреть последнее представление и можно смело переходить к легкому backend'у.

[cc lang="javascript"]
var AppView = Backbone.View.extend({
	el: $('#books'),
	
	events: {
		"click button": "create",
		"keypress #book-title": "createOnEnter"
	},

	initialize: function() {
		this.input = this.$('#book-title');
		library.bind('add', this.addOne, this);
		library.bind('reset', this.addAll, this);
		library.bind('all', this.render, this);
		library.fetch();
	},

	render: function() {
	},
[/cc]

Данное представление отвечает за создание новых книг и загрузку книг с сервера. Книга создается заполнением только лишь названия, дальше ее можно отредактировать и сохранить. В конструкторе данного представления мы связываем необходимые события коллекции с методами, которые будет вызываться при наступлении событий. **library.fetch();** - запрашивает с сервера все книги, чтобы потом их отобразить.

Почему метод рендеринга пустой? Потому что он нам тут не нужен, все отрисовывается в методе добавления новой книги, с него и начнем продолжение рассказа.

[cc lang="javascript"]
addOne: function(book) {
		var view = new BookView({model: book});
		var content = view.render().el;
		$(content).hide();
		$('#books').find('.book-container').prepend(content);
		$(content).show(1000);
	},

	addAll: function() {
		library.each(this.addOne);
	},

	createOnEnter: function(e) {
		if (e.keyCode == 13) {
			this.create();	
		}
	},

	create: function(e) {
		var text = this.input.val();
		library.create({title: text});
		this.input.val('');
	}
});

//$('.book-date').datepicker({});
var appView = new AppView();
[/cc]

Метод addOne добавляет в верстку одну книгу, создавая соответствующее представление. Метод addAll - пробегается по все коллекции книг и по одной с помощью метода addOne добавляет книги в верстку. Метод create создает для нас книгу с заголовком, который мы укажем в текстовом поле.

Осталось только создать нам представление приложения и наслаждаться результатом, хотя сначала нужно реализовать backend. Но он будет очень простым, поэтому быстрее перейдем к нему.



### Backend на Ruby


Сначала общие моменты: подключение необходимых гемов, задание дефолтного подключения для DataMapper'а и описание класса Книга:
[cc lang="ruby"]
# coding: utf-8
  require 'sinatra'
  require 'data_mapper'
  require 'json'

  DataMapper.setup(:default, ENV['DATABASE_URL'] || 'sqlite:./db/books.db')

  class Book
    include DataMapper::Resource

    property :id,           Serial
    property :title,        String
    property :year,         Integer
    property :date,         Date
    property :image,        String
    property :author,       String
    property :genre,        String
    property :isbn,         String
    property :status,       String

    #belongs_to :user


  end

  DataMapper.finalize

  get '/' do
    File.read('./public/index.html')
  end 

[/cc]

Главной страницей у нас будет статичный файл .html
Теперь давайте сделаем небольшую паузу и поговорим о том, как взаимодействует Backbone.js с серверной частью. Если заглянуть в исходники, то там мы увидим следующее:

[cc lang="javascript"]
 var methodMap = {
    'create': 'POST',
    'update': 'PUT',
    'delete': 'DELETE',
    'read':   'GET'
  };
[/cc]

Т.е. стандартный RESTfull запрос. Не будем оттягивать кота за яйца, закончим уже кодить над этим проектом, тем более, что Sinatra легко может обработать подобные запросы.

[cc lang="ruby"]
  get '/books' do
    content_type :json
    Book.all(:order => :id).to_json
  end

  post '/books' do
    data = JSON.parse(request.body.gets)
    Book.create(:title => data['title']);  
  end

  put '/books/:id' do
    data = JSON.parse(request.body.gets)
    book = Book.get(params[:id])
    result = book.update(
      :title => data['title'],
      :year => data['year'],
      :author => data['author'],
      :genre => data['genre'],
      :isbn => data['isbn'],
      :status => data['status'],
      :image => '/images/placeholder.png', #data['image'],
      :date => Time.now
    )
    "false" unless result
  end

  delete '/books/:id' do
    Book.get(params[:id]).destroy
  end
[/cc]


Здесь нет ничего сложного, если есть какие-то сложности, посмотрите мои предыдущие заметки о Ruby и о Sinatra.



### Заключение



Еще одно приложение мы разработали с вами на ruby, сегодня не обошлось без так важного на сегодняший день клиентсткого программирования. Ссылка на [пример](http://vredniy-library.heroku.com/) (редактирование и создание не сохраняются в базе)  и ссылка на [исходник](https://github.com/vredniy/Backbone-sinatra).
![backbonejs ruby sinatra](http://vredniy.ru/wp-content/uploads/2012/02/placeholder.png)
Всем спасибо за внимание и до новых встреч :) 
