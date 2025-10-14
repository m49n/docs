---
subtitle: Узнайте, как определять свойства компонентов.
---
# Типы инспектора (Inspector Types)

Типы инспектора — это типы свойств, которые используют компоненты CMS. Они применяются в следующем месте:

- [Классы компонентов CMS](../extend/cms-components.md)

Все типы инспектора идентифицируются по своему свойству **type**.

```php
public function defineProperties()
{
    return [
        'maxItems' => [
            'title' => 'Max Items',
            'type' => 'string'
        ]
    ];
}
```

## Доступные типы

Доступны следующие типы инспектора:

<div class="content-list-p" markdown="1">

[String](./inspector/type-string.md)
[String List](./inspector/type-stringlist.md)
[Text](./inspector/type-text.md)
[Autocomplete](./inspector/type-autocomplete.md)
[Checkbox](./inspector/type-checkbox.md)
[Dropdown](./inspector/type-dropdown.md)
[Dictionary](./inspector/type-dictionary.md)
[Object](./inspector/type-object.md)
[Object List](./inspector/type-objectlist.md)
[Set](./inspector/type-set.md)

</div>

## Доступные параметры

Параметры свойств задаются массивом со следующими ключами.

Key | Description
------------- | -------------
**title** | обязательный параметр, заголовок свойства; используется инспектором компонента в бэкенде CMS.
**description** | обязательный параметр, описание свойства; используется инспектором компонента в бэкенде CMS.
**default** | необязательный параметр, значение свойства по умолчанию при добавлении компонента на страницу или макет в бэкенде CMS.
**type** | определяет тип свойства, который задаёт способ отображения свойства в инспекторе.
**validation** | необязательный параметр, задаёт правила валидации значения свойства (см. ниже).
**placeholder** | необязательное значение-заполнитель для строковых и выпадающих свойств.
**options** | необязательный массив вариантов для выпадающих свойств.
**optionsMethod** | имя метода в классе компонента, который возвращает варианты.
**depends** | массив имён свойств, от которых зависит выпадающее свойство. Подробности см. в [типе dropdown](./inspector/type-dropdown.md).
**group** | необязательное имя группы. Группы создают секции в инспекторе, упрощая работу. Используйте одинаковое имя группы для нескольких свойств, чтобы объединить их.
**showExternalParam** | определяет видимость редактора внешнего параметра для свойства в инспекторе. Значение по умолчанию: `true`.
**ignoreIfDefault** | установите `true`, чтобы исключать значение из массива, если оно совпадает со значением по умолчанию. Значение по умолчанию: `false`.
**ignoreIfEmpty** | установите `true`, чтобы исключать значение из массива, если оно пустое. Значение по умолчанию: `false`.
**sortOrder** | задаёт позицию свойства в списке в виде целого числа.

## Правила валидации

Типы инспектора поддерживают несколько правил валидации, которые можно применить к свойствам. Правила можно задавать как для верхнего уровня, так и для внутренних определений свойств в редакторах объектов и списков объектов.

```php
public function defineProperties()
{
    return [
        'name' => [
            'title' => 'Name',
            'type' => 'string',
            'validation' => [
                'required' => [
                    'message' => 'The Name field is required'
                ],
                'regex' => [
                    'message' => 'The Name field can contain only Latin letters.',
                    'pattern' => '^[a-zA-Z]+$'
                ]
            ]
        ]
    ];
}
```

Ключ в объекте `validation` соответствует валидатору (см. ниже). Валидаторы настраиваются объектами, набор свойств зависит от валидатора. Свойство `message` общее для всех валидаторов.

### Валидатор required

Валидатор `required` проверяет, что значение не пустое. Его можно использовать с любым редактором, включая сложные (set, dictionary, object list и т. д.). Пример:

```php
public function defineProperties()
{
    return [
        'name' => [
            'title' => 'Name',
            'type' => 'string',
            'validation' => [
                'required' => [
                    'message' => 'The Name field is required'
                ]
            ]
        ]
    ];
}
```

### Валидатор regex

Валидатор `regex` проверяет строковые значения по регулярному выражению. Его можно использовать только со строковыми редакторами. Пример:

```php
public function defineProperties()
{
    return [
        'name' => [
            'title' => 'Name',
            'type' => 'string',
            'validation' => [
                'regex' => [
                    'message' => 'The Name field can contain only Latin letters',
                    'pattern' => '^[a-z]+$',
                    'modifiers' => 'i'
                ]
            ]
        ]
    ];
}
```

Регулярное выражение задаётся обязательным параметром `pattern`. Параметр `modifiers` необязателен и задаёт модификаторы регулярного выражения.

### Валидатор integer

Валидатор `integer` проверяет, что значение — целое число, и при необходимости контролирует попадание в заданный диапазон. Его можно использовать только со строковыми редакторами. Пример:

```php
public function defineProperties()
{
    return [
        'numOfColumns' => [
            'title' => 'Number of Columns',
            'type' => 'string',
            'validation' => [
                'integer' => [
                    'message' => 'The Number of Columns field should contain an integer value',
                    'allowNegative' => true,
                    'min' => [
                        'value' => -10,
                        'message' => 'The number of columns should not be less than -10.'
                    ],
                    'max' => [
                        'value' => 10,
                        'message' => 'The number of columns should not be greater than 10.'
                    ]
                ]
            ]
        ]
    ];
}
```

Поддерживаемые параметры:

* `allowNegative` — необязательный параметр, определяет, разрешены ли отрицательные значения. По умолчанию отрицательные значения запрещены.
* `min` — необязательный объект, задаёт минимальное значение и сообщение об ошибке. Поля объекта:
    * `value` — минимально допустимое значение.
    * `message` — необязательное сообщение об ошибке.
* `max` — необязательный объект, задаёт максимальное значение и сообщение об ошибке. Поля объекта:
    * `value` — максимально допустимое значение.
    * `message` — необязательное сообщение об ошибке.

### Валидатор float

Валидатор `float` проверяет, что значение — число с плавающей точкой. Параметры валидатора совпадают с параметрами валидатора **integer**, описанного выше. Пример:

```php
public function defineProperties()
{
    return [
        'amount' => [
            'title' => 'Amount',
            'type' => 'string',
            'validation' => [
                'float' => [
                    'message' => 'The Amount field should contain a positive floating point value'
                ]
            ]
        ]
    ];
}
```

Допустимые форматы чисел с плавающей точкой:

* 10
* 10.302
* -10 (если `allowNegative` имеет значение `true`)
* -10.84 (если `allowNegative` имеет значение `true`)

### Валидатор length

Валидатор `length` проверяет, что строка, массив или объект не короче и не длиннее заданных значений. Его можно применять к редакторам string, text, set, string list, dictionary и object list. В редакторах с множественными значениями (set, string list, dictionary и object list) он проверяет количество элементов.

::: tip
Валидатор `length` не проверяет пустые значения. Например, если он применён к редактору set и множество пустое, проверка пройдёт независимо от значений `min` и `max`. Используйте валидатор `required` вместе с `length`, чтобы убедиться, что значение не пустое перед проверкой длины.
:::

```php
public function defineProperties()
{
    return [
        'name' => [
            'title' => 'Name',
            'type' => 'string',
            'validation' => [
                'length' => [
                    'min' => [
                        'value' => 2,
                        'message' => 'The name should not be shorter than two letters.'
                    ],
                    'max' => [
                        'value' => 10,
                        'message' => 'The name should not be longer than 10 letters.'
                    ]
                ]
            ]
        ]
    ];
}
```

Поддерживаемые параметры:

* `min` — необязательный объект, задаёт минимальную длину и сообщение об ошибке. Поля объекта:
    * `value` — минимально допустимое значение.
    * `message` — необязательное сообщение об ошибке.
* `max` — необязательный объект, задаёт максимальную длину и сообщение об ошибке. Поля объекта:
    * `value` — максимально допустимое значение.
    * `message` — необязательное сообщение об ошибке.
