---
subtitle: Узнайте, как фильтровать списки с помощью областей запроса (scope).
---
# Области фильтра (Filter Scopes)

Области фильтра — это определения областей запроса (scope), которые используют фильтры, часто вместе со списком. Подобно колонкам списков, они применяются в следующих местах:

- [Поведение Backend List Controller](../extend/lists/list-controller.md)
- [Поведение Backend Relation Controller](../extend/forms/relation-controller.md)

Все области фильтра идентифицируются по своему свойству **type**.

```yaml
scopes:
    myscope:
        type: date
        # ...
```

## Доступные области

Доступны следующие области фильтра:

<div class="content-list-p" markdown="1">

[Checkbox](./filter/scope-checkbox.md)
[Switch](./filter/scope-switch.md)
[Text](./filter/scope-text.md)
[Number](./filter/scope-number.md)
[Dropdown](./filter/scope-dropdown.md)
[Group](./filter/scope-group.md)
[Date](./filter/scope-date.md)

</div>

## Свойства области

Для каждой области можно указать следующие свойства (где применимо).

Property | Description
------------- | -------------
**label** | имя при отображении области фильтра пользователю.
**type** | определяет способ отображения области. Значение по умолчанию: `group`.
**conditions** | включает или отключает функции условий либо задаёт выражение where в сыром виде, применяемое к каждому условию; подробности см. в описании типа области.
**modelClass** | класс модели, используемой как источник данных и ссылка для локальных вызовов методов.
**modelScope** | задаёт [область запроса модели](../extend/database/model.md), определённую в **модели списка**, которая применяется к запросу списка. Первый аргумент содержит объект запроса (как в обычном методе области), второй аргумент содержит определение области со значениями.
**options** | варианты выбора для фильтрации по нескольким значениям, передаются массивом.
**optionsMethod** | получить варианты из метода модели или статического метода, например `Class::method`.
**emptyOption** | необязательная метка для осознанного пустого выбора.
**default** | задаёт значение по умолчанию для фильтра в виде массива, строки или целого числа — в зависимости от типа фильтра.
**permissions** | [разрешения](../extend/backend/permissions.md), которые должны быть у текущего пользователя бэкенда, чтобы область фильтра была доступна. Поддерживает строку с одним разрешением или массив, из которого требуется хотя бы одно разрешение.
**dependsOn** | строка или массив имён других областей, от которых зависит данная область. При изменении зависимых областей эта область сбрасывается.
**nameFrom** | имя атрибута модели для отображения метки фильтра. Значение по умолчанию: `name`.
**valueFrom** | определяет атрибут модели, используемый как исходное значение. По умолчанию берётся из имени области.
**order** | числовой вес для определения порядка отображения; значение по умолчанию увеличивается на 100 для каждой области.
**after** | располагает область после существующей, используя порядок отображения (+1).
**before** | располагает область перед существующей, используя порядок отображения (−1).

### Применение областей модели

Большинство фильтров применяют ограничения области по умолчанию. Свойство `modelScope` позволяет накладывать пользовательские ограничения на запрос фильтра с помощью [определения области модели](../extend/database/model.md).

```yaml
myfilter:
    label: My Filter
    type: group
    modelScope: applyMyFilter
```

В этом примере ожидается, что на основной модели определена связь **myfilter** и метод **scopeApplyMyFilter** в связанной модели. Второй аргумент метода содержит определение области со значениями.

```php
public function scopeApplyMyFilter($query, $scope)
{
    return $query->whereIn('my_filter_attribute', (array) $scope->value);
}
```

Свойство `modelScope` также можно задать как статический метод PHP (`Class::method`).

```yaml
myfilter:
    label: My Filter
    type: group
    modelScope: "App\\MyCustomClass::applyMyFilter"
```

В этом случае метод должен быть объявлен статическим в указанном классе.

```php
namespace App;

class MyCustomClass
{
    public static function applyMyFilter($query, $scope)
    {
        return $query->whereIn('my_filter_attribute', (array) $scope->value);
    }
}
```

### Зависимости областей

Свойство `dependsOn` позволяет связывать несколько фильтров. Когда зависимость изменяется, область фильтра сбрасывается, чтобы учесть новые условия. Рассмотрим пример двух определений.

```yaml
country:
    label: Country
    type: group

state:
    label: State
    type: group
    dependsOn: country
    optionsMethod: getCityOptionsForFilter
```

Область `state` зависит от значения области `country`, а варианты фильтруются в PHP методом **getCityOptionsForFilter**. Первый аргумент метода содержит весь набор определений областей с текущими значениями.

```php
public function getCityOptionsForFilter($scopes = null)
{
    if ($scopes->country && ($countryIds = $scopes->country->value)) {
        return self::whereIn('country_id', $countryIds)->lists('name', 'id');
    }

    return self::lists('name', 'id');
}
```
