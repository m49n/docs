---
subtitle: Поле формы
shortname: Balloon Selector
---
# Поле Balloon Selector

Поле `balloon-selector` выводит список, в котором одновременно можно выбрать только один элемент. Balloon Selector поддерживает те же способы задания опций, что и [тип поля dropdown](./field-dropdown.md).

```yaml
gender:
    type: balloon-selector
    label: Gender
    options:
        female: Female
        male: Male
```

Чаще всего используются следующие [свойства полей](../form-fields.md).

Property | Description
------------- | -------------
**label** | имя, отображаемое для поля формы.
**default** | значение по умолчанию для новых записей.
**comment** | размещает поясняющий комментарий под полем.
**options** | доступные опции для списка, задаются массивом.
**optionsMethod** | получает опции из метода, определённого в модели или как статический метод, например `Class::method`.
**allowEmpty** | позволяет снять выбор, щёлкнув по активному элементу. Значение по умолчанию: `false`.

Свойство `default` задаёт значение по умолчанию; значение должно соответствовать ключу опции.

```yaml
gender:
    type: balloon-selector
    label: Gender
    default: female
```

Установите `allowEmpty` в **true**, чтобы позволить пользователю очистить значение, сняв выделение с активного элемента.

```yaml
gender:
    type: balloon-selector
    label: Gender
    allowEmpty: true
```

#### См. также

::: also
* [Поле Dropdown](./field-dropdown.md)
:::
