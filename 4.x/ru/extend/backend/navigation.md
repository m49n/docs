---
subtitle: Узнайте, как добавлять новые пункты меню в панели бэкенда.
---
# Навигация

Плагины могут расширять меню бэкенда, переопределяя метод `registerNavigation` в [регистрационном файле плагина](../extending.md). В этом разделе показано, как добавить пункты меню в навигацию бэкенда. Ниже приведён пример регистрации пункта меню верхнего уровня с двумя подпунктами.

```php
public function registerNavigation()
{
    return [
        'blog' => [
            'label' => 'Blog',
            'url' => Backend::url('acme/blog/posts'),
            'icon' => 'icon-pencil',
            'permissions' => ['acme.blog.*'],
            'order' => 500,

            'sideMenu' => [
                'posts' => [
                    'label' => 'Posts',
                    'icon' => 'icon-copy',
                    'url' => Backend::url('acme/blog/posts'),
                    'permissions' => ['acme.blog.access_posts'],
                ],
                'categories' => [
                    'label' => 'Categories',
                    'icon' => 'icon-copy',
                    'url' => Backend::url('acme/blog/categories'),
                    'permissions' => ['acme.blog.access_categories'],
                ]
            ]
        ]
    ];
}
```

При регистрации навигации бэкенда можно использовать [строки локализации](../system/localization.md) для значений `label`. Навигацию также можно контролировать через значения `permissions`, которые соответствуют определённым [правам пользователей бэкенда](./permissions.md). Порядок отображения пунктов меню определяется значением `order`: чем больше число, тем позже пункт появится в списке, и наоборот.

Чтобы сделать видимыми подпункты, установите контекст навигации в [контроллере бэкенда](../system/controllers.md) методом `BackendMenu::setContext`. Это активирует родительский пункт меню и покажет дочерние пункты в боковом меню.

Property | Description
------------- | -------------
**label** | указывает ключ строки локализации для подписи пункта меню, обязательно.
**order** | числовой вес, определяющий порядок отображения.
**icon** | имя иконки из [набора иконок October CMS](../../element/available-icons.md), необязательно.
**iconSvg** | SVG-иконка, используемая вместо стандартной. Изображение должно быть прямоугольным и может содержать цвета, необязательно.
**url** | URL, на который должен вести пункт меню (например, `Backend::url('author/plugin/controller/action')`), обязательно.
**counter** | числовое значение, отображаемое рядом с иконкой меню. Значение может быть числом или вызываемым, возвращающим число, необязательно.
**counterLabel** | строка с описанием числового значения счётчика, необязательно.
**attributes** | ассоциативный массив атрибутов и значений, применяемых к пункту меню, необязательно.
**permissions** | массив прав, которыми должен обладать пользователь бэкенда, чтобы видеть пункт меню (прямая работа с URL по-прежнему требует отдельных проверок прав), необязательно.
**sideMenu** | массив подпунктов меню с той же конфигурацией, что и пункты верхнего уровня, необязательно.
**itemType** | задаёт тип отображения для подпункта меню. Поддерживаются: `primary`, `link`, `ruler`, `section`. Значение по умолчанию — `link`.
**visibleOn** | применяется к подпунктам меню и содержит список других пунктов. Если один из них активен, делает этот пункт видимым, необязательно.

Ниже указаны системные значения, которые не передаются при регистрации пунктов навигации.

Key | Description
------------- | -------------
**code** | строковое значение, выступающее уникальным идентификатором пункта меню.
**owner** | строковое значение, указывающее плагин или модуль-владельца пункта меню в формате «Author.Plugin».

## Счётчики навигации

Пункты навигации поддерживают указание счётчика, показывающего наличие элементов, требующих внимания. Эти свойства доступны как для пунктов верхнего уровня, так и для подпунктов. Используйте `counter` и `counterLabel`, чтобы вывести числовой счётчик.

```php
'blog' => [
    // ...
    'counter' => [\Author\Plugin\Classes\MyMenuCounterService::class, 'getCounterMethod'],
    'counterLabel' => 'Label describing a dynamic menu counter',
],
```

## Типы отображения пунктов

Подпункты меню поддерживают разные типы отображения через свойство `itemType`, включая элементы пользовательского интерфейса. Установите значение `section`, чтобы вывести раздел навигации. Хороший способ показать, что это UI-элемент, — добавить подчёркивание (`_`) в начало ключа.

```php
'_section1' => [
    'itemType' => 'section',
    'label' => 'Advanced',
],
```

Можно сочетать раздел с разделителем навигации, размещённым над ним. Установите значение `ruler`, чтобы вывести разделитель.

```php
'_ruler1' => [
    'itemType' => 'ruler',
],
```

Чтобы вывести призыв к действию, установите значение `primary`, и ссылка будет показана как основная кнопка.

```php
'people_create' => [
    'label' => 'New Person',
    'icon' => 'icon-plus',
    'url' => Backend::url('acme/blog/people/create'),
    'itemType' => 'primary',
],
```

Также можно указать несколько ссылок `primary` и выводить их условно при помощи свойства `visibleOn`. В следующем примере кнопка **New Person** появляется, когда активен подпункт **people**, а кнопка **New Post** — когда активен подпункт **posts**.

```php
'people_create' => [
    'label' => 'New Person',
    'icon' => 'icon-plus',
    'url' => Backend::url('acme/blog/people/create'),
    'itemType' => 'primary',
    'visibleOn' => 'people',
],
'post_create' => [
    'label' => 'New Post',
    'icon' => 'icon-plus',
    'url' => Backend::url('acme/blog/people/create'),
    'itemType' => 'primary',
    'visibleOn' => 'posts',
],
```

## Расширение меню бэкенда

[Слушатель события](../extending.md) `backend.menu.extendItems` позволяет изменять существующие пункты навигации после того, как система и плагины зарегистрировали собственные пункты. В обработчике события передаётся экземпляр `$manager`, поддерживающий следующие методы.

Method | Description
------------- | -------------
**addMainMenuItems($owner, $definitions)** | добавить или обновить определение пункта главного меню
**getMainMenuItem($owner, $code)** | получить существующее определение пункта главного меню
**removeMainMenuItem($owner, $code)** | удалить существующее определение пункта главного меню
**addSideMenuItems($owner, $code, $definitions)** | добавить или обновить определение подпункта
**getSideMenuItem($owner, $code, $sideCode)** | получить существующее определение подпункта
**removeSideMenuItem($owner, $code, $sideCode)** | удалить существующее определение подпункта

При получении пункта меню объект позволяет изменять его свойства через чейнинг методов. В следующем примере подпись пункта Editor заменяется на **Code Editor**.

```php
Event::listen('backend.menu.extendItems', function($manager) {
    $manager->getMainMenuItem('October.Editor', 'editor')->label('Code Editor');
});
```

Далее пример изменяет подпись на **News** и добавляет счётчик **9** для плагина Acme Blog.

```php
Event::listen('backend.menu.extendItems', function($manager) {
    $manager->getMainMenuItem('Acme.Blog', 'blog')
        ->getSideMenuItem('posts')
        ->label('News')
        ->counter(9);
});
```

Аналогично можно удалять пункты меню, используя то же событие. В следующем примере показано, как удалить все пункты, удалить один пункт и удалить несколько пунктов соответственно.

```php
Event::listen('backend.menu.extendItems', function($manager) {
    $manager->removeMainMenuItem('Acme.Blog', 'blog');

    $manager->removeSideMenuItem('Acme.Blog', 'blog', 'posts');

    $manager->removeSideMenuItems('Acme.Blog', 'blog', [
        'posts',
        'categories'
    ]);
});
```
