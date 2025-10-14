---
subtitle: Узнайте, как настраивать управление темами.
---
# Настройки темы

Каталог темы может содержать файлы **theme.yaml**, **version.yaml** и **assets/images/theme-preview.png**. Они необязательны для локальной разработки, но требуются для тем, публикуемых на Marketplace October CMS.

## Файл сведений о теме

Файл сведений о теме **theme.yaml** хранит описание темы, имя автора, URL сайта автора и другую информацию. Файл должен находиться в корневом каталоге темы:

::: dir
├── themes
|   └── website
|       ├── pages
|       ├── layouts
|       ├── partials
|       ├── content
|       ├── assets
|       └── `theme.yaml`  _← Файл сведений_
:::

В файле **theme.yaml** поддерживаются следующие поля:

Field | Description
------------- | -------------
**name** | имя темы, обязательно.
**author** | имя автора, обязательно.
**homepage** | URL сайта автора, обязательно.
**description** | описание темы, обязательно.
**previewImage** | собственное изображение предпросмотра, путь относительно каталога темы, например `assets/images/preview.png`, необязательно.
**code** | код темы, необязательно. Значение используется на Marketplace October CMS для инициализации кода темы.
**authorCode** | код автора темы, необязательно. Значение используется на Marketplace October CMS для указания владельца темы.
**form** | конфигурационный массив или ссылка на файл определения полей формы, используется для настройки темы, необязательно.
**require** | массив имён плагинов, от которых зависит тема, необязательно.

Пример файла сведений о теме:

```yaml
name: "October CMS Demo"
description: "Demonstrates the basic concepts of the front-end theming."
author: "October CMS"
homepage: "https://octobercms.com"
code: "Demo"
authorCode: "Acme"
```

## Файл версий

Файл версий темы **version.yaml** определяет текущую версию темы и журнал изменений. Он должен находиться в корневом каталоге темы.

::: dir
├── themes
|   └── website
|       ├── ...
|       └── theme.yaml
|       └── `version.yaml`  _← Файл версий_
:::

Файл имеет следующий формат.

```yaml
v1.0.1: Theme initialization
v1.0.2: Added more features
v1.0.3: Some features are removed
```

## Изображение предпросмотра темы

Изображение предпросмотра темы используется в селекторе тем бэкенда. Файл изображения **theme-preview.png** нужно разместить в каталоге **assets/images** темы:

::: dir
├── themes
|   └── website
|       ├── ...
|       └── assets
|           └── images
|               └── `theme-preview.png`  _← Изображение предпросмотра_
:::

Ширина изображения должна быть не менее 600 px. Идеальное соотношение сторон — 1,5, например 600×400 px.

## Зависимости темы

Тема может зависеть от плагинов, если определить опцию **require** в файле сведений о теме. Опция содержит массив имён плагинов, которые считаются обязательными. Тема, зависящая от **Acme.Blog** и **Acme.User**, может указать требование так:

```yaml
name: "October CMS Demo"
# [...]

require:
    - "Acme.User"
    - "Acme.Blog"
```

При первой установке темы система попытается установить необходимые плагины одновременно. Для удобства стоит также [добавить эти плагины в зависимости Composer](../../extend/resources/publishing-packages.md).

## Настройка темы

Темы могут поддерживать конфигурационные значения, если определить ключ `form` в файле сведений о теме. Ключ должен содержать конфигурационный массив или ссылку на файл определения полей формы; подробнее см. [определение полей формы](../../element/form-fields.md).

Ниже показано, как задать поле конфигурации имени сайта **site_name**:

```yaml
name: My Theme
# [...]

form:
    fields:
        site_name:
            label: Site name
            comment: The website name as it should appear on the front-end
            default: My Amazing Site!
```

Значение доступно в любом шаблоне темы через [глобальную переменную Twig](../../markup/property/this-theme.md) `this.theme`.

```twig
<h1>Welcome to {{ this.theme.site_name }}!</h1>
```

Конфигурацию можно вынести в отдельный файл, путь указывается относительно темы. Следующее определение загрузит поля формы из файла **config/fields.yaml** внутри темы.

```yaml
name: My Theme
# [...]

form: config/fields.yaml
```

**themes/demo/config/fields.yaml**:

```yaml
fields:
    site_name:
        label: Site name
        comment: The website name as it should appear on the front-end
        default: My Amazing Site!
```

### Использование данных темы в CSS

Иногда нужно учесть визуальное предпочтение в таблице стилей темы. Можно использовать пользовательские свойства CSS (переменные), чтобы сделать значения доступными. В примере ниже с помощью [поля выбора цвета](../../element/form/widget-colorpicker.md) задаётся цвет ссылок.

```yaml
form:
    fields:
        # [...]

        link_color:
            label: Link color
            type: colorpicker
```

Используя этот пример, можно создать [частичное представление CMS (partial)](./partials.md), которое передаст выбранное значение в CSS через локальную таблицу стилей. Частичное представление затем подключается в [макет темы (layout)](./layouts.md) внутри тега `<head>`.

```html
<style>
    :root {
        --my-color: {{ this.theme.link_color }};
    }
</style>
```

::: tip
Имена пользовательских свойств чувствительны к регистру, поэтому `--my-color` и `--My-color` считаются разными свойствами.
:::

Теперь в таблице стилей можно использовать пользовательское свойство в любом месте, передавая его имя в функцию `var()` вместо обычного значения.

```css
a {
    color: var(--my-color);
}
```

### Использование данных темы с комбинированными ресурсами

Ресурсы, комбинированные с помощью [фильтра и комбайнера](../markup/filter-theme.md) `|theme`, могут получать значения для поддерживающих фильтров, например для LESS. Просто укажите опцию `assetVar` при определении поля формы — в значении укажите имя нужной переменной.

```yaml
form:
    fields:
        # [...]

        link_color:
            label: Link color
            type: colorpicker
            assetVar: 'link-color'
```

В этом примере выбранное значение цвета будет доступно в файле Less как `@link-color`. Предположим, что есть следующая ссылка на таблицу стилей:

```twig
<link href="{{ ['assets/less/theme.less']|theme }}" rel="stylesheet">
```

Пример содержимого **themes/yourtheme/assets/less/theme.less**:

```less
a { color: @link-color }
```
