---
subtitle: Поле Content
shortname: Nested Items
---
# Поле Nested Items

`nesteditems` — создаёт вложенные записи, принадлежащие исключительно текущей записи.

```yaml
items:
    label: Menu Items
    type: nesteditems
    span: adaptive
    form:
        fields:
            title:
                label: Title
                type: text
```

<VideoBlockLink src="https://www.youtube.com/watch?v=vhs9U3_BHqg" title="Учебник по Nested Items" description="В этом видео показано, как реализовать поле контента Nested Items пошагово." prompt="Посмотреть урок" />

Поддерживаются следующие свойства.

Property | Description
------------- | -------------
**label** | имя, отображаемое пользователю в форме.
**default** | задаёт значение массива по умолчанию (необязательно).
**comment** | добавляет описательный комментарий под полем.
**form** | встроенные определения полей формы.
**maxDepth** | отображает интерфейс для переупорядочивания записей с указанием максимальной глубины. Значение `0` означает неограниченную глубину.
**customMessages** | настраивает сообщения, используемые в пользовательском интерфейсе.

Как и в любой другой форме, во вложенных элементах можно использовать вкладки, поместив поля в свойства `tabs` или `secondaryTabs` определения `form`.

```yaml
tabbed_content:
    type: nesteditems
    form:
        tabs:
            fields:
                # ...
```

Свойство `customMessages` используется для изменения различных сообщений, применяемых в определении поля. Доступные сообщения совпадают с сообщениями [поведения Relation Controller](../../extend/forms/relation-controller.md).

```yaml
author:
    type: nesteditems
    customMessages:
        buttonCreate: New Author
        titleUpdateForm: Update Author
        titleCreateForm: Create Author
```

#### См. также

::: also
* [Поле Entries](./field-entries.md)
* [Поле формы Repeater](../form/widget-repeater.md)
:::
