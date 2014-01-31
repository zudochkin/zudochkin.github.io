---
author: admin
comments: true
date: 2011-06-11 16:15:56+00:00
layout: post
slug: mvc-framework-php

permalink: /2011/06/mvc-framework-php

title: Пишем свой первый фреймворк на PHP
wordpress_id: 731
categories:
- OOP
- php
- программирование
tags:
- framework
- mvc
- OOP
- php
---


Привет тебе, коллега-разработчик или просто случайно зашедший на мой блог посетитель. В сегодняшней заметке я хочу рассказать вам о том, как прошел у меня вчерашний вечер (нет, нет, тут не будет ничего личного, аля покатался на роликах, попил пивка в парке). Как и у каждого веб-программиста рано или поздно возникает идея создать свой велосипед, пусть и с квадратными колесами и вместо руля торчащий штырь. Вчерашним велосипедом для меня стал легкий простой фреймворк, хотя с натяжкой его можно так назвать, но есть несколько моментов в нем, которые могут послужить отправной точкой для создание нечто большего. Ну обо всем по порядку. 




### Структура папок


Так как на меня большое влияние в последнее время оказал Zend Framework, структура папок, а также несколько еще штук будут очень похожи.

В корне мы имеем две папки **app** и **library**. Также в корне есть два файла (опять же все как у ZF) .htaccess (будет перенаправлять все запросы в index.php) и сам index.php, который эти самые запросы принимать будет.
<!-- more -->


### .htaccess и index.php


![vredniy-simple-mvc](http://vredniy.ru/wp-content/uploads/2011/06/vredniy-simple-mvc-150x150.png)**.htaccess** полностью взял с ZF
[cc]
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} -s [OR]
RewriteCond %{REQUEST_FILENAME} -l [OR]
RewriteCond %{REQUEST_FILENAME} -d
RewriteRule ^.*$ - [NC,L]
RewriteRule ^.*$ index.php [NC,L]
[/cc]
Мне этого было более, чем достаточно. По сути этот .htaccess перенаправляет все запросы, если они не ведут на существующую директорию или файл нашему index.php.

**index.php**
[cc lang="php"]
run();
[/cc]



### папка library


В данной папке будет пока две директории MVC и Smarty, первая - это наш фреймворк, вторая как вы уже догадались - шаблонизатор :)

Рассмотрим поподробнее папку **MVC**
**MVC_Autoload_Autoloader**
[cc lang="php"]
MVC_Autoload_Resource** - будет отвечать за подгрузку контроллеров, правда получился он не особо гибким и пути были жестко прописаны в код (очень не хорошо получилось).

[cc lang="php"]
'@App_Controller_(\w+)Controller@',
    );

	// метод без проверок, т.к. фреймворк создавался для ознакомительных целей
    public static function autoload($className)
    {
		// пробегаемся по всем ресурсам
		// и смотрим на что заканчивает класс
        foreach(self::$_resources as $resource => $path) {
			// если на то, что нужно подгружаем его
            if (ucfirst($resource) == substr($className, -strlen($resource))) {
                if (preg_match(self::$_resources[$resource], $className, $matches)) {
                    self::load($matches[1], $resource);
                }
            }
        }
    }

    protected static function load($name, $resourceType)
    {
        switch ($resourceType) {
            case 'controller':
                require_once APPLICATION_PATH . DIRECTORY_SEPARATOR
                        . 'controllers' . DIRECTORY_SEPARATOR
                        . "{$name}Controller.php";


                break;

            default:
                break;
        }
    }

}
[/cc]



### Стукрутра папок в приложении (App)



3 папки: config, controllers, views
**config** - содержит на данный момент всего один файл с настройками приложения application.php
[cc lang="php"]
array(
        'applicationName' => 'first application'
    ),
	// самое интересное, наверное, в этом фреймворке
    'routes' => array(
		// все рауты имеют имя, шаблон или регулярное выражение
		// контроллер, экшн и параметры
        'static' => array(
            'template' => '^(\w+)\.html$',
            'controller' => 'static',
            'action' => 'show',
            'params' => array(
                'name' => 1
            )
        ),
        'main' => array(
            'template' => null,
            'controller' => 'index',
            'action' => 'index'
        ),
        'dynamic' => array(
            'template' => '^(\w+)\/(\w+)$',
            'controller' => 'dynamic',
            'action' => 'show',
            'params' => array(
                'category' => 1,
                'article' => 2
            )
        )
    ),
    'view' => array(
		// где будут лежать наши шаблоны (views)
        'viewPath' => APPLICATION_PATH . DIRECTORY_SEPARATOR . 'views',
		// имя нашего шаблона (layout)
        'template' => 'template.tpl'
    )
);
[/cc]

Вскользь хочу упоминуть лишь некоторые классы
**MVC_Registry** - стандартный паттерн Registry без излишеств, нужен нам для того, чтобы хранить данные и предоставлять доступ к ним из любого места в приложении.

**MVC_Request** - класс запроса, который мы строим в нашем "Фронт контроллере" и передаем в контроллеры приложения. Содержит параметры в запросе, активный контроллер и экшн.

**MVC_View** - класс, который просто инициализует Smarty и устанавливает все нужные нам значения путей для шаблонов. 




### "Front Controller"


Этот класс - точка входа для нашего приложение, хоть и должен был он реализован быть как Одиночка (Singleton)

**MVC_FrontController**
[cc lang="php"]
_initConfigs();
        $this->_initResources();
        $this->_initRoutes();
    }

    protected function _parseRequest()
    {
        return $_SERVER['REQUEST_URI'];
    }

    public function run()
    {
        $uri = $this->_parseRequest();

        $activeRoute = $this->_checkActiveRoute($uri);
        if (null === $activeRoute)
            throw new MVC_Exception('Cannot find active route');
        $this->_dispatch($activeRoute);

        $controllerName = sprintf('App_Controller_%sController', ucfirst($this->_controller));
        $controllerObj = new $controllerName(
                        new MVC_Request(
                                array('params' => $this->_params,
                                    'controller' => $this->_controller,
                                    'action' => $this->_action)
                        )
        );
        $methodName = $this->_action . 'Action';
        if (method_exists($controllerObj, $methodName)) {
            $controllerObj->$methodName();
            $controllerObj->render();
        } else {
            throw new MVC_Exception('action not found');
        }
    }

    protected function _initConfigs()
    {

        require_once APPLICATION_PATH . DIRECTORY_SEPARATOR . 'config' . DIRECTORY_SEPARATOR . 'application.php';

        MVC_Registry::getInstance()->set('settings', $settings);
        $this->_settings = $settings;
    }

    /**
     * init resource autoloading
     */
    protected function _initResources()
    {
        spl_autoload_register(array('MVC_Autoload_Resource', 'autoload'));
    }

    /**
     * init routes
     */
    protected function _initRoutes()
    {
        if (isset($this->_settings['routes'])) {
            $this->_routes = $this->_settings['routes'];
        } else {
            throw new MVC_Exception('Routes must be specified');
        }
    }

    /**
     * get active route name
     * 
     * @param string $uri
     * @return string active route name
     */
    protected function _checkActiveRoute($uri)
    {
        $uri = substr($uri, 1);
        if (trim($uri)) {
            $activeRoute = null;

            foreach($this->_routes as $name => $routeSettings) {
                if (!$routeSettings['template'])
                    continue;
                if (preg_match('@' . $routeSettings['template'] . '@', $uri, $matches)) {
                    if (isset($routeSettings['params'])) {
                        foreach($routeSettings['params'] as $paramName => $param) {
                            $this->_params[$paramName] = $matches[$param];
                        }
                    }
                    $activeRoute = $name;
                }
            }
        } else {
            $activeRoute = 'main';
        }

        return $activeRoute;
    }

    /**
     * dispatch
     *
     * @param string $activeRoute active route name
     */
    protected function _dispatch($activeRoute)
    {
        if (isset($this->_routes[$activeRoute])) {
            $this->_controller = $this->_routes[$activeRoute]['controller'];
            $this->_action = $this->_routes[$activeRoute]['action'];
        }
    }

}
[/cc]

Я старался комментировать не особо очевидные места, но если возникнут вопросы, обязательно задавайте их в комментериях внизу заметки. Метод _checkActiveRoute() определяет в зависимости от REQUEST_URI активный раут. И заполняет параметры для контроллера нашего приложения. 

Метод _dispatch() по активному рауту определяет нужный в данный момент контроллер и экшн.

Метод run() запускает наш контроллер, передавая ему объект запроса с нужными нам параметрами. После запуска нужного экшна мы запускаем метод контроллера render() который и выводит контент.




### MVC_Controller_Abstract


Этот класс раширяют все контроллеры нашего приложения, единственный метод, на котором стоит заостроить внимание это render()

[cc lang="php"]
public function render()
    {
        $templatePath = $this->_view->template_dir . DIRECTORY_SEPARATOR;
        $templatePath .= strtolower(
                $this->getRequest()->getController()
                . DIRECTORY_SEPARATOR
                . $this->getRequest()->getAction() . '.tpl'
        );
        if (file_exists($templatePath) && is_readable($templatePath)) {
            $this->getView()->assign('tplName', $templatePath);

            if (null === $this->_view->template)
                $this->getView()->display('template.tpl');
            else {
                $this->getView()->display($this->getView()->template);
            }
        } else {
            throw new MVC_Exception("Template '{$templatePath}' not found");
        }
    }
[/cc]

Допустим мы прошли по адресу example.com/ нашего приложения. Наш "Фронт Контроллер" запустит app/controllers/IndexController.php и метод indexAction(). В нем, к примеру, мы присвоим переменной name значение vredniy
[cc lang="php"]
public function indexAction()
    {
        $this->getView()->assign('name', 'Vredniy');
    }
[/cc]

Дальше метод **MVC_Controller_Abstract::render()** отыщет наш шаблон (layout). В данном случает это, находящийся в папке app/views, файл template.tpl. Приведу его содержимое
[cc lang="html"]

    
    
        

# template


        {include file="$tplName"}
        

## footer


    

[/cc]

Это и есть основной шаблон для нашего приложения. Вы спросите, а как же тогда выводится переменная name, значение которой мы присваивали в IndexController::indexAction(). Все просто: встроенная конструкция Smarty {include file="$tplName"} заменяется содержимым соответствующего экшна видом. 
Метода render() в MVC_Controller_Abstract
[cc lang="php"]
$this->getView()->assign('tplName', $templatePath);
[/cc]

Сам же вид экшна имеет вид
[cc lang="html"]
**{$name|upper}**


## MAin page


[/cc]

[![php-mvc-project](http://vredniy.ru/wp-content/uploads/2011/06/php-mvc-project1-300x168.png)](http://vredniy.ru/wp-content/uploads/2011/06/php-mvc-project1.png)Вот вроде и все. Данный фреймворк не предназначен для построения сложных сайтов, да и вообще для сайтов. Да и вообще ни для чего он не предназначен, кроме понимания некоторых важных для любого фреймворка вещей. В данном ознакомительном фреймворке куча ошибок и недоработок, реализация или исправления которых могли существенно захломить тестовый фреймворк и намного усложнить его понимание. 

Спасибо всем за внимание, надеюсь, хоть часть мною написанного вам пригодится. Ах да, чуть не забыл прилагаю к заметке ссылку на [репозиторий](https://github.com/vredniy/simple-mvc), чтобы вы смогли скачать данный фреймворк и запустить у себя.


UPDATE: Сегодня решил, что в связке MVC не может не быть модели, поэтому решил я ее сегодня дописать.



### Модель в MVC


Добавил я два класса: первый это MVC_Db_Exception, чтобы выбрасывать эксепшны не просто MVC_Exception. Второй - освновной MVC_Db_Abstract, который мы будем расширять в модели. Приведу его код:

[cc lang="php"]
"SET NAMES 'UTF8'");

    /**
     * PDO attributes
     *
     * @param array|string $attribs 
     */
    public function __construct()
    {
        $this->_initConnection();
    }
	
	// инициализуем соединение, если его еще нет
    private function _initConnection()
    {
        if (!$this->_connection instanceof PDO) {
            $settings = MVC_Registry::get('settings');
            $dbSettings = $settings['database'];
            $dsn = sprintf('%s:host=%s;dbname=%s', $dbSettings['adapter'], $dbSettings['params']['host'], $dbSettings['params']['dbname']);
            try {
                $this->_connection = new PDO($dsn, $dbSettings['params']['username'], $dbSettings['params']['password'],
                                $this->_options);
            } catch (PDOException $e) {
                echo 'Connection failed: ' . $e->getMessage();
            }
        }
    }

	// возвращаем соединение, которое и будем использовать в моделях
    public function getConnection()
    {
        return $this->_connection;
    }

}
[/cc]

И внимание, кто пробует это проделать по заметке, будьте внимательны, потому что я чуть дописал MVC_Autoload_Resource, чтобы он помимо контроллеров мог подгружать еще и наши модельки. Самая актуальная версия в [репозитории](https://github.com/vredniy/simple-mvc).



### Использование модели


Создадим папку app/models, где и будут лежать наши модели. В ней создадим файл PageModel.php с таким содержимым
[cc lang="php"]
getConnection()->prepare($statementString);
        $statement->execute(array($id));
        return $statement->fetch();
    }

}
[/cc]

Данная модель умеет извлекать из таблицы pages нужные нам данные с помощью PHP Data Object (PHP) по id. Дамп базы данных я выложил в репозитории в папке data/dumps

Теперь в контроллере попробуем использовать нашу модель. Я взял для примера IndexController, в index экшн которого я дописал следующее.
[cc lang="php"]
    public function indexAction()
    {
		// создаем нашу модель
        $model = new App_Model_PageModel();
		// извлекаем из нее нужные нам данные и
		// отдаем их Smarty, т.е. во View
        $this->getView()->assign('data', $model->getPage(1));
        $this->getView()->assign('name', 'Vredniy');
    }
[/cc]

В шаблоне views/index/index.tpl я добавил три строчки, которые и будут выводить содержимое:
[cc lang="html"]
**{$name|upper}**


## {$data.alias}


{$data.content}
[/cc]

Вот вроде и все, еще раз удачи вам :)
