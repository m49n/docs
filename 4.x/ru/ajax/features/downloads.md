---
subtitle: Простое управление загрузкой файлов.
---
# Загрузка файлов

October CMS предоставляет простые возможности для отправки файлов в ответах. Для максимальной производительности функция включается в AJAX-фреймворке вручную.

## Кнопки загрузки

Чтобы разрешить загрузку файлов, добавьте атрибут `data-request-download` к тегу HTML-формы или кнопки. Ниже приведён минимальный пример загрузки файла.

```html
<button data-request="onExport" data-request-download>
    Download
</button>
```

Если атрибут `data-request-download="..."` задан на элементе-триггере, указанное в нём имя будет присвоено загружаемому файлу. Следующий пример создаёт файл с именем **data.csv**.

```html
<button data-request="onExport" data-request-download="data.csv">
    Download Document
</button>
```

Чтобы открыть файл в новом окне браузера (обычно для предпросмотра PDF), задайте атрибут `data-browser-target` со значением `_blank`.

```html
<button data-request="onExport" data-request-download data-browser-target="_blank">
    Open in New Window
</button>
```

::: tip
Если используется атрибут загрузки, ответ может содержать загрузку файла или [обычное AJAX-обновление](../../ajax/update-partials.md).
:::

## Ответы с загрузкой

Внутри [AJAX-обработчика](../../ajax/handlers.md) можно вернуть [ответ загрузки файла](../../extend/services/response-view.md) `Response`, где метод `download` принимает путь к файлу на локальном диске.

```php
public function onExport()
{
    return Response::download(base_path('app/files/installer.zip'));
}
```

Чтобы преобразовать строку в ответ с загрузкой без записи содержимого на диск, используйте метод `streamDownload`, принимающий callback (первый аргумент) и имя файла (второй аргумент).

```php
public function onExport()
{
    return Response::streamDownload(function() {
        echo 'CSV Contents...';
    }, 'export.csv');
}
```

Можно использовать [сервис хранилища](../../extend/services/storage.md) для скачивания файлов из медиатеки или любого диска. Воспользуйтесь фасадом `Storage` и методом `disk`, чтобы указать имя диска (первый аргумент), затем вызовите `download`, передав имя файла (первый аргумент).

```php
public function onExport()
{
    return Storage::disk('media')->download('archive.zip');
}
```

При работе с [прикреплёнными файлами модели](../../extend/database/attachments.md) можно вызвать метод `download` на объекте файла.

```php
public function onDownload()
{
    // ...

    return $model->avatar->download();
}
```

## Пример использования

Ниже показана страница CMS, которая позволяет скачать [прикреплённый файл модели](../../extend/database/attachments.md) с пользовательским именем. Она принимает параметры `id` и `disk_name` вложения для проверки файла, а затем возвращает ответ браузеру с именем, переданным в параметре `file_name` (необязательно).

::: cmstemplate
```ini
## pages/download-file.htm

title = "Download File"
url = "/download-file/:id/:disk_name/:file_name?"
layout = "default"
```
```php
function onStart()
{
    $file = System\Models\File::find($this->param('id'));
    if (!$file || !$file->isPublic()) {
        throw new NotFoundException;
    }

    if ($file->disk_name !== $this->param('disk_name')) {
        throw new NotFoundException;
    }

    $customFileName = $this->param('file_name');
    if ($customFileName) {
        $file->file_name = $customFileName;
    }

    return $file->download();
}
```
```twig
```
:::

Далее можно сослаться на эту страницу в Twig следующей разметкой, где переменная `file` — экземпляр `System\Models\File`.

```twig
{{ 'download-file'|page({
    id: file.id,
    disk_name: file.disk_name,
    file_name: 'my-custom-name.png'
}) }}
```

::: tip
Если нужно вывести файл встроенно, например изображение, используйте метод `$file->output()`.
:::

#### См. также

::: also
* [Ответы и представления](../../extend/services/response-view.md)
:::
