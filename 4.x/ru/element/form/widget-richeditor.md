---
subtitle: Виджет формы
shortname: Rich Editor / WYSIWYG
---
# Поле Rich Editor / WYSIWYG

Виджет формы `richeditor` выводит визуальный редактор для форматированного текста, также известный как WYSIWYG-редактор.

```yaml
html_content:
    type: richeditor
    label: Contents
```

Поддерживаются и часто используются следующие [свойства поля](../form-fields.md).

Свойство | Описание
------------- | -------------
**label** | имя, отображаемое пользователю.
**default** | задаёт строковое значение по умолчанию, необязательное.
**comment** | размещает описательный комментарий под полем.
**toolbarButtons** | кнопки, отображаемые на панели редактора. Например: `bold|italic`.
**size** | задаёт размер для полей, поддерживающих этот параметр (например, textarea). Варианты: `tiny`, `small`, `large`, `huge`, `giant`.
**showMargins** | включите `true`, чтобы добавить изменяемые границы документа. Значение по умолчанию — `false`.
**useLineBreaks** | использует перевод строки вместо обёрток в параграф для каждой новой строки. Значение по умолчанию — `false`.
**editorOptions** | пользовательские параметры редактора в виде массива (для продвинутой настройки).

Размер поля задаётся свойством `size`.

```yaml
html_content:
    type: richeditor
    label: Contents
    size: huge
```

Свойство `toolbarButtons` позволяет указать набор пользовательских кнопок.

```yaml
html_content:
    type: richeditor
    label: Contents
    toolbarButtons: bold|italic|underline
```

Двойной символ `||` добавляет разделитель между кнопками.

```yaml
toolbarButtons: bold|italic|underline||insertPageLink||undo|redo||clearFormatting
```

Доступные кнопки панели инструментов:

<div class="content-list" markdown="1">

- fullscreen
- bold
- italic
- underline
- strikeThrough
- subscript
- superscript
- fontFamily
- fontSize
- color
- emoticons
- inlineStyle
- paragraphStyle
- paragraphFormat
- align
- formatOL
- formatUL
- outdent
- indent
- quote
- insertHR
- insertLink
- insertPageLink
- insertImage
- insertVideo
- insertAudio
- insertFile
- insertTable
- insertSnippet
- undo
- redo
- clearFormatting
- selectAll
- html

</div>

::: tip
Символ `|` вставляет вертикальный разделитель на панели инструментов.
:::

## Регистрация пользовательской кнопки

Следующий JavaScript-код регистрирует пользовательскую кнопку как команду.

```js
oc.richEditorRegisterButton('insertCustomThing', {
    title: 'Insert Something',
    icon: '<i class="icon-star"></i>',
    undo: true,
    focus: true,
    refreshOnCallback: true,
    callback: function () {
        this.html.insert('<strong>My Custom Thing!</strong>');
    }
});
```

Затем добавьте кнопку в коллекцию по умолчанию.

```js
oc.richEditorButtons.splice(0, 0, 'insertCustomThing');
```

При регистрации JavaScript-кода нужно подключать его после ассетов Rich Editor. Для этого можно расширить конструктор класса `RichEditor`.

```php
\Backend\FormWidgets\RichEditor::extend(function($controller) {
    $controller->addJs('/plugins/october/test/assets/js/custom-button.js');
});
```

### Вызов модального окна из пользовательской кнопки

Используйте функцию JavaScript `oc.popup`, чтобы открыть модальное окно.

```js
oc.popup({
    handler: 'onLoadPopup'
});
```

Зарегистрируйте глобальный AJAX-обработчик через `backend.ajax.beforeRunHandler`. Метод `makePartial` можно вызвать для рендера частичного представления с содержимым модального окна.

```php
Event::listen('backend.ajax.beforeRunHandler', function ($controller, $handler) {
    if ($handler === 'onLoadPopup') {
        return $controller->makePartial('~/path/to/my/partials/_popup_form.php');
    }
});
```

## Расширенные параметры редактора

Свойство `editorOptions` позволяет настроить параметры редактора. Это продвинутая настройка, поскольку все указанные опции напрямую передаются в контрол редактора.

```yaml
html_content:
    type: richeditor
    editorOptions:
        imageDefaultWidth: 0
```

Ниже перечислены некоторые примеры параметров.

Параметр | Описание
------ | -----------
**imageDefaultWidth** | Задаёт ширину изображения по умолчанию при вставке в редактор. Значение `0` отключает установку ширины. Значение по умолчанию — `300`.
**imageDefaultAlign** | Задаёт выравнивание изображения по умолчанию при вставке в редактор. Возможные значения: `left`, `center`, `right`. Значение по умолчанию — `center`.
**imageDefaultDisplay** | Задаёт отображение изображения по умолчанию при вставке в текст. Возможные варианты: `inline` и `block`. Значение по умолчанию — `block`.
**imageResize** | Отключает изменение размера изображения при значении `false`. Значение по умолчанию — `true`.
**imagePaste** | Разрешает вставку изображений из буфера обмена. Значение по умолчанию — `true`.
