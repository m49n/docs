---
subtitle: Фильтр Twig
---
# |default

Фильтр `|default` возвращает значение, переданное первым аргументом, если фильтруемое значение не определено или пусто. В противном случае возвращается исходное значение.

```twig
{{ variable|default('The variable is not defined') }}

{{ variable.foo|default('The foo property on variable is not defined') }}

{{ variable['foo']|default('The foo key in variable is not defined') }}

{{ ''|default('The variable is empty') }}
```

При использовании фильтра `default` в выражении, где переменные участвуют во вызовах методов, убедись, что фильтр применяется каждый раз, когда переменная может быть не определена:

```twig
{{ variable.method(foo|default('bar'))|default('bar') }}
```
