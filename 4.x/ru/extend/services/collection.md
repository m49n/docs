# Коллекция

Класс `October\\Rain\\Support\\Collection` предоставляет удобную, лаконичную обёртку для работы с массивами данных. Например, рассмотрим следующий код. Создадим новый экземпляр коллекции на основе массива, применим функцию `strtoupper` к каждому элементу, а затем удалим все пустые элементы.

```php
$collection = new October\\Rain\\Support\\Collection(['stewie', 'brian', null]);

$collection = $collection
    ->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    })
;
```

Класс `Collection` позволяет объединять методы в цепочку, выполняя удобное отображение и свёртку базового массива. Как правило, каждый метод `Collection` возвращает совершенно новый экземпляр `Collection`.

## Создание коллекций

Как описано выше, передача массива в конструктор класса `October\\Rain\\Support\\Collection` вернёт новый экземпляр для заданного массива. Создать коллекцию так же просто, как:

```php
$collection = new October\\Rain\\Support\\Collection([1, 2, 3]);
```

По умолчанию коллекции [моделей базы данных](../database/model.md) всегда возвращаются как экземпляры `Collection`; однако при желании можно использовать класс `Collection` в любом месте приложения.

## Доступные методы

В оставшейся части документации рассматривается каждый метод класса `Collection`. Помните, что все эти методы можно объединять в цепочки для удобной работы с базовым массивом. Кроме того, почти каждый метод возвращает новый экземпляр `Collection`, позволяя при необходимости сохранить исходную копию коллекции.

Можно выбрать любой метод из этой таблицы, чтобы увидеть пример его использования:

<div class="content-list-p" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[filter](#method-filter)
[first](#method-first)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[skip](#method-skip)
[slice](#method-slice)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[splice](#method-splice)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[whereNotNull](#method-wherenotnull)
[whereNull](#method-wherenull)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

## Список методов

<a name="method-all"></a>
#### `all()`

Метод `all` просто возвращает базовый массив, представленный коллекцией:

```php
$collection = new Collection([1, 2, 3]);

$collection->all();

// [1, 2, 3]
```

<a name="method-average"></a>
#### `average()`

Псевдоним метода [`avg`](#method-avg).

<a name="method-avg"></a>
#### `avg()`

Метод `avg` возвращает [среднее значение](https://en.wikipedia.org/wiki/Average) указанного ключа:

```php
$average = new Collection([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

// 20

$average = new Collection([1, 1, 2, 4])->avg();

// 2
```

<a name="method-chunk"></a>
#### `chunk()`

Метод `chunk` разбивает коллекцию на несколько меньших коллекций указанного размера:

```php
$collection = new Collection([1, 2, 3, 4, 5, 6, 7]);

$chunks = $collection->chunk(4);

$chunks->toArray();

// [[1, 2, 3, 4], [5, 6, 7]]
```

Этот метод особенно полезен на страницах CMS (CMS pages) при работе с системой сеток, например [Bootstrap](https://getbootstrap.tld/css/#grid). Представьте, что есть коллекция моделей, которые требуется вывести в сетке:

```twig
{% for chunk in products.chunk(3) %}
    <div class="row">
        {% for product in chunk %}
            <div class="col-4">{{ product.name }}</div>
        {% endfor %}
    </div>
{% endfor %}
```

<a name="method-collapse"></a>
#### `collapse()`

Метод `collapse` сворачивает коллекцию массивов в плоскую коллекцию:

```php
$collection = new Collection([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

$collapsed = $collection->collapse();

$collapsed->all();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

<a name="method-combine"></a>
#### `combine()`

Метод `combine` объединяет значения коллекции в качестве ключей со значениями другого массива или коллекции.

```php
$collection = new Collection(['name', 'age']);

$combined = $collection->combine(['George', 29]);

$combined->all();

// ['name' => 'George', 'age' => 29]
```

<a name="method-concat"></a>
#### `concat()`

Метод `concat` добавляет указанные значения `array` или коллекции в конец коллекции:

```php
$collection = new Collection(['John Doe']);

$concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

$concatenated->all();

// ['John Doe', 'Jane Doe', 'Johnny Doe']
```

<a name="method-contains"></a>
#### `contains()`

Метод `contains` определяет, содержит ли коллекция указанный элемент:

```php
$collection = new Collection(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk');

// true

$collection->contains('New York');

// false
```

Также можно передать парy ключ/значение методу `contains`, чтобы определить, присутствует ли указанная пара в коллекции:

```php
$collection = new Collection([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->contains('product', 'Bookcase');

// false
```

Наконец, можно передать колбэк методу `contains`, чтобы выполнить собственную проверку истинности:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$collection->contains(function ($value, $key) {
    return $value > 5;
});

// false
```

Метод `contains` использует «мягкое» сравнение значений элементов, то есть строка с целочисленным значением будет считаться равной целому числу с тем же значением. Используйте метод [`containsStrict`](#method-containsstrict), чтобы фильтровать с помощью «жёстких» сравнений.

<a name="method-containsstrict"></a>
#### `containsStrict()`

Этот метод имеет ту же сигнатуру, что и [`contains`](#method-contains), однако все значения сравниваются с помощью «жёстких» сравнений.

<a name="method-count"></a>
#### `count()`

Метод `count` возвращает общее количество элементов в коллекции:

```php
$collection = new Collection([1, 2, 3, 4]);

$collection->count();

// 4
```

<a name="method-countBy"></a>
#### `countBy()`

Метод `countBy` подсчитывает количество вхождений значений в коллекции. По умолчанию метод считает количество каждого элемента:

```php
$collection = new Collection([1, 2, 2, 2, 3]);

$counted = $collection->countBy();

$counted->all();

// [1 => 1, 2 => 3, 3 => 1]
```

Однако можно передать колбэк методу `countBy`, чтобы посчитать все элементы по пользовательскому значению:

```php
$collection = new Collection(['alice@gmail.tld', 'bob@yahoo.tld', 'carlos@gmail.tld']);

$counted = $collection->countBy(function ($email) {
    return substr(strrchr($email, "@"), 1);
});

$counted->all();

// ['gmail.tld' => 2, 'yahoo.tld' => 1]
```

<a name="method-crossjoin"></a>
#### `crossJoin()`

Метод `crossJoin` выполняет декартово произведение значений коллекции с указанными массивами или коллекциями, возвращая все возможные перестановки:

```php
$collection = new Collection([1, 2]);

$matrix = $collection->crossJoin(['a', 'b']);

$matrix->all();

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$collection = new Collection([1, 2]);

$matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

$matrix->all();

/*
    [
        [1, 'a', 'I'],
        [1, 'a', 'II'],
        [1, 'b', 'I'],
        [1, 'b', 'II'],
        [2, 'a', 'I'],
        [2, 'a', 'II'],
        [2, 'b', 'I'],
        [2, 'b', 'II'],
    ]
*/
```

<a name="method-dd"></a>
#### `dd()`

Метод `dd` выводит элементы коллекции и завершает выполнение скрипта:

```php
$collection = new Collection(['John Doe', 'Jane Doe']);

$collection->dd();

/*
    Collection {
        #items: array:2 [
            0 => "John Doe"
            1 => "Jane Doe"
        ]
    }
*/
```

Если требуется продолжить выполнение скрипта, используйте вместо этого метод [`dump`](#method-dump).

<a name="method-diff"></a>
#### `diff()`

Метод `diff` сравнивает коллекцию с другой коллекцией или обычным PHP-массивом `array`:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$diff = $collection->diff([2, 4, 6, 8]);

$diff->all();

// [1, 3, 5]
```

<a name="method-diffassoc"></a>
#### `diffAssoc()`

Метод `diffAssoc` сравнивает коллекцию с другой коллекцией или обычным PHP-массивом `array` по ключам и значениям. Метод вернёт пары ключ/значение из исходной коллекции, отсутствующие в указанной коллекции:

```php
$collection = new Collection([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6
]);

$diff = $collection->diffAssoc([
    'color' => 'yellow',
    'type' => 'fruit',
]);

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

<a name="method-diffkeys"></a>
#### `diffKeys()`

Метод `diffKeys` сравнивает коллекцию с другой коллекцией или обычным PHP-массивом `array` по ключам:

```php
$collection = new Collection([
    'one' => 10,
    'two' => 20,
    'three' => 30,
]);

$diff = $collection->diffKeys([
    'two' => 2,
    'three' => 3,
    'four' => 4,
]);

$diff->all();

// ['one' => 10]
```

<a name="method-dump"></a>
#### `dump()`

Метод `dump` выводит элементы коллекции, но не завершает выполнение скрипта:

```php
$collection = new Collection(['John Doe', 'Jane Doe']);

$collection->dump();

/*
    Collection {
        #items: array:2 [
            0 => "John Doe"
            1 => "Jane Doe"
        ]
    }
*/
```

<a name="method-duplicates"></a>
#### `duplicates()`

Метод `duplicates` извлекает и возвращает дубликаты значений с соответствующими ключами:

```php
$collection = new Collection([
    'email' => 'john@example.tld',
    'position' => 'Developer',
    'secondary' => 'Manager',
    'assistant' => 'Developer',
]);

$collection->duplicates();

// ['assistant' => 'Developer']
```

Можно указать ключ, по которому требуется определять дубликаты:

```php
$employees = new Collection([
    ['email' => 'john@example.tld', 'position' => 'Developer'],
    ['email' => 'jane@example.tld', 'position' => 'Designer'],
    ['email' => 'john@example.tld', 'position' => 'Designer'],
    ['email' => 'elaine@example.tld', 'position' => 'Developer'],
]);

$employees->duplicates('position');

// [2 => 'Developer']
```

<a name="method-duplicatesstrict"></a>
#### `duplicatesStrict()`

Этот метод имеет ту же сигнатуру, что и [`duplicates`](#method-duplicates), однако все значения сравниваются с помощью «жёстких» сравнений.

<a name="method-each"></a>
#### `each()`

Метод `each` итерирует элементы коллекции и передаёт каждый элемент в колбэк:

```php
$collection->each(function ($item, $key) {
    //
});
```

Если требуется прекратить итерацию по элементам, можно вернуть `false` из колбэка:

```php
$collection->each(function ($item, $key) {
    if (/* some condition */) {
        return false;
    }
});
```
<a name="method-every"></a>
#### `every()`

Метод `every` создаёт новую коллекцию, состоящую из каждого n-го элемента:

```php
$collection = new Collection(['a', 'b', 'c', 'd', 'e', 'f']);

$collection->every(4);

// ['a', 'e']
```

Необязательно можно передать смещение вторым аргументом:

```php
$collection->every(4, 1);

// ['b', 'f']
```

<a name="method-filter"></a>
#### `filter()`

Метод `filter` фильтрует коллекцию с помощью указанного колбэка, оставляя только элементы, которые проходят проверку истинности:

```php
$collection = new Collection([1, 2, 3, 4]);

$filtered = $collection->filter(function ($item) {
    return $item > 2;
});

$filtered->all();

// [3, 4]
```

Противоположный `filter` метод — [reject](#method-reject).

<a name="method-first"></a>
#### `first()`

Метод `first` возвращает первый элемент коллекции, который проходит указанную проверку истинности:

```php
new Collection([1, 2, 3, 4])->first(function ($value, $key) {
    return $value > 2;
});

// 3
```

Метод `first` можно вызвать без аргументов, чтобы получить первый элемент коллекции. Если коллекция пуста, возвращается `null`:

```php
new Collection([1, 2, 3, 4])->first();

// 1
```

<a name="method-first-where"></a>
#### `firstWhere()`

Метод `firstWhere` возвращает первый элемент коллекции с указанной парой ключ/значение:

```php
$collection = new Collection([
    ['name' => 'Regena', 'age' => null],
    ['name' => 'Linda', 'age' => 14],
    ['name' => 'Diego', 'age' => 23],
    ['name' => 'Linda', 'age' => 84],
]);

$collection->firstWhere('name', 'Linda');

// ['name' => 'Linda', 'age' => 14]
```

Метод `firstWhere` можно вызвать и с оператором:

```php
$collection->firstWhere('age', '>=', 18);

// ['name' => 'Diego', 'age' => 23]
```

Как и в методе [where](#method-where), можно передать методу `firstWhere` единственный аргумент. В этом случае `firstWhere` вернёт первый элемент, у которого значение указанного ключа является «истинным»:

```php
$collection->firstWhere('age');

// ['name' => 'Linda', 'age' => 14]
```

<a name="method-flatmap"></a>
#### `flatMap()`

Метод `flatMap` итерирует коллекцию и передаёт каждое значение в указанный колбэк. Колбэк может изменить элемент и вернуть его, формируя новую коллекцию изменённых элементов. Затем массив уплощается на один уровень:

```php
$collection = new Collection([
    ['name' => 'Sally'],
    ['school' => 'Harvard'],
    ['age' => 28]
]);

$flattened = $collection->flatMap(function ($values) {
    return array_map('strtoupper', $values);
});

$flattened->all();

// ['name' => 'SALLY', 'school' => 'HARVARD', 'age' => '28'];
```

<a name="method-flatten"></a>
#### `flatten()`

Метод `flatten` превращает многомерную коллекцию в одномерную:

```php
$collection = new Collection(['name' => 'peter', 'languages' => ['php', 'javascript']]);

$flattened = $collection->flatten();

$flattened->all();

// ['peter', 'php', 'javascript'];
```

<a name="method-flip"></a>
#### `flip()`

Метод `flip` меняет местами ключи коллекции и соответствующие им значения:

```php
$collection = new Collection(['name' => 'peter', 'platform' => 'october']);

$flipped = $collection->flip();

$flipped->all();

// ['peter' => 'name', 'october' => 'platform']
```

<a name="method-forget"></a>
#### `forget()`

Метод `forget` удаляет элемент из коллекции по его ключу:

```php
$collection = new Collection(['name' => 'peter', 'platform' => 'october']);

$collection->forget('name');

$collection->all();

// ['platform' => 'october']
```

> **Примечание:** в отличие от большинства других методов коллекции, `forget` не возвращает новую изменённую коллекцию; он изменяет коллекцию, на которой был вызван.

<a name="method-forpage"></a>
#### `forPage()`

Метод `forPage` возвращает новую коллекцию с элементами, которые должны быть показаны на указанном номере страницы:

```php
$collection = new Collection([1, 2, 3, 4, 5, 6, 7, 8, 9])->forPage(2, 3);

$collection->all();

// [4, 5, 6]
```

Метод принимает номер страницы и количество элементов на странице соответственно.

<a name="method-get"></a>
#### `get()`

Метод `get` возвращает элемент по указанному ключу. Если ключ не существует, возвращается `null`:

```php
$collection = new Collection(['name' => 'peter', 'platform' => 'october']);

$value = $collection->get('name');

// peter
```

Необязательно можно передать значение по умолчанию вторым аргументом:

```php
$collection = new Collection(['name' => 'peter', 'platform' => 'october']);

$value = $collection->get('foo', 'default-value');

// default-value
```

Можно даже передать колбэк в качестве значения по умолчанию. Результат колбэка будет возвращён, если указанный ключ отсутствует:

```php
$collection->get('email', function () {
    return 'default-value';
});
```

<a name="method-groupby"></a>
#### `groupBy()`

Метод `groupBy` группирует элементы коллекции по заданному ключу:

```php
$collection = new Collection([
    ['account_id' => 1, 'product' => 'Desk'],
    ['account_id' => 2, 'product' => 'Chair'],
    ['account_id' => 1, 'product' => 'Bookcase'],
]);

$grouped = $collection->groupBy('account_id');

$grouped->toArray();

/*
    [
        1 => [
            ['account_id' => 1, 'product' => 'Desk'],
            ['account_id' => 1, 'product' => 'Bookcase'],
        ],
        2 => [
            ['account_id' => 2, 'product' => 'Chair'],
        ],
    ]
*/
```

Можно передать массив ключей для создания многоуровневой группировки:

```php
$data = new Collection([
    ['user' => 1, 'role' => 'Admin', 'skill' => 'Manage'],
    ['user' => 1, 'role' => 'Admin', 'skill' => 'Create'],
    ['user' => 1, 'role' => 'Editor', 'skill' => 'Edit'],
    ['user' => 2, 'role' => 'Admin', 'skill' => 'Manage'],
]);

$data->groupBy(['user', 'role'])->toArray();

/*
    [
        1 => [
            'Admin' => [
                ['user' => 1, 'role' => 'Admin', 'skill' => 'Manage'],
                ['user' => 1, 'role' => 'Admin', 'skill' => 'Create'],
            ],
            'Editor' => [
                ['user' => 1, 'role' => 'Editor', 'skill' => 'Edit'],
            ],
        ],
        2 => [
            'Admin' => [
                ['user' => 2, 'role' => 'Admin', 'skill' => 'Manage'],
            ],
        ],
    ]
*/
```

Также можно передать колбэк:

```php
$grouped = $collection->groupBy(function ($item, $key) {
    return substr($item['product'], 0, 1);
});
```

<a name="method-has"></a>
#### `has()`

Метод `has` определяет, существует ли элемент с заданным ключом:

```php
$collection = new Collection(['account_id' => 1, 'product' => 'Desk']);

$collection->has('product');

// true

$collection->has('price');

// false
```

<a name="method-implode"></a>
#### `implode()`

Метод `implode` объединяет значения коллекции в строку:

```php
$collection = new Collection([
    ['account_id' => 1, 'product' => 'Chair'],
    ['account_id' => 2, 'product' => 'Desk'],
]);

$collection->implode('product', ', ');

// Chair, Desk
```

Если коллекция содержит простые строки или числа, достаточно передать «клей» единственным аргументом метода:

```php
new Collection([1, 2, 3, 4, 5])->implode('-');

// '1-2-3-4-5'
```

<a name="method-intersect"></a>
#### `intersect()`

Метод `intersect` удаляет значения, отсутствующие в переданном `array` или коллекции:

```php
$collection = new Collection(['Desk', 'Sofa', 'Chair']);

$intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

$intersect->all();

// [0 => 'Desk', 2 => 'Chair']
```

Как видно, результирующая коллекция сохраняет ключи исходной коллекции.

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()`

Метод `intersectByKeys` удаляет из исходной коллекции ключи, отсутствующие в переданном `array` или коллекции:

```php
$collection = new Collection([
    'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
]);

$intersect = $collection->intersectByKeys([
    'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
]);

$intersect->all();

// ['type' => 'screen', 'year' => 2009]
```

<a name="method-isempty"></a>
#### `isEmpty()`

Метод `isEmpty` возвращает `true`, если коллекция пуста; в противном случае возвращается `false`:

```php
new Collection([])->isEmpty();

// true
```

<a name="method-isnotempty"></a>
#### `isNotEmpty()`

Метод `isNotEmpty` возвращает `true`, если коллекция не пуста; иначе возвращается `false`:

```php
new Collection([])->isNotEmpty();

// false
```

<a name="method-join"></a>
#### `join()`

Метод `join` соединяет значения коллекции строкой:

```php
new Collection(['a', 'b', 'c'])->join(', '); // 'a, b, c'
new Collection(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'
new Collection(['a', 'b'])->join(', ', ' and '); // 'a and b'
new Collection(['a'])->join(', ', ' and '); // 'a'
new Collection([])->join(', ', ' and '); // ''
```

<a name="method-keyby"></a>
#### `keyBy()`

Переиндексирует коллекцию по указанному ключу:

```php
$collection = new Collection([
    ['product_id' => 'prod-100', 'name' => 'chair'],
    ['product_id' => 'prod-200', 'name' => 'desk'],
]);

$keyed = $collection->keyBy('product_id');

$keyed->all();

/*
    [
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Chair'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Desk'],
    ]
*/
```

Если у нескольких элементов одинаковый ключ, в новой коллекции останется только последний.

Можно передать собственный колбэк, который должен вернуть значение для ключа коллекции:

```php
$keyed = $collection->keyBy(function ($item) {
    return strtoupper($item['product_id']);
});

$keyed->all();

/*
    [
        'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Chair'],
        'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Desk'],
    ]
*/
```

<a name="method-keys"></a>
#### `keys()`

Метод `keys` возвращает все ключи коллекции:

```php
$collection = new Collection([
    'prod-100' => ['product_id' => 'prod-100', 'name' => 'Chair'],
    'prod-200' => ['product_id' => 'prod-200', 'name' => 'Desk'],
]);

$keys = $collection->keys();

$keys->all();

// ['prod-100', 'prod-200']
```

<a name="method-last"></a>
#### `last()`

Метод `last` возвращает последний элемент коллекции, который проходит указанную проверку истинности:

```php
new Collection([1, 2, 3, 4])->last(function ($key, $value) {
    return $value < 3;
});

// 2
```

Метод `last` можно вызвать без аргументов, чтобы получить последний элемент коллекции. Если коллекция пуста, возвращается `null`:

```php
new Collection([1, 2, 3, 4])->last();

// 4
```
<a name="method-map"></a>
#### `map()`

Метод `map` итерирует коллекцию и передаёт каждое значение в указанный колбэк. Колбэк может изменить элемент и вернуть его, формируя новую коллекцию изменённых элементов:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$multiplied = $collection->map(function ($item, $key) {
    return $item * 2;
});

$multiplied->all();

// [2, 4, 6, 8, 10]
```

> **Примечание:** как и большинство других методов коллекции, `map` возвращает новый экземпляр коллекции; он не изменяет коллекцию, на которой был вызван. Если требуется преобразовать исходную коллекцию, используйте метод [`transform`](#method-transform).

<a name="method-mapinto"></a>
#### `mapInto()`

Метод `mapInto()` итерирует коллекцию, создавая новый экземпляр указанного класса, передавая значение в конструктор:

```php
class Currency
{
    /**
     * Create a new currency instance.
     *
     * @param  string  $code
     * @return void
     */
    function __construct(string $code)
    {
        $this->code = $code;
    }
}

$collection = new Collection(['AUD', 'USD', 'GBP']);

$currencies = $collection->mapInto(Currency::class);

$currencies->all();

// [Currency('AUD'), Currency('USD'), Currency('GBP')]
```

<a name="method-mapspread"></a>
#### `mapSpread()`

Метод `mapSpread` итерирует элементы коллекции, передавая каждое вложенное значение в указанный колбэк. Колбэк может изменить элемент и вернуть его, формируя новую коллекцию изменённых элементов:

```php
$collection = new Collection([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

$chunks = $collection->chunk(2);

$sequence = $chunks->mapSpread(function ($even, $odd) {
    return $even + $odd;
});

$sequence->all();

// [1, 5, 9, 13, 17]
```

<a name="method-maptogroups"></a>
#### `mapToGroups()`

Метод `mapToGroups` группирует элементы коллекции по результату указанного колбэка. Колбэк должен вернуть ассоциативный массив, где ключ — имя группы, а значение — массив значений, которые будут добавлены в группу:

```php
$collection = new Collection([
    ['name' => 'John', 'department' => 'Sales'],
    ['name' => 'Jane', 'department' => 'Sales'],
    ['name' => 'Dave', 'department' => 'Support'],
]);

$grouped = $collection->mapToGroups(function ($item, $key) {
    return [$item['department'] => $item['name']];
});

$grouped->toArray();

/*
    [
        'Sales' => ['John', 'Jane'],
        'Support' => ['Dave'],
    ]
*/
```

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()`

Метод `mapWithKeys` итерирует коллекцию и передаёт каждое значение в указанный колбэк. Колбэк должен вернуть ассоциативный массив, содержащий единственную пару ключ/значение, которая будет добавлена в результатирующую коллекцию:

```php
$collection = new Collection([
    ['name' => 'John', 'department' => 'Sales'],
    ['name' => 'Jane', 'department' => 'Marketing'],
]);

$keyed = $collection->mapWithKeys(function ($item) {
    return [$item['name'] => $item['department']];
});

$keyed->all();

// ['John' => 'Sales', 'Jane' => 'Marketing']
```

<a name="method-max"></a>
#### `max()`

Метод `max` возвращает максимальное значение указанного ключа:

```php
$max = new Collection([['foo' => 10], ['foo' => 20]])->max('foo');

// 20

$max = new Collection([1, 2, 3, 4, 5])->max();

// 5
```

<a name="method-median"></a>
#### `median()`

Метод `median` возвращает [медиану](https://en.wikipedia.org/wiki/Median) указанного ключа:

```php
$median = new Collection([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

// 15

$median = new Collection([1, 1, 2, 4])->median();

// 1.5
```

<a name="method-merge"></a>
#### `merge()`

Метод `merge` объединяет переданный массив или коллекцию с исходной коллекцией. Если строковый ключ в переданных элементах совпадает со строковым ключом в исходной коллекции, значение переданных элементов перезапишет значение в исходной коллекции:

```php
$collection = new Collection(['product_id' => 1, 'price' => 100]);

$merged = $collection->merge(['price' => 200, 'discount' => false]);

$merged->all();

// ['product_id' => 1, 'price' => 200, 'discount' => false]
```

Если ключи переданных элементов числовые, значения будут добавлены в конец коллекции:

```php
$collection = new Collection(['Desk', 'Chair']);

$merged = $collection->merge(['Bookcase', 'Door']);

$merged->all();

// ['Desk', 'Chair', 'Bookcase', 'Door']
```

<a name="method-mergerecursive"></a>
#### `mergeRecursive()`

Метод `mergeRecursive` рекурсивно объединяет переданный массив или коллекцию с исходной коллекцией. Если строковый ключ в переданных элементах совпадает со строковым ключом исходной коллекции, значения этих ключей объединяются в массив, и процесс повторяется рекурсивно:

```php
$collection = new Collection(['product_id' => 1, 'price' => 100]);

$merged = $collection->mergeRecursive(['product_id' => 2, 'price' => 200, 'discount' => false]);

$merged->all();

// ['product_id' => [1, 2], 'price' => [100, 200], 'discount' => false]
```

<a name="method-min"></a>
#### `min()`

Метод `min` возвращает минимальное значение указанного ключа:

```php
$min = new Collection([['foo' => 10], ['foo' => 20]])->min('foo');

// 10

$min = new Collection([1, 2, 3, 4, 5])->min();

// 1
```

<a name="method-mode"></a>
#### `mode()`

Метод `mode` возвращает [моду](https://en.wikipedia.org/wiki/Mode_(statistics)) указанного ключа:

```php
$mode = new Collection([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

// [10]

$mode = new Collection([1, 1, 2, 4])->mode();

// [1]
```

<a name="method-nth"></a>
#### `nth()`

Метод `nth` создаёт новую коллекцию, состоящую из каждого n-го элемента:

```php
$collection = new Collection(['a', 'b', 'c', 'd', 'e', 'f']);

$collection->nth(4);

// ['a', 'e']
```

Необязательно можно передать смещение вторым аргументом:

```php
$collection->nth(4, 1);

// ['b', 'f']
```

<a name="method-only"></a>
#### `only()`

Метод `only` возвращает элементы коллекции с указанными ключами:

```php
$collection = new Collection(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

$filtered = $collection->only(['product_id', 'name']);

$filtered->all();

// ['product_id' => 1, 'name' => 'Desk']
```

Противоположный `only` метод — [except](#method-except).

<a name="method-pad"></a>
#### `pad()`

Метод `pad` заполняет массив указанным значением, пока он не достигнет заданного размера. Поведение аналогично функции PHP [array_pad](https://secure.php.net/manual/en/function.array-pad.php).

Чтобы дополнить слева, укажите отрицательный размер. Дополнение не выполняется, если абсолютное значение заданного размера меньше или равно длине массива:

```php
$collection = new Collection(['A', 'B', 'C']);

$filtered = $collection->pad(5, 0);

$filtered->all();

// ['A', 'B', 'C', 0, 0]

$filtered = $collection->pad(-5, 0);

$filtered->all();

// [0, 0, 'A', 'B', 'C']
```

<a name="method-partition"></a>
#### `partition()`

Метод `partition` можно сочетать с функцией PHP `list`, чтобы отделить элементы, проходящие указанную проверку истинности, от элементов, которые её не проходят:

```php
$collection = new Collection([1, 2, 3, 4, 5, 6]);

list($underThree, $equalOrAboveThree) = $collection->partition(function ($i) {
    return $i < 3;
});

$underThree->all();

// [1, 2]

$equalOrAboveThree->all();

// [3, 4, 5, 6]
```

<a name="method-pipe"></a>
#### `pipe()`

Метод `pipe` передаёт коллекцию в указанный колбэк и возвращает результат:

```php
$collection = new Collection([1, 2, 3]);

$piped = $collection->pipe(function ($collection) {
    return $collection->sum();
});

// 6
```

<a name="method-pluck"></a>
#### `pluck()`

Метод `pluck` извлекает все значения коллекции для указанного ключа:

```php
$collection = new Collection([
    ['product_id' => 'prod-100', 'name' => 'Chair'],
    ['product_id' => 'prod-200', 'name' => 'Desk'],
]);

$plucked = $collection->pluck('name');

$plucked->all();

// ['Chair', 'Desk']
```

Также можно указать ключи результирующей коллекции:

```php
$plucked = $collection->pluck('name', 'product_id');

$plucked->all();

// ['prod-100' => 'Desk', 'prod-200' => 'Chair']
```
<a name="method-pop"></a>
#### `pop()`

Метод `pop` удаляет и возвращает последний элемент коллекции:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$collection->pop();

// 5

$collection->all();

// [1, 2, 3, 4]
```

<a name="method-prepend"></a>
#### `prepend()`

Метод `prepend` добавляет элемент в начало коллекции:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$collection->prepend(0);

$collection->all();

// [0, 1, 2, 3, 4, 5]
```

Можно указать ключ, которым будет проиндексирован элемент:

```php
$collection->prepend('Desk', 'product');

$collection->all();

/*
    [
        'product' => 'Desk',
        0 => 0,
        1 => 1,
        2 => 2,
        3 => 3,
        4 => 4,
        5 => 5,
    ]
*/
```

<a name="method-pull"></a>
#### `pull()`

Метод `pull` удаляет и возвращает элемент коллекции по указанному ключу:

```php
$collection = new Collection(['product_id' => 'prod-100', 'name' => 'Desk']);

$value = $collection->pull('name');

// 'Desk'

$collection->all();

// ['product_id' => 'prod-100']
```

<a name="method-push"></a>
#### `push()`

Метод `push` добавляет элемент в конец коллекции:

```php
$collection = new Collection([1, 2, 3, 4]);

$collection->push(5);

$collection->all();

// [1, 2, 3, 4, 5]
```

<a name="method-put"></a>
#### `put()`

Метод `put` устанавливает элемент коллекции по указанному ключу:

```php
$collection = new Collection(['product_id' => 1, 'name' => 'Desk']);

$collection->put('price', 100);

$collection->all();

// ['product_id' => 1, 'name' => 'Desk', 'price' => 100]
```

<a name="method-random"></a>
#### `random()`

Метод `random` возвращает случайный элемент из коллекции:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$collection->random();

// 4 (случайное значение)
```

Можно передать количество элементов, которые требуется получить:

```php
$random = $collection->random(3);

$random->all();

// [2, 4, 5] (случайные значения)
```

<a name="method-reduce"></a>
#### `reduce()`

Метод `reduce` сводит коллекцию к единственному значению, передавая результат каждого шага следующему вызову колбэка:

```php
$collection = new Collection([1, 2, 3]);

$total = $collection->reduce(function ($carry, $item) {
    return $carry + $item;
});

// 6
```

На первой итерации значение `$carry` равно `null`; однако можно указать начальное значение, передав его вторым аргументом `reduce`:

```php
$collection->reduce(function ($carry, $item) {
    return $carry + $item;
}, 4);

// 10
```

<a name="method-reject"></a>
#### `reject()`

Метод `reject` фильтрует коллекцию, используя переданный колбэк. Колбэк должен вернуть `true` для элементов, которые нужно удалить из результирующей коллекции:

```php
$collection = new Collection([1, 2, 3, 4]);

$filtered = $collection->reject(function ($item) {
    return $item > 2;
});

$filtered->all();

// [1, 2]
```

Противоположный `reject` метод — [`filter`](#method-filter).

<a name="method-replace"></a>
#### `replace()`

Метод `replace` ведёт себя аналогично `merge`; однако помимо перезаписи совпадающих элементов со строковыми ключами он также перезаписывает элементы коллекции с совпадающими числовыми ключами:

```php
$collection = new Collection(['James', 'Scott', 'Dan']);

$replaced = $collection->replace([1 => 'Victoria', 3 => 'Finn']);

$replaced->all();

// ['James', 'Victoria', 'Dan', 'Finn']
```

<a name="method-replacerecursive"></a>
#### `replaceRecursive()`

Этот метод работает как `replace`, но рекурсивно спускается во вложенные массивы и применяет тот же процесс замены к внутренним значениям:

```php
$collection = new Collection(['George', 'Scott', ['James', 'Victoria', 'Finn']]);

$replaced = $collection->replaceRecursive(['Charlie', 2 => [1 => 'King']]);

$replaced->all();

// ['Charlie', 'Scott', ['James', 'King', 'Finn']]
```

<a name="method-reverse"></a>
#### `reverse()`

Метод `reverse` изменяет порядок элементов коллекции на обратный:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$reversed = $collection->reverse();

$reversed->all();

// [5, 4, 3, 2, 1]
```

<a name="method-search"></a>
#### `search()`

Метод `search` ищет в коллекции указанное значение и возвращает его ключ, если значение найдено. Если элемент не найден, возвращается `false`:

```php
$collection = new Collection([2, 4, 6, 8]);

$collection->search(4);

// 1
```

Поиск выполняется с «мягким» сравнением. Чтобы использовать жёсткое сравнение, передайте `true` вторым аргументом метода:

```php
$collection->search('4', true);

// false
```

Также можно передать собственный колбэк, чтобы найти первый элемент, который проходит проверку:

```php
$collection->search(function ($item, $key) {
    return $item > 5;
});

// 2
```

<a name="method-shift"></a>
#### `shift()`

Метод `shift` удаляет и возвращает первый элемент коллекции:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$collection->shift();

// 1

$collection->all();

// [2, 3, 4, 5]
```

<a name="method-shuffle"></a>
#### `shuffle()`

Метод `shuffle` случайным образом перемешивает элементы коллекции:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$shuffled = $collection->shuffle();

$shuffled->all();

// [3, 2, 5, 1, 4] (порядок выбирается случайным образом)
```

<a name="method-skip"></a>
#### `skip()`

Метод `skip` возвращает новую коллекцию без указанного количества первых элементов:

```php
$collection = new Collection([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$collection = $collection->skip(4);

$collection->all();

// [5, 6, 7, 8, 9, 10]
```
<a name="method-slice"></a>
#### `slice()`

Метод `slice` возвращает срез коллекции, начиная с указанного индекса:

```php
$collection = new Collection([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$slice = $collection->slice(4);

$slice->all();

// [5, 6, 7, 8, 9, 10]
```

Если нужно ограничить размер возвращаемого среза, передайте желаемый размер вторым аргументом метода:

```php
$slice = $collection->slice(4, 2);

$slice->all();

// [5, 6]
```

Возвращаемый срез по умолчанию сохраняет исходные ключи. Если ключи следует переиндексировать, воспользуйтесь методом [`values`](#method-values).

<a name="method-some"></a>
#### `some()`

Псевдоним метода [`contains`](#method-contains).

<a name="method-sort"></a>
#### `sort()`

Метод `sort` сортирует коллекцию:

```php
$collection = new Collection([5, 3, 1, 2, 4]);

$sorted = $collection->sort();

$sorted->values()->all();

// [1, 2, 3, 4, 5]
```

Отсортированная коллекция сохраняет исходные ключи массива. В примере выше использован метод [`values`](#method-values), чтобы сбросить ключи к последовательным индексам.

Для сортировки коллекции вложенных массивов или объектов используйте методы [`sortBy`](#method-sortby) и [`sortByDesc`](#method-sortbydesc).

Если требуется более сложный алгоритм сортировки, можно передать в `sort` собственный колбэк. Обратитесь к документации PHP по [`usort`](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters) — именно её вызывает метод `sort` коллекции под капотом.

<a name="method-sortby"></a>
#### `sortBy()`

Метод `sortBy` сортирует коллекцию по указанному ключу:

```php
$collection = new Collection([
    ['name' => 'Desk', 'price' => 200],
    ['name' => 'Chair', 'price' => 100],
    ['name' => 'Bookcase', 'price' => 150],
]);

$sorted = $collection->sortBy('price');

$sorted->values()->all();

/*
    [
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
        ['name' => 'Desk', 'price' => 200],
    ]
*/
```

Метод `sortBy` принимает колбэк для дополнительной настройки сортировки:

```php
$sorted = $collection->sortBy(function ($product, $key) {
    return $product['price'];
});
```

Можно отсортировать коллекцию по нескольким атрибутам, передав ассоциативный массив, где ключ — поле, а значение — `SORT_ASC` или `SORT_DESC`:

```php
$collection = new Collection([
    ['name' => 'Taylor Otwell', 'age' => 34],
    ['name' => 'Abigail Otwell', 'age' => 30],
    ['name' => 'Otwell Taylor', 'age' => 25],
]);

$sorted = $collection->sortBy([
    ['name', SORT_ASC],
    ['age', SORT_DESC],
]);
```

<a name="method-sortbydesc"></a>
#### `sortByDesc()`

Метод `sortByDesc` сортирует коллекцию по указанному ключу в порядке убывания:

```php
$collection = new Collection([
    ['name' => 'Desk', 'price' => 200],
    ['name' => 'Chair', 'price' => 100],
    ['name' => 'Bookcase', 'price' => 150],
]);

$sorted = $collection->sortByDesc('price');

$sorted->values()->all();

/*
    [
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Bookcase', 'price' => 150],
        ['name' => 'Chair', 'price' => 100],
    ]
*/
```

<a name="method-sortkeys"></a>
#### `sortKeys()`

Метод `sortKeys` сортирует коллекцию по ключам:

```php
$collection = new Collection([
    'id' => 22345,
    'first' => 'john',
    'last' => 'doe',
]);

$sorted = $collection->sortKeys();

$sorted->all();

/*
    [
        'first' => 'john',
        'id' => 22345,
        'last' => 'doe',
    ]
*/
```

<a name="method-sortkeysdesc"></a>
#### `sortKeysDesc()`

Метод `sortKeysDesc` сортирует коллекцию по ключам в порядке убывания:

```php
$collection = new Collection([
    'id' => 22345,
    'first' => 'john',
    'last' => 'doe',
]);

$sorted = $collection->sortKeysDesc();

$sorted->all();

/*
    [
        'last' => 'doe',
        'id' => 22345,
        'first' => 'john',
    ]
*/
```

<a name="method-splice"></a>
#### `splice()`

Метод `splice` удаляет и возвращает срез элементов, начиная с указанного индекса:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2);

$chunk->all();

// [3, 4, 5]

$collection->all();

// [1, 2]
```

Можно передать второй аргумент, чтобы ограничить размер возвращаемого фрагмента:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2, 1);

$chunk->all();

// [3]

$collection->all();

// [1, 2, 4, 5]
```

Кроме того, можно передать третий аргумент с новыми элементами, которые заменят удалённые элементы коллекции:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2, 1, [10, 11]);

$chunk->all();

// [3]

$collection->all();

// [1, 2, 10, 11, 4, 5]
```
<a name="method-split"></a>
#### `split()`

Метод `split` разбивает коллекцию на указанное количество групп:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$groups = $collection->split(3);

$groups->toArray();

// [[1, 2], [3, 4], [5]]
```

<a name="method-sum"></a>
#### `sum()`

Метод `sum` возвращает сумму всех элементов коллекции:

```php
new Collection([1, 2, 3, 4, 5])->sum();

// 15
```

Если коллекция содержит вложенные массивы или объекты, следует указать ключ, по которому нужно суммировать значения:

```php
$collection = new Collection([
    ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
    ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
]);

$collection->sum('pages');

// 1272
```

Кроме того, можно передать собственный колбэк, чтобы определить, какие значения коллекции нужно суммировать:

```php
$collection = new Collection([
    ['name' => 'Chair', 'colors' => ['Black']],
    ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
    ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
]);

$collection->sum(function ($product) {
    return count($product['colors']);
});

// 6
```

<a name="method-take"></a>
#### `take()`

Метод `take` возвращает новую коллекцию с указанным количеством элементов:

```php
$collection = new Collection([0, 1, 2, 3, 4, 5]);

$chunk = $collection->take(3);

$chunk->all();

// [0, 1, 2]
```

Можно передать отрицательное число, чтобы получить указанное количество элементов с конца коллекции:

```php
$collection = new Collection([0, 1, 2, 3, 4, 5]);

$chunk = $collection->take(-2);

$chunk->all();

// [4, 5]
```

<a name="method-tap"></a>
#### `tap()`

Метод `tap` передаёт коллекцию в указанный колбэк, позволяя «подключиться» к коллекции в определённый момент и выполнить действия с элементами, не изменяя саму коллекцию:

```php
new Collection([2, 4, 3, 1, 5])
    ->sort()
    ->tap(function ($collection) {
        Log::debug('Values after sorting', $collection->values()->toArray());
    })
    ->shift();

// 1
```

<a name="method-times"></a>
#### `times()`

Статический метод `times` создаёт новую коллекцию, вызывая колбэк указанное количество раз:

```php
$collection = Collection::times(10, function ($number) {
    return $number * 9;
});

$collection->all();

// [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]
```

<a name="method-toarray"></a>
#### `toArray()`

Метод `toArray` преобразует коллекцию в обычный PHP-массив `array`. Если элементы коллекции — [модели базы данных](../database/model.md), они также будут преобразованы в массивы:

```php
$collection = new Collection(['name' => 'Desk', 'price' => 200]);

$collection->toArray();

/*
    [
        ['name' => 'Desk', 'price' => 200],
    ]
*/
```

> **Примечание:** `toArray` также преобразует все вложенные объекты в массив. Если нужно получить базовый массив как есть, используйте вместо этого метод [`all`](#method-all).

<a name="method-tojson"></a>
#### `toJson()`

Метод `toJson` преобразует коллекцию в JSON:

```php
$collection = new Collection(['name' => 'Desk', 'price' => 200]);

$collection->toJson();

// '{"name":"Desk","price":200}'
```

<a name="method-transform"></a>
#### `transform()`

Метод `transform` итерирует коллекцию и вызывает указанный колбэк для каждого элемента. Элементы коллекции заменяются значениями, возвращёнными колбэком:

```php
$collection = new Collection([1, 2, 3, 4, 5]);

$collection->transform(function ($item, $key) {
    return $item * 2;
});

$collection->all();

// [2, 4, 6, 8, 10]
```

> **Примечание:** в отличие от большинства других методов коллекции, `transform` изменяет коллекцию, на которой был вызван. Если требуется создать новую коллекцию, используйте метод [`map`](#method-map).

<a name="method-union"></a>
#### `union()`

Метод `union` объединяет коллекцию с переданным массивом или коллекцией. Если ключ уже существует в исходной коллекции, значение не будет заменено:

```php
$collection = new Collection([
    'name' => 'Desk',
    'price' => 100,
]);

$union = $collection->union([
    'price' => 120,
    'discount' => false,
]);

$union->all();

// ['name' => 'Desk', 'price' => 100, 'discount' => false]
```

<a name="method-unique"></a>
#### `unique()`

Метод `unique` возвращает все уникальные элементы коллекции:

```php
$collection = new Collection([1, 1, 2, 2, 3, 4, 2]);

$unique = $collection->unique();

$unique->values()->all();

// [1, 2, 3, 4]
```

Метод `unique` использует «мягкое» сравнение элементов, то есть строка с целочисленным значением будет считаться равной целому числу с тем же значением. Используйте метод [`uniqueStrict`](#method-uniquestrict), чтобы фильтровать с помощью «жёстких» сравнений.

<a name="method-uniquestrict"></a>
#### `uniqueStrict()`

Этот метод имеет ту же сигнатуру, что и [`unique`](#method-unique), однако все значения сравниваются с помощью «жёстких» сравнений.

<a name="method-unless"></a>
#### `unless()`

Метод `unless` выполнит указанный колбэк, если первый аргумент метода не приводится к `true`:

```php
$collection = new Collection([1, 2, 3]);

$collection->unless(true, function ($collection) {
    return $collection->push(4);
});

$collection->unless(false, function ($collection) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 5]
```

Противоположный `unless` метод — [`when`](#method-when).

<a name="method-unlessempty"></a>
#### `unlessEmpty()`

Псевдоним метода [`whenNotEmpty`](#method-whennotempty).

<a name="method-unlessnotempty"></a>
#### `unlessNotEmpty()`

Псевдоним метода [`whenEmpty`](#method-whenempty).

<a name="method-unwrap"></a>
#### `unwrap()`

Статический метод `unwrap` возвращает базовые элементы коллекции из переданного значения, если это возможно:

```php
Collection::unwrap(new Collection('John Doe'));

// ['John Doe']

Collection::unwrap(['John Doe']);

// ['John Doe']

Collection::unwrap('John Doe');

// 'John Doe'
```

<a name="method-values"></a>
#### `values()`

Метод `values` возвращает новую коллекцию с ключами, сброшенными к последовательным целым числам:

```php
$collection = new Collection([
    10 => ['product' => 'Desk', 'price' => 200],
    11 => ['product' => 'Desk', 'price' => 200]
]);

$values = $collection->values();

$values->all();

/*
    [
        0 => ['product' => 'Desk', 'price' => 200],
        1 => ['product' => 'Desk', 'price' => 200],
    ]
*/
```

<a name="method-when"></a>
#### `when()`

Метод `when` выполнит указанный колбэк, если первый аргумент метода приводится к `true`:

```php
$collection = new Collection([1, 2, 3]);

$collection->when(true, function ($collection) {
    return $collection->push(4);
});

$collection->when(false, function ($collection) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 4]
```

Противоположный `when` метод — [`unless`](#method-unless).

<a name="method-whenempty"></a>
#### `whenEmpty()`

Метод `whenEmpty` выполнит указанный колбэк, если коллекция пуста:

```php
$collection = new Collection(['michael', 'tom']);

$collection->whenEmpty(function ($collection) {
    return $collection->push('steve');
});

$collection->all();

// ['michael', 'tom']

$collection = new Collection();

$collection->whenEmpty(function ($collection) {
    return $collection->push('steve');
});

$collection->all();

// ['steve']

$collection = new Collection(['michael', 'tom']);

$collection->whenEmpty(function ($collection) {
    return $collection->push('steve');
}, function ($collection) {
    return $collection->push('prince');
});

$collection->all();

// ['michael', 'tom', 'prince']
```

Противоположный `whenEmpty` метод — [`whenNotEmpty`](#method-whennotempty).

<a name="method-whennotempty"></a>
#### `whenNotEmpty()`

Метод `whenNotEmpty` выполнит указанный колбэк, если коллекция не пуста:

```php
$collection = new Collection(['michael', 'tom']);

$collection->whenNotEmpty(function ($collection) {
    return $collection->push('steve');
});

$collection->all();

// ['michael', 'tom', 'steve']

$collection = new Collection();

$collection->whenNotEmpty(function ($collection) {
    return $collection->push('steve');
});

$collection->all();

// []

$collection = new Collection();

$collection->whenNotEmpty(function ($collection) {
    return $collection->push('steve');
}, function ($collection) {
    return $collection->push('prince');
});

$collection->all();

// ['prince']
```

Противоположный `whenNotEmpty` метод — [`whenEmpty`](#method-whenempty).
<a name="method-where"></a>
#### `where()`

Метод `where` фильтрует коллекцию по указанной паре ключ/значение:

```php
$collection = new Collection([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->where('price', 100);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
*/
```

Метод `where` использует «мягкие» сравнения значений элементов, то есть строка с целочисленным значением будет считаться равной целому числу с тем же значением. Используйте метод [`whereStrict`](#method-wherestrict), чтобы фильтровать с помощью «жёстких» сравнений.

Необязательно можно передать оператор сравнения вторым параметром.

```php
$collection = new Collection([
    ['name' => 'Jim', 'deleted_at' => '2019-01-01 00:00:00'],
    ['name' => 'Sally', 'deleted_at' => '2019-01-02 00:00:00'],
    ['name' => 'Sue', 'deleted_at' => null],
]);

$filtered = $collection->where('deleted_at', '!=', null);

$filtered->all();

/*
    [
        ['name' => 'Jim', 'deleted_at' => '2019-01-01 00:00:00'],
        ['name' => 'Sally', 'deleted_at' => '2019-01-02 00:00:00'],
    ]
*/
```

<a name="method-wherestrict"></a>
#### `whereStrict()`

Этот метод имеет ту же сигнатуру, что и [`where`](#method-where), однако все значения сравниваются с помощью «жёстких» сравнений.

<a name="method-wherebetween"></a>
#### `whereBetween()`

Метод `whereBetween` фильтрует коллекцию по значению ключа в заданном диапазоне:

```php
$collection = new Collection([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereBetween('price', [100, 200]);

$filtered->all();

/*
    [
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]
*/
```

<a name="method-wherein"></a>
#### `whereIn()`

Метод `whereIn` фильтрует коллекцию по значению ключа, содержащемуся в переданном массиве:

```php
$collection = new Collection([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereIn('price', [150, 200]);

$filtered->all();

/*
    [
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Bookcase', 'price' => 150],
    ]
*/
```

Метод `whereIn` использует «мягкие» сравнения значений элементов, то есть строка с целочисленным значением будет считаться равной целому числу с тем же значением. Используйте метод [`whereInStrict`](#method-whereinstrict), чтобы фильтровать с помощью «жёстких» сравнений.

<a name="method-whereinstrict"></a>
#### `whereInStrict()`

Этот метод имеет ту же сигнатуру, что и [`whereIn`](#method-wherein), однако все значения сравниваются с помощью «жёстких» сравнений.

<a name="method-whereinstanceof"></a>
#### `whereInstanceOf()`

Метод `whereInstanceOf` фильтрует коллекцию, оставляя только элементы указанного класса:

```php
$collection = new Collection([
    new App\User,
    new App\User,
    new App\Post,
]);

$filtered = $collection->whereInstanceOf(App\User::class);

$filtered->all();

// [App\User, App\User]
```

<a name="method-wherenotbetween"></a>
#### `whereNotBetween()`

Метод `whereNotBetween` фильтрует коллекцию, исключая значения в заданном диапазоне:

```php
$collection = new Collection([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 80],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Pencil', 'price' => 30],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereNotBetween('price', [100, 200]);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 80],
        ['product' => 'Pencil', 'price' => 30],
    ]
*/
```

<a name="method-wherenotin"></a>
#### `whereNotIn()`

Метод `whereNotIn` фильтрует коллекцию по значению ключа, отсутствующему в переданном массиве:

```php
$collection = new Collection([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereNotIn('price', [150, 200]);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
*/
```

Метод `whereNotIn` использует «мягкие» сравнения значений элементов, то есть строка с целочисленным значением будет считаться равной целому числу с тем же значением. Используйте метод [`whereNotInStrict`](#method-wherenotinstrict), чтобы фильтровать с помощью «жёстких» сравнений.

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()`

Этот метод имеет ту же сигнатуру, что и [`whereNotIn`](#method-wherenotin), однако все значения сравниваются с помощью «жёстких» сравнений.

<a name="method-wherenotnull"></a>
#### `whereNotNull()`

Метод `whereNotNull` фильтрует элементы, у которых указанное поле не равно `null`:

```php
$collection = new Collection([
    ['name' => 'Desk'],
    ['name' => null],
    ['name' => 'Bookcase'],
]);

$filtered = $collection->whereNotNull('name');

$filtered->all();

/*
    [
        ['name' => 'Desk'],
        ['name' => 'Bookcase'],
    ]
*/
```

<a name="method-wherenull"></a>
#### `whereNull()`

Метод `whereNull` фильтрует элементы, у которых указанное поле равно `null`:

```php
$collection = new Collection([
    ['name' => 'Desk'],
    ['name' => null],
    ['name' => 'Bookcase'],
]);

$filtered = $collection->whereNull('name');

$filtered->all();

/*
    [
        ['name' => null],
    ]
*/
```

<a name="method-wrap"></a>
#### `wrap()`

Статический метод `wrap` оборачивает переданное значение в коллекцию, если это возможно:

```php
$collection = Collection::wrap('John Doe');

$collection->all();

// ['John Doe']

$collection = Collection::wrap(['John Doe']);

$collection->all();

// ['John Doe']

$collection = Collection::wrap(new Collection('John Doe'));

$collection->all();

// ['John Doe']
```

<a name="method-zip"></a>
#### `zip()`

Метод `zip` объединяет значения переданного массива со значениями исходной коллекции по соответствующим индексам:

```php
$collection = new Collection(['Chair', 'Desk']);

$zipped = $collection->zip([100, 200]);

$zipped->all();

// [['Chair', 100], ['Desk', 200]]
```
