---
author: admin
comments: true
date: 2012-09-10 19:51:27+00:00
layout: post
slug: download-torrent-files-with-transmission-daemon

permalink: /2012/09/download-torrent-files-with-transmission-daemon

title: Скачиваем Torrent файлы с помощью ruby и transmission
wordpress_id: 932
categories:
- Ruby
- Ruby On Rails
tags:
- bash
- nginx
- rails
- ruby
- unicorn
- vps

keywords: "transmission-daemon, ruby torrent, rails torrent"
description: "Сегодня я расскажу вам как я решил несложную, но очень акутальную для меня задачу, а именно скачивание файлов с торрентов."
---

{% img image /images/posts/2012-09-download-torrent-files-with-transmission-daemon/transmission-daemon-ruby-300x172.png %}

Сегодня я расскажу вам как я решил несложную, но очень акутальную для меня задачу, а именно скачивание файлов с торрентов. Я приследовал в первую очередь цель скачивать файлы с помощью айпеда, в котором очень сложно с торрент клиентами. Задача свелась к тому, что с айпеда отправляю задание на скачку с торрентов, жду некоторое время и забирая скаченный файл. Сначала хотел реализовать вариант сложнее, а именно, чтобы скаченный файл отправлялся в дропбокс (ничего сложного тут нет, есть удобный и понятный гем), но понял что для меня задача решена и вместо того, чтобы делать какие-то "плюшки", займусь чем-нибудь другим.

<!-- more -->

### Инструментарий

Нам понадобится дешевенький vps сервер, я взял у reg.ru за 128 рублей в месяц с 128 мегабайтами оперативной памяти и 2 гигабайтами жестким и хорошим интернет каналом на XEN. Для данной задачи за глаза хватит такого слабенького сервера.


Демон _transmission-daemon_ для быстрого и удобного скачивания файлов через торрент. Самое сложное тут будет правильно настроить конфиг (но и это просто).

Rails бекенд (unicorn, nginx) для сообщения transmission, что пока заканчивать прохлаждаться и начинать скачивать файлы.

### Поэтапная реализация

В данной заметке я не буду рассказывать вам, как устанавливать rvm (в данном проекте я применял его), настраивать nginx + unicorn + capistrano, мануалов полно в инете, я лишь заострю внимание на ключевых моментах. Допустим приложение, чтобы не путаться в путях у вас также как у меня находится в папке _/home/deployer/apps/my_site_current_.

Давайте разберемся с конфигом transmission-daemon'а, который будет лежать в _~/.config/transmission-daemon/settings.json_.

``` javascript
{
    "dht-enabled": true,
    "download-dir": "/home/deployer/apps/my_site/current/public/attachment/complete-downloads",
    "download-queue-enabled": true,
    "download-queue-size": 5,
    "encryption": 1,
    "idle-seeding-limit": 30,
    "idle-seeding-limit-enabled": false,
    "incomplete-dir": "/home/deployer/apps/my_site/current/public/attachment/incomplete-downloads",
    "watch-dir": "/home/deployer/apps/my_site/current/public/attachment/incomplete-dir",
    "watch-dir-enabled": true,
    "incomplete-dir-enabled": true,
    "lpd-enabled": false,
    "message-level": 2,
    "peer-congestion-algorithm": "",
    "script-torrent-done-enabled": false,
    "script-torrent-done-filename": "",
    "start-added-torrents": true,
    "trash-original-torrent-files": false,
    "umask": 18,
    "upload-slots-per-torrent": 14,
    "utp-enabled": true
}

```

Большинство опций использунется по умолчанию, но есть некоторые которые стоит пояснить.

  * _"download-dir"_ - указывает в какую папку сохранять скаченные файлы
  * _"incomplete-dir"_ - куда будет качать
  * _"incomplete-dir-enabled"_ чтобы отделить котлеты от мух (скачиваемое и скаченное будут лежать отдельно, дабы не путаться)
  * _"watch-dir"_ - за какой папкой будем следить, т.е. если в этой папке появляются торрент файлы, то автоматически начинаем их скачивать
  * _"watch-dir-enabled"_ - включаем опцию "слежения"

Для эксперимента будем скачивать торрент файл с tfile.ru, чтобы без регистрации. Чтобы скачать файл, необходимо поставить куку, которая ставится javascript'ом, но тут нет ничего сложного.

Скачивать будем wget'ом, чтобы проще, а чтобы еще проще взаимодействовать с консольными утилитами будет использовать гем Cocaine от создателей Paperclip'а и FacoryGirl'ы.

``` ruby
class Attachment < ActiveRecord::Base
  validates_presence_of :name, :url

  attr_accessible :name, :status, :file, :url

  before_create :download_torrent_file

  private
  def download_torrent_file
    response = open(self.url).read

    cookie = response[/tmbx=[^"]*/]

    torrent_url = response[/http:\/\/[^"]*/]

    output_path = '/home/deployer/apps/my_site/current/public/attachment/incomplete-dir/'

    output_file = "#{ output_path }#{ Date.today.to_s }-#{ self.name }.torrent"

    # wget 'http://tfile.ru/forum/download.php?id=549501&attempt;=1#r' --no-cookies --header "Cookie: tmbx=6df9fc90ad02d5bdf2ca7449d0cc44
    line = Cocaine::CommandLine.new('wget', ":torrent_url --no-cookies --header 'Cookie: #{ cookie }' -O :output_file",
                                    :torrent_url => torrent_url,
                                    :output_file => output_file)

    line.run
  end
end

```

Так мы будем скачивать наши торрент файлы. Первый запуск transmission-daemon'а попробуем не в режиме демона, чтобы не ждать до ишачьей пасхи, а видеть все ошибки воочию. Для этого запустим transmission-daemon с ключом **-f** и все ошибки посыпятся в stdout и мы легко их сможем исправить, если все пойдет по плану, то наш первый файл будет скачен. Если все прошло тип-топ, то можем запускать transmission-daemon без параметров и озадачивать его новыми закачками.

Для того, чтобы отобразить результат и использовал следующее костальное решение в контроллере **Attachments#index**, я написал следующее:

``` ruby
def index
  @files = Dir.glob("public/attachment/complete-downloads/*")
end

```

во вьюхе уже выводил список файлов

``` ruby
<% @files.each do |f| %>

* <%= link_to f, f.sub('public', '') %>

<% end %>

```

Так как мне нужно было лишь скачивать pdf журналы, которые распространяются одним pdf файлом, то у меня не было необходимости выводить папки в папках, достаточно было отобразить файлы в одной папке.


### Заключение

Данное решение служит мне верой и правдой уже где-то с месяц и не вызывало никаких нареканий, процессы не отваливаются, журнальчики скачиваются :) Было бы неплохо, наверное, сделать так, что при скачивании торрент файла, мы получаем список файлов, которые в нем содержатся и каким-нибудь демоном проверяем все ли файлы скачались, если да, то оповещаем по имейлу пользователя. Но данная "плюшка" выходит за рамки данной заметки. Надеюсь, вам было интересно, до новых встреч :).
