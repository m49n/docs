---
subtitle: Область фильтра
shortname: Dropdown
---
# Область Dropdown

`dropdown` — фильтр с выбором одного варианта из нескольких.

```yaml
status:
    type: dropdown
    options:
        pending: Pending
        active: Active
        closed: Closed
```

Поддерживаются следующие свойства фильтра.

Property | Description
------------- | -------------
**options** | варианты для фильтра в виде массива.
**optionsMethod** | получить варианты из метода модели или статического метода (`Class::method`).
**conditions** | произвольный SQL‑запрос, используемый фильтром.
**emptyOption** | текст, отображаемый при отсутствии выбранного значения.
**modelScope** | применяет [область запроса модели](../../extend/database/model.md) к запросу фильтра; можно указать метод модели или статический метод PHP‑класса (`Class::method`). Первым аргументом будет модель, к которой виджет привязывает значение, то есть родительская модель.

Для условий можно передать произвольный SQL‑код в виде строки; `:value` будет содержать отфильтрованное значение.

```yaml
status:
    type: dropdown
    conditions: status = :value
    # ...
```

Фильтр `dropdown` не отображает метку, поэтому для задания исходного состояния используйте свойство `emptyOption`.

```yaml
status:
    type: dropdown
    emptyOption: Select Status
    # ...
```

## PHP-интерфейс

В модели можно определить пользовательский `modelScope`, как в примере ниже.

```yaml
status:
    label: Status
    type: dropdown
    modelScope: applyStatusCode
    options:
        active: Active
        deleted: Deleted
```

Метод **scopeApplyStatusCode** получает значение в `$scope->value`.

```php
public function scopeApplyStatusCode($query, $scope)
{
    if ($scope->value === 'active') {
        return $query->withoutTrashed();
    }

    if ($scope->value === 'deleted') {
        return $query->onlyTrashed();
    }
}
```

Варианты можно формировать динамически, передав методу модели имя в свойство `optionsMethod`.

```yaml
status:
    label: Status
    type: dropdown
    optionsMethod: getStatusOptions
```

Метод **getStatusOptions**.

```php
public function getStatusOptions()
{
    return [
        'active' => 'Active',
        'deleted' => 'Deleted',
    ];
}
```
