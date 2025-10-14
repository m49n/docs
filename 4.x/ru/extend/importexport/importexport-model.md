---
subtitle: Узнайте, как настроить процесс импорта и экспорта.
---
# Модель импорта и экспорта

Модели импорта и экспорта определяют логику обработки действий импорта или экспорта и наследуются от `Backend\\Models\\ImportModel` и `Backend\\Models\\ExportModel` соответственно. Эти модели предназначены для использования с [контроллером импорта и экспорта](./importexport-controller.md), однако их можно вызывать напрямую из PHP.

## Модель импорта

Для импорта данных следует создать отдельную модель для этого процесса, расширяющую класс `Backend\\Models\\ImportModel`. Ниже пример определения класса:

```php
class SubscriberImport extends \Backend\Models\ImportModel
{
    /**
     * @var array rules to be applied to the data.
     */
    public $rules = [];

    public function importData($results, $sessionKey = null)
    {
        foreach ($results as $row => $data) {

            try {
                $subscriber = new Subscriber;
                $subscriber->fill($data);
                $subscriber->save();

                $this->logCreated();
            }
            catch (Exception $ex) {
                $this->logError($row, $ex->getMessage());
            }

        }
    }
}
```

Класс должен определять метод `importData`, выполняющий обработку импортированных данных. Первый параметр `$results` содержит массив данных для импорта. Второй параметр `$sessionKey` содержит ключ сессии запроса.

Method | Description
------------- | -------------
`logUpdated()` | Вызывается при обновлении записи.
`logCreated()` | Вызывается при создании записи.
`logError(rowIndex, message)` | Вызывается, если возникла проблема при импорте записи.
`logWarning(rowIndex, message)` | Используется для мягкого предупреждения, например при изменении значения.
`logSkipped(rowIndex, message)` | Используется, когда вся строка данных не была импортирована (пропущена).

### Импорт через PHP

Используйте метод `importFile`, чтобы выполнить импорт вручную из локального файла на диске.

```php
$importModel = new MyImportClass;

$importModel->file_format = 'json';

$importModel->importFile('/path/to/import/file.json');
```

Если файл получен из загруженного файла, воспользуйтесь фасадом `Input`, чтобы получить локальный путь.

```php
$importModel->importFile(
    Input::file('file')->getRealPath()
);
```

## Модель экспорта

Для экспорта данных следует создать отдельную модель, наследующую `Backend\\Models\\ExportModel`. Пример:

```php
class SubscriberExport extends \Backend\Models\ExportModel
{
    public function exportData($columns, $sessionKey = null)
    {
        $subscribers = Subscriber::all();

        $subscribers->each(function($subscriber) use ($columns) {
            $subscriber->addVisible($columns);
        });

        return $subscribers->toArray();
    }
}
```

Класс должен определять метод `exportData`, возвращающий данные экспорта. Первый параметр `$columns` содержит массив имён столбцов для экспорта. Второй параметр `$sessionKey` содержит ключ сессии запроса.

### Экспорт через PHP

Используйте метод `exportDownload`, чтобы выполнить экспорт вручную и вернуть ответ со скачиванием.

```php
$exportColumns = ['id', 'title'];

$exportModel = new MyExportClass;

$exportModel->file_format = 'json';

return $exportModel->exportDownload('myexportfile.json', ['columns' => $exportColumns]);
```

## Пользовательские опции

Формы импорта и экспорта поддерживают пользовательские опции, которые можно добавить через поля формы, определённые в параметре **form** конфигурации импорта или экспорта. Эти значения передаются в модель импорта или экспорта и доступны при обработке.

```yaml
# config_import_export.yaml
import:
    # ...
    form: $/acme/campaign/models/subscriberimport/fields.yaml

export:
    # ...
    form: $/acme/campaign/models/subscriberexport/fields.yaml
```

Указанные поля формы появятся на странице импорта или экспорта. Ниже пример содержимого файла `fields.yaml`:

```yaml
# fields.yaml
fields:

    auto_create_lists:
        label: Automatically create lists
        type: checkbox
        default: true
```

Значение поля формы **auto_create_lists** доступно как `$this->auto_create_lists` внутри метода `importData` модели импорта. В случае с моделью экспорта это значение доступно внутри метода `exportData`.

```php
class SubscriberImport extends \Backend\Models\ImportModel
{
    public function importData($results, $sessionKey = null)
    {
        if ($this->auto_create_lists) {
            // Do something
        }

        // ...
    }
}
```
