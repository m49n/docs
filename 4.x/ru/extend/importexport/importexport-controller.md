---
subtitle: Добавляет на страницу функции импорта и экспорта.
---
# Контроллер импорта и экспорта

Класс `Backend\\Behaviors\\ImportExportController` — поведение (behavior) контроллера, добавляющее функции импорта и экспорта данных. Поведение предоставляет две страницы — Import и Export. Страница Import позволяет загрузить CSV-файл и сопоставить столбцы с базой данных. Страница Export делает обратное и позволяет выгрузить столбцы из базы данных в CSV-файл. Поведение добавляет действия контроллера `import()` и `export()`.

Конфигурация поведения импорта и экспорта задаётся в двух частях, каждая из которых зависит от специального класса модели, а также определений полей списка и формы. Чтобы использовать поведение, добавьте его в свойство контроллера `$implement`. Также следует определить свойство класса `$importExportConfig` и указать в нём YAML-файл с настройками поведения.

```php
namespace Acme\Shop\Controllers;

class Products extends Controller
{
    public $implement = [
        \Backend\Behaviors\ImportExportController::class
    ];

    public $importExportConfig = 'config_import_export.yaml';

    // [...]
}
```

## Настройка поведения

Файл конфигурации, на который ссылается свойство `$importExportConfig`, описывается в формате YAML. Его следует разместить в [каталоге представлений](../system/views.md) контроллера. Ниже приведён пример такого файла.

```yaml
# config_import_export.yaml
import:
    title: Import Subscribers
    modelClass: Acme\Campaign\Models\SubscriberImport
    list: $/acme/campaign/models/subscriber/columns.yaml

export:
    title: Export Subscribers
    modelClass: Acme\Campaign\Models\SubscriberExport
    list: $/acme/campaign/models/subscriber/columns.yaml
```

Перечисленные ниже параметры конфигурации являются необязательными. Определите их, если требуется поддержка страницы импорта или экспорта, либо обеих сразу.

Property | Description
------------- | -------------
**defaultRedirect** | используется как запасная страница перенаправления, если конкретная страница не указана.
**import** | массив настроек или ссылка на конфигурационный файл для страницы Import.
**export** | массив настроек или ссылка на конфигурационный файл для страницы Export.
**defaultFormatOptions** | массив настроек или ссылка на конфигурационный файл с параметрами формата CSV по умолчанию.

### Страница Import

Чтобы включить страницу Import, добавьте следующую конфигурацию в YAML-файл.

```yaml
import:
    title: Import Subscribers
    modelClass: Acme\Campaign\Models\SubscriberImport
    list: $/acme/campaign/models/subscriberimport/columns.yaml
    redirect: acme/campaign/subscribers
```

Страница Import поддерживает следующие параметры конфигурации.

Property | Description
------------- | -------------
**title** | заголовок страницы; можно указать [строку локализации](../system/localization.md).
**list** | определяет столбцы списка, доступные для импорта.
**form** | предоставляет дополнительные поля с опциями импорта, необязательно.
**redirect** | страница перенаправления после завершения импорта, необязательно.
**permissions** | права доступа, необходимые для операции, необязательно.

### Страница Export

Чтобы включить страницу Export, добавьте следующую конфигурацию в YAML-файл.

```yaml
export:
    title: Export Subscribers
    modelClass: Acme\Campaign\Models\SubscriberExport
    list: $/acme/campaign/models/subscriberexport/columns.yaml
    redirect: acme/campaign/subscribers
```

Страница Export поддерживает следующие параметры конфигурации.

Property | Description
------------- | -------------
**title** | заголовок страницы; можно указать [строку локализации](../system/localization.md).
**fileName** | имя экспортируемого файла без расширения. По умолчанию `export`.
**list** | определяет столбцы списка, доступные для экспорта.
**form** | предоставляет дополнительные поля с опциями экспорта, необязательно.
**redirect** | страница перенаправления после завершения экспорта, необязательно.
**useList** | укажите `true` или имя определения списка, чтобы включить интеграцию со списками. По умолчанию `false`.

### Параметры формата

Чтобы переопределить параметры формата по умолчанию, добавьте в YAML-файл следующую конфигурацию:

```yaml
defaultFormatOptions:
    fileFormat: json
```

Ниже перечислены поддерживаемые параметры конфигурации формата (все необязательны) с указанием применимых типов формата.

Property | Description | Format
-------- | ----------- | ------
**fileFormat** | Формат файла: `json`, `csv` или `csv_custom`. По умолчанию `json`. |
**customJson** | Использовать пользовательский формат для типа `json`. | JSON
**firstRowTitles** | Первая строка содержит заголовки, только для импорта. | CSV
**delimiter** | Символ-разделитель. | CSV (Custom)
**enclosure** | Ограничивающий символ. | CSV (Custom)
**escape** | Символ экранирования. | CSV (Custom)
**encoding** | Кодировка файла, только для импорта. | CSV (Custom)

## Представления импорта и экспорта

Для каждой страницы — Import и Export — необходимо создать [файл представления](../system/views.md) с соответствующим именем: **import.htm** и **export.htm**.

Поведение импорта и экспорта добавляет в класс контроллера два метода: `importRender` и `exportRender`. Они выводят секции импорта и экспорта в соответствии с YAML-конфигурацией, описанной выше.

### Представление импорта

Представление **import.htm** описывает страницу Import, на которой выполняется импорт данных. Типичная страница содержит навигационные цепочки, сам блок импорта и кнопки отправки. Атрибут **data-request** должен ссылаться на обработчик AJAX `onImport`, предоставленный поведением. Ниже приведено типичное содержимое файла import.htm.

```php
<?= Form::open(['class' => 'layout']) ?>

    <div class="layout-row">
        <?= $this->importRender() ?>
    </div>

    <div class="form-buttons">
        <button
            type="submit"
            data-control="popup"
            data-handler="onImportLoadForm"
            data-keyboard="false"
            class="btn btn-primary">
            Импортировать записи
        </button>
    </div>

<?= Form::close() ?>
```

### Представление экспорта

Представление **export.htm** описывает страницу Export, на которой можно экспортировать файл из базы данных. Типичная страница содержит навигационные цепочки, сам блок экспорта и кнопки отправки. Атрибут **data-request** должен ссылаться на обработчик AJAX `onExport`, предоставленный поведением. Ниже приведено типичное содержимое формы export.htm.

```php
<?= Form::open(['class' => 'layout']) ?>

    <div class="layout-row">
        <?= $this->exportRender() ?>
    </div>

    <div class="form-buttons">
        <button
            type="submit"
            data-control="popup"
            data-handler="onExportLoadForm"
            data-keyboard="false"
            class="btn btn-primary">
            Экспортировать записи
        </button>
    </div>

<?= Form::close() ?>
```

## Интеграция с поведением списка

Существует альтернативный способ экспорта данных, который использует [поведение списка](../lists/list-controller.md) для предоставления данных экспорта. Чтобы задействовать эту возможность, добавьте `Backend\\Behaviors\\ListController` в массив `$implement` класса контроллера. Создавать представление экспорта не требуется — все настройки берутся из списка. Необходима только следующая конфигурация:

```yaml
export:
    useList: true
```

Затем добавьте кнопку экспорта на [панель инструментов списка](../lists/list-controller.md):

```php
<a
    href="<?= Backend::url('acme/campaign/subscribers/export') ?>"
    class="btn btn-default oc-icon-download">
    Экспортировать записи
</a>
```

Аналогично для кнопки импорта код будет таким:

```php
<a
    href="<?= Backend::url('acme/campaign/subscribers/import') ?>"
    class="btn btn-default oc-icon-upload">
    Импортировать записи
</a>
```

Если используется [несколько определений списка](../lists/list-controller.md), можно указать нужное определение.

```yaml
export:
    useList: orders
    fileName: orders.csv
```

Параметр `useList` поддерживает расширенные настройки.

```yaml
export:
    useList:
        definition: orders
        raw: true
```

Поддерживаются следующие параметры конфигурации:

Property | Description
------------- | -------------
**definition** | определение списка, из которого получать записи, необязательно.
**raw** | выводить необработанные значения атрибутов записи. По умолчанию `false`.
