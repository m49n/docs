---
subtitle: Колонка списка
shortname: Linkage
---
# Колонка Linkage

Колонка `linkage` отображает ссылку на указанную страницу.

```yaml
website:
    label: Website
    type: linkage
```

Поддерживаются следующие свойства.

Property | Description
------------- | -------------
**linkText** | текст ссылки, необязательно.
**linkUrl** | URL, который следует использовать вместо значения записи.
**attributes** | массив HTML‑атрибутов для элемента `<a>`.

Используйте свойство `attributes`, чтобы добавить пользовательские HTML‑атрибуты.

```yaml
website:
    label: Website
    type: linkage
    attributes:
        target: _blank
```

::: tip
Колонка `linkage` автоматически обрабатывает значения, возвращаемые [виджетом Page Finder](../form/widget-pagefinder.md).
:::

Свойства `linkUrl` и `linkText` позволяют явно задать URL, который может быть URI бэкенда или абсолютным адресом. Атрибуты из записи также будут автоматически подставлены.

```yaml
open_link:
    label: View
    type: linkage
    linkText: View Dashboard
    linkUrl: backend/index/:code/:id
```

## Пользовательский текст ссылки

По умолчанию значением служит URL. Если нужно изменить текст, верните из модели массив.

```php
['https://octobercms.com', 'October CMS']
```

В модели можно использовать модификатор атрибута, чтобы возвращать такие значения. Следующий пример создаёт атрибут `website_link`.

```php
public function getWebsiteLinkAttribute()
{
    return [$this->url, $this->name];
}
```

Чтобы сохранить возможность сортировки и поиска по значению из базы, используйте `displayFrom`. В примере ниже сортировка и поиск будут выполняться по атрибуту `website`, а отображение — через `website_link`.

```yaml
website:
    label: Website
    type: linkage
    displayFrom: website_link
```
