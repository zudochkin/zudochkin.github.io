---
author: admin
comments: true
date: 2011-08-26 12:29:19+00:00
layout: post
slug: php-sinatra-closures

permalink: /2011/08/php-sinatra-closures

title: 'PHP замыкания и немного рефлексии '
wordpress_id: 770
categories:
- php
tags:
- framework
- php
---

Это будет очень короткая заметка и больше экспериментальная. Вчера вечером смотрел Sinatra (фреймворк на базе Ruby) и меня он покорил своей легкостью. Да вообще мне очень нравится Ruby, обязательно его в скором будущем выучу, а сейчас поговорим о PHP. Заметка сводится к тому, что я попытался реализовать функционал Sinatra на PHP. Таких велосипедов, скажите вы, превеликое множество и несомненно будете правы. Но я нашел в сегодняшнем велосипедостроение очень много для себя полезного. Да и "фреймворк" (фреймворком пока назвать его очень сложно, но начало положено) получился очень маленьким - около 40 строк. 



### Описание фреймворка


Фреймворк может обрабатывать GET запросы, учитывая маршрутизацию, которая задется регулярным выражением с любым количеством параметров. С выходом 5.3 версии PHP нам стали доступны так называемые замыкания или анонимные функции (кому как нравится). Вот именно и благодаря замыканиям фреймворк получился очень маленьким. Когда придумывал ему название (ведь каждый программист должен назвать свое детище), в интернете натолкнулся на картинку жирафа, так и обрел название мой фреймворк (Jirafa на испанский манер).
<!-- more -->


### Использование


![PHP analog Sinatra](http://vredniy.ru/wp-content/uploads/2011/08/php-sinatra-300x230.png)Практическим назначением фреймворк не может похвастаться, но в возможном будущем он обрастется функционалом, и тогда появится реальная необходимость его использовать. А сейчас это наглядное пособие по использованию замыканий. 

Чтобы создать приложение нужно создать класс приложения и назначить на определенные маршруты функции, которые и будет обрабатывать запрос.
[cc lang="php"]
$app = new Jirafa();

$app->setRegistry(array(
	'viewObject' => 'may be smarty',
	'dbObject' => new PDO('sqlite:./db/db.db') //'may be pdo or something else'
));

[/cc]

Метод setRegistry() используется мною для того, чтобы передать во все функции необходимые параметры, это могут быть и шаблонизаторы, и PDO объекты, да вообще все что угодно можно передать. А дальше несколько строк кода для демонстрации.

[cc lang="php"]
$app->get('/', function() use ($registry) { // показываем главную
						return '  
1hi ';
					})
	->get('404', function($registry) { // отображается, если ни один маршрут не подошел
						var_dump($registry);
						return '

# Error 404

';
					})
	->get('(\w+).html', function ($alias, $registry) { // example.com/static.html
						$pdo = $registry['dbObject'];
						$html = 'Error';
						try {
							$row = $pdo->query("SELECT * FROM content WHERE alias='{$alias}'")->fetch();
							$html = "

# {$row['title']}

{$row['content']}

";
						} catch (PDOException $e) {
							return $e->getMessage();
						}
						return $html;
					})
[/cc]

Я считаю, что все должно быть предельно понятно, поэтому вернемся к классу фреймворка (на данный момент "фреймворк состоит из одного файла, двух, если считать .htaccess)



### Исходный код фреймворка


[cc lang="php"]
class Jirafa 
{
    protected $_app = array();
	protected $_registry;
	
	public function get() {
		$args = func_get_args();
        $this->_app[array_shift($args)] = array_shift($args);
		
        return $this;
    }    
    
    public function render() {	
		$uri = ($_SERVER['REQUEST_URI'] === '/') ? '/' : substr($_SERVER['REQUEST_URI'], 1);
		// iterate over all paths
		foreach($this->_app as $path => $func) {
			// if $uri and regular expression are matched
			if (preg_match('@^' . $path . '\/*$@', $uri, $matches)) {
				var_dump($path);
				array_shift($matches);
				$countInPath = count($matches);
				
				$reflection = new ReflectionFunction($func);
				$params = $reflection->getParameters();
				
				$array = array();
				foreach($params as $param) 
					$array[$param->getName()] = array_shift($matches);
				
				//if (count($array) !== $countInPath + 1)
					//goto error;
				
				echo call_user_func_array($func, array_merge($array, array('registry' => $this->_registry)));
				return;
			} 
		}
		//error:
			echo call_user_func_array($this->_app['404'], array('registry' => $this->_registry));
    }
	
	public function setRegistry($registry) {
		$this->_registry = $registry;
	}
}
[/cc]

Метод get() в нем очень простой, он записывает обрабатываемый маршрут и параметры, с которыми данный метод был вызван (понадобится нам в дальнейшем для заполнения параметров в функции). Для удобство возвращаем свой же объект, чтобы сократить код приложения. 

Метод render() находит соответствие сохраненных методом get() маршрутов и $_SERVER['REQUEST_URI']. Если соответствие найдено, то мы с помощью рефлексии (Reflection) получаем данные нашего замыкания (как называются переменные и их количество). Далее с помощью замечательного метода call_user_func_array(), позволяющего вызывать функцию или метод с переменным количество аргументов, мы вызываем функцию нашего исключения, передавая ей в качестве параметров, помимо запрашиваемых, еще и переменную $this->_registry, которая, как я уже упоминал выше, может содержать вспомогательную информацию, необходимую для работы приложения. 



### Заключение


В данной заметки мы вскользь познакомились с замыканиями и рефлексией. Изобрели еще один маленький велосипед с квадратными колесами, которые, надеюсь, поднял вам настроение и прибавил вам немного практики и мыслей по поводу дальнейшего использования полезных конструкция замечательного языка PHP. Удачи вам, веб-девелоперы :)
