---
subtitle: Свойство Twig
---
# this.layout

Текущий объект макета доступен через `this.layout`; он возвращает объект `Cms\Classes\Layout`.

## Свойства

У `this.layout` есть следующие свойства.

### id

Преобразует имя файла макета и каталог в идентификатор, пригодный для CSS.

```twig
<body class="layout-{{ this.layout.id }}">
```

Если файл макета — **default.htm**, будет создан класс `layout-default`.

### description

Описание макета из конфигурации.

```twig
<meta name="description" content="{{ this.layout.description }}">
```
