---
subtitle: Form UI
shortname: Section
---
# Элемент секции (section)

Элемент `section` выводит заголовок и подзаголовок секции. Значения `label` и `comment` не являются обязательными и содержат текст заголовка и подзаголовка.

```yaml
_section1:
    type: section
    label: User details
    comment: This section contains details about the user.
```

Поддерживаются следующие [свойства поля](../form-fields.md).

Property | Description
------------- | -------------
**label** | текст заголовка секции.
**comment** | дополнительный текст секции.
**displayMode** | определяет режим отображения секции: `simple` или `heading`. По умолчанию — `heading`.

Чтобы в секции отображался простой комментарий вместо заголовка, задайте свойство `displayMode`.

```yaml
_section1:
    type: section
    label: These fields are used to calculate some other fields.
    displayMode: simple
```
