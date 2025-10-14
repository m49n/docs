---
subtitle: Фильтр Twig
---
# |media

Фильтр `|media` возвращает адрес относительно публичного пути [библиотеки диспетчера медиафайлов](../../cms/media/introduction.md). Результатом будет URL медиафайла, указанного в параметре фильтра.

```twig
<img src="{{ 'banner.jpg'|media }}" />
```

Если адрес диспетчера медиафайлов — __https://cdn.octobercms.com__, приведённый пример выведет следующее:

```html
<img src="https://cdn.octobercms.com/banner.jpg" />
```

## Интерфейс PHP

URL можно сформировать и в PHP с помощью класса `Media\Classes\MediaLibrary` и метода `url`.

```php
\Media\Classes\MediaLibrary::url('relative/path/to/asset.jpg');
```
