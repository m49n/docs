---
subtitle: Свойство Twig
---
# this.param

Текущие параметры URL доступны через `this.param`; возвращается массив PHP.

## Доступ к параметрам страницы

В этом примере показано, как получить параметр URL `tab` на странице.

::: cmstemplate
```ini
url = "/account/:tab"
```
```twig
{% if this.param.tab == 'details' %}

    <p>Здесь показаны все ваши данные</p>

{% elseif this.param.tab == 'history' %}

    <p>Вы просматриваете страницу из прошлого</p>

{% endif %}
```
:::

Если имя параметра совпадает с переменной, можно использовать синтаксис массива.

::: cmstemplate
```ini
url = "/account/:post_id"
```
```twig
{% set name = 'post_id' %}

<p>ID записи: {{ this.param[name] }}</p>
```
:::
