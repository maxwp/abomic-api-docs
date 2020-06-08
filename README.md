# Abomic.com REST API documentation

## HTTP vs HTTPS

https://

только https.

## Передача параметров
Можно передавать методом GET (urlencoded), POST (urlencoded) или JSON POST.
Порядок параметров значения не имеет.

## Выдача результатов
Всегда в JSON. Всегда есть два поля:

- status = success или error
- resultArray = результат, выдача зависит от методов внутри
- опционально может выводиться третье поле deprecated:true в случае если вы вызываете какой-то старый метод (из api v1) или используете устаревшую конструкцию. Если вы видите поле deprecated:true - то лучше сразу отказаться от использования этого метода.

## Терминология
- user - пользователь, по сути просто набор name + email + phone + password, который может авторизироваться в abomic
- market - рынок (категория).
- customer - заказчик (лид)

## Поля customer

- customer.type=new/cold/hot/deleted
-- new - новая заявка, еще не прошла модерацию
-- hot - новая горячая заявка
-- cold - бесплатная заявка, hot становится cold через 3-7 дней (в зависимости от особенности market'a)
-- deleted - удалена, мусор
- name - string - имя человека или название компании
- email - string - почта
- phone - string - телефон
- description - string - описание customer'a (что ему надо) или provider'a (что он предлагает)
- descriptionsecret - string - секретное описание, которое открывается после покупки
- location - string - локация customer'a где он предоставляет услуги
- cdate - дата создания заявки


## Общая логика работы:
- пользователи регистрируют аккаунты с системе
- могут смотреть список всех customer'-ов в заданном market'e
- могут открывать контакты customer'-ов в зависимости от тарифа


## Авторизация
https://api.abomic.com/auth-login.php?login=XXX&password=XXX

- Важно - password передавать в md5! Перед тем как отправлять пароль - заверните его в md5 один раз.
- Вернет token, который нужен будет дальше для авторизации.

Далее во все методы можно передавать:
- ?token=xxx (рекомендуется)
- можно передавать token через заголовок X-AUTH-TOKEN
- можно передавать login и пароль через headers X-AUTH-LOGIN, X-AUTH-PASSWORD
- в поле login можно передавать именно логин, емейл или телефон. Но стоит понимать, что при регистрации поля login нет, его могут назначить только администраторы.
- Важно: токен безлимитный по времени (пока-что).

## Основные коды ошибок
- auth - что-то не так с авторизацией, неверный токен или пара логин-пароль
- invalid-xxx - неверное значение поле, которое ты передаешь
- no-balance - закончились деньги на балансе
- customerid - неверный номер заказчика или он недоступен вам
- marketid - неверный номер маркета или он недоступен вам

## Деавторизация (выход)
https://api.abomic.com/auth-logout.php?token=xxx

## регистрация нового юзера
https://api.abomic.com/auth-register.php?name=xxx&email=xxx&phone=xxx&password=xxx

Важно: если метод вернет success - то это не значит, что юзер авторизирован. Для авторизации вызывай login и получай token.

Коды ошибок:
- invalid-name
- invalid-email
- invalid-phone
- invalid-password (минимально 5 символов, любых)
- email-exists
- phone-exists






## Получить маркеты (категории)
https://api.abomic.com/market-list.php

метод работает как без авторизации, так и с авторизацией.
Если с авторизацией - то в ответе будут поля watch (true/false).

необязательные параметры:
- search (string) - поиск по категориям
- marketid (int) - выдать категорию с заданным id
- all=1 - вернет все активные маркеты (полный список, может быть тысячи)


## Получить конкретный маркет
https://api.abomic.com/market-get.php?marketid=xxx

Возвращает информацию о рынке и список последних customer-ов (лидов) на рынке.
Есть параметры пагинации: page & limit (default = 50)

## получить конкретного customer'a с рынка
https://api.abomic.com/market-customer-get.php?customerid=xxx

Авторизация не требуется. Но если авторизация была передана и пользователь уже открывал этого customer'a - ему покажутся контакты.
Иначе, контакты будут закрыты звездочками.

## создать customer'a на рынке (создать заявку)
https://api.abomic.com/market-customer-add.php

Требуется авторизация.

Параметры:

- marketid (обязательно)
- name (обязательно)
- email (обязательно phone OR email)
- phone (обязательно phone OR email)
- description (обязательно, хотя-бы 1 символ)
- location (не обязательно)

Метод вернет все поля customer'a. Структура ответа аналогична методу market/customer/get/
В том числе будет возвращен customerid.

Есть нюанс - проверка email OR phone. Иными словами, достаточно передать хотя-бы что-то одно.

Коды ошибок:

- invalid-name
- invalid-email
- invalid-phone
- need-email-or-phone
- invalid-description
- invalid-marketid


## открыть заданного customer'a и получить его контакты
https://api.abomic.com/market-customer-open.php?customerid=xxx

Требуется авторизация.
Под открыть понимается "получить контакты" (и возможно забрать с рынка).

Параметры:
- customerid (обязательно)

Коды ошибок:
- invalid-customerid
- customer-yours - вы пытаетесь открыть своего лида
- no-balance - не хватает баланса для покупки



## получить данные моего профиля и купленные тарифы
https://api.abomic.com/user-profile.php

Требуется авторизация.
Без параметров.


## получить все мои контакты
https://api.abomic.com/user-myleads.php

Метод возвращает единый массив объектов customer/provider с которыми у вас установлена связь (котоые вы открыли).
Требуется авторизация.

Дополнительные параметры: skip & limit (для пагинации)

