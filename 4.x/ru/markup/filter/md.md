---
subtitle: Фильтр Twig
---
# |md

Фильтр `|md` преобразует значение из Markdown в HTML.

```twig
{{ '**Text** is bold.'|md }}
```

Результат будет таким:

```html
<strong>Text</strong> is bold.
```

Дополнительные сведения об использовании Markdown см. в [статье о парсере Markdown](../../extend/services/parser.md).

## |md_safe

Фильтр `|md_safe` разбирает Markdown в безопасном режиме, полностью экранируя весь HTML, кроме базового HTML, который создаётся синтаксисом Markdown. Проще говоря, HTML-разметка экранируется, а также блокируются возможности выполнения сценариев, которые предоставляет Markdown. Разрешены только определённые «безопасные» протоколы HTML, например `https://`, `ftps://`, `mailto:` и т. д.

Следующий JavaScript не будет выполнен:

```twig
{{ '<a href="javascript:alert(1)">click me</a>'|md_safe }}
```

## |md_clean

Фильтр `md_clean` разбирает Markdown, поддерживая больше HTML, чем `|md_safe`, поскольку использует санитайзер для удаления потенциально опасного кода.

```twig
{{ '<script>alert(1)</script>'|md_clean }}
```

#### См. также

::: also
* [Статья о парсере Markdown](../../extend/services/parser.md)
:::
