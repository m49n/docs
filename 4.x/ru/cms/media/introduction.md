# Введение

October CMS поставляется с встроенным менеджером медиафайлов (Media Manager), что упрощает публикацию крупных ресурсов, таких как видео и фотографии. Затем эти ресурсы можно вставлять в страницы и файлы контента через пользовательский интерфейс.

![image](https://github.com/octobercms/docs/blob/develop/images/media-manager.png?raw=true)

## Ссылки на медиафайлы

В большинстве случаев при вставке медиафайлов в контент используется полный URL. Однако можно сгенерировать эти URL на основе относительных путей в каталоге медиа, используя [фильтр Twig (Twig filter)](../../markup/filter/media.md) `|media`.

```twig
{{ 'relative/path/to/asset.jpg'|media }}
```

::: tip
См. статью о [фильтре Media](../../markup/filter/media.md), чтобы узнать подробности.
:::

## Параметры конфигурации

Для тонкой настройки менеджера медиафайлов предусмотрено несколько параметров, определённых в файле **config/media.php**.

```php
/*
|--------------------------------------------------------------------------
| Ignored Files and Patterns
|--------------------------------------------------------------------------
|
| The media manager wil ignore file names and patterns specified here
|
*/

'ignore_files' => ['.svn', '.git', '.DS_Store', '.AppleDouble'],

'ignore_patterns' => ['^\\..*'],
```

Конфигурация, задающая расположение медиафайлов, определяется в системном конфигурационном файле. См. [статью о провайдерах](./providers.md) по использованию сторонних провайдеров, таких как Amazon S3.

## Аудио- и видеоплееры

По умолчанию система использует теги HTML5 audio и video для вывода аудио- и видеофайлов:

```html
<video src="video.mp4" controls></video>
```

или

```html
<audio src="audio.mp3" controls></audio>
```

Это поведение можно переопределить. Если существуют частичные представления CMS (CMS partials) **oc-audio-player.htm** и **oc-video-player.htm**, они будут использоваться для отображения аудио- и видеоконтента. Внутри частичных представлений используйте переменную **src** для вывода ссылки на исходный файл. Пример:

```html
<video src="{{ src }}" width="320" height="200" controls preload></video>
```

Если не требуется использовать проигрыватель HTML5, можно добавить любую другую разметку в частичные представления. Существует [сторонний скрипт](https://html5media.info/), который включает поддержку тегов HTML5 video и audio в старых браузерах.

Так как частичные представления написаны на Twig, можно автоматизировать добавление альтернативных видеоресурсов на основе соглашения об именовании. Например, если для каждого видео в полном разрешении всегда существует версия с меньшим разрешением, а файл меньшего разрешения имеет расширение «iphone.mp4», сгенерированная разметка может выглядеть так:

```twig
<video controls>
    <source
        src="{{ src }}"
        media="only screen and (min-device-width: 568px)"></source>
    <source
        src="{{ src|replace({'.mp4': '.iphone.mp4'}) }}"
        media="only screen and (max-device-width: 568px)"></source>
</video>
```

## События

Менеджер медиафайлов предоставляет [несколько событий](../../extend/extending.md), на которые можно подписаться для расширения функциональности.

Событие | Описание | Параметры
------------- | ------------- | -------------
**folder.delete** | Вызывается при удалении папки | `(string) $path`
**file.delete** | Вызывается при удалении файла | `(string) $path`
**folder.rename** | Вызывается при переименовании папки | `(string) $originalPath`, `(string) $newPath`
**file.rename** | Вызывается при переименовании файла | `(string) $originalPath`, `(string) $newPath`
**folder.create** | Вызывается при создании папки | `(string) $newFolderPath`
**folder.move** | Вызывается при перемещении папки | `(string) $path`, `(string) $dest`
**file.move** | Вызывается при перемещении файла | `(string) $path`, `(string) $dest`
**file.upload** | Вызывается при загрузке файла | `(string) $filePath`, `(\\Symfony\\Component\\HttpFoundation\\File\\UploadedFile) $uploadedFile`

Чтобы подключиться к этим событиям, расширьте класс `Media\Widgets\MediaManager` напрямую.

```php
Media\Widgets\MediaManager::extend(function($widget) {
    $widget->bindEvent('file.rename', function ($originalPath, $newPath) {
        // Update custom references to path here
    });
});
```

Либо подпишитесь глобально через фасад (facade) `Event` (каждому событию предшествует префикс `media.`, а первым параметром будет передан созданный объект `Media\Widgets\MediaManager`).

```php
Event::listen('media.file.rename', function($widget, $originalPath, $newPath) {
    // Update custom references to path here
});
```
