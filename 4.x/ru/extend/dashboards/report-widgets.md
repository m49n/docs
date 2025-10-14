---
subtitle: Виджет, специально созданный для использования в дашборде.
---
# Виджеты отчётов

Виджеты отчётов можно использовать на дашборде бэкенда и в других контейнерах отчётов бэкенда. Виджеты отчётов необходимо регистрировать в [регистрационном файле плагина](../extending.md).

Классы виджетов отчётов располагаются в каталоге плагина **reportwidgets**. Как и любой другой класс плагина, универсальные контроллеры виджетов должны принадлежать пространству имён плагина. Как и все виджеты бэкенда, виджеты отчётов используют частичные представления и специальную структуру каталогов. Пример структуры каталогов:

::: dir
├── `reportwidgets`
|   ├── trafficsources
|   |   └── partials
|   |       └── _widget.php  _← Файл частичного представления_
|   └── TrafficSources.php  _← Класс виджета_
:::

## Определение класса

Команда `create:reportwidget` генерирует виджет отчёта бэкенда, представление и базовые файлы ресурсов. Первый аргумент задаёт автора и имя плагина. Второй аргумент задаёт имя класса виджета отчёта.

```bash
php artisan create:reportwidget Acme.Blog TopPosts
```

Классы виджетов отчётов должны расширять класс `Dashboard\Classes\ReportWidgetBase`. Ниже приведён пример определения класса виджета отчёта. Класс должен переопределять метод `render`, чтобы отрисовать сам виджет.

```php
namespace RainLab\GoogleAnalytics\ReportWidgets;

use Dashboard\Classes\ReportWidgetBase;

class TrafficSources extends ReportWidgetBase
{
    public function render()
    {
        return $this->makePartial('widget');
    }
}
```

Частичное представление виджета может содержать любую HTML‑разметку, которую нужно отобразить в виджете. Разметку следует обернуть в элемент DIV с классом **report-widget**. Для вывода заголовка виджета предпочтительно использовать элемент H3. Пример частичного представления виджета:

```html
<div class="report-widget">
    <h3>Traffic sources</h3>

    <div
        class="control-chart"
        data-control="chart-pie"
        data-size="200"
        data-center-text="180">
        <ul>
            <li>Direct <span>1000</span></li>
            <li>Social networks <span>800</span></li>
        </ul>
    </div>
</div>
```

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/traffic-sources.png)

Внутри виджетов отчётов можно использовать любые [диаграммы или индикаторы](https://octobercms.com/docs/ui/chart), списки или другую необходимую разметку. Помните, что виджеты отчётов расширяют универсальные виджеты бэкенда, поэтому в своих виджетах отчётов можно использовать любую функциональность виджетов. Следующий пример показывает разметку виджета отчёта со списком.

```html
<div class="report-widget">
    <h3>Top pages</h3>

    <div class="table-container">
        <table class="table data" data-provides="rowlink">
            <thead>
                <tr>
                    <th><span>Page URL</span></th>
                    <th><span>Pageviews</span></th>
                    <th><span>% Pageviews</span></th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>/</td>
                    <td>90</td>
                    <td>
                        <div class="progress">
                            <div class="bar" style="90%"></div>
                            <a href="/">90%</a>
                        </div>
                    </td>
                </tr>
                <tr>
                    <td>/docs</td>
                    <td>10</td>
                    <td>
                        <div class="progress">
                            <div class="bar" style="10%"></div>
                            <a href="/docs">10%</a>
                        </div>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</div>
```

## Свойства виджета отчёта

Виджеты отчётов могут иметь свойства, которые пользователи настраивают через инспектор (Inspector):

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/report-widget-inspector.png)

Свойства следует определять в методе `defineProperties` класса виджета. Свойства описаны в разделе [типы инспектора](../../element/inspector-types.md).

```php
public function defineProperties()
{
    return [
        'title' => [
            'title' => 'Widget title',
            'default' => 'Top Pages',
            'type' => 'string',
            'validation' => [
                'required' => [
                    'message' => 'The Widget Title is required.'
                ],
            ]
        ],
        'days' => [
            'title' => 'Number of days to display data for',
            'default' => '7',
            'type' => 'string',
            'validation' => [
                'regex' => [
                    'message' => 'The days property can contain only numeric symbols.',
                    'pattern' => '^[0-9]+$'
                ]
            ]
        ]
    ];
}
```

## Регистрация виджета отчёта

Плагины могут регистрировать виджеты отчётов, переопределяя метод `registerReportWidgets` в [регистрационном файле плагина](../extending.md). Метод должен возвращать массив, в котором ключами являются классы виджетов, а значениями — конфигурация виджета (метка, группа и требуемые разрешения).

```php
public function registerReportWidgets()
{
    return [
        \RainLab\GoogleAnalytics\ReportWidgets\TrafficOverview::class => [
            'label' => 'Google Analytics traffic overview',
            'group' => 'Widgets',
            'permissions' => [
                'rainlab.googleanalytics.widgets.traffic_overview',
            ],
        ],
        \RainLab\GoogleAnalytics\ReportWidgets\TrafficSources::class => [
            'label' => 'Google Analytics traffic sources',
            'group' => 'Widgets',
            'permissions' => [
                'rainlab.googleanaltyics.widgets.traffic_sources',
            ],
        ]
    ];
}
```

Элемент **label** задаёт имя виджета, отображаемое во всплывающем окне добавления виджета. Элемент **group** определяет контекст пункта меню, в котором можно выбрать виджет.
