---
subtitle: Тип инспектора
shortname: Autocomplete
---
# Тип инспектора Autocomplete

Тип инспектора `autocomplete` работает так же, как редактор `string`, но добавляет автодополнение. Доступные варианты можно задать статически через параметр `options` или загрузить динамически.

```php
public function defineProperties()
{
    return [
        'condition' => [
            'title' => 'Condition',
            'type' => 'autocomplete',
            'options' => ['start' => 'Start', 'end' => 'End']
        ]
    ];
}
```

Сгенерированное значение представляет собой строку с выбранным вариантом, например:

```json
"condition": "start"
```

Чаще всего применяются следующие [параметры конфигурации](../inspector-types.md).

Property | Description
------------- | -------------
**title** | заголовок свойства.
**description** | краткое описание свойства, необязательно.
**default** | строка по умолчанию, необязательно.
**options** | массив вариантов для выпадающих свойств, можно не указывать при определении метода `get*PropertyName*Options`.
**showExternalParam** | не поддерживается и должен иметь значение `false`.

::: warning
Этот тип не поддерживает редактор внешнего параметра, задаваемый свойством `showExternalParam`.
:::

## Динамические варианты

Тип инспектора `autocomplete` поддерживает те же способы задания вариантов, что и [тип dropdown](./type-dropdown.md).

```php
public function defineProperties()
{
    return [
        'sortColumn' => [
            'title' => 'Sort by Column',
            'type' => 'autocomplete',
            // ...
        ],
    ];
}

public function getSortColumnOptions()
{
    return [
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ];
}
```

#### См. также

::: also
* [Тип инспектора Dropdown](./type-dropdown.md)
:::
