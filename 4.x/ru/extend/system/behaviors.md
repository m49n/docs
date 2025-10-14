---
subtitle: Комбинируйте классы с общей функциональностью.
---
# Поведения

Поведения добавляют классам возможность иметь приватные трейты, похожие на [встроенные трейты PHP](http://php.net/manual/en/language.oop5.traits.php), но с рядом важных преимуществ.

1. Поведения используют собственный конструктор.
1. Поведения могут содержать приватные или защищённые методы.
1. Имена методов и свойств могут конфликтовать без последствий.
1. Классы можно динамически расширять поведениями.

## Сравнение с трейтами

Вместо трейта PHP можно использовать такую конструкцию.

```php
class MyClass
{
    use \October\Rain\UtilityFunctions;
    use \October\Rain\DeferredBinding;
}
```

Поведение используется аналогичным образом.

```php
class MyClass extends \October\Rain\Extension\Extendable
{
    public $implement = [
        \October\Rain\UtilityFunctions::class,
        \October\Rain\DeferredBinding::class,
    ];
}
```

Определение трейта могло бы выглядеть так.

```php
trait UtilityFunctions
{
    public function sayHello()
    {
        echo "Hello from " . get_class($this);
    }
}
```

Поведение определяется следующим образом.

```php
class UtilityFunctions extends \October\Rain\Extension\ExtensionBase
{
    protected $parent;

    public function __construct($parent)
    {
        $this->parent = $parent;
    }

    public function sayHello()
    {
        echo "Hello from " . get_class($this->parent);
    }
}
```

Расширяемый объект всегда передаётся первым аргументом конструктора. В итоге при работе с поведениями следует расширять следующие классы.

- Наследуйте `October\Rain\Extension\ExtensionBase`, чтобы объявить класс поведением.
- Наследуйте `October\Rain\Extension\Extendable` в классе, который реализует поведение.

## Расширение конструкторов

Любой класс, расширяющий `Extendable`, может дополнить свой конструктор статическим методом `extend`. Аргументом необходимо передать замыкание, которое будет вызвано как часть конструктора класса.

```php
MyClass::extend(function($controller) {
    //
});
```

### Мягкое определение

Если класс поведения отсутствует, как и для трейта, будет выброшено исключение **class not found**. Иногда требуется подавить эту ошибку и подключать поведение условно, только если оно присутствует в системе. Для этого добавьте символ `@` в начало имени класса. В примере ниже, если класс `RainLab\Translate\Behaviors\TranslatableModel` отсутствует, ошибка не возникнет.

```php
class User extends \October\Rain\Extension\Extendable
{
    public $implement = [
        '@'.\RainLab\Translate\Behaviors\TranslatableModel::class
    ];
}
```

## Динамическое подключение поведения

Благодаря уникальной возможности расширять конструкторы поведения можно подключать динамически. Например, используйте метод `implementClassWith`, чтобы реализовать класс `RelationController` в контроллере `UsersController`.

```php
UsersController::extend(function($controller) {
    $controller->implementClassWith(\Backend\Behaviors\RelationController::class);
});
```

::: tip
Метод `implementClassWith` можно безопасно вызывать несколько раз: он проверяет, реализовано ли поведение, и при необходимости подключает его.
:::

Используйте `extendClassWith`, чтобы расширить объект после его создания. Следующий пример динамически расширяет класс модели, используя имя класса, сохранённое в базе данных.

```php
public function afterFetch()
{
    $this->extendClassWith($this->class_type);
}
```

Чтобы проверить, расширен ли объект поведением, вызовите на нём метод `isClassExtendedWith`.

```php
$controller->isClassExtendedWith(\Backend\Behaviors\RelationController::class);
```

Чтобы явно вызвать метод поведения, используйте `asExtension`, передав базовое имя класса или полностью квалифицированное имя.

```php
echo $controller->asExtension('RelationController')->otherMethod();
```

## Динамическое создание методов

Методы можно добавить расширяемому объекту, вызвав `addDynamicMethod` и передав имя метода и вызываемый объект, например `Closure`.

```php
MyModel::extend(function($model) {
    $model->addDynamicMethod('getTagsAttribute', function() use ($model) {
        return '...';
    });
});
```

### Проверка существования метода

Чтобы проверить наличие метода в классе `Extendable`, вызовите `methodExists` — аналог функции PHP `method_exists()`. Метод обнаруживает стандартные методы, методы из поведений и динамические методы, добавленные через `addDynamicMethod`.

```php
$post = new Post;
$post->methodExists('getTagsAttribute'); // true
$post->methodExists('missingMethod'); // false
```

### Список доступных методов

Чтобы получить список всех доступных методов класса `Extendable`, используйте `getClassMethods`. Метод работает аналогично функции PHP `get_class_methods()`: возвращает массив доступных методов, включая определённые в классе, предоставленные расширениями и добавленные через `addDynamicMethod`.

```php
$post = new Post;
$methods = $post->getClassMethods();

/**
 * $methods = [
 *   0 => '__construct',
 *   1 => 'extend',
 *   2 => 'getTagsAttribute',
 *   ...
 * ];
 */
```

## Пример использования

### Класс поведения / расширения

```php
namespace MyNamespace\Behaviors;

class FormController extends \October\Rain\Extension\ExtensionBase
{
    /**
     * @var Controller controller содержит ссылку на расширяемый объект.
     */
    protected $controller;

    /**
     * __construct
     */
    public function __construct($controller)
    {
        $this->controller = $controller;
    }

    public function someMethod()
    {
        return "I come from the FormController Behavior!";
    }

    public function otherMethod()
    {
        return "You might not see me...";
    }
}
```

### Расширение класса

Этот класс `Controller` реализует поведение `FormController`, после чего методы становятся доступными (подмешиваются) в класс. Мы переопределим метод `otherMethod`.

```php
<?php namespace MyNamespace;

class Controller extends \October\Rain\Extension\Extendable
{
    /**
     * Реализация поведения FormController
     */
    public $implement = [
        \MyNamespace\Behaviors\FormController::class
    ];

    public function otherMethod()
    {
        return "I come from the main Controller!";
    }
}
```

### Использование расширения

```php
$controller = new MyNamespace\Controller;

// Prints: I come from the FormController Behavior!
echo $controller->someMethod();

// Prints: I come from the main Controller!
echo $controller->otherMethod();

// Prints: You might not see me...
echo $controller->asExtension('FormController')->otherMethod();
```
