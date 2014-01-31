---
author: admin
comments: true
date: 2011-05-16 11:02:21+00:00
layout: post
slug: view-helper-wherever-u-want

permalink: /2011/05/view-helper-wherever-u-want

title: 'Zend Framework: вставляем viewHelper куда душе угодно'
wordpress_id: 708
categories:
- cms
- php
- Zend Framework
---

[![zf-action-helper-content-replacer](http://vredniy.ru/wp-content/uploads/2011/05/content-replacer-150x150.png)](http://vredniy.ru/wp-content/uploads/2011/05/content-replacer.png)


Здравствуйте, уважаемые коллеги-разработчики, в сегодняшней заметке я поделюсь с вами своим костылем-решением. Пишу в свободное время свою модульную CMS (ведь каждый разработчик должен написать в своей жизни хотя бы одну) на базе Zend Framework'а и возникла необходимость вставлять в контент форму обратной связи и еще что-нибудь в произвольное место. Для создания стратичных страниц я использую свой модуль Static, который ничего не умеет, кроме создания стратичных страниц WYSIWYG'ом и прикрепления к уже имеющимся. 


<!-- more -->


Первое, что пришло в голову это заменять какой-нибудь placeholder аля {placeholder_feedback_form(0)} на содержимое. Сказано-сделано, приведу кусок view Helper'а, который отображает нестандартную форму и валидирут посредстом отправления AJAX запросов на сервер.






### View_Helper: на что будем менять


[cc lang="php"]

                


                        Ваше имя
                        


                        Ваш E-mail
                        


                        Ваш телефон
                        


                        


                        


                        


                


        
karamba;
        $view = Zend_Controller_Action_HelperBroker::getStaticHelper('viewRenderer')->view;
        $view->headScript()->captureStart() ?>
            $().ready(function() {
                $('form').submit(function() {
                    $.get('/feedback', $(this).serialize(), function(response) {
                        if ('ok' == response.code) {
                            // transmission completed
                            $(':input', 'form')
                                .not(':button, :submit, :reset, :hidden')
                                .val('')
                            $('p.spasibo span').text('Спасибо! Обязательно отвечу!');
                        } else {
                            // print error message
                            $('p.spasibo span').text('Ошибка при заполнении');
                        }
                    }, 'json');
                    return false;
                });
            });
        headScript()->captureEnd();
        return $html;
    }

}

[/cc]



данный view Helper у меня располагается в папке **application/modules/feedback/view/helpers/Form.php**





###  Action_Helper: замена placeholder'ов




Более или менее универсальное решение по замене placeholder'ов вида **{placeholder_%moduleName%_%viewHelper(%param%)}** реализовал в виде Action Helper'а, вот его код



[cc lang="php"]
getResponse()->setBody('replaced');
        // label {placeholder_image_gallery(param)}
        //call_user_func_array($function, $param_arrarray);
        // \{placeholder_(\w+)_(\w+)\((((\w+)?,?)*)\)\}
        if ('admin' === substr($this->getRequest()->getControllerName(), 0, 5))
            return;
        $regExp = '@\{placeholder_(\w+)_(\w+)\((\d+)\)\}@';
        if (preg_match_all($regExp, $this->getResponse()->getBody(), $matches)) {
            //var_dump($matches);
            $matchesFound = count($matches[0]);
            $allMatches = array();
            for($i = 0; $i < $matchesFound; $i++)
                $allMatches[] = array(
                    'module' => $matches[1][$i],
                    'helper' => $matches[2][$i],
                    'param' => $matches[3][$i],
                    'assembledString' => "{placeholder_{$matches[1][$i]}_{$matches[2][$i]}({$matches[3][$i]})}"
                );
            foreach($allMatches as $currentMatch) {
                $className = ucfirst($currentMatch['module']) . '_View_Helper_' . ucfirst($currentMatch['helper']);

                if (class_exists($className)) {
                    $viewHelper = new $className;
                    if (method_exists($viewHelper, ucfirst($currentMatch['helper']))) {
                        $this->getResponse()->setBody(
                                str_replace(
                                        $currentMatch['assembledString'], // assembled regexp
                                        call_user_func(array($viewHelper, ucfirst($currentMatch['helper'])), $currentMatch['param']), // call it
                                        $this->getResponse()->getBody() // old body
                                )
                        );
                    }
                }
            }
        }
    }

}

[/cc]



###  Регистрируем наш ActionHelper




Осталось только подключить наш Action Helper, я делаю это в основном **application/Bootstrap.php**



[cc lang="php]
protected function _initActionHelpers() 
{
	Zend_Controller_Action_HelperBroker::addHelper(new Vredniy_Controller_Action_Helper_ContentReplacer());
}
[/cc]



Данное решение не претендует на абсолютную правильность, но меня оно пока устраиивает и всем спасибо за внимание, надеюсь, что данное решение вам когда-нибудь пригодится :)
