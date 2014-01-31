---
author: admin
comments: true
date: 2010-06-20 15:32:33+00:00
layout: post
slug: multilangua-zend-framework
title: Мультиязычность в формах и в сообщениях валидаторов Zend Framework
wordpress_id: 528
categories:
- Zend Framework
- программирование
tags:
- Zend Framework
- Zend_Translate
- мультиязычность
---

[![zf-translate-en](http://vredniy.ru/wp-content/uploads/2010/06/zf-translate-ru-150x150.png)](http://vredniy.ru/wp-content/uploads/2010/06/zf-translate-ru.png)Приветствую вас, интернете-разработчики. 
Стоит сейчас передо мной задача организации мультиязычного портала, который пока будет поддерживать два языка: русский и английский. 
Во-первых, я написал правило раутинга, которое исходя параметра в адресной строке будет переводить элементы управления на тот или иной язык. [![zf-translate-ru](http://vredniy.ru/wp-content/uploads/2010/06/zf-translate-en-150x139.png)](http://vredniy.ru/wp-content/uploads/2010/06/zf-translate-en.png)Выглядит это примерно так example.com/ru/blog, где ru - это параметр, задающий язык. Был еще, конечно, вариант, хранить значение языка в сессии и при этом не менять маршруты, но тогда половина контента была бы не доступна поисковым машинам, что очень не хорошо.<!-- more -->
Вот так выглядит мое правило
[cc lang="php"]
  protected function _initRouter() {
        $router = Zend_Controller_Front::getInstance()->getRouter();
        $translatorRoute = new Zend_Controller_Router_Route(
                        ':lang/:controller/:action/*',
                        array(
                            'lang' => 'ru',
                            'controller' => 'index',
                            'action' => 'index'
                        )
        );

        $router->addRoute('translator', $translatorRoute);
        //$frontController = Zend_Controller_Front::getInstance();
        //$frontController->setRouter($router);
        return $router;
    }
[/cc]
Теперь настроим переводчик в том же bootstrap'е
[cc lang="php"]
    protected function _initTranslator() {

     require_once APPLICATION_PATH . '/../lang/lang_en.php';
     require_once APPLICATION_PATH . '/../lang/lang_ru.php';
        $translate = new Zend_Translate(
                        'array',
                        $translateArrayEn,
                        'en');
        $translate->getAdapter()->addTranslation(
                $translateArrayRu,
                'ru');
        Zend_Registry::set('Zend_Translate', $translate);
        Zend_Form::setDefaultTranslator($translate);
    }
[/cc]
Последней строкой мы сообщаем фреймворку, что текстовые метки не стоит понимать буквально, сначала нужно посмотреть возможно для данной строки есть соответсвие в массиве соответствий, который выглядит примерно так.
[cc lang="php"]
'Введенное значение "%value%" неправильное. Разрешены только латинские символы и цифры',
 Zend_Validate_Alnum::STRING_EMPTY => 'Поле не может быть пустым. Заполните его, пожалуйста',
 Zend_Validate_Alpha::NOT_ALPHA => 'Введите в это поле только латинские символы',
 Zend_Validate_Alpha::STRING_EMPTY => 'Поле не может быть пустым. Заполните его, пожалуйста',
 Zend_Validate_Between::NOT_BETWEEN => '"%value%" не находится между "%min%" и "%max%", включительно',
 Zend_Validate_Between::NOT_BETWEEN_STRICT => '"%value%" не находится строго между "%min%" и "%max%"',
 Zend_Validate_Ccnum::LENGTH => '"%value%" должно быть численным значением от 13 до 19 цифр длинной',
 Zend_Validate_Ccnum::CHECKSUM => 'Подсчёт контрольной суммы неудался. Значение "%value%" неверно',
 Zend_Validate_Date::INVALID => '"%value%" - неверная дата',
 Zend_Validate_Date::FALSEFORMAT => '"%value%" - не подходит по формату',
 Zend_Validate_Digits::NOT_DIGITS => 'Значение "%value%" неправильное. Введите только цифры',
 Zend_Validate_Digits::STRING_EMPTY => 'Поле не может быть пустым. Заполните его, пожалуйста',
 Zend_Validate_EmailAddress::INVALID => '"%value%" неправильный адрес електронной почты. Введите его в формате имя@домен',
 Zend_Validate_EmailAddress::INVALID_HOSTNAME => '"%hostname%" неверный домен для адреса "%value%"',
 Zend_Validate_EmailAddress::INVALID_MX_RECORD => 'Домен "%hostname%" не имеет MX-записи об адресе "%value%"',
 Zend_Validate_EmailAddress::DOT_ATOM => '"%localPart%" не соответствует формату dot-atom',
 Zend_Validate_EmailAddress::QUOTED_STRING => '"%localPart%" не соответствует формату указанной строки',
 Zend_Validate_EmailAddress::INVALID_LOCAL_PART => '"%localPart%" не правильное имя для адреса "%value%", вводите адрес вида имя@домен',
 Zend_Validate_EmailAddress::INVALID_FORMAT => "Вы ввели неверный e-mail адрес. Введите e-mail в формате example@domain.com",
 Zend_Validate_Float::NOT_FLOAT => '"%value%" не является дробным числом',
 Zend_Validate_GreaterThan::NOT_GREATER => '"%value%" не превышает "%min%"',
 Zend_Validate_Hex::NOT_HEX => '"%value%" содержит в себе не только шестнадцатеричные символы',
 Zend_Validate_Hostname::IP_ADDRESS_NOT_ALLOWED => '"%value%" - это IP-адрес, но IP-адреса не разрешены ',
 Zend_Validate_Hostname::UNKNOWN_TLD => '"%value%" - это DNS имя хоста, но оно не дожно быть из TLD-списка',
 Zend_Validate_Hostname::INVALID_DASH => '"%value%" - это DNS имя хоста, но знак "-" находится в неправильном месте',
 Zend_Validate_Hostname::INVALID_HOSTNAME_SCHEMA => '"%value%" - это DNS имя хоста, но оно не соответствует TLD для TLD "%tld%"',
 Zend_Validate_Hostname::UNDECIPHERABLE_TLD => '"%value%" - это DNS имя хоста. Не удаётся извлечь TLD часть',
 Zend_Validate_Hostname::INVALID_HOSTNAME => '"%value%" - не соответствует ожидаемой структуре для DNS имени хоста',
 Zend_Validate_Hostname::INVALID_LOCAL_NAME => '"%value%" - адрес является недопустимым локальным сетевым адресом',
 Zend_Validate_Hostname::LOCAL_NAME_NOT_ALLOWED => '"%value%" - адрес является сетевым расположением, но локальные сетевые адреса не разрешены',
 Zend_Validate_Identical::NOT_SAME => 'Значения не совпадают',
 Zend_Validate_Identical::MISSING_TOKEN => 'Не было введено значения для проверки на идентичность',
 Zend_Validate_InArray::NOT_IN_ARRAY => '"%value%" не найдено в перечисленных допустимых значениях',
 Zend_Validate_Int::NOT_INT => '"%value%" не является целочисленным значением',
 Zend_Validate_Ip::NOT_IP_ADDRESS => '"%value%" не является правильным IP-адресом',
 Zend_Validate_LessThan::NOT_LESS => '"%value%" не меньше, чем "%max%"',
 Zend_Validate_NotEmpty::IS_EMPTY => 'Введённое значение пустое, заполните поле, пожалуйста',
 Zend_Validate_Regex::NOT_MATCH => 'Значение "%value%" не подходит под шаблон регулярного выражения "%pattern%"',
 Zend_Validate_StringLength::TOO_SHORT => 'Длина введённого значения "%value%", меньше чем %min% симв.',
 Zend_Validate_StringLength::TOO_LONG => 'Длина введённого значения "%value%", больше чем %max% симв.',
 Zend_Captcha_Word::MISSING_VALUE => 'Empty captcha value',
 Zend_Captcha_Word::MISSING_ID => 'Captcha ID field is missing',
 Zend_Captcha_Word::BAD_CAPTCHA => 'Captcha не верна',
 /**
 * метки для формы
 */
'form.signup.label.login' => 'Введите логин:',
    'form.signup.label.submit' => 'Зарегистрироваться'
/**
 * /метки для формы
 */
);
?>[/cc]
Давайте теперь создадим форму, названия меток которой будут меняться в зависимости от параметра в адресной строке.
[cc lang="php"]
 class Form_Signup extends Zend_Form {

    public function init() {
        //$translator = Zend_Registry::get('Zend_Translate');

        //$this->setTranslator($translator);

        $this->addAttribs(array('class' => 'signup'));

        $login = new Zend_Form_Element_Text('login', array(
                    'label' => 'form.signup.label.login',
                    'maxLength' => '10',
                    'validators' => array(
                        //array('Date', true, array('dd/mm/yy')),
                        array('Regex', true, array('/^\D+$/i'))
                    ),
                    'filters' => array('StringTrim')
                ));
       

        $this->addElements(array(
            $login
        ));
        return $this;
    }

}
?>[/cc]
