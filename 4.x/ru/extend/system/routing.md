---
subtitle: Полезно для определения фиксированных API и конечных точек.
---
# Маршрутизация и посредники

Маршруты для [контроллеров бэкенда](../system/controllers.md) обрабатываются автоматически, а страницы CMS задают свои URL в [конфигурации страниц](../../cms/themes/pages.md). Плагин может также включать файл **routes.php** с пользовательской логикой маршрутизации, как описано в [службе маршрутизатора Laravel](https://laravel.com/docs/12.x/routing).

::: dir
├── plugins
|   └── acme  _← Имя автора_
|       └── blog  _← Имя плагина_
|           ├── controllers
|           ├── models
|           ├── Plugin.php
|           └── `routes.php`  _← Файл маршрутов_
:::

При таком подходе маршруты определяются в PHP через фасад `Route`. Ниже пример маршрута, доступного по GET-запросу **https://yoursite.tld/api_acme_blog/cleanup_posts**.

```php
Route::get('api_acme_blog/cleanup_posts', function() {
    return Posts::cleanUp();
});
```

URL маршрута можно сгенерировать фасадом `Url`.

```php
$url = Url::to('api_acme_blog/cleanup_posts');
```

## Базовая маршрутизация

Для определения маршрутов используются методы, соответствующие HTTP-методам: `get`, `post`, `patch`, `put`, `options`, `delete`. Простейшие маршруты принимают URI и `Closure`.

```php
Route::get('/', function () {
    return 'Hello World';
});

Route::post('foo/bar', function () {
    return 'Hello World';
});

Route::put('foo/bar', function () {
    //
});

Route::delete('foo/bar', function () {
    //
});
```

### Регистрация нескольких методов

Иногда требуется маршрут, реагирующий на несколько HTTP-методов. Используйте метод `match`.

```php
Route::match(['get', 'post'], '/', function () {
    return 'Hello World';
});
```

Можно зарегистрировать маршрут, обрабатывающий все HTTP-методы, с помощью `any`.

```php
Route::any('foo', function () {
    return 'Hello World';
});
```

## Маршрутизация на класс

В крупных приложениях удобнее организовывать маршруты внутри классов вместо замыканий. Рекомендуемое место для таких классов — каталог **handlers**. Маршрут указывается массивом с именем класса и методом. В примере маршрут `/install` привязан к классу `Installer` и методу `install`.

```php
Route::any('/install', [Installer::class, 'install']);
```

Затем определите класс и метод. В этом примере файл расположен в **app/handlers/Installer.php**.

```php
namespace App\Handlers;

class Installer extends \Illuminate\Routing\Controller
{
    /**
     * Route: /install
     */
    public function install()
    {
        return 'Welcome!';
    }
}
```

## Параметры маршрута

Чтобы перехватывать сегменты URI в маршруте, определите параметры маршрута. Например, извлечём идентификатор пользователя из URL.

```php
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```

Количество параметров маршрута не ограничено.

```php
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```

Параметры маршрута заключаются в одиночные фигурные скобки. При выполнении маршрута параметры передаются в `Closure`.

::: warning
Параметры маршрутов не могут содержать символ `-`; используйте подчёркивание (`_`).
:::

### Необязательные параметры

Иногда параметр маршрута должен быть необязательным. Добавьте `?` после имени параметра:

```php
Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

### Ограничения регулярными выражениями

Формат параметров можно ограничить методом `where`, передав имя параметра и регулярное выражение.

```php
Route::get('user/{name}', function ($name) {
    //
})->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
    //
})->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

## Именованные маршруты

Именованные маршруты упрощают генерацию URL или редиректов. Имя задаётся ключом `as` при определении маршрута:

```php
Route::get('user/profile', ['as' => 'profile', function () {
    //
}]);
```

#### Группы маршрутов и имена

Если используются группы маршрутов (см. ниже), можно указать ключ `as` в массиве атрибутов группы и задать общий префикс имён.

```php
Route::group(['as' => 'admin::'], function () {
    Route::get('dashboard', ['as' => 'dashboard', function () {
        // Маршрут с именем "admin::dashboard"
    }]);
});
```

#### Генерация URL по именованным маршрутам

После присвоения имени маршруту можно использовать его при генерации URL или редиректов через `Url::route`.

```php
$url = Url::route('profile');

$redirect = Response::redirect()->route('profile');
```

Если маршрут содержит параметры, передайте их вторым аргументом метода `route`. Значения будут автоматически подставлены в URL.

```php
Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
    //
}]);

$url = Url::route('profile', ['id' => 1]);
```

## Группы маршрутов

Группы маршрутов позволяют задавать общие атрибуты для множества маршрутов без повторения. Общие атрибуты передаются первым аргументом в `Route::group` в виде массива.

### Маршруты поддоменов

Группы можно использовать для маршрутизации поддоменов с подстановками. Поддомену можно присвоить параметры так же, как URI маршрута, чтобы использовать их в маршруте или контроллере. Поддомен задаётся ключом `domain` в массиве атрибутов:

```php
Route::group(['domain' => '{account}.example.tld'], function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```

### Префиксы маршрутов

Атрибут `prefix` добавляет префикс ко всем маршрутам группы. Например, чтобы добавить `admin` ко всем маршрутам группы:

```php
Route::group(['prefix' => 'admin'], function () {
    Route::get('users', function () {
        // Соответствует URL "/admin/users"
    });
});
```

Параметры также можно включить в префикс.

```php
Route::group(['prefix' => 'accounts/{account_id}'], function () {
    Route::get('detail', function ($account_id) {
        // Соответствует URL accounts/{account_id}/detail
    });
});
```

### Посредники маршрутов

Если зарегистрировать посредник (middleware) в методе `boot()` плагина, он будет применяться глобально для каждого запроса. Чтобы назначить посредник отдельному маршруту, используйте следующий синтаксис:

```php
Route::get('info', [\App\News::class, 'info'])->middleware(\Path\To\Your\Middleware::class);
```

Для групп маршрутов:

```php
Route::group(['middleware' => \Path\To\Your\Middleware::class], function() {
    Route::get('info', [\App\News::class, 'info']);
});
```

Чтобы назначить одному маршруту несколько посредников, можно написать так:

```php
Route::middleware([\Path\To\Your\Middleware::class])->group(function() {
    Route::get('info', [\App\News::class, 'info']);
});
```

::: tip
В группе можно указать несколько посредников; в примерах показан один для простоты.
:::

## Глобальные посредники

Чтобы зарегистрировать глобальный посредник, расширьте класс контроллера `Cms\Classes\CmsController` или `Backend\Classes\BackendController` следующим образом.

```php
public function boot()
{
    \Cms\Classes\CmsController::extend(function($controller) {
        $controller->middleware(\App\Middleware::class);
    });
}
```

Либо добавьте посредник напрямую в Kernel в методе `boot()` регистрационного файла.

```php
public function boot()
{
    // Добавить посредник в начало стека
    $this->app[\Illuminate\Contracts\Http\Kernel::class]
        ->prependMiddleware(\App\Middleware::class);

    // Добавить посредник в конец стека
    $this->app[\Illuminate\Contracts\Http\Kernel::class]
        ->pushMiddleware(\App\Middleware::class);
}
```

## Генерация ошибок 404

Есть два способа вручную вызвать ошибку 404 из маршрута. Во-первых, используйте хелпер `abort`, который выбрасывает `Symfony\Component\HttpFoundation\Exception\HttpException` с указанным кодом состояния.

```php
App::abort(404);
```

Также можно выбросить `October\Rain\Exception\NotFoundException`. Подробнее о обработке 404 и пользовательских ответах см. раздел [Ошибки и логирование](../system/exceptions.md).

#### См. также

::: also
* [Маршрутизация в Laravel](https://laravel.com/docs/12.x/routing)
* [Eloquent API Resources](https://laravel.com/docs/12.x/eloquent-resources)
:::
