---
author: admin
comments: true
date: 2013-11-25 14:43:24+00:00
layout: post
slug: open-uri-progress-bar

permalink: /2013/11/open-uri-progress-bar



title: "Ruby OpenURI::open и ProgressBar"

categories:
- ruby
tags:
- ruby

keywords: ruby

description: Сегодня в короткой заметке я расскажу как сделать простой progressbar, отображающий процесс скачивания, к примеру, большого файла.
---

Рыская по документации к методу open из набора OpenURI (мне нужно было установить большее значение timeout'а). Пролистав нужное место в документации натолкнулся на интересные параметры, с которыми можно вызывать метод open.<!--more-->

* `:content_length_proc => lambda {|content_length| ... }` - если данный proc установлен, то в него передается Content-Length или nil, если данный параметр недоступен. В этот момент мы уже знаем полный размер файлы и можем нарисовать красивый ProgressBar.
* `:progress_proc => lambda {|size| ...}` - данный proc вызывается с одним параметром (размер скаченного фрагмента в байтах), когда метод **open** получает очередной фрагмент из сети.
* `:read_timeout=>10` - это тот параметр, из-за которого я и полез в документацию, устанавливает таймаут на чтение для http соединения.

А теперь небольшой пример использования данных знаний. Нам понадобится большой файл, я взял трехмегабайтный файл и положил его в Dropbox/Public, чтобы легко было получить на него ссылку. Также понадобится установленный gem [ruby-progressbar](https://github.com/jfelchner/ruby-progressbar).

Вот и все, работающий пример готов.

``` ruby
require 'ruby-progressbar'
require 'open-uri'

progress_bar = nil
open('https://dl.dropboxusercontent.com/u/11041525/DIX1.0Universal.dmg',
  content_length_proc: proc { |total|
    if total.to_i > 0
      progress_bar = ProgressBar.create(title: 'Downloaded', total: total)
    end
},
progress_proc: proc { |step|
  progress_bar.progress = step
}) { |file| puts "File #{file} successfully downloaded" }
```

<iframe width="560" height="315" src="//www.youtube.com/embed/SIWBIl1oRrc" frameborder="0" allowfullscreen></iframe>
