---
subtitle: Функция Twig
---
# str()

Функции с префиксом `str_` выполняют полезные операции со строками. Вспомогательная функция напрямую отображается на PHP-класс `Str` и его методы. Например:

```twig
{{ str_camel() }}
```

Эквивалент на PHP выглядит так:

```php
<?= Str::camel() ?>
```

::: warning
Методы в формате *camelCase* следует записывать в формате *snake_case*.
:::

Строковые функции можно применять и как фильтр Twig.

```twig
{{ ''|str_camel }}
```

## str_limit()

Ограничивает количество символов в строке.

```twig
{{ str_limit('The quick brown fox...', 100) }}
```

Чтобы добавить суффикс при обрезке строки, передай его третьим аргументом. По умолчанию используется `...`.

```twig
{{ str_limit('The quick brown fox...', 100, '... Read more!') }}
```

## str_words()

Ограничивает количество слов в строке.

```twig
{{ str_words('The quick brown fox...', 100) }}
```

Чтобы добавить суффикс при обрезке строки, передай его третьим аргументом. По умолчанию используется `...`.

```twig
{{ str_words('The quick brown fox...', 100, '... Read more!') }}
```

## str_replace

Заменяет все вхождения строки поиска строкой-заменой.

```twig
// Outputs: Bob
{{ 'Alice'|str_replace('Alice', 'Bob') }}
```

## str_camel()

Преобразует значение в формат *camelCase*.

```twig
// Outputs: helloWorld
{{ str_camel('hello world') }}
```

## str_studly()

Преобразует значение в формат *StudlyCase*.

```twig
// Outputs: HelloWorld
{{ str_studly('hello world') }}
```

## str_snake()

Преобразует значение в формат *snake_case*.

```twig
// Outputs: hello_world
{{ str_snake('hello world') }}
```

Второй аргумент может задать разделитель.

```twig
// Outputs: hello---world
{{ str_snake('hello world', '---') }}
```

## str_plural()

Возвращает форму множественного числа английского слова.

```twig
// Outputs: chickens
{{ str_plural('chicken') }}
```

## str_upper()

Преобразует строку в верхний регистр.

```twig
// Outputs: Hello I'm JACK
Hello I'm {{ 'Jack'|str_upper }}
```

## str_lower()

Преобразует строку в нижний регистр.

```twig
// Outputs: Hello I'm jack
Hello I'm {{ 'JACK'|str_lower }}
```

## str_ucfirst()

Делает первый символ строки заглавным.

```twig
// Outputs: Hello I'm Jack
Hello I'm {{ 'jack'|str_ucfirst }}
```

## str_lcfirst()

Делает первый символ строки строчным.

```twig
// Outputs: Hello I'm jack
Hello I'm {{ 'Jack'|str_lcfirst }}
```

## str_repeat()

Повторяет строку.

```twig
// Outputs: We are the best best best!
We are the {{ 'best '|str_repeat(3) }}!
```

## str_pad_both

Дополняет строку до заданной длины другой строкой с обеих сторон.

```twig
// Outputs: ooxxxoo
{{ 'xxx'|str_pad_both(7, 'o') }}
```

## str_pad_left

Дополняет строку до заданной длины другой строкой слева.

```twig
// Outputs: ooxxx
{{ 'xxx'|str_pad_left(5, 'o') }}
```

## str_pad_right

Дополняет строку до заданной длины другой строкой справа.

```twig
// Outputs: xxxoo
{{ 'xxx'|str_pad_right(5, 'o') }}
```

## str_reverse

Разворачивает строку.

```twig
// Outputs: !dlrow olleH
{{ 'Hello world!'|str_reverse }}
```
