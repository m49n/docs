---
subtitle: Доступные модели Tailor.
---
# Модели

В этой статье описано, как работать с Tailor на PHP и какие модели доступны.

## Классы моделей

Следующие классы моделей соответствуют своим типам чертежей.

Model Class | Blueprint Type
----------- | --------------
`Tailor\Models\EntryRecord` | entry
`Tailor\Models\StructureRecord` | structure
`Tailor\Models\StreamRecord` | stream
`Tailor\Models\SingleRecord` | single
`Tailor\Models\GlobalRecord` | global

## Записи entry

Модель `EntryRecord` — базовая модель для хранения контента записи. Помимо определённых полей формы, у полученной модели доступны следующие атрибуты.

Attribute | Description
-------- | -------------
**id** | Первичный ключ в базе данных.
**blueprint_uuid** | UUID связанного чертежа.
**content_group** | Имя группы контента, если используется.
**title** | Заголовок записи, например **My Blog Post**.
**slug** | Идентификатор slug записи, например `my-blog-post`.
**is_enabled** | Определяет, отображается ли запись.
**created_at** | Дата создания записи.
**updated_at** | Дата последнего обновления записи.
**expired_at** | Дата окончания действия записи.
**published_at** | Дата публикации записи.
**published_at_date** | Дата публикации или, если не задана, дата создания.

### Запись structure

Модель `StructureRecord` расширяет `EntryRecord` и хранит контент структурированных записей. Если тип записи — `structure`, используется эта модель и появляются дополнительные атрибуты.

Attribute | Description
-------- | -------------
**fullslug** | Идентификатор slug вместе со slug родителя, например `parent-slug/child-slug`.
**parent** | Родительская запись, если есть.
**children** | Дочерние записи, если есть.

### Запись stream

Модель `StreamRecord` расширяет `EntryRecord` и хранит контент потоковых записей. Если тип записи — `stream`, появляются дополнительные атрибуты.

Attribute | Description
-------- | -------------
**published_at_day** | Числовой день публикации записи.
**published_at_month** | Числовой месяц публикации записи.
**published_at_year** | Числовой год публикации записи.

### Получение нескольких записей

Чтобы работать с записью через PHP, используйте класс модели (например, `EntryRecord`) и вызовите статический метод `inSection`, передав handle. Он вернёт подготовленный [запрос модели](../../extend/database/query.md). Альтернативно можно использовать UUID и метод `inSectionUuid`.

Метод `get` вернёт [коллекцию записей](../../extend/services/collection.md).

```php
$records = EntryRecord::inSection('Blog\Post')->get();

$records = EntryRecord::inSectionUuid('a63fabaf-7c0b-4c74-b36f-7abf1a3ad1c1')->get();
```

### Получение одной записи

Сочетая метод `where`, можно найти одну запись методом `first`. Пример ниже найдёт запись со slug **first-post**.

```php
$record = EntryRecord::inSection('Blog\Post')->where('slug', 'first-post')->first();
```

Если [тип записи](../tailor/blueprints.md) равен `single`, используйте метод `findSingleForSection`, чтобы получить запись. Аналогично метод `findSingleForSectionUuid` ищет запись по UUID. Эти методы гарантируют наличие записи при поиске.

```php
$record = SingleRecord::findSingleForSection('Homepage');

$record = SingleRecord::findSingleForSectionUuid('3328c303-7989-462e-b866-27e7037ba275');
```

### Создание и обновление записей

Метод `inSection` можно использовать для создания записей динамически. Пример ниже создаёт новую запись блога. Тот же код подойдёт для обновления существующей записи после её получения.

```php
$post = EntryRecord::inSection('Blog\Post');
$post->title = 'Imported Post';
$post->save();
```

## Глобальная запись

Модель `GlobalRecord` хранит контент глобальных чертежей.

### Доступные атрибуты

Помимо определённых полей формы, у полученной модели доступны следующие атрибуты.

Attribute | Description
-------- | -------------
**id** | Первичный ключ в базе данных.
**blueprint_uuid** | UUID связанного чертежа.

### Получение глобальной записи

Чтобы найти глобальный чертёж через PHP, используйте модель `Tailor\Models\GlobalRecord` и статический метод `findForGlobal`, передав handle. Можно также воспользоваться UUID и методом `findForGlobalUuid`.

```php
GlobalRecord::findForGlobal('Blog\Config');

GlobalRecord::findForGlobalUuid('7b193500-ac0b-481f-a79c-2a362646364d');
```

## Работа со связанными полями

К связанным полям относятся [повторители (repeaters)](../../element/form/widget-repeater.md) и [поля entries](../../element/content/field-entries.md). Для чтения и записи в такие поля нужны дополнительные шаги.

### Жадная загрузка связей

Для чтения связанных полей можно заранее загрузить их в коллекцию методом `load`. Этот метод добавляет связанный контент одним запросом и обеспечивает лучшую производительность.

Ниже жадно загружается поле **categories** и добавляется к результату, а также показан пример загрузки нескольких связанных полей.

```php
$records->load('categories');

$records->load(['categories', 'author']);
```

### Создание связанных полей

Для записи в связанные поля вызовите метод с именем связи, чтобы получить определение отношения, затем вызовите `create()`, который вернёт созданную связь.

Ниже находится первая запись блога в секции **Blog\Post** и создаётся связанная категория.

```php
$post = EntryRecord::inSection('Blog\Post')->first();

$post->categories()->create(['title' => 'Test', 'price' => '100']);
```

Используйте метод `make()`, чтобы создать новый пустой экземпляр модели.

```php
$category = $post->categories()->make();
```

Если категория уже существует, используйте `add()`. В примере ниже первая категория блога добавляется к первой записи блога.

```php
$post = EntryRecord::inSection('Blog\Post')->first();
$category = EntryRecord::inSection('Blog\Category')->first();

$post->categories()->add($category);
```

::: tip
См. статью о [связях моделей](../../extend/database/relations.md), чтобы узнать больше о связях моделей.
:::

## Расширение конструкторов моделей

Аналогично [расширению обычных моделей](../../extend/extending.md), можно расширить конструктор `EntryRecord` методом `extendInSection`, чтобы нацелиться на конкретный чертёж. Метод `extendInSectionUuid` предоставляет более точное таргетирование.

```php
EntryRecord::extendInSection('Blog\Post', function($model) {
    $model->bindEvent('model.afterDelete', function () use ($model) {
        // Model has been deleted!
    });
});
```

Конструктор `GlobalRecord` также поддерживает расширение методами `extendInGlobal` и `extendInGlobalUuid` для таргетирования конкретного чертежа.

```php
GlobalRecord::extendInGlobal('Blog\Config', function($model) {
    $model->bindEvent('model.beforeSave', function () use ($model) {
        // Model has been saved!
    });
});
```

::: tip
Методы `extendInSectionUuid` и `extendInGlobalUuid` не генерируют исключение, если чертеж не найден.
:::

## Расширение моделей Tailor

Иногда требуется комбинировать модели Tailor с [обычными моделями базы данных](../../extend/system/models.md).

### Замена модели целиком

Используйте свойство `modelClass` чертежа, когда необходимо добавить сложную функциональность. Тогда чертёж будет разрешаться в пользовательский экземпляр модели.

```yaml
handle: Blog\Post
type: stream
name: Blog Post
modelClass: App\Models\TailorBlogPost
# ...
```

Важно, чтобы указанная модель расширяла класс, используемый типом чертежа. В примере выше нужно расширять класс `StreamRecord`, так как используется тип `stream`.

```php
namespace App\Models;

use Tailor\Models\StreamRecord;

class TailorBlogPost extends StreamRecord
{
    // Custom model logic goes here
}
```

Теперь при разрешении чертежа будет возвращаться экземпляр класса `TailorBlogPost`.

### Связь Tailor с обычными моделями

Поле формы `recordfinder` создаёт определение связи для обычной модели, например модели из плагина. Свойство **modelClass** должно ссылаться на класс модели, а свойство **list** требуется для одиночных связей, что задаётся через **maxItems**.

```yaml
products:
    label: Products
    type: recordfinder
    modelClass: Acme\Test\Models\Product
    list: $/acme/test/models/product/columns.yaml
    maxItems: 1
```

Подробнее о [виджете формы recordfinder](../../element/form/widget-recordfinder.md).

### Связь обычных моделей с Tailor

Поскольку все модели Tailor используют общий класс модели, в определениях связей нужны дополнительные атрибуты. Трейт `Tailor\Traits\BlueprintRelationModel` реализует такие атрибуты для ссылок на модели Tailor, поддерживая связи Belongs To и Belongs To Many.

При подключении трейта `BlueprintRelationModel` к обычной модели можно указать свойство `blueprint` со значением UUID чертежа Tailor. В примере ниже создаётся связь Belongs To с классом `Tailor\Models\EntryRecord` под именем **author**.

```php
class Product extends Model
{
    use \Tailor\Traits\BlueprintRelationModel;

    public $belongsTo = [
        'author' => [
            \Tailor\Models\EntryRecord::class,
            'blueprint' => '6947ff28-b660-47d7-9240-24ca6d58aeae'
        ]
    ];
}
```

### Создание пользовательского поля контента

Можно также сослаться на обычную модель через реализованное поле контента. Например, пользовательское поле может быть жёстко связано с классом модели `Customer`. Это требует создания пользовательского поля Tailor, которое даст полный доступ к модели и таблицам базы данных.

Внутри класса поля контента метод `extendModelObject` позволяет расширить модель записи, а `extendDatabaseTable` — добавить столбец в таблицу.

```php
class MyContentField extends ContentFieldBase
{
    public function extendModelObject($model)
    {
        $model->belongsTo[$this->fieldName] = MyOtherModel::class;
    }

    public function extendDatabaseTable($table)
    {
        $table->integer($this->fieldName . '_id')->nullable();
    }
}
```

Потребуется больше подготовки, но итогом станет простое определение **type** поля с минимальной конфигурацией.

```yaml
myfield:
    label: My Field
    type: mycontentfield
```

Подробнее о создании [пользовательских полей Tailor](../../extend/tailor-fields.md).

#### См. также

::: also
* [Поле формы RecordFinder](../../element/form/widget-recordfinder.md)
* [Создание полей Tailor](../../extend/tailor-fields.md)
:::
