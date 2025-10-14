---
subtitle: Узнайте, как поля могут зависеть от других полей.
---
# Зависимости полей

Подобно [условиям полей формы](../../element/form-fields.md), поля формы могут объявлять зависимости от других полей через свойство `dependsOn`. Это более надёжное серверное решение для обновления полей при изменении их зависимостей.

Когда поля, объявленные как зависимости, изменяются, определяющее их поле обновляется с помощью AJAX‑фреймворка. Это позволяет работать со свойствами поля в методах `filterFields` или менять доступные для него опции.

```yaml
country:
    label: Country
    type: dropdown

state:
    label: State
    type: dropdown
    dependsOn: country
```

В приведённом выше примере поле формы `state` будет обновляться при изменении значения поля `country`. Во время обновления текущие данные формы будут помещены в модель, чтобы их можно было использовать при формировании вариантов в выпадающем списке.

```php
public function getCountryOptions()
{
    return ['au' => 'Australia', 'ca' => 'Canada'];
}

public function getStateOptions()
{
    if ($this->country == 'au') {
        return ['act' => 'Capital Territory', 'qld' => 'Queensland', ...];
    }
    elseif ($this->country == 'ca') {
        return ['bc' => 'British Columbia', 'on' => 'Ontario', ...];
    }
}
```

## Фильтрация полей

Можно фильтровать определения полей формы, переопределив метод `filterFields` в используемой модели. Это позволяет управлять видимостью и другими свойствами полей в зависимости от данных модели. Метод принимает два аргумента: **$fields** — объект полей, уже определённых [конфигурацией полей](../../element/form-fields.md), и **$context** — активный контекст формы.

```php
public function filterFields($fields, $context = null)
{
    if ($this->source_type === 'http') {
        $fields->source_url->hidden = false;
        $fields->git_branch->hidden = true;
    }
    elseif ($this->source_type === 'git') {
        $fields->source_url->hidden = false;
        $fields->git_branch->hidden = false;
    }
    else {
        $fields->source_url->hidden = true;
        $fields->git_branch->hidden = true;
    }
}
```

Значение `$context` содержит контекст формы (create, update и т. д.) во время отображения и сохранения формы, однако при обновлении формы оно всегда равно **refresh**. Это полезно, когда нужно заполнить поля новыми значениями, не сохраняя их. Пример ниже сбрасывает значение поля parent name, если во время обновления изменился родитель, но не влияет на сохраняемое значение.

```php
public function filterFields($fields, $context = null)
{
    if ($context === 'refresh' && $this->parent) {
        $fields->parent_name->value = $this->parent->name;
    }
}
```

Приведённая логика устанавливает флаг `hidden` на определённых полях, проверяя значение атрибута модели `source_type`. Она применяется при первом открытии формы, а также при обновлении формы зависимым полем. Ниже показаны соответствующие определения полей формы.

```yaml
source_type:
    label: Source Type
    type: dropdown
    options:
        git: Git
        http: Http
        upload: Upload

source_url:
    label: Source URL
    type: text
    dependsOn: source_type

git_branch:
    label: Git Branch
    type: text
    dependsOn: source_type
```

## Обновление через AJAX

Иногда требуется запустить обработчик AJAX вручную при изменении значения поля. Для этого можно указать свойство `changeHandler`. В следующем примере при изменении значения будет вызван обработчик AJAX **onChangeContent**.

```yaml
content:
    label: Content
    type: textarea
    changeHandler: onChangeContent
```

Обработчик AJAX добавляется в контроллер стандартным образом. В примере ниже используется фасад `Flash`, чтобы вывести сообщение при обновлении поля.

```php
public function onChangeContent()
{
    Flash::success('Great job!');
}
```

Если нужно обновить другие поля, используйте метод `formRefreshFields`, который предоставляет контроллер формы.

```php
public function onChangeContent()
{
    return $this->formRefreshFields('is_positive');
}
```

Можно также обновить несколько полей сразу, передав массив.

```php
public function onChangeContent()
{
    return $this->formRefreshFields(['is_positive', 'internal_comments']);
}
```

#### См. также

::: also
* [Form Field Conditions](../../element/form-fields.md)
:::
