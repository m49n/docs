---
subtitle: Фильтр Twig
---
# |link

Фильтр `|link` возвращает ссылку, сгенерированную по схеме `october://`, которую выдаёт [виджет формы Page Finder](../../element/form/widget-pagefinder.md). Результатом будет публичный URL страницы, заданной в виджете.

```twig
<a href="{{ 'october://cms-page@link/about'|link }}" />
```

::: tip
Если нужно разобрать HTML с несколькими ссылками и преобразовать их в HTTP-ссылки в выводе, см. [фильтр Twig `|content`](../tag/content.md).
:::

## link()

Дополняющая функция `link()` используется для получения более подробных сведений о ссылке.

```twig
{% set resolved = link('october://cms-page@link/about') %}

{{ resolved.url }}
```

В результирующем объекте доступны следующие свойства.

Свойство | Данные
------------- | -------------
**url** | публичный URL страницы.
**mtime** | время изменения ссылки на страницу.
**title** | удобочитаемый заголовок ссылки, необязательное.
**items** | массив с сгенерированными дочерними элементами, необязательное.
**isActive** | `true`, если ссылка активна в текущий момент.

Можно запросить вложенные дочерние элементы, передав опцию `nesting` со значением `true` (второй аргумент). Это заполнит свойство `items` в результате.

```twig
{% set resolved = link('october://...', { nesting: true }) %}

{% for subitem in resolved.items %}
    {{ subitem.url }}
{% endfor %}
```

Чтобы получить URL других сайтов, передай опцию `sites` со значением `true`; это заполнит свойство `sites` в результате.

```twig
{% set resolved = link('october://...', { sites: true }) %}

{% for site in resolved.sites %}
    {{ site.url }}
{% endfor %}
```

## Интерфейс PHP

Ссылки можно разрешить в PHP с помощью класса `Cms\Classes\PageManager`. Метод `url` возвращает строку с публичным URL.

```php
Cms\Classes\PageManager::url('october://cms-page@link/about');
```

Метод `resolve` возвращает подробный объект `Cms\Models\PageLookupItem`.

```php
$page = Cms\Classes\PageManager::resolve('october://cms-page@link/about');

echo $page->url;
```

#### См. также

::: also
* [Виджет формы Page Finder](../../element/form/widget-pagefinder.md)
:::
