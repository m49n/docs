---
subtitle: Поле формы
shortname: Checkbox
---
# Поле Checkbox

Поле `checkbox` выводит одиночный чекбокс.

```yaml
show_content:
    type: checkbox
    label: Display content
```

Чаще всего используются следующие [свойства полей](../form-fields.md).

Property | Description
------------- | -------------
**label** | имя, отображаемое для поля формы.
**default** | значение по умолчанию для новых записей.
**comment** | текст, отображаемый под чекбоксом.

Свойство `default` позволяет включить чекбокс по умолчанию.

```yaml
show_content:
    type: checkbox
    label: Display content
    default: true
```

Используйте `comment`, чтобы вывести сопроводительный текст.

```yaml
is_active:
    type: checkbox
    label: Active
    comment: Check this box to make the record active.
```
