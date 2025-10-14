---
subtitle: Колонка списка
shortname: Switch
---
# Колонка Switch

`switch` — отображает состояние «вкл./выкл.» для булевых колонок.

```yaml
enabled:
    label: Enabled
    type: switch
```

Можно настроить подписи переключателя, передав массив в `options` с подписями для false и true.

```yaml
enabled:
    label: Enabled
    type: switch
    options:
        - Nope
        - Yeah
```
