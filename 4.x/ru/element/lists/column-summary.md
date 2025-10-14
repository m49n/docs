---
subtitle: Колонка списка
shortname: Summary
---
# Колонка Summary

`summary` — формирует краткое значение, удаляет HTML и обрезает текст по ближайшей границе слова.

```yaml
html_content:
    label: Content
    type: summary
```

Длина по умолчанию — 40 символов. Её можно изменить параметром `limitChars`.

```yaml
html_content:
    label: Content
    type: summary
    limitChars: 100
```

Чтобы ограничить количество слов, используйте `limitWords`. Параметр `endChars` позволяет задать суффикс.

```yaml
html_content:
    label: Content
    type: summary
    limitWords: 10
    endChars: "..."
```
