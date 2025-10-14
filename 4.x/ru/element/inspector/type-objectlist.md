---
subtitle: Тип инспектора
shortname: Object List
---
# Тип инспектора Object List

Тип инспектора `objectList` позволяет пользователям создавать несколько объектов с заранее заданной структурой. Например, можно составить список людей, где у каждого есть имя и адрес.

Свойства объектов, которые можно создать редактором, определяются параметром `itemProperties`. Параметр должен содержать массив свойств, аналогичный массиву конфигурации инспектора. Ещё один обязательный параметр — `titleProperty`, он определяет свойство, используемое в инспекторе как заголовок.

Массив свойств в `itemProperties` поддерживает все типы свойств.

```php
public function defineProperties()
{
    return [
        'address' => [
            'title' => 'Address',
            'type' => 'objectList',
            'titleProperty' => 'fullName',
            'itemProperties' => [
                'fullName' => [
                    'title' => 'Full Name',
                    'type' => 'string'
                ],
                'address' => [
                    'title' => 'Address',
                    'type' => 'string'
                ]
            ]
        ]
    ];
}
```

По умолчанию результат — неассоциативный массив, например:

```json
"people": [
    {"fullName": "John Smith", "address": "Palo Alto"},
    {"fullName": "Bart Simpson", "address": "Springfield"}
]
```

Чаще всего применяются и поддерживаются следующие [параметры конфигурации](../inspector-types.md).

Property | Description
------------- | -------------
**title** | заголовок свойства.
**description** | краткое описание свойства, необязательно.
**keyProperty** | ключ свойства, используемый как заголовок; берётся из определения **itemProperties**.
**titleProperty** | имя свойства, используемого как заголовок; берётся из определения **itemProperties**.
**itemProperties** | массив вложенных определений свойств.

::: warning
Тип Object List не поддерживает значения по умолчанию.
:::

Если результат должен быть ассоциативным массивом (объектом), используйте параметр `keyProperty`. Его значение должно ссылаться на свойство, которое будет использоваться как ключ. Свойство для ключа может использовать только редакторы string или dropdown; значения должны быть уникальными и не пустыми.

```php
public function defineProperties()
{
    return [
        'address' => [
            'title' => 'Address',
            'type' => 'objectList',
            'titleProperty' => 'fullName',
            'keyProperty' => 'login',
            'itemProperties' => [
                'fullName' => [
                    'title' => 'Full Name',
                    'type' => 'string'
                ],
                'login' => [
                    'title' => 'Login',
                    'type' => 'string'
                ],
                'address' => [
                    'title' => 'Address',
                    'type' => 'string'
                ]
            ]
        ]
    ];
}
```

В этом примере свойство `login` используется как ключ в результате:

```json
"people": {
    "john": {"fullName": "John Smith", "address": "Palo Alto"},
    "bart": {"fullName": "Bart Simpson", "address": "Springfield"}
}
```
