---
subtitle: Узнайте, как фильтровать записи в списке.
---
# Фильтрация записей

October CMS предоставляет возможности для фильтрации записей базы данных. Для поведений, поддерживающих фильтры, можно определить параметр **filter**, чтобы активировать функцию. Области фильтрации обычно хранятся в каталоге конфигурации модели в файле **scopes.yaml**.

## Настройка поведения

Поведения бэкенда [List Controller](./list-controller.md) и [Relation Controller](../forms/relation-controller.md) можно фильтровать, добавив в конфигурацию свойство **filter**. После определения доступные фильтры отображаются над списком.

```yaml
# config_list.yaml

# ...

# Отображает фильтр списка
filter: $/october/test/models/user/scopes.yaml
```

## Определение областей фильтра

::: aside
Доступные параметры областей фильтра описаны на странице [определения областей фильтра](../../element/filter-scopes.md).
:::

Фильтры также управляются собственным конфигурационным файлом, содержащим **области**. Каждая область описывает аспект, по которому можно отфильтровать список. Ниже приведён пример типичного содержимого файла с определением областей фильтра.

```yaml
# scopes.yaml
scopes:

    category:
        label: Category
        modelClass: Acme\\Blog\\Models\\Category
        conditions: category_id in (:value)
        nameFrom: name

    status:
        label: Status
        type: group
        conditions: status in (:value)
        options:
            pending: Pending
            active: Active
            closed: Closed

    published:
        label: Hide published
        type: checkbox
        default: 1
        conditions: is_published <> true

    approved:
        label: Approved
        type: switch
        default: 2
        conditions:
            - is_approved <> true
            - is_approved = true

    created_at:
        label: Date
        type: date
        conditions:
            after: created_at >= ':value'
            between: created_at >= ':after' AND created_at <= ':before'
```

### Зависимости областей фильтра

Области фильтра могут объявлять зависимости от других областей, определяя свойство `dependsOn`, которое обеспечивает серверное обновление областей при изменении зависимостей. Когда области, указанные как зависимости, изменяются, определяющая область сбрасывается и обновляется динамически. Это позволяет менять варианты, доступные для области.

```yaml
country:
    label: Country
    type: group
    conditions: country_id in (:value)
    modelClass: October\\Test\\Models\\Location
    options: getCountryOptions

city:
    label: City
    type: group
    conditions: city_id in (:value)
    modelClass: October\\Test\\Models\\Location
    options: getCityOptions
    dependsOn: country
```

В приведённом выше примере область `city` будет обновляться при изменении области `country`. Любая область, определяющая свойство `dependsOn`, получит массив всех текущих объектов областей виджета Filter вместе с их значениями; ключами массива являются имена областей.

```php
public function getCountryOptions()
{
    return Country::lists('name', 'id');
}

public function getCityOptions($scopes = null)
{
    if (!empty($scopes['country']->value)) {
        return City::whereIn('country_id', $scopes['country']->value)->lists('name', 'id');
    }
    else {
        return City::lists('name', 'id');
    }
}
```

Можно фильтровать определения областей, переопределив в модели метод `filterScopes`. Это позволяет управлять видимостью и другими параметрами областей в зависимости от значений других областей. Метод принимает два аргумента: **$scopes** — объект с областями, определёнными конфигурацией, и **$context** — активный контекст фильтра.

```php
public function filterScopes($scopes, $context = null)
{
    if ($scopes->disable_roles->value) {
        $scopes->roles->hidden = true;
    }
}
```

Приведённая логика скрывает область `roles`, если установлено значение `disable_roles`. Логика применяется при первоначальной загрузке фильтра и при обновлении через зависимые области. Например, так выглядят связанные определения областей фильтра.

```yaml
disable_roles:
    type: checkbox
    label: Disable Roles

roles:
    type: text
    label: Role
    dependsOn: disable_roles
```
