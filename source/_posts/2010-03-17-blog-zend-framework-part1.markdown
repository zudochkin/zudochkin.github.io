---
author: admin
comments: true
date: 2010-03-17 17:23:49+00:00
layout: post
slug: blog-zend-framework-part1
title: 'Пишем блог на Zend Framework и Smarty: часть 1 - подготовка'
wordpress_id: 425
categories:
- php
- Zend Framework
- программирование
tags:
- blog
- smarty
- Zend Framework
---

Сегодня мы начнем создавать свой полноценный блог с комментариями; писать в него могут сразу несколько человек по своими именами.
Мы будем использовать последнюю на сегодняшний день версию Zend Framework'а. Еще один плюс этого блога в том, что мы сделаем его таким, чтобы он смог запускаться легко на локальном компьютере и на выделенном хостинге, что не просто сделать, следую инструкции разработчиков. <!-- more -->
Что нам для этого понадобится.
1) Конечно же веб-сервер (как установить его вы можете прочесть здесь). Хотя с лихвой хватит XAMPP установленного в корень какого-нибудь диска.
2) Zend Framework последней версии, на момент написания последней версией был 1.10.2, скачать вы можете с официального сайта.
3) Среда разработки, я использую для этих целей бесплатную NetBeans, но вы можете писать весь код и в блокноте, это не возбраняется.
4) Немного терпения, потому как с первого раза может что-то не получиться.

Начнем с организации каталога с проектом. У меня установлен XAMPP на диск D, исходя из этого и будут организовываться пути, будьте внимательнее.

[![пишем блог на zend framework](http://vredniy.ru/wp-content/uploads/2010/03/zf-blog-150x150.gif)](http://vredniy.ru/wp-content/uploads/2010/03/zf-blog.gif)

Сначала перенаправляем все запросы в файл index.php, находящемуся в корне нашего проекта.
**.htaccess**
[cc lang="php"]
RewriteEngine on
RewriteCond %{SCRIPT_FILENAME} -f
RewriteCond %{SCRIPT_FILENAME} -d
RewriteRule ^(.*)$ index.php/$1
[/cc]
Вторая строка включает перенаправление, если ссылка не на реальный файл, находящийся на сервере, третья строка - для каталогов.

Теперь самый важный файл в нашем проекте
**index.php**
[cc lang="php"]
defined('APPLICATION_PATH')
        || define('APPLICATION_PATH', realpath(dirname(__FILE__) . '/application'));

defined('APPLICATION_ENV')
        || define('APPLICATION_ENV', 'development');

set_include_path(implode(PATH_SEPARATOR, array(
        realpath(APPLICATION_PATH . '/../library'),
        get_include_path(),
)));

require_once 'Zend/Application.php';
require_once 'Zend/Loader/Autoloader.php';

Zend_Loader_Autoloader::getInstance();
[/cc]
В этом участке мы определяем константы, если они не установлены, первая это путь к /application, вторая - переменная среды. Дальше мы включаем в путь папку library, в которой находится Zend Framework (скопируйте туда папку Zend, если вы не сделали этого раньше, у вас должно получиться что-то вроде ./library/Zend).

В последующих 2х строчках подключаем 2 класса, один главный, который и запускает приложение, второй для того, чтобы не подключать нужные классы во время разработки. Zend_Loader_Autoloader - позоволяет это делать автоматически.

Пока оторвемся от главного файла и напишем файл конфига, из которого мы и будем по мере надобности брать данные (подключение к базе данных, пути и прочее).

[cc lang="php"]
[development]
database.type = pdo_mysql
database.username = blog
database.password = blog
database.dbname = blog

paths.base = D:\xampp\htdocs\blog
paths.templates = D:\xampp\htdocs\blog\templates
paths.data = D:\xampp\htdocs\blog\data

logging.file = D:\xampp\htdocs\blog\debug.log

phpSettings.display_startup_errors = 1
phpSettings.display_errors = 1

bootstrap.path = APPLICATION_PATH "/Bootstrap.php"
bootstrap.class = "Bootstrap"

resources.frontController.controllerDirectory = APPLICATION_PATH "/controllers"
[/cc]

Т.к. только разрабатываем свое приложение, мы используем раздел 'development', следующие 4 строчки задают как подключаться с базе данных, рекомендую вам тоже создать базу данных blog на localhost, пользователь и пароль blog.

Следующие строчки **paths.*** задают абсолютные пути к папке с данными, о которой мы поговорим позже, базовый путь, где находится наш проект и путь к шаблонам Smarty, о которых тоже позже.

Строки **bootstrap.*** указывают расположения класса инициализации, которым мы воспользуемся чуть позже.

Последняя строка указывает местонахождения контроллеров.

Давайте теперь создадим класс, которым будем расширять все наши контроллеры, т.е. вмето того, чтобы писать 
[cc lang="php"]class IndexController extends Zend_Controller_Action[/cc]
будем писать [cc lang="php"]class IndexController extends CustomControllerAction[/cc], чтобы иметь доступ ко многим конфигурационным данным.

Класс ./library/CustomControllerAction
[cc lang="php"]
db = Zend_Registry::get('db');
    }
}
[/cc]
Как вы помнить из [Hello world на Zend Framework](/2010/02/hello-world-and-zend-framework/) в конце мы закрывающийся тег php не ставим.

Класс **./application/Bootstrap.php** пока пуст
[cc lang="php"]
./index.php**
[cc lang="php"]
require_once 'CustomControllerAction.php';

$config = new Zend_Config_Ini('/application/configs/application.ini', 'development');
Zend_Registry::set('config', $config);

$params = array('host' => $config->database->hostname,
        'username' => $config->database->username,
        'password' => $config->database->password,
        'dbname'   => $config->database->database);

$db = Zend_Db::factory($config->database->type, $params);
Zend_Registry::set('db', $db);

$application = new Zend_Application(
        APPLICATION_ENV,
        APPLICATION_PATH . '/configs/application.ini'
);
[/cc]
Здесь мы подключаем класс, которым и расширяем Zend_Controller_Action, дальше мы создаем конфиг из нашего конфигурационного файла, указывая вторым параметром имя секции, в нашем случаем **[development]**
И сохраняем этот конфиг в реестре Zend Framework, чтобы иметь доступ к нему из любого места нашего приложения.

Массив $params содержит данные для подключения к базе данных, в данном случае MySQL, но мы с легкостью сможем заменить ее на что-нибудь другое.
В переменной $db идентификатор соединения, который мы тоже сохраняем в реестр.

Дальше мы передаем параметры нашему приложению и запускаем

[cc lang="php"]$application->bootstrap()
        ->run();[/cc]


Если вам захочется запустить приложение, то вас ждет неудача, наберитесь терпения, все будет работать, я вам обещаю =)

Т.к. мы вместо встроенного шаблонизатора в Zend Framework, будем использовать Smarty, который вы можете на официальном сайте smarty.net, оттуда нам лишь понадобится лишь папка libs, из который мы копируем все в папку с нашими библиотеками **./library/Smarty**.

Теперь мы напишем свой класс Templater, расширяющий **Zend_View_Abstract**, который будет отвечать за отображение информации на нашем блоге.
Файл **./library/Templater.php**
[cc lang="php"]
_engine = new Smarty();
        $this->_engine->template_dir = $config->paths->templates;
        $this->_engine->compile_dir  =
                sprintf('%s/tmp/template_c', $config->paths->data);
        $this->_engine->plugins_dir = array($config->paths->base
                        . '/include/Templater/plugins', 'plugins');
    }

    public function getEngine() {
        return $this->_engine;
    }

    public function  __set($key,  $value) {
        $this->_engine->assign($name, $value);
    }

    public function __get($key) {
        return $this->_engine->get_template_vars($key);
    }

    public function __isset($key) {
        return $this->_engine->get_template_vars($key) !== null;
    }

    public function __unset($key) {
        $this->_engine->clear_assign($key);
    }

    public function assign($spec, $value = null) {
        if (is_array($spec)) {
            $this->_engine->assign($spec);
            return;
        }
        $this->_engine->assign($spec, $value);
    }

    public function clearVars() {
        $this->_engine->clear_all_assign();
    }

    public function render($name) {
        return $this->_engine->fetch(strtolower($name));
    }

    public function _run() {

    }
}
[/cc]

**getEngine()** - возвращает экземпляр класса Smarty. Т.к. этот метод может вызывать несколько раз, нам необходимо кэшировать объект Smarty, чтобы он не создавался более одного раза. Это происходит в конструкторе этого класса.

**__set()** - присваиваем переменную шаблону. Кто знаком со Smarty $smarty->assign('key', 'val');

**_get()** - ранее присвоенную переменную возвращаем.
**
render()** визиализируем шаблон, вызывая метод fetch().

Теперь наш класс нужно прикрутить к нашему приложению. Для этого в файле **./index.php** перед 

**$application->bootstrap()->run();**
допишем 
[cc lang="php"]
require_once 'Templater.php';
$vr = new Zend_Controller_Action_Helper_ViewRenderer();
$vr->setView(new Templater());
$vr->setViewSuffix('tpl');
Zend_Controller_Action_HelperBroker::addHelper($vr);
[/cc]

Создадим шаблон для вывода информации **./templates/index/index.tpl**
[cc lang="php"]
{include file='header.tpl'}
Мой первый блог
{include file='footer.tpl'}
[/cc]
Шаблоны для шапки и футера соответственно:
**./templates/header.tpl**
[cc lang="php"]


    
    
        


[/cc]
**./templates/footer.tpl**
[cc lang="php"]
        


    

[/cc]

Все, что сейчас умеет делать наше приложение, это выводить фразу "Мой первый блог", но на самом деле мы сделали очень много, дальше будет еще интереснее, подписывайтесь на [rss](/rss), чтобы не пропустить создание блога на Zend Framework и Smarty, вопросы, пожелания и дополнения приветствуются.
