---
author: admin
comments: true
date: 2012-02-25 13:24:37+00:00
layout: post
slug: arbor-js-ruby-sinatra

permalink: /2012/02/arbor-js-ruby-sinatra

title: 'arbor.js: красивые графы на стороне клиента с помощью Ruby'
wordpress_id: 892
categories:
- Ruby
tags:
- arborjs
- mongodb
- ruby
- sinatra

keywords: "arbor js twitter, arbor.js ruby, ruby twitter gem, twitter gem,arborjs,mongodb,ruby,sinatra"
description: "Сегодня мы займемся рисованием интерактивного графа на стороне клиента. На графе, чтобы далеко не ходить в поисках данных, будет изображать тех, кого читаем в твиттере."
---

Сегодня мы займемся рисованием интерактивного графа на стороне клиента. На графе, чтобы далеко не ходить в поисках данных, будет изображать тех, кого читаем в твиттере. <!--more-->



### Требования

  * Граф должен выглядеть привлекательно
  * При наведении на вершину графа мы должны получать дополнительную информацию, в данном приложении - это имя твиттерянина, картинка профиля, количество дней прошедших с регистрации и дата регистрации
  * Backend'ом будет выступать Mongo с Rack прослойкой в виде Sinatra

### Этапы разработки

  * Определяемся с инструментарием
  * Немного проектирования
  * Регистрируем наше Twitter приложение
  * Сохраняем всех наших друзей в Mongo
  * Шаблоны, скрипты и стили на клиенте


##### Определяемся с инструментарием

В пункте с требованиями я уже сказал, что мы будем использовать Mongo в связке MongoMapper'ом, для удобного хранения наших друзей с нужной нам информацией (именем, датой регистрации, картинкой и пр.). Для удобного взимодействия с Twitter API, используем gem Twitter, который предоставляет полный контроль через Twitter приложение.

##### Немного проектирования

Что мы имеем? На главной странице (/) будет отображаться сам граф, данные по средством ajax-запроса к серверу будут приходить от /json (данные будем возвращать в json'е). Обновлять список пользователей мы будем по адресу /friends (получаем список всех пользователей, которых мы читаем и запрашиваем их дополнительные данные, сохраняя все в базу).


##### Регистрируем наше Twitter приложение

{% img image /images/posts/2012-02-arbor-js-ruby-sinatra/twitter-application-800x614.png %}Т.к. все взаимодействие с Twitter API происходит через Twitter приложение, нам необходимо его создать, для этого заходим в [раздел для разработчиков](https://dev.twitter.com/apps/new). Заполняем обязательные поля и нажимаем кнопку "Create your twitter application". Спустя несколько секунд мы оказываемся на странице созданного приложения. Прокрутив вниз, мы увидим кнопку "Create my access token", нажав на которую мы заполним Access token и Access token secret для нашего приложения. Полученные ключи можно лицезреть на вкладке "OAuth tool" под заголовком "OAuth Settings". Все четыре строчки нам понадобятся чуть позже.



##### Сохраняем всех наших друзей в Mongo

Итак, приступим непосредственно к написанию кода. К этому моменту у вас должно быть рабочее Twitter приложение, установленный Mongo (освещение установки выходит за рамки данной заметки).

Вначале, как всегда, стандартный **./config.ru**
``` ruby
require './twitter.rb'
run Sinatra::Application
```

Дальше создадим **./Gemfile**, который будет содержать все гемы, которые мы используем

``` ruby
source :rubygems
gem 'sinatra'
gem 'twitter'

gem 'mongo'
gem 'bson_ext'
gem 'mongo_mapper'
```

После этого запустим _bundle install_, что создаст нам Gemfile.lock

А теперь наше приложение **./twitter.rb**

``` ruby
# encoding: UTF-8
require 'sinatra'
require 'twitter'

require 'mongo'
require 'mongo_mapper'
```

Подключаем необходимые гемы.

``` ruby
Twitter.configure do |config|
	config.consumer_key = 'YOUR_CONSUMER_KEY'
	config.consumer_secret = 'YOUR_CONSUMER_SECRET'
	config.oauth_token = 'YOUR_ACCESS_TOKEN'
	config.oauth_token_secret = 'YOUR_ACCESS_SECRET'
end
```

Конфигурируем twitter (заполните строки выше теми данными, которые вы получили после регистрации twitter приложения), привязывая его к нашему приложению.

Теперь настроим MongoMapper, указав ему с какими данными мы хотим работать.

``` ruby
MongoMapper.database = 'users'

class User
  include MongoMapper::Document

  key :screen_name
  key :profile_image_url
  key :followers_count
  key :statuses_count
  key :name
  key :friends_count
  key :lang
  key :url
  key :created_at, DateTime

  # id location notifications profile_image_url profile_image_url_https profile_background_color
  # followers_count default_profile time_zone is_translator utc_offset profile_background_image_url
  # statuses_count profile_link_color name friends_count listed_count protected
  # profile_use_background_image profile_background_image_url_https contributors_enabled lang
  # profile_text_color follow_request_sent description profile_sidebar_border_color show_all_inline_media
  # url verified default_profile_image created_at profile_background_tile favourites_count id_str
  # following profile_sidebar_fill_color geo_enabled screen_name
end
```

Сначала указываем базу данных ('users'), дальше описываем поля. Закомментированные строки - это дополнительные поля, которые вы можете использовать при получении данных твиттер-пользователя.

Пришло время написать экшн, которые сохранит всех наших друзей в базе Mongo и здесь нет ничего сложного.

``` ruby
get '/friends' do
  User.collection.remove # очищаем коллекцию

  friends = Twitter.friend_ids # получаем список всех друзей
  friend_ids = friends.ids

  friend_ids.each do |f| # проходимся по каждому и
    begin
      twi_user = Twitter.user(f) # получаем данные, отправляет запрос на сервер Twitter'а
      User.create( # заполняем поля
        :screen_name => twi_user.screen_name,
        :profile_image_url => twi_user.profile_image_url,
        :followers_count => twi_user.followers_count,
        :statuses_count => twi_user.statuses_count,
        :name => twi_user.name,
        :friends_count => twi_user.friends_count,
        :lang => twi_user.lang,
        :url => twi_user.url,
        :created_at => twi_user.created_at
      )
    rescue Exception => e # обрабатываем исключение
      puts e.message
      puts e.backtrace.inspect
    end
  end

  @users = User.all # получаем всех сохраненных пользователей
  erb :friends # отображаем их
end
```

Чтобы наш экшн отработал, на необхомо создать файл **./views/friends.erb** со следующим содержимым:
``` ruby
<% @users.each do |u| %>
  <h5><%= u.name %> - <%= u.screen_name %></h5>
  <img src="<%= u.profile_image_url %>">
  <p><%= u.created_at %></p>
  <hr>
<% end %>
```

Который не делает ничего более того, что отображает в цикле всех пользователей с картикой профиля. (Это нужно для того, чтобы убедиться в том, что все работает как ожидалось).



##### Шаблоны, скрипты и стили на клиенте


Сначала создадим шаблон для всех наших представлений, **./views/layout.erb**:
``` html
<html>
<head>
   <title>Twitter friends (arbor.js + Ruby)</title>
   <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
   <script src="/js/arbor.js"></script>
   <script src="/js/application.js"></script>
   <link rel="stylesheet" type="text/css" href="/styles/application.css">
</head>
<body>
   <%= yield %>
</body>
</html>
```

В котором мы подключаем библиотеку jQuery через Google CDN, файл arbor.js, который вы можете скачать [отсюда](https://github.com/samizdatco/arbor) (данная библиотека используется нами для визуализации графа на стороне клиента).

Файл arbor.js необходимо положить в папку **./public/js/arbor.js**

Далее напишем последний скрипт, который будет обрабатывать события отрисовки, drag&drop;'а и загрузки данных ajax'ом. (За основу взят пример с сайта arbor.js). **./js/application.js**

``` javascript
(function($){
    var Renderer = function(canvas){
      var canvas = $(canvas).get(0)
      var ctx = canvas.getContext("2d");
      var particleSystem
      var dom = $(canvas)

      var that = {
        init:function(system){
          particleSystem = system
          particleSystem.screenSize(canvas.width, canvas.height)
          particleSystem.screenPadding(80) // leave an extra 80px of whitespace per side
          that.initMouseHandling()
        },

        redraw:function(){
          ctx.fillStyle = "white"
          ctx.fillRect(0,0, canvas.width, canvas.height)

          particleSystem.eachEdge(function(edge, pt1, pt2){
            ctx.strokeStyle = "rgba(88,0,0, .133)"
            ctx.lineWidth = 1
            ctx.beginPath()
            ctx.moveTo(pt1.x, pt1.y)
            ctx.lineTo(pt2.x, pt2.y)
            ctx.stroke()
          })

          particleSystem.eachNode(function(node, pt){
            var w = 2;   //ширина квадрата
            ctx.fillStyle = (node.name == 'Я') ? 'yellow' : node.data.color
            ctx.fillRect(pt.x-w/2, pt.y-w/2, w,w); //рисуем
            ctx.fillStyle = (node.name == 'Я') ? 'green' : node.data.color; //цвет для шрифта
            ctx.font = (node.name == 'Я') ? 'bold 18px sans-serif' : 'italic 13px sans-serif'; //шрифт
            ctx.fillText (node.name, pt.x+8, pt.y+8); //пишем имя у каждой точки
          })
        },

        initMouseHandling:function(){
          var dragged = null;
          var handler = {
            moved: function(e) {
                var pos = $(canvas).offset();
                _mouseP = arbor.Point(e.pageX-pos.left, e.pageY-pos.top)
                nearest = particleSystem.nearest(_mouseP);
                if (!nearest.node) return false
                $('#status h1').text(nearest.node.data.full_name)
                $('#status .image').html($('').attr('src', nearest.node.data.image));
                $('#status .ago').html(nearest.node.data.days + '
' + nearest.node.data.created_at);

            },

            clicked:function(e){
              var pos = $(canvas).offset();
              _mouseP = arbor.Point(e.pageX-pos.left, e.pageY-pos.top)
              dragged = particleSystem.nearest(_mouseP);

              if (dragged && dragged.node !== null){
                dragged.node.fixed = true
              }

              $(canvas).bind('mousemove', handler.dragged)
              $(window).bind('mouseup', handler.dropped)
              that.redraw();
              return false
            },
            dragged:function(e){
              var pos = $(canvas).offset();
              var s = arbor.Point(e.pageX-pos.left, e.pageY-pos.top)

              if (dragged && dragged.node !== null){
                var p = particleSystem.fromScreen(s)
                dragged.node.p = p
              }

              return false
            },

            dropped:function(e){
              if (dragged===null || dragged.node===undefined) return
              if (dragged.node !== null) dragged.node.fixed = false
              dragged.node.tempMass = 1000
              dragged = null
              $(canvas).unbind('mousemove', handler.dragged)
              $(window).unbind('mouseup', handler.dropped)
              _mouseP = null
              return false
            }
          }
          $(canvas).mousedown(handler.clicked);
          $(canvas).mousemove(handler.moved);

        },

      }
      return that
    }

    $(document).ready(function(){
      var sys = arbor.ParticleSystem(1000, 100, 0.5) // create the system with sensible repulsion/stiffness/friction

      sys.renderer = Renderer("#viewport") // our newly created renderer will have its .init() method called shortly by sys...

      sys.addNode('Я');
      sys.addNode('Кто-то другой');
      $.getJSON('/json', function(response) {
        $.each(response.nodes, function(i,node) {
          sys.addNode(node.name, {
            days: node.days,
            image: node.image,
            full_name: node.full_name,
            color: node.color,
            created_at: node.created_at
          });
          sys.addEdge(node.name, 'Я');
        });
      });

    })

  })(this.jQuery)
```

В строках 106-117 мы получаем данные с сервера, заполняя каждую ноду именем и данными, которые будет использовать далее.

Отрисовка происходит в строках 29-36, а обработка события перемещения мыши в 42-51, где мы сохраненные в ноде данные отображаем (для этого используем возможности движка по нахождению ближайшей к указателю мыши ноды).

Шаблон для отображения главной страницы **./views/index.erb**
``` html
<div id="status">
   <div class="image"></div>
   <h1>Я</h1>
   <span class="ago"></span>
</div>
<p class="clear"></p>
<canvas id="viewport" width="1000" height="700"></canvas>
```

В нем мы создаем статусбар и канвас, в котором и будет отображать наш граф и обрабатывать события.

Добавим к нашему **./twitter.rb** последний штрих, точнее два, вывод главной страницы и отправка данных ajax'ом.
``` ruby
get '/' do
	erb :'index'
end

get '/json' do
  content_type :json
  users = []
  User.all.each do |u|
    users << {
      :name => u.screen_name,
      :full_name => u.name,
      :days => (DateTime.now-DateTime.strptime(u.created_at.to_s, '%Y-%m-%d')).to_i,
      :image => u.profile_image_url,
      :color => "#%06x" % (rand * 0xffffff),
      :created_at => u.created_at
    }
  end

  {
    nodes: users
  }.to_json
end
```

В методе _/json_ мы проходимся по списку всех пользователей, которых мы сохранили в методе _/friends_ и собираем json объект, который и выводим с заголовком _content_type :json_

Единственное, что может показаться странным, так это конструкция _(DateTime.now-DateTime.strptime(u.created_at.to_s, '%Y-%m-%d')).to_i_, которая считает разницу в днях между днем регистрации пользователя и текущим днем, и конструкция _"#%06x" % (rand * 0xffffff)_, которая генерирует случайный цвет в формате #1234ab, остальные строки, надеюсь, прозрачны и легки к пониманию.



### Демонстрация и заключение

Видео, демонстрирующее возможности разработонного приложения смотрите ниже, из него, я думаю, все станет понятно, что нельзя сказать о кучи текста и кода. Как говорят, лучше один раз увидеть, чем несколько раз прочитать.


<iframe width="560" height="315" src="//www.youtube.com/embed/-SX1AVO_bj0" frameborder="0" allowfullscreen></iframe>

В качестве заключения хочу сказать следующее: arbor.js - очень мощная библиотека для отрисовки графов на стороне клиента, которая загружает процессор на 100%, но позволяет, чертовка, визуализировать графы наглядно и красиво. Еще в данной заметке мы с вами познакомились очень поверхностно с Twitter гемом, который позволяет создавать приложения для взаимодействия с Twitter API. Надеюсь, потраченное на прочтение данной заметки, не было бесполезным :) и до новых встреч.

P.S.: ссылка на [репозиторий на Github](https://github.com/vredniy/arbor-twitter-sinatra)
