---
subtitle: Form UI
shortname: Partial
---
# Элемент частичного представления (partial)

Элемент `partial` выводит частичное представление; значение `path` может ссылаться на файл частичного представления, иначе в качестве имени частичного представления используется имя поля.

```yaml
content:
    label: Content
    type: partial
    path: field_content
```

Поддерживаются следующие [свойства поля](../form-fields.md).

Property | Description
------------- | -------------
**path** | путь к [файлу частичного представления](../../extend/system/views.md) или [коду шаблона представления](../../extend/services/response-view.md); по умолчанию используется имя поля с префиксом **field_**.

Если `path` задан в виде неполного имени файла (без пути и расширения), исходный путь определяется как каталог модели или контроллера. В следующем примере будет выполнен поиск частичного представления по путям **../models/mymodel/_field_for_content.php** или **../controllers/mycontroller/_field_for_content.php**.

```yaml
content:
    type: partial
    path: field_for_content
```

Можно указать полностью определённый `path`, чтобы подключить частичные представления вне каталогов модели или контроллера. Это полезно для совместного использования частичных представлений между определениями.

```yaml
content:
    type: partial
    path: $/acme/blog/partials/_field_content.php
```

## Доступ к переменным

При рендеринге частичного представления доступны следующие переменные.

- `$value` — текущее значение поля, если оно найдено.
- `$model` — [модель](../../extend/system/models.md), используемая для поля.
- `$field` — настроенный объект класса `Backend\Classes\FormField`.

Ниже приведён пример содержимого файла **_field_content.php**.

```php
<?php if ($model->is_active): ?>
    <p><?= $field->label ?> is active</p>
<?php endif ?>
```

## Использование шаблонов представлений

В значение `path` можно передать код шаблона представления, чтобы обратиться к шаблонам сервиса представлений внутри плагина. Следующий код будет находиться по пути **plugins/acme/blog/views/formfields/content.php**.

```yaml
content:
    type: partial
    path: acme.blog::formfields.content
```

Частичное представление можно разместить и в каталоге приложения, например **app/views/formfields/content.php**.

```yaml
content:
    type: partial
    path: app::formfields.content
```

:::tip
Путь должен содержать символы `::`, чтобы активировать сервис представлений.
:::

#### См. также

::: also
* [Элемент подсказки (hint)](./ui-hint.md)
* [Рендеринг представлений контроллера](../../extend/system/views.md)
* [Сервис Response & View](../../extend/services/response-view.md)
:::
