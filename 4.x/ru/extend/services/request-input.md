# Request & Input

## Базовый ввод

Получить все данные пользователя можно несколькими простыми методами. При использовании фасада `Input` не нужно заботиться о HTTP-методе запроса — доступ к данным осуществляется одинаково для всех методов. Глобальная [вспомогательная функция](./helpers.md) `input` является псевдонимом `Input::get`.

#### Получение значения ввода

```php
$name = Input::get('name');

$name = input('name');
```

#### Получение значения по умолчанию, если ввод отсутствует

```php
$name = Input::get('name', 'Sally');
```

#### Проверка наличия значения ввода

```php
if (Input::has('name')) {
    //
}
```

#### Получение всех данных запроса

```php
$input = Input::all();
```

#### Получение отдельных данных запроса

```php
$input = Input::only('username', 'password');

$input = Input::except('credit_card');
```

При работе с полями форм-массивами можно использовать точечную нотацию для доступа к элементам массива:

```php
$input = Input::get('products.0.name');
```

::: tip
Некоторые JavaScript-библиотеки, например Backbone, отправляют данные в приложение в формате JSON. Получить их можно через `Input::get` как обычно.
:::

## Cookies

По умолчанию все cookies, созданные October CMS, шифруются и подписываются кодом аутентификации, поэтому будут считаться недействительными, если были изменены на стороне клиента. Cookies, перечисленные в параметре конфигурации `system.unencrypt_cookies`, не шифруются.

#### Получение значения cookie

```php
$value = Cookie::get('name');
```

#### Добавление cookie к ответу

```php
$response = Response::make('Hello World');

$response->withCookie(Cookie::make('name', 'value', $minutes));
```

#### Постановка cookie в очередь для следующего ответа

Если нужно установить cookie до создания ответа, используйте метод `Cookie::queue`. Cookie автоматически будет прикреплён к финальному ответу приложения.

```php
Cookie::queue($name, $value, $minutes);
```

#### Создание бессрочного cookie

```php
$cookie = Cookie::forever('name', 'value');
```

#### Работа с cookie без шифрования

Если не нужно шифровать/дешифровать некоторые cookies, можно указать их в конфигурации. Это полезно, например, при передаче данных между фронтендом и серверной частью через cookies и обратно.

Добавьте имена cookies, которые не нужно шифровать или дешифровать, в параметр `unencrypt_cookies` конфигурационного файла `config/system.php`.

```php
'unencrypt_cookies' => [
    'my_cookie',
],
```

Для плагинов можно добавить cookies динамически в файле `Plugin.php`.

```php
public function boot()
{
    Config::push('system.unencrypt_cookies', 'my_cookie');
}
```

## Старые данные

Иногда нужно сохранить введённые данные между запросами. Например, чтобы повторно заполнить форму после проверки на ошибки валидации.

#### Сохранение данных во флэш-сессию

```php
Input::flash();
```

#### Сохранение части данных во флэш-сессию

```php
Input::flashOnly('username', 'email');

Input::flashExcept('password');
```

Поскольку часто нужно одновременно сохранять данные и перенаправлять на предыдущую страницу, можно объединять методы флэш-сохранения с редиректом.

```php
return Redirect::to('form')->withInput();

return Redirect::to('form')->withInput(Input::except('password'));
```

::: tip
Другие данные между запросами можно сохранять через класс [Session](./session.md).
:::

#### Получение старых данных

Получение отдельного значения ввода.

```php
Input::old('username');
```

Получение всех старых значений ввода.

```php
$data = Input::old();
```

## Файлы

Загруженные файлы можно получить из запроса методом `file` фасада `Input` или глобальной функцией `files()`.

```php
$file = Input::file('photo');

$file = files('photo');
```

Чтобы проверить наличие файла в запросе, используйте метод `hasFile`.

```php
if (Input::hasFile('photo')) {
    //
}
```

Помимо проверки наличия файла можно убедиться, что при загрузке не возникло ошибок, методом `isValid`.

```php
if ($file->isValid()) {
    //
}
```

Возвращаемый объект файла содержит различные методы, связанные с загруженным файлом.

Method Name | Purpose
------------- | -------------
**move($destinationPath, $fileName)** | Перемещает загруженный файл в локальный путь
**store($folder, $disk)** | Сохраняет файл с использованием [сервиса хранилища](../../extend/services/storage.md).
**storeAs($folder, $name, $disk)** | Сохраняет файл под указанным именем через [сервис хранилища](../../extend/services/storage.md).
**extension()** | Определяет расширение по содержимому файла
**getRealPath()** | Возвращает локальный путь
**getClientOriginalName()** | Возвращает исходное имя
**getClientOriginalExtension()** | Возвращает исходное расширение
**getSize()** | Возвращает размер
**getMimeType()** | Возвращает MIME-тип

Пример перемещения загруженного файла.

```php
$file->move($destinationPath);

$file->move($destinationPath, $fileName);
```

Примеры [сохранения загруженного файла](./storage.md) в папку (первый аргумент) и на указанный диск (второй аргумент).

```php
$file->store($folder);

$file->store($folder, 's3');
```

::: warning
Названия папок `media`, `resources`, `uploads` и `public` зарезервированы.
:::

Примеры `storeAs`, сохраняющего файл с пользовательским именем (второй аргумент). Метод `storePubliclyAs` сохраняет файл с публичным доступом.

```php
$file->storeAs($folder, 'avatar');

$file->storeAs($folder, 'avatar', 's3');

$file->storePublicly($folder, 's3');

$file->storePubliclyAs($folder, 'avatar', 's3');
```

Пример получения пути к загруженному файлу.

```php
$path = $file->getRealPath();
```

Пример получения исходного имени загруженного файла.

```php
$name = $file->getClientOriginalName();
```

Получение расширения загруженного файла.

```php
$extension = $file->getClientOriginalExtension();
```

Получение размера загруженного файла.

```php
$size = $file->getSize();
```

Получение MIME-типа загруженного файла.

```php
$mime = $file->getMimeType();
```

## Сведения о запросе

Класс `Request` предоставляет множество методов для анализа HTTP-запроса приложения и наследует класс `Symfony\Component\HttpFoundation\Request`. Ниже перечислены основные из них.

#### Получение URI запроса

```php
$uri = Request::path();
```

#### Получение метода запроса

```php
$method = Request::method();

if (Request::isMethod('post')) {
    //
}
```

#### Проверка соответствия пути запроса шаблону

```php
if (Request::is('admin/*')) {
    //
}
```

#### Получение URL запроса

```php
$url = Request::url();
```

#### Получение сегмента URI запроса

```php
$segment = Request::segment(1);
```

#### Получение заголовка запроса

```php
$value = Request::header('Content-Type');
```

#### Получение значения из $_SERVER

```php
$value = Request::server('PATH_INFO');
```

#### Проверка, что запрос выполнен по HTTPS

```php
if (Request::secure()) {
    //
}
```

#### Проверка, что запрос отправлен AJAX

```php
if (Request::ajax()) {
    //
}
```

#### Проверка, что запрос имеет тип содержимого JSON

```php
if (Request::isJson()) {
    //
}
```

#### Проверка, что запрошен ответ в формате JSON

```php
if (Request::wantsJson()) {
    //
}
```

#### Проверка требуемого формата ответа

Метод `Request::format` возвращает требуемый формат ответа на основе заголовка HTTP Accept:

```php
if (Request::format() == 'json') {
    //
}
```
