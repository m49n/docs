---
subtitle: Свойство Twig
---
# this.site

Активный сайт доступен через `this.site`; он возвращает объект `System\Models\SiteDefinition`, то есть [текущее определение сайта](../../cms/resources/multisite.md).

## Получение данных сайта

```twig
{{ this.site.id }}
{{ this.site.name }}
{{ this.site.code }}
{{ this.site.locale }}
{{ this.site.timezone }}
{{ this.site.theme }}
```

## Проверка активного сайта

```twig
{% if this.site.code === 'english' %}
    <h1>Отображается только для английской версии</h1>
{% endif %}
```

## Получение текущей локали

Атрибут `locale` возвращает текущую локаль, если она задана, или пустое значение, если локаль не указана.

```twig
<html lang="{{ this.site.locale }}">
```

Используйте атрибут `hard_locale`, чтобы всегда получать значение локали; в этом случае будет использована локаль по умолчанию, если текущая не задана.

```twig
<html lang="{{ this.site.hard_locale }}">
```
