---
subtitle: Тег Twig
---
# {% ajaxPartial %}

::: aside
Этот тег расширяет [тег Twig `{% partial %}`](./partial.md).
:::

Тег `{% ajaxPartial %}` выводит содержимое частичного представления на странице и поддерживает [AJAX-обработчики](../../ajax/introduction.md), самобновление и краткий синтаксис обновления. Такой частичный шаблон называют самобновляющимся.

```twig
{% ajaxPartial "contact-form" %}
```

При первом рендеринге содержимое частичного представления оборачивается в HTML-тег. Повторные обновления через AJAX выводят только содержимое без обёртки.

```html
<div data-ajax-partial="contact-form">
    ... Contents go here ...
</div>
```

## Краткий синтаксис обновления

При использовании AJAX-частичного представления больше не нужно указывать селектор для обновления. Просто передай `true` в [API data-атрибутов](../../ajax/attributes-api.md) для атрибута `data-request-update`.

```html
<button
    data-request="onRefresh"
    data-request-update="{ contact-form: true }">
    Refresh
</button>
```

Можно обновить частичное представление из него самого, используя `_self` в качестве имени.

```html
<button
    data-request="onRefresh"
    data-request-update="{ _self: true }">
    Refresh
</button>
```

Также допускается указать символ `^` для добавления содержимого в начало контейнера и `@` для добавления в конец вместо замены.

```html
<button
    data-request="onRefresh"
    data-request-update="{ _self: '@' }">
    Append
</button>
```

## Отложенная загрузка частичных представлений

Тег `{% ajaxPartial %}` поддерживает атрибут `lazy`, который откладывает рендеринг до загрузки страницы. В примере ниже частичное представление **posts** обновится после загрузки страницы.

```twig
{% ajaxPartial 'posts' lazy %}
```

Атрибуты `lazy body` позволяют определить начальное содержимое до загрузки; после него следует тег `{% endpartial %}`.

```twig
{% ajaxPartial 'posts' lazy body %}
    <p>Loading posts...</p>
{% endpartial %}
```

Важно: тег `{% ajaxPartial lazy %}` не рендерит частичное представление сразу. Вместо этого он выводит начальную разметку с data-атрибутом `data-auto-submit`, используемым [запросами опроса](../../ajax/features/polling.md). Этот атрибут инициирует AJAX-запрос после загрузки страницы. В последующих обновлениях частичного представления атрибут не добавляется, чтобы избежать бесконечного цикла.

Так выглядит вывод при первом открытии страницы:

```html
<div
    data-request="onAjax"
    data-request-update="{ _self: true }"
    data-auto-submit>
    <p>Loading posts...</p>
</div>
```

::: tip
Не вставляй приведённую выше разметку в частичное представление. Тег `{% ajaxPartial lazy %}` добавит её автоматически.
:::

## Вызов AJAX-обработчиков

При вызове AJAX-обработчика внутри AJAX-частичного представления запускается «захватывающий» жизненный цикл (см. ниже), который позволяет использовать обработчики в запрошенных частичных представлениях.

Ниже показано, как отправить простую форму обратной связи через самобновляющийся partial.

::: cmstemplate
```ini
description = "Self Updating Partial"
```
```php
<?
function onSubmitContactForm()
{
    $this['submitted'] = true;
}
?>
```
```twig
{% if submitted %}
    <p>Thank you for contacting us!</p>
{% endif %}

<button
    data-request="onSubmitContactForm"
    data-request-update="{ _self: true }">
    Submit
</button>
```
:::

Частичные представления, использующие [компоненты CMS](../../cms/themes/components.md), также получают доступ к их AJAX-обработчикам.

::: cmstemplate
```ini
[contactForm]
```
```html
<button
    data-request="contactForm::onSubmit"
    data-request-update="{ _self: true }">
    Submit
</button>
```
:::

### Жизненный цикл захвата

При вызове обработчика из AJAX-частичного представления запускается другой жизненный цикл — capture lifecycle. Он рендерит всю страницу, но отправляет содержимое «в никуда». Таким образом страница полностью инициализируется, включая все частичные представления и AJAX-обработчики.

Поскольку это полный рендер страницы, метод компонента `onRun` может выполняться и при использовании обработчика AJAX-частичного представления. Можно воспользоваться [вспомогательным методом](../../extend/services/request-input.md) `Request::ajax()`, чтобы определить, вызван ли обработчик AJAX-запросом.
