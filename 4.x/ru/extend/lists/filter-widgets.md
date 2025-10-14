---
subtitle: Виджет, специально созданный для использования в фильтре.
---
# Виджеты фильтра

С помощью виджетов фильтра можно добавлять новые типы областей в бэкенд‑фильтры. Они предоставляют функции, характерные для фильтрации списков. Виджеты фильтра необходимо регистрировать в [регистрационном файле плагина](../extending.md).

Классы виджетов фильтра располагаются в каталоге **filterwidgets** каталога плагина. Имя вложенного каталога совпадает с именем класса виджета, записанным в нижнем регистре. Виджеты могут подключать ресурсы и partial‑файлы. Пример структуры каталога виджета формы:

::: dir
├── `filterwidgets`
|   ├── discount
|   |   ├── partials
|   |   |   └── _discount.php  _← Partial-файл_
|   |   |   └── _discount_form.php
|   |   └── assets
|   |       ├── js
|   |       |   └── discount.js  _← Файл JavaScript_
|   |       └── css
|   |           └── discount.css  _← Файл стилей_
|   └── Discount.php  _← Класс виджета_
:::

## Определение класса

Команда `create:filterwidget` создаёт виджет фильтра бэкенда вместе с представлением и базовыми файлами ресурсов. Первый аргумент задаёт автора и плагин, второй — имя класса виджета формы.

```bash
php artisan create:filterwidget Acme.Blog Discount
```

Классы виджетов фильтра должны наследовать класс `Backend\\Classes\\FilterWidgetBase`. Зарегистрированный виджет можно использовать в файле [определения полей фильтра](../../element/filter-scopes.md). Пример определения класса виджета формы:

```php
namespace Backend\\FilterWidgets;

use Backend\\Classes\\FilterWidgetBase;

class Discount extends FilterWidgetBase
{
    public function render() {}

    public function renderForm() {}
}
```

## Свойства виджета фильтра

Виджеты фильтра могут иметь свойства, которые задаются через [конфигурацию областей фильтра](../../element/filter-scopes.md). Для этого объявите на классе настраиваемые свойства и вызовите метод `fillFromConfig` внутри метода `init`, чтобы заполнить их значениями.

```php
class Discount extends FormWidgetBase
{
    /**
     * @var bool allowSearch отображает поле поиска в выпадающем списке
     */
    public $allowSearch = false;

    /**
     * init виджета
     */
    public function init()
    {
        $this->fillFromConfig([
            'allowSearch',
        ]);
    }

    // ...
}
```

Значения свойств становятся доступными для указания в [определении области фильтра](../../element/filter-scopes.md) при использовании виджета.

```yaml
discount:
    label: Discount
    type: discount
    allowSearch: true
```

## Регистрация виджета фильтра

Плагины должны регистрировать виджеты фильтра, переопределяя метод `registerFilterWidgets` в [регистрационном файле плагина](../extending.md). Метод возвращает массив, где в ключах указывается класс виджета, а значения содержат короткий код виджета. Пример:

```php
public function registerFilterWidgets()
{
    return [
        \\Backend\\FilterWidgets\\Discount::class => 'discount',
    ];
}
```

Короткий код используется при ссылке на виджет в определениях областей фильтра, он должен быть уникальным, чтобы избежать конфликтов с другими полями фильтра.

## Отображение состояния фильтра

Основная задача виджета фильтра — применить область к запросу модели, то есть сначала получить значения от пользователя. Метод `render` используется для отображения начального состояния фильтра, а свойство `filterScope` содержит активное значение и другие настроенные параметры.

```php
public function render()
{
    $this->vars['scope'] = $this->filterScope;
    $this->vars['name'] = $this->getScopeName();
    $this->vars['value'] = $this->getLoadValue();

    return $this->makePartial('discount');
}
```

На базовом уровне виджет должен показывать пользователю метку и текущее состояние. Содержимое также оборачивается в ссылку, которая используется для отображения формы фильтра.

```php
<a
    href="javascript:;"
    class="filter-scope <?= $value ? 'active' : '' ?>"
    data-scope-name="<?= $name ?>"
>
    <span class="filter-label"><?= e(trans($scope->label)) ?></span>
    <?php if ($value): ?>
        <span class="filter-setting">1</span>
    <?php endif ?>
</a>
```

## Отображение формы фильтра

Когда пользователь нажимает на метку фильтра, отображается форма, в которой можно указать, как применять фильтр. Метод `renderForm` используется для вывода формы фильтра и должен соответствовать partial‑файлу `_discount_form.php`.

```php
public function renderForm()
{
    $this->vars['allowSearch'] = $this->allowSearch;
    $this->vars['scope'] = $this->filterScope;
    $this->vars['name'] = $this->getScopeName();
    $this->vars['value'] = $this->getLoadValue();

    return $this->makePartial('discount_form');
}
```

Содержимое должно включать значения формы и кнопки для применения или очистки фильтра. Тег формы не обязателен, а все поля ввода должны принадлежать массиву `Filter[]`. Чаще всего значение фильтра сохраняется в атрибуте `value`.

```php
<div class="filter-box">
    <div class="filter-facet">
        <div class="facet-item is-grow">
            <select name="Filter[value]" class="form-control form-control-sm custom-select <?= $allowSearch ? '' : 'select-no-search' ?>">
                <option value="1" <?= $scope->value === '1' ? 'selected="selected"' : '' ?>>has a discount</option>
                <option value="0" <?= $scope->value === '0' ? 'selected="selected"' : '' ?>>does not have a discount</option>
            </select>
        </div>
    </div>
    <div class="filter-buttons">
        <button class="btn btn-sm btn-primary" data-filter-action="apply">
            Apply
        </button>
        <div class="flex-grow-1"></div>
        <button class="btn btn-sm btn-secondary" data-filter-action="clear">
            Clear
        </button>
    </div>
</div>
```

::: tip
Переменная `$value` содержит массив выбранных значений. Для удобства этот массив объединяется с переменной `$scope`, поэтому получить активное значение можно через `$scope->value`. Итог: используйте `$value`, чтобы проверить, применяется ли область, и `$scope` для доступа к значениям.
:::

## Получение значения фильтра

Метод `getActiveValue` используется для получения значений формы фильтра и их сохранения. Он должен возвращать массив (или null) и использовать данные postback для поиска значений. Если присутствует значение postback `clearScope`, область необходимо очистить. Можно воспользоваться вспомогательным методом `hasPostValue`, чтобы проверить, найдено ли значение и не пустая ли строка.

```php
public function getActiveValue()
{
    if (post('clearScope')) {
        return null;
    }

    if (!$this->hasPostValue('value')) {
        return null;
    }

    return post('Filter');
}
```

## Применение области к запросу

После получения значения фильтра его можно применить к запросу методом `applyScopeToQuery`. Значение берётся из свойства `filterScope->value`, где имя `value` соответствует полю формы фильтра.

```php
public function applyScopeToQuery($query)
{
    $hasDiscount = $this->filterScope->value;

    if ($hasDiscount) {
        $query->where('discount', '>', 0);
    }
    else {
        $query->where('discount', 0);
    }
}
```

## Работа с встроенными фильтрами

Встроенные (inline) фильтры отображаются прямо в основном интерфейсе фильтра, а не всплывающей формой. Соответственно, в классе виджета фильтра метод `renderForm` не требуется, и для вывода содержимого используется только метод `render`.

В примере ниже показан встроенный фильтр поиска с кнопкой поиска. Важно учитывать, что поскольку фильтр встроенный, имена полей ввода разделяются с основной формой, поэтому поле поиска использует переменную `$name` вместо общего имени `Filter`.

```php
<?php
    $activeValue = $scope->scopeValue !== null ? $scope->value : $scope->default;
?>
<div
    class="filter-scope scope-inline"
    data-scope-name="<?= $scope->scopeName ?>">
    <input
        placeholder="<?= e($this->getHeaderValue($scope)) ?>"
        name="<?= $name ?>[value]"
        value="<?= e($activeValue) ?>"
        class="form-control form-control-sm" />
    <button
        class="btn btn-sm btn-search"
        data-filter-action="apply">
        <i class="icon-search"></i>
    </button>
</div>
```

Следующий пример показывает встроенный контрол Balloon Selector.

```php
<?php
    $activeValue = $scope->scopeValue !== null ? $scope->value : $scope->default;
?>
<div
    data-scope-name="<?= $scope->scopeName ?>"
    data-control="balloon-selector"
    data-selector-allow-empty
    class="filter-scope scope-inline control-balloon-selector form-control-sm">
    <ul class="list-unstyled m-0">
        <?php foreach ((array) $scope->options as $key => $value): ?>
            <li
                data-value="<?= $key ?>"
                class="small <?= $key === $activeValue ? 'active' : '' ?>"
                data-filter-action="apply">
                <?= $value ?>
            </li>
        <?php endforeach ?>
    </ul>
    <!-- Hidden input to store the selected filter value -->
    <input type="hidden" name="<?= $name ?>[value]" value="<?= $activeValue ?>">
</div>
```
