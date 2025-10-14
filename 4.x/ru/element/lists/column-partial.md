---
subtitle: Колонка списка
shortname: Partial
---
# Колонка Partial

Колонка `partial` рендерит содержимое с помощью частичного представления или view‑файла. Значение `path` может ссылаться на файл частичного представления, иначе имя колонки используется как имя partial. Путь по умолчанию ищется в директории представлений контроллера.

```yaml
content:
    label: Content
    type: partial
    path: content_column
```

Поддерживается следующее свойство.

Property | Description
------------- | -------------
**path** | путь к [файлу частичного представления](../../extend/system/views.md) или [коду шаблона view](../../extend/services/response-view.md); по умолчанию используется имя колонки с префиксом **column_**.

Если `path` указан как простое имя файла (без пути и расширения), исходный путь определяется в каталогах модели или контроллера. В примере ниже будут проверены файлы **../models/mymodel/_column_for_content.php** и **../controllers/mycontroller/_column_for_content.php**.

```yaml
content:
    type: partial
    path: column_for_content
```

Можно задать полный путь `path`, чтобы использовать partial из других мест — это удобно для повторного использования.

```yaml
content:
    label: Content
    type: partial
    path: $/acme/blog/partials/_content_column.php
```

Внутри partial доступны следующие переменные:

- `$value` — значение ячейки по умолчанию;
- `$record` — модель, связанная с ячейкой;
- `$column` — настроенный объект `Backend\Classes\ListColumn`.

Пример содержимого файла **_content_column.php**.

```php
<?php if ($record->is_active): ?>
    <?= e($value) ?>
<?php endif ?>
```

## Использование view‑шаблонов

Можно передать в `path` код шаблона, чтобы подключить шаблоны службы представлений внутри плагина. Следующий код будет находиться по пути **plugins/acme/blog/views/listcolumns/content.php**.

```yaml
content:
    label: Content
    type: partial
    path: acme.blog::listcolumns.content
```

::: tip
Путь должен содержать `::`, чтобы активировать службу представлений.
:::

#### См. также

::: also
* [Рендеринг представлений контроллера](../../extend/system/views.md)
* [Служба Response & View](../../extend/services/response-view.md)
:::
