---
author: admin
comments: true
date: 2010-04-20 18:43:07+00:00
layout: post
slug: zend-framework-auth-and-login
title: 'Zend Framework: аутентификация и регистрация'
wordpress_id: 458
categories:
- php
- Zend Framework
tags:
- Zend
- Zend Framework
- Zend_Auth
---

[![zend framework](http://vredniy.ru/wp-content/uploads/2010/04/Снимок-ilovezf-NetBeans-IDE-Dev-2010021520001-150x150.png)](http://vredniy.ru/wp-content/uploads/2010/04/Снимок-ilovezf-NetBeans-IDE-Dev-2010021520001.png)В сегодняшней заметке речь пойдет о аутентификации и регистрации новых пользователей с помощью базы данных на Zend Framework.<!-- more -->
Для начала нам понадобится создать таблицу, пусть называться она будет users, sql- запрос представлен ниже.
[cc lang="sql"]CREATE TABLE users (
id int(11) not null auto_increment,
firstname varchar(100) default null,
lastname varchar(100) default null,
email varchar(200) not null,
username varchar(100) not null,
password varchar(32) not null,
primary key(id)
);[/cc]
Дальше создадим модель в папке **application/models/** и назовем ее **Users.php**. Этот класс будет расширять **Zend_Db_Table**
[cc lang="php"][/cc]
Для эксперемента нам еще понадобится отдельный контроллер, который называть будет Auth, создать его можно в ZF Tools и просто ручками.
Если вы используете **NetBeans** в качестве среды разработки, то в бета версии 6.9 есть возможность взаимодействия с ZF Tools прямо из IDE.
[![zend framework регистрация](http://vredniy.ru/wp-content/uploads/2010/04/auth1zf-300x223.png)](http://vredniy.ru/wp-content/uploads/2010/04/auth1zf.png)
Затем создадим 4 метода у этой модели: для входа, регистрации, выхода и страницы, отображающей имя пользователя после успешного входа. Называться которые будет соответсвенно login, signin, logout, home.
Для тех, кто предпочитает создавать все ручками, нужно создать помимо методов в модели, еще и шаблоны для их отображения в папке **application/views/scripts/auth/**
[![zend framework аутентификация](http://vredniy.ru/wp-content/uploads/2010/04/auth2zf-300x223.png)](http://vredniy.ru/wp-content/uploads/2010/04/auth2zf.png)

Создадим форму **LoginForm.php**, которая будет создавать форму для входа на сайт.

[cc lang="php"]createElement('text', 'username', array(
                            'label' => 'Username: *',
                            'required' => TRUE
        ));

        $password = $this->createElement('password', 'password', array(
                            'label' => 'Password',
                            'required' => TRUE
        ));

        $signin = $this->createElement('submit', 'SignIn', array(
                            'label' => 'Sign In'
        ));

        $this->addElements(array(
                    $username,
                    $password,
                    $signin
        ));

    }
}
[/cc]
Далее добавим форму для регистрации новых пользователей **RegistrationForm.php**. Все формы, как вы, наверное, помините, должны располагаться в папке **APPLICATION_PATH/forms**

[cc lang="php"]createElement('text', 'firstname', array(
                            'label' => 'First Name:',
                            'required' => FALSE
        ));

        $lastname = $this->createElement('text', 'lastname', array(
                            'label' => 'Last Name',
                            'required' => FALSE
        ));

        $email = $this->createElement('text', 'email', array(
                            'label' => 'E-mail: *',
                            'required' => TRUE
        ));

        $username = $this->createElement('text', 'username', array(
                            'label' => 'Username: *',
                            'required' => TRUE
        ));

        $password = $this->createElement('password', 'password', array(
                            'label' => 'Password: *',
                            'required' => TRUE
        ));

        $confirmPassword = $this->createElement('password', 'confirmPassword', array(
                            'label' => 'Confirm Password: *',
                            'required' => TRUE
        ));

        $register = $this->createElement('submit', 'Register', array(
                            'label' => 'Sign Uo'
        ));

        $this->addElements(array(
                    $firstname,
                    $lastname,
                    $email,
                    $username,
                    $password,
                    $confirmPassword,
                    $register
        ));
    }
}
[/cc]
В этих формах не происходит ровным счетом ничего интересного, кроме создания нескольких элементов и задания им некотрых значений, к примеру, обязательности заполнения и метки (label).

Давайте создадим наш контроллер, методы которого будут отвечать за вход и регистрацию пользователей, а также их выход. Сначала приведу весь текст, дальше попытаюсь подробнее заострить внимание на важных моментах.

[cc lang="php"]_redirect('auth/login');
    }
    
    public function loginAction() {
        $users = new Users();
        $form = new LoginForm();
        $this->view->form = $form;
        if ($this->getRequest()->isPost()) {
            if ($form->isValid($_POST)) {
                $data = $form->getValues();
                $auth = Zend_Auth::getInstance();
                $authAdapter = new Zend_Auth_Adapter_DbTable($users->getAdapter(), 'users');
                $authAdapter->setIdentityColumn('username')
                        ->setCredentialColumn('password')
                        ->setIdentity($data['username'])
                        ->setCredential($data['password']);
                $result = $auth->authenticate($authAdapter);
                if ($result->isValid()) {
                    $storage = new Zend_Auth_Storage_Session();
                    $storage->write($authAdapter->getResultRowObject());
                    $this->_redirect('auth/home');
                } else {
                    $this->view->errorMessage = 'Invalid username or password. Please try again';
                }
            }
        }
    }
    
    public function signupAction() {
        $users = new Users();
        $form = new RegistrationForm();
        $this->view->form = $form;
        if ($this->getRequest()->isPost()) {
            if ($form->isValid($_POST)) {
                $data = $form->getValues();
                if ($data['password'] != $data['confirmPassword']) {
                    $this->view->errorMessage = 'Password and confirm password dont \' match';
                    return;
                }
                if ($users->isUnique($data['username'])) {
                    $this->view->errorMessage = 'Name already taken. Please choose another one.';
                    return;
                }
                unset($data['confirmPassword']);
                $users->insert($data);
                $this->_redirect('auth/login');
            }
        }
    }
    
    public function logoutAction() {
        $storage = new Zend_Auth_Storage_Session();
        $storage->clear();
        $this->_redirect('auth/login');
    }
    
    public function homeAction() {
        $storage = new Zend_Auth_Storage_Session();
        $data = $storage->read();
        if (!$data) {
            $this->_redirect('auth/login');
        }
        $this->view->username = $data->username;
    }
}
[/cc]
Метод **homeAction()**: cперва мы создаем экземпляр класса Zend_Auth_Storage_Session, который будет хранить информацию о пользователе, который пытается войти най сайт или уже вошедшего. Объект данного класса возвращает непустую запись, если пользователь вошел на сайт, null в противном случае. Если же null, перенаправляем пользователя на страницу входа на сайт для ввода имени пользователя и пароля. Дальше в файле **views/scripts/auth/home.html** напишем следующее:
[cc lang="php"]Welcome username;?>,   

This is your home page  

[click here to logout](<?php echo $this->url(array('controller' => 'auth', 'action' => 'logout'));?>)[/cc]
Чтобы пользователь, миновавший вход на сайт, увидел свое имя и ссылку, позволяющую выйти (разлогиниться не очень хорошее слово).

Метод **loginAction()**: в первых 2х строчках мы создаем экземпляры нашей модели Users и формы для входа на сайт. (из-за неправильного сначала выбранного способа именования классов приходится подключать их в инициализационном методе, это мой промах и в последующих заметках я постараюсь исправить это). Третьей строкой мы ассоциируем нашу форму с переменной вида, которую отобразим в шаблоне.
Дальше идет непосредстенно бизнес-логика. Сначала мы проверяем, чтобы отосланные данные были POST запросом, далее проверяем их правильность. Если оба этих условия соблюдены, мы получаем значения формы [cc lang="php"]$data = $form->getValues();[/cc]
Создаем экземпляр класса Zend_Auth и аутентификационный адаптер $authAdapter.
Я использую адаптер, который непосредственно связан с базой данных, предоставляемый Zend'ом. Дальще мы указываем какие поля являются именем пользователя, а какие паролем.
[cc lang="php"]                $authAdapter->setIdentityColumn('username')
                        ->setCredentialColumn('password')
                        ->setIdentity($data['username'])
                        ->setCredential($data['password']);[/cc]
И пытаемся призвести аутентификацию по присланным из формы данным, метод ->authenticate()
Если результат аутентификации валиден **->isValid()** (мы нашли хотя запись в базе данных), сохраняем результат в сессии и переадресуем пользователя на домашнюю страницу. Если аутентификация не прошла, выводим сообщения об ошибке.
Шаблон для этого метода. **application/views/scripts/login.phtml**
[cc lang="php"]errorMessage)) {
    echo $this->errorMessage;
}
?>
form; ?>hf
[
    New users click here?
](<?php echo $this->url(array('controller' => 'auth', 'action' => 'signup'));?>)
[/cc]
Метод **signinAction()** делаем примерно тоже самое, за исключением вывода другой формы, проверки паролей (одинаковы ли они?) и записи в базу данных данных нового пользователя, если не возникло ошибок. В этом методе упоминается **isUnique()** метод, который сейчас мы допишем в **Users.php**, возвращающий true, если заданное в параметре имя в базе данных существует и false в противном случае. Что предотвратит создание двух пользоватлей с одинаковыми именами. Файл **application/models/Users.php**
[cc lang="php"]_db->select()->from($this->_name)->where('username = ?', $username);
        $result = $this->getAdapter()->fetchOne($select);
        if ($result) {
            return TRUE;
        } else {
            return FALSE;

        }
    }
}[/cc]
Начало серьезному проекту положено, хоть и остаются еще неправильности в коде, но с опытом я, надеюсь, от них избавлюсь. Удачного вам программирования! =)

