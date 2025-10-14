---
subtitle: Колонка списка
shortname: Number
---
# Колонка Number

`number` — отображает числовую колонку с выравниванием по правому краю.

```yaml
age:
    label: Age
    type: number
```

Можно указать собственный формат числа, например валюту **$99.00**.

```yaml
price:
    label: Price
    type: number
    format: "$%.2f"
```

::: tip
Свойство `format` использует правила форматирования функции [PHP `sprintf()`](https://secure.php.net/manual/en/function.sprintf.php).
:::

## Подсчёт связей

Тип `number` часто используют вместе со свойством `relationCount`, чтобы подсчитать количество связанных записей. Следующая конфигурация считает записи связи **users**.

```yaml
users_count:
    label: Users
    type: number
    relation: users
    relationCount: true
```
