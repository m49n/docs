---
subtitle: Динамическая загрузка содержимого в модальное окно
---
# Модальные окна

Модальные окна можно отображать через AJAX-фреймворк, выполняя обновление частичного представления, которое нацелено на элемент содержимого модального окна. Когда элемент ожидает обновление, к нему добавляется атрибут `data-ajax-updating`, используемый для отображения состояния загрузки во время получения содержимого.

::: tip
В примерах используется [компонент модального окна](https://getbootstrap.com/docs/5.2/components/modal/), предоставляемый [Bootstrap 5](https://getbootstrap.com).
:::

## Содержимое модального окна

Содержимое модального окна определяется в частичном представлении **my-modal-content.htm**.

```html
<div class="modal-content">
    <div class="modal-header">
        <h5 class="modal-title">
            Modal Title
        </h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
    </div>
    <div class="modal-body">
        <p>Modal body text goes here.</p>
    </div>
    <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">
            Close
        </button>
        <button type="button" class="btn btn-primary">
            Save changes
        </button>
    </div>
</div>
```

## Кнопка вызова модального окна

Кнопка, открывающая модальное окно, связана с AJAX-запросом, который запрашивает частичное представление и загружает его содержимое в элемент с идентификатором `siteModalContent`.

```html
<button
    type="button"
    class="btn btn-primary"
    data-request="onAjax"
    data-request-update="{ 'my-modal-content': '#siteModalContent' }"
    data-bs-toggle="modal"
    data-bs-target="#siteModal">
    Launch demo modal
</button>
```

## Контейнер модального окна

Следующее определение модального окна универсально и может быть добавлено на любую страницу или в макет. Оно содержит два элемента `modal-dialog`. Первый используется как целевой контейнер для содержимого частичного представления, второй — для отображения состояния загрузки во время выполнения запроса.

```html
<div class="modal" id="siteModal">
    <div class="modal-dialog modal-dialog-centered" id="siteModalContent">
        <!-- Partial Contents Will Go Here -->
    </div>

    <div class="modal-dialog modal-dialog-centered modal-loading">
        <div class="spinner-border text-light mx-auto"></div>
    </div>
</div>
```

Для отображения статуса загрузки используется таблица стилей: она показывает окно загрузки во время AJAX-запроса, что определяется атрибутом `data-ajax-updating`. Атрибут добавляется элементу, когда он участвует в обновлении частичного представления и ожидает запрос.

```css
.modal-dialog[data-ajax-updating],
.modal-dialog:not([data-ajax-updating]) + .modal-loading {
    display: none;
}
```

#### См. также

::: also
* [Модальные окна Bootstrap 5](https://getbootstrap.com/docs/5.2/components/modal/)
:::
