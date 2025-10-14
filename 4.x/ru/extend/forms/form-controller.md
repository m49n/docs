---
subtitle: Добавляет возможности управления формами на любую страницу бэкенда.
---
# Контроллер формы

Класс `Backend\\Behaviors\\FormController` — это поведение контроллера, которое упрощает добавление работы с формами на страницу бэкенда. Поведение предоставляет три страницы: Create, Update и Preview. Страница Preview — это версия страницы Update только для чтения. При использовании поведения формы нет необходимости определять действия `create`, `update` и `preview` в контроллере — поведение делает это за вас. Однако нужно добавить соответствующие файлы представлений.

Поведение формы зависит от [определений полей формы](../../element/form-fields.md) и [класса модели](../database/model.md). Чтобы использовать поведение формы, необходимо добавить его в свойство `$implement` класса контроллера. Также следует определить свойство класса `$formConfig`; его значение должно указывать на YAML‑файл с параметрами поведения.

```php
namespace Acme\\Blog\\Controllers;

class Categories extends \\Backend\\Classes\\Controller
{
    public $implement = [
        \\Backend\\Behaviors\\FormController::class
    ];

    public $formConfig = 'config_form.yaml';
}
```

::: tip
Часто контроллер формы используют вместе с [контроллером списка](../lists/list-controller.md) в одном контроллере.
:::

## Настройка поведения формы

Конфигурационный файл, указанный в свойстве `$formConfig`, описывается в формате YAML. Файл необходимо поместить в [каталог представлений контроллера](../system/views.md). Ниже приведён пример типового конфигурационного файла поведения формы.

```yaml
# config_form.yaml
name: Blog Category
form: $/acme/blog/models/post/fields.yaml
modelClass: Acme\\Blog\\Post

create:
    title: New Blog Post

update:
    title: Edit Blog Post

preview:
    title: View Blog Post
```

В конфигурационном файле формы обязательны следующие параметры.

Property | Description
------------- | -------------
**name** | имя объекта, с которым работает эта форма.
**form** | массив конфигурации или ссылка на файл с определениями полей формы, см. [Form fields](../../element/form-fields.md).
**modelClass** | имя класса модели; данные формы загружаются из этой модели и сохраняются в неё.

Параметры ниже являются необязательными. Укажите их, если нужно, чтобы поведение формы поддерживало страницы Create, Update или Preview.

Property | Description
------------- | -------------
**design** | отображает форму в указанном режиме оформления (см. ниже).
**defaultRedirect** | запасная страница перенаправления, если специальная страница не задана.
**create** | массив конфигурации или ссылка на файл конфигурации страницы Create.
**update** | массив конфигурации или ссылка на файл конфигурации страницы Update.
**preview** | массив конфигурации или ссылка на файл конфигурации страницы Preview.
**customMessages** | позволяет настроить сообщения, которые использует Form Controller.
**permissions** | накладывает ограничения на действия, предоставляемые Form Controller.

### Страница Create

Чтобы добавить поддержку страницы Create, укажите в YAML‑файле следующую конфигурацию.

```yaml
create:
    title: New Blog Post
    redirect: acme/blog/posts/update/:id
    redirectClose: acme/blog/posts
```

Страница Create поддерживает следующие параметры.

Property | Description
------------- | -------------
**title** | заголовок страницы; можно указать [строку локализации](../system/localization.md).
**redirect** | страница, на которую происходит переход после сохранения записи.
**redirectClose** | страница перехода после сохранения записи, если с запросом отправлена POST‑переменная **close**.
**form** | переопределяет набор полей формы только для страницы Create.

### Страница Update

Чтобы добавить поддержку страницы Update, укажите в YAML‑файле следующую конфигурацию.

```yaml
update:
    title: Edit Blog Post
    redirect: acme/blog/posts
```

Страница Update поддерживает следующие параметры.

Property | Description
------------- | -------------
**title** | заголовок страницы; можно указать [строку локализации](../system/localization.md).
**redirect** | страница, на которую происходит переход после сохранения записи.
**redirectClose** | страница перехода после сохранения записи, если с запросом отправлена POST‑переменная **close**.
**form** | переопределяет набор полей формы только для страницы Update.

### Страница Preview

Чтобы добавить поддержку страницы Preview, укажите в YAML‑файле следующую конфигурацию:

```yaml
preview:
    title: View Blog Post
```

Страница Preview поддерживает следующие параметры.

Property  | Description
------------- | -------------
**title** | заголовок страницы; можно указать [строку локализации](../system/localization.md).
**form** | переопределяет набор полей формы только для страницы Preview.

### Пользовательские сообщения

Укажите параметр `customMessages`, чтобы переопределить сообщения, которые по умолчанию использует Form Controller. Значения могут быть как простым текстом, так и [строкой локализации](../system/localization.md).

```yaml
customMessages:
    notFound: Did not find the thing
    flashCreate: New thing created
    flashUpdate: Updated that thing
    flashDelete: Thing is gone
```

Сообщения можно менять и в контексте отображаемой формы. В примере ниже сообщение `notFound` переопределяется только для контекста `update`.

```yaml
update:
    customMessages:
        notFound: Nothing found when updating
```

Доступные для переопределения сообщения перечислены ниже.

::: details Список доступных сообщений
Message | Default Message
------------- | -------------
**notFound** | Form record with an ID of :id could not be found.
**flashCreate** | :name Created
**flashUpdate** | :name Updated
**flashDelete** | :name Deleted
:::

### Ограничение правами доступа

Укажите параметр `permissions`, чтобы ограничить действия, предоставляемые Form Controller. Используйте [значения прав доступа](../backend/permissions.md), которыми должен обладать текущий пользователь бэкенда, чтобы поле было доступно. Поддерживается строка с одним правом или массив прав, из которых достаточно одного для выдачи доступа.

```yaml
permissions:
    modelCreate: admins.manage.create
    modelDelete: admins.manage.delete
```

Ниже перечислены параметры, которые можно задать как обязательные права доступа.

::: details Список доступных параметров
Message | Default Message
------------- | -------------
**modelCreate** | required to create new records.
**modelUpdate** | required to modify existing records.
**modelPreview** | required to preview existing records.
**modelDelete** | required to delete existing records.
:::

## Определение полей формы

::: aside
Доступные параметры полей формы описаны на странице [Form field definitions](../../element/form-fields.md).
:::

Поля формы определяются в YAML‑файле. Эту конфигурацию формы использует поведение, чтобы создавать элементы управления формы и связывать их с полями модели.

Файл помещается в подкаталог каталога **models** плагина. Название подкаталога совпадает с именем класса модели в нижнем регистре. Имя файла произвольное, но чаще всего используют **fields.yaml** и **form_fields.yaml**. Пример расположения файла с определениями полей формы:

::: dir
├── plugins
|   └── acme
|       └── blog
|           └── `models`
|               ├── post  _← Каталог конфигурации_
|               |   └── fields.yaml  _← Файл конфигурации_
|               └── Post.php  _← Класс модели_
:::

Поля можно разместить в трёх областях: **внешней области**, **основных вкладках** или **дополнительных вкладках**. Ниже показано типичное содержимое файла с определениями полей формы.

```yaml
# fields.yaml
fields:
    blog_title:
        label: Blog Title
        description: The title for this blog

    published_at:
        label: Published date
        description: When this blog post was published
        type: datepicker

    # [...]

tabs:
    fields:
        # [...]

secondaryTabs:
    fields:
        # [...]
```

## Представления формы

Для каждой поддерживаемой страницы формы — Create, Update и Preview — необходимо создать [файл представления](../backend/controllers-ajax.md) с соответствующим именем: **create.php**, **update.php** и **preview.php**.

Поведение формы добавляет в класс контроллера методы `formRender`, `formRenderDesign` и `formRenderPreview`. Эти методы выводят элементы управления формы, настроенные в описанном выше YAML‑файле.

### Представление Create

Файл **create.php** отвечает за страницу Create, позволяющую создавать новые записи. Типичная страница Create содержит «хлебные крошки», саму форму и кнопки формы. Атрибут **data-request** должен указывать на обработчик AJAX `onSave`, предоставляемый поведением формы. Ниже приведено типичное содержимое файла представления create.

```php
<?= Form::open(['class' => 'd-flex flex-column h-100']) ?>

    <div class="flex-grow-1">
        <?= $this->formRender() ?>
    </div>

    <div class="form-buttons">
        <div data-control="loader-container">
            <button
                type="button"
                data-request="onSave"
                data-request-data="{ close: true }"
                data-request-message="Creating Category..."
                data-hotkey="ctrl+enter, cmd+enter"
                class="btn btn-default">
                Create and Close
            </button>
            <span class="btn-text">
                or <a href="<?= Backend::url('acme/blog/categories') ?>">Cancel</a>
            </span>
        </div>
    </div>

<?= Form::close() ?>
```

Чтобы отслеживать несохранённые изменения и отображать предупреждение при переходе со страницы формы, добавьте атрибут `data-change-monitor` к открывающему тегу формы.

```php
<?= Form::open(['class' => '...', 'data-change-monitor' => true]) ?>
```

### Представление Update

Файл **update.php** отвечает за страницу Update, на которой можно обновлять или удалять существующие записи. Типичная страница Update содержит «хлебные крошки», форму и кнопки формы. Страница Update очень похожа на Create, но обычно содержит кнопку Delete. Атрибут **data-request** должен указывать на обработчик AJAX `onSave`, предоставляемый поведением формы. Ниже приведено типичное содержимое формы update.php.

```php
<?= Form::open(['class' => 'd-flex flex-column h-100']) ?>

    <div class="flex-grow-1">
        <?= $this->formRender() ?>
    </div>

    <div class="form-buttons">
        <div data-control="loader-container">
            <button
                type="button"
                data-request="onSave"
                data-request-data="{ close: true }"
                data-request-message="Saving Category..."
                data-hotkey="ctrl+enter, cmd+enter"
                class="btn btn-default">
                Save and Close
            </button>
            <button
                type="button"
                class="oc-icon-trash-o btn-icon danger pull-right"
                data-request="onDelete"
                data-request-message="Deleting Category..."
                data-request-confirm="Do you really want to delete this category?">
            </button>
            <span class="btn-text">
                or <a href="<?= Backend::url('acme/blog/categories') ?>">Cancel</a>
            </span>
        </div>
    </div>

<?= Form::close() ?>
```

### Представление Preview

Файл **preview.php** отвечает за страницу Preview, на которой можно просматривать существующие записи в режиме только для чтения. Типичная страница Preview содержит «хлебные крошки» и саму форму. Ниже приведено типичное содержимое формы preview.php.

```php
<div class="form-preview">
    <?= $this->formRenderPreview() ?>
</div>
```

## Оформление форм

Команда `create:controller` генерирует [контроллер](../system/controllers.md) и поддерживает опцию `--design`, позволяющую выбрать нужный режим отображения, как описано ниже.

```bash
php artisan create:controller Acme.Blog Posts --design=popup
```

Режимы оформления форм полезны, когда нужно отобразить форму без ручного управления HTML‑разметкой. Это менее гибко, но позволяет быстрее создавать формы.

```yaml
design:
    displayMode: basic
```

Свойство **design** в конфигурации поведения управляет отображением формы. Поддерживаются следующие параметры.

Property | Description
------------- | -------------
**displayMode** | задаёт режим отображения; поддерживаемые значения: `custom`, `basic`, `survey`, `sidebar`, `popup`. Значение по умолчанию: `basic`
**horizontalMode** | выводит поля формы в горизонтальном виде. Значение по умолчанию: `false`
**surveyMode** | отключает вкладки и отображает все поля на странице секциями с заголовками. Значение по умолчанию: `false`
**size** | размер контейнера страницы; поддерживаемые значения: шаг `50` в пределах от `400` до `1200`, `auto`. Значение по умолчанию: `auto`
**sidebarSize** | ширина боковой панели в режиме `sidebar`; поддерживаются шаги `50` в диапазоне `300`–`750`. Значение по умолчанию: `300`

Используйте метод `formRenderDesign`, чтобы вывести оформление формы внутри файлов представлений **create.php**, **update.php** и **preview.php**.

```php
<?= $this->formRenderDesign() ?>
```

### Режимы отображения

Если в конфигурации поведения используется параметр **design**, содержимое представления формируется с помощью стандартных шаблонов формы, предоставляемых системой.

Поддерживаются следующие значения **displayMode**.

Display Mode | Description
------------- | -------------
**custom** | вывод формы с помощью пользовательских файлов представлений (значение по умолчанию)
**basic** | базовый макет для стандартных форм
**survey** | макет опроса с секциями, расположенными столбцом
**sidebar** | макет с боковой панелью, где дополнительные вкладки отображаются в боковой панели
**popup** | содержимое формы отображается во всплывающих окнах

Свойство **size** определяет размер контейнера страницы или всплывающего окна.

```yaml
design:
    displayMode: survey
    size: 950
```

### Режим отображения popup

Если для **design** задан режим отображения `popup`, создавать файлы представлений вовсе не требуется. Все функции управления формой выводятся во всплывающем окне.

```yaml
design:
    displayMode: popup
    size: 750
```

При интеграции с [контроллером списка](../lists/list-controller.md) задайте свойству **recordOnClick** значение `popup`, чтобы при щелчке по записи открывать окно управления.

```yaml
# config_list.yaml
recordOnClick: popup
```

Свойство **recordOnClick** также поддерживает передачу контекста контроллеру формы, например, значение `popup@preview` откроет контекст preview.

```yaml
# config_list.yaml
recordOnClick: popup@preview
```

Страницу создания можно открыть через обработчик AJAX `onLoadPopupForm` вместе с элементом popup, как показано ниже.

```html
<button
    type="button"
    data-control="popup"
    data-handler="onLoadPopupForm"
    class="btn btn-primary">
    New Item
</button>
```

## Расширение поведения формы

Иногда возникает необходимость изменить стандартное поведение формы; существует несколько способов сделать это.

### Расширение конфигурации формы

Можно динамически расширить конфигурацию формы, переопределив метод `formGetConfig`.

```php
public function formGetConfig()
{
    $config = $this->asExtension('FormController')->formGetConfig();

    $config->form = $this->makeConfig($config->form);

    // Set the active tab dynamically
    $config->form->tabs['activeTab'] = 'Activities';

    return $config;
}
```

### Переопределение действия контроллера

Можно реализовать собственную логику для методов действий `create`, `update` или `preview` в контроллере и при необходимости вызвать родительский метод поведения формы.

```php
public function update($recordId, $context = null)
{
    //
    // Do any custom code here
    //

    // Call the FormController behavior update() method
    return $this->asExtension('FormController')->update($recordId, $context);
}
```

### Переопределение сохраняемых данных формы

Используйте перехват `formBeforeSave` (или аналогичный), чтобы изменить сохраняемые значения формы до сохранения или обновления записи. Для замены значения поля воспользуйтесь методом `formSetSaveValue(key, value)`.

```php
public function formBeforeSave($model)
{
    // When locale dropdown is set to "custom", override with the _custom_locale text field
    if (post('MyModel[locale]') === 'custom') {
        $this->formSetSaveValue('locale', post('MyModel[_custom_locale]'));
    }
}
```

### Переопределение перенаправления контроллера

Укажите URL‑адрес для перенаправления после сохранения модели, переопределив метод `formGetRedirectUrl`. Метод возвращает адрес, на который нужно перейти; относительные URL обрабатываются как адреса бэкенда.

```php
public function formGetRedirectUrl($context = null, $model = null)
{
    return 'https://octobercms.com';
}
```

### Расширение запроса модели формы

Запрос получения [модели базы данных](../database/model.md) для формы можно расширить, переопределив в контроллере метод `formExtendQuery`. Пример ниже позволяет находить и обновлять записи с мягким удалением, добавляя к запросу область **withTrashed**.

```php
public function formExtendQuery($query)
{
    $query->withTrashed();
}
```

### Расширение полей формы

Можно расширить поля другого контроллера извне, подписавшись на [глобальное событие](../services/event.md) `backend.form.extendFields`. Обработчик события получает аргумент `$form` — объект `Backend\\Widgets\\Form`, в котором можно использовать методы `getController`, `getModel` и `getContext`, чтобы проверить контекст выполнения.

Так как это событие потенциально влияет на все формы, важно убедиться, что контроллер и модель имеют нужный тип. Ниже приведён пример с методом `addFields`, добавляющим новые поля в форму настроек почты.

```php
Event::listen('backend.form.extendFields', function($form) {
    if (
        !$form->getController() instanceof \\System\\Controllers\\Settings ||
        !$form->getModel() instanceof \\System\\Models\\MailSetting
    ) {
        return;
    }

    $form->addFields([
        'my_field' => [
            'label' => 'My Field',
            'comment' => 'This is a custom field I have added.',
        ],
    ]);
});
```

Также можно расширить поля формы изнутри, переопределив метод `formExtendFields` в контроллере. Это повлияет только на форму, которую использует поведение `FormController`.

```php
class Categories extends \\Backend\\Classes\\Controller
{
    public $implement = [
        \\Backend\\Behaviors\\FormController::class
    ];

    public function formExtendFields($form)
    {
        $form->addFields([...]);
    }
}
```

Объект `$form` поддерживает следующие методы.

Method | Description
------------- | -------------
**addFields** | добавляет новые поля во внешнюю область
**addTabFields** | добавляет новые поля на вкладки
**addSecondaryTabFields** | добавляет новые поля на дополнительные вкладки
**removeField** | удаляет поле из любой области

Каждый метод принимает массив полей, аналогичный [конфигурации полей формы](../../element/form-fields.md).

### Фильтрация полей формы

Как описано в разделе [Field dependencies](./field-dependencies.md), фильтрацию полей формы можно реализовать через расширение, подписавшись на событие `form.filterFields`.

```php
User::extend(function ($model) {
    $model->bindEvent('model.form.filterFields', function ($formWidget, $fields, $context) use ($model) {
        if ($model->source_type === 'http') {
            $fields->source_url->hidden = false;
            $fields->git_branch->hidden = true;
        }
        elseif ($model->source_type === 'git') {
            $fields->source_url->hidden = false;
            $fields->git_branch->hidden = false;
        }
        else {
            $fields->source_url->hidden = true;
            $fields->git_branch->hidden = true;
        }
    });
});
```

## Проверка данных формы

Чтобы валидировать поля формы, можно использовать [трейты Validation](../database/traits.md) в модели.
