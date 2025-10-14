---
subtitle: Проверка отправки форм с помощью AJAX-фреймворка.
---
# Валидация

Валидация форм проверяет пользовательский ввод по набору правил. При использовании [AJAX-фреймворка](../../ajax/introduction.md) валидация выполняется без дополнительной конфигурации: неверное поле получает фокус, а сообщение об ошибке отображается (обычно в виде окна предупреждения).

## Валидация со всплывающими сообщениями

Для простой валидации добавьте атрибут [`data-request-flash`](./flash-messages.md) к тегу формы. Он обеспечивает лаконичный интерфейс для отображения сообщений валидации и подходит для большинства случаев.

```html
<form data-request="onSubmit" data-request-flash>
    <div>
        <label>Name</label>
        <input name="name" />
    </div>

    <button data-attach-loading>
        Submit
    </button>
</form>
```

Внутри [AJAX-обработчика](../../ajax/handlers.md) можно сгенерировать [исключение валидации](../../extend/system/exceptions.md) с помощью класса `ValidationException`, чтобы пометить поле как неверное. В передаваемом массиве (первый аргумент) имена полей используются как ключи, а сообщения об ошибке — как значения.

```php
function onSubmit()
{
    if (!post('name')) {
        throw new ValidationException(['name' => 'You must give a name!']);
    }
}
```

Когда AJAX-фреймворк обнаруживает `ValidationException`, он автоматически фокусирует первое неверное поле и выводит сообщения об ошибке (если они настроены).

## Встроенная валидация

Для более комплексной работы включите встроенную валидацию, добавив атрибут `data-request-validate` к тегу формы. Ниже приведён минимальный пример, в котором сообщение об ошибке отображается внутри формы.

```html
<form data-request="onSubmit" data-request-validate>
    <div class="alert alert-danger" data-validate-error>
        <!-- Validation Message -->
    </div>

    <div>
        <label>Name</label>
        <input name="name" />
    </div>

    <button data-attach-loading>
        Submit
    </button>
</form>
```

### Проверка одного поля

Иногда нужно проверить поле по мере ввода значения. Добавьте атрибут `data-track-input` вместе с `data-request`, и AJAX-фреймворк отправит запрос при каждом вводе пользователя.

```html
<form data-request-validate>
    <div>
        <label>Username</label>
        <input name="username" data-request="onCheckUsername" data-track-input />
    </div>
</form>
```

Выделенный AJAX-обработчик может проверять поле. Если исключение не выбрасывается, значение считается допустимым.

```php
function onCheckUsername()
{
    if (true) {
        throw new ValidationException(['username' => 'Username is taken!']);
    }
}
```

## Использование сервиса валидации

::: aside
Подробнее о доступных правилах см. в статье [Сервис валидации](../../extend/services/validation.md).
:::

Для более сложной валидации используйте фасад `Request`, чтобы применить правила ко всему вводу. Метод `validate` выполняет проверку по указанным правилам (первый аргумент) и возвращает атрибуты с их значениями в виде массива. При ошибке валидации выбрасывается `ValidationException`.

```php
function onSubmit()
{
    $data = Request::validate([
        'name' => 'required',
        'email' => 'required|email',
    ]);

    // При ошибке валидации код сюда не дойдёт

    Flash::success('Jobs done!');
}
```

### Пользовательские сообщения и атрибуты

Чтобы изменить стандартные сообщения, передайте их в метод `validate`. Ключи массива сообщений (третий аргумент) имеют формат `attribute.rule`.

```php
$messages = [
    'email.required' => 'Please type something for the email...',
    'email.email' => 'That email is not an email...!'
];

$data = Request::validate($rules, $messages);
```

Если стандартные сообщения подходят, но нужно переименовать `:attribute`, передайте массив пользовательских атрибутов (четвёртый аргумент).

```php
$attributeNames = [
    'email' => 'e-mail address'
];

$data = Request::validate($rules, [], $attributeNames);
```

## Отображение сообщений об ошибках

Внутри формы можно показать первое сообщение об ошибке, добавив атрибут `data-validate-error` контейнеру. Содержимое контейнера заменяется текстом ошибки, а элемент становится видимым.

```html
<div data-validate-error></div>
```

Чтобы вывести несколько сообщений, добавьте элемент с атрибутом `data-message`. В приведённом примере параграф будет дублироваться и заполняться текстом для каждого сообщения.

```html
<div class="alert alert-danger" data-validate-error>
    <p data-message></p>
</div>
```

### Сообщения для отдельных полей

Чтобы показывать сообщения для конкретных полей, добавьте элемент с атрибутом `data-validate-for`, указав имя поля.

```html
<!-- Input field -->
<input name="phone" />

<!-- Validation message for the field -->
<div data-validate-for="phone"></div>
```

Если элемент оставить пустым, он заполнится текстом ошибки с сервера. При наличии собственного текста будет показан он.

```html
<div data-validate-for="phone">
    Oops.. phone number is invalid!
</div>
```

### Сообщения об ошибках и всплывающие сообщения

При совместном использовании атрибутов `data-request-validate` и [`data-request-flash`](./flash-messages.md) ошибки валидации имеют приоритет и скрывают всплывающие сообщения. Чтобы показать и то и другое, установите атрибут `data-request-flash` в `*`, что включает все типы сообщений, включая валидацию.

```html
<form
    data-request-validate
    data-request-flash="*">
```

## Работа с JavaScript

Чтобы реализовать дополнительную логику для сообщений, обработайте событие `ajax:invalid-field`, которое показывает поле, и `ajax:promise`, чтобы сбрасывать состояние формы при новой отправке. Эти события описаны в [AJAX JavaScript API](../../ajax/javascript-api.md).

```js
addEventListener('ajax:invalid-field', function(event) {
    const { element, fieldName, errorMsg, isFirst } = event.detail;
    element.classList.add('has-error');
});

addEventListener('ajax:promise', function(event) {
    event.target.closest('form').querySelectorAll('.has-error').forEach(function(el) {
        el.classList.remove('has-error');
    });
});
```

## Полный пример

Ниже приведён полный пример валидации формы. Он вызывает обработчик `onSubmitForm`, который включает индикатор на кнопке отправки, выполняет валидацию полей и выводит сообщение об успешной обработке.

Атрибут `data-request-flash` [включает всплывающие сообщения](./flash-messages.md) для успешных уведомлений и отображает текст ошибки, а `data-attach-loading` показывает [индикатор загрузки](./loaders.md) и предотвращает двойные отправки.

```html
<form
    data-request="onSubmitForm"
    data-request-validate
    data-request-flash>
    <div>
        <label>Username</label>
        <input name="username"
            data-request="onCheckUsername"
            data-track-input
            data-attach-loading />
        <span data-validate-for="username"></span>
    </div>

    <div>
        <label>Email</label>
        <input name="email" />
        <span data-validate-for="email"></span>
    </div>

    <button data-attach-loading>
        Submit
    </button>

    <div class="alert alert-danger" data-validate-error>
        <p data-message></p>
    </div>
</form>
```

AJAX-обработчик `onSubmitForm` считывает данные POST и применяет правила валидатора. Если проверка не проходит, выбрасывается `ValidationException`; иначе возвращается сообщение `Flash::success`.

`onCheckUsername` проверяет, доступно ли имя пользователя. В примере жёстко запрещаются **admin** и **jeff**. Проверка выполняется дважды: при вводе и при отправке формы.

```php
function onSubmitForm()
{
    $data = Request::validate([
        'username' => 'required',
        'email' => 'required|email',
    ]);

    $this->onCheckUsername();

    Flash::success('Jobs done!');
}

function onCheckUsername()
{
    $username = strtolower(trim(post('username')));
    $isTaken = in_array($username, ['admin', 'jeff']);

    if ($isTaken) {
        throw new ValidationException(['username' => 'Username is taken!']);
    }
}
```

#### См. также

::: also
* [Сервис валидации](../../extend/services/validation.md)
* [Трейт валидации](../../extend/database/traits.md)
:::
