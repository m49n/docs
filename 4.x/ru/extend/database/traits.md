# Трейты

Трейты моделей используют для реализации повторяемой функциональности.

## Манипуляции с атрибутами

### Nullable

Атрибуты помеченные как nullable устанавливаются в `NULL`, если оставить их пустыми. Чтобы обнулять атрибуты в модели, подключите трейты `October\Rain\Database\Traits\Nullable` и объявите свойство `$nullable` со списком атрибутов.

```php
class Product extends Model
{
    use \October\Rain\Database\Traits\Nullable;

    protected $nullable = ['sku'];
}
```

### Hashable

Хэшируемые атрибуты преобразуются сразу при первом присвоении. Чтобы автоматически хэшировать значения, подключите трейты `October\Rain\Database\Traits\Hashable` и укажите атрибуты в массиве `$hashable`.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\Hashable;

    protected $hashable = ['password'];
}
```

### Purgeable

Атрибуты из списка purgeable не сохраняются при создании или обновлении модели. Подключите трейты `October\Rain\Database\Traits\Purgeable` и перечислите атрибуты в `$purgeable`.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\Purgeable;

    protected $purgeable = ['password_confirmation'];
}
```

Получить значение, очищенное после сохранения, можно методом `getOriginalPurgeValue`.

```php
return $user->getOriginalPurgeValue('password_confirmation');
```

Чтобы вернуть все очищенные значения, вызовите `restorePurgedValues`.

```php
$user->restorePurgedValues();
```

### Encryptable

Атрибуты с шифрованием работают схоже с `Hashable`: значения шифруются при записи и расшифровываются при чтении. Подключите трейты `October\Rain\Database\Traits\Encryptable` и перечислите атрибуты в `$encryptable`.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\Encryptable;

    protected $encryptable = ['api_key', 'api_secret'];
}
```

::: warning
Зашифрованные атрибуты несовместимы с [jsonable-атрибутами](../system/models.md).
:::

### Sluggable

ЧПУ — это человекопонятные коды, часто используемые в URL страниц. Чтобы автоматически генерировать уникальный слаг, подключите трейты `October\Rain\Database\Traits\Sluggable` и объявите свойство `$slugs`.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\Sluggable;

    protected $slugs = ['slug' => 'name'];
}
```

Свойство `$slugs` — это массив, где ключ задаёт столбец назначения, а значение определяет источник строки для генерации. В примере выше, если столбец `name` равен **Cheyenne**, перед созданием модели в `slug` будет записано **cheyenne**, **cheyenne-2**, **cheyenne-3** и т. д.

Чтобы сформировать слаг из нескольких источников, передайте массив значений:

```php
protected $slugs = [
    'slug' => ['first_name', 'last_name']
];
```

Слаг генерируется только при первом создании модели. Чтобы переопределить или отключить генерацию, присвойте значение атрибуту вручную:

```php
$user = new User;
$user->name = 'Remy';
$user->slug = 'custom-slug';
$user->save(); // слаг не будет сгенерирован
```

Для повторной генерации при обновлении модели вызовите `slugAttributes`:

```php
$user = User::find(1);
$user->slug = null;
$user->slugAttributes();
$user->save();
```

## Сортировка и изменение порядка

### Sortable

Модели с сортировкой сохраняют числовое значение в столбце `sort_order`, поддерживающее порядок элементов в коллекции. Добавить столбец можно миграцией через метод `integer`.

```php
Schema::table('users', function ($table) {
    $table->integer('sort_order')->default(0);
});
```

Чтобы хранить порядок, подключите трейты `October\Rain\Database\Traits\Sortable` и убедитесь, что в схеме есть соответствующий столбец.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\Sortable;
}
```

Имя столбца можно изменить, объявив константу `SORT_ORDER`.

```php
const SORT_ORDER = 'my_sort_order_column';
```

Метод `setSortableOrder` устанавливает порядок сразу для нескольких записей. Массив содержит идентификаторы моделей в нужном порядке.

```php
$user->setSortableOrder([3, 2, 1]);
```

Если сортируется подмножество записей, вторым аргументом можно передать массив значений порядка. В примере ниже столбцу присваиваются значения 100, 200 и 300.

```php
$user->setSortableOrder([3, 2, 1], [100, 200, 300]);
```

### Simple Tree

Простое древо использует столбец `parent_id` для связи родителя и потомков. Добавить столбец можно миграцией методом `integer`.

```php
Schema::table('categories', function ($table) {
    $table->integer('parent_id')->nullable()->unsigned();
});
```

Чтобы использовать простое дерево, подключите трейты `October\Rain\Database\Traits\SimpleTree`.

```php
class Category extends Model
{
    use \October\Rain\Database\Traits\SimpleTree;
}
```

Трейт автоматически добавляет [связи модели](./relations.md) `parent` и `children`, эквивалентные следующим определениям:

```php
public $belongsTo = [
    'parent' => [Category::class, 'key' => 'parent_id'],
];

public $hasMany = [
    'children' => [Category::class, 'key' => 'parent_id'],
];
```

Определять эти связи вручную не нужно, но имя столбца можно изменить, объявив константу `PARENT_ID`:

```php
const PARENT_ID = 'my_parent_column';
```

Коллекции моделей с этим трейтом возвращают тип `October\Rain\Database\TreeCollection`, добавляющий метод `toNested`. Чтобы построить структуру для всех уровней, выполните выборку с предзагруженными связями.

```php
Category::all()->toNested();
```

#### Отрисовка

Чтобы вывести все уровни элементов и их потомков, можно воспользоваться рекурсивной обработкой:

```twig
{% macro renderChildren(item) %}
    {% if item.children is not empty %}
        <ul>
            {% for child in item.children %}
                <li>{{ child.name }}{{ _self_.renderChildren(child)|raw }}</li>
            {% endfor %}
        </ul>
    {% endif %}
{% endmacro %}

{% import _self as nav %}
{{ nav.renderChildren(category)|raw }}
```

### Nested Tree

[Модель вложенных множеств](https://en.wikipedia.org/wiki/Nested_set_model) — продвинутая техника хранения иерархий с помощью столбцов `parent_id`, `nest_left`, `nest_right` и `nest_depth`. Добавьте их миграцией.

```php
Schema::table('categories', function ($table) {
    $table->integer('parent_id')->nullable()->unsigned();
    $table->integer('nest_left')->nullable();
    $table->integer('nest_right')->nullable();
    $table->integer('nest_depth')->nullable();
});
```

Чтобы использовать вложенное множество, подключите трейты `October\Rain\Database\Traits\NestedTree`. Все возможности `SimpleTree` при этом сохраняются.

```php
class Category extends Model
{
    use \October\Rain\Database\Traits\NestedTree;
}
```

#### Создание корневого узла

По умолчанию узлы создаются корневыми:

```php
$root = Category::create(['name' => 'Root category']);
```

Можно преобразовать существующий узел в корневой:

```php
$node->makeRoot();
```

Либо обнулить `parent_id`, что даст тот же результат.

```php
$node->parent_id = null;
$node->save();
```

#### Вставка узлов

Новые узлы можно добавлять через связь:

```php
$child1 = $root->children()->create(['name' => 'Child 1']);
```

Или использовать метод `makeChildOf` для существующих узлов:

```php
$child2 = Category::create(['name' => 'Child 2']);
$child2->makeChildOf($root);
```

#### Удаление узлов

При удалении узла методом `delete` будут удалены все его потомки. Обратите внимание, что события [модели](../system/models.md) для дочерних записей вызываться не будут.

```php
$child1->delete();
```

#### Получение уровня вложенности

Метод `getLevel` возвращает текущую глубину узла.

```php
// 0 для корня
$node->getLevel()
```

#### Перемещение узлов

Доступны методы для перемещения узлов:

- `moveLeft()`: найти левого соседа и переместить левее него;
- `moveRight()`: найти правого соседа и переместить правее него;
- `moveBefore($otherNode)`: переместить узел перед указанным;
- `moveAfter($otherNode)`: переместить узел после указанного;
- `makeChildOf($otherNode)`: сделать узел потомком указанного;
- `makeRoot()`: сделать узел корневым.

## Утилитарные возможности

### Валидация

Модели October CMS используют встроенный [класс Validator](../services/validation.md). Правила описываются в свойстве `$rules`, а модель должна подключать трейты `October\Rain\Database\Traits\Validation`.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\Validation;

    public $rules = [
        'name' => ['required', 'between:4,16'],
        'email' => ['required', 'email'],
        'password' => ['required', 'alpha_num', 'between:4,8', 'confirmed'],
        'password_confirmation' => ['required', 'alpha_num', 'between:4,8']
    ];
}
```

Можно использовать и [массивный синтаксис](../services/validation.md):

```php
public $rules = [
    'links.*.url' => ['required', 'url'],
    'links.*.anchor' => ['required']
];
```

Модели проходят валидацию автоматически при вызове `save`.

```php
$user = new User;
$user->name = 'Actual Person';
$user->email = 'a.person@example.tld';
$user->password = 'passw0rd';

// Возвращает false, если модель невалидна
$success = $user->save();
```

::: tip
Модель можно проверить в любой момент методом `validate`.
:::

#### Расширенные правила

Правило `unique` настраивается автоматически, таблицу указывать не нужно.

```php
public $rules = [
    'name' => ['unique'],
];
```

Правило `required` поддерживает модификаторы **create** и **update**, позволяющие применять его только при создании или обновлении. В примере ниже поле требуется, если модель ещё не существует.

```php
public $rules = [
    'password' => ['required:create'],
];
```

#### Получение ошибок валидации

Если валидация не прошла, к модели прикрепляется объект `Illuminate\Support\MessageBag` с сообщениями. Получить коллекцию можно методом `errors()` или свойством `$validationErrors`. Все сообщения возвращает `errors()->all()`, а сообщения по конкретному атрибуту — `validationErrors->get('attribute')`.

::: tip
Модель использует объект MessageBag, который предоставляет [удобный способ форматирования ошибок](../services/validation.md).
:::

#### Принудительное сохранение

Метод `forceSave` выполняет валидацию и сохраняет модель независимо от ошибок.

```php
$user = new User;

// Создаёт пользователя без учёта ошибок
$user->forceSave();
```

#### Пользовательские сообщения об ошибках

Как и в классе Validator, можно задавать собственные сообщения, используя [тот же синтаксис](../services/validation.md).

```php
class User extends Model
{
    public $customMessages = [
        'required' => 'The :attribute field is required.',
        // ...
    ];
}
```

К массивному синтаксису правил также можно добавить собственные сообщения.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\Validation;

    public $rules = [
        'links.*.url' => ['required', 'url'],
        'links.*.anchor' => ['required'],
    ];

    public $customMessages = [
        'links.*.url.required' => 'The url is required',
        'links.*.url.*' => 'The url needs to be a valid url',
        'links.*.anchor.required' => 'The anchor text is required',
    ];
}
```

В этом примере задаются сообщения для конкретных правил (например, `required`) и для всех остальных (`*`), как в правиле `url`.

#### Пользовательские имена атрибутов

Настроить отображаемые имена атрибутов можно через массив `$attributeNames`.

```php
class User extends Model
{
    public $attributeNames = [
        'email' => 'Email Address',
        // ...
    ];
}
```

#### Динамические правила

Правила можно задавать динамически, переопределив метод события модели `beforeValidate`. В примере ниже поля `latitude` и `longitude` становятся обязательными, если `is_remote` равно `false`.

```php
public function beforeValidate()
{
    if (!$this->is_remote) {
        $this->rules['latitude'] = 'required';
        $this->rules['longitude'] = 'required';
    }
}
```

#### Пользовательские правила

Можно создавать собственные правила, аналогично [созданию правил для Validator](../services/validation.md).

### Мягкое удаление

При мягком удалении запись не удаляется из базы, а получает отметку времени в `deleted_at`. Чтобы включить мягкое удаление, подключите трейты `October\Rain\Database\Traits\SoftDelete` и добавьте столбец в массив `$dates`.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\SoftDelete;

    protected $dates = ['deleted_at'];
}
```

Добавить столбец `deleted_at` можно в миграции методом `softDeletes`:

```php
Schema::table('posts', function ($table) {
    $table->softDeletes();
});
```

При вызове `delete` у модели будет заполнен `deleted_at`. Такие записи исключаются из выборок по умолчанию.

Проверить, была ли модель мягко удалена, можно методом `trashed`:

```php
if ($user->trashed()) {
    //
}
```

#### Запросы с мягко удалёнными моделями

##### Включение удалённых моделей

Чтобы добавить удалённые записи в результат, используйте метод `withTrashed`:

```php
$users = User::withTrashed()->where('account_id', 1)->get();
```

`withTrashed` работает и для [связей](./relations.md):

```php
$flight->history()->withTrashed()->get();
```

##### Только удалённые модели

Метод `onlyTrashed` вернёт **только** мягко удалённые записи:

```php
$users = User::onlyTrashed()->where('account_id', 1)->get();
```

##### Восстановление удалённых моделей

Чтобы вернуть мягко удалённую модель в активное состояние, вызовите `restore`:

```php
$user->restore();
```

Можно восстановить несколько записей одним запросом:

```php
// Восстановить одну запись
User::withTrashed()->where('account_id', 1)->restore();

// Восстановить связанные записи
$user->posts()->restore();
```

#### Полное удаление моделей

Чтобы окончательно удалить запись из базы, используйте метод `forceDelete`:

```php
// Полное удаление одной записи
$user->forceDelete();

// Полное удаление связанных записей
$user->posts()->forceDelete();
```

#### Мягкое удаление связей

Если у обеих моделей включено мягкое удаление, можно каскадировать событие, указав опцию `softDelete` в [определении связи](./relations.md). В примере ниже при мягком удалении пользователя его комментарии также будут помечены удалёнными.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\SoftDelete;

    public $hasMany = [
        'comments' => [\Acme\Blog\Models\Comment::class, 'softDelete' => true]
    ];
}
```

::: tip
Если у связанной модели нет трейта мягкого удаления, она будет удалена полностью (как при опции `delete`).
:::

При восстановлении основной модели будут восстановлены и связанные записи с опцией `softDelete`.

```php
// Восстановить пользователя и его комментарии
$user->restore();
```

#### Включение удалённых записей в связях

По умолчанию мягко удалённые записи не участвуют в выборках связей. Чтобы их включить, добавьте область `withTrashed` к запросу.

```php
class User extends Model
{
    public $hasMany = [
        'comments' => [\Acme\Blog\Models\Comment::class, 'scope' => 'withTrashed']
    ];
}
```

### Multisite

При включении мультисайта модели управляют только записями активного сайта. Активный сайт хранится в столбце `site_id`. Чтобы включить мультисайт, подключите трейты `October\Rain\Database\Traits\Multisite` и перечислите атрибуты для распространения в свойстве `$propagatable`.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\Multisite;

    protected $propagatable = ['api_code'];
}
```

::: tip
Свойство `$propagatable` обязательно для мультисайта, но можно оставить пустым, чтобы не распространять атрибуты.
:::

Добавьте столбцы `site_id` и при необходимости `site_root_id` в миграции:

```php
Schema::table('posts', function ($table) {
    $table->integer('site_id')->nullable()->index();
    $table->integer('site_root_id')->nullable()->index();
});
```

Теперь при создании записи она привязывается к активному сайту, а при переключении на другой сайт будет создана новая запись. При обновлении значения из `$propagatable` копируются во все записи, принадлежащие корневой.

#### Синхронизация

Иногда требуется наличие записей на всех сайтах, например для категорий или тегов. Установите `$propagatableSync = true`, чтобы после сохранения модель создавалась на других сайтах при отсутствии.

```php
protected $propagatableSync = true;
```

При использовании [групп сайтов](../../cms/resources/multisite.md) записи распространяются по всем сайтам в группе. Поведение можно настроить, указав `$propagatableSync` как массив параметров:

Опция | Описание
------------- | -------------
**sync** — логика синхронизации: `all`, `group`, `locale`. По умолчанию `group`.
**delete** — удалять ли связанные записи при удалении любой записи. По умолчанию `true`.
**except** — список атрибутов, которые не следует копировать при синхронизации.

```php
protected $propagatableSync = [
    'sync' => 'all',
    'delete' => false,
    'except' => [
        'description'
    ]
];
```

#### Сохранение моделей

Модели с мультисайтом по умолчанию не распространяют изменения. Используйте метод `savePropagate`, чтобы применить правила распространения.

```php
$model->savePropagate();
```

### Revisionable

Модели October CMS могут хранить историю изменений значений. Чтобы записывать ревизии, подключите трейты `October\Rain\Database\Traits\Revisionable`, перечислите атрибуты в `$revisionable` и определите связь `$morphMany` `revision_history`, ссылающуюся на `System\Models\Revision` с именем `revisionable`.

```php
class User extends Model
{
    use \October\Rain\Database\Traits\Revisionable;

    protected $revisionable = ['name', 'email'];

    public $morphMany = [
        'revision_history' => [\System\Models\Revision::class, 'name' => 'revisionable']
    ];
}
```

По умолчанию сохраняется до 500 записей. Изменить лимит можно свойством `$revisionableLimit`.

```php
/**
 * @var int Максимальное количество хранимых ревизий.
 */
public $revisionableLimit = 8;
```

Историю ревизий можно получить как обычную связь:

```php
$history = User::find(1)->revision_history;

foreach ($history as $record) {
    echo $record->field . ' updated ';
    echo 'from ' . $record->old_value;
    echo 'to ' . $record->new_value;
}
```

Запись ревизии может ссылаться на пользователя через атрибут `user_id`. Чтобы фиксировать автора изменений, добавьте метод `getRevisionableUser`.

```php
public function getRevisionableUser()
{
    return BackendAuth::getUser()->id;
}
```
