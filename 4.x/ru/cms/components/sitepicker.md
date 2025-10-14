---
subtitle: Инструменты для работы с несколькими конфигурациями сайтов
---
# Переключатель сайтов

Компонент (component) `sitePicker` предоставляет инструменты для работы с [конфигурацией Multisite](../resources/multisite.md) сайта. Оптимально размещать его в шаблоне страницы или макета (layout).

## Базовое использование

::: cmstemplate
```ini
[sitePicker]
```
```twig
{% set availableSites = sitePicker.sites %}
```
:::

Ниже приведён пример выпадающего списка, который переключает сайты. Он используется совместно со [свойством Twig `this.site`](../../markup/property/this-site.md).

```twig
<select class="form-control" onchange="window.location.assign(this.value)">
    {% for site in sitePicker.sites %}
        <option value="{{ site.url }}" {{ this.site.code == site.code ? 'selected' }}>
            {{ site.name }}
        </option>
    {% endfor %}
</select>
```

Другой пример — генерация альтернативных ссылок на страницы через метатеги.

```twig
{% for site in sitePicker.sites %}
    <link rel="alternate" hreflang="{{ site.locale }}" href="{{ site.url }}" />
{% endfor %}
```

## Загрузка сайтов для другой страницы

По умолчанию свойство `sites` возвращает сайты, настроенные для текущей страницы, а `url` разрешается относительно текущей страницы. Функция `pageSites()` позволяет получить сайты для другой страницы; первым аргументом передаётся имя страницы CMS.

В примере ниже для каждого сайта значение `url` указывает на страницу CMS **pages/blog/index.htm**. Если страница не найдена, функция вернёт пустой массив.

::: cmstemplate
```ini
[sitePicker]
```
```twig
{% set otherSites = sitePicker.pageSites('blog/index') %}
```
:::

## Перевод параметров URL

По умолчанию компонент `sitePicker` не учитывает параметры моделей в URL, такие как slug страницы или идентификаторы. Для подстановки переведённых параметров используется [глобальное событие](../../extend/services/event.md) `cms.sitePicker.overrideParams`. Разместить обработчик события удобно в методах `init` или `onRun` [класса CMS-компонента](../../extend/cms-components.md).

Например, если модель использует [трейта `Multisite`](../../extend/database/traits.md), метод `newOtherSiteQuery` поможет найти модель для выбранного сайта и изменить параметры URL.

```php
$myModel = MyModel::find(1);
$otherModels = $myModel->newOtherSiteQuery()->get();

Event::listen('cms.sitePicker.overrideParams', function($page, $params, $currentSite, $proposedSite) use ($otherModels) {
    $otherModel = $otherModels->where('site_id', $proposedSite->id)->first();
    if ($otherModel) {
        $params['id'] = $otherModel->id;
        $params['slug'] = $otherModel->slug;
        $params['fullslug'] = $otherModel->fullslug;
    }
    return $params;
});
```

#### См. также

::: also
* [Multisite](../resources/multisite.md)
:::
