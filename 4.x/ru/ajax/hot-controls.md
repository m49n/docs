---
subtitle: Создание наблюдаемых HTML-контролов, связанных с JavaScript.
---
# Hot Controls

October CMS включает простую реализацию [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver), которая позволяет определять HTML-контролы и отслеживать, когда они добавляются или удаляются со страницы. Теперь можно инициализировать или деинициализировать контролы, добавляемые или удаляемые через [AJAX](./update-partials.md) или обновления [turbo router](./turbo-router.md).

## Регистрация наблюдаемого контрола

::: aside
Функцию можно вызывать многократно — будет использовано **последнее определение**.
:::

В базовом варианте функция JavaScript `oc.registerControl` принимает уникальное имя контрола (первый аргумент) и определение класса (второй аргумент), расширяющего базовый класс `oc.ControlBase`.

```js
oc.registerControl('hello', class extends oc.ControlBase {
    // ...
});
```

Имя контрола используется для привязки к DOM-элементу, представляющему контрол, посредством атрибута `data-control`. Например, контрол с именем **hello** будет отслеживать страницу на предмет появления элементов с атрибутом `data-control="hello"`.

```html
<div data-control="hello"></div>
```

Методы `connect` и `disconnect` внутри определения класса вызываются при добавлении или удалении контрола со страницы. Это может происходить в любой момент, поскольку наблюдатель постоянно отслеживает изменения DOM.

```js
class extends oc.ControlBase {
    connect() {
        // Element has appeared in DOM
    }

    disconnect() {
        // Element was removed from DOM
    }
}
```

## Инициализация контрола

Метод `init` позволяет загрузить конфигурацию контрола по умолчанию и настроить его дочерние элементы.

```js
class extends oc.ControlBase {
    init() {
        // Establish the control before running logic
    }
}
```

::: tip
Метод `init` вызывается один раз для каждого контрола, а `connect` — каждый раз при добавлении или удалении контрола из DOM, например при перемещении элемента в новое место.
:::

### Конфигурация

Все атрибуты `data-` элемента контрола формируют доступную конфигурацию.

```html
<div data-control="hello" data-favorite-color="red"></div>
```

Получить значения конфигурации можно через свойство `this.config`. Атрибуты данных преобразуются в camelCase без префикса `data-`. Например, атрибут `data-favorite-color` доступен как `this.config.favoriteColor`.

```js
class extends oc.ControlBase {
    init() {
        this.favoriteColor = this.config.favoriteColor || 'green';
    }

    connect() {
        console.log(`Favorite color? ${this.favoriteColor}!`);
    }
}
```

### Дочерние элементы

Для выбора дочерних элементов внутри родительского класса контрола можно использовать любые селекторы, CSS или атрибуты данных.

```html
<div data-control="hello">
    <input class="name" disabled />
</div>
```

Родительский элемент контрола доступен через `this.element`. Дочерний элемент можно получить с помощью `querySelector` для одного элемента или `querySelectorAll` для нескольких.

```js
class extends oc.ControlBase {
    init() {
        this.$name = this.element.querySelector('input.name');
    }

    connect() {
        this.$name.value = 'Jeff';
        this.$name.disabled = false;
    }
}
```

## Ссылки на другие контролы

Функция `oc.fetchControl` возвращает экземпляр контрола из существующего элемента. В качестве аргумента принимает селектор или элемент. Возвращённый экземпляр позволяет вызывать методы или обращаться к свойствам, определённым в классе контрола.

```js
const searchControl = oc.fetchControl(element);
```

Можно также передать строку-селектор и имя контрола вторым аргументом (необязательно). Это полезно, когда на одном элементе привязано несколько контролов и нужно указать конкретный идентификатор.

```js
const searchControl = oc.fetchControl('[data-control=search]', 'search');
```

Функция `oc.importControl` возвращает зарегистрированный класс контрола — полезно для вызова статических методов класса. Функция принимает строковый идентификатор контрола.

```js
const searchControlClass = oc.importControl('search');
```

Функция `oc.observeControl` немедленно создаёт экземпляр контрола и привязывает его к элементу. Это удобно, когда у элемента нет атрибута `data-control`, но нужно привязать контрол без ожидания событий наблюдателя.

```js
const searchControl = oc.observeControl(element, 'search');
```

## Работа с событиями

Наблюдаемые контролы могут связывать события локально или глобально. Локальные события отвязываются автоматически, а глобальные необходимо отвязывать вручную в методе `disconnect`.

### Локальные события

Локальный обработчик привязывается через функцию `listen`, и такие обработчики отвязываются автоматически. Чтобы привязать слушатель к самому элементу контрола, передайте в `listen` имя события и функцию-обработчик.

```js
class extends oc.ControlBase {
    connect() {
        this.listen('dblclick', this.onDoubleClick);
    }

    onDoubleClick() {
        console.log('You double clicked my control!');
    }
}
```

Чтобы привязать локальный обработчик к дочернему элементу, передайте имя события, CSS-селектор и функцию-обработчик. `event.delegateTarget` всегда содержит элемент, совпавший с селектором.

```js
class extends oc.ControlBase {
    connect() {
        this.listen('click', '.toolbar-find-button', this.onClickFindButton);
    }

    onClickFindButton(event) {
        console.log('You clicked the find button inside the control: ' + event.delegateTarget.innerText);
    }
}
```

Можно также привязать обработчик к объекту DOM: передайте имя события, HTML-элемент и функцию-обработчик.

```js
class extends oc.ControlBase {
    init() {
        this.$name = this.element.querySelector('input.name');
    }

    connect() {
        this.listen('click', this.$name, this.onClickNameInput);
    }

    onClickNameInput() {
        console.log('You clicked the name input inside the control!');
    }
}
```

### Глобальные события

Глобальные события привязываются и удаляются с помощью нативных функций JavaScript `addEventListener` и `removeEventListener`. Обработчик (второй аргумент) ссылается на метод класса того же экземпляра. Метод `proxy` привязывает текущий контекст к вызову функции.

```js
class extends oc.ControlBase {
    connect() {
        addEventListener('keydown', this.proxy(this.onKeyDown));
    }

    disconnect() {
        removeEventListener('keydown', this.proxy(this.onKeyDown));
    }

    onKeyDown(event) => {
        if (event.key === 'Escape') {
            // Escape button was pressed
        }
    }
}
```

::: tip
Чтобы избежать утечек памяти, обязательно отвязывайте глобальные события — тогда сборщик мусора сможет их удалить.
:::

### Отправка событий

Контролы могут отправлять события, передавая имя события в функцию `dispatch`. Событие генерируется на DOM-элементе, а имя события получает префикс имени контрола. В следующем примере для контрола с именем **hello** событие будет называться **hello:ready**.

```js
oc.registerControl('hello', class extends oc.ControlBase {
    connect() {
        this.dispatch('ready');
    }
});
```

Теперь можно слушать подключение контрола и получить объект через `oc.fetchControl` из целевого элемента события.

```js
addEventListener('hello:ready', function(ev) {
    const helloControl = oc.fetchControl(ev.target);
});
```

Во втором аргументе можно передать параметры, например `detail`. В слушателе к данным можно обратиться как к **ev.detail.foo**.

```js
this.dispatch('ready', { detail: {
    foo: 'bar'
}});
```

Можно указать иной `target`, по умолчанию событие отправляется от привязанного элемента.

```js
this.dispatch('ready', { target: window });
```

Если задать `prefix` со значением false, имя события станет глобальным: в примере событие называется **hello-ready**, а не **hello:hello-ready**.

```js
this.dispatch('hello-ready', { prefix: false });
```

## Примеры использования

### Пример на чистом JS

Следующий пример демонстрирует простую HTML-форму с полем имени и кнопкой приветствия. Класс контрола инициализирует элементы ввода и вывода, а затем слушает событие щелчка на кнопке Greet. При щелчке выводится приветствие с введённым именем.

```html
<div data-control="hello-world">
    <input type="text" class="name" />

    <button class="greet">
        Greet
    </button>

    <span class="output">
    </span>
</div>

<script>
oc.registerControl('hello-world', class extends oc.ControlBase {
    init() {
        this.$name = this.element.querySelector('input.name');
        this.$output = this.element.querySelector('span.output');
    }

    connect() {
        this.listen('click', 'button.greet', this.onGreet);
    }

    onGreet() {
        this.$output.textContent = `Hello, ${this.$name.value}!`;
    }
});
</script>
```

### Пример с Google Maps

Следующий пример показывает простую интеграцию сторонней JavaScript-библиотеки, например Google Maps API. Библиотека `Map` инициализируется на элементе `div` контрола, когда он появляется на странице. При удалении контрола карта уничтожается через метод `destroy`, а свойство устанавливается в `null`, что предотвращает утечки памяти.

```html
<div data-control="google-map"></div>

<script>
oc.registerControl('google-map', class extends oc.ControlBase {
    connect() {
        this.map = new Map(this.element, {
            center: { lat: -34.397, lng: 150.644 },
            zoom: 8
        });
    }

    disconnect() {
        this.map.destroy();
        this.map = null;
    }
});
</script>
```

### Пример с Vue.js

Следующий пример демонстрирует использование сторонней технологии для создания динамического интерфейса, в данном случае [Vue.js](https://vuejs.org/guide/essentials/event-handling.html). Экземпляр Vue (ViewModel, vm) создаётся и уничтожается по мере необходимости.

```html
<div data-control="my-vue-control">
    <div data-vue-template>
        <button @click="greet">Greet</button>
    </div>
</div>

<script>
oc.registerControl('my-vue-control', class extends oc.ControlBase {
    connect() {
        this.vm = new Vue({
            el: this.element.querySelector('[data-vue-template]'),
            data: {
                name: 'October CMS'
            },
            methods: {
                greet: this.greet
            }
        });
    }

    disconnect() {
        this.vm.$destroy();
    }

    greet(event) {
        alert('Hello ' + this.name + '!')
    }
});
</script>
```

Горячие контролы также можно использовать для инициализации компонентов Vue через `Vue.component`, делая их доступными внутри контролов. Следующий компонент доступен как `<my-vue-component></my-vue-component>` в Vue, однако важно регистрировать такие шаблоны до того, как их используют другие контролы.

```html
<div data-control="my-vue-component">
    <button @click="greet">Greet</button>
</div>

<script>
oc.registerControl('my-vue-component', class extends oc.ControlBase {
    init() {
        Vue.component('my-vue-component', {
            template: this.element,
            methods: {
                greet: this.greet
            }
        });
    }

    connect() {
        this.element.style.display = 'none';
    }

    greet(event) {
        alert('Hello!');
    }
});
</script>
```
