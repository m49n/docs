# Коллекции

Все наборы с несколькими результатами, которые возвращает модель, представляют экземпляр объекта `October\\Rain\\Database\\Collection`, включая результаты, полученные методом `get` или через связи. Объект `Collection` расширяет [базовую коллекцию](../services/collections.md), поэтому он наследует десятки методов для удобной работы с базовым массивом моделей.

Все коллекции также являются итераторами, что позволяет перебирать их так же, как простые массивы PHP.

```php
$users = User::where('is_active', true)->get();

foreach ($users as $user) {
    echo $user->name;
}
```

Однако коллекции гораздо мощнее массивов и предоставляют множество операций отображения и свёртки с интуитивным интерфейсом. Например, отфильтруем активные модели и соберём имя каждого пользователя, прошедшего фильтр.

```php
$users = User::get();

$names = $users->filter(function ($user) {
        return $user->is_active === true;
    })
    ->map(function ($user) {
        return $user->name;
    });
```

::: tip
Хотя большинство методов коллекции моделей возвращают новый экземпляр коллекции `Eloquent`, методы `pluck`, `keys`, `zip`, `collapse`, `flatten` и `flip` возвращают экземпляр базовой коллекции. Аналогично, если операция `map` возвращает коллекцию, не содержащую моделей, она автоматически преобразуется в базовую коллекцию.
:::

## Доступные методы

Все коллекции моделей наследуют базовый объект коллекции, поэтому у них доступны все мощные методы, предоставляемые базовым классом коллекции.

Кроме того, класс `October\\Rain\\Database\\Collection` предоставляет расширенный набор методов для работы с коллекциями моделей. Большинство методов возвращают экземпляры `October\\Rain\\Database\\Collection`; однако некоторые методы возвращают базовый экземпляр `Illuminate\\Support\\Collection`.

**contains($key, $operator = null, $value = null)**

Метод `contains` позволяет определить, содержит ли коллекция указанный экземпляр модели. Метод принимает первичный ключ или экземпляр модели:

```php
$users->contains(1);

$users->contains(User::find(1));
```

**diff($items)**

Метод `diff` возвращает все модели, отсутствующие в переданной коллекции:

```php
use App\\User;

$users = $users->diff(User::whereIn('id', [1, 2, 3])->get());
```

**except($keys)**

Метод `except` возвращает все модели, у которых отсутствуют указанные первичные ключи:

```php
$users = $users->except([1, 2, 3]);
```

**find($key)**

Метод `find` находит модель с указанным первичным ключом. Если `$key` — экземпляр модели, `find` пытается вернуть модель с совпадающим первичным ключом. Если `$key` — массив ключей, `find` вернёт все модели, соответствующие `$keys`, с использованием `whereIn()`:

```php
$users = User::all();

$user = $users->find(1);
```

**fresh($with = [])**

Метод `fresh` извлекает из базы данных обновлённый экземпляр каждой модели в коллекции. Дополнительно можно указать связи для жадной загрузки:

```php
$users = $users->fresh();

$users = $users->fresh('comments');
```

**intersect($items)**

Метод `intersect` возвращает все модели, которые также присутствуют в переданной коллекции:

```php
use App\\User;

$users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());
```

**load($relations)**

Метод `load` выполняет жадную загрузку указанных связей для всех моделей коллекции:

```php
$users->load('comments', 'posts');

$users->load('comments.author');
```

**loadMissing($relations)**

Метод `loadMissing` выполняет жадную загрузку указанных связей для всех моделей в коллекции, если эти связи ещё не загружены:

```php
$users->loadMissing('comments', 'posts');

$users->loadMissing('comments.author');
```

**modelKeys()**

Метод `modelKeys` возвращает первичные ключи всех моделей в коллекции:

```php
$users->modelKeys();

// [1, 2, 3, 4, 5]
```

**makeVisible($attributes)**

Метод `makeVisible` делает видимыми атрибуты, которые обычно скрыты у каждой модели коллекции:

```php
$users = $users->makeVisible(['address', 'phone_number']);
```

**makeHidden($attributes)**

Метод `makeHidden` скрывает атрибуты, которые обычно видимы у каждой модели коллекции:

```php
$users = $users->makeHidden(['address', 'phone_number']);
```

**only($keys)**

Метод `only` возвращает все модели, у которых есть указанные первичные ключи:

```php
$users = $users->only([1, 2, 3]);
```

**unique($key = null, $strict = false)**

Метод `unique` возвращает все уникальные модели коллекции. Любые модели того же типа с тем же первичным ключом, что и у другой модели коллекции, удаляются.

```php
$users = $users->unique();
```

## Пользовательские коллекции

Если нужен собственный объект `Collection` с дополнительными методами, переопределите метод `newCollection` в модели:

```php
class User extends Model
{
    /**
     * Create a new Collection instance.
     */
    public function newCollection(array $models = [])
    {
        return new CustomCollection($models);
    }
}
```

После определения метода `newCollection` экземпляр пользовательской коллекции будет возвращаться каждый раз, когда модель возвращает `Collection`. Если требуется использовать пользовательскую коллекцию для каждой модели в плагине или приложении, переопределите метод `newCollection` в базовой модели, от которой наследуются все модели.

```php
use October\\Rain\\Database\\Collection as CollectionBase;

class CustomCollection extends CollectionBase
{
}
```
