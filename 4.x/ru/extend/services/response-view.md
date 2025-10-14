# Response & View

## Базовые ответы

Ответ можно вернуть практически из любого PHP-метода, который используется страницей. Это относится ко всем CMS-методам, описанным в [жизненном цикле выполнения макета](../../cms/themes/layouts.md), и [определениям AJAX-обработчиков](../../ajax/handlers.md).

#### Возврат строк из метода CMS

Возврат строки из метода страницы, макета или компонента останавливает выполнение и переопределяет поведение по умолчанию, поэтому вместо страницы будет показана строка «Hello World».

```php
public function onStart()
{
    return 'Hello World';
}
```

#### Возврат строк из AJAX-обработчиков

Возврат строки из AJAX-обработчика добавляет строку в коллекцию ответа с ключом по умолчанию `result`. Запрошенные частичные представления также будут включены в ответ.

```php
public function onDoSomething()
{
    return 'Hello World';
    // ['result' => 'Hello World']
}
```

#### Возврат строк из маршрутов

Возврат строки из [определения маршрута](../system/routing.md) работает так же, как и из CMS-метода, и возвращает строку в качестве ответа.

```php
Route::get('/', function() {
    return 'Hello World';
});
```

#### Создание пользовательских ответов

Для более гибких сценариев можно возвращать объект `Response`, который предоставляет множество методов для построения HTTP-ответов. Эту тему рассмотрим подробнее далее в статье.

```php
$contents = 'Page not found';
$statusCode = 404;
return Response::make($contents, $statusCode);
```

### Добавление заголовков к ответам

Большинство методов ответа поддерживают цепочки вызовов, что позволяет строить ответ в декларативной форме. Например, можно использовать метод `header`, чтобы добавить набор заголовков к ответу перед отправкой пользователю:

```php
return Response::make($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```

Практический пример — возврат XML-ответа:

```php
return Response::make($xmlString)->header('Content-Type', 'text/xml');
```

### Добавление cookies к ответам

Метод `withCookie` позволяет легко прикреплять cookies к ответу. Например, с его помощью можно создать cookie и прикрепить её к экземпляру ответа:

```php
return Response::make($content)->withCookie('name', 'value');
```

Метод `withCookie` принимает дополнительные необязательные аргументы, чтобы задать свойства cookie:

```php
->withCookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)
```

## Другие типы ответов

Фасад `Response` упрощает создание других типов ответов.

### Ответы с представлением

Если нужен доступ к методам класса `Response`, но ответ должен вернуть представление, используйте метод `Response::view`:

```php
return Response::view('acme.blog::hello')->header('Content-Type', $type);
```

### JSON-ответы

Метод `json` автоматически установит заголовок `Content-Type` в `application/json`, а также преобразует массив в JSON с помощью функции PHP `json_encode`:

```php
return Response::json(['name' => 'Steve', 'state' => 'CA']);
```

Чтобы создать JSONP-ответ, используйте метод `json` совместно с `setCallback`:

```php
return Response::json(['name' => 'Steve', 'state' => 'CA'])
    ->setCallback(Input::get('callback'));
```

### Загрузка файлов

Метод `download` формирует ответ, который заставляет браузер скачать файл по указанному пути (первый аргумент). Второй аргумент — имя файла, отображаемое пользователю, а третий — массив HTTP-заголовков.

```php
return Response::download($pathToFile);

return Response::download($pathToFile, $name, $headers);

return Response::download($pathToFile)->deleteFileAfterSend(true);
```

::: tip
Symfony HttpFoundation, которая управляет загрузкой файлов, требует, чтобы имя скачиваемого файла состояло из ASCII-символов.
:::

#### Потоковые загрузки

Иногда нужно преобразовать строковый ответ в загружаемый без записи содержимого на диск. Метод `streamDownload` решает эту задачу и принимает callback (первый аргумент), имя файла (второй аргумент) и необязательный массив заголовков (третий аргумент).

```php
return Response::streamDownload(function() {
    echo 'CSV Contents...';
}, 'export.csv');
```

### Ответы с файлами

Метод `file` позволяет отобразить файл, например изображение или PDF, непосредственно в браузере пользователя вместо скачивания. Первый аргумент — путь к файлу, второй — массив заголовков.

```php
return Response::file($pathToFile);

return Response::file($pathToFile, $headers);
```

## Редиректы

Ответы-редиректы обычно представлены экземплярами класса `Illuminate\Http\RedirectResponse` и содержат нужные заголовки для перенаправления пользователя на другой URL. Самый простой способ создать `RedirectResponse` — метод `to` фасада `Redirect`.

```php
return Redirect::to('user/login');
```

### Возврат редиректа с флэш-данными

Редиректы и [сохранение данных в сессии](./session.md) часто выполняются вместе. Поэтому можно создать экземпляр `RedirectResponse` и передать флэш-данные одной цепочкой вызовов:

```php
return Redirect::to('user/login')->with('message', 'Login Failed');
```

::: tip
Поскольку метод `with` сохраняет данные в сессии, получить их можно стандартным методом `Session::get`.
:::

#### Редирект на предыдущий URL

Иногда нужно вернуть пользователя на предыдущую страницу, например после отправки формы. Для этого используйте метод `back`:

```php
return Redirect::back();

return Redirect::back()->withInput();
```

#### Редирект на текущую страницу

Иногда требуется просто обновить текущую страницу — используйте метод `refresh`:

```php
return Redirect::refresh();
```

## Макросы Response

Чтобы определить пользовательский ответ и переиспользовать его в маршрутах и контроллерах, используйте метод `Response::macro`:

```php
Response::macro('caps', function($value) {
    return Response::make(strtoupper($value));
});
```

Метод `macro` принимает имя в качестве первого аргумента и замыкание в качестве второго. Это замыкание выполняется при вызове имени макроса на классе `Response`:

```php
return Response::caps('foo');
```

Макросы можно определить в методе `boot` [файла регистрации плагина](../extending.md). Также плагин может содержать файл **init.php**, в котором удобно размещать регистрацию макросов.

## Представления

Представления позволяют хранить системную логику представления, например разметку, используемую API или конечными точками, или разметку, общую для CMS и бэкенда. Представления также используются [почтовым сервисом](../system/sending-mail.md) для шаблонов по умолчанию. Обычно представления размещаются в каталоге `views` плагина.

Простое представление может выглядеть так:

```twig
<!-- View stored in plugins/acme/blog/views/greeting.htm -->

<html>
    <body>
        <h1>Hello, {{ name }}</h1>
    </body>
</html>
```

Представления можно разбирать и с помощью PHP-шаблонов, если указать расширение `.php`:

```php
<!-- View stored in plugins/acme/blog/views/greeting.php -->

<html>
    <body>
        <h1>Hello, <?= $name ?></h1>
    </body>
</html>
```

Это представление можно вернуть в браузер методом `View::make`:

```php
return View::make('acme.blog::greeting', ['name' => 'Charlie']);
```

Первый аргумент — «подсказка пути», содержащая имя плагина, разделённое двойными двоеточиями `::`, и имя файла представления. Второй аргумент `View::make` — массив данных, доступных в представлении.

::: tip
Подсказка пути чувствительна к регистру, имя плагина всегда указывается строчными буквами.
:::

#### Передача данных в представления

```php
// Традиционный подход
$view = View::make('acme.blog::greeting')->with('name', 'Steve');

// Магические методы
$view = View::make('acme.blog::greeting')->withName('steve');
```

В этом примере переменная `name` будет доступна в представлении и будет содержать `Steve`. Как и выше, если нужно передать массив данных, сделайте это вторым аргументом метода `make`:

```php
$view = View::make('acme.blog::greeting', $data);
```

Также можно поделиться значением между всеми представлениями:

```php
View::share('name', 'Steve');
```

#### Передача подчинённого представления

Иногда нужно передать представление в другое представление. Например, подчинённое представление, расположенное в `plugins/acme/blog/views/child/view.php`, можно передать так:

```php
$view = View::make('acme.blog::greeting')->nest('child', 'acme.blog::child.view');

$view = View::make('acme.blog::greeting')->nest('child', 'acme.blog::child.view', $data);
```

Затем подчинённое представление можно отрисовать в родительском представлении:

```twig
<html>
    <body>
        <h1>Hello!</h1>
        {{ child|raw }}
    </body>
</html>
```

#### Проверка существования представления

Чтобы проверить наличие представления, используйте метод `View::exists`:

```php
if (View::exists('acme.blog::mail.customer')) {
    //
}
```

#### См. также

::: also
* [Uploads & Downloads](../../ajax/features/upload-download.md)
* [Laravel Responses](https://laravel.com/docs/12.x/responses)
:::
