---
subtitle: Колонка списка
shortname: Selectable
---
# Колонка Selectable

`selectable` — берёт значение колонки и сопоставляет его с вариантом из доступных значений записи. Например, если доступен массив и значение равно `open`, в колонке отобразится **Open**.

```php
['open' => 'Open', 'closed' => 'Closed']
```

Доступные варианты задаются по правилам [dropdown options](../define-options.md).

```yaml
status:
    label: Status
    type: selectable
```

Поддерживаются следующие свойства.

Property | Description
------------- | -------------
**options** | доступные варианты в виде массива.
**optionsMethod** | получить варианты из метода модели или статического метода, например `Class::method`.
**optionsPreset** | взять варианты из [предопределённого набора](../define-options.md).

Свойство `options` позволяет явно задать варианты.

```yaml
status:
    label: Status
    type: selectable
    options:
        pending: Pending
        active: Active
```

Свойство `optionsPreset` позволяет использовать значения из [определения набора вариантов](../define-options.md).

```yaml
icon:
    label: Icon
    type: selectable
    optionsPreset: phosphorIcons
```

#### См. также

::: also
* [Определение options](../define-options.md)
* [Поле Dropdown](../form/field-dropdown.md)
:::
