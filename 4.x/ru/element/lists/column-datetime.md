---
subtitle: Колонка списка
shortname: Date & Time
---
# Колонка Date & Time

`datetime` — отображает значение колонки в формате даты и времени. В следующем примере даты выводятся как **Thu, Dec 25, 1975 2:15 PM**.

```yaml
created_at:
    label: Date
    type: datetime
```

Можно указать собственный формат, например **Thursday 25th of December 1975 02:15:16 PM**.

```yaml
created_at:
    label: Date
    type: datetime
    format: l jS \of F Y h:i:s A
```

Значение автоматически конвертируется в часовой пояс, выбранный в бэкенде. Отключить конвертацию можно через параметр `useTimezone`.

```yaml
created_at:
    label: Date
    type: datetime
    useTimezone: false
```

::: tip
Параметр `useTimezone` также действует на другие типы, связанные с датой и временем: `date`, `time`, `timesince` и `timetense`.
:::

## Date

`date` — отображает значение колонки в формате даты **M j, Y**.

```yaml
created_at:
    label: Date
    type: date
```

Часовой пояс бэкенда по умолчанию не применяется. Если дата включает время, можно выполнить конвертацию через `useTimezone`.

```yaml
created_at:
    label: Date
    type: date
    useTimezone: true
```

::: tip
Колонки `date` и `time` не выполняют конвертацию часового пояса по умолчанию, поскольку для неё требуется и дата, и время.
:::

## Time

`time` — отображает значение колонки во времени формата **g:i A**.

```yaml
created_at:
    label: Date
    type: time
```

## Time Since

`timesince` — показывает человекочитаемую разницу между значением и текущим временем, например: **10 minutes ago**.

```yaml
created_at:
    label: Date
    type: timesince
```

## Time Tense

`timetense` — отображает 24‑часовое время и день, используя грамматическое время текущей даты. Примеры: **Today at 12:49**, **Yesterday at 4:00**, **18 Sep 2015 at 14:33**.

```yaml
created_at:
    label: Date
    type: timetense
```
