---
subtitle: Изучите распространённые способы расширения October CMS.
---
# Методология расширения

## Расширение через регистрацию плагина

Практически во всех случаях расширение October CMS выполняется в регистрационном файле плагина, который по сути является [провайдером служб](https://laravel.com/docs/12.x/providers) Laravel. Регистрационный файл называется **Plugin.php** и находится в корневом каталоге плагина.

В регистрационном классе плагина доступны следующие методы расширения:

Method | Description
------------- | -------------
**register()** | вызывается при первой регистрации плагина, выполняется до `boot`.
**boot()** | вызывается непосредственно перед обработкой маршрута, выполняется после `register`.
**registerMarkupTags()** | регистрирует [дополнительные теги разметки](./twig-tags.md), которые можно использовать в CMS.
**registerComponents()** | регистрирует [компоненты CMS (component)](./cms-components.md), предоставляемые плагином.
**registerNavigation()** | регистрирует [элементы навигации бэкенда](./backend/navigation.md) для плагина.
**registerPermissions()** | регистрирует [разрешения бэкенда](./backend/permissions.md), используемые плагином.
**registerSettings()** | регистрирует [ссылки настроек бэкенда](./settings/settings.md), используемые плагином.
**registerFormWidgets()** | регистрирует [виджеты форм бэкенда](./forms/form-widgets.md), которые предоставляет плагин.
**registerReportWidgets()** | регистрирует [виджеты отчётов бэкенда](./backend/report-widgets.md), включая виджеты консоли.
**registerListColumnTypes()** | регистрирует [пользовательские типы колонок списка](./lists/list-controller.md), которые предоставляет плагин.
**registerMailTemplates()** | регистрирует [почтовые шаблоны представлений](./system/sending-mail.md), которые предоставляет плагин.
**registerMailLayouts()** | регистрирует [почтовые макеты представлений](./system/sending-mail.md), которые предоставляет плагин.
**registerMailPartials()** | регистрирует [почтовые частичные представления](./system/sending-mail.md), которые предоставляет плагин.
**registerSchedule()** | регистрирует [плановые задачи](./system/scheduling.md), выполняемые регулярно.
**registerContentFields()** | регистрирует [контент-поля](../extend/tailor-fields.md), используемые в схемах Tailor.

## Расширение с помощью событий

[Сервис Event](./services/event.md) — основной способ внедрять или изменять функциональность базовых классов или других плагинов. Чтобы использовать этот сервис в любом классе, добавьте `use Event` в начало PHP-файла (после объявления пространства имён), что импортирует фасад Event.

### Подписка на события

Чаще всего подписка на событие выполняется в методе `boot` регистрационного файла плагина. Например, при первой регистрации пользователя нужно добавить его в сторонний список рассылки — это можно сделать, подписавшись на глобальное событие `rainlab.user.register`.

```php
class Plugin extends PluginBase
{
    // ...

    public function boot()
    {
        Event::listen('rainlab.user.register', function ($user) {
            // Code to register $user->email to mailing list
        });
    }
}
```

То же самое можно реализовать, расширив конструктор модели и используя локальное событие.

```php
User::extend(function ($model) {
    $model->bindEvent('user.register', function () use ($model) {
        // Code to register $model->email to mailing list
    });
});
```

### Объявление и инициирование событий

Локальные события инициируются вызовом `fireEvent()` на экземпляре объекта, который использует трейт `October\Rain\Support\Traits\Emitter`. Поскольку локальные события испускаются только конкретным экземпляром, их необязательно помещать в пространство имён — вероятность конфликтов имён в локальном контексте минимальна.

```php
$this->fireEvent('post.beforePost', [$firstParam, $secondParam]);
```

Глобальные события инициируются вызовом `Event::fire()`. Такие события доступны во всём приложении, поэтому рекомендуется включать в их имя информацию о вендоре. Если автор плагина — ACME, а название плагина — Blog, то глобальные события плагина ACME.Blog следует начинать с префикса `acme.blog`.

```php
Event::fire('acme.blog.post.beforePost', [$firstParam, $secondParam]);
```

Если локальное и глобальное событие инициируются в одном месте, сначала вызывайте локальное событие, чтобы придать ему приоритет. Также глобальное событие должно передавать экземпляр объекта, на котором было вызвано локальное событие, в первом аргументе.

```php
$this->fireEvent('post.beforePost', [$firstParam, $secondParam]);
Event::fire('rainlab.blog.beforePost', [$this, $firstParam, $secondParam]);
```

После подписки на событие параметры будут доступны в обработчике. Например:

```php
// Global
Event::listen('acme.blog.post.beforePost', function ($post, $param1, $param2) {
    Log::info($post->name . 'posted. Parameters: ' . $param1 . ' ' . $param2);
});

// Local
$post->bindEvent('post.beforePost', function ($param1, $param2) use ($post) {
    Log::info($post->name . 'posted. Parameters: ' . $param1 . ' ' . $param2);
});
```

## Расширение представлений бэкенда

Иногда требуется разрешить расширение файла представления или частичного представления бэкенда, например панели инструментов. Это возможно с помощью метода `fireViewEvent`, доступного во всех контроллерах бэкенда.

Разместите в представлении следующий код:

```php
<div class="footer-area-extension">
    <?= $this->fireViewEvent('backend.auth.extendSigninView', [$firstParam]) ?>
</div>
```

Другие плагины смогут внедрять HTML в эту область, подписываясь на событие и возвращая нужную разметку.

```php
Event::listen('backend.auth.extendSigninView', function ($controller, $firstParam) {
    return '<a href="#">Sign in with Google!</a>';
});
```

::: tip
Первым параметром обработчика события всегда будет объект-вызвавший событие (контроллер).
:::

Пример выше выведет следующую разметку:

```html
<div class="footer-area-extension">
    <a href="#">Sign in with Google!</a>
</div>
```

## Примеры использования

Ниже приведены практические примеры того, как можно использовать события.

### Расширение модели пользователя

Этот пример модифицирует событие `model.getAttribute` модели `User`, подписываясь на локальное событие. Код размещается в методе `boot` регистрационного файла плагина. В обоих случаях при обращении к атрибуту `$model->foo` будет возвращаться значение **bar**.

```php
// Local event hook that affects all users
User::extend(function ($model) {
    $model->bindEvent('model.getAttribute', function ($attribute, $value) {
        if ($attribute === 'foo') {
            return 'bar';
        }
    });
});

// Double event hook that affects user #2 only
User::extend(function ($model) {
    $model->bindEvent('model.afterFetch', function () use ($model) {
        if ($model->id !== 2) {
            return;
        }

        $model->bindEvent('model.getAttribute', function ($attribute, $value) {
            if ($attribute === 'foo') {
                return 'bar';
            }
        });
    });
});
```

Чтобы добавить валидацию модели для новых полей, подпишитесь на событие `beforeValidate` и выбросьте исключение `ValidationException`.

```php
User::extend(function ($model) {
    $model->bindEvent('model.beforeValidate', function () use ($model) {
        if (!$model->billing_first_name) {
            throw new \ValidationException(['billing_first_name' => 'First name is required']);
        }
    });
});
```

### Расширение форм бэкенда

::: aside
Существует несколько способов расширения форм бэкенда, подробности в статье [контроллера форм](./forms/form-controller.md).
:::

В этом примере используется глобальное событие `backend.form.extendFields` виджета `Backend\Widget\Form` для добавления полей при редактировании пользователя. Обработчик события также регистрируется в методе `boot` регистрационного файла плагина.

```php
// Extend all backend form usage
Event::listen('backend.form.extendFields', function($widget) {
    // Only apply this listener when the Users controller is being used
    if (!$widget->getController() instanceof \RainLab\User\Controllers\Users) {
        return;
    }

    // Only apply this listener when the User model is being modified
    if (!$widget->model instanceof \RainLab\User\Models\User) {
        return;
    }

    // Only apply this listener when the Form widget in question is a root-level
    // Form widget (not a repeater, nestedform, etc)
    if ($widget->isNested) {
        return;
    }

    // Add an extra birthday field
    $widget->addFields([
        'birthday' => [
            'label' => 'Birthday',
            'comment' => 'Select the users birthday',
            'type' => 'datepicker'
        ]
    ]);

    // Remove a Surname field
    $widget->removeField('surname');
});
```

::: tip
Для добавления полей можно использовать событие `backend.form.extendFieldsBefore`.
:::

### Расширение списка бэкенда

Этот пример использует глобальное событие `backend.list.extendColumns` класса `Backend\Widget\Lists` и добавляет значения колонок при условии, что список используется для редактирования пользователя. Обработчик также регистрируется в методе `boot` регистрационного файла плагина.

```php
// Extend all backend list usage
Event::listen('backend.list.extendColumns', function ($widget) {
    // Only for the User controller
    if (!$widget->getController() instanceof \RainLab\User\Controllers\Users) {
        return;
    }

    // Only for the User model
    if (!$widget->model instanceof \RainLab\User\Models\User) {
        return;
    }

    // Add an extra birthday column
    $widget->addColumns([
        'birthday' => [
            'label' => 'Birthday'
        ],
    ]);

    // Remove a Surname column
    $widget->removeColumn('surname');
});
```

### Расширение компонента

В этом примере объявляется новое глобальное событие `rainlab.forum.topic.post` и локальное событие `topic.post` внутри компонента `Topic`. Всё происходит в [классе компонента](./cms-components.md).

```php
class Topic extends ComponentBase
{
    public function onPost()
    {
        // ...

        $this->fireEvent('topic.post', [$post, $postUrl]);

        Event::fire('rainlab.forum.topic.post', [$this, $post, $postUrl]);
    }
}
```

Ниже показано, как подписаться на это событие в рамках [жизненного цикла выполнения макета](../cms/themes/layouts.md). В результате в журнал будет записано сообщение при вызове обработчика `onPost` компонента `Topic` (выше).

::: cmstemplate
```ini
[topic]
slug = "{{ :slug }}"
```
```php
<?
function onInit()
{
    $this->topic->bindEvent('topic.post', function($post, $postUrl) {
        trace_log('A post has been submitted at '.$postUrl);
    });
}
?>
```
:::

#### См. также

::: also
* [Сервис событий](./services/event.md)
:::
