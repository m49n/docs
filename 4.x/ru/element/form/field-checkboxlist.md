---
subtitle: Поле формы
shortname: Checkbox List
---
# Поле Checkbox List

Поле `checkboxlist` выводит список чекбоксов. Checkbox List поддерживает те же методы задания опций, что и [тип поля dropdown](./field-dropdown.md), а также поддерживает вторичные описания, доступные для [типа поля radio](./field-radio.md).

```yaml
permissions:
    label: Permissions
    type: checkboxlist
    options:
        open_account: Open account
        close_account: Close account
        modify_account: Modify account
```

Поддерживаются и чаще всего используются следующие [свойства полей](../form-fields.md).

Property | Description
------------- | -------------
**label** | имя, отображаемое для поля формы.
**options** | доступные опции для списка, задаются массивом.
**optionsMethod** | получает опции из метода, определённого в модели или как статический метод, например `Class::method`.
**default** | значение по умолчанию для новых записей.
**quickselect** | показывает кнопки быстрого выбора.
**cssClass** | позволяет расположить опции в строку.
**inlineOptions** | отображает опции в один ряд, а не столбиком, если их меньше 10.
**placeholder** | сообщение, отображаемое при отсутствии выбранных записей (в контексте предпросмотра).
**cumulative** | при вложенных чекбоксах отмечает всех потомков при выборе родителя. Значение по умолчанию: `false`.

Свойство `default` задаёт значение по умолчанию; значение должно соответствовать ключу опции.

```yaml
permissions:
    label: Permissions
    type: checkboxlist
    default: open_account
```

Чтобы расположить опции в одну строку, установите свойство `inlineOptions` в значение `true`. Это применимо только если доступно меньше 10 опций.

```yaml
permissions:
    type: checkboxlist
    inlineOptions: true
```

Меню быстрого выбора с кнопками «Select All» и «Select None» появляется автоматически, если в списке более 10 элементов. Чтобы явно включить эти кнопки, используйте опцию `quickselect`.

```yaml
permissions:
    type: checkboxlist
    quickselect: true
```

При использовании [детальных опций](../define-options.md) чекбоксы могут образовывать вложенную структуру; задайте свойство `cumulative` в `true`, чтобы при выборе родительского чекбокса отмечались все дочерние.

```yaml
permissions:
    type: checkboxlist
    cumulative: true
```

#### См. также

::: also
* [Детальное определение опций](../define-options.md)
* [Поле Dropdown](./field-dropdown.md)
* [Поле Radio](./field-radio.md)
:::
