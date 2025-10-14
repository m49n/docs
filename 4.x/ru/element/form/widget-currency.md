---
subtitle: Виджет формы
shortname: Currency
---
# Поле Currency

Виджет формы `currency` выводит поле для ввода числового значения валюты. Для этого поля используется основное определение валюты или настройка **Base Currency**, выбранная в разделе [Site Definition](../../cms/resources/multisite.md).

::: tip
Этот виджет появляется после установки [плагина Currency](https://octobercms.com/plugin/responsiv-currency), доступного на маркетплейсе October CMS. Установить его можно следующей командой.

```bash
php artisan plugin:install Responsiv.Currency
```
:::

Чтобы вывести поле ввода валюты, определите поле формы так:

```yaml
total_amount:
    label: Total amount
    type: currency
```

Свойство | Описание
------------- | -------------
**format** | необязательный формат при предпросмотре поля формы: `long`, `short` или `null`. Значение по умолчанию — `null`.

Используйте свойство `format`, чтобы изменить формат отображения поля формы в контексте предпросмотра.

```yaml
total_amount:
    label: Total amount
    type: currency
    format: short
```

#### См. также

::: also
* [Фильтр Currency в Twig](../../markup/filter/currency.md)
* [Столбец списка Currency](../../element/lists/column-currency.md)
* [Страница плагина Currency](https://octobercms.com/plugin/responsiv-currency)
:::
