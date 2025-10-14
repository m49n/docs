---
subtitle: Form Field
shortname: Textarea
---
# Многострочное текстовое поле (textarea)

Поле `textarea` выводит многострочное текстовое поле.

```yaml
blog_contents:
    type: textarea
    label: Contents
```

Ниже перечислены поддерживаемые и часто используемые [свойства поля](../form-fields.md).

Property | Description
------------- | -------------
**title** | заголовок для поля формы.
**default** | задаёт строковое значение по умолчанию, необязательное.
**placeholder** | текст, который отображается, когда поле пустое.
**comment** | размещает описательный комментарий под полем.
**size** | высота поля. Поддерживаемые значения: `tiny`, `small`, `large`, `huge`, `giant`. По умолчанию — `large`.

Размер поля задаётся свойством `size`.

```yaml
blog_contents:
    type: textarea
    label: Contents
    size: large
```

Свойство `default` задаёт значение по умолчанию.

```yaml
quote_content:
    type: textarea
    label: Details
    default: I like turtles
```

Используйте свойство `placeholder`, чтобы задать текст-заполнитель.

```yaml
point_summary:
    type: textarea
    label: Point
    placeholder: Type some key points are you trying to make
```
