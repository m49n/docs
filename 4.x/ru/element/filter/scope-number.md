---
subtitle: Область фильтра
shortname: Number
---
# Область Number

`number` — фильтр по числовому значению с логикой условий `exact`, `between`, `greater` и `lesser`.

```yaml
age:
    label: Age
    type: number
    conditions:
        greater: true
```

Поддерживаются следующие свойства фильтра.

Property | Description
------------- | -------------
**default** | задаёт значение фильтра по умолчанию.
**conditions** | для каждого условия можно указать `true` или `false`, чтобы сделать его доступным, либо передать строку с произвольным SQL‑выражением для выбранных условий. Значение по умолчанию: `true`.
**modelScope** | применяет [область запроса модели](../../extend/database/model.md) к запросу фильтра; можно указать метод модели или статический метод PHP‑класса (`Class::method`). Первым аргументом будет модель, к которой виджет привязывает значение, то есть родительская модель.

Для фильтра доступны следующие `conditions`.

Condition | Description
------------- | -------------
**exact** | совпадает с точным числом
**between** | находится между двумя заданными числами
**greater** | больше заданного числа
**lesser** | меньше заданного числа

Можно указать значение `default`, чтобы задать фильтр по умолчанию.

```yaml
age:
    label: Age
    type: number
    default: 14
```

Для условий можно передать произвольный SQL‑код в виде строки; `:value`, `:min` и `:max` будут содержать отфильтрованные значения.

```yaml
age:
    label: Age
    type: number
    conditions:
        greater: age >= :value
        between: age >= :min and age <= :max
```

## PHP-интерфейс

В модели можно определить пользовательский `modelScope`, как в примере ниже.

```yaml
age:
    label: Age
    type: number
    modelScope: numberFilter
```

Метод **scopeNumberFilter** получает значения в `$scope->value`, `$scope->min` и `$scope->max`.

```php
function scopeNumberFilter($query, $scope)
{
    if ($scope->condition === 'equals') {
        $query->where('age', $scope->value);
    }
    elseif ($scope->condition === 'between') {
        $query
            ->where('age', '>=', $scope->min)
            ->where('age', '<=', $scope->max);
    }
    elseif ($scope->condition === 'greater') {
        $query->where('age', '>=', $scope->value);
    }
    else {
        $query->where('age', '<=', $scope->value);
    }
}
```
