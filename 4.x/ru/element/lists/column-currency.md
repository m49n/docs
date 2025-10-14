---
subtitle: Колонка списка
shortname: Currency
---
# Колонка Currency

`currency` — отображает значение в формате валюты.

```yaml
total_amount:
    label: Loan amount
    type: currency
```

::: tip
Эта колонка доступна после установки [плагина Currency](https://octobercms.com/plugin/responsiv-currency) из маркетплейса October CMS. Установить его можно командой:

```bash
php artisan plugin:install Responsiv.Currency
```
:::

Поддерживаются следующие свойства.

Property | Description
------------- | -------------
**format** | задаёт формат отображения. Поддерживаемые значения: `long`, `short`, `null`.
**fromCode** | исходный код валюты.
**toCode** | код валюты для отображения.
**site** | отображает валюту в контексте определения мультисайта. Значение по умолчанию: `false`.

Используйте свойство `format`, чтобы выводить значение в расширенном формате.

```yaml
total_amount:
    label: Loan amount
    type: currency
    format: long
```

Установите `site: true`, если значение модели хранится с использованием определения мультисайта. Это автоматически задаст значения `toCode` и `fromCode` из определения сайта.

```yaml
total_amount:
    label: Loan amount
    type: currency
    site: true
```

::: also
* [Twig-фильтр Currency](../../markup/filter/currency.md)
* [Виджет формы Currency](../../element/form/widget-currency.md)
* [Страница плагина Currency](https://octobercms.com/plugin/responsiv-currency)
:::
