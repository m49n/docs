---
subtitle: Научитесь создавать простой API с использованием страниц CMS.
---
# Создание конечных точек API

::: aside
Ознакомьтесь со [статьёй о маршрутизации](../../extend/system/routing.md), если предпочтительнее определять маршруты API с помощью PHP.
:::

При работе с фреймворками на стороне клиента, например Vue.js или React, необходимо потреблять серверные API. Их можно определять в теме, где каждая страница представляет конечную точку API.

Страница выступает слоем трансформации между компонентами (components) CMS и ответами JSON, которые возвращаются приложению. Большинство объектов, таких как модели и коллекции, поддерживают сериализацию в JSON и могут возвращаться напрямую как ответ.

## Отправка ответа

В простейшем случае ресурс API можно сформировать, возвращая переменную Twig с помощью [функции Twig](../../markup/function/response.md) `response()`. Эта функция переопределяет содержимое страницы и возвращает в браузер пользовательский ответ.

::: cmstemplate
```ini
url = "/api/foobar"
```
```twig
{% do response({ foo: 'bar' }) %}
```
:::

Вызов выше вернёт ответ с типом содержимого `application/json`.

```json
{ "foo": "bar" }
```

В большинстве случаев потребуется преобразовывать переменные компонента в ответ.

```twig
{% do response({
    id: post.id,
    title: post.title,
    email: post.author.email,
    created_at: post.created_at,
    updated_at: post.updated_at
}) %}
```

### Коллекции

[Функция Twig](../../markup/function/collect.md) `collect()` формирует коллекцию для ответа. Метод `push` добавляет элементы в коллекцию и позволяет настраивать каждый результат.

```twig
{% set result = collect() %}

{% for post in posts %}
    {% do result.push({
        id: post.id,
        title: post.title,
        email: post.author.email,
        created_at: post.created_at,
        updated_at: post.updated_at
    }) %}
{% endfor %}

{% do response(result) %}
```

## Условия

Любые условия Twig можно использовать в разметке, чтобы влиять на ответ. Последний вызов `response` будет отправлен в браузер.

### Проверка HTTP-метода

Используйте [свойство Twig](../../markup/property/this-request.md) `this.request.method`, чтобы проверить метод запроса.

```twig
{% if this.request.method == 'GET' %}
    <!-- Логика для GET -->
{% else %}
    <!-- Метод не поддерживается -->
{% endif %}
```

### Прерывание запроса

[Функция Twig](../../markup/function/abort.md) `abort()` позволяет прервать запрос с ответом 404.

```twig
{% if post %}
    {% do response(post) %}
{% else %}
    {% do abort(404) %}
{% endif %}
```

## Работа со страницами и макетами

Поскольку API определяется в разделе Markup страницы или макета (layout), доступны все компоненты и события жизненного цикла.

### Использование макетов как посредника (middleware)

Посредник (middleware) позволяет применить общую логику к нескольким конечным точкам, например проверять аутентификацию или ограничивать частоту запросов. [Макет (layout) CMS с режимом приоритета](../themes/layouts.md) можно использовать для применения логики к нескольким страницам, и логика макета выполняется до логики страницы.

Не забудьте включить [тег Twig `{% page %}`](../../markup/tag/page.md), чтобы логика страницы была задействована. Например, макет с именем **api.htm** может содержать любую условную логику.

::: cmstemplate
```ini
description = "API Authentication"
is_priority = 1
```
```twig
{% if someCondition %}
    {% page %}
{% else %}
    {% do response({ message: 'Условие не выполнено' }, 400) %}
{% endif %}
```
:::

Каждая страница, использующая макет, получит условия из этого макета.

::: cmstemplate
```ini
layout = "api"
```
```twig
{% do response({ success: true }) %}
```
:::

::: warning
Всегда используйте [режим приоритета в макете](../themes/layouts.md), чтобы содержимое макета выполнялось первым.
:::

### Вызов AJAX-обработчиков

В некоторых случаях требуется вызвать AJAX-обработчик компонента или страницы. Это возможно с помощью [функции Twig](../../markup/function/ajax-handler.md) `ajaxHandler()`.

::: cmstemplate
```ini
url = "/api/signin

[account]
```
```twig
{% set result = ajaxHandler('onSignin') %}

{% if result.error %}
    {% do response({ message: 'Не удалось выполнить вход' }, 401) %}
{% else %}
    {% do response({ success: true }) %}
{% endif %}
```
:::

Можно также вызвать обработчик и сразу передать его результат как ответ. Ответ включает переменные, заданные на странице, и массивы, возвращённые функцией.

```twig
{% do response(ajaxHandler('onSubmitPost')) %}
```

Перенаправления также обрабатываются автоматически. Подробности см. в [статье о функции Twig](../../markup/function/ajax-handler.md).

## Работа с ресурсами

При работе с моделями и коллекциями рекомендуется возвращать данные, обёрнутые в атрибут **data**. Такое обёртывание обеспечивает единый интерфейс.

### Модели и коллекции

Возврат ресурса модели.

::: cmstemplate
```ini
url = "/api/blog/post/:slug"

[section post]
handle = "Blog\\Post"
identifier = "slug"
```
```twig
{% if post %}
    {% do response({
        data: post
    }) %}
{% else %}
    {% do abort(404) %}
{% endif %}
```
:::

Возврат ресурса коллекции.

::: cmstemplate
```ini
url = "/api/blog/posts"

[collection posts]
handle = "Blog\\Post"
```
```twig
{% do response({
    data: posts
}) %}
```
:::

### Пагинация

При ответе с пагинируемой коллекцией рекомендуется использовать [функцию Twig](../../markup/function/pager.md) `pager()` для формирования ответа с атрибутами **links** и **meta**.

```twig
{% set posts = blog.paginate(3) %}

{% set pager = pager(posts) %}

{% do response({
    data: posts,
    links: pager.links,
    meta: pager.meta
}) %}
```

Пример выше выводит JSON следующего формата.

```json
{
    "data": {},
    "links": {
        "first": "https://yoursite.tld/api/blog/posts?page=1",
        "last": "https://yoursite.tld/api/blog/posts?page=1",
        "prev": null,
        "next": null
    },
    "meta": {
        "path": "https://yoursite.tld/api/blog/posts",
        "per_page": 3,
        "total": 2,
        "current_page": 1,
        "last_page": 1,
        "from": 1,
        "to": 2
    }
}
```

## Примеры использования

Ниже приведены практические примеры того, как можно использовать сниппеты.

### Возврат пользователей с миниатюрами аватаров

В следующем примере переменной `users` присваиваются все пользователи, найденные в [плагине User](https://octobercms.com/plugin/rainlab-user). Связь `avatar` загружается заблаговременно, после чего атрибут `avatar_thumb` устанавливается как URL миниатюры для каждого пользователя, если аватар найден.

::: cmstemplate
```ini
## pages/api/users.htm
url = "/api/users"
```
```php
function onStart()
{
    $this['users'] = \RainLab\User\Models\User::all();
}
```
```twig
{# Загружаем связь avatar #}
{% do users.load('avatar') %}

{# Устанавливаем атрибут 'avatar_thumb' для каждого пользователя #}
{% for user in users %}
    {% do user.setAttribute(
        'avatar_thumb',
        user.avatar.getThumbUrl(100, 100, {mode: 'crop'})|default(null)
    ) %}
{% endfor %}

{# Возвращаем пользователей #}
{% do response({
    data: users
}) %}
```
:::

#### См. также

::: also
* [Функция Twig response](../../markup/function/response.md)
* [Функция Twig ajaxHandler](../../markup/function/ajax-handler.md)
:::
