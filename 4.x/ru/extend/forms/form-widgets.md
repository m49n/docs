---
subtitle: Виджет, специально созданный для использования как поле формы.
---
# Виджеты форм

С помощью виджетов форм можно добавлять новые типы элементов управления в формы бэкенда. Они предоставляют возможности, характерные для ввода данных в модели. Виджеты форм нужно зарегистрировать в [файле регистрации плагина](../extending.md).

Классы виджетов форм располагаются в каталоге **formwidgets** плагина. Имя вложенного каталога совпадает с именем класса виджета в нижнем регистре. Виджеты могут подключать ресурсы и частичные представления. Пример структуры каталога виджета формы выглядит так.

::: dir
├── `formwidgets`
|   ├── colorpicker
|   |   ├── partials
|   |   |   └── _colorpicker.php  _← Файл частичного представления_
|   |   └── assets
|   |       ├── js
|   |       |   └── colorpicker.js  _← Файл JavaScript_
|   |       └── css
|   |           └── colorpicker.css  _← Файл таблицы стилей_
|   └── ColorPicker.php  _← Класс виджета_
:::

### Определение класса

Команда `create:formwidget` генерирует виджет формы бэкенда, представление и базовые файлы ресурсов. Первый аргумент задаёт автора и имя плагина. Второй аргумент — имя класса виджета формы.

```bash
php artisan create:formwidget Acme.Blog ColorPicker
```

Классы виджетов формы должны наследовать класс `Backend\\Classes\\FormWidgetBase`. Зарегистрированный виджет можно использовать в [определении поля формы](../../element/form-fields.md). Ниже приведён пример определения класса виджета формы.

```php
namespace Backend\\FormWidgets;

use Backend\\Classes\\FormWidgetBase;

class ColorPicker extends FormWidgetBase
{
    /**
     * @var string defaultAlias для идентификации этого виджета.
     */
    protected $defaultAlias = 'colorpicker';

    public function render() {}
}
```

### Свойства виджета формы

Виджеты форм могут иметь свойства, которые задаются через [конфигурацию поля формы](../../element/form-fields.md). Достаточно определить на классе свойства для настройки и вызвать метод `fillFromConfig` внутри `init`, чтобы заполнить их значениями.

```php
class DatePicker extends FormWidgetBase
{
    //
    // Настраиваемые свойства
    //

    /**
     * @var string режим отображения: datetime, date, time.
     */
    public $mode = 'datetime';

    /**
     * @var string minDate — минимальная дата, которую можно выбрать.
     * например: 2000-01-01
     */
    public $minDate = null;

    /**
     * @var string maxDate — максимальная дата, которую можно выбрать.
     * например: 2020-12-31
     */
    public $maxDate = null;

    //
    // Свойства объекта
    //

    /**
     * {@inheritDoc}
     */
    protected $defaultAlias = 'datepicker';

    /**
     * {@inheritDoc}
     */
    public function init()
    {
        $this->fillFromConfig([
            'mode',
            'minDate',
            'maxDate',
        ]);
    }

    // ...
}
```

После этого значения свойств можно задавать в [определении поля формы](../../element/form-fields.md) при использовании виджета.

```yaml
born_at:
    label: Date of Birth
    type: datepicker
    mode: date
    minDate: 1984-04-12
    maxDate: 2014-04-23
```

### Регистрация виджета формы

Плагины должны регистрировать виджеты форм, переопределяя метод `registerFormWidgets` в [файле регистрации плагина](../extending.md). Метод возвращает массив, где в ключах находятся классы виджетов, а в значениях — их короткие коды. Пример:

```php
public function registerFormWidgets()
{
    return [
        \\Backend\\FormWidgets\\ColorPicker::class => 'colorpicker',
        \\Backend\\FormWidgets\\DatePicker::class => 'datepicker'
    ];
}
```

Короткий код необязателен. Его можно использовать при ссылке на виджет в [определениях полей формы](./form-controller.md); значение должно быть уникальным, чтобы избежать конфликтов с другими полями формы.

### Загрузка данных формы

Основная задача виджета формы — взаимодействовать с моделью, то есть в большинстве случаев загружать и сохранять значения в базе данных. При рендеринге виджет формы запрашивает сохранённое значение через метод `getLoadValue`. Методы `getId` и `getFieldName` возвращают уникальные идентификатор и имя HTML‑элемента формы. Эти значения часто передаются в частичное представление виджета при рендеринге.

```php
public function render()
{
    $this->vars['id'] = $this->getId();
    $this->vars['name'] = $this->getFieldName();
    $this->vars['value'] = $this->getLoadValue();

    return $this->makePartial('myformwidget');
}
```

В простейшем случае виджет формы может вернуть введённое пользователем значение через элемент ввода. В приведённом выше примере элемент можно вывести в частичном представлении **myformwidget**, используя подготовленные переменные.

```php
<input id="<?= $id ?>" name="<?= $name ?>" value="<?= e($value) ?>" />
```

### Сохранение данных формы

Когда нужно принять данные от пользователя и сохранить их в базе, виджет формы вызывает метод `getSaveValue`, чтобы получить значение. Чтобы изменить поведение, переопределите этот метод в своём классе виджета.

```php
public function getSaveValue($value)
{
    return $value;
}
```

Иногда намеренно не требуется возвращать какое-либо значение, например, если виджет отображает информацию и ничего не сохраняет. Верните специальную константу `FormField::NO_SAVE_DATA` из класса `Backend\\Classes\\FormField`, чтобы проигнорировать значение.

```php
public function getSaveValue($value)
{
    return \\Backend\\Classes\\FormField::NO_SAVE_DATA;
}
```
