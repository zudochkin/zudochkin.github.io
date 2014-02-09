---
author: admin
comments: true
date: 2011-03-22 19:10:12+00:00
layout: post
slug: zend-framework-wysiwyg-filebrowser

permalink: /2011/03/zend-framework-wysiwyg-filebrowser

title: 'Zend Framework: WYSIWYG + FileBrowser'
wordpress_id: 684
categories:
- cms
- jQuery
- php
- Zend Framework
tags:
- jQuery
- jQuery UI
- wysiwyg
- Zend Framework

keywords: "zend framework filebrowser, zend framework wysiwyg, zend framework elrte, zend framework elfinder, zf elrte, zf elfinder, zf wysiwyg,jquery,jquery ui,wysiwyg,zend framework,cms,php"
description: "Этот пост будет посвящен php фреймворку от Zend'а, точнее интеграции визуального редактора и файлового менеджера (elRTE и elFinder)."
---

Здравствуй, неизвестный читатель. На просторах интернета я не встретил толковой реализации, кроме одной, ссылку на которую я вставлю в конце поста.
После проделывания всех шагов, описанных в этом посте, у вас будет иметься возможность вставлять в форму визуальный редактор с файловым менеджером парой строк<!-- more -->:



``` php
<?php
    $wysiwyg = new Vredniy_Form_Element_WysiwygElrte('content', array(
        'label' => 'Контент'
    ));

   $this->addElement($wysiwyg);
?>
```




Не правда ли удобно? Итак, начнем все подробно разбирать.

##### Шаг номер 1.

Качаем [elrte](http://elrte.org/) и [elFinder](http://elrte.org/elfinder) с официального сайта. Здесь ни у кого не должно возникнуть сложностей. Дальше, чтобы не путаться в путях предлагаю эти библиотеки разместить как у меня. **/public/elrte** и **/public/elfinder** соответственно.


##### Шаг номер 2.

Создаем кастомизированный элемент формы **Vredniy_Form_Element_WysiwygElrte** (над названием, скорей всего нужно будет подумать)


``` php
<?php
class Vredniy_Form_Element_WysiwygElrte extends Zend_Form_Element_Xhtml
{
    public $helper = 'formWysiwygElrte';
}
?>
```

##### Шаг номер 3.

Этот элемент формы будет вести себя как стандартный зендовский. Теперь напишем свой вью хелпер, который и будет заниматься всей грязной работой: подключать все необходимые скрипты, стили и генерировать контент исходного элемента формы.



``` php
<?php
class Vredniy_View_Helper_FormWysiwygElrte extends Zend_View_Helper_FormElement
{

    public function formWysiwygElrte($name = null, $value = null, $attribs = null)
    {
        if (!$this->view->jQuery()->isEnabled())
                $this->view->jQuery()->enable();

        if (!$this->view->jQuery()->uiIsEnabled())
                $this->view->jQuery()->uiEnable();


        $elrte_base_uri = "/js/elrte/";

        $this->view->headLink()->appendStylesheet("{$elrte_base_uri}css/elrte.full.css");
        $this->view->headScript()->appendFile("{$elrte_base_uri}js/elrte.full.js");
        $this->view->headScript()->appendFile("{$elrte_base_uri}js/i18n/elrte.ru.js");


        $elfinder_base_uri = "/js/elfinder/";
        $this->view->headLink()->appendStylesheet("{$elfinder_base_uri}css/elfinder.css");
        $this->view->headScript()->appendFile("{$elfinder_base_uri}js/elfinder.full.js");
        $this->view->headScript()->captureStart() ?>
            var opts = {
                lang : 'ru',
                styleWithCss : false,
                absoluteURLs: false, // чтобы отображался не полный путь, а относительный от корня
                width   : 800,
                height  : 200,
                toolbar : 'maxi', // все возможные панели инструментов, удалить легче :)
                fmAllow  : true,
                fmOpen   : function(callback) {
                    $('<div id="myelfinder" />').elfinder({
                        url : '<?php echo $elfinder_base_uri; ?>connectors/php/connector.php',
                        lang : 'ru',
                        dialog : {
                            width : 900, // ширина файлового менеджера
                            modal : true,
                            title : 'Выбираем файлик'
                        }, // открываем в диалоговом окне
                        closeOnEditorCallback : true, // закрываем после выбора файла
                        editorCallback : callback
                    })
                }
            };

            $().ready(function() {
              // создаем редактор
                $('#<?php echo $name; ?>').elrte(opts);
            });
        <?php $this->view->headScript()->captureEnd();

        $info = $this->_getInfo($name, $value, $attribs);
        extract($info); // name, value, attribs, options, listsep, disable

        // is it disabled?
        $disabled = '';
        if ($disable) {
            // disabled.
            $disabled = ' disabled="disabled"';
        }

        // build the element
        $xhtml = '<textarea name="' . $this->view->escape($name) . '"'
                . ' id="' . $this->view->escape($id) . '"'
                . $disabled
                . $this->_htmlAttribs($attribs) . '>'
                . $this->view->escape($value) . '</textarea>';

        return $xhtml;

    }

}

?>
```


В исходном коде я старался максимально комментировать не особо очевидные моменты, кроме подключения jQuery и jQueryUI (как их подключить расскажу в конце, чтобы не отвлекать вас от основной идеи).

##### Шаг номер 4.

Изменяем опции коннектора, который отвечает за файловый менеджер **public/js/elfinder/php/connector.php**

``` php
<?php
// из основного файла ZF index.php я вынес в отдельный
// константы, чтобы было удобно их подключать извне
require_once realpath(dirname(__FILE__) . '/../../../../constants.php');
error_reporting(0); // Set E_ALL for debuging

if (function_exists('date_default_timezone_set')) {
   date_default_timezone_set('Europe/Moscow');
}

include_once dirname(__FILE__).DIRECTORY_SEPARATOR.'elFinder.class.php';

/**
 * Simple example how to use logger with elFinder
 **/
class elFinderLogger implements elFinderILogger {

   public function log($cmd, $ok, $context, $err='', $errorData = array()) {
      if (false != ($fp = fopen('./log.txt', 'a'))) {
         if ($ok) {
            $str = "cmd: $cmd; OK; context: ".str_replace("\n", '', var_export($context, true))."; \n";
         } else {
            $str = "cmd: $cmd; FAILED; context: ".str_replace("\n", '', var_export($context, true))."; error: $err; errorData: ".str_replace("\n", '', var_export($errorData, true))."\n";
         }
         fwrite($fp, $str);
         fclose($fp);
      }
   }

}

$opts = array(
    'root'            => APPLICATION_PUBLIC . '/upload/files', // path to root directory
    'URL'             => '/upload/files/', // root directory URL
    'rootAlias'       => 'Галерея', // display this instead of root directory name
);

$fm = new elFinder($opts);
$fm->run();
?>
```


Тут самое интересное это массив **$opts**, в котором мы задаем корневую папку нашего файлового хранилища и меняем название "Home" на более привычное русскому глазу "Галерея".


##### Шаг номер 5 (костыль).

У меня не получилось "из коробки" заставить принимать правильный размер диалоговое окно, поэтому я заменил 6303 строчку в файле /js/elrte/js/elrte.full.js на свое


``` javascript
width    : 670,
// @todo: чтобы помещалась волшебная кнопочка
```


##### Шаг номер 6.

Подключаем скрипты jQuery, jQuery и необходимые для них стили. Корневой файл **Bootstrap.php**

``` php
<?php
    protected function _initViewHelpers()
    {
        $this->bootstrap('layout');
        $layout = $this->getResource('layout');
        $view = $layout->getView();
        $viewRenderer = Zend_Controller_Action_HelperBroker::getStaticHelper('viewRenderer');
        $viewRenderer->view->addHelperPath("Vredniy/View/Helper", 'Vredniy_View_Helper');

        // init 960gs-fluid
        $view->headLink()->appendStylesheet('/css/960gs-fluid/reset.css')
                ->headLink()->appendStylesheet('/css/960gs-fluid/text.css')
                ->headLink()->appendStylesheet('/css/960gs-fluid/grid.css')
                ->headLink()->appendStylesheet('/css/960gs-fluid/layout.css')
                ->headLink()->appendStylesheet('/css/960gs-fluid/ie/ie6.css', 'all', 'IE6')
                ->headLink()->appendStylesheet('/css/960gs-fluid/ie/ie.css', 'all', 'IE')
        ;


        // init jQuery & UI
        ZendX_JQuery::enableView($view);
        $view->jQuery()
                ->setLocalPath('/js/jquery/jquery-1.4.4.min.js')
                ->setUiLocalPath('/js/jqueryui/jquery-ui-1.8.10.custom.min.js')
                ->addStylesheet('/css/jqueryui/themes/start/jquery-ui-1.8.10.custom.css')
        ;

        // set doctype
        $view->doctype('HTML5');

        // set title separator
        $view->headTitle()->setSeparator(' :: ');
    }
?>
```


jQuery располагаем в папку **/js/jquery/**, jQueryUI - в **/js/jquery/ui** и темы в **/css/jqueryui/themes/**


И последнее, разобьем наш файл /public/index.php на два: сам файл

``` php
<?php
require_once 'constants.php';
// Ensure library/ is on include_path
set_include_path(implode(PATH_SEPARATOR, array(
    realpath(APPLICATION_PATH . '/../library'),
    get_include_path(),
)));

/** Zend_Application */
require_once 'Zend/Application.php';

// Create application, bootstrap, and run
$application = new Zend_Application(
    APPLICATION_ENV,
    APPLICATION_PATH . '/configs/application.ini'
);
$application->bootstrap()
            ->run();


?>
```


и **constants.php**

``` php
<?php

// Define path to application directory
defined('APPLICATION_PATH')
        || define('APPLICATION_PATH', realpath(dirname(__FILE__) . '/../application'));

// Define application environment
defined('APPLICATION_ENV')
        || define('APPLICATION_ENV', (getenv('APPLICATION_ENV') ? getenv('APPLICATION_ENV') : 'production'));

// public folder
defined('APPLICATION_PUBLIC')
        || define('APPLICATION_PUBLIC', realpath(APPLICATION_PATH . '/../public'));


?>
```

Дальше используем. Создадим форму с 2мя элементами: нашим wysiwyg'ом и простую кнопку submit. Сказано-сделано.

``` php
<?php

class Static_Form_Page extends Zend_Form
{

    public function init()
    {
        $wysiwyg = new Vredniy_Form_Element_WysiwygElrte('content', array(
                    'label' => 'Контент'
                ));
        // @todo add filters and validators

        $this->addElement($wysiwyg);
        $this->addElement('submit', 'submit', array(
            'label' => 'Change me'
                // @todo add filters and validators
        ));
    }

}

?>
```


И в каком-нибудь контроллере напишем следующее:

``` php
<?php

class Static_AdminController extends Zend_Controller_Action
{

    public function indexAction()
    {
        $form = new Static_Form_Page();
        $form->getElement('submit')->setLabel('Создать');
        $this->view->form = $form;
        if ($this->getRequest()->isPost()) {
            if ($form->isValid($this->getRequest()->getPost())) {
                var_dump($form->getValues());
            }
        }
    }

}
?>
```

В соответствующем скрипте вида напишем пару строк для вывода формы

``` php
Форма с Wysisyg и FileBrowser'ом
<?php echo $this->form; ?>
```


Спасибо за потраченное на чтение сего поста время, надеюсь, информация вам оказалась полезной. Мне данной действо очень облегчило жизнь и я стал ближе на одну ступень к изобретению своего велосипеда - собственной CMS.

[{% img image /images/posts/2011-03-zend-framework-wysiwyg-filebrowser/elrte-300x164.jpg %}](/images/posts/2011-03-zend-framework-wysiwyg-filebrowser/elrte.jpg)

Ссылка на [пост](http://bit.ly/fDlu4C), которая собственно и подтолкнула меня на все работу. Автору того поста большое спасибо.

