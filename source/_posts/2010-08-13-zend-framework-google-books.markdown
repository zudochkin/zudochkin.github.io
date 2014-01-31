---
author: admin
comments: true
date: 2010-08-13 12:32:09+00:00
layout: post
slug: zend-framework-google-books
title: Zend Framework для книголюбов
wordpress_id: 574
categories:
- Zend Framework
tags:
- gdata
- google
- Zend Framework
---

[caption id="attachment_577" align="alignleft" width="150" caption="googlebooks"][![](http://vredniy.ru/wp-content/uploads/2010/08/googlebooks-150x150.jpg)](http://vredniy.ru/wp-content/uploads/2010/08/googlebooks.jpg)[/caption]Сегодня мы поговорим с вами о Google Books, о замечательном сервисе компании Google и о том легко искать там книги с помощью замечательного php фреймворка Zend Framework. (Пример можно посмотреть на картинки слева).
Для этого приведу код с подробными комментариями. С этим сервисом поиск книг становится очень простым. <!-- more -->
[cc lang="php"]try {
                    $email = 'inzendwetrust@gmail.com'; // укажите свой аккаунт на гугле, чтобы посмотреть оценку для книги
                    $password = '***';
                    $client = Zend_Gdata_ClientLogin::getHttpClient(
                                    $email, $password, 'print');
                    $books = new Zend_Gdata_Books($client);
                    $query = $books->newVolumeQuery();
                    $query->setQuery($data['query']); // данные из формы
                    $query->setMaxResults(20); // максимальное количество выводимых результатов

                    $feed = $books->getVolumeFeed($query);
                } catch (Exception $e) {
                    die('An error: ' . $e->getMessage());
                }

                if (isset($feed)) {
                    echo "

Количество найденных книг: {$feed->totalResults}

";

                    foreach ($feed as $entry) {
                        $link = $entry->getLink();
                        $imageLink = $entry->getThumbnailLink()->href;
                        $x = $entry->getFormats();
                        $pages = @$x[0];
                        $type = @$x[1];

                        echo "


              [{$entry->getTitle()}]({$link[1]->getHref()})  
";
                        echo "![]({$link[0]->getHref()})";
                        echo '  
' . 'страниц: ' . $pages;
                        echo '  
';
                        echo 'тип:  ' . $type . '  
';

                        echo 'Дата издания: ', implode(',', $entry->getDates());
                        echo '  
';
                        $x = $entry->getIdentifiers();
                        echo 'ISBN10: ', str_replace('ISBN:', '', @$x[1]->text);
                        echo '  
';
                        echo 'ISBN13: ', str_replace('ISBN:', '', @$x[2]->text);
                        echo '  
';
                        if (method_exists($entry->getRating(), 'getAverage'))
                            echo 'Средний рейтинг: ', $entry->getRating()->getAverage();
                        echo '

';
                    }
                }[/cc]
