---
subtitle: Фильтр Twig
---
# |page

Фильтр `|page` создаёт ссылку на страницу, принимая в параметре имя файла страницы без расширения. Например, если существует страница about.htm, можно использовать следующий код, чтобы сгенерировать ссылку на неё:

```twig
<a href="{{ 'about'|page }}">About Us</a>
```

Если страница находится в подкаталоге, указывай название подкаталога:

```html
<a href="{{ 'contacts/about'|page }}">About Us</a>
```

::: tip
Подробнее об использовании подкаталогов см. в [документации по темам](../../cms/themes/themes.md).
:::

Чтобы получить ссылку на конкретную страницу в PHP-разделе, можно использовать `$this->pageUrl('page-name-without-extension')`.

::: cmstemplate
```ini
```
```php
<?
function onStart()
{
    $this['newsPage'] = $this->pageUrl('blog/overview');
}
?>
```
```twig
{{ newsPage }}
```
:::

Создать ссылку на текущую страницу можно, отфильтровав переменную `this`.

```twig
<a href="{{ this|page }}">Refresh page</a>
```

Чтобы получить ссылку на текущую страницу в PHP, вызови метод `$this->pageUrl()` без аргументов.

::: cmstemplate
```ini
```
```php
<?
function onStart()
{
    $this['currentUrl'] = $this->pageUrl();
}
?>
```
```twig
{{ currentUrl }}
```
:::

## Обратный роутинг

При создании ссылки на страницу с параметрами URL фильтр `|page` поддерживает обратный роутинг: передай массив в качестве первого аргумента.

::: cmstemplate
```ini
url = "/blog/post/:post_id"
```
```twig
[...]
```
:::

Если приведённое содержимое находится в файле CMS-страницы **post.htm**, можно сослаться на эту страницу так:

```twig
<a href="{{ 'post'|page({ post_id: 10 }) }}">
    Blog post #10
</a>
```

Если адрес сайта — __https://octobercms.com__, приведённый пример выведет следующее:

```html
<a href="https://octobercms.com/blog/post/10">
    Blog post #10
</a>
```

## Постоянные параметры URL

Если параметр URL уже присутствует в окружении, фильтр `|page` использует его автоматически.

```ini
url = "/blog/post/:post_id"

url = "/blog/post/edit/:post_id"
```

Если есть две страницы, **post.htm** и **post-edit.htm**, с указанными выше URL, можно ссылаться на любую из них, не задавая параметр `post_id`.

```twig
<a href="{{ 'post-edit'|page }}">
    Edit this post
</a>
```

Когда эта разметка размещена на странице **post.htm**, она выведет следующее:

```html
<a href="https://octobercms.com/blog/post/edit/10">
    Edit this post
</a>
```

Значение `post_id` равно *10* уже известно и сохраняется в окружении. Можно отключить эту функциональность, передав вторым аргументом `false`:

```twig
<a href="{{ 'post'|page(false) }}">
    Unknown blog post
</a>
```

Или указав другое значение:

```twig
<a href="{{ 'post'|page({ post_id: 6 }) }}">
    Blog post #6
</a>
```
