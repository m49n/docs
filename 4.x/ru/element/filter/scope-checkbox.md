---
subtitle: Область фильтра
shortname: Checkbox
---
# Область Checkbox

`checkbox` — бинарный чекбокс, который применяет к списку заранее заданное условие или запрос в состояниях «включено» либо «выключено». Значение по умолчанию: 0 — выключено, 1 — включено.

```yaml
is_published:
    label: Hide Published
    type: checkbox
    conditions: is_published <> true
```

Поддерживаются следующие свойства фильтра.

Property | Description
------------- | -------------
**default** | установить `true`, чтобы фильтр был включён по умолчанию. Значение по умолчанию: `false`.
**conditions** | произвольный SQL‑запрос, используемый фильтром.

Можно указать значение `default`, чтобы фильтр был включён по умолчанию.

```yaml
is_published:
    label: Hide Published
    type: checkbox
    default: 1
```
