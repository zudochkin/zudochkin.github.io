---
author: admin
comments: true
date: 2010-02-20 14:07:22+00:00
layout: post
slug: hello-world-and-zend-framework
title: '''Hello world'' и Zend Framework'
wordpress_id: 265
categories:
- Zend Framework
tags:
- hello world
- Zend Framework
---

[![Zend Framework для начинающих](http://vredniy.ru/wp-content/uploads/2010/02/logopost.png)](http://vredniy.ru/wp-content/uploads/2010/02/logopost.png)

Буквально несколько дней назад решил я освоить какой-нибудь PHP фрейворк, много статей и обзоров прочитать пришлось, чтобы остановиться все же на Zend Framework'е. Как пишут почти везде, что начало в нем самое сложное, дальше будет все намного понятнее. У меня для того, чтобы заработало первое приложение, которое выводит лишь строчку всем известную 'Hello world', ушел целый вечер. Хватит слов, ближе к делу.



Первым делом нам понадобиться конечно же web-сервер, подойдет и Denwer, и XAMPP, и Zend Server и думаю еще много чего подойдет нам.



Затем нам нужно скачать сам фреймворк, либо с [официального сайта](http://www.zend.com/en/community/downloads) (надо будет пройти регистрацию), либо с [моего сайта](/files/ZendFramework-1.10.1.zip) (последняя на момент публикации этой заметки версия).



Теперь построим структуру папок по определенной иерархии. Многие советуют папки с классами, папку с настройками выносить на уровень выше, чем находится Root директория web-сервера, но на многих хостингах не имеется возможности это реализовать, поэтому будет организовывать все в папке проекта. Вот что у нас должно получиться в итоге (картинка кликабельна). [![Zend Framework для начинающих](http://vredniy.ru/wp-content/uploads/2010/02/catalog-tree-150x150.gif)](http://vredniy.ru/wp-content/uploads/2010/02/catalog-tree.gif)



Осталось теперь разархивировать скачанный архив и из папки **library** скопировать папку **Zend** в папку **library** нашего проекта и переходить непосредственно к кодингу.
Нам потребуется, если мы работаем с сервером Apache, установленный и включенный модуль


**mod_rewrite**, чтобы все запросы к серверу обрабатывались одним файлов, в данном случае **index.php**. У меня проект называется **zend-hello**, если у вас отличается имейте это ввиду, чтобы не допустить ошибок при составлении путей.



Чтобы все запросы перенаправлялись одним файлов нам нужно указать это в файле **zend-hello/.htaccess**
`
RewriteEngine on
RewriteRule .* index.php
php_flag magic_quotes_gpc off
php_flag register_globals off
`



Первые две строчки включают **mod_rewrite** и перенаправляют все запросы соответственно. Третья и четвертая устанавливают нужные для безопасности параметры php.



И еще нам нужно сразу предусмотреть возможность отдавать сервером картинки и *.css файлы в обход **index.php**. Мы об этом позаботились и заранее разместили эти файлы в папке **public**. Создадим теперь файл **zend-hello/public/.htaccess** с таким содержимым.
`
RewriteEngine off
`


Тем самым отключив **mod_rewrite** для этой папки.



Переходим к главному файлу нашего проекта, который делает всю "черную" работу.
**zend-hello/index.php**
`
< ?php
    error_reporting(E_ALL && E_STRICT);
    set_include_path('.' . PATH_SEPARATOR . './library/'
            . PATH_SEPARATOR . './application/models/'
            . PATH_SEPARATOR . get_include_path());
    //включаем в путь папки из нашего проекта
    //PATH_SEPARATOR используется для разделения папок в пути
    //для *nix систем - это ':', для win - ';'
    include "Zend/Loader.php";
    Zend_Loader::loadClass('Zend_Controller_Front');

    //установка контроллеров
    $frontController = Zend_Controller_Front::getInstance();
    $frontController->throwExceptions(true);
    $frontController->setControllerDirectory('./application/controllers');

    //запускаем
    $frontController->dispatch();
`



Если что-то не понятно, а для новичка должно быть непонятно почти все, то не расставайтесь, все поймем дальше на примерах.
Важно пока лишь для нас знать, что закрывающийся php тег **НЕ надо ставить**



Теперь нам надо настроить контроллер, чтобы он отобразил нам заветную фразу.
**zend-hello/application/controllers/IndexController.php**
`
< ?php
class IndexController extends Zend_Controller_Action {
    function indexAction() {
        $this->view->title = "Hello world's page";
        $this->view->header = "Hello, world!";
    }
}
`



Здесь мы присваиваем переменной title название нашей страницы, а переменной header заветную фразу. Отсалось настроить виды (views), чтобы выводить информацию.



Файл **zend-hello/application/views/scripts/index/index.phtml**
`

    
    
        

# < ?php echo $this->escape($this->header); ?>


    

`



Если вы все сделали правильно и четко следовали моим инструкциям, то в конечном итоге, у вас должна появиться фраза, обещанная мной в самом начале.



Эта заметка не претендует звание подробного мануала по разработке высоконагруженных приложений на Zend Framework. Но в самом начале она может подтолкнуть на дальнейшее изучение данного фреймворка. Спасибо за внимание и удачи вам в веб-программировании и не только. 




*При написании заметки использовалась Netbeans IDE для PHP, скачать бесплатно которую можно [тут](http://netbeans.org/downloads/index.html)
