---
author: admin
comments: true
date: 2010-11-27 18:02:20+00:00
layout: post
slug: mvc-models-doctrine-zend-framework

permalink: /2010/11/mvc-models-doctrine-zend-framework

description: "Давайте напишем пару слов о популярном сейчас ORM Doctrine, научимся устанавливать его, использовать в своих проектах наряду с Zend Framework'ом."
keywords: "doctrine, zend framework, doctrine orm, zf doctine, zf orm,models,mvc,orm,zend framework,db,oop"

title: Модели, Doctrine и Zend Framework
wordpress_id: 629
categories:
- db
- Doctrine
- OOP
- Zend Framework
tags:
- Doctrine
- models
- mvc
- orm
- Zend Framework
---


Давайте напишем пару слов о популярном сейчас ORM Doctrine, научимся устанавливать его, использовать в своих проектах наряду с Zend Framework'ом.
<!-- more -->

[{% img image /images/posts/2010-11-mvc-models-doctrine-zend-framework/zend-framework-doctrine1-150x150.jpg %}](/images/posts/2010-11-mvc-models-doctrine-zend-framework/zend-framework-doctrine1.jpg)



### Установка Doctrine


  В первую очередь скачиваем с официального сайта [Doctrine](http://www.doctrine-project.org/projects/orm/download), я свое ознакомление начал со стабильной версии 1.2, хотя для скачивания доступна вторая версия (бета), думаю ее "пощупаю" немного попозже. Распаковываем ее и копируем папку lib из Doctrine-* в include_path, я поместил ее в /usr/share/php/. Теперь напишем первый скрипт, который проверит работу Doctrine. Хотя подождите, подготовим сначала базу данных и наполним ее данными для тестов. За неимением лучшего я всего скачиваю mysql wordl database (на этот раз это будет innoDB версия) по [ссылке ](http://dev.mysql.com/doc/index-other.html) и создаю пользователя для тестов с паролем и логином таким же, как и название базы данных. У меня получился пользователь world с паролем world и база данных тоже называется world, теперь приступим.


``` php
<?php
  include_once 'Doctrine/Doctrine.php';

  // подключаем автозагрузку
  spl_autoload_register(array('Doctrine', 'autoload'));

  $manager = Doctrine_Manager::getInstance();

  // подключаемся к базе данных
  $conn = $magager->connection('mysql://world:world@localhost/world', 'doctrine');

  // получаем список всех баз данных
  $databases = $conn->import->listDatabases();
  var_dump($databases);
?>
```


Дальше запускаем наш скрипт и видим, если все сделали правильно, список всех баз данных.





### Генерация моделей Doctrine


Так как Doctrine идет в комплекте с мощным генератором моделей, который может "читать" существующую базу данных и создавать классы моделей. Чтобы увидеть это в действии, запустим  следующее.


``` php
<?php
  include_once 'Doctrine/Doctrine.php';
  spl_autoload_register(array('Doctrine', 'autoload'));

  $manager = Doctrine_Manager::getInstance();

  $conn = $manager->connection('mysql://world:world@localhost/world', 'doctrine');

  // генерируем модели в папке со скриптом
  // 2 параметр соединение, которое мы указывали на строчку выше
  // 3 префикс для названия файлов и классов
  Doctrine::generateModelsFromDb('./models',
    array('doctrine'),
    array('classPrefix' => 'Model_', 'classPrefixFiles' => false)
  );
?>

```

К сведению, Doctrine для каждой таблицы в базе данных создает 2 класса модели, базовый класс и класс-наследник. Базовый класс расширяет Doctrine_Record и включает определения таблицы и базовые операции по работе с таблицей. Класс-наследник - это место для написания дополнительного функционала, который мы собираемся добавить в модель.

Теперь скопируем все получившиеся модели (их должно получиться 6 штук) в наш Zend Framework проект в папку application/models.




### Настраиваем модельные связи


[{% img image /images/posts/2010-11-mvc-models-doctrine-zend-framework/doctrine-300x194.png %}](http://vredniy.ru/wp-content/uploads/2010/11/doctrine.png)
Doctrine - очень мощный ORM, но она не может справиться с установлением связей между таблицами, чем мы займемся сами. Doctrine поддерживает все виды связи один-к-одной, один-ко-многим, многие-ко-многим и рекурсивную используя всего 2 метода hasOne() и hasMany(). У каждого метода два аргумента: имя другой модели для объявления связи и массив опций, характеризующих связь. Для того, чтобы связи автоматически загружались поместим их в setUp() метод.

**models/City.php**

``` php
<?php
  class Model_City extends Model_BaseCity
  {

      public function setUp()
      {
     $this->hasOne('Model_Country', array(
         'local' => 'CountryCode',
         'foreign' => 'Code'
     ));
      }

  }
?>
```


**models/Country.php**

``` php
<?php
  class Model_Country extends Model_BaseCountry
  {

      public function setUp()
      {
     $this->hasOne('Model_City', array(
         'local' => 'Code',
         'foreign' => 'CountryCode'
     ));
     $this->hasOne('Model_CountryLanguage', array(
         'local' => 'Code',
         'foreign' => 'CountryCode'
     ));
      }

  }
?>
```


**models/CountryLanguage.php**

``` php
<?php
  class Model_CountryLanguage extends Model_BaseCountryLanguage
  {

      public function setUp()
      {
     $this->hasOne('Model_Country', array(
         'local' => 'CountryCode',
         'foreign' => 'Code'
     ));
      }

  }
?>
```





### Автозагрузка классов Doctrine в Zend Framework


Все что осталось сделать, так это то, чтобы приложение Zend Framework и Doctrine хорошо жили совместно. В первую очередь поместим Doctrine в папку library Zend Framework'а. Следующая стадия - обновить конфигурационный файл application.ini, поместив туда строку DSN, чтобы Doctrine могла без проблем соединяться с базой данных. Допишем в [production] секцию


``` php
doctrine.dsn = "mysql://world:world@localhost/world"
```


Финальным шагом для инициализации Doctrine напишем защищенный метод в нашем бутстрапе.
**application/Bootstrap.php**

``` php
<?php
class Bootstrap extends Zend_Application_Bootstrap_Bootstrap
{
  protected function _initDoctrine()
  {
    require_once 'Doctrine/Doctrine.php';
    $this->getApplication()
    ->getAutoloader()
    ->pushAutoloader(array('Doctrine', 'autoload'), 'Doctrine');

    $manager = Doctrine_Manager::getInstance();
    $magager->setAttribute(
      Doctrine::ATTR_MODEL_LOADING, Doctrine::MODEL_LOADING_CONSERVATIVE
    );

    $config = $this->getOption('doctrine');
    $conn = Doctrine_Manager::connection($config['dsn'], 'doctrine');
  }
}
?>
```





### Операции с моделью Doctrine


До того, как погрузиться в кроличью, освоим элементарные операции с моделями.
Извлечение данных
... по ID:

``` php
<?php
  $city = Doctrine::getTable('Model_City')->find(10);
  var_dump($city);
?>
```

... всех записей:

``` php
<?php
  $cities = Doctrine::getTable('Model_City')->findAll();
  var_dump($cities);
?>
```


Также в Doctrine есть свой язык запросов DQL (Doctrine Query Language), который предоставляет гибкий интерфейс для создания и выполнения запросов. Посмотрим на примеры.

``` php
<?php
  $query = Doctrine_Query::create()
       ->from('Model_City c');
  $result = $query->fetchArray();
?>
```

Использование условий


``` php
<?php
  $query = Doctrine_Query::create()
       ->from('Model_City c');
       ->where('c.id = ?', 10);
  $result = $query->fetchArray();
?>
```


В DQL много приятных плюшек, среди которых группировка и сортировка, но мы остановимся чуть подробнее на присоединении таблиц, для которого предусмотрены два метода leftJoin() и innerJoin(). А где же присоединение справа спросите вы, оно как считают разработчики не нужно, ведь можно так спроектировать запрос, чтобы присоединение справа не было нужно.

``` php
<?php
  $query = Doctrine_Query::create()
       ->from('Catalog_Model_Country co')
       ->leftJoin('co.Catalog_Model_City ci')
       ->where('ci.id = ?', 10)
  $result = $query->execute();
?>
```

  Поля необходимые для соединения таблиц автоматически устанавливаются Doctrine, основываясь на связях, определенных в методе setUp модели.





>    Небольшой совет: используя DQL, иногда необходимо посмотреть что же там получилось, для этого и существует метод getSqlQuery(), который и возвращает сырой SQL.








### Doctrine и CRUD


Сейчас научимся элементарным операциям аля create, read, update и delete (CRUD).
Для добавление новой записи, вам необходимо создать экземпляр класса, заполнить все поля (свойства) и вызывать метод save(). Вот пример

``` php
<?php
  $city = new Model_City();
  $city->Name = 'New City';
  $city->CountryCode = 'RUS';
  $city->District = 'New City';
  $city->Population = 1;
  $city->save();
?>
```


Изменим-ка что-нибудь в уже существующей записи. Для начала найдем ту, в которой и будем что-либо менять, затем сохраним, вызвав save() метод.

``` php
<?php
  $city = Doctrine::getTable('Model_City')->find(10);
  $city->Name = 'Updated city';
  $city->save();
?>
```

Удалять записи также легко, как и создавать новые, только вместо save() нужно использовать delete().





### Вернемся к Zend Framework'у


Чтобы закрепить знания, давайте попробуем все это дело привести в более приятный вид. Сделаем все это в IndexController, чтобы не плодить тестовых контроллеров. Вот его код.

``` php
<?php
class IndexController extends Zend_Controller_Action
{
  public function init() {}

  public function showAction()
  {
    $filters = array(
      'id' => array('HtmlEntities', 'StripTags', 'StringTrim')
    );

    $validators = array(
      'id' => array('NotEmpty', 'Int')
    );

    $input = new Zend_Filter_Input(
      $filters,
      $validators,
      $this->getRequest()->getParam('id')
    );

    if ($input->isValid()) {
      $query = Doctrine_Query::create()
            ->from('Model_Country co')
            ->leftJoin('co.Model_City ci')
            ->leftJoin('co.Model_CountryLanguage cl')
            ->where('ci.id = ?', $input->id);

      $this->view->query = $query->getSql();
      $result = $query->fetchArray();
      if (count($result) == 1) {
     $this->view->item = $result[0];
      } else {
     throw new Zend_Controller_Action_Exception('Страница не найдена');
      }
    } else {
      throw new Zend_Controller_Action_Exception('Неверный параметр');
    }
  }
}
?>
```


И для этого дела приделаем способ отображение а-ля index/show.phtml

``` php
<?php
<table>
  <tr>
    <th>County name</th>
    <th>Continent</th>
    <th>Country population</th>
    <th>City name</th>
    <th>LAnguage</th>
  </tr>
  <tr>
    <td><?php echo $this->item['Name'] ?></td>
    <td><?php echo $this->item['Continent'] ?></td>
    <td><?php echo $this->item['Population'] ?></td>
    <td><?php echo $this->item['Model_City']['Name'] ?></td>
    <td><?php echo $this->item['CountryLanguage']['Language'] ?></td>
  </tr>
</table>
?>
```


Теперь когда вы зайдете по адресу yourhost.local/index/show/id/10, вы увидите часть информации о нем.

Вот и подошла к концу очередная заметка на тему веб-программирования, так мне интересную, надеюсь, вам понравилось.

И кстати, если кто увидит ошибки, прошу сообщать, очень не уверен, что правильно расставил связи один-ко-многим и один-к-одному. Заранее благодарю откликнувшихся.
