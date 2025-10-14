---
subtitle: Тег Twig
---
# {% partial %}

Тег `{% partial %}` разбирает [частичное представление CMS](../../cms/themes/partials.md) и выводит его содержимое на странице. Чтобы показать partial **footer.htm**, передай имя после тега `partial` в кавычках.

```twig
{% partial "footer" %}
```

Частичное представление в подкаталоге вызывается так же.

```twig
{% partial "sidebar/menu" %}
```

::: tip
Подробнее об использовании подкаталогов см. в [документации по темам](../../cms/themes/themes.md).
:::

Имя partial можно передать переменной:

```twig
{% set tabName = "profile" %}
{% partial tabName %}
```

## Переменные

Переменные передаются после имени partial:

```twig
{% partial "blog-posts" posts=posts %}
```

Можно определить новые переменные для использования внутри partial:

```twig
{% partial "location" city="Vancouver" country="Canada" %}
```

Внутри partial переменные доступны как обычные переменные разметки:

```twig
<p>Country: {{ country }}, city: {{ city }}.</p>
```

## Передача разметки как переменной

Разметку можно передать в partial с помощью атрибута `body`.

```twig
{% partial "card" body %}
    Здесь содержимое карточки
{% endpartial %}
```

Содержимое будет доступно как переменная `body`.

```twig
{{ body|raw }}
```

### Компонуемые partial

Компонуемые partial возможны в сочетании с [тегом Twig `{% placeholder %}`](./placeholder.md). Следующий partial определяет секции `header` и `body`, куда можно добавить HTML-содержимое.

```twig
<div class="header">
    {% placeholder header %}
</div>
<div class="body">
    {{ body|raw }}
</div>
```

Далее можно использовать тег `{% put %}` внутри `body`, чтобы составить результат partial из двух HTML-секций.

```twig
{% partial "card" body %}
    {% put header %}
        <h2>Это заголовок карточки</h2>
    {% endput %}
    <p>Это содержимое карточки</p>
{% endpartial %}
```

## Сохранение содержимого partial в переменную Twig

В любом шаблоне можно сохранить содержимое partial в переменную функцией `partial()`. Это позволяет обработать вывод перед отображением. Не забудь применить фильтр `|raw`, чтобы отключить экранирование.

```twig
{% set cardPartial = partial('my-cards/card') %}

{{ cardPartial|raw }}
```

Переменные можно передать вторым аргументом.

```twig
{% set cardPartial = partial('my-cards/card', { foo: 'bar' }) %}
```

## Проверка наличия partial

Функция `hasPartial()` позволяет проверить наличие partial, не выводя содержимое; она возвращает `true` или `false`.

```twig
{% if hasPartial('my-cards/card') %}
    {% partial 'my-cards/card' %}
{% else %}
    <p>Карточка не найдена!</p>
{% endif %}
```
