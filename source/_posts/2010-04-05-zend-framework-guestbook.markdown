---
author: admin
comments: false
date: 2010-04-05 18:22:19+00:00
layout: post
slug: zend-framework-guestbook
title: Гостевая книга на Zend Framework
wordpress_id: 442
categories:
- php
- Zend Framework
- программирование
tags:
- guestbook
- mvc
- php
- Zend Framework
---

	

Zend Framework - это объектно-ориентированный веб-фреймворк с открытым исходным кодом для PHP 5. Zend Framework также часто называют 'component library', потому что он имеет множество слабо связанных между собой компонентов, которые вы можете использовать независимо друг от друга. Но Zend Framework также предоставляет расширенную MVC реализацию, с помощью которой вы можете построить структурированное приложение. 


	Этот быстрый старт познакомит вас с несколькими часто используемыми компонентами, включая Zend_Controller, Zend_Layout, Zend_Config, Zend_Db, Zend_Db_Table, Zend_Registry и несколькими хелперами (helpers).<!-- more -->

	

Использую эти компоненты мы сможем создать простое приложение Гостевая книга, использующее базу данных, за несколько минут.
	


### 	Модель - Представление - Контроллер




	Так о чем же говорят люди, упоминая MVC и почему мы должны придерживаться этого паттерна? MCV - это больше чем прорсто трехбуквенный акроним (three-letter acronym - TLA), он стал стандартом в проектировании современных веб-приложений. Код большинства веб-приложений делится на 3 составляющих: представление, бизнес логика и доступ к данным. Паттерн MVC прекрасно справляется с этим разделением. Конечный результат в том, что код представления будет в одной части вашего веб-приложения, также как и бизнес логика, и код доступа к данным. Многие разработчики находят это четкое разделение необходимым для сохранения кода исходного приложения организованным, особенно когда более, чем один человек работает над этим проектом. 
	
	

Давайте оторвавшись от паттерна, познакомимся с его отдельными частями.
	[![model-view-controller](http://vredniy.ru/wp-content/uploads/2010/04/mvc-150x150.png)](http://vredniy.ru/wp-content/uploads/2010/04/mvc.png)
	

Модель - это та часть приложения, которая определяет базовую функциональность, используя множество абстракций. Здесь может быть определена некоторая бизнес логика и шаблоны доступа к данным. 


	Представление четко определяет то, что увидит пользователь. Обычно контроллеры передают данные презентации для отображения их в определенном формате. Это подходящее место для вставки HTML разметки в ваше приложение. 
	

Контроллер - связывает всю структуру воедино. Также управляет моделями, решает какую презентацию использовать для отображения данных, исходя из запроса пользователя или других факторов, передает данные презентации те, которые ей нужны. Эксперты в области MVC рекомендуют использвать в контроллерах минимум кода по мере возможностей. 
	

Конечно здесь речь идет о строгом разделении паттерна, но это поможет нам понять всю поднаготную нашей гостевой книги, которую мы и собрались писать.
	


### 	Создание проекта.




	Для начала вам нужно скачать последнюю версию Zend Framework и разархивировать его.
	
	

### Установка Zend Framework




	Если хотите, то можете добавить путь к папке library/ только что разархивированного фреймворка в include_path вашего php.ini. Чтобы не копировать в каждый проект эту библиотеку.
	В папке bin/ Zend Framework находятся 2 скрипта: zf.sh и zf.bat для Unix-систем и Windows соответственно, при обращении к которым используйте абсолютный путь. 
	Откройте терминал (в Windows, Пуск -> Выполнить, cmd). Войдите в папку, в которой хотите расположить свой проект и запустите соответсвующий скрипт:
	**zf.sh  create project quickstart #Unix**  

	**zf.bat create project quickstart #Windows**
	

Выполнение этих команд создаст вам базовую структуру приложения, включая инициализацию контроллеров и презентаций. Дерево каталогов после выполнения выглядит так: [![mvc](http://vredniy.ru/wp-content/uploads/2010/04/mvc-150x150.gif)](http://vredniy.ru/wp-content/uploads/2010/04/mvc.gif) 
	

Теперь самое время скопировать папку с библиотеками Zend Framework library/Zend в папку нашего проекта в /library/
	

Теперь наш проект создан, и мы готовы перейти к начальной загрузке (Bootstrap), конфигурации и контроллерам действий (Action Controllers).
	
	

### Начальная загрузка




	Класс Bootstrap определяет инициализацию ресурсов и компонентов. По умолчанию инициализируется Front Controller, используя application/controllers/ как основную папку для контроллеров (об этом чуть позже). 
	[cc lang="php"]
//application/Bootstrap.php
	class Bootstrap extends Zend_Application_Bootstrap_Bootstrap
	{
	}
[/cc]
	

### Конфигурация


	Обычно конфигурация всего проекта хранится в application/configs/application.ini и хранит несолько основных директив для установки параметров среды для PHP (включение и отключение вывода ошибок и др.), указывает путь к классу первоначальной загрузки (и имя класса) и путь к контроллерам.
	; application/configs/application.ini
	[cc lang="php"][production]
	phpSettings.display_startup_errors = 0
	phpSettings.display_errors = 0
	includePaths.library = APPLICATION_PATH "/../library"
	bootstrap.path = APPLICATION_PATH "/Bootstrap.php"
	bootstrap.class = "Bootstrap"
	resources.frontController.controllerDirectory = APPLICATION_PATH "/controllers"
	
	[staging : production]
	
	[testing : production]
	phpSettings.display_startup_errors = 1
	phpSettinfs.display_errors = 1
	
	[development : production]
	phpSettings.display_startup_errors = 1
	phpSettinfs.display_errors = 1
	[/cc]
	

На некоторых вещах все же стоит остановиться поподробнее. Первое, когда используется конфигурация в INI-файле, вы можете пользоваться константами; APPLICATION_PATH - константа. И еще о дополнительно объявленных секциях: [production], [staging], [testing] и [development]. Последние наследуют параметры из секции [production]. Это очень удобный способ организовать отдельную конфигурацию для отдельной стадии проектирования и создания приложения.
	
	

### Action Controllers


	

Контроллеры распределяют запросы между соответствующих моделей и презентаций. 


	Контроллер должен содержать один или более методов, заканчивающихся на "Action"; эти методы впоследствии будут вызваны через веб. По умолчанию, URLы Zend Framework придерживаются схеме /controller/action, где "controller" указывает на имя контроллера (минус суффикс "Controller") и "action" указывает на действие (минус суффикс "Action").
	

Обычно, у вас имеется IndexController, который отображает домашнюю страницу сайта и ErrorController, который используется для перехвата таких вещей как ошибка HTTP 404 (контроллер или действие не найдено) и ошибка HTTP 500 (ошибка в приложении).
	[cc lang="php"]//application/controllers/IndexController.php
	class IndexController extends Zend_Controller_Action
	{
		public function init()
		{
		}
		
		public function indexAction() 
		{
		}
	}
	[/cc]
	и 
	[cc lang="php"]//application/controllers/ErrorController.php
	class ErrorController extends Zend_Controller_Action
	{
		public function errorAction()
		{
			$errors = $this->_getParam('error_handler');
			
			switch($errors->type) {
				case Zend_Controller_Plugin_ErrorHandler::EXCEPTION_NO_ROUTE:
				case Zend_Controller_Plugin_ErrorHandler::EXCEPTION_NO_CONTROLLER:
				case Zend_Controller_Plugin_ErrorHandler::EXCEPTION_NO_ACTION::
					
					this->getResponse()->setHttpResponseCode(404);
					$this->view->message = 'Страница не найдена';
					break;
				default:
					$this->getResponse()->setHttpResponseCode(500);
					$this->view->message = 'Ошибка приложения';
					break;
			}
			
			$this->view->exception = $errors->exception;
			$this->view->request   = $errors->request;
		}
	}
	[/cc]
	

### Презентация


	Презентационные скрипты находятся по адресу application/views/scripts/, где они дополнительно подразделяются, используя имена контроллеров. В нашем примере, у нас есть IndexController и ErrorController, которые соответственно ссылаются на index/ и error/ в поддериктории наших скриптов.
В этих поддерикториях также находятся папки с именами действий, по умолчанию, index/index.phtml и error/error.phtml.
Скрипты презентации состоят из HTML разметки и тегов  для вставки PHP директив.
	

### Создание слоя




	Как вы успели заметить скрипты вида в предыдущем разделе содержат только фрагменты HTML - не полные страницы.
	

Теперь же мы превратим генерируемый контент в полную HTML страницу, воспользовавшись глобальным слоем (layout).
	

Для того, чтобы воспользоваться Zend_Layout нам нужно сообщить первоначальному загрузчику применять Layout ресурсы, добавив следующие строки в application/configs/application.ini в раздел [production]:  

	**resources.layout.layoutPath = APPLICATION_PATH "/layouts/scripts"**
  



Создайте папку application/layouts/scripts.
	

Также вы можете вставлять заголовки DocType в свои приложения, для этого нужно добавить соотвутсвующий ресурс в начальную загрузку.
	

Простой способ добавить ресурс в начальную загрузку - это создать protected метод, начинающийся с _init. Т.к. мы хотим инициализировать doctype, значит создаем _initDoctype() метод.
	[cc lang="php"]//application/Bootstrap.php
	class Bootstrap extends Zen_Application_Bootstrap_Bootstrap
	{
		protected function _initDoctype()
		{
		}
	} [/cc]
	

Этим методом мы сообщаем виду использовать заданный doctype. Чтобы инициализировать вид ресурса добавим следующие строки в application/cofigs/application.ini в раздел **[producation]**
	**resources.view[] =**
	

Тем самым мы инициализируем вид без опций. 
	[cc lang="php"]class Bootstrap extends Zend_Application_Bootstrap_Bootstrap
	{
		protected function _initDoctype()
		{
			$this->bootstrap('view');
			$view = $this->getResource('view');
			$view->doctype('XHTML1_STRICT');
		}
	}
	[/cc]
	

Теперь мы инициализируем Zend_Layout и устанавливаем Doctype, осталось только создать слой
	[cc lang="php"]//application/layouts/scripts/layout.phtml
	echo $this->doctype() ?>






    


        **ZF Quickstart Application**
    


    


        [Guestbook](<?php echo $this->url(
            array('controller'=>'guestbook'),
            'default',
            true) ?>)
    





layout()->content ?>

[/cc]
	

Мы пропускаем весь контент через функцию layout помощника вида и получаем доступ к ключу "content". Мы также используем headLink() для удобной вставки  тегов в шаблон. 
	

Теперь вы можете пройти по адресу http://localhost и посмотреть на исходный текст страницы, в нем вы должны увидеть XHTML заголовок, раздел body, head и title.
	


### 	Создание таблицы в базе данных и создание модели


	

Zend_Controller_Front имеет такое понятие как модули, которая являются мини приложениями. Они располагаются в папке application и имеют префикс, имя модуля.
	

Zend_Application_Module_Autoloader предоставляет функционал необходимый для доступа к разнообразным ресурсам в соответсвующих директориях и предоставляет стандартный механизм именования. Экземпляр класса создается по умолчанию во время инициализации начальной загрузки; начальная загрузка по умолчанию использует префикс "Application". В нашем примере вем модели и формы будут также начинаться с этого префикса.
	

Давайте рассмотрим из чего состит гостевая книга. Обычно гостевая книга состит из записей с комментарием, временем его создания и часто с адресом электронной почты. Т.к. мы будем сохранять все данные в базе данных, мы также добавим уникальный идентификатор. Наше приложение будет уметь выводить все записи, сохранять новые и  выводить нужную. Чуть ниже приводится простая модель базы данных.
	[cc lang="php"]//application/models/Guestbook.php
	class Application_Model_Guestbook
{
    protected $_comment;
    protected $_created;
    protected $_email;
    protected $_id;
    public function __set($name, $value);
    public function __get($name);
    public function setComment($text);
    public function getComment();
    public function setEmail($email);
    public function getEmail();
    public function setCreated($ts);
    public function getCreated();
    public function setId($id);
    public function getId();
    public function save();
    public function find($id);
	public function fetchAll();
}
	[/cc]
	

**__get()** и **__set()** - обеспечивают удобный механизм для доступа к отдельным свойствам записей.
	

**find()** и **fetchAll()** - позволяют получить доступ к отдельной записе или ко всем вместе.


	С этого момента мы можем начать думать о конфигурации нашей базы данных.
	Для начала нужно инициализировать Db ресурс. Мы задавали параметры в конфигурационном файле для Layout и View ресурсов, сделаем тоже самое и для Db.
	[cc lang="php"];application/configs/application.ini
	[production]
	resources.db.adapter = "PDO_MYSQL"
	resources.db.params.dbname = "guestbook-db"
	resources.db.params.host = "localhost"
	resources.db.params.username = "root"
	resources.db.params.password = ""
[/cc]
	*** укажите здесь параметры для подключения к своей базе данных**
	На этом этапе нам понадобится создать в этой базе данных таблицу, в которой и будут храниться все записи. Для этого выполним SQL запрос.
	[cc lang="sql"]CREATE TABLE guestbook (
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(32) NOT NULL DEFAULT 'noemail@example.com',
    comment TEXT NULL,
    created DATETIME NOT NULL
);[/cc]
	
	
	

Теперь у нас есть полностью рабочая база данных и таблица в ней для нашего приложения. Дальнейшие наши шаги будут заключаться в создании кода приложения, включая кастомизацию источника данных (в нашем примере это Zend_Db_Table) и преобразователь данных для подключения к источнику. В конце мы создадим контроллер, который будет взаимодействовать с этой моделью и для вывода существующих в базе данных записи, и для созданя новых. 
	

Мы используем Table Data Gateway для соединения нашим источником данных; Zend_Db_Table обеспечивает этот функционал. Начнем с создания класса, базирующегося на Zend_Db_Table в папке application/models/DbTable/, файл будет называться Guestbook.php	
	[cc lang="php"]//application/models/DbTable/Guestbook.php
	class Application_Model_DbTable_Guestbook extends Zend_Db_Table_Abstract
{
    
    protected $_name    = 'guestbook';
}
[/cc]
	

Важно отметить то, что префикс нашего класса Application_Model_DbTable соответствует пути его месторасположения. 
	

Теперь давайте создадим **Data Mapper**. 
	[cc lang="php"]//application/models/GuestbookMapper.php
class Application_Model_GuestbookMapper
{
    protected $_dbTable;
    public function setDbTable($dbTable)
    {
        if (is_string($dbTable)) {
            $dbTable = new $dbTable();
        }
        if (!$dbTable instanceof Zend_Db_Table_Abstract) {
            throw new Exception('Неприавльные данные');
        }
        $this->_dbTable = $dbTable;
        return $this;
    }
    public function getDbTable()
    {
        if (null === $this->_dbTable) {
            $this->setDbTable('Application_Model_DbTable_Guestbook');
        }
        return $this->_dbTable;
    }
    public function save(Application_Model_Guestbook $guestbook)
    {
        $data = array(
            'email'   => $guestbook->getEmail(),
            'comment' => $guestbook->getComment(),
            'created' => date('Y-m-d H:i:s'),
        );
		if (null === ($id = $guestbook->getId())) {
            unset($data['id']);
            $this->getDbTable()->insert($data);
        } else {
            $this->getDbTable()->update($data, array('id = ?' => $id));
        }
    }
    public function find($id, Application_Model_Guestbook $guestbook)
    {
        $result = $this->getDbTable()->find($id);
        if (0 == count($result)) {
            return;
        }
        $row = $result->current();
        $guestbook->setId($row->id)
                  ->setEmail($row->email)
                  ->setComment($row->comment)
                  ->setCreated($row->created);
    }
    public function fetchAll()
    {
        $resultSet = $this->getDbTable()->fetchAll();
        $entries   = array();
        foreach ($resultSet as $row) {
            $entry = new Application_Model_Guestbook();
            $entry->setId($row->id)
                  ->setEmail($row->email)
                  ->setComment($row->comment)
                  ->setCreated($row->created)
                  ->setMapper($this);
            $entries[] = $entry;
        }
        return $entries;
    }
}[/cc]
	

Сейчас самое время для того, чтобы написать класс для взаимодействия с GuestbookMapper. 
	[cc lang="php"]//application/models/Guestbook.php
	class Application_Model_Guestbook
{
    protected $_comment;
    protected $_created;
    protected $_email;
    protected $_id;
    protected $_mapper;
    public function __construct(array $options = null)
    {
        if (is_array($options)) {
            $this->setOptions($options);
        }
		}
    public function __set($name, $value)
    {
        $method = 'set' . $name;
        if (('mapper' == $name) || !method_exists($this, $method)) {
            throw new Exception('Неправильное свойство');
        }
        $this->$method($value);
    }
    public function __get($name)
    {
        $method = 'get' . $name;
        if (('mapper' == $name) || !method_exists($this, $method)) {
            throw new Exception('Неправильное свойство');
        }
        return $this->$method();
    }
    public function setOptions(array $options)
    {
        $methods = get_class_methods($this);
        foreach ($options as $key => $value) {
            $method = 'set' . ucfirst($key);
            if (in_array($method, $methods)) {
                $this->$method($value);
            }
        }
        return $this;
    }
    public function setComment($text)
    {
        $this->_comment = (string) $text;
        return $this;
    }
    public function getComment()
    {
        return $this->_comment;
    }
    public function setEmail($email)
    {
        $this->_email = (string) $email;
        return $this;
    }
    public function getEmail()
    {
        return $this->_email;
    }
    public function setCreated($ts)
    {
        $this->_created = $ts;
        return $this;
    }
	public function getCreated()
    {
        return $this->_created;
    }
    public function setId($id)
    {
        $this->_id = (int) $id;
        return $this;
    }
    public function getId()
    {
        return $this->_id;
    }
    public function setMapper($mapper)
    {
        $this->_mapper = $mapper;
        return $this;
    }
    public function getMapper()
    {
        if (null === $this->_mapper) {
            $this->setMapper(new Application_Model_GuestbookMapper());
        }
        return $this->_mapper;
    }
    public function save()
    {
        $this->getMapper()->save($this);
    }
    public function find($id)
    {
        $this->getMapper()->find($id, $this);
        return $this;
    }
    public function fetchAll()
    {
        return $this->getMapper()->fetchAll();
    }
}[/cc]
	

Теперь, для того, чтобы связать все эти элементы воедино, давайте напишем контроллер.
	[cc lang="php"]//application/controllers/GuestbookController.php
	class GuestbookController extends Zend_Controller_Action
{
    public function indexAction()
    {
        $guestbook = new Application_Model_Guestbook();
        $this->view->entries = $guestbook->fetchAll();
    }
}[/cc]
	

И конечно скрипт для отображения вида
	[cc lang="php"]//application/views/scripts/guestbook/index.phtml
	

[Sign Our Guestbook](<?php echo $this->url(
    array(
        'controller' => 'guestbook',
        'action'     => 'sign'
    ),
    'default',
    true) ?>)


Guestbook Entries:   




    entries as $entry): ?>
    escape($entry->email) ?>

        escape($entry->comment) ?>

    
[/cc]
	

Перейдя теперь по ссылке** http://localhost/guestbook/**  вы увидите записи из базы данных.
	
	

### Создание формы




	Для нашей гостевой книги просто необходима форма для добавления новых записией.
	

Для начала создадим класс формы
	[cc lang="php"]//application/forms/Guestbook.php
	class Application_Form_Guestbook extends Zend_Form
{
    public function init()
    {
	 $this->setMethod('post');
        // Add an email element
        $this->addElement('text', 'email', array(
            'label'      => 'Почтовый ящик:',
            'required'   => true,
            'filters'    => array('StringTrim'),
            'validators' => array(
                'EmailAddress',
            )
        ));
        // Add the comment element
        $this->addElement('textarea', 'comment', array(
            'label'      => 'Комментарий:',
            'required'   => true,
            'validators' => array(
                array('validator' => 'StringLength', 'options' => array(0, 20))
                )
        ));
        // Add a captcha
        $this->addElement('captcha', 'captcha', array(
            'label'      => 'Введите 5 цифр, которвые вы видите снизу:',
            'required'   => true,
            'captcha'    => array(
                'captcha' => 'Figlet',
                'wordLen' => 5,
                'timeout' => 300
            )
        ));
        // добавить
        $this->addElement('submit', 'Добавить', array(
            'ignore'   => true,
            'label'    => 'Sign Guestbook',
        ));
        // And finally add some CSRF protection
        $this->addElement('hash', 'csrf', array(
            'ignore' => true,
        ));
    }
}[/cc]
	

Форма состоит из 5 элементов: поле для ввода электронной почты, поле для текста комментария, CAPTCHA для защиты от спама, кнопка отправить и зашита от подделки межсайтовых запросов.


	Дальше мы добавим к нашему контроллеру GuestbookController метод signAction() для отображения формы.
	[cc lang="php"]//application/controllers/GuestbookController.php
	class GuestbookController extends Zend_Controller_Action
{
    // snipping indexAction()...
    public function signAction()
    {
        $request = $this->getRequest();
        $form    = new Application_Form_Guestbook();
        if ($this->getRequest()->isPost()) {
            if ($form->isValid($request->getPost())) {
                $model = new Application_Model_Guestbook($form->getValues());
                $model->save();
                return $this->_helper->redirector('index');
            }
        }
        $this->view->form = $form;
    }
}[/cc]
	Также нам обязательно нужно создать скрипт для отображения формы **application/views/scripts/guestbook/sign.phtml**
	[cc lang="html"]
Please use the form below to sign our guestbook!
form->setAction($this->url());
echo $this->form;[/cc]
	

Теперь пройдя по ссылке **http://localhost/guestbook/sign** вы должны увидеть нашу форму добавления.
	
	

Поздравляю мы только что создали простейшией приложения использяю некоторые из часто применяемых компонентов Zend Framework.
