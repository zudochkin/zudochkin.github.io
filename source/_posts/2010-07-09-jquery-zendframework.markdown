---
author: admin
comments: true
date: 2010-07-09 19:16:27+00:00
layout: post
slug: jquery-zendframework
title: jQuery и Zend Framework - красивый симбиоз
wordpress_id: 539
categories:
- jQuery
- Zend Framework
tags:
- jQuery
- jQuery UI
- viewhelper
- Zend Framework
---

[![jquery-zf](http://vredniy.ru/wp-content/uploads/2010/07/jquery-zf-150x150.jpg)](http://vredniy.ru/wp-content/uploads/2010/07/jquery-zf.jpg)Приветствую вас, веб-разработчики. В сегодняшней заметке мы с вами научимся встраивать в наши проекты на Zend Framework jQuery. После этой заметки вы поймете, что это очень просто. Для начала идем на сайт jqueryUI и скачиваем скрипты и стили в одном флаконе, благо на сайте все это предумотрено. Нам оттуда потребуется 2 JavaScript файла: jquery и jqueryUI, файл стилей и картинки, которые украсят наш динимический сайт. На картинке можно разглядеть какая структура получилась у меня. (папка **public **или document_root сайта)<!-- more -->

В создании нового проекта на Zend Framework вам очень поможет [эта](/2010/05/zend-framework-project-creation-netbeans/) заметка, если вы используете NetBeans как среду разработки.

В первую очередь нужно включить слои **enable layout** и настроить наш загрузчик **bootstrap.php**, который находится в APPLICATION_PATH. Вот каким он получился у меня
[cc lang="php"]
bootstrap('layout');
        $layout = $this->getResource('layout');
        $view = $layout->getView();
        $viewRenderer = Zend_Controller_Action_HelperBroker::getStaticHelper('viewRenderer');

        // добавляем поддежрку jQuery
        $viewRenderer->view->addHelperPath(APPLICATION_PATH . "/views/helpers", 'My_Helper');
        $view->addHelperPath('ZendX/JQuery/View/Helper', 'ZendX_JQuery_View_Helper');

        // задаем локальные пути для скриптов и стилей
        ZendX_JQuery::enableView($view);
        $view->jQuery()
                ->enable()
                ->uiEnable()
                ->setLocalPath('/js/jquery/jquery-1.4.2.min.js')
                ->setUiLocalPath('/js/jquery/jquery-ui-1.8.2.custom.min.js')
                ->addStylesheet('/styles/jquery-ui-1.8.2.custom.css');

        // инициализация doctype
        $view->doctype('XHTML1_STRICT');

        // настройка title
        $view->headTitle()->setSeparator(' - ')->headTitle('ZF 4 fun');
    }

}
[/cc]
Старался комментировать важные моменты. А еще, чуть не забыл, вам понадобится библиотека из ZF ZendX, которая находится в папке /extras/library/ZendX распакованного архива с Zend Framework.
Теперь создадим лейаут для всего проекта, подкорректировав файл layout.phtml.
[cc lang="html"]
 ?>
doctype(); ?>

    
    
        layout()->content; ?>
    

[/cc]
Добавим немного магии в виде добавочного тайтла в контроллер по умолчанию IndexController.
[cc lang="php"]
public function indexAction()
{
    $this->view->headTitle('Index Controller');
}[/cc]
А сейчас самое интересное, покажем всю красу jQuery в виде datePicker'а. Для этого нам нужно в файле, отвечающем за вывод действия и контроллера по умолчанию **views/scripts/index/index.phtml** написать следующее 
[cc lang="php"]
datePicker('dp1'); ?>
[/cc]
Махмут, поджигай или запускаем и смотрим что получилось. На первый взгляд отображается только текстовое поле без ничего более, но это лишь на первый взгляд, попробуйте щелкнуть по этому полю и вуаля отобразится красивые календарик[![jquery-datepicker-zf](http://vredniy.ru/wp-content/uploads/2010/07/jquery-datepicker-zf-150x150.png)](http://vredniy.ru/wp-content/uploads/2010/07/jquery-datepicker-zf.png) Если заглянуть в исходный код, то можно будет увидеть, что всю черную работу хелперы вида Zend Framework'а сделали за нас. Вот и все =)
