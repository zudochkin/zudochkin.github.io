---
author: admin
comments: true
date: 2010-07-10 08:38:01+00:00
layout: post
slug: zend-framework-wysiwyg

permalink: /2010/07/zend-framework-wysiwyg

title: Zend Framework и WYSIWYG (CKEditor)
wordpress_id: 554
categories:
- javascript
- Zend Framework
- программирование
tags:
- wysiwyg
- Zend Framework
---

[![wysiwyg-zendframework](http://vredniy.ru/wp-content/uploads/2010/07/wysiwyg-zendframework1-150x150.png)](http://vredniy.ru/wp-content/uploads/2010/07/wysiwyg-zendframework1.png)Сегодня я расскажу вам как встроить в своей проект визуальный редактор, так часто встречающийся на многих сайтах. Он может применяться в разных местах, будь то редактирование статей/новостей на сайте, может заменить простую форму для отправления комментариев. Возможности ограничиваются лишь вашей фантазией. <!-- more -->

Перво-наперво скачиваем CKEditor с официального [ сайта](http://ckeditor.com/) и копируем в папку public нашего проекта, точнее в public/js/ckeditor, чтобы визуальный редактор лежал в отдельной папке дабы не запутаться с остальными скриптами. Я оттуда ничего не удалял, ведь это ознакомительная версия, на продакшне легко можно удалить большую половину файлов: примеры, скины, ненужные языки локализации. 

Дальше создаем папку Izwt (здесь вы можете назвать ее как хотите, главное не забыть потом поменять название в том месте, где встречается этот префикс) в папке library, в которой будут располагаться дополнительные классы, которые мы можем переносить от проекта к проекту. И настраиваем автозагрузку, чтобы помимо Zend_ классов, загружались еще и наши Izwt_. Для этого в загрузчике bootstrap.php пропишем следующее.
[cc lang="php"]
protected function _initAutoload()
{
    $autoloader = Zend_Loader_Autoloader::getInstance();
    $autoloader->registerNamespace('Izwt_');
    $moduleLoader = new Zend_Application_Module_Autoloader(array(
                'namespace' => '',
                'basePath' => APPLICATION_PATH
            ));
    return $moduleLoader;
}

protected function _initViewHelpers()
{
    $loader = new Zend_Loader_PluginLoader();
    $loader->addPrefixPath('Izwt_View_Helper', 'Izwt/View/Helper');
    $viewRenderer = Zend_Controller_Action_HelperBroker::getStaticHelper('viewRenderer');
    $viewRenderer->view->addHelperPath('Izwt/View/Helper', 'Izwt_View_Helper');
    return $loader;
}[/cc]
**_initAutoload()** зарегистрирует наш префикс и, если следовать стандарам Zend по наименованию, будет загружать наши файлы автоматически. **_initViewHelpers()** укажет где искать помощники вида.

Далее создадим класс Izwt_Form_Element_Wysiwyg, который будет расширять Zend_Form_Element_Xhtml нашим помощником вида.
**library/Izwt/Form/Element/Wysiwyg.php**
[cc lang="php"]
library/Izwt/View/Helper/FormWysiwyg.php**
[cc lang="php"]
_getInfo($name, $value, $attribs);
        extract($info); // name, value, attribs, options, listsep, disable

        $editor = new CKEditor();
        // пусть возвращает значение, а не выводит в браузер
        $editor->returnOutput = true;

        // задаем базовый путь к визуальному редактору
        $editor->basePath = '/js/ckeditor/';
        
        // ширина редактора
        $editor->config['width'] = 600;

        // $value содержит значение по умолчанию
        return $editor->editor('wysiwyg', $value);
    }

}[/cc]

Дальше все должно быть элементарно просто, поэтому не буду заострять ваше внимания на этом, просто приведу исходный код.

Наша форма **application/forms/Wysiwyg.php**
[cc lang="php"] 'form.label.wysiwyg',
                    'required' => true,
                    'filters' => array('StringTrim')
                ));

        $sumbit = $this->createElement('submit', 'submit', array(
                    'label' => 'form.label.submit'
                ));

        $this->addElements(array(
            $wysiwyg, $sumbit
        ));
    }

}[/cc]
В 8 строке мы создаем наш элемент, как обычный элемент зендовской формы.

Наш контроллер **application/controllers/IndexController.php**
[cc lang="php"]
view->headTitle('встраиваем Wysiwyg');
        $form = new Form_Wysiwyg();
        if ($form->isValid($_POST)) {
            $this->view->message = $form->getValues();
        } else {
            $this->view->form = $form;
        }
    }

}[/cc]

И собственно сам шаблон, который все это дело и будет отображать. **application/views/scripts/index/index.phtml**
[cc lang="php"]
form)): ?>
form; ?>


message); ?>
[/cc]

Вот и все, запускаем и радуемся, интересных вам проектов =)

