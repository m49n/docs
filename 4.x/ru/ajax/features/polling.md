---
subtitle: Используется для отложенной загрузки и повторяющихся обновлений.
---
# Опрос

Опрос — приём, позволяющий отложить или повторить AJAX-обновление, добавив атрибут `data-auto-submit` элементу запроса. Эту возможность следует применять осторожно, поскольку при больших нагрузках она может увеличивать нагрузку на сервер; для снижения эффекта атрибут срабатывает только когда окно браузера пользователя активно.

## Отложенные запросы

Запросы опроса часто используются вместе с [Twig-тегом AJAX Partial](../../markup/tag/ajax-partial.md), поскольку он позволяет частичным представлениям обновляться самостоятельно. Рассмотрим AJAX-обработчик на странице, загружающий результаты. Поскольку операция затратная, можно запросить данные после полной загрузки страницы.

```php
public function onFetchResults()
{
    $this['results'] = [1, 2, 3];
}
```

Следующий шаг — подключить на странице частичное представление с поддержкой AJAX под названием **posts.htm**.

```twig
{% ajaxPartial 'posts' %}
```

Внутри частичного представления содержимое выводит результаты, если переменная `results` существует. В противном случае отображается сообщение о загрузке с атрибутом `data-auto-submit`, который загружает результаты вторичным запросом.

```twig
{% if results %}
    <h3>Found results</h3>
    {{ d(results) }}
{% else %}
    <h3>Loading the results...</h3>
    <div
        data-request="onFetchResults"
        data-request-update="{ _self: true }"
        data-auto-submit>
    </div>
{% endif %}
```

## Повторяющиеся запросы

Чтобы периодически обновлять содержимое, задайте `data-auto-submit` числовое значение (в миллисекундах), которое запускает автоматический запрос после задержки. Например, частичное представление, рендеримое через `{% ajaxPartial %}`, может содержать такую разметку:

```twig
<div>
    {% set launchDate = carbon('2025-01-01') %}
    {% set days = launchDate.diffInDays %}
    {% set hours = launchDate.subDays(days).diffInHours %}
    {% set minutes = launchDate.subHours(hours).diffInMinutes %}
    {% set seconds = launchDate.subMinutes(minutes).diffInSeconds %}

    <h2>
        Launch in...
        {{ days }} days,
        {{ hours }} hours,
        {{ minutes }} minutes,
        {{ seconds }} seconds
    </h2>
</div>

<div
    data-request="onAjax"
    data-request-update="{ _self: true }"
    data-auto-submit="2000"></div>
```

Частичное представление отображает таймер обратного отсчёта и включает элемент `div`, который автоматически обновляет себя. Поскольку ответ также содержит этот элемент, запрос повторяется каждые 2 секунды.

Опрос можно остановить, если в следующих ответах не включать элемент. Например, код ниже прекращает опрос, если переменная `launchDone` равна true.

```twig
{% if not launchDone %}
    <div
        data-request="onAjax"
        data-request-update="{ _self: true }"
        data-auto-submit="2000"></div>
{% endif %}
```

## Отложенная загрузка частичных представлений

Тег `data-auto-submit` используется Twig-тегом `{% ajaxPartial lazy %}` для отложенной загрузки частичных представлений. Маркап добавляется автоматически при первом рендеринге страницы, чтобы динамически загрузить содержимое.

```twig
{% ajaxPartial 'posts' lazy %}
```

Подробнее см. в статье [Twig-тег AJAX Partial](../../markup/tag/ajax-partial.md).

#### См. также

::: also
* [Twig-тег AJAX Partial](../../markup/tag/ajax-partial.md)
:::
