---
author: admin
comments: true
date: 2010-03-14 10:00:08+00:00
layout: post
slug: calculator-with-brackets-php

permalink: /2010/03/calculator-with-brackets-php

title: Калькулятор со скобками на php
wordpress_id: 417
categories:
- php
- программирование
tags:
- php
- алгоритмы
- калькулятор
---

[![calculator with brackets php](http://vredniy.ru/wp-content/uploads/2010/03/calculator-150x31.gif)](http://vredniy.ru/wp-content/uploads/2010/03/calculator.gif)


В сегодняшней заметке я поделюсь с вами своей наработкой.А именно калькулятор, который может помимо того, что складывать 2 и 2, умеет это делать, учитывая неограниченную вложенность скобок. 


Реализовано это все дело бы на PHP  с применением регулярных выражений. Конечно, его можно проапрейдить, внеся проверку правильности выражения, разработать фронтенд, но для меня это уже не очень важные задачи. Цель, которую ставил перед собой, достигнута. 

<!-- more -->


Алгоритм калькуляции состоит из 3х этапов:




  1. Находим подскобочное выражение, не содержащее других скобок и отдаем его в пункт 2


  2. Выражение вида `\d+[-+*\/]\d+([-+*\/]\d+)*` разделяем на элементарные: 1+2, 1/3, 2.4*12 и передаем в пункт 3


  3. Калькулируем полученное из пункта 2 выражение, заменяем его результатом, и передаем выше. 




Повторяем весь процесс до того, как не удалим все скобки + 1 раз.






### 3. Функция подсчета элементарных выражений


[cc lang="php"]
function elementary ($exp) {
    $pattern = '@(-?(:?\d+\.)?\d+)([-+*\/])(-?(:?-?\d+\.)?\d+)@';
    if (preg_match($pattern, $exp, $matches)) {
        $arg1 = (real) $matches[1];
        $action = $matches[3];
        $arg2 = (real) $matches[4];
        switch ($action) {
            case '+':
                return $arg1 + $arg2;

            case '-':
                return $arg1 - $arg2;

            case '*':
                return $arg1 * $arg2;

            case '/':
                return $arg1 / $arg2;
        }
    } else {
        die('Что-то пошло не так');
    }
}
[/cc]



### 2.Функция разбиения бесскобочного выражения на элементарные выражения


[cc lang="php"]
function calculateWithoutBrackets($exp) {
    $pattern = '@-?(:?\d+\.)?\d+[*\/]-?(:?\d+\.)?\d+@';
    while (preg_match($pattern, $exp, $matches)) {
        $tmpExp = $matches[0];
        $replaceTo = elementary($tmpExp);
        $exp = str_replace($tmpExp, $replaceTo, $exp);
    }

    $pattern = '@-?(:?\d+\.)?\d+[-+]-?(:?\d+\.)?\d+@';
    while (preg_match($pattern, $exp, $matches)) {
        $tmpExp = $matches[0];
        $replaceTo = elementary($tmpExp);
        $exp = str_replace($tmpExp, $replaceTo, $exp);
    }
    return $exp;
}
[/cc]


Не нашел как сделать подсчет в один проход, поэтому использую 2: один для произведения/частного, другой для суммы/разности, чтобы сохранить приоритеты операций




### 1. Основная часть скрипта




В ней мы задаем выражение для подсчета, удаляем из него пробелы, подсчитываем результаты в скобках и возвращаем в строку для дальнейших манипуляций.


[cc lang="php"]
$exp = '(((1 / 2) / 3) * (1 + (2 / 4) - (12 / 5) + ((1 / 2) + (4 * 2 * (1 - 2))))) + 12 + (((1 / 2) / 3) * (1 + (2 / 4) - (12 / 5) + ((1 / 2) + (4 * 2 * (1 - 2))))) + 12';
$exp = preg_replace('@\s+@', '', $exp);

$pattern = '@\([^\(\)]+\)@';
while (preg_match($pattern, $exp, $matches)) {
    preg_match_all($pattern, $exp, $matches);
    if (!empty($matches[0][1])) {
        $expTmp = $matches[0][1];
    } else {
        $expTmp = $matches[0][0];
    }
    $inBrackets = substr($expTmp, 1, strlen($expTmp) - 2);
    $inBrackets = calculateWithoutBrackets($inBrackets);
    $exp = str_replace($expTmp, $inBrackets, $exp);
}

echo 'Результат: ' . calculateWithoutBrackets($exp);
[/cc]



Вроде работает, проверял результаты вычислений [Google'ом](http://tinyurl.com/y8rpedr)




Будут интересны любые возможные пожелания и дополнения, спасибо за внимание, может кому-нибудь пригодятся мои наработки, потому что исходников калькулятора со скобками я не находил.




Скачать исходник вы можете [здесь](/files/calculator.zip)
