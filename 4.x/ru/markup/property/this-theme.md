---
subtitle: Свойство Twig
---
# this.theme

Текущий объект темы доступен через `this.theme`; он возвращает объект `Cms\Classes\Theme`, представляющий [объект настройки темы](../../cms/themes/settings.md).

## Свойства

`this.theme` предоставляет прямой доступ к значениям полей формы, заданным настройками темы. Также доступны следующие свойства по умолчанию.

### id

Преобразует имя каталога темы в идентификатор, пригодный для CSS.

```twig
<body class="theme-{{ this.theme.id }}">
```

Если каталог темы называется **website**, будет сгенерирован класс `theme-website`.

### config

Массив со всеми значениями конфигурации темы из файла `theme.yaml`.

```twig
<meta name="description" content="{{ this.theme.config.description }}">
```
