---
subtitle: Простое управление загрузкой файлов.
---
# Загрузка файлов

October CMS предоставляет простые возможности для загрузки файлов через формы. Для максимальной производительности функция включается в AJAX-фреймворке вручную.

## Загрузка файлов

Чтобы разрешить загрузку файлов в форме, добавьте атрибут `data-request-files` к тегу формы. Ниже приведён минимальный пример.

```html
<form data-request="onUploadFiles" data-request-files>
    <div>
        <label>Single File</label>
        <input name="single_file" type="file">
    </div>

    <button data-attach-loading>
        Upload
    </button>
</form>
```

::: aside
Подробнее о доступных методах функции `files()` см. в статье [Request и Input](../../extend/services/request-input.md).
:::

Во [внутри AJAX-обработчика](../../ajax/handlers.md) используйте вспомогательную функцию `files()` для доступа к загруженному файлу и вызовите метод `store`, чтобы сохранить файл на [диске хранилища](../../extend/services/storage.md). В результате вы получите путь к сохранённому файлу на локальном диске.

Следующий пример сохраняет файл в каталоге **storage/app/userfiles** с автоматически сгенерированным именем.

```php
function onUploadFiles()
{
    $filePath = files('single_file')->store('userfiles');

    // ...

    Flash::success('File saved');
}
```

### Загрузка нескольких файлов

Если у поля загрузки есть атрибут `multiple`, функция `files()` вернёт массив.

```html
<div>
    <label>Multi File</label>
    <input name="multi_file[]" type="file" multiple>
</div>
```

```php
function onUploadFiles()
{
    $filePaths = [];

    foreach (files('multi_file') as $file) {
        $filePaths[] = $file->store('userfiles');
    }

    // ...

    Flash::success('File saved');
}
```

### Проверка загружаемых файлов

Аналогично [стандартной валидации форм](./validation.md), файлы можно проверять с помощью фасада `Request` и метода `validate`. Для проверки нескольких элементов используйте суффикс `.*`. Пример ниже удостоверяется, что файл — изображение размером до 1 МБ.

```php
function onUploadFiles()
{
    Request::validate([
        'single_file' => 'required|image|max:1024',
        'multi_file.*' => 'required|image|max:1024',
    ]);

    Flash::success('Files are valid!');
}
```

## Загрузка в модели

Если модель настроена на [использование прикреплённых файлов](../../extend/database/attachments.md), включая модели Tailor с [виджетом загрузки файлов](../../element/form/widget-fileupload.md), файлы можно сохранять напрямую на модель.

Самый простой способ — присвоить атрибут модели значению из функции `files()`. Поддерживаются одиночные и множественные загрузки.

```php
function onUploadFiles()
{
    $model = new MyModel;

    $model->avatar = files('single_file');

    $model->save();

    // ...

    Flash::success('File saved');
}
```

Также можно присваивать атрибуту объект `System\Models\File` напрямую в разных сценариях.

```php
$model->avatar = (new File)->fromFile('/path/to/somefile.jpg');

$model->avatar = (new File)->fromData('Some content', 'sometext.txt');

$model->avatar = (new File)->fromUrl('https://example.tld/path/to/avatar.jpg');
```

Подробнее о работе с файловыми вложениями моделей см. в статье [Прикреплённые файлы](../../extend/database/attachments.md).

#### См. также

::: also
* [Request и Input](../../extend/services/request-input.md)
* [Диски хранилища](../../extend/services/storage.md)
* [Прикреплённые файлы](../../extend/database/attachments.md)
:::
