---
subtitle: Функция Twig
---
# pager()

Функция `pager()` используется для работы с [пагинированными записями](../../extend/database/pagination.md) (первый аргумент). Она возвращает объект с информацией о записях, включая номера страниц и ссылки «вперёд/назад». При преобразовании к строке выводится стандартная HTML-разметка.

После получения результатов их можно вывести и отрендерить ссылки страниц с помощью функции `pager()` в Twig.

```twig
<div class="container">
    {% for user in users %}
        {{ user.name }}
    {% endfor %}
</div>

{{ pager(users) }}
```

Поддерживаются следующие настраиваемые опции (второй аргумент).

Опция | Описание
------------- | -------------
**template** | Указывает стандартный шаблон или [имя представления](../../extend/services/response-view.md). Пример: `app::my-custom-view`
**partial** | Указывает [имя частичного представления](../../cms/themes/partials.md) в теме (только CMS). Пример: `my-partial`
**withQuery** | Добавляет существующие параметры запроса в сгенерированные ссылки. По умолчанию `false`
**appends** | Необязательный массив значений, добавляемых в параметры запроса.
**fragment** | Необязательная строка фрагмента, добавляемая к URL.

## Изменение URL

Используйте `withQuery`, чтобы сохранить текущую строку запроса в URL.

```twig
{{ pager(records, { withQuery: true }) }}
```

Параметры можно дополнить, используя опцию `appends`. Например, чтобы добавить `&sort=votes` к каждой ссылке пагинации, вызовите `appends` следующим образом.

```twig
{{ pager(records, { appends: { sort: 'votes' } }) }}
```

Чтобы добавить «хеш»-фрагмент к URL пагинации, примените опцию `fragment`. Например, чтобы дописать `#foo` в конец каждой ссылки, вызовите `fragment` так.

```twig
{{ pager(records, { fragment: 'foo' }) }}
```

## Доступ к переменным пагинатора

Если сохранить результат `pager()` в переменную, можно получить ссылки и метаданные пагинированного запроса. Это особенно полезно при [создании API-эндпоинтов](../../cms/resources/building-apis.md) (JSON), но также применимо для доступа к переменным в Twig.

Начнём с пагинированной коллекции.

```twig
{% set records = postModel.paginate(3) %}
```

Функция `pager()` вернёт извлечённый объект.

```twig
{% set paginator = pager(records) %}
```

После этого каждое значение доступно по отдельности.

```twig
<a href="{{ paginator.links.first }}"></a>
```

Возвращаемый объект разделён на **links** и **meta** со следующими атрибутами.

Атрибут | Описание
------------- | -------------
**links.first** | URL первой страницы
**links.last** | URL последней страницы
**links.prev** | URL предыдущей страницы
**links.next** | URL следующей страницы
**meta.path** | URL текущей страницы
**meta.per_page** | Количество записей на странице
**meta.total** | Общее число записей
**meta.current_page** | Текущий номер страницы
**meta.last_page** | Последний номер страницы
**meta.from** | Номер первой записи
**meta.to** | Номер последней записи

Пример в формате JSON.

```json
{
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

## Рендеринг пагинатора

Если вызвать `pager()` напрямую и преобразовать в строку, будет выведен стандартный системный шаблон пагинации.

```twig
{{ pager(records) }}
```

Связанная функция `ajaxPager()` выводит шаблон пагинации с поддержкой AJAX (см. AJAX-шаблон ниже). Её стоит использовать внутри [AJAX-частичного представления](../tag/ajax-partial.md).

```twig
{{ ajaxPager(records) }}
```

### Шаблон по умолчанию

Шаблон `default` рендерит стандартную пагинацию. Используется по умолчанию методом `paginate()` в запросе к базе данных.

```html
<ul class="pagination">
    <li class="page-item first">
        <span class="page-link">&larr;</span>
    </li>
    <li class="page-item">
        <a class="page-link" href="?page=1">1</a>
    </li>
    <li class="page-item last">
        <a class="page-link" href="?page=2">&rarr;</a>
    </li>
</ul>
```

Расположение файла: `~/modules/system/views/pagination/default.htm`

### Простой шаблон

Шаблон `simple` выводит пагинацию только с кнопками «вперёд» и «назад». Используется по умолчанию методом `simplePaginate()` в запросе к базе данных.

```html
<ul class="pagination">
    <li class="page-item first">
        <span class="page-link">&larr;</span>
    </li>
    <li class="page-item last">
        <a class="page-link" href="?page=2">&rarr;</a>
    </li>
</ul>
```

Расположение файла: `~/modules/system/views/pagination/simple.htm`

### AJAX-шаблон

Шаблон `ajax` выводит пагинацию с поддержкой AJAX. Используется по умолчанию методом `paginate()` в запросе к базе данных и функцией `ajaxPager()`.

```html
<ul class="pagination">
    <li class="page-item first">
        <span class="page-link">&larr;</span>
    </li>
    <li class="page-item">
        <a
            class="page-link"
            data-request="onAjax"
            data-request-data="{ page: 1 }"
            data-request-update="{ _self: true }">1</a>
    </li>
    <li class="page-item last">
        <a
            class="page-link"
            data-request="onAjax"
            data-request-data="{ page: 2 }"
            data-request-update="{ _self: true }">&rarr;</a>
    </li>
</ul>
```

Расположение файла: `~/modules/system/views/pagination/ajax.htm`

## Пользовательская разметка

::: tip
Инструкции по использованию собственной разметки пагинации см. в [статье о пагинации](../../ajax/features/pagination.md).
:::

#### См. также

::: also
* [Создание API-ресурсов](../../cms/resources/building-apis.md)
* [Пагинация CMS](../../ajax/features/pagination.md)
* [Пагинация моделей](../../extend/database/pagination.md)
:::
