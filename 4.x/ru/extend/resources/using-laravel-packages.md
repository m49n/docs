# Использование пакетов Laravel

При подключении пакетов Laravel в плагинах October CMS следует учесть несколько моментов.

### Файлы конфигурации

Пакеты Laravel часто предоставляют конфигурационные файлы; продублируйте такую конфигурацию в каталоге плагина. Например, если файл называется **purifier.php** и содержит базовые значения.

```php
return [
    'encoding' => 'UTF-8',
    'finalize' => true,
    'cachePath' => storage_path('app/purifier'),
    'cacheFileMode' => 0755,
];
```

Скопируйте этот файл в каталог плагина, например **plugins/acme/blog/config/purifier.php**. Важно скопировать и поддерживать весь файл, поскольку отсутствующие ключи не наследуются из базовой конфигурации.

Затем перенесите содержимое конфигурации плагина в конфигурацию пакета внутри метода `boot()`.

```php
public function boot()
{
    Config::set('purifier', Config::get('acme.blog::purifier'));
}
```

Это установит все значения конфигурации пакета равными значениям конфигурации плагина. Следующие значения будут равны.

```php
Config::get('purifier.encoding') === Config::get('acme.blog::purifier.encoding');
```

Теперь можно предоставлять значения конфигурации пакета так же, как и обычные значения конфигурации плагина, используя [стандартный подход к конфигурации](../settings/file-settings.md).

### Псевдонимы и провайдеры служб

Если пакет Laravel содержит провайдеров служб (Service Providers) и псевдонимы (aliases), зарегистрируйте их вручную в плагине через фасад (facade) `App` в методе `register()`.

```php
public function register()
{
    // Register the aliases provided by the packages used by your plugin
    App::registerClassAlias('Purifier', \Mews\Purifier\Facades\Purifier::class);

    // Register the service providers provided by the packages used by your plugin
    App::register(\Mews\Purifier\PurifierServiceProvider::class);
}
```

### Миграции и модели

Пакеты Laravel, которые взаимодействуют с базой данных, часто включают собственные миграции (migrations) и модели Eloquent. Продублируйте эти миграции и модели в каталоге плагина.

Измените классы моделей так, чтобы они расширяли базовый класс `October\Rain\Database\Model`, а не базовый класс модели Eloquent Laravel. Это позволит использовать дополнительные возможности платформы October CMS.

Также рекомендуется переименовать таблицы базы данных и добавить к ним префикс с авторским кодом и именем плагина. Например, таблицу `posts` следует переименовать в `rainlab_blog_posts`.
