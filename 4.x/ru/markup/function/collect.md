---
subtitle: Функция Twig
---
# collect()

Функция `collect()` предоставляет удобный интерфейс для создания массивов в Twig. Twig намеренно минималистичен как слой представления, и построение массива требует постоянного объединения.

Рассмотрим пример создания массива с использованием встроенного фильтра Twig `|merge`:

```twig
{% set array = [] %}
{% for item in items %}
    {% set array = array|merge([{ title: item.title, ... }]) %}
{% endfor %}
```

Функция `collect()` возвращает [объект коллекции](../../extend/services/collection.md), позволяющий добавлять элементы методом push. Тот же пример можно реализовать методом `push`.

```twig
{% set array = collect() %}
{% for item in items %}
    {% do array.push({ title: item.title, ... }) %}
{% endfor %}
```

Передача массива первым аргументом инициализирует коллекцию заранее заполненными элементами.

```twig
{% set array = collect([
    { title: item.title, ... },
    { title: item.title, ... }
]) %}
```

## shuffle

Метод `shuffle()` перемешивает коллекцию.

```twig
{{ collect(songs).shuffle() }}
```

В цикле `for`:

```twig
{% for fruit in collect(['apple', 'banana', 'orange']).shuffle() %}
    {{ fruit }}
{% endfor %}
```

## sortBy

Методы `sortBy()` и `sortByDesc()` сортируют коллекцию по заданному полю (ключу).

```twig
collect(data).sortBy('age')
```

Например:

```twig
// Output: John David
{% set data = [{'name': 'David', 'age': 31}, {'name': 'John', 'age': 28}] %}

{% for item in collect(data).sortBy('age') %}
    {{ item.name }}&nbsp;
{% endfor %}
```

#### См. также

::: also
* [Создание API-ресурсов](../../cms/resources/building-apis.md)
:::
