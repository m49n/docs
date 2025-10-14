---
subtitle: Тип инспектора
shortname: Object
---
# Тип инспектора Object

Тип инспектора `object` позволяет задавать объект со свойствами, которые пользователь может редактировать. Свойства объекта задаются атрибутом `properties`. Значением атрибута служит массив с той же структурой, что и массив свойств инспектора.

Пример ниже создаёт объект с тремя свойствами: два поля отображаются как текстовые, третье — как выпадающий список.

```php
public function defineProperties()
{
    return [
        'address' => [
            'title' => 'Address',
            'type' => 'object',
            'properties' => [
                'streetAddress' => [
                    'title' => 'Street Address',
                    'type' => 'string'
                ],
                'city' => [
                    'title' => 'City',
                    'type' => 'string'
                ],
                'country' => [
                    'title' => 'Country',
                    'type' => 'dropdown',
                    'options' => [
                        'us' => 'US',
                        'ca' => 'Canada'
                    ]
                ]
            ],
        ]
    ];
}
```

Сгенерированное значение — объект, например:

```json
"address": {
    "streetAddress": "321-210 Second ave",
    "city": "Springfield",
    "country": "us"
}
```

Чаще всего применяются и поддерживаются следующие [параметры конфигурации](../inspector-types.md).

Property | Description
------------- | -------------
**title** | заголовок свойства.
**description** | краткое описание свойства, необязательно.
**properties** | массив вложенных определений свойств.
**default** | массив значений по умолчанию с ключами и значениями.
**ignoreIfPropertyEmpty** | перечислите свойства, при пустом значении которых объект исключается из результата.

::: warning
Этот тип не поддерживает редактор внешнего параметра, задаваемый свойством `showExternalParam`.
:::

Свойства объекта могут иметь любой тип, поддерживаемый инспектором, включая вложенные объекты. Можно полностью исключить объект из результата инспектора, если одно из его полей пустое. Поле задаётся параметром `ignoreIfPropertyEmpty`. Например:

```php
public function defineProperties()
{
    return [
        'address' => [
            'title' => 'Address',
            'type' => 'object',
            'ignoreIfPropertyEmpty' => 'title',
            'properties' => [
                'streetAddress' => [
                    'title' => 'Street Address',
                    'type' => 'string'
                ],
                'city' => [
                    'title' => 'City',
                    'type' => 'string'
                ]
            ],
        ]
    ];
}
```

В этом примере, если улица не задана, объект (`"address"`) полностью исключается из результата инспектора. Если для других полей объекта заданы правила валидации, а обязательное поле пустое, эти правила будут проигнорированы.

Значение `default`, если указано, должно быть объектом с теми же свойствами, что определены в параметре `properties`.

```php
public function defineProperties()
{
    return [
        'address' => [
            'title' => 'Address',
            'type' => 'object',
            'properties' => [/*...*/],
            'default' => [
                'streetAddress' => '321-210 Second ave',
                'city' => 'Springfield',
                'country' => 'us'
            ]
        ]
    ];
}
```
