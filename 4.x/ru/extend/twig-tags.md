---
subtitle: Расширение Twig собственными фильтрами и функциями.
---
# Создание тегов Twig

Пользовательские фильтры и функции Twig регистрируются в CMS методом `registerMarkupTags` [регистрационного класса плагина](../extend/extending.md). Эта статья описывает регистрацию собственных фильтров и функций Twig, которые можно использовать в CMS и почтовых шаблонах.

```php
public function registerMarkupTags()
{
    return [
        'filters' => [
            // ...Filters defined here
        ],
        'functions' => [
            // ...Functions defined here
        ]
    ];
}
```

## Регистрация фильтра

Ключ `filters` в массиве регистрации предназначен для фильтров Twig. Например, фильтр `|plural` можно сопоставить с глобальной PHP-функцией `str_plural()` следующим образом.

```php
'filters' => [
    'plural' => 'str_plural'
]
```

Также можно указать локальный метод. Например, фильтр `|uppercase` можно сопоставить с методом `$this->makeAllCaps()`.

```php
'filters' => [
    'uppercase' => [$this, 'makeTextAllCaps']
]
```

После этого фильтр будет доступен в Twig.

```twig
{{ 'my text'|uppercase }}
```

## Регистрация функции

Аналогично фильтрам, ключ `functions` используется для создания функций Twig. Например, функцию можно сопоставить со статическим методом `Form::open()`.

```php
'functions' => [
    'form_open' => [Form::class, 'open']
]
```

Статические вызовы поддерживают и подстановку `*`. В этом примере вызов функции `url_foobar()` будет преобразован во вызов метода `Url::foobar` и т. д.

```php
'functions' => [
    'url_*' => [Url::class, '*'],
]
```

Можно передавать и замыкание в любой из определений.

```php
'functions' => [
    'hello_world' => function() { return 'Hello World!'; }
]
```

Теперь функция доступна в Twig следующим образом.

```twig
{{ hello_world() }}
```

## Экранированный вывод

Важно помнить, что пользовательские фильтры и функции Twig экранируются по умолчанию. Чтобы отключить экранирование, передайте в определении последним элементом массива значение `false`.

```php
public function registerMarkupTags()
{
    return [
        'functions' => [
            // Escaped Functions
            'input' => 'input',

            // Raw Functions
            'link_to' => ['link_to', false],

            // Escaped Classes
            'str_*' => [\Str::class, '*'],

            // Raw Classes
            'url_*' => [\Url::class, '*', false],
        ],
        'filters' => [
            // Escaped Filters
            'display_name' => [fn ($user) => $user->getDisplayName()],

            // Raw Filters
            'avatar_url' => [fn ($user) => $user->getAvatarUrl(), false],

            // Escaped Classes
            'str_*' => [\Str::class, '*'],

            // Raw Classes
            'url_*' => [\Url::class, '*', false],
        ]
    ];
}
```

## Расширенные параметры

Помимо передачи `false` в качестве последнего элемента можно использовать массив, чтобы задать [дополнительные параметры расширению Twig](https://twig.symfony.com/doc/3.x/advanced.html). Например, можно передавать в функцию объекты окружения и контекст.

```php
public function registerMarkupTags()
{
    return [
        'filters' => [
            // Unescaped
            'rot13' => ['str_rot13', ['is_safe' => ['html']]],

            // Has Environment
            'env_filter' => [fn ($env, $string) => '...', ['needs_environment' => true]],

            // Has Context
            'context_filter' => [fn ($context, $string) => '...', ['needs_context' => true]],

            // Has Both
            'context_both' => [fn ($env, $context, $string) => '...', [
                'needs_environment' => true,
                'needs_context' => true
            ]],
        ]
    ];
}
```

#### См. также

::: also
* [Extending Twig](https://twig.symfony.com/doc/3.x/advanced.html)
:::
