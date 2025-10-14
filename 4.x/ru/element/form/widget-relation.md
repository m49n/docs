---
subtitle: Виджет формы
shortname: Relation
---
# Поле Relation

Виджет формы `relation` выводит выпадающий список или список с флажками в зависимости от типа связи. Для одиночных связей отображается выпадающий список, для множественных — список с флажками. Подпись каждой связи берётся из определения `nameFrom` или `select`.

```yaml
categories:
    label: Categories
    type: relation
```

Поддерживаются и часто используются следующие [свойства поля](../form-fields.md).

Свойство | Описание
------------- | -------------
**label** | имя, отображаемое пользователю.
**comment** | размещает описательный комментарий под полем.
**nameFrom** | имя атрибута модели, используемое для отображения подписи связи. Значение по умолчанию — `name`.
**excludeFrom** | атрибут родительской модели, используемый для исключения связанных ключей из списка, необязательный.
**select** | пользовательское выражение SQL SELECT для получения подписи.
**emptyOption** | текст, отображаемый при отсутствии доступных вариантов.
**conditions** | произвольное выражение WHERE, применяемое к запросу модели.
**modelScope** | применяет [область запроса модели](../../extend/database/model.md) к **связанной модели формы**. Можно указать метод модели или статический метод PHP (`Class::method`).
**defaultSort** | задаёт столбец и направление сортировки по умолчанию. Поддерживает строку с именем столбца или массив с ключами `column` и `direction`. Для направления используйте `asc` (по возрастанию, по умолчанию) или `desc` (по убыванию).
**useController** | автоматически определяет, настроено ли поле через [поведение Relation Controller](../../extend/forms/relation-controller.md), и использует его. Значение по умолчанию — `true`.
**controller** | задаёт массив для ручной интеграции с [поведением Relation Controller](../../extend/forms/relation-controller.md).

Используйте свойство `nameFrom`, чтобы настроить подпись связанной записи.

```yaml
categories:
    label: Categories
    type: relation
    nameFrom: title
```

Также можно формировать подпись пользовательским выражением `select`. Здесь подходит любое корректное выражение SQL.

```yaml
user:
    label: User
    type: relation
    select: concat(first_name, ' ', last_name)
```

## Применение условий

Доступные записи можно фильтровать с помощью условий SQL или PHP по методикам ниже.

### Условие SQL-запроса

Ограничить связанную модель можно через необработанный SQL-запрос, задав свойство `conditions`.

```yaml
user:
    label: User
    type: relation
    conditions: is_featured = true
```

Значение также поддерживает простые параметры, полученные из атрибутов родительской модели. Имена параметров начинаются с двоеточия (`:`).

```yaml
country:
    label: Country
    type: relation

state:
    label: State
    type: relation
    dependsOn: country
    conditions: custom_country_id = :country_id
```

### Области запроса PHP

Чтобы отфильтровать результаты, можно указать область модели через свойство `modelScope`.

```yaml
user:
    label: User
    type: relation
    modelScope: withTrashed
```

Свойство `modelScope` позволяет связать два поля, например модели `Country` и `State`, где доступные штаты фильтруются по выбранной стране. Свойство `dependsOn` включает [зависимости полей](../../extend/forms/field-dependencies.md) и обновляет варианты `state` при выборе `country`.

```yaml
country:
    label: Country
    type: relation

state:
    label: State
    type: relation
    dependsOn: country
    modelScope: filterStates
```

Значение `modelScope` **filterStates** соответствует методу `scopeFilterStates`, определённому в модели `State`. Модель `$model` (второй аргумент), передаваемая в [область запроса модели](../../extend/database/model.md), позволяет получить выбранную страну и отфильтровать доступные варианты.

```php
public function scopeFilterStates($query, $model)
{
    if ($countryId = $model->country_id) {
        $query->where('country_id', $countryId);
    }
}
```

## Интеграция с Relation Controller

Если контроллер реализует [поведение Relation Controller](../../extend/forms/relation-controller.md), и поле определено там, оно будет отображено по этому определению. Установите свойство `useController` в `false`, чтобы отключить эту функциональность.

```yaml
countries:
    label: Categories
    type: relation
    useController: false
```

Свойство `controller` позволяет задать конфигурацию непосредственно в поле.

```yaml
products:
    label: Products
    tab: Products
    type: relation
    controller:
        label: Product
        list: $/october/test/models/product/columns.yaml
        form: $/october/test/models/product/fields.yaml
```
