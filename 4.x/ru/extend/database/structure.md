# Миграции и заполнение данных

Миграции и сидеры позволяют создавать, изменять и наполнять таблицы базы данных. Чаще всего они используются [в файле обновлений плагина](../system/plugins.md) и связаны с историей версий плагина. Все классы хранятся в каталоге `updates` плагина. Миграции рассказывают историю схемы БД, и эту историю можно воспроизвести вперёд и назад, создавая и удаляя таблицы.

Создать файл миграции можно командой генерации. Первый аргумент указывает автора и плагин, второй — имя миграции.

```bash
php artisan create:migration Acme.Blog CreatePostsTable
```

## Структура миграции

Файл миграции должен определять класс, наследующий `October\Rain\Database\Updates\Migration`, и содержащий два метода: `up` и `down`. Метод `up` добавляет таблицы, столбцы и индексы, а `down` должен отменять операции, выполненные в `up`. В обоих методах можно использовать конструктор схемы, чтобы выразительно создавать и изменять таблицы. Ниже пример миграции, создающей таблицу `october_blog_posts`:

```php
use October\Rain\Database\Schema\Blueprint;
use October\Rain\Database\Updates\Migration;

return new class extends Migration
{
    public function up()
    {
        Schema::create('october_blog_posts', function($table)
        {
            $table->increments('id');
            $table->string('title');
            $table->string('slug')->index();
            $table->text('excerpt')->nullable();
            $table->text('content');
            $table->timestamp('published_at')->nullable();
            $table->boolean('is_published')->default(false);
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::drop('october_blog_posts');
    }
}
```

## Создание таблиц

::: aside
При создании таблицы можно использовать любые методы конструктора схемы для определения столбцов (см. ниже).
:::

Чтобы создать новую таблицу, вызовите метод `create` фасада `Schema`. Он принимает два аргумента: имя таблицы и `Closure`, который получает объект для описания структуры.

```php
Schema::create('users', function ($table) {
    $table->increments('id');
});
```

Проверить существование таблицы или столбца можно методами `hasTable` и `hasColumn`.

```php
if (Schema::hasTable('users')) {
    //
}

if (Schema::hasColumn('users', 'email')) {
    //
}
```

### Подключение и механизм хранения

Чтобы выполнить операцию над схемой в подключении, отличном от подключения по умолчанию, используйте метод `connection`.

```php
Schema::connection('foo')->create('users', function ($table) {
    $table->increments('id');
});
```

Для указания механизма хранения установите свойство `engine` в конструкторе схемы.

```php
Schema::create('users', function ($table) {
    $table->engine = 'InnoDB';
    $table->increments('id');
});
```

## Переименование и удаление таблиц

Чтобы переименовать таблицу, используйте метод `rename`.

```php
Schema::rename($from, $to);
```

Для удаления таблицы доступны методы `drop` и `dropIfExists`.

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

## Создание столбцов

Для изменения существующей таблицы используйте метод `table` фасада `Schema`. Как и `create`, он принимает имя таблицы и `Closure`, в котором можно добавить столбцы:

```php
Schema::table('users', function ($table) {
    $table->string('email');
});
```

### Доступные типы столбцов

Конструктор схемы содержит множество типов столбцов, доступных при создании таблиц.

Команда | Описание
------------- | -------------
`$table->bigIncrements('id');` | Инкрементный первичный ключ с типом «UNSIGNED BIG INTEGER».
`$table->bigInteger('votes');` | Эквивалент BIGINT.
`$table->binary('data');` | Эквивалент BLOB.
`$table->boolean('confirmed');` | Эквивалент BOOLEAN.
`$table->char('name', 4);` | Эквивалент CHAR с длиной.
`$table->date('created_at');` | Эквивалент DATE.
`$table->dateTime('created_at');` | Эквивалент DATETIME.
`$table->decimal('amount', 5, 2);` | Эквивалент DECIMAL с точностью и масштабом.
`$table->double('column', 15, 8);` | Эквивалент DOUBLE с 15 цифрами всего и 8 после запятой.
`$table->enum('choices', ['foo', 'bar']);` | Эквивалент ENUM.
`$table->float('amount');` | Эквивалент FLOAT.
`$table->increments('id');` | Инкрементный первичный ключ с типом «UNSIGNED INTEGER».
`$table->integer('votes');` | Эквивалент INTEGER.
`$table->json('options');` | Эквивалент JSON.
`$table->jsonb('options');` | Эквивалент JSONB.
`$table->longText('description');` | Эквивалент LONGTEXT.
`$table->mediumInteger('numbers');` | Эквивалент MEDIUMINT.
`$table->mediumText('description');` | Эквивалент MEDIUMTEXT.
`$table->morphs('taggable');` | Добавляет INTEGER `taggable_id` и STRING `taggable_type`.
`$table->nullableTimestamps();` | Аналог `timestamps()`, но допускает NULL.
`$table->rememberToken();` | Добавляет `remember_token` (VARCHAR(100) NULL).
`$table->smallInteger('votes');` | Эквивалент SMALLINT.
`$table->softDeletes();` | Добавляет столбец `deleted_at` для мягкого удаления.
`$table->string('email');` | Эквивалент VARCHAR.
`$table->string('name', 100);` | Эквивалент VARCHAR с длиной.
`$table->text('description');` | Эквивалент TEXT.
`$table->time('sunrise');` | Эквивалент TIME.
`$table->tinyInteger('numbers');` | Эквивалент TINYINT.
`$table->timestamp('added_on');` | Эквивалент TIMESTAMP.
`$table->timestamps();` | Добавляет столбцы `created_at` и `updated_at`.

### Модификаторы столбцов

Помимо типов столбцов, существуют модификаторы, применяемые при добавлении. Например, чтобы сделать столбец допускающим `NULL`, вызовите метод `nullable`.

```php
Schema::table('users', function ($table) {
    $table->string('email')->nullable();
});
```

Ниже приведён список доступных модификаторов (индексные модификаторы не включены).

Модификатор | Описание
------------- | -------------
`->nullable()` | Разрешает вставку NULL.
`->default($value)` | Задаёт значение по умолчанию.
`->unsigned()` | Устанавливает `integer` в `UNSIGNED`.
`->first()` | Помещает столбец первым в таблице (только MySQL).
`->after('column')` | Помещает столбец после указанного (только MySQL).
`->comment('my comment')` | Добавляет комментарий к столбцу (только MySQL).

## Изменение столбцов

Метод `change` позволяет изменить тип существующего столбца или его параметры. Например, увеличим длину строкового столбца `name` с 25 до 50.

```php
Schema::table('users', function ($table) {
    $table->string('name', 50)->change();
});
```

Можно также сделать столбец допускающим `NULL`:

```php
Schema::table('users', function ($table) {
    $table->string('name', 50)->nullable()->change();
});
```

### Переименование столбцов

Для переименования столбца используйте метод `renameColumn` конструктора схемы.

```php
Schema::table('users', function ($table) {
    $table->renameColumn('from', 'to');
});
```

::: info
Переименование столбцов в таблице, содержащей столбец `enum`, пока не поддерживается.
:::

### Удаление столбцов

Чтобы удалить столбец, используйте метод `dropColumn` конструктора схемы.

```php
Schema::table('users', function ($table) {
    $table->dropColumn('votes');
});
```

Можно удалить несколько столбцов, передав массив их имён в `dropColumn`.

```php
Schema::table('users', function ($table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

## Создание индексов

Конструктор схемы поддерживает несколько типов индексов. Например, чтобы значения столбца были уникальны, добавьте к определению столбца метод `unique`.

```php
$table->string('email')->unique();
```

Индекс можно создать и отдельно:

```php
$table->unique('email');
```

Чтобы создать составной индекс, передайте массив столбцов.

```php
$table->index(['account_id', 'created_at']);
```

В большинстве случаев стоит явно указать имя индекса вторым аргументом, чтобы избежать автоматической генерации слишком длинного имени.

```php
$table->index(['account_id', 'created_at'], 'account_created');
```

#### Доступные типы индексов

Команда | Описание
------------- | -------------
`$table->primary('id');` | Добавляет первичный ключ.
`$table->primary(['first', 'last']);` | Добавляет составной ключ.
`$table->unique('email');` | Добавляет уникальный индекс.
`$table->index('state');` | Добавляет обычный индекс.

### Переименование индексов

Чтобы переименовать индекс, используйте метод `renameIndex` у схемы. Первым аргументом передаётся текущее имя индекса, вторым — желаемое.

```php
$table->renameIndex('from', 'to')
```

### Удаление индексов

Для удаления индекса необходимо указать его имя. Если имя не задано вручную, система сформирует его автоматически, объединяя имя таблицы, имя столбца и тип индекса. Примеры:

Команда | Описание
------------- | -------------
`$table->dropPrimary('users_id_primary');` | Удаляет первичный ключ из таблицы `users`.
`$table->dropUnique('users_email_unique');` | Удаляет уникальный индекс из таблицы `users`.
`$table->dropIndex('geo_state_index');` | Удаляет обычный индекс из таблицы `geo`.

### Внешние ключи

Конструктор схемы поддерживает создание внешних ключей, обеспечивающих ссылочную целостность на уровне базы данных. Например, добавим столбец `user_id` в таблицу `posts`, который ссылается на `id` таблицы `users`:

```php
Schema::table('posts', function ($table) {
    $table->integer('user_id')->unsigned();

    $table->foreign('user_id')->references('id')->on('users');
});
```

Как и раньше, имя ограничения можно задать вручную, передав второй аргумент методу `foreign`:

```php
$table->foreign('user_id', 'user_foreign')
    ->references('id')
    ->on('users');
```

Можно указать действие для свойств `on delete` и `on update`:

```php
$table->foreign('user_id')
    ->references('id')
    ->on('users')
    ->onDelete('cascade');
```

Чтобы удалить внешний ключ, используйте метод `dropForeign`. Имена внешних ключей формируются по тому же принципу, что и имена индексов: имя таблицы + столбцы + суффикс `_foreign`.

```php
$table->dropForeign('posts_user_id_foreign');
```

## Структура сидера

Как и миграция, класс сидера по умолчанию содержит один метод `run` и наследует класс `Seeder`. Метод `run` вызывается при выполнении обновления. Внутри метода можно добавлять данные в базу любым удобным способом: через [конструктор запросов](./query.md) или [модели](./model.md). В примере ниже создаётся новый пользователь с помощью модели `User`.

```php
<?php namespace Acme\Users\Updates;

use Seeder;
use Acme\Users\Models\User;

class SeedUsersTable extends Seeder
{
    public function run()
    {
        $user = User::create([
            'email' => 'user@example.tld',
            'login' => 'user',
            'password' => 'password123',
            'password_confirmation' => 'password123',
            'first_name' => 'Actual',
            'last_name' => 'Person',
            'is_activated' => true
        ]);
    }
}
```

Аналогичный результат можно получить методом `Db::table` из [конструктора запросов](./query.md).

```php
public function run()
{
    $user = Db::table('users')->insert([
        'email' => 'user@example.tld',
        'login' => 'user',
        // ...
    ]);
}
```

### Вызов дополнительных сидеров

В классе `DatabaseSeeder` можно вызывать другие сидеры методом `call`. Это позволяет разбить наполнение базы на несколько файлов, чтобы один класс не разрастался. Просто передайте имя класса сидера.

```php
public function run()
{
    $this->call(\Acme\Users\Updates\UserTableSeeder::class);
    $this->call(\Acme\Users\Updates\PostsTableSeeder::class);
    $this->call(\Acme\Users\Updates\CommentsTableSeeder::class);
}
```
