---
subtitle: Узнайте, как отправлять почту и создавать шаблоны.
---
# Отправка почты

В этой статье описано, как формировать содержимое писем, использовать макеты и частичные шаблоны, а также методы отправки. В письмах поддерживаются базовые теги и выражения Twig. Также поддерживается синтаксис Markdown, см. раздел [Использование HTML в Markdown](../services/parser.md).

## Содержимое сообщения

Письма в October CMS можно отправлять с помощью почтовых представлений или почтовых шаблонов. Представление размещается в файловой системе приложения или плагина в каталоге **/views**. Шаблон управляется через панель бэкенда в разделе **Settings → Mail Templates**.

При необходимости почтовые представления можно зарегистрировать в [регистрационном файле плагина](../extending.md) методом `registerMailTemplates`. Это автоматически создаст шаблон, который затем можно настроить через панель бэкенда.

### Определение шаблонов в панели бэкенда

Шаблоны писем, хранящиеся в базе данных, создаются в панели бэкенда через **Settings → Mail Templates**. Значение **code** служит уникальным идентификатором и не может быть изменено после создания.

Например, создав шаблон с кодом `my-template`, отправьте его следующим образом:

```php
Mail::send('my-template', $data, function($message) {
    // ...
});
```

::: tip
Если шаблон с указанным кодом отсутствует, будет предпринята попытка найти почтовое представление в файловой системе с тем же кодом.
:::

### Определение макетов в панели бэкенда

Почтовые макеты создаются через **Settings → Mail Templates** на вкладке **Layouts**. Они работают так же, как макеты CMS, и содержат каркас сообщения. Почтовые представления и шаблоны поддерживают использование макетов.

По умолчанию October CMS включает два макета.

Layout | Code | Description
------------- | ------------- | -------------
Default | `default` | Используется для писем, отправляемых на внешние адреса.
System | `system` | Используется для внутренних писем бэкенда.

### Определение представлений в файловой системе

Почтовые представления находятся в файловой системе, а код соответствует пути к файлу. Например, отправка письма с кодом `author.plugin::mail.message` использует содержимое следующего файла:

::: dir
├── plugins
|   └── author  _← Сегмент «author»_
|       └── myplugin  _← Сегмент «plugin»_
|           └── `views`
|               └── mail  _← Сегмент «mail»_
|                   └── message.htm  _← Сегмент «message»_
:::

Файл представления может содержать до трёх секций: **configuration**, **plain text** и **HTML markup**. Секции разделяются последовательностью `==`. Пример:

::: cmstemplate
```ini
subject = "Your product has been added to October CMS project"
```
```twig
Hi {{ name }},

Good news! User {{ user }} just added your product "{{ product }}" to a project.

This message was sent using no formatting (plain text)
```
```twig
<p>Hi {{ name }},</p>

<p>
    <strong>Good news!</strong>
    User {{ user }} just added your product "{{ product }}" to a project.
</p>

<p>This email was sent using formatting (HTML)</p>
```
:::

Секция **plain text** необязательна: представление может содержать только **configuration** и **HTML markup**. В качестве альтернативы поддерживается синтаксис Markdown.

::: cmstemplate
```ini
layout = "default"
subject = "Your product has been added to October CMS project"
```
```twig
Hi {{ name }},

**Good news!** User {{ user }} just added your product "{{ product }}" to a project.
```
:::

#### Секция configuration

В секции configuration задаются параметры почтового представления. Поддерживаются следующие свойства:

Property | Description
------------- | -------------
**subject** | Тема письма (обязательно).
**layout** | Код или представление макета (необязательно). По умолчанию — `default`.

### Регистрация шаблонов, макетов и частичных шаблонов

::: aside
Значение **code** в панели бэкенда совпадает с путём почтового представления. Например, `author.plugin:mail.message`.
:::

Почтовые представления можно зарегистрировать как шаблоны по умолчанию. Для этого переопределите метод `registerMailTemplates` в [регистрационном файле плагина](../extending.md). Шаблоны могут использовать частичные шаблоны и макеты, подобно темам CMS, — их регистрируют методами `registerMailLayouts` и `registerMailPartials`.

```php
public function registerMailTemplates()
{
    return [
        // ...Templates defined here
    ];
}

public function registerMailLayouts()
{
    return [
        // ...Layouts defined here
    ];
}

public function registerMailPartials()
{
    return [
        // ...Partials defined here
    ];
}
```

В панели бэкенда, после первого сохранения сгенерированного шаблона, при отправке писем для соответствующего кода будет использоваться настроенное содержимое. Зарегистрированные представления выступают как шаблоны по умолчанию (fallback).

#### Регистрация шаблона

Ключ `templates` в массиве регистрации используется для добавления представлений как почтовых шаблонов. Метод возвращает массив, где ключ — код шаблона, а значение — имя [пути к представлению](../services/response-view.md).

```php
public function registerMailTemplates()
{
    return [
        'rainlab.user:activate' => 'rainlab.user::mail.activate',
        'rainlab.user:restore' => 'rainlab.user::mail.restore',
    ];
}
```

При отправке используется код шаблона:

```php
Mail::send('rainlab.user:activate', ...);
```

#### Регистрация макета

Метод `registerMailLayouts` регистрирует макеты. Каждый макет должен иметь уникальный `code` и ссылаться на представление по умолчанию.

```php
public function registerMailLayouts()
{
    return [
        'marketing' => 'acme.blog::layouts.marketing',
        'notification' => 'acme.blog::layouts.notification',
    ];
}
```

Теперь макет можно указать в шаблонах по его коду.

::: cmstemplate
```ini
layout = "marketing"
```
```twig
Page contents...
```
:::

#### Регистрация частичного шаблона

Частичные шаблоны регистрируются методом `registerMailPartials`. Каждый частичный шаблон должен иметь уникальный `code` и ссылаться на представление по умолчанию.

```php
public function registerMailPartials()
{
    return [
        'tracking' => 'acme.blog::partials.tracking',
        'promotion' => 'acme.blog::partials.promotion',
    ];
}
```

Теперь частичный шаблон можно вызвать в письме тегом `{% partial %}` и кодом.

```twig
{% partial 'tracking' %}
```

#### Макеты на файловой системе

Чтобы ссылаться на файловый макет, укажите код представления в свойстве **layout**. В примере ниже используется макет `acme.blog::mail.custom-layout`.

```ini
layout = "acme.blog::mail.custom-layout"
subject = "Your product has been added to October CMS project"
==
...
```

Этот код загрузит содержимое макета из **plugins/acme/blog/views/mail/custom-layout.htm**. Пример содержимого:

```twig
<html>
<body>
    <h1>HTML Contents</h1>
    <div>
        {{ content|raw }}
    </div>
</body>
</html>
```

::: warning
Содержимое файловых макетов нельзя редактировать в панели администратора.
:::

### Глобальные переменные

Глобальные переменные для всех почтовых шаблонов регистрируются методом `View::share`.

```php
View::share('site_name', 'October CMS');
```

Этот код можно разместить в методе `register` или `boot` [регистрационного файла плагина](../plugin/registration.md). После этого переменная `{{ site_name }}` будет доступна во всех шаблонах.

## Отправка почты

Для отправки письма используйте метод `send` фасада `Mail`. Он принимает три аргумента. Первый — код, по которому находится представление или шаблон. Второй — массив данных для передачи в шаблон. Третий — `Closure`, который получает объект сообщения и позволяет настроить получателей, тему и другие параметры.

```php
// These variables are available inside the message as Twig
$vars = ['name' => 'Joe', 'user' => 'Mary'];

Mail::send('acme.blog:message', $vars, function($message) {
    $message->to('admin@domain.tld', 'Admin Person');
    $message->subject('This is a reminder');
});
```

Поскольку массив содержит ключ `name`, его значение можно вывести в шаблоне Twig.

```twig
{{ name }}
```

::: warning
Не передавайте переменную `message` в шаблон — она используется для встроенного подключения вложений.
:::

### Быстрая отправка

В October CMS есть вспомогательный метод `sendTo` для упрощения отправки.

```php
// Отправить на адрес без имени
Mail::sendTo('admin@domain.tld', 'acme.blog:message', $params);

// Отправить, используя свойства объекта
Mail::sendTo($user, 'acme.blog:message', $params);

// Отправить нескольким адресатам
Mail::sendTo(['admin@domain.tld' => 'Admin Person'], 'acme.blog:message', $params);

// Отправить простое письмо без параметров
Mail::rawTo('admin@domain.tld', 'Hello friend');
```

Первый аргумент `sendTo` определяет получателей и может принимать разные типы значений.

Type | Description
------------- | -------------
String | Один адрес без имени.
Array | Несколько адресатов, где ключ — адрес, значение — имя.
Object | Один объект-получатель; используется свойство *email*, а *name* — при наличии.
Collection | Коллекция объектов-получателей, как выше.

Полная сигнатура метода `sendTo`:

```php
Mail::sendTo($recipient, $message, $params, $callback, $options);
```

- `$recipient` — получатели, как описано выше.
- `$message` — имя шаблона или содержимое письма для отправки «как есть».
- `$params` — массив переменных для шаблона.
- `$callback` — функция с одним аргументом (builder), как в методе `send` (необязательно, по умолчанию null). Если значение не является callable, оно используется вместо `$options`.
- `$options` — дополнительные параметры отправки (массив, необязательно).

Поддерживаемые параметры `$options`:

Option | Description
------------- | -------------
**queue** | Указывает, поставить ли письмо в очередь или отправить сразу. По умолчанию `false`.
**bcc** | Добавлять получателей как Bcc или как обычных адресатов. По умолчанию `false`.

### Формирование сообщения

Как упоминалось, третий аргумент метода `send` — `Closure`, позволяющее настроить письмо: копии, скрытые копии и т. д.

```php
Mail::send('acme.blog:welcome', $vars, function($message) {
    $message->from('us@example.tld', 'October');
    $message->to('foo@example.tld')->cc('bar@example.tld');
});
```

Доступные методы объекта `$message`:

```php
$message->from($address, $name = null);
$message->sender($address, $name = null);
$message->to($address, $name = null);
$message->cc($address, $name = null);
$message->bcc($address, $name = null);
$message->replyTo($address, $name = null);
$message->subject($subject);
$message->priority($level);
$message->attach($pathToFile, array $options = []);

// Attach a file from a raw $data string...
$message->attachData($data, $name, array $options = []);
```

#### Текстовые письма

По умолчанию представление, переданное в метод `send`, считается HTML-представлением. Чтобы отправить текстовую версию дополнительно к HTML, используйте массив с ключом `text`.

```php
Mail::send('acme.blog:message', $data, $callback);
```

Если нужно отправить только текстовое письмо, укажите ключ `text` в массиве.

```php
Mail::send(['text' => 'acme.blog:text'], $data, $callback);
```

#### Отправка разобранных строк

Метод `raw` отправляет строку напрямую. Содержимое будет обработано Markdown.

```php
Mail::raw('Text to e-mail', function ($message) {
    //
});
```

Эта строка также обрабатывается Twig. Чтобы передать переменные, используйте метод `send`, передав содержимое в ключе `raw`.

```php
Mail::send(['raw' => 'Text to email'], $vars, function ($message) {
    //
});
```

#### Отправка сырых строк

Если передать массив с ключами `text` или `html`, письмо будет отправлено без макетов и обработки Markdown.

```php
Mail::raw([
    'text' => 'This is plain text',
    'html' => '<strong>This is HTML</strong>'
], function ($message) {
    //
});
```

### Вложения

Чтобы добавить вложение, используйте метод `attach` объекта `$message`. Первым аргументом передаётся полный путь к файлу.

```php
Mail::send('acme.blog:welcome', $data, function ($message) {
    //

    $message->attach($pathToFile);
});
```

Можно указать отображаемое имя и MIME-тип, передав `array` вторым аргументом:

```php
$message->attach($pathToFile, ['as' => $display, 'mime' => $mime]);
```

### Вложения inline

#### Встраивание изображения в письмо

Встраивать inline-изображения обычно сложно, но существует удобный способ прикрепить изображение и получить нужный CID. Используйте метод `embed` переменной `message` в шаблоне письма. Переменная `message` доступна во всех представлениях.

```twig
<body>
    Here is an image:

    <img src="{{ message.embed(pathToFile) }}">
</body>
```

Если используется очередь писем, путь к файлу должен быть абсолютным. Можно получить его с помощью [фильтра app](../../markup/filter/app.md):

```twig
<body>
    Here is an image:
    {% set pathToFile = 'storage/app/media/path/to/file.jpg'|app %}
    <img src="{{ message.embed(pathToFile) }}">
</body>
```

#### Встраивание сырых данных

Если есть строка с сырыми данными, используйте метод `embedData` переменной `message`:

```twig
<body>
    Here is an image from raw data:

    <img src="{{ message.embedData(data, name) }}">
</body>
```

## Очередь для писем

### Постановка письма в очередь

Отправка писем может заметно увеличить время ответа приложения, поэтому их часто ставят в очередь. Используйте встроенный [унифицированный API очередей](../services/queue.md). Для постановки письма в очередь используйте метод `queue` фасада `Mail`.

```php
Mail::queue('acme.blog:welcome', $data, function ($message) {
    //
});
```

Этот метод автоматически помещает задачу в очередь для фоновой отправки письма. Перед использованием настройте [очереди](../services/queue.md).

### Отложенная отправка

Чтобы отложить отправку письма в очереди, используйте метод `later`. Передайте количество секунд задержки первым аргументом.

```php
Mail::later(5, 'acme.blog:welcome', $data, function ($message) {
    //
});
```

### Отправка в конкретные очереди

Чтобы указать очередь, используйте методы `queueOn` и `laterOn`.

```php
Mail::queueOn('queue-name', 'acme.blog:welcome', $data, function ($message) {
    //
});

Mail::laterOn('queue-name', 5, 'acme.blog:welcome', $data, function ($message) {
    //
});
```

#### См. также

::: also
* [Настройка почты](../../setup/mail-config.md)
:::
