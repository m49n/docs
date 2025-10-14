---
subtitle: Определение свойства options, используемого во многих конфигурациях.
---
# Определение options

Чаще всего встречаются свойства **options**, **optionsMethod** или **optionsPreset**. В этой статье подробно описано, как настраивать варианты выбора.

## Массивы options

Свойство `options` должно напрямую задавать варианты в определении в виде пар ключ‑значение, где значение и метка задаются независимо.

```yaml
options:
    draft: Draft
    published: Published
    archived: Archived
```

Ключами могут быть целые числа с собственной меткой.

```yaml
options:
    1: Simple
    2: Complex
```

Помимо простых массивов некоторые поля, например [списки переключателей](./form/field-radio.md), поддерживают задание описания как части значения `options`. В этом случае значение задаётся массивом с синтаксисом `key: [label, description]`.

```yaml
options:
    all: [All, Guests and customers will be able to access this page.]
    registered: [Registered only, Only logged in member will be able to access this page.]
    guests: [Guests only, Only guest users will be able to access this page.]
```

Другие поля, например [выпадающие списки](./form/field-dropdown.md), поддерживают указание иконки, изображения или цвета как части значения `options`. Если второй элемент массива начинается с `#`, он считается цветом; если значение содержит `.`, оно считается изображением, иначе — CSS‑классом иконки.

```yaml
options:
    red: [Color, '#ff0000']
    icon: [Icon, 'oc-icon-calendar']
    image: [Image, '/path/to/image.png']
```

## Наборы options

Свойство `optionsPreset` задаёт код набора, по которому можно запросить доступные варианты.

```yaml
optionsPreset: icons
```

Доступны следующие наборы:

Preset | Description
------ | -----------
**icons** | Список доступных имён иконок (например, `icon-calendar`)
**phosphorIcons** | Список доступных имён иконок (например, `ph ph-calendar`)
**locales** | Список доступных локалей (например, `en-au`)
**flags** | Список локалей с их иконками-флагами (например, `[en-au, flag-au]`)
**timezones** | Список доступных часовых поясов (например, `Australia/Sydney`)

## Методы options

Свойство `optionsMethod` указывает вызываемый PHP‑метод, с помощью которого можно запросить доступные варианты. Обычно имя метода ссылается на метод связанной модели.

```yaml
optionsMethod: getMyOptionsFromModel
```

Имя метода также может ссылаться на статический метод любого класса.

```yaml
optionsMethod: MyAuthor\MyPlugin\Helpers\FormHelper::getMyStaticMethodOptions
```

### Детальные определения options

Внутри метода можно использовать детальное определение, чтобы задавать более сложные варианты — например, настраивать индивидуальные атрибуты для каждого варианта. Детальное определение распознаётся по соответствующей структуре массива.

```php
public function getDetailedFieldOptions()
{
    return [
        1 => [
            'label' => 'Option 1',
            'comment' => 'This is option one'
        ],
        2 => [
            'label' => 'Option 2',
            'comment' => 'This is option two',
            'disabled' => true
        ]
    ];
}
```

Поддерживаются следующие свойства (где возможно):

Property | Description
------------- | -------------
**label** | имя при отображении варианта пользователю.
**comment** | размещает описание под меткой варианта.
**readOnly** | определяет, является ли вариант доступным только для чтения.
**disabled** | определяет, отключён ли вариант.
**hidden** | определяет вариант, не выводя его.
**color** | задаёт цвет индикатора статуса для варианта в формате hex (для dropdown).
**icon** | указывает имя иконки для варианта (для dropdown).
**image** | указывает URL изображения для варианта (для dropdown).
**optgroup** | установите `true`, чтобы дочерние элементы принадлежали структуре группы вариантов; по умолчанию `false` (для dropdown).
**children** | задаёт дочерние варианты в виде массива для вложенной структуры (для checkbox list).

Используйте свойство `children`, если определение вариантов поддерживает вложенность. Обычно оно отображает структуру для списков с флажками и создаёт группу вариантов для выпадающих списков.

```php
public function getDetailedFieldOptions()
{
    return [
        1 => [
            'label' => 'Option 1',
            'comment' => 'This is option one',
            'children' => [
                2 => [
                    'label' => 'Option 2',
                    'comment' => 'This is option two',
                ],
                // ...
            ]
        ],
    ];
}
```

#### См. также

::: also
* [Поле Checkbox List](./form/field-checkboxlist.md)
* [Поле Dropdown](./form/field-dropdown.md)
* [Поле Radio](./form/field-radio.md)
:::
