---
subtitle: Функция Twig
---
# ajaxHandler()

Функция `ajaxHandler()` запускает AJAX-обработчик в Twig и возвращает объект ответа `Cms\Classes\AjaxResponse`. Пример вызова обработчика **onResetPassword**:

```twig
{% set result = ajaxHandler('onResetPassword') %}
```

В результирующем объекте доступны следующие свойства.

Свойство | Данные
------------- | -------------
**data** | Данные, назначенные или возвращённые обработчиком; доступны и при обращении к объекту напрямую.
**error** | Ошибка, возникшая при выполнении обработчика.
**flash** | Flash-сообщения, установленные обработчиком.
**redirect** | Перенаправление, возвращённое обработчиком.

## Доступ к данным

Переменные, присвоенные странице, доступны через результирующий объект. Рассмотрим определение обработчика:

```php
function onResetPassword()
{
    $this['someVariable'] = 'someValue';
}
```

Пример вызова обработчика **onResetPassword**:

```twig
{% set result = ajaxHandler('onResetPassword') %}
```

Переменные из **data**, возвращённые обработчиком или назначенные на страницу во время вызова, доступны через результат.

```twig
{{ result.someVariable }}
```

## Использование ответов

При [создании API в теме](../../cms/resources/building-apis.md) результат можно сразу передать функции [`response()`](./response.md).

```twig
{% do response(ajaxHandler('onResetPassword')) %}
```

При возврате ответа данные будут доступны в JSON в свойстве **data**.

```json
{
    "data": {}
}
```

Переменные страницы не включаются в ответ из соображений безопасности. Вызовите метод `withPageVars`, чтобы добавить их.

```twig
{% do response(ajaxHandler('onResetPassword').withPageVars()) %}
```

Метод `withVars` позволяет добавить к ответу дополнительные данные.

```twig
{% do response(ajaxHandler('onResetPassword').withVars({ 'token': 'foobar' })) %}
```

## Обработка ошибок

Если во время выполнения обработчика произошла ошибка, сообщение будет доступно в свойстве **error.message**. Для `ValidationException` недопустимые поля находятся в **error.fields**.

```twig
{% if result.error %}
    An error occurred: {{ result.error.message }}
{% endif %}
```

При возникновении исключения информация об ошибке появляется в свойстве **error**.

```json
{
    "error": {
        "message": "An error occurred"
    }
}
```

При возврате ответа из AJAX-обработчика для разных типов исключений используются следующие коды состояния.

Исключение | Код состояния
------------- | -------------
`ValidationException` | 422 Unprocessable Entity
`ApplicationException` | 400 Bad Request
`Exception` | 500 Internal Server Error

## Обработка перенаправлений

Если AJAX-обработчик выполнил перенаправление, оно будет доступно в свойстве **redirect** и может быть возвращено напрямую. Например:

```php
function onRedirect()
{
    return Redirect::to('https://octobercms.com');
}
```

Следующий код перенаправит браузер в ответе.

```twig
{% do response(ajaxHandler('onTest')) %}
```

Объект содержит сведения о **redirect** вместе с данными.

```json
{
    "data": {},
    "redirect": "https://octobercms.com"
}
```

## Обработка flash-сообщений

Если использовались flash-сообщения, они будут доступны в свойстве **flash**. Рассмотрим обработчик:

```php
function onTest()
{
    Flash::success('Test successful');
}
```

Вызов обработчика и отправка ответа:

```twig
{% do response(ajaxHandler('onTest')) %}
```

Вывод содержит сообщения **flash** вместе с данными.

```json
{
    "data": {},
    "flash": {
        "success": "Test successful"
    }
}
```

#### См. также

::: also
* [Создание API-ресурсов](../../cms/resources/building-apis.md)
:::
