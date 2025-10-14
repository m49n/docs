---
subtitle: Проектирование API и динамическое обновление страницы.
---
# Обработчики событий

AJAX-обработчики событий — это конечные точки API, через которые AJAX-фреймворк взаимодействует с сервером. Они могут возвращать сырые данные, перенаправлять браузер или динамически обновлять частичные представления на странице.

## AJAX-обработчики

Чтобы создать AJAX-обработчик, определите его в виде PHP-функции в PHP-секции страницы, частичного представления или макета либо [внутри компонентов CMS](../themes/components.md). Имена обработчиков должны следовать шаблону `onSomething`, например `onName`. Все обработчики поддерживают [обновление частичных представлений](./update-partials.md) в составе AJAX-запроса.

```php
function onSubmitContactForm()
{
    // ...
}
```

::: tip
Если обработчики с одинаковым именем определены одновременно на странице и в макете, приоритет будет у обработчика страницы. Обработчики, определённые в компонентах, имеют самый низкий приоритет.
:::

### Вызов обработчика

Каждый AJAX-запрос должен указывать имя обработчика с помощью [API атрибутов данных](../ajax/attributes-api.md) или [JavaScript API](../ajax/javascript-api.md). При выполнении запроса сервер ищет все зарегистрированные обработчики и находит первый совпадающий.

```html
<!-- Attributes API -->
<button data-request="onSubmitContactForm">Go</button>

<!-- JavaScript API -->
<script> oc.ajax('onSubmitContactForm') </script>
```

Обработчики, определённые страницами, макетами и компонентами, регистрируются автоматически. Если обработчик вызывается внутри частичного представления, используйте [Twig-тег `{% ajaxPartial %}`](../../markup/tag/ajax-partial.md), который корректирует цикл страницы, чтобы зарегистрировать его обработчики.

### Сериализация формы

Когда AJAX-запрос выполняется внутри тега HTML-формы, все значения ввода формы доступны обработчику. В примере ниже значение `first_name` будет отправлено вместе с запросом.

```html
<form id="myForm">
    <input name="first_name" />
    <button data-request="onSubmitContactForm">Go</button>
</form>
```

JavaScript API поддерживает эту логику с помощью функции `oc.request`.

```html
<script> oc.request('#myForm', 'onSubmitContactForm') </script>
```

Можно использовать PHP-функцию `input()` для доступа к переменной.

```php
function onSubmitContactForm()
{
    $firstName = input('first_name');
}
```

### Универсальный обработчик

Иногда требуется отправить AJAX-запрос только ради обновления содержимого страницы без выполнения какого-либо кода. Для этого можно использовать обработчик `onAjax`. Он доступен везде и не требует написания кода.

```html
<button data-request="onAjax">Do nothing</button>
```

### Обработчики компонентов

Если два компонента регистрируют обработчик с одинаковым именем, рекомендуется добавить префикс с [кратким именем или псевдонимом компонента](../../cms/themes/components.md). Если компонент использует псевдоним **mycomponent**, к обработчику можно обратиться как к `mycomponent::onName`.

```html
<button data-request="mycomponent::onSubmitContactForm">Go</button>
```

Подробнее см. в статье [Разработка компонентов](../../extend/cms-components.md).

## Перенаправления в AJAX-обработчиках

Если необходимо перенаправить браузер в другое место, верните объект ответа `Redirect` из AJAX-обработчика. Фреймворк перенаправит браузер сразу после получения ответа от сервера. Пример AJAX-обработчика с перенаправлением:

```php
function onRedirectMe()
{
    return Redirect::to('http://google.com');
}
```

## Возврат данных из AJAX-обработчиков

Ответ AJAX-обработчика может служить потребляемым API, если возвращает структурированные данные. Если AJAX-обработчик возвращает массив, доступ к его элементам можно получить в обработчике события `success`. Пример AJAX-обработчика, возвращающего объект данных:

```php
function onFetchDataFromServer()
{
    // Some server-side code

    return [
        'totalUsers' => 1000,
        'totalProjects' => 937
    ];
}
```

Данные можно получить с помощью API атрибутов данных.

```html
<form data-request="onHandleForm" data-request-success="console.log(data)">
```

То же самое через JavaScript API.

```html
<form onsubmit="oc.request(this, 'onHandleForm', {
        success: function(data) {
            console.log(data);
        }
    }); return false"
>
```

## Запуск кода до обработчиков

Иногда нужно выполнить код до запуска обработчика. Определение функции `onInit` в рамках [жизненного цикла выполнения макета](../../cms/themes/layouts.md) позволяет запускать код перед каждым AJAX-обработчиком.

```php
function onInit()
{
    // From a page or layout PHP code section
}
```

Можно определить метод `init` внутри [класса компонента CMS](../../extend/cms-components.md).

```php
function init()
{
    // From a component or widget class
}
```

## Генерация AJAX-исключения

Можно сгенерировать [AJAX-исключение](../../extend/system/exceptions.md) с помощью класса `AjaxException`, чтобы обработать ответ как ошибку, при этом сохранив возможность отправлять содержимое ответа в обычном формате. Просто передайте содержимое ответа первым аргументом исключения.

```php
throw new AjaxException([
    'error' => 'Not enough questions',
    'questionsNeeded' => 2
]);
```

Эти ошибки обрабатываются AJAX-фреймворком.

```html
<form data-request="onHandleForm" data-request-error="console.log(data)">
```

То же самое через JavaScript API.

```html
<form onsubmit="oc.request(this, 'onHandleForm', {
        error: function(data) {
            console.log(data);
        }
    }); return false"
>
```

::: tip
При генерации этого типа исключения [частичные представления обновляются](./update-partials.md) в обычном порядке.
:::

## Отправка событий браузера

::: aside
Отправленные события инициируются в ответе AJAX после завершения запроса и до обновления частичных представлений.
:::

Можно отправлять JavaScript-события из AJAX-обработчиков с помощью метода `dispatchBrowserEvent`. Метод принимает любое имя события (первый аргумент) и переменные для передачи в событие (второй аргумент); переменные должны поддерживать сериализацию в JSON.

```php
function onPerformAction()
{
    $this->dispatchBrowserEvent('app:update-profile');

    $this->dispatchBrowserEvent('app:update-profile', ['name' => 'Jeff']);
}
```

В браузере используйте `addEventListener`, чтобы прослушивать отправленное событие после завершения AJAX-запроса. Переменные события доступны через объект `event.detail`.

```js
addEventListener('app:update-profile', function (event) {
    alert('Profile updated with name: ' + event.detail.name);
});
```
