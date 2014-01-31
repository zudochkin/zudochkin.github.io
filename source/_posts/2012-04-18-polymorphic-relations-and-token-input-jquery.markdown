---
author: admin
comments: true
date: 2012-04-18 19:26:02+00:00
layout: post
slug: polymorphic-relations-and-token-input-jquery

permalink: /2012/04/polymorphic-relations-and-token-input-jquery

title: Rails Полиморфная связь и TokenInput jQuery
wordpress_id: 916
categories:
- jQuery
- Ruby On Rails
tags:
- jQuery
- rails
---

Представьте ситуацию, что у вас имеется некая модель, к которой вы хотите добавлять по средствам связей другие модели. Хорошо, когда данную связь можно описать обычным **has_many**, другое дело, когда нужно привязать разнородные модели, к примеру, у вас имеется модель вопросов, к которой нужно привязать город или страну <!-- more -->(не очень наглядный пример, вряд ли он вам в жизни встретится, но я поделюсь своим опытом).



### Инструментарий


Как вы, наверное, догадались, основным инструментом будет выступать ruby an rails. Из гемов будем использовать **simple_form** для удобного создания форм, **inherited_resources**, чтобы писать намного меньше кода в контроллерах (сейчас не представляю без этого гема жизни). 

Также нам понадобится плагин для jQuery [tokenInput](http://loopj.com/jquery-tokeninput/), который будет отображать выпадающий список с autoComplete'ом, из которого мы сможем выбрать одно значение из стран или городов (список будет один).



### Реализация


Начнем с контроллера **QuestionableController** (не очень, конечно, удачное название), который будет отдавать солянку из городов и стран с возможностю поиска. 
[cc lang="ruby"]
class QuestionableController < ApplicationController
  def index
    countries = Country.find(:all, conditions: ['name LIKE(?)', "%#{ params[:q] 
    cities = City.find(:all, conditions: ['name LIKE(?)', "%#{ params[:q] }%"])

    @questionable = countries + cities

    respond_to do |format|
      format.json { render }
    end  
  end  
end
[/cc]

Лучше вынести всю логику в модель, но данный пример создан исключительно для демонстрации. Что мы делаем в этом контроллере? Ищем страны и города, удовлетворяющих параметру **q** и заполняем инстанс переменную **@questionable**

Также давайте заполним нашу вьюшку _index.json.erb_
[cc lang="ruby"]
<%= sanitize @questionable.to_json(methods: [:id_with_class_name], only: [:id, :name]) %>
[/cc]

в которой мы рендерим наш json, т.к. нам не нужны все поля модели мы используем только необходимые (only: [:id, :name]), ключ methods: указывает на то, что мы помимо физических свойст, будем использовать метод модели _id_with_class_name_.

Пока не забыл, давайте пропишем пару строк в наш _config/routes.rb_
[cc lang="ruby"]
resources :questions # для того, чтобы создавать/показывать/удалять вопросы
resources :questionable, only: :index # для того, чтобы отображать json
[/cc]

Перейдем к моделям:

_models/question.rb_
[cc lang="ruby"]
class Question < ActiveRecord::Base
  attr_accessor :location

  belongs_to :questionable, polymorphic: true
  attr_accessible :name, :questionable_id, :questionable_type

  before_save do
    return unless self.location =~ /_/
    location_id, location_type = self.location.split('_')
    self.questionable_id   = location_id   unless location_id.nil?
    self.questionable_type = location_type unless location_type.nil?
  end
end
[/cc]

Самая интересная строчка - это та, которая начинается с belongs_to, в ней мы объявляем полиморфную связь questionable. Дальше идет метод, который мы 
вызываем каждый раз перед сохранением в базу данных. В нем мы парсим строку из autoComplete'а, которая будет иметь вид id_ModelName и сохраняем отдельно id и ModelName.

Модель _city.rb_
[cc lang="ruby"]
class City < ActiveRecord::Base
  has_many :questions, as: :questionable
  attr_accessible :name

  def id_with_class_name
    "#{ id }_#{ self.class.name }"
  end
end
[/cc]

В ней объявляется связь has_many через questionable, который мы обявили в модели question полиморфной. Далее идем метод, который возвращает id записи и название класса модели (чтобы можно было отделить зерно от плевел).

Модель _country.rb_ почти такая же
[cc lang="ruby"]
class Country < ActiveRecord::Base
  has_many :questions, as: :questionable
  attr_accessible :name
  
  def id_with_class_name
    "#{ id }_#{ self.class.name }"
  end
end
[/cc]

Связь обявлена также как и в предыдущей модели.

Наш маленький контроллер _QuestionController_
[cc lang="ruby"]
class QuestionsController < InheritedResources::Base

end
[/cc]
где остальное спросите вы. Это все. Это все, что нам нужно для того, чтобы создавать, удалять, редактировать вопросы. Как вы, наверное, заметили наш контроллер наследуется не от ApplicationController'а, а от inherited_resources. Что это значит? А это значит, если абстрагироваться от 80% возможностей этого гема, то, что гем берет всю "грязную" работу на себя, предоставляя нам в пользование переменные resource и collection, хранятся в которых ссылка на текущую запись (resource) и список всех записей (collection) соответственно. Для того, чтобы мы смогли создать свой первый вопрос нам необходимы лишь вьюшки. Начнем с _form.html.erb, которую мы будем подключать и в создании вопроса, и в редактировании.

_app/views/questions/_form.html.erb_
[cc lang="ruby"]
<%= simple_form_for resource do |f| %>
  <% pre = if resource.questionable %>
    <% [resource.questionable].to_json(only: [:id, :name]) %>
  <% end %>
  <%= f.input :name %>
  <%= f.input :location, input_html: { 'data-pre' => pre, class: 'token-input-questionable' } %>
  <%= f.submit nil %>
<% end %>
[/cc] 

Как видите, почти никаких различий с нативным form_for нет. Во второй строке мы заполняем переменную pre json'ом выбранного города или страны, в предпоследней создаем инпут с атрибутом data-pre, равным json'у и классом, чтобы мы могли найти этот элемент без проблем.

_new.html.erb_ и _edit.html.erb_ абсолютно одинаковы и представляют собой следующее:

[cc lang="ruby"]
<%= render 'form' %>
[/cc]

в них мы отрисовываем только форму.

Далее кинем наш скачанный jquery.tokeninput.js в папку _app/assets/javascripts_ и допишем в _application.js_ следующее:
[cc lang="javascript"]
$(function() {
  var $input = $('.token-input-questionable');
   $input.tokenInput('/questionable.json', {
      tokenLimit: 1,
      tokenValue: 'id_with_class_name',
      prePopulate: $input.data('pre')
    });
});
[/cc]

Первым параметром к tokenInput мы указываем откуда брать данные, tokenLimit:1 указывает на то, что одной записи нам будет достаточно (одного элемента из выпадающего списка), tokenValue - откуда мы будем брать имя, которое будем отображать из json'а, prePopulate используем для того, чтобы заполнить элемент, если мы его уже выбрали (редактирование).

![polymorphic relations rails](http://vredniy.ru/wp-content/uploads/2012/04/rsz_1screenshot_from_2012-04-18_230555-300x209.png)Сумбурным получилось изложения материала, потому что для меня это в новинку и писалось это для того, чтобы не забыть полезную "плюшку" в дальнейшем. Если заметка кому-нибудь еще пригодится, я буду очень рад :)

P.S.: Если данную заметку читает Александр, мало ли, то, надеюсь, он будет не против, что я позаимствовал его идею :)

Демо проект "лежит" на [heroku](http://vredniy-polymorphic.heroku.com/questions/), репозиторий, как всегда на [github](https://github.com/vredniy/rails-token-input) (там же лежат и миграции, нужные для запуска проекта).

Удачи вам в любых начинаниях и до новых встреч :)
