# Мутаторы

Аксессоры и мутаторы позволяют форматировать атрибуты при чтении из модели или при их установке. Например, можно использовать [сервис шифрования](../services/hash-crypt.md), чтобы зашифровать значение при сохранении в базе, а при обращении к атрибуту на модели автоматически расшифровать его.

Помимо пользовательских аксессоров и мутаторов, можно автоматически приводить поля даты к экземплярам [Carbon](https://github.com/briannesbitt/Carbon) или даже приводить текстовые значения к JSON.

## Аксессоры и мутаторы

#### Определение аксессора

Чтобы определить аксессор, создайте в модели метод `getFooAttribute`, где `Foo` — имя столбца в верблюжьем регистре. В этом примере мы определим аксессор для атрибута `first_name`. Аксессор будет вызываться автоматически при попытке получить значение `first_name`:

```php
namespace Acme\Blog\Models;

use Model;

class User extends Model
{
    /**
     * getFirstNameAttribute is available as `first_name` on the model
     */
    public function getFirstNameAttribute($value)
    {
        return strtolower($value);
    }
}
```

Как видно, оригинальное значение «Johnny» преобразовано в «johnny». Этот метод доступен через динамическое свойство `first_name` на экземпляре модели:

```php
$user = User::find(1);

echo $user->first_name;
```

#### Определение мутатора

Чтобы определить мутатор, создайте метод `setFooAttribute` в модели, где `Foo` — имя столбца в верблюжьем регистре. В этом примере зададим мутатор для `first_name`.

```php
namespace Acme\Blog\Models;

use Model;

class User extends Model
{
    /**
     * setFirstNameAttribute is available as `first_name` on the model
     */
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = strtolower($value);
    }
}
```

Как и раньше, мутатор доступен через динамическое свойство `first_name` на экземпляре модели:

```php
$user = User::find(1);

$user->first_name = 'Johnny';

$user->save();
```

Мутаторы особенно полезны при загрузке файлов. Рассмотрим модель, которая принимает файл и автоматически загружает его в каталог `storage/app/uploads`.

```php
class FileModel extends Model
{
    /**
     * setUploadedFilePathAttribute stores the uploaded file
     */
    public function setUploadedFilePathAttribute($value)
    {
        $uploadedFile = Input::file('uploaded_file');

        $path = $uploadedFile->store('uploads');

        $this->attributes['uploaded_file_path'] = $path;
    }
}
```

Теперь можно создавать модели с новыми файлами. При сохранении модели вы увидите, что файл загружен и путь записан в атрибут `uploaded_file_path`.

```php
$model = new FileModel;

$model->uploaded_file_path = 'some/file/path.txt';

$model->save();
```

#### Глобальные мутаторы

В дополнение к определению аксессоров и мутаторов на уровне модели можно регистрировать события модели, чтобы изменить базовое поведение системы.

```php
Event::listen('eloquent.booted: October\Rain\Database\Model', function ($model) {
    $model->bindEvent('model.setAttribute', function ($attribute, $value) use ($model) {
        if ($attribute === 'first_name') {
            $model->attributes['first_name'] = strtolower($value);
        }
    });
});
```

## Мутаторы даты

По умолчанию модели October преобразуют столбцы `created_at` и `updated_at` в экземпляры объекта [Carbon](https://github.com/briannesbitt/Carbon), который содержит множество полезных методов и расширяет встроенный класс PHP `DateTime`.

Список полей, которые автоматически преобразуются, можно настроить или полностью отключить, переопределив свойство `$dates` в модели:

```php
class User extends Model
{
    /**
     * @var array dates return as \Carbon\Carbon instances
     */
    protected $dates = ['created_at', 'updated_at', 'disabled_at'];
}
```

Если столбец считается датой, ему можно присвоить UNIX-метку времени, строку даты (`Y-m-d`), строку даты и времени или экземпляр `DateTime` / `Carbon`. Значение будет автоматически сохранено в базе в корректном виде.

```php
$user = User::find(1);

$user->disabled_at = Carbon::now();

$user->save();
```

Как отмечено выше, при получении атрибутов, перечисленных в `$dates`, они автоматически приводятся к экземплярам [Carbon](https://github.com/briannesbitt/Carbon), что позволяет использовать любые методы Carbon.

```php
$user = User::find(1);

return $user->disabled_at->getTimestamp();
```

По умолчанию метки времени форматируются как `'Y-m-d H:i:s'`. Чтобы настроить формат, задайте свойство `$dateFormat` в модели. Оно определяет, как даты сохраняются в базе и как сериализуются при преобразовании модели в массив или JSON:

```php
class Flight extends Model
{
    /**
     * @var string dateFormat for storage of the model's date columns.
     */
    protected $dateFormat = 'U';
}
```

## Приведение атрибутов

Свойство `$casts` в модели — удобный способ приводить атрибуты к стандартным типам данных. `$casts` — это массив, где ключ — имя приводимого атрибута, а значение — тип, к которому нужно привести столбец. Поддерживаются типы `integer`, `real`, `float`, `double`, `string`, `boolean`, `object` и `array`.

Например, приведём атрибут `is_admin`, который хранится в базе как целое число (`0` или `1`), к логическому значению.

```php
class User extends Model
{
    /**
     * @var array casts attributes to native types.
     */
    protected $casts = [
        'is_admin' => 'boolean',
    ];
}
```

Теперь атрибут `is_admin` всегда приводится к логическому типу при доступе к нему, даже если исходное значение в базе — целое число.

```php
$user = User::find(1);

if ($user->is_admin) {
    //
}
```

#### Приведение к массиву

Тип приведения `array` особенно полезен для столбцов, где хранятся сериализованные данные JSON. Например, если в базе есть поле типа `TEXT` с сериализованным JSON, добавление приведения `array` автоматически десериализует атрибут в массив PHP при обращении к нему в модели Eloquent:

```php
class User extends Model
{
    /**
     * @var array casts attributes to native types.
     */
    protected $casts = [
        'options' => 'array',
    ];
}
```

После добавления приведения можно обратиться к атрибуту `options`, и он автоматически будет преобразован из JSON в массив PHP. При установке значения атрибута `options` переданный массив будет автоматически сериализован обратно в JSON для хранения:

```php
$user = User::find(1);

$options = $user->options;

$options['key'] = 'value';

$user->options = $options;

$user->save();
```
