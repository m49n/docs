---
subtitle: Тип инспектора
shortname: Text
---
# Тип инспектора Text

Тип инспектора `text` позволяет вводить многострочный текст во всплывающем окне. У редактора нет специфических параметров. Необязательный параметр `default` должен содержать строку.

```php
public function defineProperties()
{
    return [
        'description' => [
            'title' => 'Description',
            'type' => 'text',
            'default' => 'This is a default description'
        ]
    ];
}
```

Сгенерированное значение — строка, например:

```json
"description": "This is a description"
```

Чаще всего применяются следующие [параметры конфигурации](../inspector-types.md).

Property | Description
------------- | -------------
**title** | заголовок свойства.
**description** | краткое описание свойства, необязательно.
**default** | строка по умолчанию, необязательно.
