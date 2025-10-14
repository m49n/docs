---
subtitle: Form Field
shortname: Switch
---
# Поле переключателя (switch)

Поле `switch` выводит переключатель. Оно похоже на [поле checkbox](./field-checkbox.md), но отображается в виде тумблера.

```yaml
show_content:
    label: Display Content
    type: switch
...
```

Ниже перечислены часто используемые [свойства поля](../form-fields.md).

Property | Description
------------- | -------------
**label** | имя, отображаемое пользователю.
**placeholder** | текст, который отображается в пустом поле.
**default** | задаёт строковое значение по умолчанию, необязательное.
**comment** | размещает описательный комментарий под полем.

Свойство `default` задаёт значение по умолчанию.

```yaml
quote_content:
    type: text
    label: Details
    default: I like turtles
```

Используйте свойство `placeholder`, чтобы задать текст-заполнитель.

```yaml
point_summary:
    type: text
    label: Point
    placeholder: Type some key points are you trying to make
```
