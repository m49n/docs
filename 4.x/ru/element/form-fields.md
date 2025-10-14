---
subtitle: Узнайте о различных типах полей форм.
---
# Поля форм (Form Fields)

Поля форм, элементы Form UI и виджеты формы — это определения полей, которые используют формы, например текстовое поле ввода. Их часто применяют в следующих местах:

- [Настройки темы CMS](../cms/themes/settings.md)
- [Поля контента Tailor](../cms/tailor/content-fields.md)
- [Поведение Backend Form Controller](../extend/forms/form-controller.md)
- [Поведение Backend Relation Controller](../extend/forms/relation-controller.md)

Все поля форм идентифицируются по своему свойству **type**.

```yaml
fields:
    myfield:
        type: textarea
        # ...
```

Поля форм содержат общие и простые поля. Form UI предназначен для элементов интерфейса, которые можно включать в формы для помощи в построении макета. Виджеты формы часто добавляют более сложную функциональность; плагинам обычно свойственно поставлять собственные виджеты формы. Поля Tailor доступны только в чертежах Tailor.

## Доступные поля

Доступны следующие поля формы:

<div class="content-list-p" markdown="1">

[Mixin](./content/field-mixin.md)
[Entries](./content/field-entries.md)
[Text](./form/field-text.md)
[Number](./form/field-number.md)
[Password](./form/field-password.md)
[Email](./form/field-email.md)
[Textarea](./form/field-textarea.md)
[Dropdown](./form/field-dropdown.md)
[Radio List](./form/field-radio.md)
[Balloon Selector](./form/field-balloon.md)
[Checkbox](./form/field-checkbox.md)
[Checkbox List](./form/field-checkboxlist.md)
[Switch](./form/field-switch.md)
[Code Editor](./form/widget-codeeditor.md)
[Color Picker](./form/widget-colorpicker.md)
[Data Table](./form/widget-datatable.md)
[Date Picker](./form/widget-datepicker.md)
[File Upload](./form/widget-fileupload.md)
[Markdown Editor](./form/widget-markdown.md)
[Media Finder](./form/widget-mediafinder.md)
[Nested Form](./form/widget-nestedform.md)
[Record Finder](./form/widget-recordfinder.md)
[Relation](./form/widget-relation.md)
[Repeater](./form/widget-repeater.md)
[Rich Editor](./form/widget-richeditor.md)
[Page Finder](./form/widget-pagefinder.md)
[Sensitive](./form/widget-sensitive.md)
[Tag List](./form/widget-taglist.md)
[Currency](./form/widget-currency.md)
[Boxes](./form/widget-boxes.md)
[Section](./form/ui-section.md)
[Hint](./form/ui-hint.md)
[Ruler](./form/ui-ruler.md)
[Partial](./form/ui-partial.md)

</div>

## Свойства поля

Для каждого поля можно указать следующие общие свойства (где применимо).

Property | Description
------------- | -------------
**label** | имя при отображении поля формы пользователю.
**type** | определяет способ отображения поля. Значение по умолчанию: `text`.
**span** | выравнивает поле формы по стороне. Варианты: `auto`, `left`, `right`, `row`, `full`. Значение по умолчанию: `full`.
**spanClass** | используется со значением `row` для `span`, чтобы отображать форму как сетку Bootstrap, например `spanClass: col-4`.
**size** | задаёт размер для полей, которые его поддерживают, например для textarea. Варианты: `tiny`, `small`, `large`, `huge`, `giant`.
**placeholder** | значение заполнитель, если поле его поддерживает.
**comment** | размещает описание под полем.
**commentAbove** | размещает описание над полем.
**commentHtml** | разрешает HTML‑разметку внутри описания. Варианты: `true`, `false`.
**default** | задаёт значение по умолчанию. Для виджетов `dropdown`, `checkboxlist`, `radio` и `balloon-selector` можно указать ключ варианта, который будет выбран по умолчанию.
**defaultFrom** | берёт значение по умолчанию из значения другого атрибута модели.
**tab** | назначает поле вкладке.
**cssClass** | назначает CSS‑класс контейнеру поля.
**autoFocus** | фокусирует поле при загрузке формы. Значение по умолчанию: `false`.
**readOnly** | запрещает изменять поле. Варианты: `true`, `false`.
**disabled** | запрещает изменять поле и исключает его из сохраняемых данных. Варианты: `true`, `false`.
**hidden** | скрывает поле и исключает его из сохраняемых данных. Варианты: `true`, `false`.
**stretch** | указывает, растягивается ли поле по высоте родителя.
**context** | определяет контекст, в котором следует отображать поле. Контекст можно передать, используя символ `@` в имени поля, например `name@update`.
**dependsOn** | массив имён полей, от которых [зависит](../extend/forms/field-dependencies.md) текущее поле. При их изменении поле обновляется.
**changeHandler** | имя AJAX‑обработчика, который вызывается при изменении значения поля, необязательное.
**trigger** | задаёт условия для поля с помощью событий триггера.
**preset** | позволяет изначально заполнить значение поля значением другого поля, преобразованным конвертером пресетов ввода.
**required** | добавляет красную звёздочку к метке поля, указывая на обязательность. Не забудьте использовать [трейты валидации в модели](../extend/database/traits.md), поскольку поведение формы не принуждает к проверке.
**attributes** | задаёт пользовательские HTML‑атрибуты для элемента поля формы.
**containerAttributes** | задаёт пользовательские HTML‑атрибуты контейнеру поля формы.
**order** | числовой вес для определения порядка отображения; значение по умолчанию увеличивается на 100 для каждого поля.
**permissions** | [разрешения](../extend/backend/permissions.md), которые должны быть у текущего пользователя бэкенда, чтобы поле было доступно. Поддерживает строку с одним разрешением или массив, из которого требуется хотя бы одно разрешение.

### Свойства вкладок

Пример определения полей во вкладках.

```yaml
tabs:
    fields:
        username:
            type: text
            label: Username
            tab: User

        groups:
            type: relation
            label: Groups
            tab: Groups
```

Для каждой конфигурации вкладок — `tabs` и `secondaryTabs` — доступны следующие свойства.

Property | Description
------------- | -------------
**stretch** | указывает, растягивается ли вкладка по высоте родителя.
**defaultTab** | вкладка по умолчанию для назначения полей. Значение по умолчанию: Misc.
**activeTab** | выбранная вкладка при первой загрузке формы, имя или индекс. Значение по умолчанию: `1`.
**icons** | назначает иконки вкладкам, используя имена вкладок как ключи.
**lazy** | массив вкладок, которые загружаются динамически при клике. Полезно для вкладок с большим объёмом данных.
**identifiers** | массив пользовательских HTML‑идентификаторов для таргетинга вкладки. Полезно для управления видимостью через JavaScript.
**linkable** | определяет, можно ли связывать вкладки через фрагменты URL. Значение по умолчанию: `true`.
**cssClass** | назначает CSS‑класс контейнеру вкладок.
**paneCssClass** | назначает CSS‑класс отдельной панели вкладки. Значение — массив: ключом служит индекс или метка вкладки, значением — CSS‑класс. Можно указать строку, тогда класс применяется ко всем вкладкам.

Пример применения свойств вкладок.

```yaml
tabs:
    stretch: true
    defaultTab: User
    cssClass: text-blue

    lazy:
        - Groups

    paneCssClass:
        1: first-tab
        2: second-tab

    icons:
        User: icon-user
        Groups: icon-group

    identifiers:
        User: userTab

    fields:
        # [...]
```

### Пользовательские типы полей

Для настройки свойства **type** доступны различные встроенные типы. Также можно выводить поле напрямую, указав имя PHP‑класса [виджета поля формы](../extend/forms/form-widgets.md).

```yaml
blog_content:
    type: Backend\FormWidgets\RichEditor
    size: huge
```

### Выбор вложенных полей

```yaml
avatar[name]:
    label: Avatar
    comment: will be saved in the Avatar table
```

В этом примере значение в PHP будет получено и сохранено как `$record->avatar->name` или `$record->avatar['name']` соответственно.

### Поля-фасады

Иногда необходимо отобразить поле, но не отправлять его. Чтобы определить поле как фасад, добавьте символ подчёркивания (`_`) перед именем поля. Такие поля автоматически очищаются и больше не сохраняются в модель, как в следующем примере с полем `_map`.

```yaml
address:
    label: Title
    type: text

_map:
    label: Point your address on the map
    type: mapviewer
```

## Условия для поля

Иногда требуется изменить значение или внешний вид поля формы при определённых условиях, например скрыть ввод, если установлен флажок, или заполнить другое поле значением.

### События-триггеры

События-триггеры задаются свойством `trigger` и представляют собой простое браузерное решение на JavaScript. Они позволяют изменять атрибуты элементов, такие как видимость или значение, в зависимости от состояния другого элемента. Пример определения:

```yaml
is_delayed:
    label: Send later
    comment: Place a tick in this box if you want to send this message at a later time.
    type: checkbox

send_at:
    label: Send date
    type: datepicker
    cssClass: field-indent
    trigger:
        action: show
        field: is_delayed
        condition: checked
```

В приведённом примере поле `send_at` отображается только тогда, когда поле `is_delayed` включено. Иными словами, поле отображается (action), если другое поле (field) отмечено (condition).

Определение `trigger` поддерживает следующие свойства.

Property | Description
------------- | -------------
**action** | определяет действие, применяемое к полю при выполнении условия. Поддерживаемые значения: `show`, `hide`, `enable`, `disable`, `empty`, `fill[somevalue]`.
**field** | ссылка на имя другого поля, которое инициирует действие. Пример: `color` или `color[]`.
**condition** | определяет условие, которому должно соответствовать указанное поле, чтобы считать его истинным. Поддерживаемые значения: `checked`, `unchecked`, `value[somevalue]`.

#### Несколько действий

Можно комбинировать несколько действий, разделив их символом `|`. В следующем примере поле отображается и очищается, когда выполняется условие.

```yaml
trigger:
    action: show|empty
    condition: checked
    field: name
```

#### Несколько значений в условии

При использовании условия `value[]` можно проверять несколько значений, передав дополнительные значения после первого в формате `value[][]`.

```yaml
trigger:
    action: show
    condition: value[csv][csv_custom]
    field: file_format
```

#### Подстановочные знаки в условии

Можно проверять, совпадает ли `value[]` с несколькими возможными значениями, используя символ подстановки (`*`). Например, **foo\*** соответствует любому значению, начинающемуся с «foo», а **\*bar** — заканчивающемуся на «bar».

```yaml
trigger:
    action: show
    condition: value[*.mp4]
    field: file_name
```

Условие `value[]` можно использовать для проверки пустого значения.

```yaml
trigger:
    action: show
    condition: value[]
    # ...
```

А условие `value[*]` — для проверки наличия любого значения.

```yaml
trigger:
    action: show
    condition: value[*]
    # ...
```

#### Несколько значений поля

Некоторые поля, такие как [Checkbox List](./form/field-checkboxlist.md) и [Tag List](./form/widget-taglist.md), сохраняют значения в виде массива. При ссылке на такие поля в имени необходимо использовать суффикс массива (`[]`), чтобы учитывать все возможные значения. Если, например, поле называется `colors` и поддерживает несколько значений, в качестве ссылки следует использовать имя `colors[]`.

```yaml
trigger:
    action: show
    condition: value[red][green]
    field: colors[]
```

#### Ссылки на родительские поля

Обычно имя поля относится к полю на том же уровне формы. Например, если поле находится в [виджете-повторителе](./form/widget-repeater.md), будут проверяться только поля в этом же повторителе. Однако если перед именем поля указать символ каретки `^`, например `^parent_field`, то оно будет ссылаться на повторитель или форму уровнем выше.

В примере ниже поле `colors` отображается, если поле `type` установлено в значение **Complex**.

```yaml
fields:
    type:
        label: Type
        type: dropdown
        options:
            1: Simple
            2: Complex

    content:
        label: Content
        type: nestedform
        form:
            fields:
                colors:
                    label: Colors
                    type: colorpicker
                    trigger:
                        action: show
                        field: ^type
                        condition: value[2]
```

::: tip
Если использовать несколько символов каретки `^`, можно обратиться к полям на соответствующее число уровней выше: `^^grand_parent_field`, `^^^grand_grand_parent_field` и т. д.
:::

### Конвертер пресетов ввода

Конвертер пресетов ввода задаётся свойством `preset` и позволяет преобразовывать текст, введённый в один элемент, в значение URL, slug или имени файла в другом поле ввода.

В следующем примере поле `url` заполняется автоматически, когда пользователь вводит текст в поле `title`. Если ввести текст **Hello world** в поле Title, то в URL будет записано преобразованное значение **/hello-world**. Такое поведение выполняется только тогда, когда поле‑получатель (`url`) пустое и не изменялось.

```yaml
title:
    label: Title

url:
    label: URL
    preset:
        field: title
        type: url
```

В качестве альтернативы значение `preset` можно задать строкой, указав только **field**. В этом случае `type` по умолчанию принимает значение **slug**.

```yaml
slug:
    label: Slug
    preset: title
```

Для параметра `preset` доступны следующие варианты.

Option | Description
------------- | -------------
**field** | задаёт имя другого поля, из которого берётся значение.
**type** | определяет тип преобразования. См. список поддерживаемых значений ниже.
**prefixInput** | необязательный параметр, добавляет к преобразованному значению значение из элемента ввода, найденного по CSS‑селектору.

Поддерживаемые типы:

Type | Description
------------- | -------------
**exact** | копирует значение как есть.
**slug** | форматирует значение как slug.
**url** | то же, что slug, но с префиксом `/`.
**camel** | форматирует значение в camelCase.
**file** | форматирует значение как имя файла, заменяя пробелы дефисами.
