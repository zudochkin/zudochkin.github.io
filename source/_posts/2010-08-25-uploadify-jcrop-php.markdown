---
author: admin
comments: true
date: 2010-08-25 14:08:58+00:00
layout: post
slug: uploadify-jcrop-php
title: Создание миниатюр на стороне клиента без перезагрузки страницы или немного
  уличной магии
wordpress_id: 597
categories:
- javascript
- jQuery
- php
- программирование
tags:
- jcrop
- uploadify
- миниатюры
---

[![uploadify-jcrop](http://vredniy.ru/wp-content/uploads/2010/08/uploadify-jcrop1-150x150.jpg)](http://vredniy.ru/wp-content/uploads/2010/08/uploadify-jcrop1.jpg)Сейчас я расскажу вам, как связывал uploadify - плагин, для jQuery для асинхронной загрузки файлов на сервер, прочитать о котором вы можете у меня в заметке ["Добавление файлов на сервер без перезагрузки страницы"](/2010/05/jquery-plugin-upload-without-reload/), и плагин jCrop, для ресайзинга изображений на стороне клиента. Я посчитал, что заполнять форму, загружать изображения и делать миниатюры для загруженных файлов легче для пользователей на одной странице. Есть в этой связке нереализованные места, но обо всем поподробнее. Список нужных файлов в конце заметки.<!-- more -->

Будем считать, что вы расположили все файлы как  в заметке, а теперь мы ее немного расширим. Вот весь javascript на стринице, придется заменить тот, который остался после последней заметки. 

[cc lang="javascript"]
/**
     * обновляем координаты прямоугольника, миниатюры создающего
     */
function updateCoords(c)
{
    $('#x').val(c.x);
    $('#y').val(c.y);
    $('#w').val(c.w);
    $('#h').val(c.h);
};

/**
     * чтобы не отправлять нулевые координаты
     */
function checkCoords()
{
    if (parseInt($('#w').val())) return true;
    alert('Please select a crop region then press submit.');
    return false;
};

$(function() {

    $('form').submit(function(){
        $('#files').val($('#response').html());
       
    });

    // загрузить файлы
    $('#upload').click(function(){
        $('#uploadify').uploadifyUpload();
        return false;
    });

    // асинхронная загрузка
    $('#uploadify').uploadify({
        'uploader': 'uploadify.swf',
        'script': 'uploadify.php',
        'folder': 'upload',
        'cancelImg': 'cancel.png',
        'checkscript'  : 'check.php',
        'fileDesc'   : 'jpg;png;bmp;gif',
        /*'queueID'    : 'fileQueue',*/ // id где будет появляться список
        'multi'     : false,
        'fileExt'   : '*.jpg;*.png;*.gif,;*.jpeg',
        'onComplete'   : function(event,queueID,fileObj,response,data) {
            $('#response').slideDown('slow');
            //$('#response').append(response);

            $('#response').append('

**' + response+ '**

');

            /*-----------------------------уличная магия будет здесь ;)-----------------------------*/
            $.ajax({
                'url' : 'ajaxhandler.php',
                'type': 'post',
                'data': {
                    'path': response,
                    'action': 'parsepath'
                },
                'success': function(data){
                    //alert(data);
                    $('#cropcontainer').html('');
                    // $('#path').val('ff');
                    $('#pic').attr('src', 'iframe.php' + '?filename=upload/'  + data);
                }
            })
        /*-------------------------------------конец уличной магии =)------------------------------*/


        }
    });
});[/cc]



Алгоритм прост до безумия: после асинхронно загрузки файла на сервер (мы будем загружать по одному файлу), обрабатывается событие onComplete uploadify, которое загружает специально подготовленный iframe, проделывающий всю грязную работу. 

Вот собственно он сам.



[cc lang="html"]

    
    
        ![](<?php echo $filename; ?>)

        
            
            
            
            
            
            
        
    
[/cc]


И файл** cut.php**, ему помогающий. Здесь следует отметить то, что поддерживаются только файлы jpeg, но не реализовано никакой защиты, дабы не загромождать пример.



[cc lang="php"][/cc]


Еще нам понадобится небольшой обработчик AJAX запросов, потому что я сразу не придумал как реализовать это в javascript, поэтому сделал это на PHP, **если кто подскажет как буду премного _благодарен_ =)**

**ajaxhandler.php**
[cc lang="php"]
', '', $_POST['path']));
        break;

    default:
        break;
}
?>[/cc]


_Плагины для jQuery:_



	
  * [jCrop](/files/jquery.Jcrop-0.9.8.zip)


	
  * [uploadify](/files/jquery.uploadify-v2.1.0.zip)



Что стоит подправить?

	
  1. Сделать проверку на загружаемые файлы


	
  2. Реализовать очистку мусора, коего скапливается не очень мало



Удачного вам программирования.






