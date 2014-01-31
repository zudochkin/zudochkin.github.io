---
author: admin
comments: true
date: 2010-02-24 19:30:59+00:00
layout: post
slug: statistics-of-downloads

permalink: /2010/02/statistics-of-downloads

title: Статистика по количеству скачиваний
wordpress_id: 314
categories:
- php
- программирование
tags:
- mysql
- php
---

[![количество скачиваний php](http://vredniy.ru/wp-content/uploads/2010/02/stats-150x120.png)](http://vredniy.ru/wp-content/uploads/2010/02/stats.png)
Сегодня будет короткая заметка, посвященная статистике, ведь каждому, наверное, интересно, сколько раз скачали с его сайта тот или иной файл. Я не хотел делать все это дело на файлах, чтобы не захламлять сервер, пусть лучше эта информация хранится в базе данных. 
Создаем файл конфигурации, чтобы в каждом файле пароли не хранить.
**config.php**
[cc lang="php"]
< ?php
$CONFIG['mysql']['host']='localhost';
$CONFIG['mysql']['login']='root';
$CONFIG['mysql']['password']='';
$CONFIG['mysql']['db']='test';
?>
[/cc]

До того, как переходить к главному файлу нашей системы, нужно создать в базе данных таблицу с двумя полями и вставить туда пару записей о файлах, с которыми мы хотим работать.
[cc lang="sql"]
 CREATE TABLE `downloads` (
`filename` TEXT NOT NULL ,
`counter` INT NOT NULL
) ENGINE = InnoDB;



Теперь пару записей

    
    
    INSERT INTO `test`.`downloads` (
    `filename` ,
    `counter`
    )
    VALUES (
    'catalog.pdf', '0'
    ), (
    'catalog.rar', '0'
    );
    [/cc]
    А теперь главный файл, к которому мы и будем обращаться при расстановки ссылок для тех файлов, для которых мы хотим вести статистику скачиваний.
    <strong>download.php</strong>
    [cc lang="php"]
    < ?php
    require_once 'config.php';
    
    //соединение с БД или вывод ошибки и выход
    $conn = mysql_connect($CONFIG['mysql']['host'], $CONFIG['mysql']['login'], $CONFIG['mysql']['password']) or die(mysql_error());
    @mysql_select_db($CONFIG['mysql']['db'], $conn);
    
    //проверяем какой файл пытаются скачать
    switch ($_GET['file']) {
        case 'catalogpdf':
    		$sql = "UPDATE downloads SET `counter` = `counter` + 1 WHERE `filename`='catalog.pdf'";
    		mysql_query($sql);
    		mysql_close($conn);
            header('Location: /files/catalog.pdf');
        case 'mikasa2010rar':
            $sql = "UPDATE downloads SET `counter` = `counter` + 1 WHERE `filename`='catalog.rar'";
            header('Location: /files/catalog.rar');
    		mysql_query($sql);
    		mysql_close($conn);
        default:
    		mysql_close($conn);
    		break;
    }
    ?>
    [/cc]
    Здесь ничего сложного, проверяем какой файл пытаются скачать и увеличиваем количество скачиваний в базе данных на единицу.
    Файл вывода статистики <strong>counter.php</strong>
    [cc lang="php"]
    < ?php
    require_once 'config.php';
    
    $conn = mysql_connect($CONFIG['mysql']['host'], $CONFIG['mysql']['login'], $CONFIG['mysql']['password']) or die('mysql error');
    @mysql_select_db($CONFIG['mysql']['db'], $conn);
    $sql = 'SELECT downloads.filename, downloads.counter FROM downloads';
    $res = mysql_query($sql, $conn);
    
    //составляем красивый вывод в таблице
    echo "<table cellspacing="10">";
    while ($row = mysql_fetch_array($res)) {
    	echo "<tr><td>";
    	echo $row['filename'] . '</td><td><b style="font-size: 16px; color: blue">'
    		. $row['counter'] . '</b></td></tr>';
    }
    echo "</table>";
    ?>
    [/cc]
    И теперь в том месте, где надо нам вывести статистику вставляем код
    [cc lang="php"]
    < ?php
    	$stats = file_get_contents('http://localhost/downloader/counter.php');
    	echo $stats;
    ?>
    [/cc]
    Ничего сложного как и все гениальное =)
    Пример можно улучшить, если нужны более точные данные, например не хотим учитывать скачивания дважды с одного компьютера. Для этого нужно воспользоваться кукисами в файле <strong>download.php</strong>. А в тексте ссылки вместо прямого обращения к файлу, нужно указать скрипт: [cc lang="html"]<a href="http://localhost/downloads/download.php?file=catalogpdf">
    Скачать каталог в PDF формате
    </a>[/cc]
    Вот и все, будьте в курсе количества скачиваний ваших файлов =)
