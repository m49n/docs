---
subtitle: Form UI
shortname: Hint
---
# Элемент подсказки (hint)

Элемент `hint` идентичен [частичному представлению (partial)](./ui-partial.md), но отображается внутри контейнера подсказки, который пользователь может закрыть.

```yaml
_hint1:
    type: hint
    path: content_field
```

Ниже перечислены поддерживаемые [свойства поля](../form-fields.md).

Property | Description
------------- | -------------
**label** | заголовок секции.
**comment** | дополнительный текст секции.
**mode** | режим отображения: `tip`, `info`, `warning`, `danger`, `success`. По умолчанию — `info`.
**path** | путь к [файлу частичного представления](../../extend/system/views.md).

Подсказка поддерживает содержимое непосредственно внутри поля. Значения `label` и `comment` необязательны и содержат заголовок и подзаголовок. Для значений можно использовать синтаксис Markdown.

```yaml
_tip1:
    type: hint
    mode: tip
    label: Pro Tip
    comment: Always check to make sure this field is populated.
```

Свойство `mode` поддерживает значения: tip, info, warning, danger, success.

```yaml
_warning1:
    type: hint
    mode: warning
    label: Always wash your hands
    comment: This is good for stopping the spread of germs.
```
