---
subtitle: Узнайте, как настроить и защитить сервер, а также повысить производительность приложения.
---
# Конфигурация веб-сервера

## Безопасность и производительность

### Публичный каталог

В типовой конфигурации каталог October CMS находится в корне веб-доступа. Для максимальной безопасности в продакшн-средах настройте веб-сервер так, чтобы использовался публичный каталог, и только файлы из определённых директорий были доступны извне.

Для начала создайте публичный каталог командой `october:mirror`:

```bash
php artisan october:mirror
```

Команда создаст новый каталог `public` в корне проекта. Внутри него появятся символьные ссылки на каталоги ресурсов и ассетов всех плагинов, модулей и тем, установленных в проекте.

::: tip
В Apache расположение корневого каталога виртуального хоста задаётся директивой `DocumentRoot`.
:::

Обновите конфигурацию веб-сервера, указав в качестве корня каталог `public`, а не корневой каталог проекта.

::: aside
В операционных системах Windows команду `october:mirror` можно выполнить только в консоли, запущенной от имени администратора.
:::

Команду `october:mirror` следует выполнять после каждого обновления системы или установки нового плагина. Можно настроить October CMS на автоматический запуск команды после обновления проекта через Composer. За автоматическое зеркалирование отвечает параметр конфигурации `system.auto_mirror_public`.

### Повышение производительности

Следующие действия помогают увеличить производительность приложения и рекомендуются для всех боевых окружений — они существенно сокращают время загрузки страниц.

В конфигурации отключите [режим отладки](../setup/configuration.html#debug-mode) и включите кэширование. Например, если вы используете файл `.env`:

```ini
APP_DEBUG=false
CMS_ROUTE_CACHE=true
CMS_ASSET_CACHE=true
CMS_TWIG_CACHE=true
```

В командной строке выполните команды для кэширования структуры системы:

```bash
php artisan october:optimize

composer dump-autoload --optimize
```

### Безопасность на виртуальном хостинге

На виртуальном хостинге необходимо предпринять дополнительные шаги, чтобы защитить файлы проекта от других пользователей на сервере.

::: warning
Уточните у хостинг-провайдера подходящие маски прав. Главное правило: файлы приложения не должны быть доступны другим пользователям. Все файлы должны быть доступны и управляемы владельцем и веб-сервером. Конфигурационные файлы должны быть доступны владельцу и веб-серверу, но веб-сервер не должен иметь права изменять их.
:::

October CMS может автоматически задавать права для новых файлов и каталогов. Права по умолчанию настраиваются параметрами `system.default_mask.file` и `system.default_mask.folder`. Например, если вы задаёте значения в файле `.env`:

```ini
DEFAULT_FILE_MASK=644
DEFAULT_FOLDER_MASK=755
```

### Использование обратного прокси

При использовании обратного прокси (например, CloudFlare) хост-сервер может применять небезопасный протокол внутри сети, и October CMS будет отражать это при генерации ссылок. В результате в ответе могут появиться смешанные ссылки `http://` и `https://`. Используйте настройку `system.link_policy`, чтобы принудительно включить HTTPS (`secure`) везде.

```ini
LINK_POLICY=secure
```

Также можно принудительно использовать URL приложения для всех ссылок. Он определяется в конфигурации `app.url`.

```ini
LINK_POLICY=force
```

### Безопасный режим

Безопасный режим — дополнительная защита, которая запрещает выполнение произвольного PHP-кода, отключая PHP-раздел в редакторе. Включение безопасного режима также активирует защищённую среду Twig, запрещающую небезопасные вызовы методов.

Параметр `cms.safe_mode` находится в файле `config/cms.php`. По умолчанию значение считывается из переменной окружения `CMS_SAFE_MODE`. Безопасный режим отключает [PHP-раздел](../../cms/themes.md#php-code-section) в шаблонах CMS.

Параметр принимает следующие значения:

- `true` — безопасный режим включён;
- `false` — безопасный режим отключён;
- `null` — безопасный режим активен, если [режим отладки](../setup/configuration.md#debug-mode) выключен.

Если вы планируете использовать безопасный режим в продакшне, включите его и в разработке, чтобы убедиться, что тема корректно работает в защищённой среде Twig. Возможно, потребуется изменить плагины, добавив интерфейсы `October\\Contracts\\Twig\\CallsAnyMethod` и `October\\Contracts\\Twig\\CallsMethods`, чтобы разрешить вызовы методов.

В качестве альтернативы можно выбрать более мягкую политику Twig, задав `cms.security_policy_v1`, которая вместо этого блокирует небезопасные методы.

```ini
CMS_SECURITY_POLICY_V1=true
```

## Конфигурация для конкретных серверов

Ниже приведены примеры конфигурации для различных веб-серверов.

::: details Apache
Чтобы запустить приложения October CMS, сервер Apache должен быть настроен следующим образом:

* установлен модуль [mod_rewrite](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html);
* директива [AllowOverride](https://httpd.apache.org/docs/2.4/mod/core.html#AllowOverride) для каталога приложения должна иметь значение `All`.

В некоторых случаях может понадобиться раскомментировать директиву [RewriteBase](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html#rewritebase) в файле `.htaccess` проекта:

```text
# RewriteBase /
```

Если October CMS установлена в подкаталог, добавьте его имя в значение директивы. Это позволит использовать URL вида example.tld/subdirectory/page.

```text
# RewriteBase /subdirectory/
```
:::

::: details Nginx
Добавьте следующий код в секцию server файла конфигурации сайта Nginx. Если October CMS установлена в подкаталог, замените первый `/` в директивах location на имя подкаталога.

```text
location / {
    # Пусть October CMS обрабатывает все запросы по умолчанию.
    # Путь, не найденный роутером October CMS, вернёт страницу 404 от October CMS.
    # Всё, что не попало в список разрешённых ниже, будет обработано здесь.
    rewrite ^/.*$ /index.php last;
}

# Передаём PHP-скрипты серверу FastCGI
location ~ ^/index.php {
    # Укажите здесь конфигурацию FPM
}

# Список разрешённых ресурсов
location ~ ^/(favicon\.ico|sitemap\.xml|robots\.txt|humans\.txt) { try_files $uri /index.php; }

# Блокируем все скрытые файлы, кроме well-known
location ~ /\.(?!well-known).* { deny all; }

## Статические файлы
location ~ ^/storage/app/(uploads/public|media|resources) { try_files $uri 404; }
location ~ ^/storage/temp/public { try_files $uri 404; }
location ~ ^/modules/.*/(assets|resources) { try_files $uri 404; }
location ~ ^/modules/.*/(behaviors|widgets|formwidgets|reportwidgets)/.*/(assets|resources) { try_files $uri 404; }
location ~ ^/plugins/.*/.*/(assets|resources) { try_files $uri 404; }
location ~ ^/plugins/.*/.*/(behaviors|reportwidgets|formwidgets|widgets)/.*/(assets|resources) { try_files $uri 404; }
location ~ ^/themes/.*/(?:assets|resources) { try_files $uri 404; }
```
:::

::: details Lighttpd
Вставьте следующий код в файл конфигурации сайтов Lighttpd и измените `host address` и `server.document-root` в соответствии с расположением проекта.

```text
$HTTP["host"] =~ "domain.example.tld" {
    server.document-root = "/var/www/example/"

    url.rewrite-once = (
        "^/(plugins|modules/(system|backend|cms))/(([\w-]+/)+|/|)assets/([\w-]+/)+[-\w^&'@{}[\],$=!#().%+~/ ]+\.(jpg|jpeg|gif|png|svg|swf|avi|mpg|mpeg|mp3|flv|ico|css|js|woff|ttf)(\?.*|)$" => "$0",
        "^/(system|themes/[\w-]+)/assets/([\w-]+/)+[-\w^&'@{}[\],$=!#().%+~/ ]+\.(jpg|jpeg|gif|png|svg|swf|avi|mpg|mpeg|mp3|flv|ico|css|js|woff|ttf)(\?.*|)$" => "$0",
        "^/storage/app/uploads/public/[\w-]+/.*$" => "$0",
        "^/storage/app/media/.*$" => "$0",
        "^/storage/app/resources/.*$" => "$0",
        "^/storage/temp/public/[\w-]+/.*$" => "$0",
        "^/(favicon\.ico)$" => "$0",
        "(.*)" => "/index.php$1"
    )
}
```
:::

::: details Microsoft IIS
Используйте следующую конфигурацию файла `web.config`, чтобы запустить October CMS на IIS:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <clear />
                <rule name="October CMS to handle all non-allowlisted URLs" stopProcessing="true">
                    <match url="^(.*)$" ignoreCase="false" />
                    <conditions logicalGrouping="MatchAll">
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/.well-known/*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/app/uploads/public/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/app/media/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/app/resources/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/temp/public/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/themes/.*/(assets|resources)/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/plugins/.*/(assets|resources)/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/modules/.*/(assets|resources)/.*" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="index.php" appendQueryString="true" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```
:::
