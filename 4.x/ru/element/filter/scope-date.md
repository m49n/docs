---
subtitle: Область фильтра
shortname: Date
---
# Область Date

`date` — фильтр по дате с логикой условий `equals`, `between`, `before` и `after`.

```yaml
created_at:
    label: Created
    type: date
```

Поддерживаются следующие свойства фильтра.

Property | Description
------------- | -------------
**minDate** | минимальная (самая ранняя) дата, доступная для выбора.
**maxDate** | максимальная (самая поздняя) дата, доступная для выбора.
**firstDay** | первый день недели. Значение по умолчанию: `0` (воскресенье).
**showWeekNumber** | отображать номера недель в начале строки. Значение по умолчанию: `false`.
**useTimezone** | преобразовывать дату и время в указанный в бэкенде часовой пояс. Значение по умолчанию: `true`.
**conditions** | для каждого условия можно указать `true` или `false`, чтобы сделать его доступным, либо передать строку с произвольным SQL‑выражением для выбранных условий. Значение по умолчанию: `true`.

Для фильтра доступны следующие `conditions`.

Condition | Description
------------- | -------------
**equals** | попадает в выбранную дату с начала до конца дня
**notEquals** | не попадает в выбранную дату с начала до конца дня
**between** | находится между двумя выбранными датами
**before** | раньше выбранной даты
**after** | позже выбранной даты

Отфильтрованное значение автоматически приводится к часовому поясу, указанному в настройках бэкенда. Это можно отключить с помощью параметра `useTimezone`.

```yaml
created_at:
    label: Created
    type: date
    useTimezone: false
```

Чтобы разрешить поиск только по точной дате, передайте **equals** в параметр `conditions`. Чтобы находить результаты, попадающие в любой участок диапазона, передайте в `conditions` значения **between**, **before** или **after**.

```yaml
created_at:
    label: Created
    type: date
    conditions:
        equals: true
```

Можно указать значение `default`, заключив его в кавычки, чтобы оно воспринималось как строка. Значение **now** задаёт текущую дату.

```yaml
created_at:
    label: Created
    type: date
    default: '2020-01-02'
```

Минимальную и максимальную доступные даты задают свойства `minDate` и `maxDate`.

```yaml
created_at:
    label: Date
    type: date
    minDate: '2001-01-23'
    maxDate: '2030-10-13'
```

Для условий можно передать произвольный SQL‑код в виде строки с поддержкой подстановок.

```yaml
created_at:
    label: Created
    type: date
    conditions:
        before: created_at <= :value
        between: created_at >= :after AND created_at <= :before
```

Поддерживаются следующие параметры.

- `:value`: выбранная дата в формате `Y-m-d 00:00:00`
- `:valueDate`: выбранная дата в формате `Y-m-d`
- `:before`: конечная дата в формате `Y-m-d 00:00:00`
- `:beforeDate`: конечная дата в формате `Y-m-d`
- `:after`: начальная дата в формате `Y-m-d 00:00:00`
- `:afterDate`: начальная дата в формате `Y-m-d`

## PHP-интерфейс

Для доступа из PHP можно определить пользовательский `modelScope` в модели, как в примере ниже.

```yaml
created_at:
    label: Created
    type: date
    modelScope: dateFilter
```

Метод **scopeDateFilter** получает значения в `$scope->value`, `$scope->before` и `$scope->after`.

```php
function scopeDateFilter($query, $scope)
{
    if ($scope->condition === 'equals') {
        $query->where('created_at', $scope->value);
    }
    elseif ($scope->condition === 'notEquals') {
        $query->where('created_at', '<>', $scope->value);
    }
    elseif ($scope->condition === 'between') {
        $query
            ->where('created_at', '>=', $scope->after)
            ->where('created_at', '<=', $scope->before);
    }
    elseif ($scope->condition === 'after') {
        $query->where('created_at', '>=', $scope->value);
    }
    else {
        $query->where('created_at', '<=', $scope->value);
    }
}
```
