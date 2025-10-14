---
subtitle: Предоставляет интерфейс для доступа к обобщённым данным.
---
# Источники данных

## Создание источников данных

Любой плагин может зарегистрировать любое количество источников данных. Рекомендуется создавать отдельный источник данных для каждого типа данных, предоставляемых плагином. Например, если плагин предлагает данные о клиентах и продажах, наличие двух отдельных источников данных упростит настройку виджетов для конечных пользователей.

Источники данных — это классы, которые расширяют класс `Dashboard\Classes\ReportDataSourceBase`. Каждый источник данных должен зарегистрировать хотя бы одну метрику и одно измерение. Кроме того, классы источников данных обязаны реализовать метод `fetchData`, который должен возвращать объект `ReportFetchDataResult`. В большинстве случаев, когда источник данных получает данные из базы данных, можно использовать класс `ReportDataQueryBuilder`, который строит и выполняет запросы к базе данных на основе конфигурации измерений и метрик.

```php
use Db;
use Carbon\Carbon;
use Dashboard\Classes\ReportMetric;
use Dashboard\Classes\ReportDimension;
use Dashboard\Classes\ReportDimensionField;
use Dashboard\Classes\ReportDataSourceBase;
use Dashboard\Classes\ReportFetchData;
use Dashboard\Classes\ReportFetchDataResult;
use Dashboard\Classes\ReportDataOrderRule;
use Dashboard\Classes\ReportDataPaginationParams;
use Dashboard\Classes\ReportDataQueryBuilder;

class MyReportDataSource extends ReportDataSourceBase
{
    public function __construct()
    {
        // Здесь регистрируются измерения и метрики
    }

    protected function fetchData(ReportFetchData $data): ReportFetchDataResult
    {
        // Сформируйте и верните объект ReportFetchDataResult
        // или используйте класс ReportDataQueryBuilder, чтобы выполнить основную работу.
    }
}
```

Объект `ReportFetchData` предоставляет следующие свойства:

Property | Type | Description
-------- | ---- | -----------
**$dimension** | `ReportDimension` | измерение, по которому нужно сгруппировать данные
**$metrics** | `array` | метрики, которые нужно вернуть
**$metricsConfiguration** | `array` | конфигурация метрик отчёта
**$dateStart** | `?Carbon` | дата начала
**$dateEnd** | `?Carbon` | дата окончания
**$startTimestamp** | `?int` | метка времени начала
**$dimensionFilters** | `array` | фильтры, применяемые к значениям измерения
**$groupInterval** | `?string` | интервал группировки.
**$orderRule** | `?ReportDataOrderRule` | правило сортировки данных.
**$limit** | `?int` | максимальное количество записей для возврата.
**$paginationParams** | `?ReportDataPaginationParams` | параметры пагинации.
**$hideEmptyDimensionValues** | `bool` | указывает, нужно ли исключать пустые значения измерения из набора данных.
**$totalsOnly** | `bool` | указывает, что метод должен вернуть только итоговые значения метрик, без строк.

Плагины должны регистрировать свои источники данных в регистрационном файле плагина (Plugin.php) в методе `boot`.

```php
use Dashboard\Classes\DashManager;

public function boot()
{
    DashManager::instance()->registerDataSourceClass(
        MyReportDataSource::class,
        'My Custom Data Source' // Можно использовать ссылку на строку локализации
    );
}
```

В примерах документации используется простая структура базы данных плагина для интернет‑магазина. В плагин входят следующие таблицы.

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/data-source-data-structure.png)

Мы создадим источник данных для отображения данных о продажах товаров и категорий в виде таблицы, графика и индикатора.

### Регистрация измерений

Источники данных должны регистрировать метрики и измерения в конструкторе. Используйте метод `registerDimension`, чтобы зарегистрировать измерение. Начнём с регистрации измерения продукта для нашего примера интернет‑магазина.

```php
const DIMENSION_PRODUCT = 'product';

public function __construct()
{
    $this->registerDimension(new ReportDimension(
        self::DIMENSION_PRODUCT,
        'acme_shop_products.id',
        'Product',
        'product_name'
    ));
}
```

Метод `registerDimension` принимает объект `ReportDimension`. Конструктор `ReportDimension` принимает следующие аргументы.

Argument | Type | Description
-------- | ---- | -----------
**$code** | `string` | код ссылки на измерение. Источник данных будет использовать этот код, чтобы различать измерения в вызовах `fetchData`. Это может быть простая строка, например «city».
**$databaseColumnName** | `string` | имя столбца измерения в таблице источника данных. Имя столбца используется `ReportDataQueryBuilder` для построения запросов к базе данных. Часто необходимо указывать имя таблицы (`acme_shop_products` в данном случае) вместе с именем столбца, чтобы избежать неоднозначности, когда запрос источника данных включает несколько таблиц. Если источник данных не работает с базой данных, для этого аргумента можно указать любое значение. В большинстве случаев столбец измерения соответствует первичному ключу в таблице измерения.
**$displayName** | `string` | имя измерения, используемое в отчётах. Например, в виджете Table оно может стать заголовком столбца измерения. Также используется во всплывающем окне настройки виджета в раскрывающемся списке измерений. Значение может быть статической строкой или ссылкой на строку локализации.
**$labelColumnName** | `?string` | имя столбца для подписи измерения. Этот столбец позволяет предоставить удобочитаемую подпись для измерения. Если аргумент не указан, значение `$databaseColumnName` используется как подпись измерения. Имейте в виду, что сортировка и фильтры по измерению в виджете будут использовать значение подписи измерения, если указано имя столбца подписи.

Метод `registerDimension` возвращает зарегистрированный объект измерения, что позволяет использовать цепочки вызовов.

После регистрации источника данных и измерения их можно увидеть в конфигураторе виджета дашборда.

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/dimension-dropdown.png)

### Регистрация метрик

Можно зарегистрировать две метрики, связанные с продуктами: сумму продаж и количество. Эти метрики регистрируются методом `registerMetric` класса источника данных.

```php
const METRIC_TOTAL_AMOUNT = 'total_amount';
const METRIC_TOTAL_QUANTITY = 'total_quantity';

public function __construct()
{
    ...

    $this->registerMetric(new ReportMetric(
        self::METRIC_TOTAL_AMOUNT,
        'acme_shop_sales.total',
        'Total amount',
        ReportMetric::AGGREGATE_SUM
    ));

    $this->registerMetric(new ReportMetric(
        self::METRIC_TOTAL_QUANTITY,
        'acme_shop_sales.quantity',
        'Quantity',
        ReportMetric::AGGREGATE_SUM
    ));
}
```

Метод `registerMetric` принимает объект `ReportMetric`. Конструктор этого класса имеет следующие аргументы:

Argument | Type | Description
-------- | ---- | -----------
**$code** | `string` | код ссылки на метрику.
**$databaseColumnName** | `string` | имя столбца метрики. Всегда желательно указывать имя таблицы вместе с именем поля, чтобы избежать неоднозначности в SQL‑запросах.
**$displayName** | `string` | имя метрики, используемое в отчётах.
**$aggregateFunction** | `string` | агрегирующая функция для метрики. Одно из значений констант `ReportMetric::AGGREGATE_XXX`.
**$intlFormatOptions** | `?array` | параметры форматирования на стороне клиента, совместимые с аргументом options конструктора `Intl.NumberFormat()`. Пропустите аргумент, чтобы использовать параметры форматирования по умолчанию.

Дашборд может агрегировать данные метрик с помощью одной из следующих функций:

```php
ReportMetric::AGGREGATE_SUM
ReportMetric::AGGREGATE_AVG
ReportMetric::AGGREGATE_MIN
ReportMetric::AGGREGATE_MAX
ReportMetric::AGGREGATE_COUNT
ReportMetric::AGGREGATE_NONE
ReportMetric::AGGREGATE_COUNT_DISTINCT
ReportMetric::AGGREGATE_COUNT_DISTINCT_NOT_NULL
```

Для наших задач лучше всего подходит функция `SUM`, поскольку метрики — это общее количество и сумма. Поэтому в конструкторах обеих метрик используется `ReportMetric::AGGREGATE_SUM`.

После регистрации метрик их можно добавить в конфигурацию виджета дашборда:

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/metrics.png)

### Возврат данных из источника данных

Источники данных должны реализовать метод `fetchData`, чтобы возвращать данные для запрошенных измерения, метрик и полей измерений. В этом методе довольно много аргументов, но в большинстве случаев, если источник данных работает с базой данных, их можно напрямую передать классу `ReportDataQueryBuilder`.

Использование класса `ReportDataQueryBuilder` необязательно. Единственное требование для метода `fetchData` — вернуть объект `Dashboard\Classes\ReportFetchDataResult`. Неважно, каким образом этого добиться. Если источник данных не работает с базой данных или `ReportDataQueryBuilder` недостаточно гибок для ваших задач, можно использовать стандартные классы Laravel для загрузки данных.

Ниже приведена частичная реализация нашего демо‑источника данных интернет‑магазина:

```php
protected function fetchData(ReportFetchData $data): ReportFetchDataResult
{
    if ($dimension->getCode() !== self::DIMENSION_PRODUCT) {
        throw new SystemException('Invalid dimension');
    }

    $reportQueryBuilder = new ReportDataQueryBuilder(
        'acme_shop_products',
        $data->dimension,
        $data->metrics,
        $data->orderRule,
        $data->dimensionFilters,
        $data->limit,
        $data->paginationParams,
        $data->groupInterval,
        $data->hideEmptyDimensionValues,
        $data->dateStart,
        $data->dateEnd,
        $data->startTimestamp,
        'acme_shop_sales.sale_date',
        null,
        $data->totalsOnly
    );

    // ...
}
```

Конструктор класса `ReportDataQueryBuilder` принимает большинство аргументов `fetchData` с несколькими дополнениями:

- Первый аргумент — имя основной таблицы, используемой для выборки данных. Обычно это таблица, связанная с запрошенным измерением. В нашем случае, так как измерение — идентификатор продукта, основной таблицей запроса будет `acme_shop_products`.
- Конструктор также принимает имя столбца даты, который используется для ограничения возвращаемых данных интервалом, указанным пользователем. В нашем случае мы используем столбец даты продажи `acme_shop_sales.sale_date`.
- Дополнительно конструктор принимает имя столбца с меткой времени. Хотя минимальное разрешение времени для дашборда по умолчанию — один день, некоторые виджеты могут получать данные за прошлый час, что обеспечивает столбец метки времени. Для простоты мы не реализуем эту функцию и передаём в аргументе значение `null`.

Поскольку наши метрики принадлежат таблице, отличной от таблицы измерений, необходимо настроить объект построителя запроса отчёта на загрузку данных метрик из таблицы `acme_shop_sales`. Для этого используется метод `onConfigureMetrics`:

```php
$reportQueryBuilder->onConfigureMetrics(
    function(Builder $query, ReportDimension $dimension, array $metrics) {
        $query->leftJoin('acme_shop_sales', function($join) {
            $join->on('acme_shop_sales.product_id', '=', 'acme_shop_products.id');
        });
    }
);
```

Метод принимает callback‑функцию, которая должна получать объект `Illuminate\Database\Query\Builder`, объект измерения и массив метрик в качестве аргументов. Это обеспечивает высокую настраиваемость реализации. В нашем простом случае мы используем объект построителя запросов Laravel, чтобы выполнить соединение с таблицей продаж.

И наконец, после настройки построителя запросов отчёта можно выполнить запросы и вернуть загруженные данные:

```php
return $reportQueryBuilder->getFetchDataResult($metricsConfiguration);
```

Ниже приведена полная реализация метода `fetchData`:

```php
protected function fetchData(ReportFetchData $data): ReportFetchDataResult
{
    if ($data->dimension->getCode() !== self::DIMENSION_PRODUCT) {
        throw new SystemException('Invalid dimension');
    }

    $reportQueryBuilder = new ReportDataQueryBuilder(
        'acme_shop_products',
        $data->dimension,
        $data->metrics,
        $data->orderRule,
        $data->dimensionFilters,
        $data->limit,
        $data->paginationParams,
        $data->groupInterval,
        $data->hideEmptyDimensionValues,
        $data->dateStart,
        $data->dateEnd,
        $data->startTimestamp,
        'acme_shop_sales.sale_date',
        null,
        $data->totalsOnly
    );

    $reportQueryBuilder->onConfigureMetrics(
        function(Builder $query, ReportDimension $dimension, array $metrics) {
            $query->leftJoin('acme_shop_sales', function($join) {
                $join->on('acme_shop_sales.product_id', '=', 'acme_shop_products.id');
            });
        }
    );

    return $reportQueryBuilder->getFetchDataResult($data->metricsConfiguration);
}
```

Такой конфигурации достаточно, чтобы отобразить данные источника данных в виджете Table:

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/table-all-data.png)

### Форматирование данных метрик

По умолчанию дашборд форматирует данные метрик как числа. Если форматирование по умолчанию не подходит, можно настроить метрики для отображения данных в других форматах. В нашем примере интернет‑магазина логично отображать общую сумму в валюте.

Последний параметр конструктора класса `ReportMetric` принимает массив опций, совместимых с аргументом `options` [Intl.NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat/NumberFormat), API браузера. Пример ниже демонстрирует конфигурацию, идеально подходящую для отображения валюты в формате США:

```php
$this->registerMetric(new ReportMetric(
    self::METRIC_TOTAL_AMOUNT,
    'acme_shop_sales.total',
    'Total amount',
    ReportMetric::AGGREGATE_SUM,
    [
        'style' => 'currency',
        'currency' => 'USD',
    ]
));
```

Обновлённые данные метрики отображаются на дашборде следующим образом:

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/currency.png)

### Отображение дополнительных данных измерений

В нашей демо‑структуре базы данных в таблице товаров есть столбец `brand`. Кроме того, он ссылается на таблицу категорий, что обеспечивает привязку каждого товара к категории. Мы можем показывать в отчётах названия бренда и категории вместе с названием продукта, используя функциональность полей измерений.

Поля измерений следует добавлять к измерению в конструкторе источника данных. Объект измерения включает метод `addDimensionField`, принимающий настроенный объект поля измерения. Вот как можно добавить поля измерения для названий бренда и категории к измерению продукта:

```php
$this->registerDimension(new ReportDimension(
    self::DIMENSION_PRODUCT,
    'acme_shop_products.id',
    'Product',
    'product_name'
))->addDimensionField(new ReportDimensionField(
    'oc_field_brand',
    'Brand',
    'brand',
    true,
    true
))->addDimensionField(new ReportDimensionField(
    'oc_field_category',
    'Category',
    'acme_shop_categories.category_name',
    true,
    true
));
```

Конструктор `ReportDimensionField` принимает следующие аргументы:

Argument | Type | Description
-------- | ---- | -----------
**$code** | `string` | код ссылки на поле. Код должен начинаться с префикса `oc_field_`.
**$displayName** | `string` | имя поля, используемое в отчётах. Например, в виджете Table оно может стать заголовком столбца измерения. Также используется во всплывающем окне настройки виджета в раскрывающемся списке измерений. Значение может быть статической строкой или ссылкой на строку локализации.
**$columnName** | `?string` | необязательное имя столбца базы данных для фильтрации или сортировки. Укажите имя столбца, чтобы включить сортировку и фильтрацию. В большинстве случаев следует задавать значение для этого аргумента.
**$sortable** | `bool` | указывает, можно ли сортировать поле.
**$filterable** | `bool` | указывает, можно ли фильтровать поле.

После регистрации поле измерения появится в конфигураторах виджетов дашборда:

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/dimension-fields.png)

Остаётся вернуть данные для полей измерений. Поскольку класс `ReportDataQueryBuilder` не автоматизирует этот процесс, необходимо настроить его базовый запрос, добавив соединение и соответствующие столбцы в выборку. Это можно сделать в callback‑функции `onConfigureQuery`:

```php
$reportQueryBuilder->onConfigureQuery(
    function(Builder $query, ReportDimension $dimension, array $metrics) {
        $query->leftJoin('acme_shop_categories', function($join) {
            $join->on('acme_shop_categories.id', '=', 'acme_shop_products.category_id');
        });

        $query->addSelect([
            Db::raw('max(acme_shop_products.brand) as oc_field_brand'),
            Db::raw('max(acme_shop_categories.category_name) as oc_field_category'),
        ]);
    }
);
```

Может показаться странным, что для бренда и категории продукта используется функция `max`. Это необходимо, потому что запрос группируется по столбцу идентификатора продукта (измерение), и для всех столбцов нужно применять агрегирующие функции, чтобы избежать ошибок MySQL. Этот простой приём решает проблему.

Теперь поля измерений доступны для предварительного просмотра на дашборде. Пользователи могут настраивать виджеты для сортировки данных по столбцу бренда или категории:

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/dimension-fields-table.png)

### Использование источника данных

Правильно настроенный источник данных сразу готов к использованию во всех типах виджетов дашборда без дополнительной настройки. Например, можно настроить виджет Chart для отображения топовых продуктов.

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/chart-widget.png)

В этом примере виджет диаграммы настроен следующим образом:

- Тип диаграммы: Bar
- Направление: Horizontal
- Источник данных: наш тестовый источник данных
- Измерение: Product
- Метрики: Total amount
- Лимит: 5
- Сортировка по: Total amount
- Порядок сортировки: Descending
- Отображение: интервал дашборда

Также можно настроить виджет Indicator для отображения итоговых значений с возможностью фильтрации данных по конкретному бренду или категории:

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/indicator-widgets.png)

В этом примере виджет индикатора Smartphones настроен так:

- Источник данных: наш тестовый источник данных
- Измерение: Product
- Значение: Total amount
- Атрибут фильтра: Category
- Операция фильтра: Equals
- Значение: Smartphones

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/indicator-smartphones.png)

Виджет индикатора iPhones использует следующую конфигурацию фильтра:

- Атрибут: Product
- Операция: Includes
- Значение: iPhone

![image](https://raw.githubusercontent.com/octobercms/docs/develop/images/dashboards/indicator-iphones.webp)
