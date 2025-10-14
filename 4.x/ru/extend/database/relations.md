# Связи

Таблицы базы данных часто связаны друг с другом. Например, у записи блога может быть множество комментариев, а заказ связан с пользователем, который его оформил. October упрощает работу с такими связями и поддерживает несколько их типов.

## Определение связей

Связи моделей задаются в виде свойств классов модели. Пример определения связей:

```php
class User extends Model
{
    public $hasMany = [
        'posts' => \Acme\Blog\Models\Post::class
    ]
}
```

Связи, как и сами модели, являются мощными [конструкторами запросов](./query.md): обращение к ним как к методам позволяет применять метод-цепочки и условия выборки. Например:

```php
$user->posts()->where('is_active', true)->get();
```

Можно обращаться к связи и как к свойству.

```php
$user->posts;
```

## Расширенное определение

Каждое определение может быть массивом, где ключ — имя связи, а значение — массив параметров. Первое значение в массиве всегда задаёт класс связанной модели, остальные элементы указываются в формате «ключ → значение».

```php
public $hasMany = [
    'posts' => [\Acme\Blog\Models\Post::class, 'delete' => true]
];
```

Для любых связей доступны следующие параметры:

Аргумент | Описание
------------- | -------------
**order** | сортировка для набора записей.
**conditions** | фильтрация связи с помощью «сырого» выражения `where`.
**scope** | фильтрация связи методом [области запроса (scope)](../database/model.md) у модели.
**push** | если установить `false`, эта связь не будет сохранена через метод `push`. По умолчанию: `true`.
**delete** | если установить `true`, связанная модель удаляется при удалении основной модели или самой связи. По умолчанию: `false`.
**softDelete** | если установить `true`, связанная модель будет мягко удалена при [мягком удалении основной модели](./traits.md). По умолчанию: `false`.
**replicate** | если установить `true`, связанная модель будет продублирована или привязана при вызове метода `replicate`. По умолчанию: `false`.
**relationClass** | пользовательский класс для объекта связи.

Пример фильтрации с параметрами `order` и `conditions`.

```php
public $belongsToMany = [
    'categories' => [
        \Acme\Blog\Models\Category::class,
        'order' => 'name desc',
        'conditions' => 'is_active = 1'
    ]
];
```

Пример фильтрации с параметром `scope`.

```php
class Post extends Model
{
    public $belongsToMany = [
        'categories' => [
            \Acme\Blog\Models\Category::class,
            'scope' => 'isActive'
        ]
    ];
}

class Category extends Model
{
    public function scopeIsActive($query)
    {
        return $query->where('is_active', true)->orderBy('name', 'desc');
    }
}
```

Параметр `scope` может ссылаться и на статический метод.

```php
public $belongsToMany = [
    'categories' => [
        \Acme\Blog\Models\Category::class,
        'scope' => [self::class, 'myFilterMethod']
    ]
];

public static function myFilterMethod($query, $related, $parent)
{
    // ...
}
```

Пример с использованием параметра `relationClass`.

```php
public $belongsToMany = [
    'users' => [
        \Backend\Models\User::class,
        'relationClass' => \Backend\Classes\MyBelongsToMany::class
    ]
];
```

::: tip
Класс `relationClass` должен наследовать класс соответствующего типа связи. Например, для `belongsTo` класс обязан наследовать `October\Rain\Database\Relations\BelongsTo`.
:::

## Типы связей

Доступны следующие типы связей:

- [Один к одному](#relation-one-to-one)
- [Один ко многим](#relation-one-to-many)
- [Многие ко многим](#relation-many-to-many)
- [«Имеет много через»](#relation-has-many-through)
- [«Имеет одну через»](#relation-has-one-through)
- [Полиморфные связи](#relation-polymorphic-relations)

<a id="relation-one-to-one"></a>
### Один к одному

Связь «один к одному» — самая простая. Например, у модели `User` может быть один `Phone`. Чтобы определить такую связь, добавьте в свойство `$hasOne` модели `User` элемент `phone`.

```php
namespace Acme\Blog\Models;

use Model;

class User extends Model
{
    public $hasOne = [
        'phone' => \Acme\Blog\Models\Phone::class
    ];
}
```

После определения связи можно получить связанную запись, обращаясь к одноимённому свойству. Эти свойства создаются динамически и доступны как обычные атрибуты модели.

```php
$phone = User::find(1)->phone;
```

По умолчанию модель определяет внешний ключ исходя из имени модели. В данном случае предполагается, что у модели `Phone` есть внешний ключ `user_id`. Чтобы переопределить соглашение, передайте параметр `key` в определении.

```php
public $hasOne = [
    'phone' => [\Acme\Blog\Models\Phone::class, 'key' => 'my_user_id']
];
```

Также предполагается, что внешний ключ ссылается на столбец `id` родителя. Иначе говоря, связь ищет значение `id` пользователя в столбце `user_id` записи `Phone`. Чтобы использовать другой столбец, передайте параметр `otherKey`.

```php
public $hasOne = [
    'phone' => [\Acme\Blog\Models\Phone::class, 'key' => 'my_user_id', 'otherKey' => 'my_id']
];
```

#### Обратная связь

Теперь, когда доступ к `Phone` из `User` настроен, определим обратную связь на модели `Phone`, чтобы получить пользователя, которому принадлежит телефон. Для обратной связи `hasOne` используйте свойство `$belongsTo`:

```php
class Phone extends Model
{
    public $belongsTo = [
        'user' => \Acme\Blog\Models\User::class
    ];
}
```

В примере выше модель пытается сопоставить `user_id` из `Phone` с `id` из `User`. Название внешнего ключа определяется по имени связи с добавлением суффикса `_id`. Если внешний ключ на модели `Phone` отличается от `user_id`, укажите своё имя параметром `key`:

```php
public $belongsTo = [
    'user' => [Acme\Blog\Models\User::class, 'key' => 'my_user_id']
];
```

Если родительская модель использует в качестве первичного ключа не `id` или требуется связать дочернюю модель с другим столбцом, задайте параметр `otherKey`:

```php
public $belongsTo = [
    'user' => [\Acme\Blog\Models\User::class, 'key' => 'my_user_id', 'otherKey' => 'my_id']
];
```

#### Модель по умолчанию

Связь `belongsTo` позволяет определить модель по умолчанию, возвращаемую при `null`. Этот подход известен как [паттерн Null Object](https://en.wikipedia.org/wiki/Null_Object_pattern) и помогает избежать лишних проверок. В примере ниже связь `user` вернёт пустую модель `Acme\Blog\Models\User`, если пост не привязан к пользователю.

```php
public $belongsTo = [
    'user' => [\Acme\Blog\Models\User::class, 'default' => true]
];
```

Чтобы заполнить модель по умолчанию атрибутами, передайте массив в параметр `default`.

```php
public $belongsTo = [
    'user' => [
        \Acme\Blog\Models\User::class,
        'default' => ['name' => 'Guest']
    ]
];
```

<a id="relation-one-to-many"></a>
### Один ко многим

Связь «один ко многим» используется, когда одна модель владеет несколькими экземплярами другой модели. Например, у записи блога может быть бесконечное число комментариев. Как и другие связи, она задаётся добавлением элемента в свойство `$hasMany`:

```php
class Post extends Model
{
    public $hasMany = [
        'comments' => \Acme\Blog\Models\Comment::class
    ];
}
```

Модель автоматически определяет внешний ключ на модели `Comment`. По соглашению берётся имя модели во «змеином регистре» и добавляется суффикс `_id`. В данном случае ожидается, что внешний ключ — `post_id`.

После определения связи доступ к коллекции комментариев осуществляется через свойство `comments`. Поскольку модели предоставляют «динамические свойства», их можно читать как обычные атрибуты.

```php
$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    //
}
```

Поскольку любая связь также является конструктором запросов, можно накладывать дополнительные ограничения, обращаясь к методу `comments` и продолжая цепочку:

```php
$comments = Post::find(1)->comments()->where('title', 'foo')->first();
```

Как и в `hasOne`, можно переопределить внешний и локальный ключи, указав параметры `key` и `otherKey` соответственно:

```php
public $hasMany = [
    'comments' => [\Acme\Blog\Models\Comment::class, 'key' => 'my_post_id', 'otherKey' => 'my_id']
];
```

#### Обратная связь

Теперь, когда мы получаем все комментарии поста, определим связь, чтобы комментарий мог обратиться к родительскому посту. Обратная связь для `hasMany` задаётся свойством `$belongsTo` на дочерней модели:

```php
class Comment extends Model
{
    public $belongsTo = [
        'post' => \Acme\Blog\Models\Post::class
    ];
}
```

После определения связи получить модель `Post` для `Comment` можно через динамическое свойство `post`:

```php
$comment = Comment::find(1);

echo $comment->post->title;
```

В этом примере модель пытается сопоставить `post_id` из `Comment` с `id` в `Post`. Название внешнего ключа формируется из имени связи с суффиксом `_id`. Если внешний ключ отличается, передайте параметр `key`:

```php
public $belongsTo = [
    'post' => [\Acme\Blog\Models\Post::class, 'key' => 'my_post_id']
];
```

Если родительская модель использует иной первичный ключ или нужно связать дочернюю модель с другим столбцом, задайте параметр `otherKey`:

```php
public $belongsTo = [
    'post' => [Acme\Blog\Models\Post::class, 'key' => 'my_post_id', 'otherKey' => 'my_id']
];
```

<a id="relation-many-to-many"></a>
### Многие ко многим

Связи «многие ко многим» немного сложнее, чем `hasOne` и `hasMany`. Пример — пользователь с множеством ролей, которые разделяются другими пользователями. Например, у многих пользователей может быть роль «Admin». Чтобы определить такую связь, нужны три таблицы: `users`, `roles` и `role_user`. Таблица `role_user` формируется из названий моделей в алфавитном порядке и содержит столбцы `user_id` и `role_id`.

Ниже приведён пример [структуры таблиц](./structure.md) для создания таблицы соединения.

```php
Schema::create('role_user', function($table) {
    $table->integer('user_id')->unsigned();
    $table->integer('role_id')->unsigned();
    $table->primary(['user_id', 'role_id']);
});
```

Связи «многие ко многим» задаются добавлением элемента в свойство `$belongsToMany` модели. Например, определим метод `roles` в модели `User`:

```php
class User extends Model
{
    public $belongsToMany = [
        'roles' => \Acme\Blog\Models\Role::class
    ];
}
```

После определения связи можно получить роли пользователя через динамическое свойство `roles`:

```php
$user = User::find(1);

foreach ($user->roles as $role) {
    //
}
```

Как и для других типов связей, можно вызвать метод `roles`, чтобы продолжить формировать запрос:

```php
$roles = User::find(1)->roles()->orderBy('name')->get();
```

Как упоминалось выше, имя таблицы соединения формируется объединением имён моделей в алфавитном порядке. При необходимости соглашение можно переопределить, передав параметр `table` в определении `belongsToMany`:

```php
public $belongsToMany = [
    'roles' => [\Acme\Blog\Models\Role::class, 'table' => 'acme_blog_role_user']
];
```

Помимо имени таблицы, можно переопределить имена столбцов ключей, передав дополнительные параметры. Параметр `key` задаёт имя внешнего ключа модели, где определена связь; `otherKey` — имя внешнего ключа связанной модели:

```php
public $belongsToMany = [
    'roles' => [
        \Acme\Blog\Models\Role::class,
        'table' => 'acme_blog_role_user',
        'key' => 'my_user_id',
        'otherKey' => 'my_role_id'
    ]
];
```

#### Обратная связь

Чтобы определить обратную связь «многие ко многим», добавьте свойство `$belongsToMany` в связанной модели. Продолжая пример с ролями, определим связь `users` в модели `Role`:

```php
class Role extends Model
{
    public $belongsToMany = [
        'users' => \Acme\Blog\Models\User::class
    ];
}
```

Как видно, связь задаётся аналогично модели `User`, только ссылается на `Acme\Blog\Models\User`. Поскольку используется то же свойство `$belongsToMany`, доступен весь набор параметров кастомизации таблицы и ключей.

#### Дополнительные столбцы таблицы-посредника

Работа со связями «многие ко многим» требует промежуточной таблицы. Модели предоставляют удобный способ взаимодействия с ней. Предположим, объект `User` связан со многими объектами `Role`. После обращения к связи можно получить данные промежуточной таблицы через атрибут `pivot` моделей:

```php
$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

Каждая модель `Role`, полученная из связи, автоматически содержит атрибут `pivot`. Это отдельная модель, представляющая промежуточную таблицу, и с ней можно работать как с обычной моделью.

По умолчанию объект `pivot` содержит только ключи моделей. Если в таблице есть дополнительные атрибуты, перечислите их при определении связи:

```php
public $belongsToMany = [
    'roles' => [
        \Acme\Blog\Models\Role::class,
        'pivot' => ['column1', 'column2']
    ]
];
```

Чтобы автоматически управлять столбцами `created_at` и `updated_at` в таблице соединения, используйте параметр `timestamps`:

```php
public $belongsToMany = [
    'roles' => [
        \Acme\Blog\Models\Role::class,
        'timestamps' => true
    ]
];
```

Если требуется собственная модель для представления промежуточной таблицы, укажите её параметром `pivotModel`. Кастомные модели для обычных связей должны наследовать `October\Rain\Database\Pivot`, а для полиморфных — `October\Rain\Database\MorphPivot`.

```php
public $belongsToMany = [
    'roles' => [
        \Acme\Blog\Models\Role::class,
        'pivotModel' => \Acme\Blog\Models\UserRolePivot::class
    ]
];
```

#### Разрешение дубликатов связей

Иногда требуется несколько раз связать одни и те же модели, сохраняя разные данные в таблице-посреднике. В примере ниже показана [структура таблиц](./structure.md) с автоинкрементным первичным ключом вместо составного.

```php
Schema::create('role_user', function($table) {
    $table->increments('id');
    $table->integer('user_id')->unsigned();
    $table->integer('role_id')->unsigned();
});
```

Для такой конфигурации нужна собственная модель `pivotModel`, а параметр `pivotKey` указывает имя автоинкрементного столбца (`id`).

```php
public $belongsToMany = [
    'roles' => [
        \Acme\Blog\Models\Role::class,
        'pivotModel' => \Acme\Blog\Models\UserRolePivot::class,
        'pivotKey' => 'id'
    ]
];
```

В определении модели-посредника следует установить свойство `$incrementing = true`, чтобы включить автоинкрементный первичный ключ, который по умолчанию называется `id`.

```php
class UserRolePivot extends \October\Rain\Database\Pivot
{
    public $incrementing = true;
}
```

Ниже приведены параметры, поддерживаемые для связей `belongsToMany`:

Свойство | Описание
------------- | -------------
**table** | имя таблицы соединения.
**key** | имя столбца ключа модели, где определена связь (в таблице-посреднике). По умолчанию — имя модели + `_id`, например `user_id`.
**parentKey** | имя столбца ключа модели, где определена связь (в таблице модели). По умолчанию: `id`.
**otherKey** | имя столбца ключа связанной модели (в таблице-посреднике). По умолчанию — имя модели + `_id`, например `role_id`.
**relatedKey** | имя столбца ключа связанной модели (в таблице связанной модели). По умолчанию: `id`.
**timestamps** | если `true`, таблица соединения должна содержать `created_at` и `updated_at`. По умолчанию: `false`.
**detach** | если `false`, связанная модель не будет отвязываться при удалении основной модели или разрушении связи. По умолчанию: `true`.
**pivot** | массив столбцов таблицы-посредника; атрибуты доступны через `$model->pivot`.
**pivotModel** | пользовательский класс модели для доступа к таблице-посреднику. По умолчанию `October\Rain\Database\Pivot`, для полиморфных связей — `October\Rain\Database\MorphPivot`.
**pivotSortable** | столбец сортировки таблицы-посредника, используется вместе с [трейтом модели SortableRelation](../lists/structures.md).
**pivotKey** | имя автоинкрементного первичного ключа таблицы-посредника; требует пользовательской модели `pivotModel` с определённым первичным ключом.

<a id="relation-has-many-through"></a>
### «Имеет много через»

Связь `hasManyThrough` предоставляет удобный способ доступа к удалённым связям через промежуточную. Например, модель `Country` может иметь много моделей `Post` через промежуточную модель `User`. Так можно получить все записи блога для конкретной страны. Структура таблиц:

```
countries
    id - integer
    name - string

users
    id - integer
    country_id - integer
    name - string

posts
    id - integer
    user_id - integer
    title - string
```

Хотя в `posts` нет столбца `country_id`, связь `hasManyThrough` позволяет получить посты страны через `$country->posts`. При выполнении запроса модель анализирует `country_id` в промежуточной таблице `users`, находит подходящие идентификаторы пользователей и использует их для выборки из `posts`.

Определим связь в модели `Country`:

```php
class Country extends Model
{
    public $hasManyThrough = [
        'posts' => [
            \Acme\Blog\Models\Post::class,
            'through' => \Acme\Blog\Models\User::class
        ],
    ];
}
```

Первый аргумент в `$hasManyThrough` — имя конечной модели, а параметр `through` — имя промежуточной модели.

При выполнении запроса используются стандартные соглашения об именах ключей. Чтобы переопределить ключи связи, передайте параметры `key`, `otherKey` и `throughKey`. Параметр `key` задаёт имя внешнего ключа на промежуточной модели, `throughKey` — имя внешнего ключа в конечной модели, `otherKey` — локальный ключ.

```php
public $hasManyThrough = [
    'posts' => [
        \Acme\Blog\Models\Post::class,
        'key' => 'my_country_id',
        'through' => \Acme\Blog\Models\User::class,
        'throughKey' => 'my_user_id',
        'otherKey' => 'my_id',
        'secondOtherKey' => 'my_country_id'
    ],
];
```

<a id="relation-has-one-through"></a>
### «Имеет одну через»

Связь `hasOneThrough` связывает модели через одну промежуточную связь. Например, если у каждого поставщика есть один пользователь, а у пользователя — одна запись истории, модель поставщика может получить историю пользователя через него. Необходимые таблицы:

```
users
    id - integer
    supplier_id - integer

suppliers
    id - integer

history
    id - integer
    user_id - integer
```

Хотя в таблице `history` нет `supplier_id`, связь `hasOneThrough` предоставляет доступ к истории пользователя из модели поставщика. Определим её в модели `Supplier`:

```php
class Supplier extends Model
{
    public $hasOneThrough = [
        'userHistory' => [
            \Acme\Supplies\Model\History::class,
            'through' => \Acme\Supplies\Model\User::class
        ],
    ];
}
```

Первый элемент в массиве свойства `$hasOneThrough` — конечная модель, параметр `through` — промежуточная модель.

Используются стандартные соглашения об именах ключей. Чтобы переопределить ключи, передайте параметры `key`, `otherKey` и `throughKey` (аналогично `hasManyThrough`).

```php
public $hasOneThrough = [
    'userHistory' => [
        \Acme\Supplies\Model\History::class,
        'key' => 'supplier_id',
        'through' => \Acme\Supplies\Model\User::class,
        'throughKey' => 'user_id',
        'otherKey' => 'id',
        'secondOtherKey' => 'my_country_id'
    ],
];
```

<a id="relation-polymorphic-relations"></a>
## Полиморфные связи

Полиморфные связи позволяют модели принадлежать сразу нескольким другим моделям через одну ассоциацию.

### Полиморфная связь «один к одному»

#### Структура таблиц

Полиморфная связь «один к одному» похожа на обычную, но целевая модель может принадлежать нескольким типам. Например, нужно хранить фотографии сотрудников и товаров. Полиморфная связь позволит использовать одну таблицу `photos` для обоих случаев. Структура таблиц:

```
staff
    id - integer
    name - string

products
    id - integer
    price - integer

photos
    id - integer
    path - string
    imageable_id - integer
    imageable_type - string
```

Ключевые столбцы — `imageable_id` и `imageable_type`. В `imageable_id` хранится идентификатор владельца (сотрудника или товара), в `imageable_type` — имя класса владельца. По столбцу `imageable_type` ORM определяет, модель какого типа нужно вернуть при обращении к связи `imageable`.

#### Структура моделей

Определения моделей для этой связи:

```php
class Photo extends Model
{
    public $morphTo = [
        'imageable' => []
    ];
}

class Staff extends Model
{
    public $morphOne = [
        'photo' => [\Acme\Blog\Models\Photo::class, 'name' => 'imageable']
    ];
}

class Product extends Model
{
    public $morphOne = [
        'photo' => [\Acme\Blog\Models\Photo::class, 'name' => 'imageable']
    ];
}
```

#### Получение полиморфных связей

После определения таблиц и моделей можно обращаться к связям. Например, чтобы получить фотографию сотрудника, достаточно обратиться к динамическому свойству `photo`:

```php
$staff = Staff::find(1);

$photo = $staff->photo
```

Можно получить владельца полиморфной связи, обратившись к имени `morphTo`. В нашем случае это определение `imageable` в модели `Photo`, доступное как динамическое свойство:

```php
$photo = Photo::find(1);

$imageable = $photo->imageable;
```

Связь `imageable` вернёт экземпляр `Staff` или `Product` в зависимости от владельца фотографии.

### Полиморфная связь «один ко многим»

#### Структура таблиц

Полиморфная связь «один ко многим» аналогична обычной, но целевая модель может принадлежать нескольким типам. Например, пользователи могут оставлять комментарии и к постам, и к видео. Полиморфная связь позволяет использовать одну таблицу `comments` для обоих сценариев. Структура таблиц:

```
posts
    id - integer
    title - string
    body - text

videos
    id - integer
    title - string
    url - string

comments
    id - integer
    body - text
    commentable_id - integer
    commentable_type - string
```

#### Структура моделей

Определения моделей:

```php
class Comment extends Model
{
    public $morphTo = [
        'commentable' => []
    ];
}

class Post extends Model
{
    public $morphMany = [
        'comments' => [\Acme\Blog\Models\Comment::class, 'name' => 'commentable']
    ];
}

class Product extends Model
{
    public $morphMany = [
        'comments' => [\Acme\Blog\Models\Comment::class, 'name' => 'commentable']
    ];
}
```

#### Получение связи

После определения таблиц и моделей можно получать связи. Например, чтобы получить все комментарии поста, воспользуйтесь динамическим свойством `comments`:

```php
$post = Author\Plugin\Models\Post::find(1);

foreach ($post->comments as $comment) {
    //
}
```

Получить владельца полиморфной связи можно через имя, указанное в `$morphTo`. В нашем случае это определение `commentable` в модели `Comment`:

```php
$comment = Author\Plugin\Models\Comment::find(1);

$commentable = $comment->commentable;
```

Связь `commentable` вернёт экземпляр `Post` или `Video` в зависимости от владельца комментария.

Можно изменить владельца связанной модели, присвоив значение атрибуту с именем `morphTo`, в данном случае `commentable`.

```php
$comment = Author\Plugin\Models\Comment::find(1);
$video = Author\Plugin\Models\Video::find(1);

$comment->commentable = $video;
$comment->save()
```

### Полиморфная связь «многие ко многим»

#### Структура таблиц

Помимо связей «один к одному» и «один ко многим», можно определить полиморфные связи «многие ко многим». Например, модели `Post` и `Video` могут делить полиморфную связь с моделью `Tag`. Такая связь позволяет хранить уникальные теги, общие для постов и видео. Структура таблиц:

```
posts
    id - integer
    name - string

videos
    id - integer
    name - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_id - integer
    taggable_type - string
```

#### Структура моделей

Определим связи в моделях. В `Post` и `Video` связь `tags` задаётся в свойстве `$morphToMany` базовой модели:

```php
class Post extends Model
{
    public $morphToMany = [
        'tags' => [\Acme\Blog\Models\Tag::class, 'name' => 'taggable']
    ];
}
```

#### Обратная связь

В модели `Tag` необходимо определить связь для каждой из связанных моделей. В этом примере — `posts` и `videos`:

```php
class Tag extends Model
{
    public $morphedByMany = [
        'posts'  => [\Acme\Blog\Models\Post::class, 'name' => 'taggable'],
        'videos' => [\Acme\Blog\Models\Video::class, 'name' => 'taggable']
    ];
}
```

#### Получение связи

После определения таблиц и моделей можно работать со связями. Например, чтобы получить все теги поста, используйте динамическое свойство `tags`:

```php
$post = Post::find(1);

foreach ($post->tags as $tag) {
    //
}
```

Получить владельца полиморфной связи можно через определения в `$morphedByMany`, то есть через методы `posts` или `videos` модели `Tag`, доступные как динамические свойства:

```php
$tag = Tag::find(1);

foreach ($tag->videos as $video) {
    //
}
```

#### Пользовательские типы полиморфизма

По умолчанию в базе хранится полное имя класса, представляющего тип связанной модели. Например, если `Photo` принадлежит `Staff` или `Product`, значение `imageable_type` будет `Acme\Blog\Models\Staff` либо `Acme\Blog\Models\Product`.

Пользовательские типы позволяют отвязать базу данных от структуры приложения. Можно определить «карту» полиморфизма и задать собственные имена вместо имён классов:

```php
use October\Rain\Database\Relations\Relation;

Relation::morphMap([
    'staff' => \Acme\Blog\Models\Staff::class,
    'product' => \Acme\Blog\Models\Product::class,
]);
```

Чаще всего `morphMap` регистрируют в методе `boot` [регистрационного файла плагина](../extending.md).

<a id="oc-querying-relations"></a>
## Запросы к связям

Любой тип связи можно вызвать как метод, получив экземпляр связи без фактического выполнения запроса. Кроме того, все связи являются [конструкторами запросов](./query.md), поэтому к ним можно цепочкой добавлять условия перед выполнением SQL.

Предположим, в блоге модель `User` связана со многими моделями `Post`:

```php
class User extends Model
{
    public $hasMany = [
        'posts' => \Acme\Blog\Models\Post::class
    ];
}
```

### Доступ через метод связи

Связь **posts** можно вызывать как метод и накладывать дополнительные ограничения, продолжая цепочку методов [конструктора запросов](./query.md).

```php
$user = User::find(1);

$posts = $user->posts()->where('is_active', 1)->get();

$post = $user->posts()->first();
```

### Доступ через динамическое свойство

Если дополнительные ограничения не нужны, можно обращаться к связи как к свойству. Продолжая пример, доступ ко всем постам пользователя осуществляется через `$user->posts`.

```php
$user = User::find(1);

foreach ($user->posts as $post) {
    // ...
}
```

Динамические свойства загружают данные «лениво», то есть только при первом обращении. Поэтому часто используется предварительная загрузка (eager loading), чтобы заранее получить связи и сократить число SQL-запросов.

### Проверка существования связей

Иногда нужно ограничить результаты наличием связанной записи. Например, получить посты с хотя бы одним комментарием. Для этого передайте имя связи в метод `has`:

```php
// Посты, у которых есть хотя бы один комментарий...
$posts = Post::has('comments')->get();
```

Можно указать оператор и количество:

```php
// Посты с тремя и более комментариями...
$posts = Post::has('comments', '>=', 3)->get();
```

Вложенные выражения `has` можно строить через «точечную» нотацию. Например, получить посты с хотя бы одним комментарием и голосом:

```php
// Посты с хотя бы одним комментарием, у которого есть голос...
$posts = Post::has('comments.votes')->get();
```

Для сложных условий используйте методы `whereHas` и `orWhereHas`, добавляющие `where`-условия к проверке наличия связи, например для поиска по содержимому комментария:

```php
// Посты с хотя бы одним комментарием, начинающимся на foo
$posts = Post::whereHas('comments', function ($query) {
    $query->where('content', 'like', 'foo%');
})->get();
```

#### Компактные запросы на существование

Чтобы проверить существование связи с одним условием, удобнее использовать методы `whereRelation`, `orWhereRelation`, `whereMorphRelation` или `orWhereMorphRelation`.

```php
$posts = Post::whereRelation('comments', 'is_approved', false)->get();
```

Методы `searchWhereRelation` и `orSearchWhereRelation` позволяют выполнять поиск по столбцам связей. Подобно [поисковым запросам](./query.md), первый аргумент — поисковая фраза, второй — имя связи, третий — массив столбцов; используется регистронезависимое условие LIKE.

```php
$posts = Post::searchWhereRelation('foo bar', 'author', ['name', 'bio'])->get();
```

#### Проверка отсутствия связей

Методы `doesntHave` и `orDoesntHave` ограничивают результаты отсутствием связи.

```php
$posts = Post::doesntHave('comments')->get();
```

Методы `whereDoesntHave` и `orWhereDoesntHave` добавляют дополнительные условия к проверке отсутствия связи.

```php
$posts = Post::whereDoesntHave('comments', function ($query) {
    $query->where('content', 'like', 'code%');
})->get();
```

### Подсчёт связанных записей

В некоторых случаях нужно посчитать количество связанных записей. Метод `withCount` добавляет к выбранным моделям столбец `{relation}_count`.

```php
$users = User::withCount('roles')->get();

foreach ($users as $user) {
    echo $user->roles_count;
}
```

`withCount` поддерживает несколько связей и дополнительные условия.

```php
$posts = Post::withCount(['votes', 'comments' => function ($query) {
    $query->where('content', 'like', 'foo%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

Столбец можно подгрузить позднее методом `loadCount`:

```php
$user = User::first();
$user->loadCount('roles');
```

Дополнительные условия также поддерживаются.

```php
$user->loadCount(['roles' => function ($query) {
    $query->where('clearance', '>', 5);
}])
```

## Предварительная загрузка

При обращении к связям как к свойствам данные загружаются «лениво». Это значит, что связь не загружается, пока к ней не обратятся. Однако модели можно «жадно» загружать связи заранее при выборке родительской модели. Предварительная загрузка решает проблему N + 1. Рассмотрим модель `Book`, связанную с `Author`.

```php
class Book extends Model
{
    public $belongsTo = [
        'author' => \Acme\Blog\Models\Author::class
    ];
}
```

Получим все книги и их авторов:

```php
$books = Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
```

Этот цикл выполнит один запрос для получения книг, а затем по запросу на каждого автора. Если книг 25, получится 26 запросов: один для книг и 25 для авторов.

Предварительная загрузка сокращает операцию до двух запросов. При выборке можно указать связи, которые нужно загрузить, методом `with`:

```php
$books = Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

Выполнятся лишь два запроса:

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

#### Предзагрузка нескольких связей

Иногда требуется загрузить сразу несколько связей. Для этого передайте дополнительные аргументы в метод `with`:

```php
$books = Book::with('author', 'publisher')->get();
```

#### Вложенная предзагрузка

Чтобы загрузить вложенные связи, используйте «точечную» нотацию. Например, загрузим авторов книги и их личные контакты:

```php
$books = Book::with('author.contacts')->get();
```

### Ограничение предзагрузки

Иногда нужно загрузить связь заранее, но при этом наложить дополнительные условия. Пример:

```php
$users = User::with([
    'posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }
])->get();
```

В этом случае будут загружены только посты, в названии которых есть слово `first`. Разумеется, доступны и другие методы [конструктора запросов](./query.md):

```php
$users = User::with([
    'posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }
])->get();
```

### Лениво-добранная предзагрузка

Иногда требуется загрузить связь уже после получения родительской модели, например если решение принимается динамически:

```php
$books = Book::all();

if ($someCondition) {
    $books->load('author', 'publisher');
}
```

Чтобы добавить дополнительные условия к запросу предзагрузки, передайте `Closure` методу `load`:

```php
$books->load([
    'author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }
]);
```

## Добавление связанных моделей

Как и при выполнении запросов, October CMS позволяет определять связи через метод или динамическое свойство. Допустим, нужно добавить новый `Comment` к модели `Post`. Вместо ручной установки `post_id` у комментария можно добавить его через связь.

### Добавление через метод связи

October предоставляет удобные методы для добавления и удаления моделей из связей. Модель можно ассоциировать со связью или отвязать от неё. В каждом случае связь создаётся или разрывается соответственно.

#### Метод add

Метод `add` связывает новую модель.

```php
$comment = new Comment(['message' => 'A new comment.']);

$post = Post::find(1);

$comment = $post->comments()->add($comment);
```

Обратите внимание, что мы обратились к связи `comments` как к методу, чтобы получить экземпляр связи. Метод `add` автоматически присвоит новому `Comment` значение `post_id`.

Чтобы сохранить несколько моделей за раз, используйте `addMany`:

```php
$post = Post::find(1);

$post->comments()->addMany([
    new Comment(['message' => 'A new comment.']),
    new Comment(['message' => 'Another comment.']),
]);
```

#### Метод remove

Метод `remove` отвязывает модель, превращая её в «осиротевшую» запись.

```php
$post->comments()->remove($comment);
```

Для связей «многие ко многим» запись удаляется из коллекции связи.

```php
$post->categories()->remove($category);
```

Для связи `belongsTo` можно использовать метод `dissociate`, которому не требуется передавать связанную модель.

```php
$post->author()->dissociate();
```

#### Добавление с данными pivot

Для связей «многие ко многим» метод `add` принимает второй аргумент — массив дополнительных данных таблицы-посредника.

```php
$user = User::find(1);

$pivotData = ['expires' => $expires];

$user->roles()->add($role, $pivotData);
```

Второй аргумент также может содержать ключ сессии, используемый отложенным связыванием, если передан строкой. В таком случае данные pivot передаются третьим аргументом.

```php
$user->roles()->add($role, $sessionKey, $pivotData);
```

#### Метод create

Методы `add` и `addMany` принимают готовый экземпляр модели, но можно воспользоваться методом `create`, который принимает массив атрибутов, создаёт модель и сохраняет её в базе.

```php
$post = Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```

Перед использованием `create` изучите документацию по [массовому присвоению](./model.md): атрибуты массива ограничиваются списком `fillable` модели.

### Добавление через динамическое свойство

Связи можно задавать напрямую через свойства, как и при чтении. Присвоение значения свойству перезаписывает существующую связь. После присвоения модель нужно сохранить, как и другие атрибуты.

```php
$post->author = $author;

$post->comments = [$comment1, $comment2];

$post->save();
```

Можно присвоить первичные ключи, что удобно при работе с HTML-формами.

```php
// Назначить автора с идентификатором 3
$post->author = 3;

// Назначить комментарии с идентификаторами 1, 2 и 3
$post->comments = [1, 2, 3];

$post->save();
```

Связи можно отвязать, присвоив значению `null`.

```php
$post->author = null;

$post->comments = null;

$post->save();
```

Как и при отложенном связывании, связи у несохранённых моделей удерживаются в памяти до их сохранения. В примере ниже пост ещё не существует, поэтому `post_id` нельзя присвоить комментарию через `$post->comments`. Ассоциация откладывается до вызова `save`.

```php
$comment = Comment::find(1);

$post = new Post;

$post->comments = [$comment];

$post->save();
```

### Связи «многие ко многим»

#### Привязка и отвязка

При работе с «многие ко многим» модели предоставляют дополнительные методы. Например, у пользователя может быть много ролей и наоборот. Чтобы добавить роль пользователю, вставив запись в таблицу-посредник, используйте метод `attach`:

```php
$user = User::find(1);

$user->roles()->attach($roleId);
```

При привязке можно передать массив дополнительных данных для таблицы-посредника:

```php
$user->roles()->attach($roleId, ['expires' => $expires]);
```

Чтобы удалить роль у пользователя, используйте метод `detach`. Он удалит запись из таблицы-посредника, но сами модели останутся.

```php
// Удалить одну роль пользователя...
$user->roles()->detach($roleId);

// Удалить все роли пользователя...
$user->roles()->detach();
```

Методы `attach` и `detach` принимают массивы идентификаторов:

```php
$user = User::find(1);

$user->roles()->detach([1, 2, 3]);

$user->roles()->attach([1 => ['expires' => $expires], 2, 3]);
```

#### Синхронизация

Метод `sync` создаёт связи «многие ко многим» из массива идентификаторов. Все идентификаторы, отсутствующие в массиве, будут удалены из таблицы-посредника. После выполнения операции в таблице останутся только указанные идентификаторы.

```php
$user->roles()->sync([1, 2, 3]);
```

Можно передать и дополнительные данные таблицы-посредника:

```php
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

### Обновление времени родительской модели

Если модель `belongsTo` или `belongsToMany` другой модели (например, `Comment` принадлежит `Post`), бывает полезно обновить метку времени родителя при обновлении дочерней модели. Чтобы при обновлении `Comment` автоматически обновлялась колонка `updated_at` модели `Post`, добавьте свойство `touches` со списком связей в дочерней модели:

```php
class Comment extends Model
{
    /**
     * Связи, метки времени которых нужно обновлять.
     */
    protected $touches = ['post'];

    /**
     * Связи
     */
    public $belongsTo = [
        'post' => \Acme\Blog\Models\Post::class
    ];
}
```

Теперь при обновлении `Comment` будет обновляться `updated_at` у связанного `Post`:

```php
$comment = Comment::find(1);

$comment->text = 'Edit to this comment!';

$comment->save();
```

## Отложенное связывание

Отложенное связывание позволяет переносить установление связей до сохранения основной записи. Это полезно, когда нужно подготовить несколько моделей (например, загрузки файлов) и связать их с моделью, которой ещё нет.

Можно откладывать любое количество **дочерних** моделей для **главной** модели, используя **ключ сессии**. Когда основная запись сохраняется с этим ключом, связи обновляются автоматически. Отложенное связывание поддерживается в [поведении формы бэкенда](../backend/form.md), но его можно использовать и в других случаях.

### Генерация ключа сессии

Ключ сессии требуется для отложенного связывания. Его можно рассматривать как идентификатор транзакции. Тот же ключ используется для связывания/отвязывания и сохранения основной модели. Сгенерировать ключ можно через PHP-функцию `uniqid()`. [Помощник формы](../../markup/function/form.md) автоматически создаёт скрытое поле с ключом.

```php
$sessionKey = uniqid('session_key', true);
```

### Отложенное связывание

Комментарий из следующего примера не будет добавлен к посту, пока пост не сохранён.

```php
$comment = new Comment;
$comment->content = "Hello world!";
$comment->save();

$post = new Post;
$post->comments()->add($comment, $sessionKey);
```

::: tip
Объект `$post` ещё не сохранён, но связь будет создана после сохранения.
:::

### Отложенное отвязывание

Комментарий из следующего примера не будет удалён, пока пост не сохранён.

```php
$comment = Comment::find(1);
$post = Post::find(1);
$post->comments()->remove($comment, $sessionKey);
```

### Список всех связей

Метод `withDeferred` позволяет загрузить все записи связи, включая отложенные. В результат войдут и существующие связи.

```php
$post->comments()->withDeferred($sessionKey)->get();
```

### Отмена всех связей

Рекомендуется отменять отложенные связи и удалять дочерние объекты, чтобы не оставлять «сирот».

```php
$post->cancelDeferred($sessionKey);
```

### Подтверждение связей

Можно подтвердить (привязать или отвязать) все отложенные связи при сохранении основной модели, передав ключ сессии вторым аргументом метода `save`.

```php
$post = new Post;
$post->title = "First blog post";
$post->save(['sessionKey' => $sessionKey]);
```

Аналогично работает метод `create`:

```php
$post = Post::create(['title' => 'First blog post'], $sessionKey);
```

### Ленивое подтверждение связей

Если при сохранении нельзя передать `$sessionKey`, можно подтвердить связи в любой момент следующим образом:

```php
$post->commitDeferred($sessionKey);
```

### Очистка «осиротевших» связей

Удаляет все неподтверждённые связи старше одного дня:

```php
October\Rain\Database\Models\DeferredBinding::cleanUp(1);
```

::: tip
October CMS автоматически удаляет отложенные связи старше пяти дней в ходе сборки мусора.
:::

### Отключение отложенного связывания

Иногда нужно полностью отключить отложенное связывание для модели, например при работе с другой базой данных. Для этого убедитесь, что свойство `sessionKey` модели равно `null` до выполнения хуков предварительного и последующего связывания во внутреннем методе сохранения. Можно подписаться на событие `model.saveInternal` модели.

```php
public function __construct()
{
    parent::__construct(...func_get_args());

    $this->bindEvent('model.saveInternal', function () {
        $this->sessionKey = null;
    });
}
```

::: tip
Эта настройка полностью отключает отложенное связывание для любой модели, где она применена.
:::
