---
subtitle: Пункты меню для управления контентом в админ-панели.
---
# Определение навигации

В бэкенде записи по умолчанию отображаются в разделе Content, а глобальные настройки — в разделе Settings. Это поведение можно контролировать с помощью свойства **navigation** в файле чертежа. Пример ниже задаёт иконку и порядок отображения.

```yaml
navigation:
    icon: icon-pencil
    order: 200
```

Следующие свойства поддерживаются определениями `navigation` и `primaryNavigation`.

Property | Description
------------- | -------------
**label** | Задаёт ключ локализации метки меню, обязательное.
**order** | Числовой вес при определении порядка отображения.
**parent** | Привязывает пункт навигации к родительскому с помощью handle чертежа.
**icon** | Имя иконки из [набора иконок October CMS](../../element/available-icons.md), необязательное.
**iconSvg** | SVG-иконка вместо стандартной, должна быть прямоугольной и может содержать цвета, необязательное.

Чтобы поместить пункт в раздел Settings, установите **parent** в `settings`. Определение **category** может быть строкой или ссылкой на константу настроек, например `CATEGORY_COLLECTIONS`.

```yaml
navigation:
    parent: settings
    category: Collections
```

Чтобы разместить пункт в разделе Content, установите **parent** в `content`.

```yaml
navigation:
    parent: content
```

Чтобы сделать пункт элементом основной навигации, добавьте определение **primaryNavigation**.

```yaml
primaryNavigation:
    label: Blog
    icon: icon-copy
    order: 500

navigation:
    label: Main Menu Item
```

Чтобы добавить пункт вторичной навигации, свойство **parent** должно указывать UUID или handle элемента основной навигации.

```yaml
navigation:
    parent: <handle|uuid>
```

Чтобы отключить вторичную навигацию, определите **primaryNavigation** для одного чертежа, не делая его родителем других чертежей.

```yaml
primaryNavigation:
    label: Page
    icon: icon-magic
    order: 500
```

Можно полностью отключить навигацию, указав свойство **navigation** со значением `false`.

```yaml
navigation: false
```

## Дополнительная навигация

Используйте свойство `extraNavigation`, чтобы зарегистрировать пользовательские пункты навигации, добавляемые к чертежу. Значение — это массив, соответствующий определению `sideMenu` из [спецификации навигации бэкенда](../../extend/backend/navigation.md). В примере ниже добавляются секция и разделитель с пользовательскими типами отображения, порядок задаётся свойством `order`.

```yaml
navigation:
    label: Authors
    parent: Blog\Post
    icon: icon-user
    order: 230

extraNavigation:
    _authors_section:
        itemType: section
        label: Authors
        order: 210

    _authors_ruler:
        itemType: ruler
        order: 220
```

Можно регистрировать ссылки на [контроллеры, добавленные плагинами](../../extend/system/controllers.md), указав свойство `url`. Значение должно содержать URL контроллера; ниже показана ссылка на контроллер **acme/blog/posts**.

```yaml
navigation:
    label: Authors
    # ...

extraNavigation:
    testimonials:
        label: Testimonials
        order: 210
        icon: icon-group
        url: acme/blog/posts
```

Чтобы задать контекст навигации внутри контроллера, используйте метод `setTailorContext` фасада (facade) `BackendMenu`. Также можно указать UUID чертежа методом `setTailorContextUuid`. Метод принимает handle или `uuid` чертежа (первый аргумент) и ключ элемента дополнительной навигации (второй аргумент).

```php
BackendMenu::setTailorContext('Blog\Post', 'testimonials');

BackendMenu::setTailorContextUuid('edcd102e-0525-4e4d-b07e-633ae6c18db6', 'testimonials');
```

#### См. также

::: also
* [Навигация бэкенда](../../extend/backend/navigation.md)
:::
