---
subtitle: Тег Twig
---
# {% verbatim %}

Тег `{% verbatim %}` помечает целые фрагменты как сырой текст, который не должен парситься.

```twig
{% verbatim %}<p>Hello, {{ name }}</p>{% endverbatim %}
```

В браузере вывод будет точным:

```twig
<p>Hello, {{ name }}</p>
```

Например, AngularJS использует такой же синтаксис шаблонов, поэтому можно решить, какие переменные обрабатывает каждая сторона.

```twig
<p>Hello {{ name }}, this is parsed by Twig</p>

{% verbatim %}
    <p>Hello {{ name }}, this is parsed by AngularJS</p>
{% endverbatim %}
```
