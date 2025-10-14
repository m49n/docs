---
subtitle: Поле Content
shortname: Mixin
---
# Поле Mixin

`mixin` — включает другой набор полей.

```yaml
_include1:
    type: mixin
    source: <uuid|handle>
```

::: tip
При использовании миксина рекомендуется добавлять к названию поля префикс в виде символа подчёркивания (\_), чтобы их было легче заметить.
:::

Чтобы подключить миксин, можно указать `source` как дескриптор blueprint.

```yaml
_location_fields:
    type: mixin
    source: Fields\Location
```

Для более надёжной ссылки можно также указать UUID.

```yaml
_blog_fields:
    type: mixin
    source: 6d6a5efa-3ce7-4b9d-bddc-ac48867552cb
```

Подробности о создании миксинов см. в статье [Blueprint-схемы Tailor](../../cms/tailor/blueprints.md).


#### См. также

::: also
* [Blueprint-схемы Tailor](../../cms/tailor/blueprints.md)
:::
