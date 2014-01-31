---
author: admin
comments: true
date: 2013-03-05 19:29:26+00:00
layout: post
slug: sinatra-datamapper-news-combinator

permalink: /2013/03/sinatra-datamapper-news-combinator

title: Парсим news.ycombinator.com или не рельсами едиными жив человек (Sinatra, DataMapper)
wordpress_id: 986
categories:
- Ruby
- программирование
tags:
- cron
- datamapper
- parser
- ruby
- sinatra
- whenever
---


Дело было вечером, делать было нечего и решил я написать небольшое приложение на Sinatra с Datamapper'ом. За идеей далеко ходить также не пришлось: решил написать небольшой "фильтратор" интересного для меня контента из новостей news.ycombinator.ru.<!-- more --> Не стал изобретать велосипед на этот раз и интересными буду считать новости, названия которых содержат определенные слова. Будем отображать список прочитанных и непрочитанных новостей. Список новостей каждый час будет обновляться по cron'у - вот и вся задача.

Начнем с реализации: для этого нам понадобится:

  * data_mapper с двумя адаптерами (sqlite3 для локального использование и postgresql для production'а)
  * sinatra
  * coffeeScript, хоть можно было и легко обойтись без него
  * slim в качестве шаблонизатора


Итак, поехали:

Gemfile:

{% highlight ruby %}
source 'https://rubygems.org'

gem 'sinatra'
gem 'data_mapper'

group :development do
  gem 'dm-sqlite-adapter'
  gem 'capistrano'
end

group :production do
  gem 'dm-postgres-adapter'
end

gem 'slim'
gem 'coffee-script'
gem 'whenever', :require => false

gem 'nokogiri'

gem 'unicorn'
{% endhighlight %}

В нем нет ничего необычного, добавляем необходимые гемы для разных сред.

Теперь самое интересное: основной файл приложения, который занимает больше всего места.

./app.rb

[cc lang="ruby"]
require 'sinatra'
require 'slim'

require 'coffee-script'
require 'data_mapper'

DataMapper::Property::String.length(400)

configure :development do
  DataMapper.setup(:default, 'sqlite3:./db/articles.db')
end

configure :production do
  DataMapper.setup(:default, 'postgres://deployer:funnydb@localhost/ycombinator')
end

class Article
  include DataMapper::Resource

  INTERESTING_KEYWORDS = %w(ruby rails coffee js javascript ember angular
    backbone tdd rspec shoulda gem unicorn nginx sinatra vim mac)

  property :id, Serial
  property :url, String, :unique_index => :u, :required => true, :format => :url
  property :title, String, :required => true, :index => true
  property :interesting, Boolean, :default => false
  property :read_at, DateTime
  timestamps :created_at, :updated_on

  def interesting?
    !!(title =~ Regexp.new(INTERESTING_KEYWORDS.join('|'), Regexp::IGNORECASE))
  end

  def self.interesting_to_me
    all(:interesting => true)
  end

  def self.unread
    all(:read_at => nil)
  end

  def self.read
    all(:read_at.not => nil)
  end

  def self.search(term='')
    if DataMapper.repository.adapter.options[:scheme] == 'sqlite3'
      all(:title.like => "%#{term.to_s}%")
    else
      all(:conditions => [ 'title ILIKE ?', "%#{term.to_s}%" ])
    end
  end
end

DataMapper.finalize
#DataMapper.auto_migrate!
DataMapper.auto_upgrade!

helpers do
  def do_process(scope=nil)
    @search_term = params[:term].nil? ? nil : params[:term]
    @articles = case scope
                when :all
                  Article
                when :all_read
                  Article.read
                when :all_unread
                  Article.unread
                else
                  Article.interesting_to_me.unread
                end.search(@search_term)
    slim :index
  end
end

get '/application.js' do
  coffee :application
end

post '/:id/read' do
  @article = Article.get(params[:id])
  @article.read_at = Time.now
  @article.save
end

get '/all' do
  do_process :all
end

get '/all/read' do
  do_process :all_read
end

get '/all/unread' do
  do_process :all_unread
end

get '/*' do
  do_process
end
[/cc]

А теперь немного комментариев:


  * **1-5 строки** - подключаем необходимые для работы файлы


  * **7 строка** - сообщаем DataMapper'y, что длина строки (String) не 80 символов, а 400, 255 не хватает.


  * **9-15** - конфигурируем два адаптера: один для разработки, другой для продакшна.


  * **17-53**



    * **20,21** - объявляем интересные мне ключевые слова


    * **23-28** - описываем все поля, которые будут в нашей модели


    * **30-32** - метод interesting? определяет по заголовку новости интересна она мне или нет


    * **34-40** - несколько используемых в приложении scope'ов


    * **46-52** - метод search (из-за того, что в Postgresql like учитывает регистр букв, пришлось переписать оператор поиска на ilike, который этого не делает)




  * **60-73** - объявляем метод, который является "сердцем" и в зависимости от параметра заполняет коллекцию определенными статьями и рендерит вьюху ./views/index.slim



  * **76-78** - рендерим coffeeScript, которые делает следующее, если мы кликам по новости, то отправляем ajax post запрос и помечаем новость как прочитанную (read_at = Time.now)



  * **80-84** - сам метод, который помечает новость прочитанной при post запросе


  * **86-100** - разные коллекции (все, прочитанные, непрочитанные и т.д.)


теперь Rakefile, который будет парсить news.ycombinator.com каждый час

[cc lang="ruby"]
require './app'

require 'nokogiri'
require 'open-uri'

desc 'Parse all articles'
task :parse do
  doc = Nokogiri::HTML(open('http://news.ycombinator.com/'))
  links = doc.css('td.title a')
  next_page_link = links.pop

  links.each do |link|
    href = link[:href]
    text = link.children.text

    unless Article.first(:url => href)
      Article.create(:url => href, :title => text)
    end
  end

  puts Time.now.to_s
end

desc 'Update Interesting tasks'
task :update_interesting do
  Article.all.each do |a|
    a.update(:interesting => a.interesting?)
    puts "#{a.id} updated"
  end
end
[/cc]

В нем всего две задачи: первая - парсит новости, вторая нужна для того, что если вдруг изменися интересные ключевые слова, то вы сможете легко обновить список инетересных вам новостей.

Файл, который отвечает за частоту выполнения определенных тасков ./config/shedule.rb
[cc lang="ruby"]
set :output, '/home/deployer/projects/ycombinator/shared/log/shedule.log'

job_type :rake, "cd :path && RACK_ENV=:environment bundle exec rake :task --silent :output"

every :hour do
  rake 'parse'
end
[/cc]

В первой строчке я указываю путь до файла с логами, чтобы каждый раз при запуске rake task'а в конец добавлялось время последнего обновления. В блоке с every можно очень гибко указать как часто выполняться, смотрите документацию к гему whenever.

Также я добавил несколько строк к файлу, выполняющего деплой из [Разворачиваем Rails приложение вместе с Capistrano](/unicorn-rbenv-nginx-postgresql/). ./config/deploy.rb

[cc lang="ruby"]
...
set :application, 'ycombinator'
set :whenever_command, "bundle exec whenever"
require "whenever/capistrano"
...
[/cc]

Теперь мы можем запустить обновления cron'а deployer'а командой `cap whenever:update_crontab`

После ее запуска вы можете проверить, что вышло, обновился ли cron, запустив на сервере, список cron задач: `crontab -l`

Без комментариев оставлю вьюхи, но текст их приведу.

./views/application.coffee
[cc lang="coffeescript"]
$ ->
  $(".article").click ->
    $.post "/" + $(this).attr("id") + "/read", ->
    $(this).parent('li').remove()
[/cc]

./views/layout.slim
[cc lang="ruby"]
doctype html

head
  title Ycombinator
  script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"
  script src="/application.js"

body
  a{ href="/all" } All
  br
  a{ href="/all/read" } All read
  br
  a{ href="/all/unread" } All unread
  br
  a{ href="/" } Home
  hr
  == yield
[/cc]

И наконец, ./views/index.slim
[cc lang="ruby"]
h1
  ' Unread articles
  - if @search_term
    = "searched by: '#{ @search_term }'"

p Search form
form{ method="get" action="" }
  input{ type="text" name="term" value="#{ @search_term }" }
  input{ type="submit" value="Find" }

- if @search_term
  a{ href="/" } Home

ul
  - @articles.each do |article|
    li
      a{ href="#{ article.url }" id="#{ article.id }" class="article" target="_blank" }
        = article.title
[/cc]

Получислось такое незамысловатое и некрасивое приложение :).

![Sinatra DataMapper](http://vredniy.ru/wp-content/uploads/2013/03/Screen-Shot-2013-03-05-at-10.33.43-PM.png)

Вместо заключения: чтобы мозги не были напичканы только рельсами (читать как одним фреймворком), мне кажется, необходимо покидать зону комфорта и писать небольшые приложения для души на смежных технологиях. Скажу честно, для того, чтобы реализовать это несложное приложение у меня ушло масса времени на чтение мануалов к Sinatra, DataMapper'у, нежели на написание кода. Но мне понравилось, практической ценности, конечно, приложение почти не имеет, но мозги размялись однозначно. Разминай мозги, коллега :)
