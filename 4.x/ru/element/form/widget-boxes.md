---
subtitle: Form Widget
shortname: Boxes
---
# Поле Boxes

Поле `boxes` выводит визуальный редактор блоков для создания страниц и работает как конструктор страниц на фронтенде.

::: tip
Это поле доступно после установки [плагина Boxes](https://octobercms.com/plugin/offline-boxes) с маркетплейса October CMS. После получения лицензии можно установить его командой ниже.

```bash
php artisan plugin:install OFFLINE.Boxes
```
:::

Чтобы показать Boxes Editor в форме Tailor в бэкенде, определите поле формы следующим образом:

```yaml
fields:
    boxes_content:
        label: Boxes Content
        span: adaptive  # Это гарантирует, что Boxes Editor корректно отображается в Tailor.
        type: boxes     # Загружает Boxes Editor.
```

На фронтенде затем можно использовать метод render поля, чтобы получить сгенерированный HTML-контент:

::: cmstemplate
```ini
[section yourSectionVar]
handle = "Your\\Handle"
```
```twig
{{ yourSectionVar.boxes_content.render|raw }}
```
:::

#### См. также

::: also
* [Страница плагина Boxes](https://octobercms.com/plugin/offline-boxes)
:::
