---
author: admin
comments: true
date: 2013-03-03 12:13:05+00:00
layout: post
slug: unicorn-rbenv-nginx-postgresql

permalink: /2013/03/unicorn-rbenv-nginx-postgresql

title: Разворачиваем rails приложение вместе с capistrano
wordpress_id: 974
categories:
- Ruby
- Ruby On Rails
tags:
- capistrano
- nginx
- postgresql
- rbenv
- ruby
- ruby-build
- unicorn
---

Здравствуйте, дорогие друзья, сегодня я расскажу вам как быстро развернуть Ruby on Rails приложение на сервере. Если у вас маленькое приложение, которое посещает 10 человек в день, то вам достаточно будет задеплоить его на heroku (имею в виду бесплатный тариф) и не читать все то, что написано ниже. Если же у вас приложение побольше, то милости прошу на огонек.<!-- more -->




### Сервер


Сервер: я буду настраивать все на Ubuntu 12.04 (64x), но не думаю, что должны возникнуть сложности с подобной операционной системой. Если же появятся траблы, пишите в комментах, попытаемся решить вместе.

Итак, поехали, логинимся по root'ом и производим базовую настройку сервера. Я начинаю всегда с локалей, для этого делаю следующее.

**localedef ru_RU.UTF-8 -i ru_RU -fUTF-8** и в файл **/etc/default/locale** добавляю

```
LANG="ru_RU.UTF-8"
LC_CTYPE="ru_RU.UTF-8"
LC_NUMERIC=C
LC_TIME=C
LC_COLLATE=C
LC_MESSAGES=C
```

Затем создаем пользователя, от лица которого мы и будем все делать и наделим его привилегиями запускать команды через sudo.

`useradd -m -g staff -s /bin/bash deployer && passwd deployer` - создаем пользователя deployer в группе staff (флаг -g) и с домашней папкой /home/deployer (флаг -m), флаг -s назначает пользоватлю шел по умолчанию. И задаем ему пароль. В дальнейшем весь экшн будет производиться от имени этого пользователя.

Чтобы наш пользователь мог выполнять команды от рута, необходимо добавить его в группу в файл **/etc/sudoers**

```
(%staff ALL=(ALL) ALL)
```

Теперь выходим из под рута и логинимся под деплоером. Для того, чтобы не писать пароль каждый раз при входе, необходимо добавить на сервер инфо, что мы свои (авторизация по ключу).

`cat ~/.ssh/id_rsa.pub | ssh deployer@server "mkdir ~/.ssh; cat >> ~/.ssh/authorized_keys"`

Если у кого на рабочей машине ubuntu, то они могут запустить эту операцию намного проще (ssh-copy-id deployer@server).

Проверяем, ssh deployer@server более не должно спрашивать пароль, а сразу же пустить нас на сервер.

Для удобства можно добавить в файл **~/.ssh/config**, чтобы удобно было заходить, печатая только **ssh my_serv**

```
Host my_serv
  HostName server.com$
  User deployer$

```

`sudo apt-get update && sudo apt-get upgrade` - обновляем все источники приложений и накатываем обновления


### Установка и базовая настройка Postgres'а


`sudo apt-get install postgresql libpq-dev` (второй пакет нужен для того, чтобы гем pg поставился)

Данная установка подразумевает, что вы не собираетесь соединяться с базой данных с других машин. По умолчанию в Ubuntu, Postgresql сконфигурирован так, чтобы использовать логин текущего пользователя, т.е. если вы вошли под пользователем deployer и в Postgresql есть пользователь deployer, то спрашивать пароль у вас никто не будет.


`sudo -u postgres createuser --superuser $USER` (создаем пользователя deployer, который будет являться суперпользователем, этого делать не желательно, если у вас более, чем один проект, лучше под каждую бд создать отдельных пользователей с разными паролями)

`sudo -u postgres psql` - запускаем postgres консоль

postgres=# \password deployer - задаем пароль для нашего пользователя (имя пользователя заменить на свое) - этот пароль будет использоваться в конфигах вашего приложения (config/database.yml)

`createdb $USER` - создаем базу данных deployer


### Ставим rbenv, ruby и gem bundler


Ставим все необходимое, чтобы склонировать репозиторий rbenv, установить последние ruby и пользоваться загрузкой картинок в наших приложениям (последние два пакета).

`sudo apt-get install build-essential bison openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libxml2-dev libxslt1-dev autoconf libc6-dev libncurses5-dev libmagickcore-dev imagemagick`

Теперь давайте перейдем к установки ruby с помощью rbenv и ruby-build. Здесь нет ничего не обычного, в основном все взято со страниц README rbenv и ruby-build.



##### rbenv

`git clone git://github.com/sstephenson/rbenv.git ~/.rbenv`

`echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile` - для доступа к команде rbenv

`echo 'eval "$(rbenv init -)"' >> ~/.bash_profile` - для доступа к бинарникам установленных гемов и автокомплиту rbenv команд

`exec $SHELL` - перезапускам шел




##### ruby-build - для простой загрузки и сборки ruby из исходников


`git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build`

устанвливаем ruby `rbenv install 1.9.3-p392` (для этого необходимо немало времени, поэтому наберитесь терпения, на машине с 512 оперативы эта операция занимает около 7 минут)

Пока приложение настраивается можно добавить пару строк в ~/.gemrc файл, для того, чтобы не устанавливать документацию вместе с гемами

install: --no-ri --no-rdoc
update: --no-ri --no-rdoc

после этого `rbenv global 1.9.3-p392` и `gem install bundler`



### Локальное приложение для деплоя


Теперь давайте подготовим наше локальное приложение (я для
этих целей создал простое приложение rails

`rails new simple_deployment -T`


`rails g scaffold item name description:text` - для того, чтобы проверить и соединение с базой данных.

создадим файл .rbenv-version (`echo '1.9.3-p392' > .rbenv-version`) для того, чтобы unicorn запускал необходимую версию ruby

также добавим в Gemfile gem unicorn и gem capistrano (последний в группу :development)

``` ruby
gem 'unicorn'

group :development do
	gem 'capistrano'
end

```

и запустим в консоли `capify .` (capistrano создаст для нас файлик config/deploy.rb, который и будет "пультом управления" нашего приложения на сервере)

**config/deploy.rb**

``` ruby

require 'bundler/capistrano'
load 'deploy/assets'

set :repository, 'YOUR_GIT_OR_SVN_REPOSITORY_URL'
set :scm, :git

server 'YOUR_SERVER_IP_OR_HOSTNAME', :app, :web, :db, :primary => true

set :ssh_options, { :forward_agent => true }
default_run_options[:shell] = 'bash -l'

set :user, 'deployer'
set :group, 'staff'
set :use_sudo, false
set :rails_env, 'production'

set :project_name, 'simple_deployment'

set :deploy_to, "/home/deployer/projects/#{ project_name }"

desc "Restart of Unicorn"
task :restart, :except => { :no_release => true } do
  run "kill -s USR2 `cat /home/deployer/projects/#{ project_name }/shared/pids/unicorn.pid`"
end

desc "Start unicorn"
task :start, :except => { :no_release => true } do
  run "cd #{current_path} ; bundle exec unicorn_rails -c config/unicorn.rb -D -E #{ rails_env }"
end

desc "Stop unicorn"
task :stop, :except => { :no_release => true } do
  run "kill -s QUIT `cat /home/deployer/projects/#{ project_name }/shared/pids/unicorn.pid`"
end

after 'deploy:finalize_update', 'deploy:symlink_db'

namespace :deploy do
  desc "Symlinks the database.yml"
  task :symlink_db, :roles => :app do
    run "ln -nfs #{deploy_to}/shared/config/database.yml #{release_path}/config/database.yml"
  end
end

```

Проекты у нас на сервере будут жить в домашнией папке + projects, т.е. для нашего пользователя deployer это будет /home/deployer/projects.

В каждой папке приложения находятся еще три папки (releases - где хранятся все релизы нашего приложения, current - симлинк на текущий релиз из папки releases и shared, где хранится общая шняга для все релизов: пиды, идентификаторы сессий, логи и прочее.)


Также необходимо добавить ssh ключ сервера на github или bitbucket

содержимое ключа `cat ~/.ssh/id_rsa.pub`, если такого файла нет, то нужно его сгенерировать `ssh-keygen -t rsa`

и сделать с сервера `ssh git@bitbucket.org`, чтобы подтвердить соединение.




##### config/unicorn.rb - минимальный конфиг для нашего HTTP сервера.


``` ruby
worker_processes 2
user 'deployer', 'staff'
preload_app true
timeout 30

project_name = 'simple_deployment'

working_directory "/home/deployer/projects/#{ project_name }/current"

listen "/tmp/#{ project_name }.socket", :backlog => 64
pid "/home/deployer/projects/#{ project_name }/shared/pids/unicorn.pid"

stderr_path "/home/deployer/projects/#{ project_name }/shared/log/unicorn.stderr.log"
stdout_path "/home/deployer/projects/#{ project_name }/shared/log/unicorn.stdout.log"
```

Если вы указали все верно в config/deploy.rb, то можно запустить **cap deploy:setup** для создания capistrano необходимых для разворачивания приложения папок на стороне сервера.




### Nginx


Устанвливаем nginx командой `sudo apt-get install nginx` и переходим к его настройке. Для начала отредактируем файл **/etc/nginx/nginx.conf**

```
user deployer staff;
worker_processes 2;

pid /var/run/nginx.pid;
events {
  worker_connections 1024;
  multi_accept on;
}

http {
  sendfile on;
  tcp_nopush on;
  tcp_nodelay off;

  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;

  gzip on;
  gzip_disable "msie6";
  gzip_proxied any;
  gzip_min_length 500;
  gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

  include /etc/nginx/conf.d/*.conf;
  include /etc/nginx/sites-enabled/*;
}

```

Последняя строчка нужна, чтобы nginx загружал свои конфиги и из папка **/etc/nginx/sites-enabled/** (это нужно для того, чтобы не хранить все конфиги для проектов в одном файле).

А теперь nginx конфиг для нашего приложения.Будем считать, что наше приложение называется simple_deployment, а домен у него simple-deployment.com (чтобы вам удобнее было заменять на свои данные).

```
upstream simple_deployment {
  server unix:/tmp/simple_deployment.socket fail_timeout=0;
}

server {
  server_name simple-deployment.com;
  root /home/deployer/projects/simple_deployment/current/public;
  access_log /var/log/nginx/simple_deployment_access.log;
  rewrite_log on;

  location / {
    proxy_pass http://simple_deployment;
    proxy_redirect off;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    client_max_body_size 10m;
    client_body_buffer_size 128k;

    proxy_connect_timeout 90;
    proxy_send_timeout 90;
    proxy_read_timeout 90;
    proxy_buffer_size 4k;
    proxy_buffers 4 32k;
    proxy_busy_buffers_size 64k;
    proxy_temp_file_write_size 64k;
  }

  location ~ ^/(assets|images|javascripts|stylesheets|system)/ {
    root /home/deployer/projects/simple_deployment/current/public;
    expires max;
    break;
  }
}
```

и удалить файлы **/etc/nginx/sites-available/default** и **/etc/nginx/sites-enabled/default**


Теперь можно смело пробовать запускать `cap deploy:migrations`  в консоли локального приложения. Если все было выполнено по данной заметке, то вы должны получить развернутое приложения на сервере в папке **/home/deployer/projects/simple_deployment**, если вы конечно строго следовали инструкциям.

Запустим последную команду на сервере, запустим nginx (`sudo service nginx start`).

Теперь, после деплоя можно запускать команду cap start или cap stop для запуска сервера или его остановки. Надеюсь, следуя этому мануалу, у вас получилось развернуть свое приложение на сервере. Если нет, то милости прошу в комментарии, чем смогу, постараюсь помочь.

Ссылки на дополнительные материлы:

  * [деплой вместе с bundler](http://gembundler.com/deploying.html)
  * [конфигурация unicorn'а](http://unicorn.bogomips.org/Unicorn/Configurator.html)


Enjoy, my little colleagues :)

UPDATE: 06.03.2013
Немного защитим наш сервер, в первую очередь давайте запретим логин рута по ssh, у нас на это есть пользователь deployer, который может выполлнять команды от рута с помощью sudo. Для этого в файле **/etc/ssh/sshd_config** необходимо изменить
**PermitRootLogin** на **no** и **PasswordAuthentication** на **no**, последняя настройка запретит авторизацию для SSH по паролю (только по ключу).
