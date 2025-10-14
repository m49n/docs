# Журнал

По умолчанию October настроен на создание одного файла журнала приложения, который хранится в каталоге `storage/logs`. Записывать сведения в журнал можно через фасад (facade) `Log`.

```php
$user = User::find(1);
Log::info('Showing user profile for user: '.$user->name);
```

Логгер предоставляет восемь уровней логирования, определённых в [RFC 5424](https://datatracker.ietf.org/doc/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** и **debug**.

```php
Log::emergency($error);
Log::alert($error);
Log::critical($error);
Log::error($error);
Log::warning($error);
Log::notice($error);
Log::info($error);
Log::debug($error);
```

#### Контекстная информация

В методы журнала можно передавать массив контекстных данных. Эти данные будут отформатированы и показаны вместе с сообщением журнала:

```php
Log::info('User failed to login.', ['id' => $user->id]);
```

### Вспомогательные функции

Для упрощения логирования доступны глобальные вспомогательные функции. Функция `trace_log` является псевдонимом `Log::info` с поддержкой массивов и исключений в качестве сообщения.

```php
// Запись строкового значения
$val = 'Hello world';
trace_log('The value is '.$val);

// Выгрузка массивов
$val = ['Some', 'array', 'data'];
trace_log($val);

// Трассировка исключений
try {
    //
}
catch (Exception $ex) {
    trace_log($ex);
}
```

Функция `trace_sql` включает логирование базы данных. После вызова она будет записывать каждую команду, отправленную в базу данных. Эти записи появляются только в файле `system.log` и не отображаются в журнале панели бэкенда, так как тот хранится в базе данных, что привело бы к циклической записи.

```php
trace_sql();

Db::table('users')->count();

// select count(*) as aggregate from users
```
