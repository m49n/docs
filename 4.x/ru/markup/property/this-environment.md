---
subtitle: Свойство Twig
---
# this.environment

Получить текущий объект окружения можно через `this.environment`; он возвращает строку, ссылающуюся на [конфигурацию текущего окружения](../../setup/configuration.md).

## Пример

Следующий пример покажет баннер, если сайт работает в тестовом окружении:

```twig
{% if this.environment == 'test' %}

    <div class="banner">Test Environment</div>

{% endif %}
```
