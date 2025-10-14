---
subtitle: Область фильтра
shortname: Text
---
# Область Text

`text` — фильтр по строковому значению с условиями `exact` или `contains`.

```yaml
username:
    label: Username
    type: text
```

Поддерживаются следующие свойства фильтра.

Property | Description
------------- | -------------
**conditions** | для каждого условия можно указать `true` или `false`, чтобы сделать его доступным, либо передать строку с произвольным SQL‑выражением для выбранных условий. Значение по умолчанию: `true`.
**modelScope** | применяет [область запроса модели](../../extend/database/model.md) к запросу фильтра; можно указать метод модели или статический метод PHP‑класса (`Class::method`). Первым аргументом будет модель, к которой виджет привязывает значение, то есть родительская модель.

Для фильтра доступны следующие `conditions`.

Condition | Description
------------- | -------------
**equals** | совпадает с точным текстом
**contains** | содержит указанный текст

Чтобы искать только точное совпадение, передайте **equals** в параметр `conditions`. Чтобы находить результаты, содержащие часть текста, используйте в `conditions` значение **contains**.

```yaml
username:
    label: Username
    type: text
    conditions:
        equals: true
```

Для условий можно передать произвольный SQL‑код в виде строки; `:value` будет содержать отфильтрованное значение.

```yaml
username:
    label: Username
    type: text
    conditions:
        equals: username = :value
        contains: username like %:value%
```

## PHP-интерфейс

В модели можно определить пользовательский `modelScope`, как в примере ниже.

```yaml
username:
    label: Username
    type: text
    modelScope: textFilter
```

Метод **scopeTextFilter** получает значение в `$scope->value`.

```php
function scopeTextFilter($query, $scope)
{
    if ($scope->condition === 'equals') {
        $query->where('username', $scope->value);
    }
    else {
        $query->where('username', 'LIKE', "%{$scope->value}%");
    }
}
```
