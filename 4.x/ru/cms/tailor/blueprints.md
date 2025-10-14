---
subtitle: Структура шаблонов для контента сайта.
---
# Чертежи

## Entry

Чертёж типа entry — это стандартная структура контента, используемая для описания областей сайта. Структуры контента и записи можно организовывать по-разному, как описано далее.

Тип `entry` поддерживает несколько записей и подходит, когда не применяется другая разновидность.

```yaml
handle: Team\Member
type: entry
name: Team Member

fields:
    name:
        label: First Name
        type: text
```

Чертёж entry поддерживает следующие свойства.

Property | Description
-------- | -------------
**handle** | Осмысленный уникальный код для идентификации записи.
**type** | Тип чертежа, может быть вариантом `entry`, `single`, `structure` или `stream`.
**name** | Метка, отображаемая при работе с записью.
**fields** | Поля формы группы, см. [поля форм бэкенда](../../element/form-fields.md).
**groups** | Ссылается на группу полей формы и включает режим групп для записи (см. ниже).
**structure** | Конфигурация структуры при использовании типа `structure`.
**drafts** | Включает черновики для записи. Значение по умолчанию: `false`.
**softDeletes** | Включает мягкое удаление для записи. Значение по умолчанию: `true`.
**multisite** | Включает Multisite для записи, синхронизация между группой, локалью или всеми сайтами. Поддерживаемые значения: `true`, `false`, `sync`, `locale`, `all`. Значение по умолчанию: `false`.
**pagefinder** | Включает тип чертежа в [виджет формы pagefinder](../../element/form/widget-pagefinder.md), поддерживаемые значения: `true`, `false`, `item`, `list` или массив (см. ниже). Значение по умолчанию: `true`.
**defaultSort** | Используется типами `entry` и `stream`, задаёт столбец и направление сортировки по умолчанию, если предпочтения пользователя не определены. Поддерживает строку или массив с ключами `column` и `direction`. Направление может быть `asc` (по возрастанию, по умолчанию) или `desc` (по убыванию).
**customMessages** | Настраивает сообщения интерфейса (см. ниже).
**showExport** | Показывает кнопку панели инструментов для экспорта записей. Значение по умолчанию: `true`.
**showImport** | Показывает кнопку панели инструментов для импорта записей. Значение по умолчанию: `true`.
**modelClass** | Заменяет PHP-модель на [пользовательский экземпляр модели](./models.md).

### Варианты entry

Хотя тип entry не имеет особого поведения, доступны несколько вариантов для организации контента. Следующие типы являются вариантами entry.

- **entry** — базовая запись для распространённых сценариев.
- **single** — одиночная запись с собственными полями, например страница контактов.
- **structure** — набор структурированных записей, например страницы документации.
- **stream** — поток записей с отметкой времени, например блог.

#### Одиночные записи

Тип `single` принудительно создаёт одну запись для каждого определения секции. Это удобно для однотипного контента, например главной страницы или страницы «Свяжитесь с нами». В примере ниже определяется секция **Homepage** с текстовым полем Welcome Message (`welcome_message`).

```yaml
handle: Homepage
type: single
name: Homepage Content

fields:
    welcome_message:
        label: Welcome Message
        type: text
```

#### Структурированные записи

Тип `structure` позволяет создавать несколько структурированных записей с отношениями родитель—потомок. Это полезно для вложенного контента, например раздела документации. Записи типа `structure` поддерживают сортировку. Порядок можно менять, перетаскивая их в списке. Ниже определяется древовидная секция **Documentation** с полем Markdown Article Content (`article_content`).

```yaml
handle: Docs\Article
type: structure
name: Documentation Article

fields:
    content:
        label: Article Content
        type: markdown
```

По умолчанию структуры поддерживают неограниченную вложенность, но можно задать максимальную глубину дерева с помощью `maxDepth` в свойстве `structure`. В следующем примере допустим только верхний уровень и второй уровень.

```yaml
# ...
type: structure

structure:
    maxDepth: 2
    # ...
```

Свойство `structure` поддерживает следующие значения.

Property | Description
-------- | -------------
**maxDepth** | Максимальная глубина структуры. Значение по умолчанию: `0` (без ограничений).
**treeExpanded** | Определяет, раскрывать ли узлы дерева по умолчанию. Значение по умолчанию: `true`.
**showReorder** | Отображает интерфейс переупорядочивания записей. Значение по умолчанию: `true`.
**showSorting** | Включает сортировку записей, отключает структуру при сортировке. Значение по умолчанию: `true`.

#### Потоковые записи

Тип `stream` используют для записей, привязанных ко времени, которые часто выводятся в хронологическом порядке. Это подходит для публикации последних событий, например в разделе блога. Ниже определяется секция ленты **Blog** с полем редактора Post Content (`content`).

```yaml
handle: Blog\Post
type: stream
name: Blog Post
...
```

```yaml
navigation:
    icon: icon-pencil
    order: 200
```

Следующие свойства поддерживаются определениями `navigation` и `primaryNavigation`.

Property | Description
------------- | -------------
**label** | Задаёт ключ локализации метки меню, обязательное.
**order** | Числовой вес при определении порядка отображения.
**parent** | Привязывает пункт навигации к родительскому с помощью handle чертежа.
**icon** | Имя иконки из [набора иконок October CMS](../../element/available-icons.md), необязательное.
**iconSvg** | SVG-иконка вместо стандартной, должна быть прямоугольной и может содержать цвета, необязательное.

Чтобы поместить пункт в раздел Settings, установите **parent** в `settings`. Определение **category** может быть строкой или ссылкой на константу настроек, например `CATEGORY_COLLECTIONS`.

```yaml
navigation:
    parent: settings
    category: Collections
```

Чтобы разместить пункт в разделе Content, установите **parent** в `content`.

```yaml
navigation:
    parent: content
```

Чтобы сделать пункт элементом основной навигации, добавьте определение **primaryNavigation**.

```yaml
primaryNavigation:
    label: Blog
    icon: icon-copy
    order: 500

navigation:
    label: Main Menu Item
```

Чтобы добавить пункт вторичной навигации, свойство **parent** должно указывать UUID или handle элемента основной навигации.

```yaml
navigation:
    parent: <handle|uuid>
```

Чтобы отключить вторичную навигацию, определите **primaryNavigation** для одного чертежа, не делая его родителем других чертежей.

```yaml
primaryNavigation:
    label: Page
    icon: icon-magic
    order: 500
```

Можно полностью отключить навигацию, указав свойство **navigation** со значением `false`.

```yaml
navigation: false
```

## Дополнительная навигация

Используйте свойство `extraNavigation`, чтобы зарегистрировать пользовательские пункты навигации, добавляемые к чертежу. Значение — это массив, соответствующий определению `sideMenu` из [спецификации навигации бэкенда](../../extend/backend/navigation.md). В примере ниже добавляются секция и разделитель с пользовательскими типами отображения, порядок задаётся свойством `order`.

```yaml
navigation:
    label: Authors
    parent: Blog\Post
    icon: icon-user
    order: 230

extraNavigation:
    _authors_section:
        itemType: section
        label: Authors
        order: 210

    _authors_ruler:
        itemType: ruler
        order: 220
```

Можно регистрировать ссылки на [контроллеры, добавленные плагинами](../../extend/system/controllers.md), указав свойство `url`. Значение должно содержать URL контроллера; ниже показана ссылка на контроллер **acme/blog/posts**.

```yaml
navigation:
    label: Authors
    # ...

extraNavigation:
    testimonials:
        label: Testimonials
        order: 210
        icon: icon-group
        url: acme/blog/posts
```

Чтобы задать контекст навигации внутри контроллера, используйте метод `setTailorContext` фасада (facade) `BackendMenu`. Также можно указать UUID чертежа методом `setTailorContextUuid`. Метод принимает handle или `uuid` чертежа (первый аргумент) и ключ элемента дополнительной навигации (второй аргумент).

```php
BackendMenu::setTailorContext('Blog\Post', 'testimonials');

BackendMenu::setTailorContextUuid('edcd102e-0525-4e4d-b07e-633ae6c18db6', 'testimonials');
```

### Группы контента

Все записи при необходимости поддерживают определение нескольких групп контента для секции. Например, раздел блога может иметь обычную запись и избранную запись — это две группы записей.

Группы записей задаются свойством **groups** в файле чертежа секции, и для каждого типа можно указать разные **fields**. Выбранное значение группы доступно в атрибуте `content_group` записи.

```yaml
handle: Blog\Post
type: stream
name: Blog Post

groups:
    regular_post:
        name: Regular Post
        fields:
            # ...

    featured_post:
        name: Featured Post
        fields:
            # ...
```

::: tip
Рекомендуется использовать [чертежи-примеси](#mixin), чтобы группировать общие определения полей.
:::

### Пользовательские сообщения

Задайте свойство `customMessages`, чтобы переопределить стандартные сообщения интерфейса. Значения могут быть обычным текстом или ссылаться на [строки локализации](../../extend/system/localization.md).

```yaml
customMessages:
    buttonCreate: Create New Event
```

Ниже приведён список сообщений, доступных для переопределения.

::: details Посмотреть список доступных сообщений
Message | Default Message
------------- | -------------
**buttonCreate** | Create :name Entry
**titleIndexList** | Manage :name Entries
**titleCreateForm** | Create :name
**titleUpdateForm** | Update :name
**pagefinderItemType** | :name Entry
**pagefinderListType** | All :name Entries
:::

### Отключение обязательных полей

Запись требует заполнения полей `title` и `slug` перед сохранением, причём значение `slug` должно быть уникальным. Можно изменить это поведение, переопределив поля в чертеже: задать свойству `validation` значение **false** или `hidden` — **true**, чтобы скрыть поле в интерфейсе. В примере ниже отключается проверка для обоих полей, а поле slug скрывается.

```yaml
fields:
    title:
        validation: false

    slug:
        hidden: true
```

### Настройка page finder

По умолчанию все записи включены в значения поиска [page finder](../../element/form/widget-pagefinder.md). Это можно отключить, установив **pagefinder** в `false`.

```yaml
pagefinder: false
```

Можно ограничить контекст page finder, разрешив находить страницу только как отдельный элемент `item` (например, публикацию блога) или список `list` (например, список записей блога). Значение `all` покажет оба варианта.

```yaml
pagefinder: item
pagefinder: list
```

Page finder автоматически определяет атрибуты `id`, `code`, `slug` и `fullslug` и использует их как подстановки в [параметрах URL страницы](../themes/pages.md). Можно указать пользовательские **replacements** как массив в свойстве **pagefinder**, включая необязательный **context**, приведённый выше.

```yaml
pagefinder:
    context: list
    replacements: []
```

Каждый ключ заменителя должен совпадать с именем параметра URL и использовать путь к атрибуту в точечной нотации. Рассмотрим пример URL страницы публикации блога.

```ini
url = "/blog/post/:author/:category/:slug/:id"
```

Следующие **replacements** установят параметр `:author` в значение атрибута slug связанного автора, а параметр `:category` — в slug первой связанной категории.

```yaml
pagefinder:
    replacements:
        author: author.slug
        category: categories.0.slug
```

## Global

Чертежи global используют для определения глобального контента сайта. Значения полей часто применяют в [макетах (layouts) CMS](../../cms/themes/layouts.md) и содержат настройки, например ссылки на социальные сети.

Ниже задаётся глобальный чертёж **Footer Config** с текстовым полем Facebook Link (`facebook_link`).

```yaml
handle: Site\Footer
type: global
name: Footer Config

fields:
    facebook_link:
        label: Facebook Link
        type: text
```

Чертёж global поддерживает следующие свойства.

Property | Description
-------- | -------------
**handle** | Осмысленный уникальный код для идентификации записи.
**name** | Метка, отображаемая при работе с записью.
**fields** | Поля формы группы, см. [поля форм бэкенда](../../element/form-fields.md).
**multisite** | Включает Multisite для записи, поддерживаемые значения: `true`, `false`. Значение по умолчанию: `false`.
**formSize** | Размер формы настроек, поддерживаемые значения: `tiny`, `small`, `medium`, `large`, `huge`, `giant`, `adaptive`. Значение по умолчанию: `huge`.

## Mixin

Примеси — это группы полей, используемые, чтобы избежать повторов при описании структур контента. Например, поля расположения можно задать один раз с помощью определения примеси.

Ниже задаётся коллекция **Location** с текстовыми полями Country (`country_code`) и State (`state_code`).

```yaml
handle: Fields\Location
type: mixin
name: Location

fields:
    country_code:
        label: Country
        type: text

    state_code:
        label: State
        type: text
```

Чертёж mixin поддерживает следующие свойства.

Property | Description
-------- | -------------
**handle** | Осмысленный уникальный код для идентификации записи.
**name** | Метка, отображаемая при работе с записью.
**fields** | Поля формы группы, см. [поля форм бэкенда](../../element/form-fields.md).

::: tip
Рекомендуется добавлять к именам файлов и полей примеси префикс подчёркивания (\_), чтобы было проще найти чертежи этого типа. Например: `_location_fields.yaml`.
:::

### Использование примеси

Чтобы включить эти поля в записи как обычные поля формы, укажите `type` со значением **mixin** и ссылку на UUID или handle в свойстве `source`.

```yaml
_location_fields:
    type: mixin
    source: Fields\Location
```

Дополнительные сведения о примесях см. в [описании поля Mixin](../../element/content/field-mixin.md).

#### См. также

::: also
* [Поле содержимого Mixin](../../element/content/field-mixin.md)
:::
