---
subtitle: Виджет формы
shortname: Page Finder
---
# Поле Page Finder

Виджет формы `pagefinder` выводит поле для выбора ссылки на страницу. При раскрытии поле открывает селектор для поиска страницы. Результатом выбора будет строка с типом и идентификатором.

```yaml
featured_page:
    label: Featured Page
    type: pagefinder
```

Выбранное значение сохраняется в следующем формате.

```
october://<TYPE>@link/<REFERENCE>?<PARAM>=<VALUE>
```

Поддерживаются и часто используются следующие [свойства поля](../form-fields.md).

Свойство | Описание
------------- | -------------
**label** | имя, отображаемое пользователю.
**default** | задаёт строковое значение по умолчанию, необязательное.
**comment** | размещает описательный комментарий под полем.
**singleMode** | разрешает выбирать только элементы, которые приводят к единственному URL. Значение по умолчанию — `false`.

::: tip
Отображайте значение `pagefinder` в списках, используя [тип столбца linkage](../lists/column-linkage.md).
:::

## Ссылки на страницы

Используйте [фильтр Twig `|link`](../../markup/filter/link.md), чтобы преобразовать значение виджета в URL.

```twig
{{ featured_page|link }}
```

Примените [фильтр Twig `|content`](../../markup/tag/content.md), чтобы обработать HTML-разметку и заменить все ссылки внутри содержимого.

```twig
{{ blog_html|content }}
```

## Создание новых типов страниц

Плагины могут расширять виджет Page Finder новыми типами страниц с помощью событий. Чаще всего на событие подписываются в методе `boot` файла регистрации плагина.

```php
public function boot()
{
    Event::listen('cms.pageLookup.listTypes', function() {
        // ...
    });

    Event::listen('cms.pageLookup.getTypeInfo', function($type) {
        // ...
    });

    Event::listen('cms.pageLookup.resolveItem', function($type, $item, $url, $theme) {
        // ...
    });
}
```

### Регистрация новых типов страниц

Обработчик события `cms.pageLookup.listTypes` возвращает список типов страниц, поддерживаемых плагином. Он должен вернуть ассоциативный массив, где коды типов являются ключами, а названия типов — значениями. Настоятельно рекомендуется включать имя плагина в коды типов, чтобы избежать конфликтов с другими поставщиками типов страниц. Например:

```php
Event::listen('cms.pageLookup.listTypes', function() {
    return [
        'blog-post' => 'Blog Post'
    ];
});
```

Для типов страниц, поддерживающих вложенные подэлементы, например ссылку на все записи блога, значение названия типа должно быть массивом, где последний элемент равен `true`. Это исключит тип из сценариев, когда можно выбрать только одну ссылку. Следующий пример помечает тип **blog-posts** как поддерживающий вложенность.

```php
Event::listen('cms.pageLookup.listTypes', function() {
    return [
        'blog-posts' => ['All Blog Posts', true]
    ];
});
```

### Возврат информации о типе страницы

Обработчик события `cms.pageLookup.getTypeInfo` возвращает подробную информацию о поддерживаемых типах страниц. Он получает один параметр — код типа страницы (один из кодов, зарегистрированных в обработчике `cms.pageLookup.listTypes`). В коде обработчика необходимо проверить, относится ли запрошенный тип к плагину. Обработчик должен вернуть ассоциативный массив следующего вида.

```php
Event::listen('cms.pageLookup.getTypeInfo', function($type) {
    if ($type == 'blog-post') {
        return [
            'references' => [
                11 => 'News',
                12 => 'Tutorials',
                13 => 'Philosophy',
            ],
            'cmsPages' => Page::withComponent('blogPosts')->all()
        ];
    }
});
```

#### References

Элемент `references` содержит список объектов, на которые может ссылаться страница. Например, тип страницы **Blog Category** возвращает список категорий блога. Некоторые объекты поддерживают вложенность, например полный список страниц. Другие — нет, например категории блога. Формат значения `references` зависит от того, есть ли у ссылок подэлементы. Для ссылок без подэлементов используйте следующий формат.

```php
'references' => [
    'item-key' => 'Item title'
]
```

Формат ссылок с подэлементами выглядит так.

```php
'references' => [
    'item-key' => [
        'title' => 'Item title',
        'items' => [...]
    ]
]
```

Следующий итератор можно использовать для генерации ссылок, когда модель имеет дочерние записи.

```php
$iterator = function($records) use (&$iterator) {
    $result = [];
    foreach ($records as $record) {
        if (!$record->children) {
            $result[$record->id] = $record->title;
        }
        else {
            $result[$record->id] = [
                'title' => $record->title,
                'items' => $iterator($record->children)
            ];
        }
    }
    return $result;
};

return ['references' => $iterator($records)];
```

#### CMS Pages

Элемент `cmsPages` содержит список страниц CMS, способных отображать объекты указанного типа. Например, для типа **Blog Category** список страниц включает страницы, содержащие компонент `blogPosts`. Этот компонент может выводить содержимое категории блога. Элемент `cmsPages` должен быть массивом объектов `Cms\Classes\Page`.

Следующий метод `withComponent` найдёт все страницы активной темы, использующие компонент `blogPosts`.

```php
'cmsPages' => Page::withComponent('blogPosts')->all();
```

Используйте `whereComponent`, чтобы найти все страницы с компонентом `section`, где свойство `handle` имеет значение **Your\Handle**.

```php
'cmsPages' => Page::whereComponent('section', 'handle', 'Your\Handle')->all();
```

Используйте `inTheme`, чтобы найти страницы в другой теме, передав её код.

```php
'cmsPages' => Page::inTheme('demo')->withComponent('blogPosts')->all();
```

### Разрешение ссылок на страницы

Когда виджет pagefinder генерирует ссылки, каждую ссылку должен **разрешить** плагин, предоставивший этот тип элемента. Процесс разрешения включает генерацию реального URL элемента, определение того, активен ли элемент, и генерацию подэлементов (если требуется).

Обработчик события `cms.pageLookup.resolveItem` разрешает информацию о странице и возвращает фактический URL элемента, заголовок, признак активности и подэлементы при их наличии. Обработчик получает четыре аргумента:

- `$type` — имя типа элемента. Плагины должны обрабатывать только те типы, которые они предоставляют, и игнорировать остальные.
- `$item` — объект элемента (`Cms\Models\PageLookupItem`). Он представляет конфигурацию, заданную пользователем, и содержит свойства `title`, `type`, `reference`, `cmsPage`.
- `$url` — текущий абсолютный URL в нижнем регистре. Всегда используйте хелпер `Url::to()` для генерации ссылок и сравнения с текущим URL.
- `$theme` — текущий объект темы (`Cms\Classes\Theme`).

Обработчик должен проверить соответствие `type` и вернуть массив.

```php
Event::listen('cms.pageLookup.resolveItem', function($type, $item, $url, $theme) {
    if ($type === 'blog-post') {
        return [...];
    }

    if ($item->type == 'all-blog-posts') {
        return [...];
    }
});
```

Элементы `url` и `isActive` обязательны для элементов, указывающих на конкретную страницу.

```php
return [
    'title' => 'Some Category',
    'url' => 'https://example.tld/blog/category/some-category',
    'isActive' => true
];
```

### Разрешение вложенных ссылок

Разрешённая ссылка также может возвращать несколько элементов. Например, тип **All Pages** не имеет конкретной страницы, поскольку может содержать несколько ссылок.

В таких случаях резолвер запрашивает подэлементы, когда у аргумента `$item` установлено свойство `nesting` со значением true.

```php
Event::listen('cms.pageLookup.resolveItem', function($type, $item, $url, $theme) {
    // Resolve item
    $result = [...];

    // Subitems requested
    if ($item->nesting) {
        $result['items'] = [...];
    }

    return $result;
});
```

Подэлементы необходимо перечислить в элементе `items`. Элемент `items` следует добавлять только для элементов, помеченных как вложенные.

```php
return [
    'url' => 'https://example.tld/blog/category/another-category',
    'isActive' => true,
    'items' => [
        [
            'title' => 'Another category',
            'url' => 'https://example.tld/blog/category/another-category',
            'isActive' => true
        ],
        [
            'title' => 'News',
            'url' => 'https://example.tld/blog/category/news',
            'isActive' => false
        ]
    ]
];
```

### Разрешение ссылок для других сайтов

Разрешённая ссылка может возвращать другие сайты, содержащие эту ссылку. В этом случае у аргумента `$item` может быть свойство `sites` со значением true. Здесь полезны фасады `Site` и `Cms`, см. пример ниже.

```php
Event::listen('cms.pageLookup.resolveItem', function($type, $item, $url, $theme) {
    // Resolve item
    $result = [...];

    $page = \Cms\Classes\Page::loadCached($theme, $item->reference);

    // Sites requested
    if ($item->sites) {
        $sites = [];
        if (Site::hasMultiSite()) {
            foreach (Site::listEnabled() as $site) {
                $url = Cms::siteUrl($page, $site, [
                    'id' => $record->id,
                    'slug' => $record->slug,
                    'fullslug' => $record->fullslug
                ]);

                $sites[] = [
                    'url' => $url,
                    'id' => $site->id,
                    'code' => $site->code,
                    'locale' => $site->hard_locale,
                ];
            }
        }
        $result['sites'] = $sites;
    }

    return $result;
});
```

Возвращаемый элемент должен содержать массив `sites` с объектами, описывающими сайты; у каждого объекта должно быть свойство `url`.

### Пример использования

Ниже приведён базовый пример разрешения URL страницы путём поиска модели и URL страницы с использованием класса `Cms\Classes\Controller` и метода `pageUrl`. Он также рекурсивно обрабатывает дочерние элементы через связь `children` модели, если это запрошено через `$item->nesting`.

```php
Event::listen('cms.pageLookup.resolveItem', function($type, $item, $url, $theme) {
    if ($type !== 'my-model') {
        return;
    }

    $model = MyModel::find($item->reference);
    if (!$model) {
        return;
    }

    $controller = new Controller($theme);

    $pageUrl = $controller->pageUrl($item->cmsPage, [
        'id' => $model->id,
        'slug' => $model->slug
    ]);

    $result = [
        'url' => $pageUrl,
        'isActive' => $pageUrl == $url,
        'title' => $model->title,
        'mtime' => $model->updated_at,
    ];

    if (!$item->nesting) {
        return $result;
    }

    $iterator = function($children) use (&$iterator, &$item, &$theme, $url, $controller, $model) {
        $branch = [];

        foreach ($children as $child) {
            $childUrl = $controller->pageUrl($item->cmsPage, [
                'id' => $model->id,
                'slug' => $model->slug
            ]);

            $childItem = [
                'url' => $childUrl,
                'isActive' => $childUrl == $url,
                'title' => $child->title,
                'mtime' => $child->updated_at,
            ];

            if ($child->children) {
                $childItem['items'] = $iterator($child->children);
            }

            $branch[] = $childItem;
        }

        return $branch;
    };

    $result['items'] = $iterator($model->children);

    return $result;
});
```

::: tip
Поскольку процесс разрешения выполняется при каждом рендеринге страницы фронтенда, по возможности имеет смысл кэшировать всю информацию, необходимую для разрешения элементов.
:::
