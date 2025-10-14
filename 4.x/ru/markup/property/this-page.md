---
subtitle: Свойство Twig
---
# this.page

Текущий объект страницы доступен через `this.page`; он возвращает объект `Cms\Classes\Page`. К нему также можно обратиться в [PHP-коде страницы](../../cms/themes/pages.md) как `$this->page`.

## Свойства

У `this.page` есть следующие свойства.

### layout

Содержит имя макета, используемого страницей (если задан). Не путать с `this.layout`.

```twig
{{ this.page.layout }}
```

### id

Преобразует имя файла страницы и каталог в идентификатор, пригодный для CSS.

```twig
<body class="page-{{ this.page.id }}">
```

Если файл страницы — **home/index.htm**, будет создан класс `page-home-index`.

### title

Заголовок страницы из конфигурации.

```twig
<h1>{{ this.page.title }}</h1>
```

### description

Описание страницы из конфигурации.

```twig
<p>{{ this.page.description }}</p>
```

### meta_title

Альтернативное поле `title`, обычно более информативное для SEO.

```twig
<title>{{ this.page.meta_title }}</title>
```

### meta_description

Альтернативное поле `description`, обычно более информативное для SEO.

```twig
<meta name="description" content="{{ this.page.meta_description }}">
```

### hidden

Скрытые страницы доступны только авторизованным пользователям бэкенда.

```twig
{% if this.page.hidden %}
    <p>Примечание для администраторов: мы работаем над этой страницей.</p>
{% endif %}
```

### fileName

Имя файла страницы в теме с расширением.

### baseFileName

Имя файла страницы в теме без расширения.
