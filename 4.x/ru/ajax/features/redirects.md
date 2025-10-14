---
subtitle: Узнайте, как перенаправлять на другую страницу или URL.
---
# Перенаправления

Иногда требуется перенаправить пользователя на новую страницу после отправки формы или другого AJAX-запроса.

```html
<form data-request="onSignup">
    <div>
        <label>Email</label>
        <input name="email" />
    </div>

    <button data-attach-loading>
        Sign Up
    </button>
</form>
```

Внутри [AJAX-обработчика](../../ajax/handlers.md) можно вернуть [тип ответа `Redirect`](../../extend/services/response-view.md), где метод `to` принимает относительный или абсолютный URL (первый аргумент).

```php
function onSignup()
{
    return Redirect::to('/signup-complete');
}
```

Метод `refresh` обновляет текущую страницу. Поддерживаются и [всплывающие сообщения](./flash-messages.md).

```php
function onSignup()
{
    Flash::success('Signup complete!');

    return Redirect::refresh();
}
```

## Перенаправление на страницу CMS

Фасад `Cms` и метод `redirect` позволяют перенаправить на конкретную страницу CMS (первый аргумент) с необязательными параметрами маршрута (второй аргумент).

```php
function onRedirect()
{
    return Cms::redirect('blog/post', ['slug' => 'foobar']);
}
```

Метод `pageUrl` возвращает URL в виде строки.

```php
$postPage = Cms::pageUrl('blog/post', ['slug' => 'foobar']);
```

## Перенаправления в Twig

[Функцию Twig `redirect()`](../../markup/function/redirect.md) можно использовать для перенаправления из разметки страницы.

```php
function onSignup()
{
    $this['success'] = true;
}
```

Функция принимает URL или имя страницы CMS.

```twig
{% if success %}
    {% do redirect('/signup-complete') %}
{% endif %}
```

## Перенаправления в AJAX

[AJAX-фреймворк](../../ajax/introduction.md) поддерживает перенаправления через атрибут `data-request-redirect`. Значение атрибута должно содержать URL, на который нужно перейти после успешного AJAX-запроса.

```html
<button
    data-request="onAjax"
    data-request-redirect="/signup-complete">
    Save and Redirect
</button>
```

[Turbo router](../../ajax/turbo-router.md) поддерживает перенаправления с учётом истории через атрибут `data-browser-redirect-back`. Его можно добавить к любой ссылке или элементу с AJAX-запросом. Атрибут переопределяет адрес перенаправления и срабатывает только при наличии предыдущего состояния истории браузера.

```html
<button
    data-request="onRedirect"
    data-browser-redirect-back>
    Save and Back
</button>
```

Атрибут можно использовать и на ссылках.

```html
<a
    href="/home"
    data-browser-redirect-back>
    Go Back
</a>
```

::: warning
Атрибут `data-browser-redirect-back` следует использовать вместе с традиционным перенаправлением как запасным вариантом.
:::

#### См. также

::: also
* [Функция Twig `redirect`](../../markup/function/redirect.md)
* [Ответы и представления](../../extend/services/response-view.md)
:::
