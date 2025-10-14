---
subtitle: Тег Twig
---
# {% page %}

Тег `{% page %}` вставляет содержимое [страницы](../../cms/themes/pages.md) в шаблон макета. Базовый пример см. в разделе [макеты](../../cms/themes/layouts.md).

Тег `{% page %}` разбирает исходную разметку шаблона страницы. Шаблон может как заполнять плейсхолдеры, так и определять собственную разметку.

::: cmstemplate
```ini
description="example layout"
```
```twig
<html>
    <head>
        {% placeholder head %}
    </head>
    <body>
        {% page %}
        ...
```
:::

Размещение содержимого в плейсхолдере `head`:

::: cmstemplate
```ini
description="example page"
```
```twig
{% put head %}
    <meta name="foo" content="bar">
{% endput %}

<p>Моё содержимое.</p>
```
:::

Рендер страницы с таким шаблоном даст результат:

```html
<html>
    <head>
        <meta name="foo" content="bar">
    </head>
    <body>
        <p>Моё содержимое.</p>
        ...
```
