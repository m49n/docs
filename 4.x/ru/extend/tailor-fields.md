---
subtitle: Расширение Tailor пользовательскими контент-полями.
---
# Создание полей Tailor

Можно создавать собственные контент-поля, определив файл с описанием поля и зарегистрировав его в регистрационном файле плагина.

Классы определения контент-полей находятся в каталоге плагина **contentfields**. Внутренний каталог должен совпадать с именем класса виджета, записанным в нижнем регистре. Контент-поля могут подключать ресурсы и частичные представления. Пример структуры каталога:

::: dir
├── `contentfields`
|   ├── mycontentfield
|   |   ├── assets
|   |   └── partials
|   |       └── _column_content.php  _← Файл частичного представления_
|   └── MyContentField.php  _← Класс поля_
:::

### Определение класса

Команда `create:contentfield` генерирует класс контент-поля. Первый аргумент задаёт автора и название плагина. Второй аргумент задаёт имя класса контент-поля.

```bash
php artisan create:contentfield Acme.Blog MyContentField
```

Класс контент-поля должен расширять `Backend\Classes\FormWidgetBase`. Зарегистрированное контент-поле можно использовать в [полях форм Tailor](../element/form-fields.md) и схемах. Класс определяет, как поле взаимодействует с системой. Например, **plugins/acme/blog/contentfields/MyContentField.php** с содержимым ниже.

```php
namespace Acme\Blog\ContentFields;

use Tailor\Classes\ContentFieldBase;
use October\Contracts\Element\FormElement;
use October\Contracts\Element\ListElement;
use October\Contracts\Element\FilterElement;

class MyContentField extends ContentFieldBase
{
    public function defineConfig(array $config) {}

    public function defineFormField(FormElement $form, $context = null) {}

    public function defineListColumn(ListElement $list, $context = null) {}

    public function defineFilterScope(FilterElement $filter, $context = null) {}

    public function extendModelObject($model) {}

    public function extendDatabaseTable($table) {}
}
```

### Регистрация контент-поля

Плагины регистрируют контент-поля, переопределяя метод `registerContentFields` в [регистрационном файле плагина](./extending.md). Метод должен возвращать массив, в котором ключ — класс виджета, а значение — короткий код поля. Пример:

```php
public function registerContentFields()
{
    return [
        \Acme\Blog\ContentFields\MyContentField::class => 'mycontentfield'
    ];
}
```

Короткий код используется при ссылке на поле в [шаблонах схем](introduction.md). Значение должно быть уникальным, чтобы избежать конфликтов с другими полями.

## Обработка конфигурации

Допустим, нужно включить параметр `secondaryTitle`. Сначала объявите одноимённое свойство в классе, затем заполните его в переопределённом методе `defineConfig`.

```php
class MyContentField extends ContentFieldBase
{
    public $secondaryTitle;

    public function defineConfig(array $config)
    {
        if (isset($config['secondaryTitle'])) {
            $this->secondaryTitle = $config['secondaryTitle'];
        }
    }
}
```

Теперь параметр доступен в конфигурации:

```yaml
my_field:
    type: mycontentfield
    secondaryTitle: Custom value goes here
```

## Определение элементов бэкенда

Контент-поле описывает, как оно отображается в бэкенде как поле формы, колонка списка и область фильтра. Результирующий объект в каждом случае — «текучая» конфигурация с цепочкой методов или массивом через `useConfig`. Подробности см. в статье [об определении контент-полей](../cms/tailor/content-fields.md).

### Поле формы

Метод `defineFormField` описывает внешний вид контент-поля в форме. Каждое поле создаётся методом `addFormField`, который принимает имя поля и отображаемую метку.

```php
public function defineFormField(FormElement $form, $context = null)
{
    $form->addFormField($this->fieldName, $this->label)->useConfig($this->config);
}
```

### Колонка списка

Метод `defineListColumn` описывает отображение контент-поля в списке. Каждая колонка создаётся методом `defineColumn`, который принимает имя поля и метку.

```php
public function defineListColumn(ListElement $list, $context = null)
{
    $list->defineColumn($this->fieldName, $this->label)->displayAs('switch');
}
```

### Область фильтра

Метод `defineFilterScope` описывает отображение контент-поля в фильтре. Каждая область создаётся методом `defineScope`, который принимает имя поля и метку.

```php
public function defineFilterScope(FilterElement $filter, $context = null)
{
    $filter->defineScope($this->fieldName, $this->label)->displayAs('switch');
}
```

## Расширение модели

Метод `extendModelObject` позволяет расширить модель записи, например класс `Tailor\Models\EntryRecord`. В качестве примера можно сделать поле jsonable с помощью `addJsonable`.

```php
public function extendModelObject($model)
{
    $model->addJsonable($this->fieldName);
}
```

Другой вариант — объявить связь `belongsTo`.

```php
public function extendModelObject($model)
{
    $model->belongsTo[$this->fieldName] = MyOtherModel::class;
}
```

## Расширение таблицы базы данных

Метод `extendDatabaseTable` описывает, какие колонки базы данных нужны полю. Он использует упрощённую версию [стандартной структуры миграций](../extend/database/structure.md).

```php
public function extendDatabaseTable($table)
{
    $table->mediumText($this->fieldName)->nullable();
}
```

## Полный пример использования

Ниже приведён полный пример создания контент-поля для плагина October Test. Он добавляет тип `mycontentfield`, доступный всем схемам, как показано ниже.

```yaml
fields:
    mycontentfield:
        label: Custom Content Field
        type: mycontentfield
        firstColor: red
        secondColor: blue
```

Поле регистрируется в файле **plugins/october/test/Plugin.php** методом `registerContentFields`.

```php
public function registerContentFields()
{
    return [
        \October\Test\ContentFields\MyContentField::class => 'mycontentfield'
    ];
}
```

Класс поля создаётся в файле **plugins/october/test/contentfields/MyContentField.php**. Он регистрирует себя как [тип поля partial](../element/form/ui-partial.md) и, для простоты, не добавляет колонку списка и область фильтра. Вызов `addJsonable` делает имя поля [jsonable-свойством](../extend/system/models.md), чтобы хранить значение в виде массива. В таблице база данных использует тип `mediumText` [схемы базы данных](../extend/database/structure.md) и модификатор `nullable`, позволяющий оставлять поле пустым.

```php
namespace October\Test\ContentFields;

use Tailor\Classes\ContentFieldBase;
use October\Contracts\Element\FormElement;

class MyContentField extends ContentFieldBase
{
    public function defineFormField(FormElement $form, $context = null)
    {
        $form->addFormField($this->fieldName, $this->label)
            ->useConfig($this->config)
            ->displayAs('partial')
            ->path('$/october/test/contentfields/mycontentfield/partials/_field.php');
    }

    public function extendModelObject($model)
    {
        $model->addJsonable($this->fieldName);
    }

    public function extendDatabaseTable($table)
    {
        $table->mediumText($this->fieldName)->nullable();
    }
}
```

Файл **plugins/october/test/contentfields/mycontentfield/partials/_field.php** содержит частичное представление для отображения поля формы. Значения читаются и сохраняются как массив `[first_value => 'foo', second_value => 'bar']`.

```php
<div class="row">
    <div class="col">
        <input
            type="text"
            name="<?= $field->getName() ?>[first_value]"
            value="<?= e($field->value['first_value'] ?? '') ?>"
            class="form-control"
            style="color:<?= $field->firstColor ?: 'red' ?>"
        />
    </div>
    <div class="col">
        <input
            type="text"
            name="<?= $field->getName() ?>[second_value]"
            value="<?= e($field->value['second_value'] ?? '') ?>"
            class="form-control"
            style="color:<?= $field->secondColor ?: 'blue' ?>"
        />
    </div>
</div>
```

## Form Widgets против контент-полей

Часто возникает вопрос о различиях между виджетом формы и контент-полем и о том, что выбрать в конкретной ситуации. [Виджеты форм](./forms/form-widgets.md) — это поля форм, которые использует виджет `Backend\Widgets\Form`, а также стандартные [типы полей формы](../element/form-fields.md) (text, number, dropdown, partial и т. д.).

Контент-поля — надмножество полей форм, предназначенное исключительно для Tailor. Они расширяют возможности поля, определяя, как оно:

- отображается как [колонка списка](../element/list-columns.md);
- отображается как [область фильтра](../element/filter-scopes.md);
- хранится в [таблице базы данных](./database/structure.md);
- должно применять [правила валидации](./services/validation.md);
- должно [расширять модель](./system/models.md) (jsonable/связь).

Если виджет формы создан без контент-поля, он всё равно доступен в Tailor и разрешается к типу `Tailor\ContentFields\FallbackField`, который базовый и хранит значение в колонке TEXT.

Для полноценного решения лучше определять и виджет формы, и контент-поле. Тогда поле можно [использовать в плагинах](../extend/extending.md) и для контента в [схемах Tailor](../cms/tailor/blueprints.md). Ниже приведены примеры поля валюты, которое доступно везде.

- [Currency Form Widget](https://github.com/responsiv/currency-plugin/blob/master/formwidgets/Currency.php)
- [Currency Content Field](https://github.com/responsiv/currency-plugin/blob/master/contentfields/Currency.php)

Внутри контент-поля заметно, что YAML-описание определяется в PHP. Синтаксис YAML и PHP во многом схож: имя PHP-метода соответствует имени свойства в YAML, а значение передаётся первым аргументом (по умолчанию `true`). Единственное существенное отличие — свойство `type` задаётся методом `displayAs`.

Можно вызывать любые методы, объединяя их в цепочку на PHP-объекте. В таблице ниже представлены примеры преобразования YAML → PHP.

YAML | PHP
---- | ----
`autoFocus: true` | `->autoFocus()`
`label: my field` | `->label('my field')`
`type: partial`   | `->displayAs('partial')`

::: tip
Весь YAML-набор параметров можно передать массивом через `->useConfig([...])`.
:::

#### См. также

::: also
* [Контент-поля Tailor](../cms/tailor/content-fields.md)
:::
