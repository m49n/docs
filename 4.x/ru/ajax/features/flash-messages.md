---
subtitle: Показывать сообщение о результате запроса.
---
# Всплывающие сообщения

Всплывающие сообщения позволяют быстро сообщить пользователю результат запроса — успешный он или завершился ошибкой. Просто используйте фасад `Flash`, чтобы вывести сообщение после завершения запроса. Обычно всплывающие сообщения задаются внутри [AJAX-обработчиков](../../ajax/handlers.md), [логики компонента](../../extend/cms-components.md) или [PHP-секции](../themes/themes.md) страницы либо макета.

```php
function onSave()
{
    // Sets a successful message
    Flash::success('Settings successfully saved!');

    // Sets an error message
    Flash::error('Something went wrong...');

    // Sets a warning message
    Flash::warning('Please confirm your email address soon');

    // Sets an informative message
    Flash::info('The export is still processing. Please try again in a minute.');
}
```

Всплывающие сообщения исчезают через 3 секунды. Щелчок по сообщению останавливает его скрытие.

## Встроенные всплывающие сообщения

AJAX-фреймворк имеет встроенную поддержку всплывающих сообщений: достаточно указать атрибут `data-request-flash` на форме, чтобы включить сообщения для завершённых AJAX-запросов.

```html
<form
    data-request="onSuccess"
    data-request-flash>
    <!-- ... -->
</form>
```

Чтобы всплывающие сообщения отображались и при перенаправлении браузера, отрендерьте [встроенное всплывающее сообщение](../../markup/tag/flash.md) при загрузке страницы, разместив следующий код на странице или в макете.

```twig
{% flash %}
    <p
        data-control="flash-message"
        data-type="{{ type }}"
        data-interval="5">
        {{ message }}
    </p>
{% endflash %}
```

Чтобы отображать только определённые типы сообщений, передайте значение атрибуту — **success**, **error**, **info**, **warning** или **validate**. Несколько значений разделяются запятыми.

```html
<form data-request-flash="success,warning"></form>
```

При использовании [валидации](./validation.md) вместе с атрибутом `data-request-flash` ошибки валидации имеют приоритет и скрывают всплывающее сообщение. Чтобы показать и то и другое, добавьте тип **validate** к атрибуту.

```html
<form
    data-request-validate
    data-request-flash="success,error,validate">
```

### Сообщение во время загрузки

Атрибут `data-request-message` позволяет показывать сообщение о прогрессе, пока запрос выполняется. Это особенно полезно для длительных процессов.

```html
<button
    data-request="onSubmit"
    data-request-message="Please wait while we process your request...">
    Submit
</button>
```

### Стилизация всплывающего сообщения

Чтобы изменить внешний вид всплывающего сообщения, используйте CSS-класс `.oc-flash-message`.

```css
.oc-flash-message.success {
    background: green;
}
.oc-flash-message.error {
    background: red;
}
.oc-flash-message.warning {
    background: orange;
}
.oc-flash-message.info {
    background: aqua;
}
.oc-flash-message.loading {
    background: aqua;
}
```

## Пользовательские всплывающие сообщения

::: aside
Подробнее о теге `{% flash %}` см. в статье [Flash Twig Tag](../../markup/tag/flash.md).
:::

Чтобы вывести всплывающие сообщения inline или полностью изменить стандартную разметку, создайте в теме новое частичное представление с нужным содержимым. Например, создайте частичное представление **flash-messages.htm** и вставьте следующий код.

```twig
{% flash %}
    <div class="alert alert-{{ type }}">
        {{ message }}
    </div>
{% endflash %}
```

Затем подключите частичное представление в форме как [самообновляющееся частичное представление](../../markup/tag/ajax-partial.md) с помощью тега `{% ajaxPartial %}`. Указание имени частичного представления в `data-request-update` автоматически обновит его и отключит встроенные всплывающие сообщения.

```twig
<form>
    {% ajaxPartial 'flash-messages' %}

    <label>Title</label>
    <input name="title" />

    <button
        data-request="onSave"
        data-request-update="{ flash-messages: true }">
        Save
    </button>
</form>
```

Или можно включить частичное представление в макет и обновлять его глобально вместо добавления `data-request-flash` каждому элементу. Добавьте метатег `ajax-request-update` в раздел head страницы и задайте атрибут content для [глобального обновления частичного представления](../../ajax/update-partials.md).

```html
<head>
    <meta name="ajax-request-update" content="{ flash-messages: true }" />
</head>
<body>
    <!-- Updates with every AJAX request -->
    {% ajaxPartial 'flash-messages' %}
</body>
```

## Работа с JavaScript

Используйте функцию `oc.flashMsg`, чтобы вывести всплывающее сообщение через JavaScript. Тип можно задать как `success`, `error` или `warning`. Необязательный параметр `interval` определяет, сколько секунд показывать сообщение.

```js
oc.flashMsg({
    message: 'Record has been successfully saved. This message will disappear in 1 second.',
    type: 'success',
    interval: 1
});
```

#### См. также

::: also
* [Flash Twig Tag](../../markup/tag/flash.md)
:::
