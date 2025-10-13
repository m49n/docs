---
subtitle: Как перенести существующее приложение или проект Laravel в October CMS
---
# Миграция проекта Laravel

Следующее руководство поможет установить October CMS v3 поверх существующего приложения Laravel 9. Это полезно, если вы хотите сохранить ту же базу данных без потери данных или предпочитаете использовать Laravel в качестве отправной точки.

Ключевые шаги включают замену пакета Illuminate из Laravel на пакет Rain из October, который представляет расширенную версию Laravel и добавляет необходимые базовые возможности для запуска October CMS.

::: tip
Следуйте инструкции внимательно — в названиях классов есть небольшие различия.
:::

## Установите Laravel 9/10, а затем October Rain

Для начала предположим, что мы выполняем чистую установку Laravel следующей командой:

```bash
composer create-project laravel/laravel:^9.0 mylaravel
```

В созданном каталоге подключите библиотеку October CMS Rain.

```bash
cd mylaravel
composer require october/rain
```

## Аутентификация и установка October CMS

Авторизуйтесь в шлюзе October CMS, установив ключ проекта.

```bash
php artisan project:set <лицензионный_ключ>
```

Подключите все модули October CMS.

```bash
composer require october/all
```

Когда Composer спросит о доверии к пакету composer/installers, ответьте `Y`.

```bash
Do you trust "composer/installers" to execute code and wish to enable it now?
```

## Замените ссылки на Illuminate библиотекой Rain

Следующие шаги заменяют Illuminate на Rain.

### Обновите контейнер приложения

В файле **bootstrap/app.php** класс `Illuminate\Foundation\Application` нужно заменить на `October\Rain\Foundation\Application`.

```bash
// Файл bootstrap/app.php

// Было
$app = new Illuminate\Foundation\Application(
    $_ENV['APP_BASE_PATH'] ?? dirname(__DIR__)
);

// Стало
$app = new October\Rain\Foundation\Application(
    $_ENV['APP_BASE_PATH'] ?? dirname(__DIR__)
);
```

### Обновите HTTP-ядро

В файле **app/Http/Kernel.php** класс `App\Http\Kernel` должен наследовать `October\Rain\Foundation\Http\Kernel`.

```bash
// Файл app/Http/Kernel.php

// Было
use Illuminate\Foundation\Http\Kernel as HttpKernel;

// Стало
use October\Rain\Foundation\Http\Kernel as HttpKernel;
```

### Обновите консольное ядро

В файле **app/Console/Kernel.php** класс `App\Console\Kernel` должен наследовать `October\Rain\Foundation\Console\Kernel`.

```bash
// Файл app/Console/Kernel.php

// Было
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

// Стало
use October\Rain\Foundation\Console\Kernel as ConsoleKernel;
```

### Обновите обработчик исключений

В файле **app/Exceptions/Handler.php** класс `App\Exceptions\Handler` должен наследовать `October\Rain\Foundation\Exception\Handler`.

```bash
// Файл app/Exceptions/Handler.php

// Было
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

// Стало
use October\Rain\Foundation\Exception\Handler as ExceptionHandler;
```

## Публикация файлов October CMS

Следующие шаги скопируют файлы из поставки October CMS. Чтобы опубликовать их, выполните:

1. Откройте [репозиторий October CMS](https://github.com/octobercms/october)
1. Перейдите в раздел **Code → Download ZIP**
1. Сохраните архив и распакуйте его локально

Скопируйте из архива следующие каталоги.

- **config/**
- **plugins/**
- **themes/**

## Завершающие шаги

Предполагая, что база данных настроена и работает, запустите миграции October CMS.

```bash
php artisan october:migrate
```

Затем, чтобы сделать страницы CMS доступными во фронтенде, удалите или закомментируйте маршрут по умолчанию в файле **routes/web.php**.

Теперь можно открыть маршрут `/backend`, чтобы создать учётную запись администратора.

### Дополнительные шаги

Необязательные действия по настройке системы:

- Если вы планируете использовать стандартный путь Laravel для представлений, раскомментируйте соответствующую настройку в файле **config/view.php**.

#### Смотрите также

::: also
* [Установка Laravel 9](https://laravel.com/docs/9.x/installation)
* [Установка Laravel 10](https://laravel.com/docs/12.x/installation)
:::
