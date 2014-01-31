---
author: admin
comments: true
date: 2012-03-30 10:55:02+00:00
layout: post
slug: unobtrusive-javascript-rails
title: Собеседование и Unobtrusive Javascript на Rails
wordpress_id: 911
categories:
- Ruby
- Ruby On Rails
tags:
- heroku
- rails
- ujs
---

![unobtrusive javascript rails](http://vredniy.ru/wp-content/uploads/2012/03/ajax-crud-150x150.jpg)Приветствую всех гостей и постояльцев на своем техническом, иногда не совсем, блоге. Кто читает меня в твиттере, наверное, помнят то, что вчера у меня было первое в жизни собеседование на ruby on rails вакансию, кто не читает, милости просим [дружить в твиттере](http://twitter.com/vredniy).<!-- more -->

Встретились мы сначала с генеральным директором, он рассказал о проекте, пока его не буду называть, но это компания, которая занимается развитием сайта, посвященному туризму: туры, бронирование отелей, такси и прочей смежной тематикой. Обсудили нетехнические вопросы. Затем подошел старший программист и по большей части мы уже общались непосредственно с ним. Конечно, сказывалось отсутствие большого опыта программирования на ruby и ruby on rails, но на большую часть вопросов, как мне кажется, я ответил правильно. Вопросы были следующего характера: 



	
  * какие виды связей и как их создать в rails

	
  * чем отличается include от extend'а

	
  * имеется ли множественное наследование в ruby



Параллельно с вопросами я рассказывал о своих проектах и на ruby, и на php. Рассказал почему захотел перейти с php на ruby, как решал задачу из предыдущего поста тоже рассказал :).

После собеседования мне дали тестовое задание, оно заключается в следующем: необходимо без сторонних гемов, плагинов и библиотек реализовать простое приложение по управлению записями (обычный CRUD), только все это дело должно располагаться на одной странице, все запросы должны происходить в фоне (ajax) с применением unobtrusive javascript'а. Еще мне нужно было засечь сколько времени займет все это дело. 

На ознакомление с мат. частью (railscasts, guides.rubyonrails и пр.), так сказать, ушло около двух часов, на реализацию проекта столько же. Что получилось и ссылку на репозиторий вы можете увидеть в конце данной заметки.

Итак, начнем.
Единственную "плюшку", которую я применил в проекте был gem 'twitter-bootstrap-rails' для придания "красивости", на конечную функциональность он никак не повлиял. 

Сначала, как и всегда, создаем проект **rails new unobtrusive && unobtrusive**. Вписываем в _Gemfile_, что будем использовать twitter-bootstrap-rails **gem 'twitter-bootstrap-rails'** и делаем **bundle**, чтобы обновить _Gemfile.lock_

Далее установим нужные стили, js-файлы, написав в консоли _rails g bootstrap:install_ и обновим наш шаблон **application** командой _rails g bootstrap:layout application fluid_.

Скаффолдом создадим все необходимое для задачи _rails g scaffold item name:string description:string_, теперь мы можем создавать, удалять и редактировать, но без ajax'а. Также давайте добавим красоты нашим item'ам _rails g bootstrap:themed Items_. Сейчас можно зайти на /items своего приложения и увидеть что получилось. Но нам пока рано останавливаться. 

Давайте добавим валидацию, которую я упустил, в файл **app/models/item.rb**
[cc lang="ruby"]
class Item < ActiveRecord::Base
  validates_presence_of :name, :description
  validates_uniqueness_of :name
end
[/cc]

Первой строкой мы добавляем, как, наверное, ясно из название, наличие и имени, и описания, второй мы проверям уникальность имени, чтобы не было двух айтемов с одним и тем же именем.


Далее давайте немного изменим нашу форму **app/views/items/_form.html.erb**
[cc lang="ruby"]
<%= form_for @item, :remote => true, :html => { :class => 'form-horizontal' } do |f| %>
  



  
    <%= controller.action_name.capitalize %> /Item

    


      <%= f.label :name, :class => 'control-label' %>
      


        <%= f.text_field :name, :class => 'text_field' %>
      


    



    


      <%= f.label :description, :class => 'control-label' %>
      


        <%= f.text_field :description, :class => 'text_field' %>
      


    



    


      <%= f.submit nil, :class => 'btn btn-primary' %>
      <%= link_to 'Cancel', items_path, :class => 'btn' %>
    


  
<% end %>
[/cc]

Небольшого изменения коснулась первоя строка, а именно добавился атрибут **:remote => true**, что сообщает рельсам задейстовать ujs (unobtrusive javascript). Также был добавлен тег с id item_errors, где мы будем выводить все ошибки валидации нашей модели.

Так как у нашего приложение будет только одна страница, можно смело избавляться от других вьюшек, заодно изменим наш контроллер **app/controllers/items_controller.rb**
[cc lang="ruby"]
class ItemsController < ApplicationController

  before_filter do
    @items = Item.all
    @item = Item.new
  end

  before_filter :find_an_item, :only => [:edit, :update, :destroy]
    
  def index
  end

  def create
    @item = Item.new(params[:item])
    if @item.save
      flash[:notice] = "Successfully created item."
      @items = Item.all
    end
  end

  def edit
    @items = Item.all
  end

  def update
    if @item.update_attributes(params[:item])
      flash[:notice] = "Successfully updated item."
    end
    @items = Item.all
    render :action => 'create'
  end

  def destroy
    @item.destroy
    flash[:notice] = "Successfully destroyed item."
    @items = Item.all
  end

  protected 
    def find_an_item
      @item = Item.find(params[:id])
    end

end
[/cc]

Тут я, конечно, перемудрил с before_filter'ами, но это то, как смог сделать за два часа. Как видите, контроллер шан стал намного полегче от того состояния, в котором он был после скаффолдинга: мы избавились от редиректов и вынести многое в before_filter'ы.

Далее я добавил в шаблон **app/view/layouts/application.html.erb** рядом с тем где выводится контент экшнов, вывод flash сообщения.
[cc lang="html"]



  





<%= yield %>
[/cc]

Это не весь шаблон, а только измененная его часть. 

Дело за вьюшками, создадим **_items.html.erb**
[cc lang="html"]



  
    


      ID
      Name
      Description
      Actions
    
  
  
    <% @items.each do |item| %>
      


        
<%= item.id %>

        
<%= link_to item.name, item_path(item) %>

        
<%= item.description %>

        

          <%= link_to 'Edit', edit_item_path(item), :class => 'btn btn-mini', :remote => true %>
          <%= link_to 'Destroy', item_path(item), :method => :delete, :confirm => 'Are you sure?', :class => 'btn btn-mini btn-danger', :remote => true %>
        

      
    <% end %>
  

[/cc]

Здесь нет тоже ничего необычного, кроме атрибута **:remote => true** у ссылок. Тем самым мы сообщаем рельсам, чтобы они инициировали ajax запрос, вместо обычного get'а.

Так как мы создали все необходимые partial'ы, мы модем изменить наш index view.
[cc lang="html"]


<%= render 'form' %>




# Items




<%= render 'items' %>


[/cc]

Который только собирает воедино нашу форму и список айтемов. Если вы запустите сейчас приложение и попытаетесь создать что-либо или отредактировать, получите множество сообщений об ошибках, связанных с отсутствием нужных шаблонов (если вы удалили все вьюшки, кроме **index.html.erb**).

Теперь переходим к магии: вмето того, чтобы отдавать html файлы, мы будем отдавать javascript, который rails вставит в нушное место. Вместо create.html.erb, у нас будет **app/views/items/create.js.erb**
[cc lang="ruby"]
<% if @item.errors.any? -%>
  $("#flash_notice").hide(300);
  $("#item_errors").html("<%= escape_javascript(@item.errors.full_messages.join '  
').html_safe %>");
  $("#item_errors").show(300);
<% else -%>
  $("#item_errors").hide(300);
  $("#flash_notice").html("<%= escape_javascript(flash[:notice]) %>");
  $("#flash_notice").show(300);
  $(":input[type=text]").val('');
  $("#items_list").html("<%= escape_javascript(render 'items') %>");
<% end -%>
[/cc]
Вкратце мы делаем следующее: если есть ошибки, то выводим их, если нет, то очищаем наши текстовые поля и обновляем список айтемов.

Самая длинная строчка в этом файле мне не очень понравилась, может можно как-то попроще, покороче?

Если запустить наше приложение, то вы сможете создавать новые записи, видеть сообщения об успешном или неуспешном (если возникли ошибки валидации) создании.

Также поступим с двумя оствшимися вьюшками **app/views/items/edit.js.erb**
[cc lang="ruby"]
$("#item_form").html("<%= escape_javascript(render 'form')%>");
[/cc]
Где мы просто обновляем наш список элементов.

и **app/views/items/destroy.js.erb**
[cc lang="ruby"]
$("#item_errors").hide();
$("#flash.notice").html("<%= escape_javascript(flash[:notice]) %>").show();
$("#items_list").html("<%= escape_javascript( render 'items') %>"); 
[/cc]

Вот и все :) Заранее хочу извиниться за сумбурное, местами беглое, изложение, но я сейчас нахожусь в ожидании ответа от работодателя и не могу спокойно сидеть на месте :).

Репозиторий доступен, как всегда на [github](https://github.com/vredniy/unobtrusive-javascript-rails), работающее приложение на [heroku](http://vredniy-unobtrusive.herokuapp.com/items)

P.S. деплой на heroku




  * запускаем _RAILS_ENV=production bundle exec rake assets:precompile_, чтобы скомпилировать наши стили и javascript'ы


  * заменяем в **Gemfile** sqlite3 на pg и далеем _bundle_


  * создаем приложение на heroku _heroku create --stack cedar_ (у вас приложение должно быть под git'ом и вы должны войти в heroku аккаунт)


  * _git push heroku master_ отправляем наше приложение за тысячи километров :)


  * на heroku сайт пишут, что нужно сделать _heroku run rake db:migrate
_, но у меня с этим не вышло, поэтому я запустил эту же команду в фоновом режиме _heroku run:detached rake db:migrate_


  * открываем приложение


  * наслаждаемся


  * и еще раз наслаждаемся :)


