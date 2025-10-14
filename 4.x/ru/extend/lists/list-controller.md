---
subtitle: Добавляет функции управления списками на любую страницу бэкенда.
---
# Контроллер списка

Класс `Backend\\Behaviors\\ListController` — это поведение контроллера, которое упрощает добавление списка записей на страницу. Поведение предоставляет сортируемый и доступный для поиска список с необязательными ссылками на его записи. Поведение реализует действие контроллера `index`, однако список можно отрисовать в любом месте, а также можно использовать несколько определений списка.

Поведение списка зависит от [определений колонок списка](../../element/list-columns.md) и [класса модели](../database/model.md). Чтобы использовать поведение списка, добавьте его в свойство `$implement` класса контроллера. Также необходимо определить свойство `$listConfig`, значение которого указывает на YAML‑файл с настройками поведения.

```php
namespace Acme\\Blog\\Controllers;

class Categories extends \\Backend\\Classes\\Controller
{
    public $implement = [
        \\Backend\\Behaviors\\ListController::class
    ];

    public $listConfig = 'config_list.yaml';
}
```

::: tip
Очень часто контроллер списка и [контроллер формы](../forms/form-controller.md) используются вместе в одном контроллере.
:::

## Настройка поведения списка

Конфигурационный файл, указанный в свойстве `$listConfig`, задаётся в формате YAML. Файл должен располагаться в [каталоге представлений контроллера](../system/controllers.md). Ниже приведён пример типичного конфигурационного файла поведения списка.

```yaml
# config_list.yaml
title: Blog Posts
list: ~/plugins/acme/blog/models/post/columns.yaml
modelClass: Acme\\Blog\\Models\\Post
recordUrl: acme/blog/posts/update/:id
```

Следующие параметры в конфигурационном файле списка являются обязательными.

Property | Description
------------- | -------------
**title** | заголовок списка.
**list** | конфигурационный массив или ссылка на файл с определением колонок списка, см. [колонки списка](../../element/list-columns.md).
**modelClass** | класс модели, из которой загружаются данные списка.

Ниже перечислены необязательные параметры конфигурации.

Property | Description
------------- | -------------
**filter** | конфигурация фильтра, см. [фильтры списка](./filters.md).
**recordUrl** | связывает каждую запись списка с другой страницей, например: **users/update:id**. Часть `:id` будет заменена идентификатором записи. Это позволяет связать поведение списка с [поведением формы](../forms/form-controller.md).
**recordOnClick** | пользовательский JavaScript‑код, выполняемый при клике по записи.
**noRecordsMessage** | сообщение, отображаемое при отсутствии записей; можно сослаться на [строку локализации](../system/localization.md).
**deleteMessage** | сообщение, отображаемое при массовом удалении записей; можно сослаться на [строку локализации](../system/localization.md).
**noRecordsDeletedMessage** | сообщение, отображаемое при запуске массового удаления без фактического удаления; можно сослаться на [строку локализации](../system/localization.md).
**recordsPerPage** | количество записей на страницу, используйте 0, чтобы отключить пагинацию. Значение по умолчанию: `0`.
**perPageOptions** | варианты количества записей на страницу. Значение по умолчанию: `[20, 40, 80, 100, 120]`.
**showPageNumbers** | отображает номера страниц в постраничной навигации. Отключите, чтобы повысить производительность списка при работе с большими таблицами. Значение по умолчанию: `true`.
**toolbar** | ссылка на конфигурационный файл виджета Toolbar или массив с конфигурацией (см. ниже).
**showSorting** | отображает ссылку сортировки в каждой колонке. Значение по умолчанию: `true`.
**defaultSort** | задаёт колонку и направление сортировки по умолчанию, если пользовательское предпочтение не определено. Поддерживает строку или массив с ключами `column` и `direction`. Направление может быть `asc` (по возрастанию, по умолчанию) или `desc` (по убыванию).
**showCheckboxes** | отображает флажки рядом с каждой записью. Значение по умолчанию: `false`.
**showSetup** | показывает кнопку настройки колонок списка. Значение по умолчанию: `false`.
**structure** | включает структурированный список, подробности см. в статье [сортировка записей](./structures.md).
**customViewPath** | задаёт пользовательский путь к представлениям, переопределяющим partial‑файлы списка (необязательно).
**customPageName** | задаёт пользовательское имя параметра в URL страницы для пагинированных записей. Установите `false`, чтобы отключить хранение номера страницы в URL. Значение по умолчанию: `page`.

### Добавление панели инструментов

Чтобы добавить к списку панель инструментов, внесите следующую конфигурацию в YAML‑файл списка:

```yaml
toolbar:
    buttons: list_toolbar
    search:
        prompt: Find records
```

Конфигурация панели инструментов поддерживает следующие параметры:

Property | Description
------------- | -------------
**buttons** | ссылка на partial контроллера с кнопками панели инструментов, например: **_list_toolbar.htm**.
**search** | ссылка на конфигурационный файл виджета Search или массив с конфигурацией.

Конфигурация поиска поддерживает следующие параметры:

Property | Description
------------- | -------------
**prompt** | текст placeholder при отсутствии активного поиска; можно сослаться на [строку локализации](../system/localization.md).
**mode** | определяет стратегию поиска: содержать все слова, любое слово или точную фразу. Допустимые значения: `all`, `any`, `exact`. Значение по умолчанию: `all`.
**scope** | указывает метод [области запроса модели](../database/model.md), определённый в **модели списка**, который будет применён к поисковому запросу. Первый аргумент содержит объект запроса (как в обычном методе области), второй — поисковый термин, третий — массив колонок для поиска.
**searchOnEnter** | при значении true виджет поиска будет ждать нажатия клавиши Enter перед запуском поиска (по умолчанию поиск запускается автоматически после ввода текста и короткой паузы). Значение по умолчанию: `false`.

Partial панели инструментов, на который дана ссылка выше, должен содержать определение панели с кнопками. В partial также можно разместить [scoreboard control](https://octobercms.com/docs/ui/scoreboard) с графиками. Ниже пример partial‑файла панели с кнопкой **New Post**, ссылающейся на действие **create**, предоставляемое [поведением формы](forms.md):

```php
<div data-control="toolbar">
    <a href="<?= Backend::url('acme/blog/posts/create') ?>"
        class="btn btn-primary oc-icon-plus">
        New Post
    </a>
</div>
```

При использовании флажков списка можно переключать состояние кнопки при помощи атрибута `data-list-checked-trigger`.

```php
<button
    type="button"
    class="btn btn-primary"
    data-list-checked-trigger>
    Delete Selected
</button>
```

Можно передавать отмеченные значения в AJAX‑запрос с помощью атрибута `data-list-checked-request`.

```php
<button
    type="button"
    class="btn btn-primary"
    data-request="onDelete"
    data-list-checked-request>
    Delete Selected
</button>
```

### Фильтрация списка

Чтобы фильтровать список по пользовательскому вводу, добавьте в YAML‑файл следующую конфигурацию:

```yaml
filter: $/acme/blog/models/post/scopes.yaml
```

Свойство **filter** должно ссылаться на путь к [конфигурационному файлу фильтра](./filters.md) или содержать массив с настройками.

## Определение колонок списка

::: aside
Доступные параметры колонок списка описаны на странице [определения колонок списка](../../element/list-columns.md).
:::

Колонки списка определяются в YAML‑файле. Конфигурация колонок используется поведением списка для построения таблицы записей и отображения колонок модели в ячейках таблицы. Файл помещается в подкаталог каталога **models** плагина. Имя подкаталога совпадает с именем класса модели в нижнем регистре. Имя файла произвольное, но чаще всего используют **columns.yaml** или **list_columns.yaml**. Пример расположения файла с колонками списка:

::: dir
├── plugins
|   └── acme
|       └── blog
|           └── `models`
|               ├── post  _← Каталог конфигурации_
|               |   └── columns.yaml  _← Конфигурационный файл_
|               └── Post.php  _← Класс модели_
:::

Следующий пример демонстрирует типичное содержимое файла с определениями колонок списка.

```yaml
# columns.yaml
columns:
    name: Name
    email: Email
```

## Отображение списка

Обычно списки выводятся в файле [index‑представления](../system/views.md). Поскольку списки включают панель инструментов, представление будет содержать лишь вызов метода `listRender`.

```php
<?= $this->listRender() ?>
```

## Несколько определений списка

Поведение списка поддерживает несколько списков в одном контроллере при использовании именованных определений. Свойство `$listConfig` можно определить как массив, где ключ — имя определения, а значение — конфигурационный файл.

```php
public $listConfig = [
    'templates' => 'config_templates_list.yaml',
    'layouts' => 'config_layouts_list.yaml'
];
```

Каждое определение затем можно отобразить, передав имя определения первым аргументом при вызове метода `listRender`.

```php
<?= $this->listRender('templates') ?>
```

## Расширение поведения списка

Иногда требуется изменить стандартное поведение списка, и для этого существует несколько способов.

### Расширение конфигурации списка

Можно динамически расширить конфигурацию списка с помощью метода `listGetConfig`.

```php
public function listGetConfig($definition)
{
    $config = $this->asExtension('ListController')->listGetConfig($definition);

    // Реализовать структуру динамически
    $config->structure = [
        'showTree' => true
    ];

    return $config;
}
```

### Переопределение действия контроллера

Можно использовать собственную логику в методе действия `index` контроллера и при необходимости вызвать родительский метод `index` поведения List.

```php
public function index()
{
    //
    // Пользовательский код
    //

    // Вызов метода index() поведения ListController
    $this->asExtension('ListController')->index();
}
```

### Переопределение представлений

Поведение `ListController` имеет основное представление‑контейнер, которое можно переопределить, создав в каталоге контроллера специальный файл `_list_container.php`. В следующем примере к списку добавляется боковая панель:

```php
<?php if ($toolbar): ?>
    <?= $toolbar->render() ?>
<?php endif ?>

<?php if ($filter): ?>
    <?= $filter->render() ?>
<?php endif ?>

<div class="row row-flush">
    <div class="col-sm-3">
        [Insert sidebar here]
    </div>
    <div class="col-sm-9 list-with-sidebar">
        <?= $list->render() ?>
    </div>
</div>
```

Поведение создаёт виджет `Lists`, который также содержит множество представлений, доступных для переопределения. Это можно сделать, указав параметр `customViewPath`, описанный в настройках конфигурации списка. Виджет сперва ищет представление по указанному пути, затем использует расположение по умолчанию.

```yaml
# Custom view path
customViewPath: $/acme/blog/controllers/reviews/list
```

::: tip
Рекомендуется использовать подкаталог, например `list`, чтобы избежать конфликтов.
:::

Например, чтобы изменить разметку строки тела списка, создайте в каталоге контроллера файл `list/_list_body_row.php`.

```php
<tr>
    <?php foreach ($columns as $key => $column): ?>
        <td><?= $this->getColumnValue($record, $column) ?></td>
    <?php endforeach ?>
</tr>
```

### Расширение определений колонок

Можно расширить колонки другого контроллера извне, подписавшись на [глобальное событие](../services/event.md) `backend.list.extendColumns`. Обработчик события получает аргумент `$list`, представляющий объект `Backend\\Widgets\\Lists`, у которого можно использовать методы `getController` и `getModel` для проверки контекста выполнения.

Поскольку это событие потенциально влияет на все списки, важно убедиться, что контроллер и модель имеют нужный тип. В следующем примере метод `addColumns` добавляет новые колонки в список журнала событий и изменяет существующую колонку.

```php
Event::listen('backend.list.extendColumns', function($list) {
    if (
        !$list->getController() instanceof \\System\\Controllers\\EventLogs ||
        !$list->getModel() instanceof \\System\\Models\\EventLog
    ) {
        return;
    }

    // Добавить новую колонку
    $list->addColumns([
        'my_column' => [
            'label' => 'My Column'
        ]
    ]);

    // Изменить существующую колонку
    $list->getColumn('title')->useConfig([
        'path' => 'column_title'
    ]);
});
```

Также можно расширить колонки списка изнутри контроллера, переопределив метод `listExtendColumns`. Это повлияет только на список, используемый поведением `ListController`.

```php
class Categories extends \\Backend\\Classes\\Controller
{
    public $implement = [
        \\Backend\\Behaviors\\ListController::class
    ];

    public function listExtendColumns($list)
    {
        $list->addColumns([...]);

        $list->getColumn(...);
    }
}
```

Доступные методы объекта `$list` приведены ниже.

Method | Description
------------- | -------------
**addColumns** | добавляет новые колонки в список
**removeColumn** | удаляет колонку из списка
**getColumn** | возвращает определение существующей колонки

Каждый метод принимает массив колонок, аналогичный [конфигурации колонок списка](../../element/list-columns.md).

### Вставка CSS‑класса строки

Можно добавить пользовательский CSS‑класс строки, реализовав в контроллере метод `listInjectRowClass`. Метод принимает два аргумента: **$record** — отдельная запись модели, **$definition** — имя определения виджета List. Верните строку с нужными классами, которые будут добавлены к HTML‑разметке строки.

```php
class Lessons extends \\Backend\\Classes\\Controller
{
    // ...

    public function listInjectRowClass($lesson, $definition = null)
    {
        // Перечёркивать прошедшие занятия
        if ($lesson->lesson_date->lt(Carbon::today())) {
            return 'strike';
        }
    }
}
```

Существует специальный CSS‑класс `nolink`, который делает строку некликабельной, даже если заданы свойства `recordUrl` или `recordOnClick` виджета списка. Возврат этого класса в обработчике позволит запретить переход по записи, например для мягко удалённых или информационных строк:

```php
public function listInjectRowClass($record, $value)
{
    if ($record->trashed()) {
        return 'nolink';
    }
}
```

### Переопределение URL колонки

Можно указать действие при клике по записи колонки, переопределив метод `listOverrideRecordUrl`. Метод может вернуть строку с новым URL бэкенда или массив со сложным определением.

```php
public function listOverrideRecordUrl($record, $definition = null)
{
    if ($record->is_active) {
        return "acme/test/services/preview/{$record->id}";
    }
}
```

Чтобы переопределить поведение onclick, верните массив с ключом `onclick` и установите `url` в null.

```php
public function listOverrideRecordUrl($record, $definition = null)
{
    if ($record->is_banned) {
        return ['onclick' => "alert('Unable to click')", 'url' => null];
    }
}
```

Чтобы полностью сделать колонку некликабельной, верните массив с ключом `clickable`, установленным в false.

```php
public function listOverrideRecordUrl($record, $definition = null)
{
    if ($record->is_disabled) {
        return ['clickable' => false];
    }
}
```

### Расширение областей фильтра

Можно расширить области фильтра другого контроллера, подписавшись на [глобальное событие](../services/event.md) `backend.filter.extendScopes`. Обработчик принимает аргумент `$filter`, представляющий объект `Backend\\Widgets\\Filter`, у которого доступны методы `getController`, `getModel` и `getContext` для проверки контекста.

Поскольку событие потенциально влияет на все фильтры, важно убедиться, что контроллер и модель имеют нужный тип. В следующем примере метод `addScopes` добавляет новые поля в список журнала событий и настраивает CSS‑классы.

```php
Event::listen('backend.filter.extendScopes', function($filter) {
    if (
        !$filter->getController() instanceof \\System\\Controllers\\EventLogs ||
        !$filter->getModel() instanceof \\System\\Models\\EventLog
    ) {
        return;
    }

    // Добавить новую область
    $filter->addScopes([
        'my_scope' => [
            'label' => 'My Filter Scope'
        ]
    ]);

    // Добавить пользовательские CSS‑классы виджету фильтра
    $filter->cssClasses = array_merge(
        $filter->cssClasses,
        ['my-array', 'of-classes']
    );
});
```

Также можно расширить области фильтра внутри класса контроллера, переопределив метод `listFilterExtendScopes`.

```php
class Categories extends \Backend\Classes\Controller
{
    public $implement = [
        \Backend\Behaviors\ListController::class
    ];

    public function listFilterExtendScopes($filter)
    {
        $filter->addScopes([...]);
    }
}
```

У объекта `$filter` доступны следующие методы. Области соответствуют [конфигурации фильтра списка](./filters.md).

Method | Description
------------- | -------------
**addScopes** | добавляет новые области в виджет фильтра с помощью [конфигурации фильтра списка](./filters.md)
**removeScope** | удаляет область из виджета фильтра
**getScope** | возвращает определение существующей области

#### Расширение ответа фильтра

Метод `listExtendRefreshResults` позволяет дополнить AJAX‑ответ при обновлении списка и должен возвращать массив с дополнительными partial‑обновлениями. Метод `listGetFilterWidget` вернёт виджет фильтра для доступа к областям.

```php
public function listExtendRefreshResults($filter, $result)
{
    $statusCode = $this->listGetFilterWidget()->getScope('status_code')->value;

    return ['#my-partial-id' => $this->makePartial(...)];
}
```

### Расширение запроса модели

Запрос, получающий данные [модели базы данных](../database/model.md) для списка, можно расширить, переопределив метод `listExtendQuery` в контроллере. В следующем примере к запросу применяется область **withTrashed**, чтобы включить мягко удалённые записи.

```php
public function listExtendQuery($query)
{
    $query->withTrashed();
}
```

Если в контроллере используется несколько определений списка, можно воспользоваться вторым параметром `listExtendQuery`, содержащим имя определения.

```php
public $listConfig = [
    'inbox' => 'config_inbox_list.yaml',
    'trashed' => 'config_trashed_list.yaml'
];

public function listExtendQuery($query, $definition)
{
    if ($definition === 'trashed') {
        $query->onlyTrashed();
    }
}
```

Можно также присоединять другие таблицы для упрощения поиска и сортировки. В примере ниже выполняется соединение с таблицей `post_statuses` и добавляются в запрос колонки `status_sort_order` и `status_name`.

```php
public function listExtendQuery($query, $definition = null)
{
    $query->leftJoin('post_statuses', 'posts.status_id', 'post_statuses.id');

    $query->addSelect(
        'post_statuses.sort_order as status_sort_order',
        'post_statuses.name as status_name'
    );
}
```

Запрос модели [фильтра списка](./filters.md) также можно расширить, переопределив метод `listFilterExtendQuery`.

```php
public function listFilterExtendQuery($query, $scope)
{
    if ($scope->scopeName == 'status') {
        $query->where('status', '<>', 'all');
    }
}
```

### Расширение коллекции записей

Коллекцию записей, используемую списком, можно расширить, переопределив метод `listExtendRecords` в контроллере. В примере ниже метод `sort` [коллекции записей](../database/collection.md) меняет порядок сортировки.

```php
public function listExtendRecords($records)
{
    return $records->sort(function ($a, $b) {
        return $a->computedVal() > $b->computedVal();
    });
}
```

### Пользовательские типы колонок

Пользовательские типы колонок списка регистрируются в бэкенде методом `registerListColumnTypes` [регистрационного файла плагина](../extending.md). Метод должен возвращать массив, где ключ — имя типа, а значение — вызываемая функция. Функция получает три аргумента: исходное значение `$value`, объект определения `$column` и объект модели `$record`.

```php
public function registerListColumnTypes()
{
    return [
        // Локальный метод, например $this->evalUppercaseListColumn()
        'uppercase' => [$this, 'evalUppercaseListColumn'],

        // Встроенное замыкание
        'loveit' => function($value) { return "I love {$value}"; }
    ];
}

public function evalUppercaseListColumn($value, $column, $record)
{
    return strtoupper($value);
}
```

Использовать пользовательский тип колонки можно, указав его имя в параметре `type`.

```yaml
# columns.yaml
columns:
    secret_code:
        label: Secret code
        type: uppercase
```
