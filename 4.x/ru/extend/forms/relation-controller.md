---
subtitle: Управляет вложенными данными формы с помощью связанных записей.
---
# Контроллер связей

Класс `Backend\\Behaviors\\RelationController` — это поведение контроллера, которое упрощает управление сложными [связями модели](../database/model.md) на странице.

Поведение связей опирается на типы связей, описанные ниже. Чтобы использовать его, добавьте определение `Backend\\Behaviors\\RelationController` в поле `$implement` класса контроллера. Также необходимо определить свойство класса `$relationConfig`; его значение должно указывать на YAML‑файл с параметрами поведения.

```php
namespace Acme\\Projects\\Controllers;

class Projects extends Controller
{
    public $implement = [
        \\Backend\\Behaviors\\FormController::class,
        \\Backend\\Behaviors\\RelationController::class
    ];

    public $formConfig = 'config_form.yaml';
    public $relationConfig = 'config_relation.yaml';
}
```

::: tip
Контроллер связей часто используют вместе с [контроллером формы](./form-controller.md).
:::

## Настройка поведения связей

Конфигурационный файл, указанный в свойстве `$relationConfig`, описывается в формате YAML. Файл следует поместить в [каталог представлений контроллера](../system/views.md). Требуемые параметры зависят от типа связи между целевой моделью и связанной моделью.

Первый уровень в конфигурационном файле поведения определяет имя связи в целевой модели. Например:

```php
class Invoice extends Model
{
    public $hasMany = [
        'items' => \\Acme\\Pay\\Models\\InvoiceItem::class,
    ];
}
```

Модель `Invoice` со связью `items` должна описать на первом уровне поле с таким же именем связи.

```yaml
# config_relation.yaml
items:
    label: Invoice Line Item
    view:
        list: $/acme/pay/models/invoiceitem/columns.yaml
        toolbarButtons: create|delete
        recordsPerPage: 10
    manage:
        form: $/acme/pay/models/invoiceitem/fields.yaml
```

Далее для каждого определения имени связи используются следующие параметры.

Property | Description
------------- | -------------
**label** | метка связи в единственном числе, обязательна.
**view** | параметры, относящиеся к контейнеру просмотра; см. ниже.
**manage** | параметры, относящиеся к всплывающему окну управления; см. ниже.
**pivot** | ссылка на файл с определениями полей формы; используется для связей с данными в промежуточной таблице.
**emptyMessage** | сообщение, выводимое, если связь пуста; необязательно.
**readOnly** | запрещает добавление, обновление, удаление или создание связей. Значение по умолчанию: `false`
**deferredBinding** | [откладывает все операции связывания, используя ключ сессии](../database/relations.md), когда он доступен. Значение по умолчанию: `false`
**popupSize** | изменяет размер всплывающих окон управления: `giant`, `huge`, `large`, `small`, `tiny` или `adaptive`. Значение по умолчанию: `huge`
**valueFrom** | задаёт пользовательский атрибут модели в качестве источника значения. По умолчанию используется имя определения.

Следующие параметры можно указывать в свойствах **view** или **manage**, если применимо к типу рендеринга (список, форма или оба варианта).

Property | Type | Description
------------- | ------------- | -------------
**form** | Form | ссылка на файл с определениями полей формы, см. [Backend form fields](../../element/form-fields.md).
**list** | List | ссылка на файл с определениями колонок списка, см. [Backend list columns](../../element/list-columns.md).
**showFlash** | Both | включает отображение всплывающих сообщений после успешной операции. Значение по умолчанию: `true`
**showSearch** | List | показывает поле ввода для поиска записей. Значение по умолчанию: `false`
**showSorting** | List | добавляет ссылку сортировки к каждой колонке. Значение по умолчанию: `true`
**showSetup** | List | отображает кнопку настройки для конфигурации колонок списка и числа записей на странице. Значение по умолчанию: `false`
**defaultSort** | List | задаёт колонку и направление сортировки по умолчанию, если пользовательские предпочтения не определены. Поддерживает строку или массив с ключами `column` и `direction`. Направление может быть `asc` (по возрастанию, значение по умолчанию) или `desc` (по убыванию).
**recordsPerPage** | List | максимальное количество строк на страницу.
**noRecordsMessage** | List | сообщение, выводимое при отсутствии записей; можно указать [строку локализации](../system/localization.md).
**conditions** | List | указывает необработанное выражение where для применения к запросу модели списка.
**scope** | List | указывает [область запроса модели](../database/model.md), определённую в связанной модели формы, которую следует всегда применять к запросу списка. Модель, к которой относится связь (родительская модель), передаётся в этот метод области вторым аргументом (первый — `$query`).
**searchMode** | List | определяет стратегию поиска: совпадение всех слов, любого слова или точной фразы. Поддерживаемые значения: `all`, `any`, `exact`. Значение по умолчанию: `all`.
**searchScope** | List | указывает [область запроса модели](../database/model.md), определённую в связанной модели формы, которую нужно применить к поисковому запросу; первым аргументом в область передаётся поисковый термин.
**filter** | List | ссылка на файл с определениями областей фильтра, см. [Backend list filters](../lists/filters.md).
**customPageName** | List | задаёт пользовательское имя переменной в URL для нумерации страниц. Укажите `false`, чтобы не сохранять номер страницы в URL.

Следующие параметры можно указывать только в свойстве **view**.

Property | Type | Description
------------- | ------------- | -------------
**showCheckboxes** | List | отображает флажки рядом с каждой записью.
**recordUrl** | List | связывает каждую запись списка с другой страницей. Например: **users/update/:id**. Значение `:id` заменяется идентификатором записи.
**customViewPath** | List | задаёт собственный путь к представлениям для переопределения частичных шаблонов списка.
**recordOnClick** | List | выполняет пользовательский JavaScript‑код при щелчке по записи.
**toolbarPartial** | Both | ссылка на файл частичного представления контроллера с кнопками тулбара, например `_relation_toolbar.php`. Переопределяет свойство `toolbarButtons`.
**toolbarButtons** | Both | набор кнопок, которые нужно отображать. Можно указать массив или строку со значениями через вертикальную черту либо `false`, чтобы скрыть кнопки. Доступные значения: `create`, `update`, `delete`, `add`, `remove`, `link`, `unlink`. Пример: `add|remove`.
**structure** | List | параметры для включения [сортировки записей](../lists/structures.md) в списке.

Следующие параметры можно указывать только в свойстве **manage**.

Property | Type | Description
------------- | ------------- | -------------
**title** | Both | заголовок всплывающего окна; можно указать [строку локализации](../system/localization.md).
**context** | Form | контекст отображаемой формы. Можно указать строку или массив с ключами `create`, `update`.

### Пользовательские сообщения

Укажите параметр `customMessages`, чтобы переопределить сообщения, которые по умолчанию использует Relation Controller. Значения могут быть простым текстом или [строкой локализации](../system/localization.md).

```yaml
customMessages:
    buttonCreate: Make Thing
    buttonDelete: Destroy Thing
```

Сообщения можно менять и в контексте отображаемого связанного поля. Пример ниже переопределяет сообщение `createButton` только для связи `items`.

```yaml
items:
    customMessages:
        buttonCreate: New Item!
```

Доступные для переопределения сообщения перечислены ниже.

::: details Список доступных сообщений
Message | Default Message
------------- | -------------
**buttonCreate** | Create :name
**buttonCreateForm** | Create
**buttonCancelForm** | Cancel
**buttonCloseForm** | Close
**buttonUpdate** | Update :name
**buttonUpdateForm** | Update
**buttonAdd** | Add :name
**buttonAddMany** | Add Selected
**buttonAddForm** | Add
**buttonLink** | Link :name
**buttonDelete** | Delete
**buttonDeleteMany** | Delete Selected
**buttonRemove** | Remove
**buttonRemoveMany** | Remove Selected
**buttonUnlink** | Unlink
**buttonUnlinkMany** | Unlink Selected
**confirmDelete** | Are you sure?
**confirmUnlink** | Are you sure?
**titlePreviewForm** | Preview :name
**titleCreateForm** | Create :name
**titleUpdateForm** | Update :name
**titleLinkForm** | Link a New :name
**titleAddForm** | Add a New :name
**titlePivotForm** | Related :name Data
**flashCreate** | :name Created
**flashUpdate** | :name Updated
**flashDelete** | :name Deleted
**flashAdd** | :name Added
**flashLink** | :name Linked
**flashRemove** | :name Removed
**flashUnlink** | :name Unlinked
:::

### Вложенные определения

Relation Controller поддерживает вложенные связи — то есть управление связями через другие связи. Вложенная связь использует стандартный синтаксис вложения полей. Например, определение `countries[cities]` позволяет управлять связью `cities` через связь `countries`.

```yaml
countries:
    label: Country
    form: $/acme/location/models/country/fields.yaml
    list: $/acme/location/models/country/columns.yaml

countries[cities]:
    label: City
    form: $/acme/location/models/city/fields.yaml
    list: $/acme/location/models/city/columns.yaml
```

::: tip
Вложенные определения связей идеально работают с [виджетом формы relation](../../element/form/widget-relation.md), если для него задано свойство `useController` со значением `true`.
:::

## Типы связей

Способ отображения менеджера связей зависит от типа связи, определённой в целевой модели. Тип связи также определяет требования к конфигурации, отмеченные **жирным**. Доступны следующие типы связей.

### Has Many

1. Связанные записи отображаются списком (`view.list`).
1. Щелчок по записи открывает форму обновления (`manage.form`).
1. Кнопка **Add** выводит список выбора (`manage.list`).
1. Кнопка **Create** открывает форму создания (`manage.form`).
1. Кнопка **Delete** удаляет записи.
1. Кнопка **Remove** разрывает связь.

Например, если у **Blog Post** много **Comments**, целевой моделью будет запись блога, а список комментариев выводится с колонками из определения `list`. Щелчок по комментарию открывает всплывающую форму с полями из `form` для обновления комментария. Комментарии также можно создавать тем же способом. Ниже показан пример конфигурационного файла поведения связи.

```yaml
# config_relation.yaml
comments:
    label: Comment
    manage:
        form: $/acme/blog/models/comment/fields.yaml
        list: $/acme/blog/models/comment/columns.yaml
    view:
        list: $/acme/blog/models/comment/columns.yaml
        toolbarButtons: create|delete
```

### Belongs to Many

1. Связанные записи отображаются списком (`view.list`).
1. Кнопка **Add** выводит список выбора (`manage.list`).
1. Кнопка **Create** открывает форму создания (`manage.form`).
1. Кнопка **Delete** удаляет записи из промежуточной таблицы.
1. Кнопка **Remove** разрывает связь.

Например, если **User** принадлежит многим **Roles**, целевой моделью будет пользователь, а список ролей выводится с колонками из определения `list`. Существующие роли можно добавить или удалить у пользователя. Ниже приведён пример конфигурационного файла поведения связи.

```yaml
# config_relation.yaml
roles:
    label: Role
    view:
        list: $/acme/user/models/role/columns.yaml
        toolbarButtons: add|remove
    manage:
        list: $/acme/user/models/role/columns.yaml
        form: $/acme/user/models/role/fields.yaml
```

### Belongs to Many (с данными в pivot)

1. Связанные записи отображаются списком (`view.list`).
1. Щелчок по записи открывает форму обновления (`pivot.form`).
1. Кнопка **Add** отображает список выбора (`manage.list`), затем форму ввода данных (`pivot.form`).
1. Кнопка **Remove** удаляет записи из промежуточной таблицы.

Продолжая пример со связью **Belongs to Many**, если у роли есть дата истечения, щелчок по роли откроет всплывающую форму с полями из `pivot` для обновления даты. Ниже приведён пример конфигурационного файла поведения связи.

```yaml
# config_relation.yaml
roles:
    label: Role
    view:
        list: $/acme/user/models/role/columns.yaml
    manage:
        list: $/acme/user/models/role/columns.yaml
    pivot:
        form: $/acme/user/models/role/fields.yaml
```

Данные pivot доступны при определении полей форм и колонок списков через связь `pivot`, как показано ниже.

```yaml
# config_relation.yaml
teams:
    label: Team
    view:
        list:
            columns:
                name:
                    label: Name
                pivot[team_color]:
                    label: Team color
    manage:
        list:
            columns:
                name:
                    label: Name
    pivot:
        form:
            fields:
                pivot[team_color]:
                    label: Team color
```

### Belongs To

1. Связанная запись отображается в виде формы предпросмотра (`view.form`).
1. Кнопка **Create** открывает форму создания (`manage.form`).
1. Кнопка **Update** открывает форму обновления (`manage.form`).
1. Кнопка **Link** выводит список выбора (`manage.list`).
1. Кнопка **Unlink** разрывает связь.
1. Кнопка **Delete** удаляет запись.

Например, если у **Phone** есть связь Belongs To с **Person**, менеджер связей выводит форму с полями из `form`. Кнопка Link откроет список людей, которых можно сопоставить с телефоном. Кнопка Unlink отменит привязку телефона к человеку.

```yaml
# config_relation.yaml
person:
    label: Person
    view:
        form: $/acme/user/models/person/fields.yaml
        toolbarButtons: link|unlink
    manage:
        form: $/acme/user/models/person/fields.yaml
        list: $/acme/user/models/person/columns.yaml
```

### Has One

1. Связанная запись отображается в виде формы предпросмотра (`view.form`).
1. Кнопка **Create** открывает форму создания (`manage.form`).
1. Кнопка **Update** открывает форму обновления (`manage.form`).
1. Кнопка **Link** выводит список выбора (`manage.list`).
1. Кнопка **Unlink** разрывает связь.
1. Кнопка **Delete** удаляет запись.

Например, если у **Person** есть связь Has One с **Phone**, менеджер связей выводит форму с полями из `form` для телефона. При щелчке на кнопку Update открывается всплывающая форма с доступными для редактирования полями. Если у пользователя уже есть телефон, поля обновляются; иначе создаётся новый телефон.

```yaml
# config_relation.yaml
phone:
    label: Phone
    view:
        form: $/acme/user/models/phone/fields.yaml
        toolbarButtons: update|delete
    manage:
        form: $/acme/user/models/phone/fields.yaml
        list: $/acme/user/models/phone/columns.yaml
```

## Отображение менеджера связей

Прежде чем управлять связями на какой-либо странице, необходимо инициализировать целевую модель в контроллере, вызвав метод `initRelation`.

```php
$post = Post::where('id', 7)->first();
$this->initRelation($post);
```

::: tip
[Контроллер формы](./form-controller.md) автоматически инициализирует модель в действиях create, update и preview.
:::

Затем менеджер связей можно вывести для указанного определения связи, вызвав метод `relationRender`. Например, если нужно отобразить менеджер связей на странице [Preview](./form-controller.md), содержимое представления **preview.htm** может выглядеть так.

```php
<?= $this->formRenderPreview() ?>

<?= $this->relationRender('comments') ?>
```

Можно указать менеджеру связей режим только для чтения, передав параметр во втором аргументе.

```php
<?= $this->relationRender('comments', ['readOnly' => true]) ?>
```

## Расширение поведения связей

Иногда требуется изменить стандартное поведение связей; для этого есть несколько подходов.

### Расширение конфигурации связей

Позволяет изменить конфигурацию связей. Следующий пример показывает, как подставить другой файл columns.yaml в зависимости от свойства модели.

```php
public function relationExtendConfig($config, $field, $model)
{
    // Make sure the model and field matches those you want to manipulate
    if (!$model instanceof MyModel || $field !== 'myField') {
        return;
    }

    // Show a different list for business customers
    if ($model->mode == 'b2b') {
        $config->view['list'] = '$/author/plugin_name/models/mymodel/b2b_columns.yaml';
    }
}
```

### Расширение виджета просмотра

Даёт возможность изменить виджет просмотра. Например, можно отключать showCheckboxes в зависимости от свойства модели.

```php
public function relationExtendViewWidget($widget, $field, $model)
{
    // Make sure the model and field matches those you want to manipulate
    if (!$model instanceof MyModel || $field !== 'myField') {
        return;
    }

    if ($model->constant) {
        $widget->showCheckboxes = false;
    }
}
```

#### Как удалить колонку

На этом этапе жизненного цикла виджет ещё не завершил инициализацию, поэтому вызвать `$widget->removeColumn()` нельзя. Метод `addColumns()`, описанный в [документации по контроллеру списка](../lists/list-controller.md), работает как ожидается, но для удаления колонки нужно подписаться на событие `list.extendColumns` внутри метода `relationExtendViewWidget()`. Ниже показано, как удалить колонку.

```php
public function relationExtendViewWidget($widget, $field, $model)
{
    // Make sure the model and field matches those you want to manipulate
    if (!$model instanceof MyModel || $field !== 'myField') {
        return;
    }

    // This will work
    $widget->bindEvent('list.extendColumns', function () use ($widget) {
        $widget->removeColumn('my_column');
    });
}
```

### Расширение виджета управления

Позволяет изменить виджет управления для связи.

```php
public function relationExtendManageWidget($widget, $field, $model)
{
    // Make sure the field is the expected one
    if ($field !== 'myField') {
        return;
    }

    // Manipulate widget as needed
}
```

### Расширение виджета pivot

Позволяет изменить виджет pivot для связи.

```php
public function relationExtendPivotWidget($widget, $field, $model)
{
    // Make sure the field is the expected one
    if ($field !== 'myField') {
        return;
    }

    // Manipulate widget as needed
}
```

### Расширение виджетов фильтрации

Существует два виджета фильтрации, которые можно расширять следующими методами: один для режима просмотра, другой — для режима управления `RelationController`.

```php
public function relationExtendViewFilterWidget($widget, $field, $model)
{
    // Extends the view filter widget
}

public function relationExtendManageFilterWidget($widget, $field, $model)
{
    // Extends the manage filter widget
}
```

Примеры программного добавления и удаления областей фильтра в этих виджетах см. в разделе **Extending filter scopes** [документации по контроллеру списка](../lists/list-controller.md).

### Расширение результатов обновления

Виджет просмотра обычно обновляется, когда виджет управления вносит изменения. С помощью этого метода можно добавить дополнительные контейнеры при обновлении. Верните массив с дополнительными значениями, которые нужно отправить в браузер, например:

```php
public function relationExtendRefreshResults($field)
{
    // Make sure the field is the expected one
    if ($field !== 'myField') {
        return;
    }

    return ['#myCounter' => 'Total records: 6'];
}
```

#### См. также

::: also
* [Relation Form Widget](../../element/form/widget-relation.md)
* [Entries Content Field](../../element/content/field-entries.md)
:::
