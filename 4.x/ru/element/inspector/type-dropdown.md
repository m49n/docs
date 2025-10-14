---
subtitle: Тип инспектора
shortname: Dropdown
---
# Тип инспектора Dropdown

Тип инспектора `dropdown` используется для выбора одного значения из набора заранее определённых вариантов. Список вариантов для свойств dropdown и set может быть статическим или динамическим. Статические варианты задаются элементом `options` в определении свойства.

```php
public function defineProperties()
{
    return [
        'unit' => [
            'title' => 'Unit',
            'type' => 'dropdown',
            'default' => 'imperial',
            'placeholder' => 'Select units',
            'options' => ['metric' => 'Metric', 'imperial' => 'Imperial']
        ]
    ];
}
```

Сгенерированное значение — строка с выбранным вариантом, например:

```json
"unit": "metric"
```

Чаще всего применяются следующие [параметры конфигурации](../inspector-types.md).

Property | Description
------------- | -------------
**title** | заголовок свойства.
**description** | краткое описание свойства, необязательно.
**default** | строка по умолчанию, необязательно.
**options** | массив вариантов для выпадающих свойств, можно не указывать при определении метода `get*PropertyName*Options`.

## Динамические варианты

Список вариантов может загружаться динамически с сервера при открытии инспектора. Если параметр `options` в определении выпадающего свойства отсутствует, список считается динамическим. Класс компонента должен определить метод, возвращающий список вариантов. Имя метода должно соответствовать шаблону `get*PropertyName*Options`, где **Property** — имя свойства (например, `getCountryOptions`). Метод возвращает массив с ключами значений и метками. Пример определения динамического списка.

```php
public function defineProperties()
{
    return [
        'country' => [
            'title' => 'Country',
            'type' => 'dropdown',
            'default' => 'us'
        ]
    ];
}

public function getCountryOptions()
{
    return ['us' => 'United states', 'ca' => 'Canada'];
}
```

Динамические выпадающие списки и списки set могут зависеть от других свойств. Например, список штатов может зависеть от выбранной страны. Зависимости объявляются параметром `depends`. В следующем примере определены два динамических свойства, и список штатов зависит от страны.

```php
public function defineProperties()
{
    return [
        'country' => [
            'title' => 'Country',
            'type' => 'dropdown',
            'default' => 'us'
        ],
        'state' => [
            'title' => 'State',
            'type' => 'dropdown',
            'default' => 'dc',
            'depends' => ['country'],
            'placeholder' => 'Select a state'
        ]
    ];
}
```

Чтобы загрузить список штатов, необходимо знать, какая страна выбрана в инспекторе. Инспектор передаёт все значения свойств в обработчик `getPropertyOptions`, поэтому можно сделать следующее.

```php
public function getStateOptions()
{
    // Получаем значение свойства country из POST
    $countryCode = post('country');

    $states = [
        'ca' => ['ab' => 'Alberta', 'bc' => 'British columbia'],
        'us' => ['al' => 'Alabama', 'ak' => 'Alaska']
    ];

    return $states[$countryCode];
}
```

## Свойства со списком страниц

Иногда компоненту нужно создать ссылку на страницу сайта. Например, список записей блога содержит ссылки на страницу с подробным описанием записи. В этом случае компонент должен знать имя файла страницы (чтобы использовать [Twig‑фильтр page](../../markup/filter/page.md)). October включает помощник для создания динамических списков страниц. В следующем примере определяется свойство `postPage`, которое отображает список страниц:

```php
public function defineProperties()
{
    return [
        'postPage' => [
            'title' => 'Post page',
            'type' => 'dropdown',
            'default' => 'blog/post'
        ]
    ];
}

public function getPostPageOptions()
{
    return Page::sortBy('baseFileName')->lists('baseFileName', 'baseFileName');
}
```
