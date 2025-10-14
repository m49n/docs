---
subtitle: Тип инспектора
shortname: Dictionary
---
# Тип инспектора Dictionary

Тип инспектора `dictionary` позволяет создавать пары ключ‑значение через простой интерфейс в виде таблицы из двух колонок. Значение `default`, если указано, должно быть объектом с парами ключ‑значение.

```php
public function defineProperties()
{
    return [
        'options' => [
            'title' => 'Options',
            'type' => 'dictionary',
            'default' => ['option1' => 'Option 1'],
        ]
    ];
}
```

Сгенерированное значение представляет собой объект, например:

```json
"options": {"option1": "Option 1", "option2": "Option 2"}
```

Чаще всего применяются следующие [параметры конфигурации](../inspector-types.md).

Property | Description
------------- | -------------
**title** | заголовок свойства.
**description** | краткое описание свойства, необязательно.
**default** | массив ключей и значений по умолчанию, необязательно.

## Дополнительная валидация

Редактор `dictionary` поддерживает валидацию для всего набора (`required` и `length`) и отдельно для ключей и значений. Подробности см. в [описании валидации](../inspector-types.md). Свойства `validationKey` и `validationValue` задают проверку для ключей и значений соответственно, например:

```php
public function defineProperties()
{
    return [
        'options' => [
            'title' => 'Options',
            'type' => 'dictionary',
            'validation' => [
                'required' => [
                    'message' => 'Please create options'
                ],
                'length' => [
                    'min' => [
                        'value' => 2,
                        'message' => 'Create at least two options.'
                    ]
                ]
            ],
            'validationKey' => [
                'regex' => [
                    'pattern' => '^[a-z]+$',
                    'message' => 'Keys can contain only lowercase Latin letters'
                ]
            ],
            'validationValue' => [
                'regex' => [
                    'pattern' => '^[a-zA-Z0-9]+$',
                    'message' => 'Values can contain only Latin letters and digits'
                ]
            ]
        ]
    ];
}
```
