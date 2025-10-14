---
subtitle: Добавляет на страницу коллекцию записей модели.
---
# Коллекция

Компонент (component) `collection` отображает на странице коллекцию записей. Компонент можно использовать на любой странице, в любом макете (layout) или частичном представлении (partial).

## Доступные свойства

Компонент поддерживает следующие свойства.

Property | Description
-------- | -------------
**handle** | Служебное имя [blueprint-схемы записи](../tailor/blueprints.md).
**recordsPerPage** | Количество записей, отображаемых на одной странице. Оставьте пустым, чтобы отключить пагинацию.
**pageNumber** | Значение, определяющее, на какой странице находится пользователь.
**sortColumn** | Имя столбца, по которому нужно упорядочить записи.
**sortDirection** | Направление сортировки записей. Поддерживаемые значения — `asc` и `desc`.

## Базовое использование

Ниже приведён пример добавления на страницу коллекции записей **Blog\\Post**. В Twig коллекция доступна через перебор переменной `collection` по умолчанию.

::: cmstemplate
```ini
[collection]
handle = "Blog\\Post"
```
```twig
{% for post in collection %}
    <h1>{{ post.title }}</h1>
{% endfor %}
```
:::

Если на одной странице используется несколько коллекций, компоненту можно присвоить имя **posts** с помощью псевдонима компонента. Именно под этим именем переменная будет доступна на странице. Следующая коллекция доступна через переменную `posts`.

::: cmstemplate
```ini
[collection posts]
handle = "Blog\\Post"
```
```twig
{% for post in posts %}
    <h1>{{ post.title }}</h1>
{% endfor %}
```
:::

Используйте выражения `is empty` или `is not empty`, чтобы проверить, содержит ли коллекция хотя бы одну запись для отображения.

```twig
{% if posts is not empty %}
    {# ... #}
{% endif %}
```

## Выполнение запросов

При обращении к переменной компонента через метод происходит переключение на [запрос модели](../../extend/database/query.md). Например, чтобы показывать только записи, у которых поле `color` имеет значение **blue**, используйте метод запроса `where`. С помощью тега Twig `{% set %}` результат можно присвоить новой переменной.

```twig
{% set bluePosts = posts.where('color', 'blue').get() %}
```

Чтобы отображать только записи с привязанным автором, примените метод запроса `whereRelation`. Для завершения запроса вызовите метод `get()`. В следующем примере выводятся записи, связанные с автором `author`, у которого slug равен **bella-vista**.

```twig
{% set authorPosts = posts.whereRelation('author', 'slug', 'bella-vista').get() %}
```

### Доступ к типу записи

Чтобы отфильтровать записи по типу, выполните запрос `where()` по атрибуту `content_group`. Следующий пример выводит записи типа **featured_post**.

```twig
{% set featuredPosts = posts.where('content_group', 'featured_post').get() %}
```

### Пагинация записей

Можно применить к коллекции пагинацию с помощью метода `paginate()`. В следующем примере записи разбиваются постранично по **10** штук, а затем выводится навигация пагинатора.

```twig
{% set authorPosts = posts.whereRelation(...).paginate(10) %}

{{ pager(authorPosts) }}
```

::: tip
Подробнее о постраничном выводе записей см. раздел [Pagination](../../ajax/features/pagination.md).
:::

### Поиск по записям

Для поиска используйте [метод `searchWhere()`](../../extend/database/query.md), чтобы выполнить запрос по значениям в столбцах. Следующий пример выполняет регистронезависимый поиск по переданному термину и указанным столбцам.

```twig
{% set foundPages = pages.searchWhere(searchTerm, ['title', 'content']).get() %}
```

Можно воспользоваться и [методом `searchWhereRelation()`](../../extend/database/relations.md), чтобы искать связанные записи. В этом случае имя связи включается в метод для проверки существования связи.

```twig
{% set foundPages = pages.searchWhereRelation(searchTerm, 'author', ['title']).get() %}
```

## Жадная загрузка связанных записей

В некоторых случаях для повышения производительности может потребоваться жадная загрузка связанных записей. Вызовите у коллекции метод `load`, передав имя связи. В следующем примере связь `categories` будет загружена для каждой записи в коллекции.

```twig
{% do authorPosts.load('categories') %}
```

## Подсчёт записей

Предположим, требуется вывести список категорий блога и подсчитать количество записей в каждой категории. Сначала определите [обратную связь](../../element/content/field-entries.md) в blueprint-схеме категории под именем **posts**.

```yaml
posts:
    type: entries
    source: Blog\Post
    inverse: categories
    hidden: true
```

Затем вызовите функцию `withCount` у компонента коллекции, а после — `get`, чтобы вернуть коллекцию записей категорий. Это создаст для каждой категории атрибут **post_count**.

```twig
{% set categories = collection.withCount('posts').get() %}
{% for category in categories %}
    <h5>{{ category.title }} ({{ category.post_count }} posts)</h5>
{% endfor %}
```

#### См. также

::: also
* [Pagination](../../ajax/features/pagination.md)
* [Model Queries](../../extend/database/model.md)
* [Database Relationships](../../extend/database/relations.md)
:::
