---
subtitle: Свойство Twig
---
# this.request

Предоставляет доступ к текущему объекту запроса, включая метод запроса.

## this.request.method

Получить текущий метод запроса можно через `this.request.method`; возвращается HTTP-глагол в верхнем регистре.

```twig
{% if this.request.method == 'GET' %}
    <!-- Выполнить логику GET -->
{% elseif this.request.method == 'POST' %}
    <!-- Выполнить логику POST -->
{% endif %}
```

## this.request.ajax

Чтобы проверить, использует ли текущий запрос AJAX-заголовок, обращайтесь к свойству `this.request.ajax`.

```twig
{% if this.request.ajax %}
    Запрос отправлен через AJAX
{% endif %}
```

## this.request.pjax

Чтобы проверить, был ли запрос выполнен через [Turbo Router](../../ajax/turbo-router.md), используйте свойство `this.request.pjax`.

```twig
{% if this.request.pjax %}
    Страница загружена через PJAX
{% endif %}
```

## this.request.pjaxCached

Свойство `this.request.pjaxCached` позволяет также проверить, был ли запрос [Turbo Router](../../ajax/turbo-router.md) предварительно обслужен из снимка.

```twig
{% if this.request.pjaxCached %}
    Страница загружена через PJAX with a snapshot
{% endif %}
```
