---
subtitle: Тип инспектора
shortname: Set
---
# Тип инспектора Set

Тип инспектора `set` используется для множественного выбора из заранее определённых вариантов. Он поддерживает те же способы задания вариантов, что и [тип dropdown](./type-dropdown.md).

```php
public function defineProperties()
{
    return [
        'units' => [
            'title' => 'Select Muitple Units',
            'type' => 'set',
            'items' => [
                'metric' => 'Metric',
                'imperial' => 'Imperial'
            ]
        ]
    ];
}
```

Сгенерированное значение — массив выбранных вариантов, например:

```json
"units": ["metric", "imperial"]
```

Чаще всего применяются и поддерживаются следующие [параметры конфигурации](../inspector-types.md).

Property | Description
------------- | -------------
**title** | заголовок свойства.
**description** | краткое описание свойства, необязательно.
**items** | массив доступных элементов с ключами и значениями, можно не указывать при определении метода `get*PropertyName*Options`.
**default** | массив ключей, выбранных по умолчанию.

Если задано значение `default`, оно должно быть массивом с ключами элементов, выбранных по умолчанию.

```php
public function defineProperties()
{
    return [
        'context' => [
            'title' => 'Context',
            'type' => 'set',
            'items' => [
                'create' => 'Create',
                'update' => 'Update',
                'preview' => 'Preview'
            ],
            'default' => ['create', 'update']
        ]
    ];
}
```

Чтобы задать `items` динамически, создайте в модели метод `get*PropertyName*Options`.

```php
public function getContextOptions()
{
    return ContextModel::pluck('name', 'code')->all();
}
```

#### См. также

::: also
* [Тип инспектора Dropdown](./type-dropdown.md)
:::
