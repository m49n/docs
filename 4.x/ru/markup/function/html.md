---
subtitle: Функция Twig
---
# html()

Функции с префиксом `html_` выполняют полезные операции с HTML-разметкой. Вспомогательная функция напрямую отображается на PHP-класс `Html` и его методы. Например:

```twig
{{ html_strip() }}
```

Эквивалент на PHP выглядит так:

```php
<?= Html::strip() ?>
```

::: warning
Методы в формате *camelCase* следует записывать в формате *snake_case*.
:::

HTML-функции можно применять и как фильтр Twig.

```twig
{{ ''|html_strip }}
```

## html_strip()

Удаляет HTML из строки.

```twig
// Outputs: Hello world
{{ '<strong>Hello world</strong>'|html_strip }}
```

Первым аргументом можно передать разрешённые теги.

```twig
// Outputs: <p>Text</p>
{{ '<p><b>Text</b></p>'|html_strip('<p>') }}
```

## html_limit()

Ограничивает HTML до указанной длины с корректной обработкой тегов.

```twig
{{ '<p>Post content...</p>'|html_limit(100) }}
```

Чтобы добавить суффикс при обрезке, передай его вторым аргументом. По умолчанию используется `...`.

```twig
{{ '<p>Post content...</p>'|html_limit(100, '... Read more!') }}
```

## html_clean()

Очищает HTML, предотвращая большинство XSS-атак.

```twig
{{ '<script>window.location = "http://google.com"</script>'|html_clean }}
```

## html_email()

Маскирует адрес электронной почты, чтобы защитить его от спам-ботов.

```twig
{{ 'me@mysite.tld'|html_email }}
```

Например:

```twig
<a href="mailto: {{ 'me@mysite.tld'|html_email }}">Email me</a>

<!-- The above will output -->
<a href="mailto: &#109;&#97;&#105;&#108;&#x74;o&#x3a;&#97;&#64;b.&#x63;">Email me</a>
```

## html_mailto()

Выводит полноценную ссылку с маскированным адресом электронной почты.

```twig
{{ 'me@mysite.tld'|html_mailto }}
```
