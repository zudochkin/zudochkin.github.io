---
author: admin
comments: true
date: 2012-03-17 14:14:35+00:00
layout: post
slug: lets-game-with-ruby-and-php

permalink: /2012/03/lets-game-with-ruby-and-php

title: Поиграем? или совместное использование ruby и php
wordpress_id: 901
categories:
- php
- Ruby
- алгоритмы
tags:
- mongodb
- php
- redis
- ruby
---

Сегодня заметка будет посвящена теме далекой от веб-программирования, она будет посвящена немного алгоритмам, немного парсингу и немного Mongodb. Будем сегодня играть в игру.<!-- more -->

Часто по вечерам мы с женой играем в приложение на iPad'е под названием Словомания. ![словомания](http://vredniy.ru/wp-content/uploads/2012/03/slovomania.jpeg) Смысл в нем, как вы, наверное, заметили из названия, поиск слов. На входе матрица 4х4, можно начать из любой клетки и задействовать соседние по одному разу, чтобы отыскать имеющееся у них в словаре слово (в том словаре есть и наречия, и глаголы, и прилагательные, и конечно же существительные).

Задача: отыскать все слова. Но я немного облегчу себе задачу, потому как по примерным подсчетам в матрице 4х4 всевозможных комбинаций поиска слова около 16 миллионов, что даже по скромным подсчетам и при словаре в ~160 тысяч слов займет около 5 часов. В данной заметке мы будем использовать матрицу 3х3, что сокращает количество комбинаций до 8 с небольшим тысяч, что вполне приемлимо, но недостаточно быстро, для одной игры, которая длится около минуты. Да и плевать, для меня главное решить задачу, хоть и читерить в игре не получится ).

Для начала отыщем все возможные комбинации, для этого воспользуемся php-скриптом [поиска в ширину](/2011/02/depth-first-search-php/), немного его изменив для наших нужд. На выходе должны получить файл, где в каждой строке одна комбинация.

Цифры на матрице я обозначил от 1 до 9 (3х3), поэтому в выходном файле будут строки подобного вида - 1,2,3,6,5,4,7,8,9.

Но сначала нам пригодится небольшой скрипт, который поможет построить граф для таблицы 3х3.

[cc lang="ruby"]
for i in (1..9) do 
  print "%-2s" % i
  if i % 3 == 0 then print "\n" end
end
[/cc]

На выходе получим таблицу
[cc]
1 2 3
4 5 6
7 8 9 
[/cc]
которая поможет нам построить граф, ключами массива которого будут вершины "откуда", значениями - точки "куда", мы можем попасть из данной вершины. Из таблицы следует, что из точки 1, мы можем попасть в точки: 2,5 (по диагонали тоже можно) и 4. Так нам нужно описать каждую вершину графа.

[cc lang="php"]
array('2', '4', '5'),
  '2' => array('1', '3', '5', '4', '6'),
  '3' => array('2', '5', '6'),
  '4' => array('1', '2', '7', '8', '5'),
  '5' => array('1', '2', '3', '4', '6', '7', '8'),
  '6' => array('3', '2', '5', '9', '8'),
  '7' => array('4', '5', '8'),
  '8' => array('4', '6', '6', '7', '9'),
  '9' => array('5', '6', '8')
);

# циклы по всем вершинам графа,
# каждая вершина может быть начальной и конечной
for($i = 1; $i <= 9; $i++)  
  for($j = 1; $j <= 9; $j++)
    find_path($graph, (string) $i, (string) $j);

# избавляемся от дубликатов
$result = array_unique($paths);
echo "\n\nКоличество комбинаций: ", count($result), "\n";

# записываем результаты в файл
$fh = fopen('combinations.dat', 'a');
 fwrite($fh, implode("\n", $result));
fclose($fh);
[/cc]

В коде старался комментировать каждый момент, поэтому, думаю, должно быть понятно. На выходе у нас файлы _combinations.dat_ с 8175 строками из всевозможных комбинаций.

Теперь давайте подготовим наш словарь. Я скачал какой-то орфографический словарь, каждая строка которого предствляла собой следующее
[cc]жопа#ж`опа%ж`опа, -ы[/cc]
Я решил сохранить два первый слова, точнее слово и ударение (которое в данном проекте не используется). Первое слово отделено от второго #, второе от остального %. По мере парсинга словаря, мы будем записывать все слова в нашу mongo базу данных для быстрого поиска по ней в дальнейшем.

[cc lang="ruby"]
# coding: utf-8

require 'mongo'
require 'mongo_mapper'

MongoMapper.database = 'words'

class Word
  include MongoMapper::Document

  key :word
  key :word2
end

f = File.open('./dic.txt', 'r')

# цикл по всем строкам словаря
f.lines.each do |line|
  word, word2 = line.split('#') # отделяем первое слово
  word2 = word2.split('%')[0] # и второе
  puts "#{word} #{word2}\n"
  
  # и сохраняем его
  Word.create(
    :word => word,
    :word2 => word2
  )
end
[/cc]

Если вам лень запускать подобный скрипт, то в корне проекта лежит файл _words.json_, содержащий все слова, вы можете импортировать его в свою mongo базу командой
[cc]mongoimport -d DB -c COLLECTION words.json[/cc].
Далее, чтобы ускорить и без того медленное приложение, нужно создать индекс в базе на поле word, запускаем mongo-терминал, выбираем базу данных _use DB_ и пишем следующее
[cc]db.COLLECTION.ensureIndex({word: 1})[/cc], где DB - имя ваше базы данных, а COLLECTION - имя коллекции.

Итак, переходим к завершающей части - моменту истины, так сказать, поиску слов.

[cc lang="ruby"]
# encoding: utf-8

require 'mongo'
require 'mongo_mapper'

MongoMapper.database = 'words'

class Word
  include MongoMapper::Document

  key :word
  key :word2
end

# файл с полученным в предыдущих шагах комбинациями
COMBINATIONS_FILE = './combinations.dat'
# файл с результатами
OUTPUT_FILE = './results.dat'

# наше игровое поле
alphabet = %w{
  к о ж
  а п у
  а р р
}

# поиск текущего слова в словаре
def find_a_word(word)
  w = Word.where(:word => word).first
  if w
    return w.word
  else
    return nil
  end
end

# из комбинации, учитывая alphabet, составляем "слово"
def number_to_letters(alphabet, line)
  letters = line.split(',')
  word = []
  letters.each do |letter|
    word << alphabet[letter.to_i - 1]
  end
  word.join()
end

# основной цикл приложение
# проходимся во всему списку комбинаций
File.open(COMBINATIONS_FILE, 'r') do |f|
  f.lines.each do |line|
    # составляем слово
    n_t_l = number_to_letters(alphabet, line)
    # находим в бд
    result = find_a_word(n_t_l)
    unless result.nil? then # если нашли
      puts n_t_l # выводим его 
      File.open(OUTPUT_FILE, 'a') do |output| 
        output.puts n_t_l # и записываем в файл
      end 
    end
  end
end

[/cc]

После этого запускаем скрипт ruby find-words.rb, откидываемся на скпинку стула и ждем результат. Начнут появляться на экране слова, которые были найдены этим скриптом, возможны повторения, потому что одно слово может быть составлено из разных комбинаций.

Удачи в кодинге и побольше интересных задач, которые вам под силу решить :).

P.S.: репозиторий, как всегда, доступен на [github](https://github.com/vredniy/game-ruby-php).



* * *



update 25.03.2012
Все-таки намного интереснее заставить приложение работать как задумывалось, а именно, чтобы оно находило слова из матрицы 4х4. Для этого мною были предприняты следующие меры:




  * удаление лишних слов из словаря


  * оптимизация самого алгоритма


  * сужение диапазона поиска





### Удаление лишних слов из словаря


Пробежался я по словарю и понял, что там много слово, содержащих дефисы, скобки, пробелы. Некоторые слова состояли из одной-двух букв. Я решил с ними попрощаться, благо mongo поддерживает регулярные выражения.

[cc lang="ruby"]
MongoMapper.database = 'words'

class Word
  include MongoMapper::Document
  key :word
end

#s = Word.all(:word => /^.*[-А-Я].*$/).each do |word|
#s = Word.all(:word => /^.*[ё].*$/).each do |word|
#s = Word.all(:word => /^.*[\(\)\.\s+].*$/).each do |word|
s = Word.all(:word => /^.*[-,.:;].*$/).each do |word|
  word.destroy
end
[/cc]
Запускаем и удаляем ненужные слова, из 160 тысяч слов, останется 140 тысяч, неплохо :)



### Оптимизация самого алгоритма


Самый быстрый код - это мало кода. Поэтому максимально постараемся избавиться от методов. Выбросив все лишнее получим следующее.
[cc lang="ruby"]
# encoding: utf-8

require 'mongo'
require 'mongo_mapper'

MongoMapper.database = 'words'

class Word
  include MongoMapper::Document

  key :word
end

COMBINATIONS_FILE = './game.dat'

Alphabet = %w{
ё ж е т
у ч е р
к о ю т
р м п ь
}

start = _start = Time.now
counter = 0

combinations = []

File.open(COMBINATIONS_FILE, 'r') do |f|
    while (line = f.gets)
      counter += 1
      line = line.split(',')
      if (3..9).include? line.size
          w = Word.first(:word => line.map{ |l| Alphabet[l.to_i - 1] }.join )
          line = nil
          combinations << w.word unless w.nil?
      end

      if counter % 100_000 == 0
          p combinations.join ' ' unless combinations.empty?
          combinations = []
        p "#{Time.now - start} - #{counter}"
        start = Time.now
      end
    end
end

p (Time.now - _start)
[/cc]

Код получился намного "легче" и быстрее. 



### Сужаем диапазон поиска


Чтобы хоть как-то еще прибавить скорость, я сузил длину слова, теперь проверяются только строки от 3 до 9 символов, это тоже очень хорошо сужает конечную выборку.



### Обратная связь приветствуется


Если вы видите, что есть какие-то неточности или гипотезы по поводу оптимизации быстродействия кода, то всегда пожалуйста, я открыт для обсуждения всегда.

Не хочу быть излишне самоуверенным, но мне, кажется, что сейчас весь алгоритм упирается в "железо", точнее скорость считывания с диска. (К сожалению, у меня нет ssd диска, чтобы это проверить).

В заключение картинка с какой скоростью отрабатывает скрипт. Первое число на картинке - количество секунд на 100_000 строк, считанных из файла и проверенных с каждой строкой словаря, содержащего ~ 140.000 слов. Второе число - сколько строк уже просмотрено. Всего комбинаций получилось, если рассматривать поле 4х4 - 12.000.000 штук. :)

![ruby mongo files](http://vredniy.ru/wp-content/uploads/2012/03/optimization.jpeg)


* * *




### Обновление от 27.03.2012 (redis)


Пришла еще одна мысля по оптимизации данного приложения, вместо mongodb, я решил попробовать использовать redis - очень быстрое ключ-значение хранилище. 

Сначала нужно экспортировать базу mongo в redis, тут ничего сложного (с учетом того, что у вас и сам redis установлен, и redis gem).
[cc lang="ruby"]
require 'mongo'
require 'mongo_mapper'
require 'redis'

MongoMapper.database = 'words'

class Word
	include MongoMapper::Document

	key :word
end

redis = Redis.new(:host => "127.0.0.1", :port => 6379)
Word.all.each do |w|
  w = w.word
  unless /[-А-Я\(\)]/.match w
    redis.set("word:#{w}", w)
  end
end
[/cc]

Пробегаемся по всем словам из коллекции mongodb и сохраняем все слова в redis под ключом word:СЛОВО, чтобы не перепутать с другими ключами.

Теперь основной файл, только в качестве хранилища мы будем использовать redis.
[cc lang="ruby"]
# encoding: utf-8

require 'redis'
redis = Redis.new(:host => "127.0.0.1", :port => 6379)

COMBINATIONS_FILE = './game.dat'

Alphabet = %w{
ё ж е т
у ч е р
к о ю т
р м п ь
}

for i in (1..16) do 
    print "%-3s" % Alphabet[i-1] 
    if i % 4 == 0 then print "\n" end
end

start = Time.now
_start = Time.now
counter = 0

combinations = []
lines = []

File.open(COMBINATIONS_FILE, 'r') do |f|
  f.each do |line|
    counter += 1
    line = line.split(',')
    if (3..9).include? line.size
      w = redis.get("word:" + line.map{ |l| Alphabet[l.to_i - 1] }.join)
      line = nil
      combinations << w unless w.nil?
    end

    if counter % 100_000 == 0
      p combinations.uniq.join ' ' unless combinations.empty?
      combinations = []
      p "#{Time.now - start} - #{counter}"
      start = Time.now
    end
  end
end

p (Time.now - _start)
p counter
[/cc]

Вывод: приложение стало заметно быстрее (в 3-5 раз), скриншот прилагаю.
![redis-ruby-game](http://vredniy.ru/wp-content/uploads/2012/03/redis-game.png)

Надеюсь, смогу добиться скорость выполнения в раза больше, чем сейчас. Мне нужно чтобы приложение проходило по всем комбинациям, сравнивая со всем словами в словаре и тратило при этом меньше минуты, сейчас около 3х.


