---
subtitle: Фильтр Twig
---
# |trans

Фильтры `|trans` и `|trans_choice` переводят переданное значение, используя конфигурацию локализации приложения. Строки локализации можно загрузить, передав значение перевода по умолчанию.

```twig
{{ 'I love programming.'|trans }};
```

Чтобы подставить параметры в перевод, передай массив первым аргументом. Каждый параметр получает префикс `:`.

```twig
{{ ':name loves programming.'|trans({ name: 'Jeff' }) }}
```

## Множественное число

Фильтр `trans_choice` используется для обработки форм множественного числа.

```twig
{{ 'There is one apple|There are many apples'|trans_choice(3) }}
```

Во втором аргументе можно передать параметры.

```twig
{{ '{1} :value minute ago|[2,*] :value minutes ago'|trans_choice(5, { value: 5 }) }}
```

## Сокращённый синтаксис

Фильтры `_` и `__` взаимозаменяемы с `trans` и `trans_choice`.

```twig
{{ 'I love programming.'|_ }}

{{ '{1} :value minute ago|[2,*] :value minutes ago'|__(1, { value: 1 }) }}
```

#### См. также

::: also
* [Локализация тем CMS](../../cms/themes/settings.md)
* [Локализация Laravel](https://laravel.com/docs/12.x/localization)
:::
