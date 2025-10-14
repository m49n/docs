---
subtitle: Показать, что страница загружается.
---
# Индикаторы загрузки

Когда AJAX-фреймворк отправляет запрос на сервер, полезно отобразить индикатор загрузки, поскольку страница может обновиться не сразу. Существуют разные подходы и стандартные индикаторы, которые показывают, что AJAX-запрос запущен и выполняется.

## Индикатор прогресса

Важная возможность AJAX-фреймворка — индикатор прогресса, отображаемый в верхней части страницы во время выполнения AJAX-запроса. Индикатор отслеживает AJAX-запросы, появляется, если запрос выполняется дольше 300 мс, и скрывается после завершения.

Чтобы отключить индикатор прогресса для конкретного запроса, задайте атрибут `data-request-progress-bar` со значением `false`.

```html
<button
    data-request="onDoSomething"
    data-request-progress-bar="false">
    Do something
</button>
```

В JavaScript установите опцию `progressBar` запроса в `false`.

```js
oc.ajax('onSilentRequest', { progressBar: false });
```

Чтобы отключить индикатор прогресса глобально, задайте стиль `visibility: hidden` через таблицу стилей.

```css
.oc-progress-bar {
    visibility: hidden;
}
```

Показать индикатор можно через JavaScript, используя объект `oc.progressBar` и методы `show` / `hide`.

```js
oc.progressBar.show();

oc.progressBar.hide();
```

## Кнопка загрузки

При отправке форм пользователи могут случайно нажать кнопку дважды, что приведёт к двойной отправке. Это решается кнопкой загрузки. Во время AJAX-запроса элементы кнопок с атрибутом `data-attach-loading` отключаются, а к ним добавляется CSS-класс `oc-attach-loader`. Класс создаёт индикатор загрузки на кнопках и ссылках с помощью CSS-селектора `:after`.

```html
<a href="#"
    data-request="onDoSomething"
    data-attach-loading>
    Do Something
</a>
```

Если кнопка находится внутри формы с атрибутом `oc-attach-loader`, будет отображён индикатор загрузки.

```html
<form data-request="onSubmit">
    <button data-attach-loading>
        Submit
    </button>
</form>
```

Поскольку элементы input не поддерживают селектор `:after`, после них вставляется новый элемент. Он удаляется после завершения AJAX-запроса. Это полезно с атрибутом `data-track-input`, который отслеживает изменения и отправляет AJAX-запрос.

```html
<input name="username"
    data-request="onCheckUsername"
    data-track-input
    data-attach-loading />
```

Индикатор можно добавить на кнопку вручную с помощью объекта `oc.attachLoader` и методов `show` / `hide`, передавая селектор или элемент первым аргументом.

```js
oc.attachLoader.show('.my-element');

oc.attachLoader.hide('.my-element');
```

## Переключение элементов

Атрибут `data-request-loading` делает элемент видимым во время AJAX-запроса. Значение — CSS-селектор; видимость элемента управляется стилями `display: block` и `display: none`.

```html
<button
    data-request="onPay"
    data-request-loading=".is-loading">
    Pay Now
</button>

<div style="display:none" class="is-loading">
    Processing Payment...
</div>
```

### Обнаружение глобальных запросов

Можно определить, что выполняется глобальный AJAX-запрос, проверив атрибут `data-ajax-progress` на элементе HTML. В таблице стилей это выглядит так:

```css
html[data-ajax-progress] {
    /* Display loading indicators */
}
```

Атрибут также добавляется элементам формы.

```css
form[data-ajax-progress] {
    /* The form is loading */
}
```

### Нацеливание на конкретные обработчики

Иногда требуется показывать индикатор загрузки для конкретного [AJAX-обработчика](../../ajax/handlers.md). Атрибут `data-ajax-progress` содержит имя последнего обработчика, его можно использовать для нацеливания на нужный запрос.

```html
<form>
    <button data-request="onPay">Pay Now</button>
    <button data-request="onCancel">Cancel</button>

    <div class="is-payment-loading">
        Processing Payment...
    </div>
</form>
```

Настроить отображение можно через селектор по значению атрибута в таблице стилей.

```css
.is-payment-loading {
    display: none;
}

form[data-ajax-progress=onPay] .is-payment-loading {
    display: block;
}
```

## Работа с JavaScript

Для более сложных сценариев подключайтесь к [AJAX JavaScript API](../../ajax/javascript-api.md), используя события `ajax:promise` и `ajax:always`. Эти события можно привязывать к документу, форме или целевым элементам.

```js
formElement.addEventListener('ajax:promise', function() {
    // A new request has started
});

formElement.addEventListener('ajax:always', function() {
    // A request has ended
});
```

Следующий пример отключает все поля ввода внутри формы на время выполнения запроса.

```js
addEventListener('ajax:promise', function(event) {
    event.target.closest('form').querySelectorAll('input').forEach(function(el) {
        el.disabled = true;
    });
});

addEventListener('ajax:always', function() {
    event.target.closest('form').querySelectorAll('input').forEach(function(el) {
        el.disabled = false;
    });
});
```
