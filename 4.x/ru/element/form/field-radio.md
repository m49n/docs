---
subtitle: Form Field
shortname: Radio List
---
# Поле списка переключателей (radio)

Поле `radio` выводит список переключателей, в котором можно выбрать только один вариант. Поля переключателей поддерживают те же способы задания опций, что и [тип поля dropdown](./field-dropdown.md).

```yaml
security_level:
    type: radio
    label: Access Level
    options:
        all: All
        registered: Registered only
        guests: Guests only
```

Ниже перечислены часто используемые [свойства поля](../form-fields.md).

Property | Description
------------- | -------------
**label** | имя, отображаемое пользователю.
**default** | значение по умолчанию для новых записей.
**options** | доступные варианты списка переключателей в виде массива.
**optionsMethod** | получение опций из метода модели или статического метода, например `Class::method`.
**cssClass** | используется для вывода опций в одну строку.
**inlineOptions** | отображает варианты горизонтально, а не в столбик.

Свойство `default` задаёт значение по умолчанию, где значение — это ключ варианта.

```yaml
security_level:
    type: radio
    label: Access Level
    default: guests
```

Помимо простых массивов, список переключателей поддерживает вторичное описание как часть свойства `options`.

```yaml
security_level:
    type: radio
    label: Access Level
    options:
        all: [All, Guests and customers will be able to access this page.]
        registered: [Registered only, Only logged in member will be able to access this page.]
        guests: [Guests only, Only guest users will be able to access this page.]
```

Чтобы отображать варианты горизонтально, присвойте свойству `inlineOptions` значение `true`.

```yaml
security_level:
    type: radio
    label: Access Level
    inlineOptions: true
```

## Динамические опции

Списки переключателей поддерживают те же способы задания опций, что и [тип поля dropdown](./field-dropdown.md).

В дополнение к этим определениям метод для списка переключателей может возвращать простой массив **key => value** или массив массивов для описаний: **key => [label, description]**.

```php
public function listAccessLevels($fieldName, $value, $formData)
{
    return [
        'all' => ['All', 'Guests and customers will be able to access this page.'],
        // ...
    ];
}
```

#### См. также

::: also
* [Поле dropdown](./field-dropdown.md)
:::
