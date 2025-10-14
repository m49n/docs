---
subtitle: Фильтр Twig
---
# |currency

Фильтр `|currency` используется для отображения значения в валюте.

```twig
{{ 100|currency }}
```

::: tip
Этот фильтр Twig доступен после установки [плагина Currency](https://octobercms.com/plugin/responsiv-currency) из маркетплейса October CMS. Установить его можно следующей командой.

```bash
php artisan plugin:install Responsiv.Currency
```
:::

Фильтр принимает аргумент `options` в виде массива с разными значениями.

Опция | Описание
------ | -----------
**to** | Конвертировать в указанный код валюты
**from** | Конвертировать из кода валюты
**format** | Формат отображения. Варианты: long, short, null.
**site** | Установите `true`, чтобы использовать коды валют из определения сайта. По умолчанию `false`.

Например, чтобы конвертировать сумму из USD в AUD:

```php
{{ 1000|currency({ from: 'USD', to: 'AUD' }) }}
```

Чтобы использовать базовую и отображаемую валюту из определения сайта, задайте опцию **site** в `true`.

```php
{{ 1000|currency({ site: true }) }}
```

Чтобы вывести валюту в формате `long` или `short`:

```php
// $10.00
{{ 1000|currency({ format: '' }) }}

// $10.00 USD
{{ 1000|currency({ format: 'long' }) }}

// $10
{{ 1000|currency({ format: 'short' }) }}
```

## Интерфейс PHP

Работать с валютой можно через глобальный фасад `Currency`.

Например, чтобы конвертировать сумму из USD в AUD:

```php
Currency::format(1000, ['from' => 'USD', 'to' => 'AUD']);
```

Чтобы вывести валюту в длинном или коротком формате:

```php
// $10.00 USD
Currency::format(1000, ['format' => 'long']);

// $10
Currency::format(1000, ['format' => 'short']);
```

#### См. также

::: also
* [Виджет формы Currency](../../element/form/widget-currency.md)
* [Колонка списка Currency](../../element/lists/column-currency.md)
* [Страница плагина Currency](https://octobercms.com/plugin/responsiv-currency)
:::
