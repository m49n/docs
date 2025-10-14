---
subtitle: Узнайте, как выводить ссылки пагинации.
---
# Пагинация

October CMS включает возможности пагинации из коробки, они интегрируются со стандартными шаблонами и полностью поддерживают пользовательскую разметку. Пагинированные записи тесно связаны с [пагинацией запросов моделей](../../extend/database/pagination.md) и [функцией Twig `pager()`](../../markup/function/pager.md).

## Пагинация данных

Набор данных с пагинацией можно получить из [логики компонента](../../extend/cms-components.md), из [PHP-секции](../themes/themes.md) страницы или макета, а также из [компонента Tailor](../tailor/components.md). Ниже показан пример страницы, которая получает пагинированные данные из компонента Tailor по 10 записей на страницу.

::: cmstemplate
```ini
url = "/blog"

[collection]
handle = "Blog\Post"
```
```twig
{% set posts = collection.paginate(10) %}
```
:::

Теперь переменная `posts` доступна. Можно пройти по каждой записи и вывести ссылки пагинации.

```twig
<div>
    {% for post in posts %}
        <h2>{{ post.title }}</h2>
    {% endfor %}
</div>

<nav>
    {{ pager(posts) }}
</nav>
```

### Несколько экземпляров пагинации

По умолчанию пагинация берёт номер страницы из строки запроса `?page`, поэтому один и тот же номер используется для двух и более наборов пагинированных данных. Чтобы решить проблему, используйте метод `paginateCustom`, указав уникальное имя параметра.

::: cmstemplate
```ini
url = "/blog"

[collection blog]
handle = "Blog\Post"

[collection category]
handle = "Blog\Category"
```
```twig
{% set posts = blog.paginateCustom(10, 'postPage') %}

{% set comments = comments.paginateCustom(10, 'commentPage') %}
```
:::

Установите опцию `withQuery`, чтобы сохранять номер страницы для других экземпляров пагинации (необязательно).

```twig
{{ pager(categories, { withQuery: true }) }}
```

В результате строка запроса содержит оба номера страниц, например<br>`?postPage=1&commentPage=2`.

### Пользовательская разметка пагинации

Чтобы использовать пользовательскую разметку, начните с указанных ниже файлов и скопируйте содержимое в частичное представление темы.

Template | Детали
------------- | -------------
`default` | Рендерит стандартный шаблон пагинации.<br>Расположение: `~/modules/system/views/pagination/default.htm`
`simple` | Рендерит пагинацию только с кнопками «вперёд» и «назад».<br>Расположение: `~/modules/system/views/pagination/simple.htm`
`ajax` | Рендерит пагинацию с AJAX.<br>Расположение: `~/modules/system/views/pagination/ajax.htm`

Затем отрендерите частичное представление, передав опцию `partial` функции `pager`.

```twig
{{ pager(records, { partial: 'my-custom-pagination' }) }}
```

## Пагинация AJAX

Используйте функцию Twig `ajaxPager()`, чтобы динамически обновлять пагинированные записи через AJAX. Частичное представление должно выводить записи и включать пагинатор. Например, частичное представление **latest-posts.htm** со следующим содержимым:

```twig
<div>
    {% for post in posts %}
        <h2>{{ post.title }}</h2>
    {% endfor %}
</div>

<nav>
    {{ ajaxPager(posts) }}
</nav>
```

Затем отрендерите частичное представление на странице с помощью [тега Twig `{% ajaxPartial %}`](../../markup/tag/ajax-partial.md).

::: cmstemplate
```ini
url = "/blog"

[collection blog]
handle = "Blog\Post"
```
```twig
{% set posts = blog.paginate(10) %}

<h3>Latest Posts</h3>
{% ajaxPartial 'latest-posts' %}
```
:::

Также можно поместить всю логику внутрь частичного представления, чтобы сделать его полностью переносимым.

::: cmstemplate
```ini
[collection blog]
handle = "Blog\Post"
```
```twig
{% set posts = blog.paginate(10) %}

<div>
    {% for post in posts %}
        <h2>{{ post.title }}</h2>
    {% endfor %}
</div>

<nav>
    {{ ajaxPager(posts) }}
</nav>
```
:::

Такое частичное представление можно подключить на любой странице или в макете без дополнительной конфигурации.

::: cmstemplate
```ini
url = "/blog"
```
```twig
{% ajaxPartial 'latest-posts' %}
```
:::

## Пагинация «Загрузить ещё»

Кнопка «Загрузить ещё», также известная как «бесконечная» загрузка, позволяет показывать записи в одном списке вместо перехода по страницам.

Этот подход использует AJAX-частичное представление, которое добавляет новое содержимое и самоуничтожающуюся кнопку. Например, частичное представление **load-more-posts.htm** со следующим содержимым:

```twig
{% set posts = blog.paginate(10) %}

<div>
    {% for post in posts %}
        <h2>{{ post.title }}</h2>
    {% endfor %}
</div>

{% if posts.hasMorePages %}
    <button
        data-request="onAjax"
        data-request-update="{ _self: '@' }"
        data-request-success="this.remove()"
        data-request-data="{ page: {{ posts.currentPage + 1 }} }"
        data-attach-loading>
        Load More
    </button>
{% endif %}
```

Кнопка использует сочетание AJAX-атрибутов данных для [самообновления в режиме добавления](../../ajax/update-partials.md), передаёт номер следующей страницы и удаляет себя после завершения.

Частичное представление следует выводить с помощью [тега Twig `{% ajaxPartial %}`](../../markup/tag/ajax-partial.md).

::: cmstemplate
```ini
url = "/blog"
```
```twig
{% ajaxPartial 'load-more-posts' %}
```
:::

#### См. также

::: also
* [Пагинация моделей](../../extend/database/pagination.md)
* [Функция Twig `pager`](../../markup/function/pager.md)
:::
