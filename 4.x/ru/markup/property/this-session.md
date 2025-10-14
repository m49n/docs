---
subtitle: Свойство Twig
---
# this.session

Текущий менеджер сессии доступен через `this.session`; он возвращает объект `Illuminate\Session\Store` (см. [конфигурацию сессии](../../extend/services/session.md)).

## this.session.get()

Получить данные из сессии можно, передав имя ключа первым аргументом в `this.session.get`.

```twig
{{ this.session.get('key') }}
```

Вторым аргументом можно задать значение по умолчанию.

```twig
{{ this.session.get('key', 'default') }}
```

## this.session.has()

Метод `this.session.has` позволяет определить, существует ли элемент в сессии.

```twig
{% if this.session.has('key') %}
    <h1>Ключ найден в сессии</h1>
{% endif %}
```

## this.session.put()

Метод `this.session.put` используется для сохранения данных сессии.

```twig
{% do this.session.put('my-preference', 'value') %}
```

## this.session.forget()

Метод `this.session.forget` удаляет один ключ (первый аргумент) из сессии.

```twig
{% do this.session.forget('key') %}
```

Чтобы удалить все данные сессии, используйте `this.session.flush`.

```twig
{% do this.session.flush() %}
```

#### См. также

::: also
* [Сервис сессий](../../extend/services/session.md)
:::
