---
subtitle: Область фильтра
shortname: Group
---
# Область Group

`group` — фильтр по набору элементов, как правило, связанных моделей или массива предопределённых вариантов.

Чтобы фильтровать по модели, укажите свойства `modelClass` и `nameFrom`, задающие модель и атрибут для отображения.

```yaml
roles:
    label: Role
    type: group
    nameFrom: name
    modelClass: October\Test\Models\Role
```

Поддерживаются следующие свойства фильтра.

Property | Description
------------- | -------------
**options** | варианты для фильтра в виде массива.
**optionsMethod** | получить варианты из метода модели или статического метода (`Class::method`).
**optionsScope** | применяет [область модели](../filter-scopes.md) к запросу, формирующему варианты.
**conditions** | произвольный SQL‑запрос, используемый фильтром.
**nameFrom** | имя столбца модели, используемое для отображения названия. Значение по умолчанию: `name`.
**modelClass** | класс модели, используемой для получения записей фильтра.
**modelScope** | применяет [область модели](../filter-scopes.md) к запросу фильтра.
**matchMode** | определяет способ применения выбранных значений: `include`, `exclude` или `toggle`. Значение по умолчанию: `include`.

Чтобы фильтровать по массиву, задайте свойство `options`.

```yaml
status:
    label: Role
    type: group
    options:
        developer: Developer
        publisher: Publisher
```

Для условий можно передать произвольный SQL‑код в виде строки; `:value` будет содержать отфильтрованные значения.

```yaml
status:
    label: Role
    type: group
    conditions: role in (:value)
    # ...
```

Можно указать `default` как массив с выбранными ключами.

```yaml
status:
    # ...
    default:
        - developer
        - publisher
```

Используйте свойство `matchMode`, чтобы управлять логикой применения фильтра: включать, исключать или переключать выбранные элементы.

```yaml
status:
    # ...
    matchMode: toggle
```

## PHP-интерфейс

В модели можно определить пользовательский `modelScope`, как в примере ниже.

```yaml
roles:
    label: Role
    type: group
    nameFrom: name
    modelClass: October\Test\Models\Role
    modelScope: groupFilter
```

Метод **scopeGroupFilter** получает значения в `$scope->value`.

```php
public function scopeGroupFilter($query, $scope)
{
    return $query->whereHas('roles', function($q) use ($scope) {
        $q->whereIn('id', $scope->value);
    });
}
```

Варианты можно формировать динамически, передав методу модели имя в свойство `optionsMethod`.

```yaml
roles:
    label: Role
    type: group
    nameFrom: name
    modelClass: October\Test\Models\Role
    optionsMethod: getRoleGroupOptions
```

Метод **getRoleGroupOptions**.

```php
public function getRoleGroupOptions()
{
    return $this->whereNull('parent_id')->pluck('name', 'id')->all();
}
```

Свойство `optionsScope` позволяет применить область к стандартному запросу, который получает варианты.

```yaml
roles:
    label: Role
    type: group
    nameFrom: name
    modelClass: October\Test\Models\Role
    optionsScope: applyRoleOptionsFilter
```

```php
public function scopeApplyRoleOptionsFilter($query)
{
    return $query->where('id', '<>', 1);
}
```
