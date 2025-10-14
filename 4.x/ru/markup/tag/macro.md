---
subtitle: Тег Twig
---
# {% macro %}

Тег `{% macro %}` позволяет объявлять собственные функции в шаблонах, как и в обычных языках программирования.

```twig
{% macro input() %}
    ...
{% endmacro %}
```

Для удобства можно указать имя макроса после закрывающего тега:

```twig
{% macro input() %}
    ...
{% endmacro input %}
```

В следующем примере определяется функция `input()`, принимающая четыре аргумента. Значения доступны как переменные внутри разметки.

```twig
{% macro input(name, value, type, size) %}
    <input
        type="{{ type|default('text') }}"
        name="{{ name }}"
        value="{{ value|e }}"
        size="{{ size|default(20) }}" />
{% endmacro %}
```

> **Примечание.** Аргументы макросов не имеют значений по умолчанию и всегда считаются необязательными.

## Вызов макросов

Перед использованием макрос необходимо «импортировать» с помощью тега `{% import %}`. Если макрос определён в том же шаблоне, можно использовать специальную переменную `_self`.

```twig
{% import _self as form %}
```

Теперь функции макроса доступны через переменную `form` и вызываются как обычные функции.

```twig
<p>{{ form.input('username') }}</p>
<p>{{ form.input('password', null, 'password') }}</p>
```

Макросы можно разместить в [частичном представлении темы](../../cms/themes/partials.md) и импортировать по имени. Например, чтобы подключить макросы из **macros/form.htm**, передай имя файла в кавычках после тега `import`.

```twig
{% import 'macros/form' as form %}
```

Также можно импортировать макросы из [системного view-файла](../../extend/services/response-view.md). Чтобы загрузить их из **plugins/acme/blog/views/macros.htm**, передай подсказку пути.

```twig
{% import 'acme.blog::macros' as form %}
```

## Вложенные макросы

Если нужно вызвать макрос внутри другого макроса того же шаблона, импортируй его локально.

```twig
{% macro input(name, value, type, size) %}
    <input
        type="{{ type|default('text') }}"
        name="{{ name }}"
        value="{{ value|e }}"
        size="{{ size|default(20) }}" />
{% endmacro %}

{% macro wrapped_input(name, value, type, size) %}
    {% import _self as form %}

    <div class="field">
        {{ form.input(name, value, type, size) }}
    </div>
{% endmacro %}
```

## Переменные контекста

Макросы не имеют доступа к текущим переменным страницы.

```twig
<!-- October CMS -->
{{ site_name }}

{% macro myFunction() %}
    <!-- NULL -->
    {{ site_name }}
{% endmacro %}
```

Можно передать переменные в функцию через специальную переменную `_context`.

```twig
{% macro myFunction(vars) %}
    {{ vars.site_name }}
{% endmacro %}

{% import _self as form %}

<!-- October CMS -->
{{ form.myFunction(_context) }}
```
