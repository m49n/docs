# Парсер

October CMS использует несколько стандартов для обработки разметки, шаблонов и конфигураций. Каждый из них подобран так, чтобы упростить процесс разработки и обучение. Например, [объекты темы](../../cms/themes/themes.md) используют Twig и формат INI в структуре шаблонов. Ниже подробно описаны все парсеры.

## Парсер Markdown

Markdown позволяет писать текст в простом для чтения и создания формате, который затем преобразуется в HTML. Фасад (facade) `Markdown` используется для разбора синтаксиса Markdown и основан на [GitHub Flavored Markdown](https://help.github.com/articles/github-flavored-markdown/). Несколько быстрых примеров Markdown:

```md
This text is **bold**, this text is *italic*, this text is ~~crossed out~~.

# The largest heading (an <h1> tag)
## The second largest heading (an <h2> tag)
...
###### The 6th largest heading (an <h6> tag)
```

Используйте метод `Markdown::parse`, чтобы отрендерить Markdown в HTML:

```php
$html = Markdown::parse($markdown);
```

Также можно применять фильтр `|md` для [разбора Markdown во фронтенд-разметке](../../markup/filter/md.md).

```twig
{{ '**Text** is bold.'|md }}
```

### Использование HTML в Markdown

Markdown — надмножество HTML, поэтому HTML и Markdown можно объединять в одном шаблоне. Когда Markdown встречает любой блочный HTML-тег, синтаксис Markdown отключается для всего содержимого внутри него.

```html
<div>
    This **text** won't be parsed by *Markdown*
</div>
```

Важно, что парсер Markdown принимает только один HTML-узел в строке. В примере ниже второй узел не попадёт в вывод.

```html
<!-- Output: <p>Foo</p> -->
<p>Foo</p><p>Bar</p>
```

При выводе сложного HTML, особенно через переменную Twig, следует оборачивать переменную в один HTML-узел, чтобы всё содержимое было выведено.

```twig
<div>
    {{ messageBody|raw }}
</div>
```

Если требуется намеренно включить Markdown внутри блочного тега, добавьте атрибут `markdown` со значением `1`.

```html
<div markdown="1">
    This **text** is now bold.
</div>
```

## Парсер шаблонов Twig

Twig — это простой, но мощный шаблонизатор, который преобразует HTML-шаблоны в оптимизированный PHP-код. Он лежит в основе [фронтенд-разметки](../../markup/templating.md), [содержимого представлений](./response-view.md) и [содержимого почтовых сообщений](../system/sending-mail.md).

Фасад `Twig` используется для разбора синтаксиса Twig, метод `Twig::parse` рендерит Twig в HTML.

```php
$html = Twig::parse($twig);
```

Во втором аргументе можно передать переменные для Twig-разметки.

```php
$html = Twig::parse($twig, ['foo' => 'bar']);
```

Парсер Twig можно расширить и зарегистрировать пользовательские возможности через [файл регистрации плагина](../twig-tags.md).

## Парсер скобок

October CMS также содержит простой шаблонизатор на основе фигурных скобок в качестве альтернативы парсеру Twig, который сейчас применяется для передачи переменных в [контент-блоки темы](../../cms/themes/content.md). Этот движок быстрее рендерит HTML и больше подходит для нетехнических пользователей. Для этого парсера не предусмотрен фасад, поэтому следует использовать полностью квалифицированный класс `October\Rain\Parse\Bracket` и его метод `parse`.

```php
use October\Rain\Parse\Bracket;

$html = Bracket::parse($content, ['foo' => 'bar']);
```

Синтаксис использует одиночные *фигурные скобки* для вывода переменных:

```
<p>Hello there, {foo}</p>
```

Можно передать массив объектов для разбора в качестве переменной.

```php
$html = Template::parse($content, ['likes' => [
    ['name' => 'Dogs'],
    ['name' => 'Fishing'],
    ['name' => 'Golf']
]]);
```

Массив можно перебирать с помощью следующего синтаксиса:

```
<ul>
    {likes}
        <li>{name}</li>
    {/likes}
</ul>
```

## Парсер конфигурации YAML

YAML («YAML Ain't Markup Language») — это конфигурационный формат. Как и Markdown, он создан простым для чтения и написания и преобразуется в массив PHP. Он используется практически везде при разработке бэкенда October CMS, например, для [описания полей форм](../../element/form-fields.md). Пример YAML:

```yaml
receipt: Acme Purchase Invoice
date: 2015-10-02
user:
    name: Joe
    surname: Blogs
```

Фасад `Yaml` используется для разбора YAML. Метод `Yaml::parse` преобразует YAML в массив PHP:

```php
$array = Yaml::parse($yamlString);
```

Содержимое файла можно разобрать методом `parseFile`:

```php
$array = Yaml::parseFile($filePath);
```

Парсер также работает в обратном направлении, создавая YAML из массива PHP. Для этого используйте метод `render`:

```php
$yamlString = Yaml::render($array);
```

## Парсер конфигурации Initialization (INI)

Формат INI — стандарт для простых конфигурационных файлов, часто применяемый [компонентами в шаблонах темы](../../cms/themes/components.md). Его можно считать родственником YAML, но в отличие от YAML он крайне прост, менее чувствителен к опечаткам и не зависит от отступов. Он поддерживает пары ключ—значение и секции, например:

```ini
receipt = "Acme Purchase Invoice"
date = "2015-10-02"

[user]
name = "Joe"
surname = "Blogs"
```

Фасад `Ini` используется для разбора INI. Метод `Ini::parse` преобразует INI в массив PHP:

```php
$array = Ini::parse($iniString);
```

Содержимое файла можно разобрать методом `parseFile`:

```php
$array = Ini::parseFile($filePath);
```

Парсер также поддерживает обратную операцию, формируя INI из массива PHP. Для этого используйте метод `render`:

```php
$iniString = Ini::render($array);
```

### INI со спецификой October

Обычно стандартный парсер INI, используемый функцией PHP `parse_ini_string`, ограничивается массивами глубиной до трёх уровней. Например:

```ini
level1Value = "foo"
level1Array[] = "bar"

[level1Object]
level2Value = "hello"
level2Array[] = "world"
level2Object[level3Value] = "stop here"
```

October расширяет эту функциональность с помощью *INI со спецификой October*, позволяя создавать массивы бесконечной вложенности, вдохновлённые синтаксисом HTML-форм. Продолжая пример выше, поддерживается следующий синтаксис:

```ini
[level1Object]
level2Object[level3Array][] = "Yay!"
level2Object[level3Object][level4Value] = "Yay!"
level2Object[level3Object][level4Array][] = "Yay!"
level2Object[level3Object][level4Object][level5Value] = "Yay!"
; ... to infinity and beyond!
```

## Парсер динамического синтаксиса

Dynamic Syntax — это уникальный для October шаблонизатор, который в корне поддерживает два режима рендеринга. Разбор шаблона возвращает два результата: **режим просмотра** (view) и **режим редактора** (editor). Рассмотрим пример текста шаблона: внутреннее содержимое тега `{text}...{/text}` — это значение по умолчанию для режима просмотра, а атрибуты `name` и `label` используются как свойства для режима редактора.

```
<h1>{text name="websiteName" label="Website Name"}Our wonderful website{/text}</h1>
```

Для этого парсера не предусмотрен фасад, поэтому следует использовать полностью квалифицированный класс `October\Rain\Parse\Syntax\Parser` и его метод `parse`. Первый аргумент метода `parse` принимает содержимое шаблона строкой и возвращает объект `Parser`.

```php
use October\Rain\Parse\Syntax\Parser as SyntaxParser;

$syntax = SyntaxParser::parse($content);
```

### Режим просмотра

Если использовать пример выше в качестве содержимого шаблона, вызов метода `render` без аргументов отрендерит шаблон со значением по умолчанию:

```php
echo $syntax->render();
// <h1>Our wonderful website</h1>
```

Как и в любом шаблонизаторе, передача массива переменных в первый аргумент `render` заменит переменные внутри шаблона. Здесь значение по умолчанию `websiteName` заменяется на новое значение:

```php
echo $syntax->render(['websiteName' => 'October CMS']);
// <h1>October CMS</h1>
```

Дополнительно вызов метода `toTwig` вернёт шаблон, подготовленный для рендеринга движком Twig.

```php
echo $syntax->toTwig();
// <h1>{{ websiteName }}</h1>
```

### Режим редактора

Пока что парсер Dynamic Syntax мало отличается от обычного шаблонизатора, однако режим редактора раскрывает его преимущества. Режим редактора открывает новые возможности, например, когда [макеты добавляют кастомные поля форм на страницы](https://octobercms.com/plugin/rainlab-pages), которым они принадлежат, или для [динамически создаваемых форм в e-mail-кампаниях](https://octobercms.com/plugin/responsiv-campaign).

Продолжая примеры выше, вызов метода `toEditor` у объекта `Parser` вернёт массив свойств, описывающих, как переменная должна заполняться, например конструктором форм.

```php
$array = $syntax->toEditor();
// 'websiteName' => [
//     'label' => 'Website name',
//     'default' => 'Our wonderful website',
//     'type' => 'text'
// ]
```

Обратите внимание, что свойства похожи на параметры из [определений полей формы](../../element/form-fields.md). Это сделано намеренно, чтобы возможности дополняли друг друга. Теперь можно легко преобразовать массив в YAML и записать в файл `fields.yaml`:

```php
$form = [
    'fields' => $syntax->toEditor()
];

File::put('fields.yaml', Yaml::render($form));
```

### Поддерживаемые теги

Парсер Dynamic Syntax поддерживает разные типы тегов, соответствующие распространённым [типам полей формы](../../element/form-fields.md).

#### Text

Однострочное поле для коротких текстовых блоков.

```html
{text name="websiteName" label="Website Name"}Our wonderful website{/text}
```

#### Textarea

Многострочное поле для больших текстовых блоков.

```html
{textarea name="websiteDescription" label="Website Description"}
    This is our vision for things to come
{/textarea}
```

#### Dropdown

Рендерит поле формы с выпадающим списком.

```html
{dropdown name="dropdown" label="Pick one" options="One|Two"}{/dropdown}
```

Рендерит поле формы с выпадающим списком и независимыми значениями и подписями.

```html
{dropdown name="dropdown" label="Pick one" options="one:One|two:Two"}{/dropdown}
```

Рендерит поле формы с выпадающим списком и массивом, возвращаемым статическим методом класса (нужно указывать полное пространство имён).

```html
{dropdown name="dropdown" label="Pick one" options="Path\\To\\Class::method"}{/dropdown}
```

#### Radio

Рендерит переключатели (radio).

```html
{radio name="radio" label="Thoughts?" options="y:Yes|n:No|m:Maybe"}{/radio}
```

#### Variable

Рендерит поле формы ровно того типа, который указан в атрибуте `type`. Этот тег просто задаёт переменную и в режиме просмотра выводит пустую строку.

```html
{variable type="text" name="name" label="Name"}John{/variable}
```

#### Rich editor

Текстовое поле для богатого содержимого (WYSIWYG).

```html
{richeditor name="content" label="Main content"}Default text{/richeditor}
```

В Twig отображается как

```twig
{{ content|raw }}
```

#### Markdown

Поле для ввода содержимого в формате Markdown.

```html
{markdown name="content" label="Markdown content"}Default text{/markdown}
```

В Twig отображается как

```twig
{{ content|md }}
```

#### Media finder

Выборщик файлов из медиатеки. Значение тега содержит относительный путь к файлу.

```html
{mediafinder name="logo" label="Logo"}defaultlogo.png{/mediafinder}
```

В Twig отображается как

```twig
{{ logo|media }}
```

#### File upload

Поле загрузки файлов. Значение тега содержит полный путь к файлу.

```html
{fileupload name="logo" label="Logo"}defaultlogo.png{/fileupload}
```

#### Color picker

Виджет выбора цвета. Значение тега содержит выбранное шестнадцатеричное значение. Дополнительно можно указать атрибут `availableColors`, чтобы задать доступные варианты.

```html
{colorpicker name="bg_color" label="Background colour" allowEmpty="true" availableColors="#ffffff|#000000"}{/colorpicker}
```

#### Repeater

Рендерит повторяющийся блок с другими полями внутри.

```html
{repeater name="content_sections" prompt="Add another content section"}
    <h2>{text name="title" label="Title"}Title{/text}</h2>
    <p>{textarea name="content" label="Content"}Content{/textarea}</p>
{/repeater}
```

В Twig отображается как

```twig
{% for fields in repeater %}
    <h2>{{ fields.title }}</h2>
    <p>{{ fields.content|raw }}</p>
{% endfor %}
```

Вызов `$syntax->toEditor` вернёт для поля повторителя другой массив:

```php
'repeater' => [
    'label' => 'Website name',
    'type' => 'repeater',
    'fields' => [

        'title' => [
            'label' => 'Title',
            'default' => 'Title',
            'type' => 'text'
        ],
        'content' => [
            'label' => 'Content',
            'default' => 'Content',
            'type' => 'textarea'
        ]

    ]
]
```

Поле повторителя также поддерживает режим групп, который используется с парсером динамического синтаксиса следующим образом:

```html
{variable name="sections" type="repeater" prompt="Add another section" tab="Sections"
        groups="$/author/plugin/repeater_fields.yaml"}{/variable}
```

Пример конфигурационного файла `repeater_fields.yaml` для групп повторителя:

```yaml
quote:
    name: Quote
    description: Quote item
    icon: icon-quote-right
    fields:
        quote_position:
            span: auto
            label: Quote Position
            type: radio
            options:
                left: Left
                center: Center
                right: Right
        quote_content:
            span: auto
            label: Details
            type: textarea
```

Подробнее о групповом режиме повторителя см. [виджет Repeater](../../element/form/widget-repeater.md).
