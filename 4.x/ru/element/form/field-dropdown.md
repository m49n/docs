---
subtitle: Поле формы
shortname: Dropdown
---
# Поле Dropdown

Поле `dropdown` выводит список с заданными опциями. Существует несколько способов передать опции выпадающего списка, большинство из них сводится к указанию значения `options`.

```yaml
status_type:
    type: dropdown
    label: Blog Post Status
    options:
        draft: Draft
        published: Published
        archived: Archived
```

Поддерживаются и чаще всего используются следующие [свойства полей](../form-fields.md).

Property | Description
------------- | -------------
**title** | заголовок поля формы.
**placeholder** | текст, отображаемый, когда поле пусто.
**default** | значение по умолчанию для новых записей.
**comment** | размещает поясняющий комментарий под полем.
**options** | доступные опции для списка, задаются массивом.
**optionsMethod** | получает опции из метода, определённого в модели или как статический метод, например `Class::method`.
**optionsPreset** | получает опции из [предустановленного набора](../define-options.md).
**emptyOption** | текст, отображаемый при разрешённой пустой опции.
**showSearch** | позволяет пользователю искать опции. Значение по умолчанию: `true`.
**attributes** | ассоциативный массив атрибутов и значений, применяемых к элементу select; удобно для передачи собственной конфигурации Select2 (см. ниже).

Как правило, `options` задаются в виде пары ключ—значение, где значение и подпись определяются независимо.

```yaml
status_type:
    label: Blog Post Status
    type: dropdown
    options:
        draft: Draft
        published: Published
        archived: Archived
```

Свойство `default` задаёт значение по умолчанию; значение должно соответствовать ключу опции.

```yaml
status_type:
    label: Blog Post Status
    type: dropdown
    default: published
```

Чтобы обработать ситуацию, когда значение не выбрано, задайте `emptyOption`, чтобы добавить пустую опцию, доступную для повторного выбора.

```yaml
status:
    label: Blog Post Status
    type: dropdown
    emptyOption: -- no status --
```

Или используйте `placeholder`, чтобы показать «одноразовую» пустую опцию, которую нельзя выбрать повторно.

```yaml
status:
    label: Blog Post Status
    type: dropdown
    placeholder: -- select a status --
```

По умолчанию выпадающий список поддерживает поиск, позволяющий быстро выбрать значение. Отключите эту возможность, установив `showSearch` в `false`.

```yaml
status:
    label: Blog Post Status
    type: dropdown
    showSearch: false
```

## Динамические опции

Дальнейшие подходы используют класс модели в плагине или приложении. Если значение `options` опущено, фреймворк ожидает метод с именем `get*FieldName*Options`, определённый в модели.

В приведённом ниже примере в классе модели должен быть метод `getStatusTypeOptions`. Первый аргумент метода — текущее значение поля, второй — текущий объект данных всей формы. Метод должен возвращать массив опций формата **key => label**.

```yaml
status_type:
    label: Blog Post Status
    type: dropdown
```

Ниже показан пример метода в модели, который возвращает опции списка. Обратите внимание, что имя метода соответствует имени столбца в формате _TitleCase_.

```php
public function getStatusTypeOptions($value, $formData)
{
    return ['all' => 'All', ...];
}
```

Можно определить универсальный метод, который используется как запасной вариант, когда специфический метод не задан. Он будет применяться ко всем полям типа dropdown в модели.

Первый аргумент метода — имя поля, второй — текущее значение поля, третий — текущий объект данных формы. Метод должен возвращать массив опций формата **key => label**.

```php
public function getDropdownOptions($fieldName, $value, $formData)
{
    if ($fieldName == 'status') {
        return ['all' => 'All', ...];
    }
    else {
...
```

Чтобы использовать собственное имя метода, укажите его явно в параметре `options`; тогда оно будет точно соответствовать методу, определённому в модели.

В следующем примере в классе модели должен быть метод `listStatuses`. Он принимает те же аргументы, что и `getDropdownOptions`, и возвращает массив опций формата **key => label**.

```yaml
status:
    label: Blog Post Status
    type: dropdown
    options: listStatuses
```

Ниже показан пример пользовательского метода в модели, который возвращает опции списка.

```php
public function listStatuses($fieldName, $value, $formData)
{
    return ['published' => 'Published', ...];
}
```

Чтобы добавить собственный значок для каждой опции в выпадающем списке, задайте опции как многомерный массив формата **key => [label-text, label-icon]**.

```php
public function listStatuses($fieldName, $value, $formData)
{
    return [
        'published' => ['Published', 'icon-check-circle'],
        'unpublished' => ['Unpublished', 'icon-minus-circle'],
        'draft' => ['Draft', 'icon-clock-o']
    ];
}
```

Также поддерживается вывод пользовательского цвета, если задать опции в формате **key => [label-text, label-color]**, где цвет — hex-значение, начинающееся с символа `#`.

```php
public function listStatuses($fieldName, $value, $formData)
{
    return [
        'published' => ['Published', '#666666'],
        'unpublished' => ['Unpublished', '#ff9999'],
        'draft' => ['Draft', '#ff0000']
    ];
}
```

Чтобы вызвать метод внешнего класса, укажите строку в параметре `options` в формате `ClassName::method` и вызовите статический метод любого полностью определённого класса.

```yaml
status:
    label: Blog Post Status
    type: dropdown
    options: MyAuthor\MyPlugin\Helpers\FormHelper::staticMethodOptions
```

Ниже показан пример статического метода в вспомогательном классе. Первый аргумент — объект модели, второй — определение поля формы.

```php
public static function staticMethodOptions($model, $formField)
{
    return ['published' => 'Published', ...];
}
```

Чтобы использовать группы опций (`optgroup`), задайте дочерние элементы с помощью [детального определения опций](../define-options.md). В примере ниже подпись группы опций берётся из значения, поэтому её не нужно повторять. Свойство `children` содержит опции группы; поддерживается только один уровень вложенности.

```php
public function getDetailedFieldOptions()
{
    return [
        'Option Group' => [
            'optgroup' => true,
            'children' => [
                1 => 'Option 1',
                2 => 'Option 2',
                // ...
            ]
        ],
    ];
}
```

## Пользовательская конфигурация Select2

Поле dropdown использует [элемент управления Select2](https://select2.org/) для отображения. В некоторых случаях требуется задать собственную конфигурацию поля. Это можно сделать через свойство `attributes` и [настройку data-атрибутов](https://select2.org/configuration/data-attributes), поддерживаемую Select2.

Например, можно точнее настроить автодополнение списка.

```yaml
attributes:
    data-handler: onGetClientOptions
    data-minimum-input-length: 3
    data-process-Results: true
    data-ajax--delay: 300
```

При использовании с [полем Tag List](./widget-taglist.md) настройка ниже оставит список открытым после выбора элемента.

```yaml
categories:
    type: taglist
    attributes:
        data-close-on-select: false
```

#### См. также

::: also
* [Поле Tag List](./widget-taglist.md)
* [Элемент управления Select2](https://select2.org/)
:::
