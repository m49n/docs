---
subtitle: Фильтр Twig
---
# |raw

Переменные вывода в October CMS экранируются автоматически. Фильтр `|raw` помечает значение как «безопасное» и не будет экранирован, если `raw` — последний применённый фильтр.

```twig
{# This variable won't be escaped #}
{{ variable|raw }}
```

Будьте осторожны при использовании фильтра `raw` внутри выражений:

```twig
{% set hello = '<strong>Hello</strong>' %}
{% set hola = '<strong>Hola</strong>' %}

{{ false ? '<strong>Hola</strong>' : hello|raw }}

{# The above will not render the same as #}
{{ false ? hola : hello|raw }}

{# But renders the same as #}
{{ (false ? hola : hello)|raw }}
```
