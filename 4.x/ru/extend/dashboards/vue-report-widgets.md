---
subtitle: Виджет дашборда на базе Vue.js
---
# Vue‑виджеты отчётов

## Создание пользовательских виджетов дашборда

Система дашбордов позволяет разработчикам создавать настраиваемые виджеты, предоставляя функциональность, выходящую за рамки возможностей встроенных виджетов. Эти пользовательские виджеты могут иметь любой внешний вид и функциональность, включая загрузку данных и обработку событий через AJAX‑запросы.

### Создание и регистрация пользовательских виджетов

Виджеты дашборда используют Vue‑фреймворк October CMS. Этот фреймворк упрощает разработку как серверной, так и клиентской частей.

В этой документации создаётся простой компонент, который отображает текущее время и содержит кнопку. Нажатие кнопки обрабатывается на сервере через AJAX.

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/widget-example.webp)

Виджеты дашборда следует размещать в каталоге `vuecomponents`, расположенном в корне плагина, например `author/plugin/vuecomponents`. Каждый компонент должен включать PHP‑файл для серверной реализации, JavaScript‑файл с Vue‑компонентом и PHP‑файл с шаблоном Vue. Ниже приведён пример структуры файлов для виджета. Обратите внимание, что имена файлов CSS, JavaScript и частичного представления формируются из имени файла класса виджета:

::: dir
├── `vuecomponents`
|   ├── mycustomwidget
|   |   └── assets
|   |       └── css
|   |           └── mycustomwidget.css  _← Файл таблицы стилей_
|   |       └── js
|   |           └── mycustomwidget.js  _← Файл JavaScript_
|   |   └── partials
|   |       └── _mycustomwidget.php  _← Файл частичного представления_
|   └── MyCustomWidget.php  _← Класс Vue‑виджета_
:::

### Серверный класс

Серверный код виджета дашборда должен определять класс, который расширяет `Backend\Classes\VueReportWidgetBase`. Единственный обязательный метод, который должен реализовать класс виджета, — `getData`. Ниже приведена начальная реализация класса (MyCustomWidget.php):

```php
namespace Acme\MyPlugin\VueComponents;

use Dashboard\Classes\VueReportWidgetBase;
use Dashboard\Classes\ReportFetchData;
use Carbon\Carbon;

class MyCustomWidget extends VueReportWidgetBase
{
    public function getData(ReportFetchData $data): mixed
    {
        return [
            'current_time' => Carbon::now()->format('Y-m-d H:i:s')
        ];
    }
}
```

### Клиентский компонент

Vue‑компонент для виджета должен быть определён в файле, расположенном в каталоге assets/js, как указано выше. Пример кода компонента (assets/js/mycustomwidget.js):

```jsx
Vue.component('plugin-author-component-mycustomwidget', {
    extends: Vue.options.components['dashboard-component-dashboard-widget-base'],
    data: function () {
        return {
        }
    },
    methods: {
        useCustomData: function () {
            return true;
        },

        makeDefaultConfigAndData: function () {
            Vue.set(this.widget.configuration, 'title', 'My Custom Widget');
        },

        getSettingsConfiguration: function () {
            const result = [{
                property: "title",
                title: "Title",
                type: "string",
            }];

            return result;
        }
    },
    template: '#plugin_author_vuecomponents_mycustomwidget'
});
```

При определении компонента убедитесь, что используются корректные пространства имён в вызовах `register` и `component`, а также в идентификаторе шаблона. Они должны соответствовать пространству имён PHP вашего плагина и имени класса плагина.

Vue‑компоненты виджетов должны включать все методы, показанные в примере. В частности:

- `useCustomData` — метод должен возвращать `true`, чтобы сообщить системе дашборда, что компонент управляет собственным циклом данных.
- `makeDefaultConfigAndData` — задаёт конфигурацию виджета по умолчанию. В нашем примере здесь назначается заголовок виджета. Ключи объекта конфигурации произвольные и предназначены для внутреннего использования виджета. Позднее будет показано, как получить доступ к конфигурации виджета в шаблоне компонента. В серверном коде конфигурация виджета доступна в методе `getData` через аргумент `widgetConfig`.
- `getSettingsConfiguration` — возвращает конфигурацию формы настроек виджета. Конфигурация определяется с помощью объектов JavaScript и соответствует [конфигурации полей инспектора](https://docs.octobercms.com/3.x/element/inspector-types.html).

Шаблон Vue‑компонента должен быть определён в файле частичного представления, расположенном в каталоге `partials`, как отмечено выше. Ниже приведена базовая реализация шаблона компонента (partials/_mycustomwidget.php):

```html
<div class="widget-body">
    <h3 class="widget-title" v-text="widget.configuration.title"></h3>

    <div v-if="!loading">
        <p>Current server time: <span v-if="fullWidgetData" v-text="fullWidgetData.data.current_time"></span></p>
    </div>
    <p v-else>Loading...</p>
</div>
```

Шаблон показывает, как получить свойство `title` из конфигурации виджета, ранее заданной в методе `makeDefaultConfigAndData`.

Также показано использование свойства `loading`, которое предоставляет система дашборда. Свойство переключается в `true` во время AJAX‑запросов загрузки данных, которые обрабатывает дашборд. Эти запросы могут запускаться при изменении конфигурации компонента или при выборе диапазона дат на дашборде. Можно использовать свойство `loading`, чтобы отображать состояние загрузки виджета.

Наконец, шаблон демонстрирует доступ к данным, предоставленным сервером, через структуру `fullWidgetData.data`.

### Регистрация виджета

Виджеты дашборда необходимо регистрировать в регистрационном файле плагина (Plugin.php) в методе `registerReportWidgets`. Метод должен возвращать массив, в котором ключами являются классы виджетов, а значениями — конфигурация виджета (метка, группа и требуемые разрешения). Свойство `vue` следует установить в `true`, чтобы указать, что это Vue‑компонент.

```php
public function registerReportWidgets()
{
    return [
        \October\Test\VueComponents\MyCustomWidget::class => [
            'label' => 'Custom Widget',
            'group' => 'Acme Author'
        ]
    ];
}
```

Свойства label и group в возвращаемом массиве `registerReportWidgets` — это имя виджета и имя автора. Они отображаются в интерфейсе дашборда, в частности в меню Create Widget. После регистрации виджета его можно добавить на дашборд:

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/adding-widget.webp)

При добавлении виджета автоматически откроется форма его настроек:

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/config-widget.webp)

### Обработка событий

Виджеты дашборда могут инициировать и обрабатывать события с помощью встроенного AJAX‑фреймворка. В нашем демонстрационном виджете есть кнопка. Мы создадим обработчик, чтобы при нажатии кнопки сервер возвращал случайное число, которое будет показано в теле виджета.

Сначала кнопку нужно определить в шаблоне компонента. Приведённый ниже код добавляет кнопку и элемент для отображения значения, возвращённого сервером:

```php
<button
    class="btn btn-primary"
    :disabled="loadingButtonData"
    @click.stop.prevent="onButtonClick"
>Click me</button>
<span v-text="buttonClickResult"></span>
```

Значения `loadingButtonData` и `buttonClickResult` ещё не определены, их нужно добавить в Vue‑компонент виджета:

```jsx
data: function () {
    return {
        buttonClickResult: null,
        loadingButtonData: false
    }
},
```

Также необходимо создать метод `onButtonClick`. Он использует метод `request` базового Vue‑компонента виджета. Метод устанавливает значение `loadingButtonData` в true, сигнализируя о начале загрузки. После получения ответа от сервера метод присваивает `buttonClickResult` значение свойства `result` из ответа. Подробности создания обработчика на сервере будут приведены далее. Метод включает `some_var` в отправляемые данные; в текущей серверной реализации это значение не используется, но может пригодиться в других сценариях. На сервере значение `some_var` доступно через аргумент `extraData` обработчика.

```jsx
onButtonClick: async function () {
    this.loadingButtonData = true;
    this.buttonClickResult = null;
    try {
        const response = await this.request('onGetSomeData', {
            some_var: "some value"
        });
        this.buttonClickResult = response.result;
    }
    catch (err) {
        $.oc.alert(err.message);
    }
    finally {
        this.loadingButtonData = false;
    }
}
```

Наконец, обработчик на сервере нужно добавить в PHP‑класс виджета. Убедитесь, что имя обработчика совпадает с именем, указанным в клиентском вызове `request`. Кроме того, метод должен быть защищённым и принимать массив конфигурации виджета и массив дополнительных данных.

```jsx
protected function onGetSomeData(array $widgetConfig, array $extraData)
{
    return [
        'result' => rand(1, 100)
    ];
}
```

Ниже показан скриншот виджета, отображающего случайное значение, полученное с сервера:

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/custom-widget-data.webp)
