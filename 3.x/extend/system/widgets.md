---
subtitle: Self-contained blocks of functionality that solve different tasks.
---
# Widgets

Widgets are reusable controls that have a user interface and a backend controller (the widget class) that prepares the widget data and handles AJAX requests generated by the widget user interface.

## Generic Widgets

Widgets are the backend equivalent of [CMS Components](../../cms/themes/components.md). They are similar because they are modular bundles of functionality, supply partials and are named using aliases. The key difference is that backend widgets use YAML markup for their configuration and bind themselves to backend controllers manually.

Widget classes reside inside the **widgets** directory of the plugin directory. The inner directory name matches the name of the widget class written in lowercase. Widgets can supply assets and partials. An example widget directory structure looks like this:

::: dir
├── `widgets`
|   ├── form
|   |   ├── partials
|   |   |   └── _form.php  _← Partial File_
|   |   └── assets
|   |       ├── js
|   |       |   └── form.js  _← JavaScript File_
|   |       └── css
|   |           └── form.css  _← StyleSheet File_
|   └── Form.php  _← Widget Class_
:::

### Class Definition

The generic widget classes must extend the `Backend\Classes\WidgetBase` class. As any other plugin class, generic widget controllers should belong to the [plugin namespace](./plugins.md). The following is an example of a widget class definition.

```php
namespace Backend\Widgets;

use Backend\Classes\WidgetBase;

class Lists extends WidgetBase
{
    /**
     * @var string defaultAlias to identify this widget.
     */
    protected $defaultAlias = 'list';

    // ...
}
```

The widget class must contain a **render()** method for producing the widget markup by rendering a widget partial. Example:

```php
public function render()
{
    return $this->makePartial('list');
}
```

To pass variables to partials you can either add them to the `$vars` property.

```php
public function render()
{
    $this->vars['var'] = 'value';

    return $this->makePartial('list');
}
```

Alternatively you may pass the variables to the second parameter of the makePartial() method:

```php
public function render()
{
    return $this->makePartial('list', ['var' => 'value']);
}
```

## Binding Widgets to Controllers

::: aside
Binding to a controller is also required to make [AJAX handlers available](./ajax.md).
:::

A widget should be bound to a backend controller before you can start using it in a backend page or partial. Use the widget's `bindToController` method for binding it to a controller. The best place to initialize a widget is the controller's `beforeDisplay` method, which is called from the constructor.

For example, creating a new widget instance and binding it to the controller. Note the constructor takes the controller as the first argument.

```php
public function beforeDisplay()
{
    $myWidget = new MyWidgetClass($this);
    $myWidget->alias = 'myWidget';
    $myWidget->bindToController();
}
```

After binding the widget you can access it in the controller's view or partial by its alias using the `$this->widget` property.

```php
<?= $this->widget->myWidget->render() ?>
```

## Running Code Before AJAX Handlers

Sometimes you may want code to execute before an AJAX handler executes. Defining an `init` method in the widget allows code to run before every AJAX handler.

```php
function init()
{
    // From a widget class
}
```

#### See Also

::: also
* [Form Widgets](../forms/form-widgets.md)
* [Filter Widgets](../lists/filter-widgets.md)
* [Report Widgets](../backend/report-widgets.md)
:::