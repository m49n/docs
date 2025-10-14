---
subtitle: Виджет формы
shortname: Nested Form
---
# Поле Nested Form

Виджет формы `nestedform` выводит вложенную форму на основе связанной записи или [атрибута jsonable](../../extend/system/models.md). Поля можно определить непосредственно в конфигурации или вынести во внешний файл YAML.

```yaml
content:
    type: nestedform
    showPanel: false
    form:
        fields:
            added_at:
                label: Date Added
                type: datepicker
            details:
                label: Details
                type: textarea
            title:
                label: This the title
                type: text
```

Поддерживаются и часто используются следующие [свойства поля](../form-fields.md).

Свойство | Описание
------------- | -------------
**label** | имя, отображаемое пользователю.
**comment** | размещает описательный комментарий под полем.
**form** | встроенные определения полей или ссылка на файл с определением полей формы.
**showPanel** | размещает форму внутри контейнера-панели. Значение по умолчанию — `true`.
**defaultCreate** | если связанная запись не найдена, пытается создать её. Значение по умолчанию — `false`.

Передайте строку в свойство `form`, чтобы сослаться на внешний YAML-файл.

```yaml
profile:
    label: Profile
    type: nestedform
    form: $/october/demo/models/profile/fields.yaml
```

Как и любая другая форма, виджет nestedform поддерживает вкладки: поместите поля в свойства `tabs` или `secondaryTabs` определения `form`.

```yaml
tabbed_content:
    type: nestedform
    form:
        tabs:
            fields:
                # ...
```

#### См. также

::: also
* [Виджет формы Repeater](./widget-repeater.md)
:::
