---
author: admin
comments: true
date: 2010-11-27 18:02:20+00:00
layout: post
slug: mvc-models-doctrine-zend-framework

permalink: /2010/11/mvc-models-doctrine-zend-framework

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

[![zend-framework-doctrine](http://vredniy.ru/wp-content/uploads/2010/11/zend-framework-doctrine1-150x150.jpg)](http://vredniy.ru/wp-content/uploads/2010/11/zend-framework-doctrine1.jpg)
Давайте напишем пару слов о популярном сейчас ORM Doctrine, научимся устанавливать его, использовать в своих проектах наряду с Zend Framework'ом. 
<!-- more -->



### Установка Doctrine


  В первую очередь скачиваем с официального сайта [Doctrine](http://www.doctrine-project.org/projects/orm/download), я свое ознакомление начал со стабильной версии 1.2, хотя для скачивания доступна вторая версия (бета), думаю ее "пощупаю" немного попозже. Распаковываем ее и копируем папку lib из Doctrine-* в include_path, я поместил ее в /usr/share/php/. Теперь напишем первый скрипт, который проверит работу Doctrine. Хотя подождите, подготовим сначала базу данных и наполним ее данными для тестов. За неимением лучшего я всего скачиваю mysql wordl database (на этот раз это будет innoDB версия) по [ссылке ](http://dev.mysql.com/doc/index-other.html) и создаю пользователя для тестов с паролем и логином таким же, как и название базы данных. У меня получился пользователь world с паролем world и база данных тоже называется world, теперь приступим.

[cc lang="php"]
connection('mysql://world:world@localhost/world', 'doctrine');
  
  // получаем список всех баз данных
  $databases = $conn->import->listDatabases();
  var_dump($databases);
?>
[/cc]

Дальше запускаем наш скрипт и видим, если все сделали правильно, список всех баз данных.





### Генерация моделей Doctrine


Так как Doctrine идет в комплекте с мощным генератором моделей, который может "читать" существующую базу данных и создавать классы моделей. Чтобы увидеть это в действии, запустим  следующее.

[cc lang="php"]
connection('mysql://world:world@localhost/world', 'doctrine');

  // генерируем модели в папке со скриптом
  // 2 параметр соединение, которое мы указывали на строчку выше
  // 3 префикс для названия файлов и классов
  Doctrine::generateModelsFromDb('./models', 
    array('doctrine'),
    array('classPrefix => 'Model_', 'classPrefixFiles' => false)
  );
?>
[/cc]
К сведению, Doctrine для каждой таблицы в базе данных создает 2 класса модели, базовый класс и класс-наследник. Базовый класс расширяет Doctrine_Record и включает определения таблицы и базовые операции по работе с таблицей. Класс-наследник - это место для написания дополнительного функционала, который мы собираемся добавить в модель.

Теперь скопируем все получившиеся модели (их должно получиться 6 штук) в наш Zend Framework проект в папку application/models. 




### Настраиваем модельные связи


[![doctrine-zend-framework](http://vredniy.ru/wp-content/uploads/2010/11/doctrine-300x194.png)](http://vredniy.ru/wp-content/uploads/2010/11/doctrine.png)
Doctrine - очень мощный ORM, но она не может справиться с установлением связей между таблицами, чем мы займемся сами. Doctrine поддерживает все виды связи один-к-одной, один-ко-многим, многие-ко-многим и рекурсивную используя всего 2 метода hasOne() и hasMany(). У каждого метода два аргумента: имя другой модели для объявления связи и массив опций, характеризующих связь. Для того, чтобы связи автоматически загружались поместим их в setUp() метод.

**models/City.php**
[cc lang="php"]hasOne('Model_Country', array(
	      'local' => 'CountryCode',
	      'foreign' => 'Code'
	  ));
      }

  }
?>[/cc]

**models/Country.php**
[cc lang="php"]hasOne('Model_City', array(
	      'local' => 'Code',
	      'foreign' => 'CountryCode'
	  ));
	  $this->hasOne('Model_CountryLanguage', array(
	      'local' => 'Code',
	      'foreign' => 'CountryCode'
	  ));
      }

  }
?>[/cc]

**models/CountryLanguage.php**
[cc lang="php"]hasOne('Model_Country', array(
	      'local' => 'CountryCode',
	      'foreign' => 'Code'
	  ));
      }

  }
?>[/cc]




### Автозагрузка классов Doctrine в Zend Framework


Все что осталось сделать, так это то, чтобы приложение Zend Framework и Doctrine хорошо жили совместно. В первую очередь поместим Doctrine в папку library Zend Framework'а. Следующая стадия - обновить конфигурационный файл application.ini, поместив туда строку DSN, чтобы Doctrine могла без проблем соединяться с базой данных. Допишем в [production] секцию 
[cc lang="php"]  doctrine.dsn = "mysql://world:world@localhost/world"[/cc]

Финальным шагом для инициализации Doctrine напишем защищенный метод в нашем бутстрапе.
**application/Bootstrap.php**
[cc lang="php"]getApplication()
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
?>[/cc]
  



### Операции с моделью Doctrine


До того, как погрузиться в кроличью, освоим элементарные операции с моделями. 
Извлечение данных
... по ID:
[cc lang="php"]find(10);
  var_dump($city);
?>[/cc]
... всех записей:
[cc lang="php"]findAll();
  var_dump($cities);
?>[/cc]

Также в Doctrine есть свой язык запросов DQL (Doctrine Query Language), который предоставляет гибкий интерфейс для создания и выполнения запросов. Посмотрим на примеры.
[cc lang="php"]from('Model_City c');
  $result = $query->fetchArray();
?>[/cc]
Использование условий
[cc lang="php"]from('Model_City c');
	    ->where('c.id = ?', 10);
  $result = $query->fetchArray();
?>[/cc]

В DQL много приятных плюшек, среди которых группировка и сортировка, но мы остановимся чуть подробнее на присоединении таблиц, для которого предусмотрены два метода leftJoin() и innerJoin(). А где же присоединение справа спросите вы, оно как считают разработчики не нужно, ведь можно так спроектировать запрос, чтобы присоединение справа не было нужно.
[cc lang="php"]from('Catalog_Model_Country co')
	    ->leftJoin('co.Catalog_Model_City ci')
	    ->where('ci.id = ?', 10)
  $result = $query->execute();
?>[/cc]
  Поля необходимые для соединения таблиц автоматически устанавливаются Doctrine, основываясь на связях, определенных в методе setUp модели.





>    Небольшой совет: используя DQL, иногда необходимо посмотреть что же там получилось, для этого и существует метод getSqlQuery(), который и возвращает сырой SQL.








### Doctrine и CRUD


Сейчас научимся элементарным операциям аля create, read, update и delete (CRUD).
Для добавление новой записи, вам необходимо создать экземпляр класса, заполнить все поля (свойства) и вызывать метод save(). Вот пример
[cc lang="php"]Name = 'New City';
  $city->CountryCode = 'RUS';
  $city->District = 'New City';
  $city->Population = 1;
  $city->save();
?>[/cc]

Изменим-ка что-нибудь в уже существующей записи. Для начала найдем ту, в которой и будем что-либо менять, затем сохраним, вызвав save() метод.
[cc lang="php"]find(10);
  $city->Name = 'Updated city';
  $city->save();
?>[/cc]
Удалять записи также легко, как и создавать новые, только вместо save() нужно использовать delete().





### Вернемся к Zend Framework'у


Чтобы закрепить знания, давайте попробуем все это дело привести в более приятный вид. Сделаем все это в IndexController, чтобы не плодить тестовых контроллеров. Вот его код.
[cc lang="php"] array('HtmlEntities', 'StripTags', 'StringTrim')
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
?>[/cc]

И для этого дела приделаем способ отображение а-ля index/show.phtml
[cc lang="php"]
  


    County name
    Continent
    Country population
    City name
    LAnguage
  
  


    
item['Name'] ?>

    
item['Continent'] ?>

    
item['Population'] ?>

    
item['Model_City']['Name'] ?>

    
item['CountryLanguage']['Language'] ?>

  

?>[/cc]

Теперь когда вы зайдете по адресу yourhost.local/index/show/id/10, вы увидите часть информации о нем.

Вот и подошла к концу очередная заметка на тему веб-программирования, так мне интересную, надеюсь, вам понравилось.

И кстати, если кто увидит ошибки, прошу сообщать, очень не уверен, что правильно расставил связи один-ко-многим и один-к-одному. Заранее благодарю откликнувшихся.
