---
subtitle: Виджет формы
shortname: Repeater
---
# Поле Repeater

Виджет формы `repeater` выводит повторяющийся набор полей формы на основе связанной записи или [атрибута jsonable](../../extend/system/models.md).

```yaml
extra_information:
    type: repeater
    form:
        fields:
            added_at:
                label: Date Added
                type: datepicker
            details:
                label: Details
                type: textarea
```

Поддерживаются и часто используются следующие [свойства поля](../form-fields.md).

Свойство | Описание
------------- | -------------
**label** | имя, отображаемое пользователю.
**default** | задаёт значение по умолчанию в виде массива, необязательное.
**comment** | размещает описательный комментарий под полем.
**form** | встроенные определения полей или ссылка на файл с определением полей формы.
**prompt** | текст для кнопки создания. Значение по умолчанию — «Add new item».
**displayMode** | управляет интерфейсом: **accordion** или **builder**. Значение по умолчанию — `accordion`.
**useTabs** | включает вкладки, позволяя полям указывать свойство `tab`. Значение по умолчанию — `false`.
**itemsExpanded** | раскрывать элементы повторителя по умолчанию в режиме accordion. Значение по умолчанию — `true`.
**titleFrom** | имя поля внутри элемента, используемое как заголовок свёрнутого элемента, необязательное.
**minItems** | минимальное количество элементов. При отсутствии групп эти элементы отображаются заранее. Например, при `minItems: 1` первая строка будет показана и не скрыта.
**maxItems** | максимальное количество элементов в повторителе.
**groups** | ссылка на группу определений полей, переводящая повторитель в групповый режим (см. ниже). Можно задать и встроенное определение.
**groupKeyFrom** | атрибут с ключом группы, сохраняемый вместе с данными. Значение по умолчанию — `_group`.
**showReorder** | отображает интерфейс сортировки элементов. Значение по умолчанию — `true`.
**showDuplicate** | отображает интерфейс клонирования элементов. Значение по умолчанию — `true`.

Свойство `titleFrom` задаёт значение, используемое при сворачивании повторителя.

```yaml
extra_information:
    type: repeater
    titleFrom: title_when_collapsed
    form:
        fields:
            # ...
            title_when_collapsed:
                label: This field is the title when collapsed
                type: text
```

Повторитель поддерживает вкладки: установите `useTabs` в `true`.

```yaml
extra_information:
    type: repeater
    useTabs: true
    form:
        added_at:
            label: Date added
            type: datepicker
            tab: Date
        details:
            label: Details
            type: textarea
            tab: Details
```

## Групповые повторители

Поле повторителя поддерживает групповой режим через `groups`, позволяя выбирать набор полей для каждого элемента.

```yaml
content:
    type: repeater
    prompt: Add content block
    groups: $/acme/blog/config/fields_repeater.yaml
```

Ниже пример файла конфигурации группы, расположенного в **/plugins/acme/blog/config/fields_repeater.yaml**. Для удобства организации `groups` может указывать отдельный файл для каждой группы.

```yaml
groups:
    textarea: $/acme/blog/config/fields_textarea.yaml
    quote: $/acme/blog/config/fields_quote.yaml
```

Также определения можно задать непосредственно в повторителе. Если ключ группы начинается с подчёркивания (`_`), он игнорируется.

```yaml
groups:
    textarea:
        name: Textarea
        description: Basic text field
        icon: icon-file-text-o
        fields:
            text_area:
                label: Text Content
                type: textarea
                size: large

    quote:
        name: Quote
        description: Quote item
        icon: icon-quote-right
        fields:
            quote_position:
                span: auto
                label: Quote Position
                type: radio
                options:
                    left: Left
                    center: Center
                    right: Right
            quote_content:
                span: auto
                label: Details
                type: textarea
```

Каждая группа должна иметь уникальный ключ, а определение поддерживает следующие параметры.

Параметр | Описание
------------- | -------------
**name** | название группы.
**description** | краткое описание группы.
**icon** | иконка группы, необязательная.
**titleFrom** | имя поля, используемого в качестве заголовка элемента, необязательное.
**fields** | поля формы, принадлежащие группе.
**useTabs** | включает вкладки только для этой группы, необязательное.

::: tip
Ключ группы сохраняется вместе с данными как атрибут `_group`. При необходимости его можно изменить свойством `groupKeyFrom`.
:::

## Пример использования связанных записей

Виджет повторителя автоматически определяет, что атрибут модели является связью, и использует её. Ниже приведён пример реализации. Если модель использует связь `hasMany`, ссылающуюся на модель **RepeaterItem**, повторитель применит эту связанную модель для каждого элемента.

```php
public $hasMany = [
    'extra_information' => [
        RepeaterItem::class,
        'key' => 'parent_id',
        'delete' => true
    ],
];
```

Простую [структуру таблицы базы данных](../../extend/database/structure.md) для модели можно определить с ссылкой на `id` родительской модели и сериализованным JSON-полем `value` для динамических атрибутов (см. ниже).

```php
Schema::create('acme_blog_repeater_items', function($table) {
    $table->increments('id');
    $table->integer('parent_id')->unsigned()->nullable()->index();
    $table->mediumText('value')->nullable();
    $table->integer('sort_order')->nullable();
    $table->timestamps();
});
```

Модель расширяет базовый класс `October\Rain\Database\ExpandoModel`, позволяющий устанавливать динамические атрибуты и сохранять их в базе данных в формате JSON. Модель может [подключать вложения](../../extend/database/attachments.md) и любые другие связанные поля.

```php
use October\Rain\Database\ExpandoModel;

class RepeaterItem extends ExpandoModel
{
    use \October\Rain\Database\Traits\Sortable;

    public $table = 'acme_blog_repeater_items';

    protected $expandoPassthru = ['parent_id', 'sort_order'];

    public $attachMany = [
        'photos' => \System\Models\File::class,
    ];
}
```

Наконец, элемент повторителя можно определить как поле формы со своими полями, включая поля, использующие [связи модели](../../extend/database/relations.md).

```yaml
extra_information:
    type: repeater
    form:
        fields:
            title:
                label: title
            is_enabled:
                label: Enabled
                type: switch
            photos:
                label: Photos
                type: fileupload
                mode: image
```
