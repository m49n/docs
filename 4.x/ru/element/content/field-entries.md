---
subtitle: Поле Content
shortname: Entries
---
# Поле Entries

`entries` — создаёт связь с другими элементами по UUID или дескриптору (handle).

```yaml
author:
    label: Author
    type: entries
    source: <uuid|handle>
```

Поддерживаются следующие свойства.

Property | Description
------------- | -------------
**source** | UUID или имя дескриптора (handle) связанного blueprint.
**maxItems** | ограничивает количество записей, которые можно выбрать.
**displayMode** | управляет отображением поля. Поддерживаются значения: `relation`, `recordfinder`, `taglist`, `controller`. Значение по умолчанию: `relation`.
**conditions** | задаёт выражение where в чистом виде, применяемое к запросу модели.
**modelScope** | применяет [область запроса модели](../../extend/database/model.md) к **связанной модели формы**; может ссылаться на метод модели или статический метод PHP-класса (`Class::method`).
**inverse** | если определено обратное отношение, имя связанного поля в исходном blueprint.

Чтобы ограничить количество выбираемых элементов, используйте свойство `maxItems`.

```yaml
author:
    type: entries
    maxItems: 1
```

Чтобы отобразить вместо стандартного элемента управления поиск записи (record finder), используйте свойство `displayMode`. Этот режим доступен только при выборе одного элемента.

```yaml
author:
    type: entries
    displayMode: recordfinder
```

Если можно выбрать несколько элементов, `displayMode` поддерживает выбор при помощи списка тегов.

```yaml
author:
    type: entries
    displayMode: taglist
```

## Применение условий

Связанный запрос можно ограничить через SQL или PHP описанными ниже способами. В примерах у связанной записи есть поле `is_featured`, которое отображается флажком. Можно ограничить связанные записи только теми, где установлен этот флажок.

### Условие SQL-запроса

Можно ограничить связанную модель выражением на чистом SQL в свойстве `conditions`.

```yaml
categories:
    label: Categories
    type: entries
    source: Blog\\Category
    conditions: is_featured = true
```

### Области запроса на PHP

Можно ограничить связанный запрос методом PHP через свойство `scope`.

```yaml
basic_entries:
    label: Basic Entry
    type: entries
    source: Basic\\Entry
    scope: App\\Classes\\ScopeHelper::applyScope
```

Здесь используется класс `App\\Classes\\ScopeHelper`, который, например, может находиться в файле **app/classes/ScopeHelper.php**.

```php
namespace App\\Classes;

class ScopeHelper
{
    public static function applyScope($query)
    {
        return $query->where('is_featured', true);
    }
}
```

## Определение обратной связи

В некоторых случаях требуется получить доступ к связи в обратном направлении, например найти все публикации, принадлежащие определённой категории. Свойство `inverse` связывает отношение в противоположную сторону, где значение свойства указывает на имя поля в исходном blueprint.

Например, в blueprint **Blog\\Post** уже определена связь `categories`.

```yaml
categories:
    type: entries
    source: Blog\\Category
```

Blueprint **Blog\\Category** может включать поле `posts` как `inverse` для поля `categories`, определённого в исходном blueprint (выше). Поле можно исключить из форм, установив для `hidden` значение `true`; это необязательно.

```yaml
posts:
    type: entries
    source: Blog\\Post
    inverse: categories
    hidden: true
```

## Отображение в столбце списка

По умолчанию поле entries отображается как гиперссылка на связанную запись.

### Отображение счётчика

Чтобы отобразить в столбце списка счётчик связанных записей, используйте следующую [конфигурацию столбца](../list-columns.md). Свойство `relation` должно быть установлено в имя поля, `relationCount` — в `true`, а тип столбца — `number`.

```yaml
categories:
    label: Categories
    type: entries
    # ...
    column:
        relation: categories
        relationCount: true
        type: number
```

## Расширенное управление записями

Чтобы создавать, изменять и удалять элементы прямо в форме, установите `displayMode` в `controller`, чтобы включить расширенный режим управления на базе [поведения Relation Controller](../../extend/forms/relation-controller.md).

```yaml
author:
    type: entries
    displayMode: controller
```

Если в blueprint значение `navigation` установлено в `false`, по умолчанию отображаются кнопки **Create** и **Delete**. Если навигация определена, кнопки будут **Add** и **Remove**. Кнопки можно настроить через свойство `toolbarButtons`.

```yaml
author:
    type: entries
    toolbarButtons: create|add|remove|delete
```

Тексты сообщений, которые использует контроллер связей, берутся из свойства `customMessages` исходного blueprint; их также можно изменить через `customMessages` в определении поля.

```yaml
author:
    type: entries
    customMessages:
        buttonCreate: New Author
        titleUpdateForm: Update Author
        titleCreateForm: Create Author
```

#### См. также

::: also
* [Поле Nested Items](./field-nesteditems.md)
:::
