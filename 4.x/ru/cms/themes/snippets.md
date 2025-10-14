---
subtitle: Устраняйте разрыв между разработчиками и редакторами контента.
---
# Сниппеты

Сниппеты (snippets) — это блоки, которые вставляются в [визуальный редактор](../../element/form/widget-richeditor.md) или [редактор Markdown](../../element/form/widget-markdown.md) и настраиваются через [инструмент Inspector](../../element/inspector-types.md). Когда функция доступна, на панели инструментов отображается кнопка вставки сниппетов, а при выборе сниппета он вставляется в редактор.

Сниппеты можно определить как [частичные представления (partials)](./partials.md) или [компоненты (components)](./components.md), что позволяет разработчикам создавать переиспользуемые и настраиваемые фрагменты содержимого. Варианты применения сниппетов:

- Встроенные видео — вывод настроенного видео с YouTube или трансляции Twitch.
- Сниппет Google Maps — вывод карты с заданными координатами и заранее определённым масштабом, полезен для страниц с информацией о маршрутах.
- Универсальная система комментариев — позволяет посетителям оставлять комментарии на любой странице.
- Интеграции со сторонними сервисами — например, с Yelp или TripAdvisor для вывода дополнительной информации на странице.

При включении сниппетов в содержимое страницы требуется [фильтр Twig `|content`](../../markup/tag/content.md), чтобы обработать и отрендерить сниппеты в итоговом выводе.

```twig
{{ blog_html|content }}
```

## Создание сниппетов из частичных представлений

Сниппеты на основе частичных представлений реализуют простую функциональность и обычно являются контейнерами для HTML-разметки или разметки на Twig внутри сниппета.

Чтобы создать сниппет из частичного представления, откройте частичное представление в редакторе и нажмите кнопку **Snippet**. Затем укажите код сниппета, его название и описание в форме частичного представления.

Свойства сниппета необязательны и задаются через табличный контрол на форме настроек частичного представления. Таблица содержит следующие столбцы.

Column         | Description
-------------- | -----------
Property Title | заголовок свойства, отображаемый пользователю во всплывающем окне Inspector сниппета.
Code           | код свойства, используется для доступа к значению свойства в разметке частичного представления.
Type           | тип свойства, доступны `string`, `dropdown` и `checkbox`.
Default        | значение по умолчанию; для флажков используйте `0` и `1`.
Options        | список вариантов для свойств типа dropdown (см. ниже).

Любое свойство из списка доступно в Markdown частичного представления как обычная переменная, например:

```twig
The country name is {{ country }}
```

Кроме того, свойства можно передавать в компоненты частичного представления через [внешние значения свойств](../themes/components.md).

### Определение вариантов

При настройке **options** список должен иметь формат `key:Value | key2:Value`. Ключи представляют внутреннее значение варианта, а значения — строку, отображаемую пользователю в выпадающем списке. Отдельные варианты разделяются вертикальной чертой, например `us:US | ca:Canada`.

Ключ необязателен. Если его опустить (`US | Canada`), внутренние значения вариантов будут целыми числами, начиная с нуля (`0`, `1`, ...). Рекомендуется всегда задавать ключи явно. Ключи могут содержать только латинские буквы, цифры и символы `-` и `_`.

Также свойство **options** можно указать как ссылку на статический метод PHP-класса (`Class::method`).

## Создание сниппетов из компонентов

Любой [компонент CMS](./components.md) можно зарегистрировать как сниппет с помощью метода `registerPageSnippets` в классе плагина в [файле регистрации](../../extend/system/plugins.md). API регистрации сниппетов аналогичен [регистрации компонентов](../../extend/cms-components.md). Метод должен возвращать массив, где ключи — имена классов, а значения — псевдонимы.

```php
public function registerPageSnippets()
{
    return [
        \RainLab\Weather\Components\Weather::class => 'weather'
    ];
}
```

::: tip
Один и тот же компонент можно зарегистрировать через `registerPageSnippets` и `registerComponents`, чтобы использовать его и на CMS-страницах, и в редакторах контента.
:::

Чтобы разрешить использование AJAX-обработчиков, сниппеты можно рендерить через [AJAX-частичное представление](../../markup/tag/ajax-partial.md). Для этого установите `snippetAjax` в `true` в [определении класса компонента](../../extend/cms-components.md).

```php
public function componentDetails()
{
    return [
        // ...
        'snippetAjax' => true
    ];
}
```

## Примеры использования

Ниже приведены практические примеры применения сниппетов.

### Просмотр записи Tailor

Этот сниппет отображает краткую информацию о записи блога из записи Tailor. Он подключает [компонент section](../components/section.md) и устанавливает значение `value` из свойства сниппета `post_id`, используя внешние значения свойств.

Редактор задаёт **Blog Post ID** нужной записи блога, а сниппет выводит ссылку на запись в виде карточки.

::: cmstemplate
```ini
## partials/snippets/blog-post-reference.htm

[viewBag]
snippetCode = "blogPostReference"
snippetName = "Blog Post Reference"
snippetDescription = "Display a reference to a blog post"
snippetProperties[post_id][title] = "Blog Post ID"
snippetProperties[post_id][type] = "string"

[section post]
handle = "Blog\Post"
identifier = "id"
value = "{{ post_id }}"
```
```twig
{% if post is not empty %}
    <div class="card shadow-sm">
        <div class="card-body">
            <h4>{{ post.title }}</h4>
        </div>
        <div class="card-footer">
            <div class="d-flex justify-content-between align-items-center">
                <a href="{{ 'blog/post'|page({ slug: post.slug }) }}" class="stretched-link">
                    {{ post.categories.first.title|default('') }}
                </a>
                <small class="text-muted">{{ post.published_at_date|date('j M Y') }}</small>
            </div>
        </div>
    </div>
{% else %}
    <!-- Post Missing: Unable to Find an Entry -->
{% endif %}
```
:::

### Встраивание видео с YouTube

Этот сниппет реализует встраивание видео YouTube как [частичное представление CMS](./partials.md). В нём есть метод для извлечения кода YouTube из URL браузера и преобразования строки времени в секунды.

Редактор задаёт значения **Video URL** и **Start At**, а сниппет выводит стандартный код встраивания YouTube с элементом iframe.

::: cmstemplate
```ini
## partials/snippets/youtube-video.htm

[viewBag]
snippetCode = "youtubeVideo"
snippetName = "YouTube Video"
snippetDescription = "Embed a Youtube Video on the page"
snippetProperties[url][title] = "Video URL"
snippetProperties[url][type] = "string"
snippetProperties[start_at][title] = "Start At"
snippetProperties[start_at][type] = "string"
```
```php
// Converts https://www.youtube.com/watch?v=k_H2zJ7UZfs to k_H2zJ7UZfs
function urlToCode($link = '')
{
    $parts = parse_url($link);
    if (isset($parts['query'])) {
        parse_str($parts['query'], $qs);
        if (isset($qs['v'])){
            return $qs['v'];
        }
        elseif (isset($qs['vi'])){
            return $qs['vi'];
        }
    }
    if (isset($parts['path'])){
        $path = explode('/', trim($parts['path'], '/'));
        return $path[count($path)-1];
    }
    return null;
}

// Converts 15:00 to 900
function timeToSeconds($time = '')
{
    $parts = explode(':', $time);
    if (count($parts) === 3) {
        return $parts[0] * 3600 + $parts[1] * 60 + $parts[2];
    }
    elseif (count($parts) === 2) {
        return $parts[0] * 60 + $parts[1];
    }
    return $time ?: 0;
}
```
```twig
{% if url %}
    <iframe
        width="560"
        height="315"
        src="https://www.youtube.com/embed/{{ this.urlToCode(url) }}?start={{ this.timeToSeconds(start_at) }}"
        title="YouTube video player"
        frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
        allowfullscreen></iframe>
{% else %}
    <!-- Video URL Missing -->
{% endif %}
```
:::

### Простая контактная форма

Следующий сниппет выводит базовую контактную форму и показывает, как обработать отправку. Код [валидации формы](../../ajax/features/validation.md) и [отправки письма](../../extend/system/sending-mail.md) не включён.

У сниппета нет свойств: редактору достаточно добавить виджет на страницу, и он отобразит форму со сообщением об успехе. Атрибут `snippetAjax` установлен в `1`, чтобы включить использование AJAX-обработчиков.

::: cmstemplate
```ini
## partials/snippets/contact-form.htm

[viewBag]
snippetCode = "contactForm"
snippetName = "Contact Form"
snippetDescription = "Display a contact form"
snippetAjax = 1
```
```php
function onSubmitContact()
{
    $this['submitted'] = true;
}
```
```twig
{% if not submitted %}
    <h3>Tell us what you think!</h3>
    <form data-request="onSubmitContact" data-request-update="{ _self: true }">
        <div class="row">
            <div class="col-md-6">
                <div class="form-floating mb-3">
                    <input name="name" type="text" class="form-control">
                    <label>Name</label>
                </div>
            </div>
            <div class="col-md-6">
                <div class="form-floating mb-3">
                    <input name="email" type="email" class="form-control">
                    <label>Email Address</label>
                </div>
            </div>
        </div>
        <div class="mb-3 form-floating">
            <textarea class="form-control h-100"></textarea>
            <label>Message</label>
        </div>
        <div class="form-buttons d-flex pt-2">
            <div>
                <button type="submit" class="btn btn-primary btn-pill">Submit</button>
            </div>
        </div>
    </form>
{% else %}
    <div class="alert alert-success">
        Thanks for contacting us!
    </div>
{% endif %}
```
:::

#### См. также

::: also
* [Частичные представления CMS](./partials.md)
* [Разработка компонентов](../../extend/cms-components.md)
:::
