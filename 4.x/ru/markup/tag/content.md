---
subtitle: Тег Twig
---
# {% content %}

Тег `{% content %}` выводит [контент-блок CMS](../../cms/themes/content.md) на странице. Чтобы отобразить блок **contacts.htm**, передай имя файла после тега `content` в кавычках.

```twig
{% content "contacts.htm" %}
```

Контент-блок в подкаталоге выводится аналогично.

```twig
{% content "sidebar/content.htm" %}
```

::: tip
Подробнее об использовании подкаталогов см. в [документации по темам](../../cms/themes/themes.md).
:::

Контент-блоки можно выводить как обычный текст:

```twig
{% content "readme.txt" %}
```

Допустим и синтаксис Markdown:

```twig
{% content "changelog.md" %}
```

Контент-блоки можно использовать вместе с [плейсхолдерами макета](../../cms/themes/layouts.md).

```twig
{% put sidebar %}
    {% content 'sidebar-content.htm' %}
{% endput %}
```

## Переменные

Переменные передаются в контент-блоки после имени файла:

```twig
{% content "welcome.htm" name=user.name %}
```

Можно определить новые переменные для использования в контенте:

```twig
{% content "location.htm" city="Vancouver" country="Canada" %}
```

Внутри контента переменные доступны с помощью простого синтаксиса с одинарными *фигурными скобками*:

```
<p>Country: {country}, city: {city}.</p>
```

Также можно передать коллекцию переменных простым массивом:

```twig
{% content "welcome.htm" likes=[
    {name:'Dogs'},
    {name:'Fishing'},
    {name:'Golf'}
] %}
```

Коллекция переменных доступна через парные скобки:

```
<ul>
    {likes}
        <li>{name}</li>
    {/likes}
</ul>
```

> **Примечание.** В контент-блоках синтаксис Twig не поддерживается; при необходимости используйте [частичное представление CMS](../cms/partials.md).

## Сохранение содержимого в переменную Twig

В любом шаблоне можно сохранить содержимое в переменную с помощью функции `content()`. Это позволяет обработать результат перед выводом. Не забудь применить фильтр `|raw`, чтобы отключить экранирование.

```twig
{% set welcomeContent = content('welcome.htm') %}

{{ welcomeContent|raw }}
```

Переменные можно передать вторым аргументом.

```twig
{% set welcomeContent = content('welcome.htm', { foo: 'bar' }) %}
```

## Проверка наличия контента

Функция `hasContent()` позволяет проверить существование контента, не выводя его. Чтобы избежать рендеринга, передай вторым аргументом `false` — функция вернёт `true` или `false` в зависимости от наличия файла.

```twig
{% if hasContent('welcome.htm') %}
    {% content 'welcome.htm' %}
{% else %}
    <p>Блок welcome не найден!</p>
{% endif %}
```

## Разбор контента как строки

При использовании тега `{% content %}` автоматически обрабатываются [сниппеты CMS](../../cms/themes/snippets.md) и ссылки, созданные [виджетом формы Page Finder](../../element/form/widget-pagefinder.md).

Аналогично фильтр `|content` можно применить к HTML-строке, чтобы распознать несколько контент-блоков и подставить их в вывод.

```twig
{{ post.content|content }}
```

Фильтр `|md` также пригоден для разбора Markdown-контента, содержащегося в строке.

```twig
{{ post.markdown_content|md|content }}
```

#### См. также

::: also
* [Сниппеты CMS](../../cms/themes/snippets.md)
* [Фильтр Twig link](../../markup/filter/link.md)
* [Фильтр Twig md](../../markup/filter/md.md)
:::
