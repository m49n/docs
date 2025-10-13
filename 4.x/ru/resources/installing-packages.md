---
subtitle: Узнайте, как устанавливать и управлять плагинами и темами.
---
# Установка плагинов и тем

## Управление проектом

October CMS содержит команды для управления вашим проектом.

### Синхронизация проекта

`project:sync` устанавливает все плагины и темы, прикреплённые к проекту.

```bash
php artisan project:sync
```

<a id="oc-set-project"></a>
### Назначение проекта

`project:set` задаёт лицензионный ключ для текущей установки.

```bash
php artisan project:set <лицензионный_ключ>
```

## Управление плагинами

October CMS включает набор команд для управления плагинами.

### Установка плагина

`plugin:install` — скачивает и устанавливает плагин по его имени. Следующий пример установит плагин **AuthorName.PluginName**.

```bash
php artisan plugin:install AuthorName.PluginName
```

Используйте параметр `--want`, чтобы установить конкретную версию плагина.

```bash
php artisan plugin:install AuthorName.PluginName --want=1.0
```

Вы можете установить плагин из удалённого источника с помощью параметра `--from`.

```bash
php artisan plugin:install AuthorName.PluginName --from=git@github.com:authorname/pluginname-plugin.git
```

Используйте параметр `--want`, чтобы указать целевую ветку или версию.

```bash
php artisan plugin:install AuthorName.PluginName --from=git@github.com:authorname/pluginname-plugin.git --want=dev-develop
```

Добавьте параметр `--oc`, если имя вашего пакета имеет префикс `oc`.

```bash
php artisan plugin:install AuthorName.PluginName --from=git@github.com:authorname/pluginname-plugin.git --oc
```

### Проверка зависимостей

`plugin:check` — выполняет проверку зависимостей установленных плагинов во всей системе. Команда просматривает каждую установленную тему и плагин, проверяя, установлены ли их зависимости. Если она обнаружит отсутствующие требования, то попытается установить их.

```bash
php artisan plugin:check
```

### Обновление плагина

`plugin:refresh` — удаляет таблицы базы данных плагина и создаёт их заново. Эта команда полезна во время разработки.

```bash
php artisan plugin:refresh AuthorName.PluginName
```

Используйте параметр `--rollback`, чтобы только удалить таблицы базы данных без их повторного создания.

```bash
php artisan plugin:refresh AuthorName.PluginName --rollback
```

Вы также можете указать номер версии вместе с параметром `--rollback`, чтобы остановиться на определённой версии.

```bash
php artisan plugin:refresh AuthorName.PluginName --rollback=1.0.3
```

### Список плагинов

`plugin:list` — выводит список установленных плагинов и их версий.

```bash
php artisan plugin:list
```

### Отключение плагина

`plugin:disable` — отключает существующий плагин.

```bash
php artisan plugin:disable AuthorName.PluginName
```

### Включение плагина

`plugin:enable` — включает отключённый плагин.

```bash
php artisan plugin:enable AuthorName.PluginName
```

### Удаление плагина

`plugin:remove` — удаляет таблицы базы данных плагина и стирает файлы плагина из файловой системы.

```bash
php artisan plugin:remove AuthorName.PluginName
```

## Управление темами

October предоставляет набор команд для управления темами.

### Установка темы

`theme:install` — скачивает и устанавливает тему из [Marketplace](https://octobercms.com/themes/). Пример ниже установит тему в каталог `/themes/authorname-themename`.

```bash
php artisan theme:install AuthorName.ThemeName
```

Вы можете установить тему из удалённого источника с помощью параметра `--from`.

```bash
php artisan theme:install AuthorName.ThemeName --from=git@github.com:authorname/themename-theme.git
```

Используйте параметр `--want`, чтобы указать целевую ветку или версию.

```bash
php artisan theme:install AuthorName.ThemeName --from=git@github.com:authorname/themename-theme.git --want=dev-develop
```

Добавьте параметр `--oc`, если имя вашего пакета имеет префикс `oc`.

```bash
php artisan theme:install AuthorName.ThemeName --from=git@github.com:authorname/oc-themename-theme.git --oc
```

### Проверка защиты

`theme:check` — выполняет проверку тем по всей системе, чтобы определить, следует ли пометить их как доступные только для чтения и защитить от изменений. Команда просматривает каждую тему и проверяет, была ли она установлена через Composer; если да, создаётся [файл блокировки темы](../cms/themes/child-themes.md) и создаётся дочерняя тема.

```bash
php artisan theme:check
```

### Список тем

`theme:list` — выводит список установленных тем.

```bash
php artisan theme:list
```

### Активация темы

`theme:use` — переключает активную тему. Следующий пример активирует тему в `/themes/rainlab-vanilla`.

```bash
php artisan theme:use rainlab-vanilla
```

### Удаление темы

`theme:remove` — удаляет тему. Пример ниже удалит каталог `/themes/rainlab-vanilla`.

```bash
php artisan theme:remove rainlab-vanilla
```

### Копирование темы

`theme:copy` — дублирует существующую тему для создания новой, включая поддержку дочерних тем.

```bash
php artisan theme:copy <исходная_тема> [целевая_тема]
```
