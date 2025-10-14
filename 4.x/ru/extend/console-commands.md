---
subtitle: Создавайте собственные команды для запуска в консоли.
---
# Создание консольных команд

Чтобы добавить собственные команды для работы с приложением, разместите их в каталоге плагина **console**. Сгенерировать файл класса можно командой генератора. Первый аргумент задаёт автора и название плагина. Второй аргумент задаёт имя команды.

```bash
php artisan create:command Acme.Blog MyCommand
```

## Создание команды

Чтобы создать консольную команду `acme:mycommand`, определите класс команды в файле **plugins/acme/blog/console/MyCommand.php** и добавьте следующий код в качестве основы:

```php
namespace Acme\Blog\Console;

use Illuminate\Console\Command;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class MyCommand extends Command
{
    /**
     * @var string signature for the console command.
     */
    protected $signature = 'acme:mycommand {user}';

    /**
     * @var string description for the console command.
     */
    protected $description = 'Does something cool.';

    /**
     * handle executes the console command.
     */
    public function handle()
    {
        $username = $this->argument('user');

        $this->output->writeln("Hello {$username}!");
    }
}
```

После создания класса заполните свойства `signature` и `description` — они используются при отображении команды на экране `list`.

Метод `handle` вызывается при выполнении команды. В нём размещается логика команды.

### Определение аргументов

Все пользовательские аргументы и опции указываются в сигнатуре и заключаются в фигурные скобки. Аргументы описываются в сигнатуре в фигурных скобках — укажите любые аргументы, которые принимает команда. Например:

```php
protected $signature = 'mail:send {user}';
```

Сделать аргумент необязательным можно, добавив вопросительный знак (`?`) после имени аргумента.

```php
protected $signature = 'mail:send {user?}';
```

Также можно задать значение по умолчанию, указав знак равенства (`=`) и нужное значение.

```php
protected $signature = 'mail:send {user=foo}';
```

### Определение опций

Опции, как и аргументы, являются формой пользовательского ввода и обозначаются двумя дефисами (`--`) в сигнатуре. Опции могут принимать значение; если значение не указано, они работают как булев переключатель. Например, переключатель **queue**.

```php
protected $signature = 'mail:send {user} {--queue}';
```

В этом примере переключатель `--queue` можно указать при вызове команды. Если передан `--queue`, значение опции будет `true`. Иначе значение будет `false`.

```bash
php artisan mail:send 1 --queue
```

Когда опция ожидает значение, добавьте знак равенства (`=`) после имени опции.

```php
protected $signature = 'mail:send {user} {--queue=}';
```

В этом случае опция может принимать значение; если оно не задано, значение будет `null`.

```bash
php artisan mail:send 1 --queue=default
```

Также можно определить значение по умолчанию, указав знак равенства (`=`) и нужное значение.

```php
protected $signature = 'mail:send {user} {--queue=default}';
```

Сокращения позволяют использовать более короткий синтаксис для опций.

```php
protected $signature = 'mail:send {user} {--Q|queue}';
```

При использовании сокращения есть важное отличие: опция вызывается с одним дефисом (`-`), и для передачи значения знак равенства не используется.

```bash
php artisan mail:send 1 -Qdefault
```

### Получение ввода

Во время работы команды потребуется получать значения аргументов и опций. Для этого используйте методы `argument` и `option`.

Передайте имя аргумента методу `argument`, чтобы получить его значение.

```php
$value = $this->argument('name');
```

Без аргумента метод вернёт все аргументы.

```php
$arguments = $this->argument();
```

Аналогично, передача имени методу `option` вернёт значение опции.

```php
$value = $this->option('name');
```

Без аргумента метод вернёт все опции.

```php
$options = $this->option();
```

### Вывод в консоль

Чтобы отправить вывод в консоль, используйте методы `info`, `comment`, `question` и `error`. Каждый метод применяет подходящий ANSI-цвет по своему назначению.

Метод `info` отправляет информационное сообщение пользователю.

```php
$this->info('Display this on the screen');
```

Метод `error` выводит сообщение об ошибке.

```php
$this->error('Something went wrong!');
```

Также доступны методы `ask` и `confirm` для запроса ввода у пользователя.

```php
$name = $this->ask('What is your name?');
```

Метод `secret` предназначен для запроса скрытого ввода.

```php
$password = $this->secret('What is the password?');
```

Метод `confirm` запрашивает подтверждение и возвращает `true`, если пользователь согласился.

```php
if ($this->confirm('Do you wish to continue? [yes|no]')) {
    //
}
```

Методу `confirm` можно передать значение по умолчанию (`true` или `false`).

```php
$this->confirm($question, true);
```

## Регистрация команд

#### Регистрация консольной команды

После завершения работы над классом команду нужно зарегистрировать, чтобы её можно было использовать. Обычно это делается в методе `register` [регистрационного файла плагина](./extending.md) с помощью вспомогательного метода `registerConsoleCommand`.

```php
class Blog extends PluginBase
{
    public function pluginDetails()
    {
        // ...
    }

    public function register()
    {
        $this->registerConsoleCommand('acme.mycommand', \Acme\Blog\Console\MyConsoleCommand::class);
    }
}
```

Альтернативно плагин может содержать файл **init.php** в корне каталога плагина, где можно разместить логику регистрации команд. Внутри файла используйте метод `Artisan::add`, чтобы зарегистрировать команду.

```php
Artisan::add(new Acme\Blog\Console\MyCommand);
```

#### Регистрация команды в контейнере приложения

Если команда зарегистрирована в [контейнере приложения](./services/application.md), можно использовать метод `Artisan::resolve`, чтобы сделать её доступной Artisan.

```php
Artisan::resolve('binding.name');
```

## Вызов других команд

Иногда требуется вызвать другие команды из своей команды. Сделать это можно методом `call`.

```php
$this->call('october:migrate');
```

Также можно передавать аргументы в виде массива.

```php
$this->call('plugin:refresh', ['namespace' => 'October.Demo']);
```

И опции.

```php
$this->call('october:update', ['--force' => true]);
```

#### См. также

::: also
* [Laravel Artisan Console Documentation](https://laravel.com/docs/12.x/artisan)
:::
