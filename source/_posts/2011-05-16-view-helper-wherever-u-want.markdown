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

keywords: "actionhelper, viewhelper, zf, zf action helper, zf content replacer, zf cms,cms,php,zend framework"
description: "возникла необходимость вставлять в контент форму обратной связи и еще что-нибудь в произвольное место"
---




Здравствуйте, уважаемые коллеги-разработчики, в сегодняшней заметке я поделюсь с вами своим костылем-решением. Пишу в свободное время свою модульную CMS (ведь каждый разработчик должен написать в своей жизни хотя бы одну) на базе Zend Framework'а и возникла необходимость вставлять в контент форму обратной связи и еще что-нибудь в произвольное место. Для создания стратичных страниц я использую свой модуль Static, который ничего не умеет, кроме создания стратичных страниц WYSIWYG'ом и прикрепления к уже имеющимся.


<!-- more -->
[{% img image /images/posts/2011-05-view-helper-wherever-u-want/content-replacer-150x150.png %}](/images/posts/2011-05-view-helper-wherever-u-want/content-replacer.png)

Первое, что пришло в голову это заменять какой-нибудь placeholder аля {placeholder_feedback_form(0)} на содержимое. Сказано-сделано, приведу кусок view Helper'а, который отображает нестандартную форму и валидирут посредстом отправления AJAX запросов на сервер.






### View_Helper: на что будем менять



``` php
<?php

class Feedback_View_Helper_Form extends Zend_View_Helper_Abstract
{
    // form with ajax validation
    public function form($form = null)
    {
        $html = <<< karamba
        <form action="#" method="post">
                <div class="fancyform">
                        <label for="username"><span>Ваше имя</span></label>
                        <div class="it"><div><input type="text" name="username" id="username"></div></div>
                        <label for="email"><span>Ваш E-mail</span></label>
                        <div class="it"><div><input type="text" name="email" id="email"></div></div>
                        <label for="phone"><span>Ваш телефон</span></label>
                        <div class="it"><div><input type="text" name="phone" id="phone"></div></div>
                        <div class="ta"><div><textarea name="message"></textarea></div></div>
                        <div class="is"><div><input type="submit" name="submit" id="submit" value="Отправить"></div></div>
                        <p class="spasibo"><span></span></p>
                </div><!-- /fancyform -->
        </form>
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
        <?php $view->headScript()->captureEnd();
        return $html;
    }

}
```




данный view Helper у меня располагается в папке **application/modules/feedback/view/helpers/Form.php**





###  Action_Helper: замена placeholder'ов




Более или менее универсальное решение по замене placeholder'ов вида **{placeholder_%moduleName%_%viewHelper(%param%)}** реализовал в виде Action Helper'а, вот его код




``` php
<?php

/**
 * content replacer
 *
 * replace {placeholder_moduleName_helperName(param)} with
 * Modulename_View_Helper_HelperName::helperName($param) return
 */
class Vredniy_Controller_Action_Helper_ContentReplacer extends Zend_Controller_Action_Helper_Abstract
{

    public function postDispatch()
    {
        //$this->getResponse()->setBody('replaced');
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
```




###  Регистрируем наш ActionHelper




Осталось только подключить наш Action Helper, я делаю это в основном **application/Bootstrap.php**




``` php
protected function _initActionHelpers()
{
   Zend_Controller_Action_HelperBroker::addHelper(new Vredniy_Controller_Action_Helper_ContentReplacer());
}
```




Данное решение не претендует на абсолютную правильность, но меня оно пока устраиивает и всем спасибо за внимание, надеюсь, что данное решение вам когда-нибудь пригодится :)
