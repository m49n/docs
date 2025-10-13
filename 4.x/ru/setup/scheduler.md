---
subtitle: Узнайте, как запускать планировщик задач и очереди.
---
# Настройка планировщика

Чтобы запланированные задачи выполнялись корректно, добавьте на сервере следующую [cron-задачу](https://www.cyberciti.biz/faq/how-do-i-add-jobs-to-cron-under-linux-or-unix-oses/):

```bash
* * * * * php /october/artisan schedule:run >> /dev/null 2>&1
```

Не забудьте заменить `/october/artisan` на абсолютный путь к файлу artisan в каталоге установки October CMS. Это задание будет запускать планировщик каждую минуту. Затем October CMS проверяет все задачи и выполняет те, у которых наступило время.

::: tip
Если вы добавляете cron-файл в /etc/cron.d, укажите имя пользователя после `* * * * *`:

```bash
* * * * * alice php /october/artisan schedule:run >> /dev/null 2>&1
```
:::

## Настройка обработчиков очередей

При необходимости можно настроить драйвер очередей для обработки [фоновых задач](../../extend/services/queue.md). Драйвер задаётся в файле `config/queue.php`.

Для драйвера database можно создать cron-задачу, выполняющую команду `php artisan queue:work --once`, которая запускает первую доступную задачу в очереди.

```bash
* * * * * php /october/artisan queue:work --once >> /dev/null 2>&1
```

Также можно запустить обработчик очередей как демон-процесс:

```bash
php artisan queue:work
```

## Cron без доступа к командной строке

Если ваш хостинг не предоставляет доступ к таблице cron, можно вместо этого вызывать публичный URL каждые N минут. Например, плагину может требоваться запуск следующей команды каждые 15 минут:

```bash
php artisan campaign:run
```

Для этого понадобится небольшое PHP-решение: используйте фасад `Artisan` и [файл маршрутов](../../extend/system/routing.md), чтобы создать конечную точку, вызывающую команду. Например, файл **routes.php** может выглядеть так:

```php
Route::get('/campaign-run', function () {
    return Artisan::call('campaign:run');
});
```

При обращении к URL `/campaign-run` будет выполняться команда artisan.

#### См. также

::: also
* [Планирование задач](../../extend/system/scheduling.md)
* [Очереди](../../extend/services/queue.md)
:::
