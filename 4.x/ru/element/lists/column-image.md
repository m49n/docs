---
subtitle: Колонка списка
shortname: Image
---
# Колонка Image

Колонка `image` отображает изображение с возможностью задать размеры вывода.

```yaml
avatar:
    label: Avatar
    type: image
```

Поддерживаются следующие свойства.

Property | Description
------------- | -------------
**sortable** | отключает сортировку колонки. Значение по умолчанию: `false`.
**width** | ширина миниатюры, необязательно.
**height** | высота миниатюры, необязательно.
**options** | параметры [ресайзера изображений](../../extend/services/resizer.md).
**limit** | максимальное количество отображаемых изображений. Значение по умолчанию: `3`.

Используйте свойство `sortable`, чтобы отключить сортировку.

```yaml
avatar:
    label: Avatar
    type: image
    sortable: false
```

Свойства `width` и `height` позволяют указать пользовательский размер изображения.

```yaml
avatar:
    label: Avatar
    type: image
    width: 150
    height: 150
```

Свойство `options` задаёт параметры ресайзера.

```yaml
avatar:
    label: Avatar
    type: image
    options:
        quality: 80
```

Подробнее о поддерживаемых параметрах см. в статье [о ресайзе изображений](../../extend/services/resizer.md).

#### См. также

::: also
* [Ресайзер изображений](../../extend/services/resizer.md)
:::
