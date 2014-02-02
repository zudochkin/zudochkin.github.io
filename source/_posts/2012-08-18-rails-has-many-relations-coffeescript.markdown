---
author: admin
comments: true
date: 2012-08-18 16:52:03+00:00
layout: post
slug: rails-has-many-relations-coffeescript

permalink: /2012/08/rails-has-many-relations-coffeescript

title: Один вопрос и несколько ответов
wordpress_id: 925
categories:
- Ruby On Rails
tags:
- coffeescript
- rails
- simple_form

keywords: "rails, coffeescript, accepts_nested_attributes_for rails,coffeescript,simple_form,ruby on rails"
description: "Имеется, к примеру, у нас вопросы и ответы и мы, вместо того, чтобы создавать сначала вопрос и потом привязывать к нему по одному ответы, будем делать это все на одной странице."
---





Давно что-то я не писал, а тут выдался свободный денек, чтобы рассказать еще об одной плюшке, которую я узнал. Все приведенное ниже не претендует на ортодоксальную правильность, это лишь мое мнение и мое решение.





Имеется, к примеру, у нас вопросы и ответы и мы, вместо того, чтобы создавать сначала вопрос и потом привязывать к нему по одному ответы, будем делать это все на одной странице.


<!-- more -->

{% img image /images/posts/2012-08-rails-has-many-relations-coffeescript/rails-nested-relations-300x243.png %}

### Решение

В решении нам помогут gem _simple_form_, который в разы облегчает работу с формами и _coffescript_, который хоть и транслируется в javascript, но по синтаксису очень похож на руби со своим "сахаром".

#### Модели

Как ясно из название, у нас будут две модели: Question и Answer, в них нет ничего сложного

Вопрос

``` ruby
class Question < ActiveRecord::Base
  attr_accessible :name, :answers_attributes

  has_many :answers

  accepts_nested_attributes_for :answers

  validates :name, :presence => true
end

```

Ответ

``` ruby
class Answer < ActiveRecord::Base
  attr_accessible :name, :question_id

  belongs_to :question

  validates :name, :presence => true
end

```

Единственное, что может показаться странным с первого взгляда, так это строка accepts_nested_attributes. Которая говорит, что модель Question может принимать атрибуты и для ответа, т.е. один post запрос может нам и вопрос создать и ответы к нему.

Я для экономии времени использовал twitter-bootstrap-rails gem и scaffold, поэтому все получилось почти готовое. Лишь небольшие правки я внес в форму _app/views/questions/_form.html.erb_


``` html
<%= f.input :name, :label => 'Текст вопроса' %>

<div class="answers">
  <%= f.simple_fields_for :answers do |answers_form| %>
    <div class="answer">
      <%= answers_form.input :name, :label => 'Текст ответа' %>

      <%= link_to '#', :class => 'remove_answer' do %>
        <i class="icon-minus-sign"></i>
      <% end %>
    </div>
  <% end %>
</div>

<%= link_to '#', :id => 'add_answer' do %>
  <i class="icon-plus-sign"></i>
<% end %>
```

Я обернул ответы на вопрос классом .answers, чтобы они визуально отличались, добавил вложенную форму для вопросов с помощью simple_form и две кнопки: "+" - для добавления ответа и "-" - для его удаления

Сейчас задача сводится к тому, чтобы динамически добавлять/удалять ответы на странице без ее перезагрузки, взглянем на генерируемый html

``` html
<input class="string required" id="question_answers_attributes_0_name" name="question[answers_attributes][0][name]" size="50" type="text">
```

При добавлении нового элемента возрастает цифра в айдишнике и в имени текстового поля. Для того, чтобы создать новый ответы мы будем действовать так: склонируем последний ответ на странице с помощью $.clone(), поменяем все элементы name и id, заменив у них цифру в середине. Заменять будем на new Date().getTime(), который генирует случайное число, хоть и большое, но оно нам подходит. Также нам нужно будет не забыть о лейблах, а то получится, что мы кликаем на лейбл, а в фокус попадает совсем не соответствующий элемент. Итак, поехали.

``` coffeescript
$ ->
  # обрабатываем клик по кнопке добавления
  $('#add_answer').click ->
    $last = $('.answer:last') # последний ответ на странице

    $cloned = $last.clone() # клонируем его

    # получаем рандомное число
    randValue = new Date().getTime()

    # пробегаемся по всем элементам, у которых есть атрибут name или for
    $('[name], [for]', $cloned).each (index, item) ->
      $item = $(item)

      # если это текстовое поле, заменяем id и name
      if $item[0].nodeName is 'INPUT'
        $item.attr 'id', $item.attr('id').replace(/\d+/, randValue)
        $item.attr 'name', $item.attr('name').replace(/\d+/, randValue)

      # если это лейбл, то заменяем атрибут for
      $item.attr 'for', $item.attr('for').replace(/\d+/, randValue)  if $item[0].nodeName is 'LABEL'

    # скрываем добавляемый ответ, чтобы красиво появиться
    $cloned.hide()
    $cloned.appendTo $last.parent()
    $cloned.slideDown()

    false

```

Осталось добавить обработчик события удаления ответа

``` coffeescript
# при клике на удаление ответа, удаляем его
$('body').delegate '.remove_answer', 'click', ->
  $(this).closest($('.answer')).slideUp ->
    $(this).remove()

  false

```

Метод $.closest находит ближайший элемент с классом .answer, поднимаясь вверх по дереву DOM. Находим его, скрываем, а после проигрыша анимации удаляем.

### Что можно сделать лучше

  * Добавить валидацию на количество ответов в модели, к примеру, `validates_length_of :answers, :minimum => 1`
  * Добавить проверку, чтобы не удалить последний ответ, после удаления которого мы не сможем добавить новый - клонировать некого :)
  * Не использовать скаффолд, который генерирует много ненужного мусора, по крайней мере, для данной задачи
  * Да и вообще предела-то совершенству нет :)

Надеюсь, данная неидеальная реализация кому-нибудь в жизни пригодится, вопросы, советы? - велком в комментарии.

Чуть не забыл ссылку на [репозиторий на гитхабе](https://github.com/vredniy/nested-relations)
