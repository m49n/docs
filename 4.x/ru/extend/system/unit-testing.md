---
subtitle: Тестируйте бизнес-логику программно и повышайте надёжность.
---
# Модульное тестирование

Тесты отдельных плагинов запускаются командой `plugin:test` с указанием кода плагина. Например, следующая команда выполнит тесты из каталога **plugins/acme/demo**.

```bash
php artisan plugin:test acme.demo
```

::: tip
Если `phpunit` установлен глобально, команду можно запустить из каталога плагина.
:::

## Создание тестов плагина

Сначала создайте файл **phpunit.xml** в корневом каталоге плагина. Ниже пример **/plugins/acme/blog/phpunit.xml** для плагина `Acme.Blog`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit
    backupGlobals="false"
    backupStaticAttributes="false"
    bootstrap="../../../modules/system/tests/bootstrap.php"
    colors="true"
    convertErrorsToExceptions="true"
    convertNoticesToExceptions="true"
    convertWarningsToExceptions="true"
    processIsolation="false"
    stopOnFailure="false"
>
    <testsuites>
        <testsuite name="Plugin Test Suite">
            <directory>./tests</directory>
            <exclude>./tests/browser</exclude>
        </testsuite>
    </testsuites>
    <php>
        <env name="APP_ENV" value="testing" />
        <env name="CACHE_DRIVER" value="array" />
        <env name="SESSION_DRIVER" value="array" />
        <env name="ACTIVE_THEME" value="test" />
        <env name="CONVERT_LINE_ENDINGS" value="true" />
        <env name="CMS_ROUTE_CACHE" value="true" />
        <env name="CMS_TWIG_CACHE" value="false" />
        <env name="ENABLE_CSRF" value="false" />
        <env name="DB_CONNECTION" value="sqlite" />
        <env name="DB_DATABASE" value=":memory:" />
    </php>
</phpunit>
```

## Создание тестового класса

Команда `create:test` генерирует тестовый класс. Первый аргумент задаёт автора и плагин, второй — имя класса, которое должно оканчиваться на **Test**.

```bash
php artisan create:test Acme.Blog UserTest
```

Все тесты следует помещать в каталог **tests**, где хранятся тестовые классы. Имя класса должно оканчиваться на `Test`. Пространство имён необязательно. Класс теста должен наследовать `PluginTestCase` — специальный класс, подготавливающий базу данных October CMS в памяти в методе `setUp`.

```php
use Acme\Blog\Models\Post;

class PostTest extends PluginTestCase
{
    public function testCreateFirstPost()
    {
        $post = Post::create(['title' => 'Hi!']);
        $this->assertEquals(1, $post->id);
    }
}
```

## Регистрация и загрузка плагинов

В тестовой среде плагин и его зависимости регистрируются и загружаются автоматически. Это даёт полный контроль и предотвращает влияние других плагинов, например регистрацию событий. Чтобы отключить автоматическую загрузку текущего плагина, установите свойство `autoRegister` в `false`.

```php
/**
 * @var bool autoRegister отключён для этого теста.
 */
protected $autoRegister = false;
```

Плагин можно зарегистрировать и загрузить вручную методом `loadPlugin`. В некоторых случаях потребуется вручную выполнить миграции (см. ниже).

```php
public function setUp(): void
{
    parent::setUp();

    // Выполняет методы register() и boot()
    $this->loadPlugin('Acme.Blog');
}
```

Доступны следующие методы регистрации:

Method Name | Purpose
------------- | -------------
**loadAllPlugins()** | Загружает все плагины в системе.
**loadCurrentPlugin()** | Загружает текущий плагин и его зависимости.
**loadPlugin($code)** | Загружает плагин по коду, например `Acme.Blog`.
**loadPlugins($codes)** | Загружает несколько плагинов по массиву кодов.

## Работа с базой данных

По умолчанию тесты автоматически выполняют миграции таблиц модулей ядра, текущего плагина и его зависимостей. Это эквивалентно выполнению команд перед каждым тестом:

```bash
php artisan october:migrate
php artisan plugin:refresh Acme.Blog
[php artisan plugin:refresh <dependency>, ...]
```

Можно отключить эту функцию, установив свойство `autoMigrate` в `false` в тестовом классе. Это актуально, если тест не использует базу данных.

```php
class PostTest extends PluginTestCase
{
    /**
     * @var bool autoMigrate отключён для этого теста.
     */
    protected $autoMigrate = false;
}
```

Миграции можно выполнить вручную методом `migratePlugin` при подготовке теста. Метод `migrateModules` делает доступными таблицы системы.

```php
public function setUp(): void
{
    parent::setUp();

    // Миграция модулей ядра
    $this->migrateModules();

    // Миграция плагина блога
    $this->migratePlugin('Acme.Blog');
}
```

Доступные методы миграции:

Method Name | Purpose
------------- | -------------
**migrateDatabase()** | Мигрирует всю базу данных (как `october:migrate`).
**migrateModules()** | Мигрирует только модули ядра.
**migrateCurrentPlugin()** | Мигрирует текущий плагин и его зависимости.
**migratePlugin($code)** | Мигрирует указанный плагин по коду, например `Acme.Blog`.

### Смена базы данных

По умолчанию модульные тесты используют SQLite в памяти. Настройку можно изменить в `phpunit.xml`. Значения соответствуют параметрам из `/config/database.php`.

```xml
<env name="DB_CONNECTION" value="sqlite" />
<env name="DB_DATABASE" value=":memory:" />
```
