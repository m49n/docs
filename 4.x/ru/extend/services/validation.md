# Validation

Класс валидатора предоставляет простой и удобный способ проверять данные и получать сообщения об ошибках через класс `Validator`. Он полезен при обработке данных форм, отправляемых пользователем.

::: tip
При работе с моделями October CMS поставляется с полезным [поведением Validation](../database/traits.md), которое реализует класс `Validator` и поддерживает те же определения правил.
:::

Ниже перечислены все доступные правила валидации:

<div class="content-list" markdown="1">

- [Accepted](#rule-accepted)
- [Active URL](#rule-active-url)
- [After (Date)](#rule-after)
- [Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alpha Numeric](#rule-alpha-num)
- [Array](#rule-array)
- [Before (Date)](#rule-before)
- [Between](#rule-between)
- [Boolean](#rule-boolean)
- [Confirmed](#rule-confirmed)
- [Date](#rule-date)
- [Date Format](#rule-date-format)
- [Different](#rule-different)
- [Digits](#rule-digits)
- [Digits Between](#rule-digits-between)
- [E-Mail](#rule-email)
- [Exists (Database)](#rule-exists)
- [Image (File)](#rule-image)
- [In](#rule-in)
- [Integer](#rule-integer)
- [IP Address](#rule-ip)
- [Max](#rule-max)
- [MIME Types](#rule-mimes)
- [Min](#rule-min)
- [Not In](#rule-not-in)
- [Nullable](#rule-nullable)
- [Numeric](#rule-numeric)
- [Regular Expression](#rule-regex)
- [Required](#rule-required)
- [Required If](#rule-required-if)
- [Required Unless](#rule-required-unless)
- [Required With](#rule-required-with)
- [Required With All](#rule-required-with-all)
- [Required Without](#rule-required-without)
- [Required Without All](#rule-required-without-all)
- [Same](#rule-same)
- [Size](#rule-size)
- [String](#rule-string)
- [Timezone](#rule-timezone)
- [Unique (Database)](#rule-unique)
- [Site Unique (Database)](#rule-site-unique)
- [URL](#rule-url)

</div>

## Базовое использование

Обычно сначала получают пользовательский ввод и передают его в метод `make` (первый аргумент), а также указывают правила валидации (второй аргумент). В следующем примере данные POST получаются вспомогательной функцией `post()`.

```php
$data = post();

$validator = Validator::make($data, [
    'name' => 'required|min:5'
]);
```

Несколько правил можно разделять символом «вертикальная черта» или перечислять отдельными элементами массива.

```php
$validator = Validator::make($data, [
    'name' => ['required', 'min:5']
]);
```

Чтобы проверять несколько полей, добавьте их в массив.

```php
$data = [
    'name' => 'Joe',
    'password' => 'lamepassword',
    'email' => 'email@example.tld'
];

$validator = Validator::make($data, [
    'name' => 'required',
    'password' => 'required|min:8',
    'email' => 'required|email|unique:users'
]);
```

### Проверка результатов валидации

После создания экземпляра `Validator` проверка запускается методами `fails` или `passes`.

```php
if ($validator->fails()) {
    // Переданные данные не прошли валидацию
}
```

Если проверка не прошла, можно получить сообщения об ошибках у валидатора.

```php
$messages = $validator->messages();
```

Также доступен массив правил, по которым валидация завершилась неудачно, без сообщений. Его возвращает метод `failed`.

```php
$failed = $validator->failed();
```

### Проверка файлов

Класс `Validator` содержит несколько правил для проверки файлов, например `size`, `mimes` и другие. При проверке файлы передаются вместе с остальными данными.

```php
$data = files() + post();

$validator = Validator::make($data, [...]);
```

::: warning
Не рекомендуется использовать нефильтрованные значения `input()`, поскольку они содержат GET-параметры, которые можно использовать для потенциально вредоносных ссылок.
:::

## Генерация исключения валидации

Чаще всего валидация применяется к данным, отправленным формой, и при ошибке удобно выбрасывать `ValidationException`. Чтобы сократить запись, можно воспользоваться методом `validate`.

```php
$data = Validator::validate($data, $rules);
```

::: tip
Метод `validate` возвращает отфильтрованные пользовательские данные — атрибуты и значения, прошедшие проверку.
:::

Этот метод эквивалентен следующему коду. Здесь также показано, как передать экземпляр валидатора напрямую в [исключение валидации](../system/exceptions.md) (первый аргумент).

```php
$validation = Validator::make($data, $rules);

if ($validation->fails()) {
    throw new ValidationException($validation);
}
```

### Проверка запроса

Ещё один способ — использовать фасад `Request` и выполнить проверку всех входных данных. В этом случае передаются только правила (первый аргумент). Метод `validate` возвращает отфильтрованные данные — атрибуты и значения, прошедшие проверку.

```php
$data = Request::validate($rules);
```

Метод `validate` возвращает результат, отфильтрованный правилами. Поле, не указанное в правилах, в набор данных не попадёт.

## Работа с сообщениями об ошибках

Метод `messages`, вызванный у экземпляра `Validator`, возвращает `Illuminate\Support\MessageBag`, предоставляющий удобные методы работы с сообщениями об ошибках.

#### Получение первого сообщения об ошибке для поля

```php
echo $messages->first('email');
```

#### Получение всех сообщений об ошибках для поля

```php
foreach ($messages->get('email') as $message) {
    //
}
```

#### Получение всех сообщений об ошибках для всех полей

```php
foreach ($messages->all() as $message) {
    //
}
```

#### Проверка наличия сообщений об ошибках для поля

```php
if ($messages->has('email')) {
    //
}
```

#### Получение сообщения об ошибке с форматированием

```php
echo $messages->first('email', '<p>:message</p>');
```

> **Примечание.** По умолчанию сообщения форматируются в синтаксисе, совместимом с Bootstrap.

#### Получение всех сообщений об ошибках с форматированием

```php
foreach ($messages->all('<li>:message</li>') as $message) {
    //
}
```

## Сообщения об ошибках и представления

После выполнения валидации необходимо удобно передать сообщения об ошибках в представления. В October CMS это реализовано автоматически. Рассмотрим следующий обработчик AJAX:

```php
public function onRegister()
{
    $rules = [];

    $validator = Validator::make(input(), $rules);

    if ($validator->fails()) {
        return Redirect::to('register')->withErrors($validator);
    }
}
```

Если проверка не прошла, экземпляр `Validator` передаётся в редирект методом `withErrors`. Этот метод сохраняет сообщения об ошибках во flash-данных сессии, делая их доступными при следующем запросе.

October CMS всегда проверяет наличие ошибок в данных сессии и автоматически привязывает их к представлению, если они существуют. **Поэтому переменная `errors` доступна на всех страницах и при каждом запросе**, что позволяет уверенно предполагать её наличие. `errors` — это экземпляр `MessageBag`.

После перенаправления можно использовать автоматически привязанную переменную `errors` в представлении:

```twig
{{ errors.first('email') }}
```

### Именованные наборы ошибок

Если на одной странице несколько форм, можно присвоить набору ошибок `MessageBag` имя. Это позволит получать сообщения ошибок для конкретной формы. Передайте имя вторым аргументом метода `withErrors`.

```php
return Redirect::to('register')->withErrors($validator, 'login');
```

После этого доступ к именованному экземпляру `MessageBag` выполняется через переменную `$errors`:

```twig
{{ errors.login.first('email') }}
```

## Доступные правила валидации

Ниже приведены правила валидации и их назначение.

<a name="rule-accepted"></a>
#### accepted

Поле должно содержать значение _yes_, _on_ или _1_. Правило полезно для проверки принятия условий использования.

<a name="rule-active-url"></a>
#### active_url

Поле должно содержать корректный URL по результатам функции PHP `checkdnsrr`.

<a name="rule-after"></a>
#### after:_date_

Поле должно содержать значение позже указанной даты. Даты передаются в функцию PHP `strtotime`.

<a name="rule-alpha"></a>
#### alpha

Поле должно состоять только из буквенных символов.

<a name="rule-alpha-dash"></a>
#### alpha_dash

Поле может содержать буквенно-цифровые символы, а также дефисы и подчёркивания.

<a name="rule-alpha-num"></a>
#### alpha_num

Поле должно состоять только из буквенно-цифровых символов.

<a name="rule-array"></a>
#### array

Поле должно быть массивом.

<a name="rule-before"></a>
#### before:_date_

Поле должно содержать значение, предшествующее указанной дате. Даты передаются в функцию PHP `strtotime`.

<a name="rule-between"></a>
#### between:_min_,_max_

Поле должно иметь размер между значениями _min_ и _max_. Строки, числа и файлы проверяются так же, как в правиле `size`.

<a name="rule-boolean"></a>
#### boolean

Поле должно приводиться к булеву типу. Допустимые значения: `true`, `false`, `1`, `0`, `"1"` и `"0"`.

<a name="rule-confirmed"></a>
#### confirmed

Поле должно иметь совпадающее поле `foo_confirmation`. Например, для поля `password` требуется наличие поля `password_confirmation`.

<a name="rule-date"></a>
#### date

Поле должно содержать корректную дату согласно функции PHP `strtotime`.

<a name="rule-date-format"></a>
#### date_format:_format_

Поле должно соответствовать формату _format_ по результатам функции PHP `date_parse_from_format`.

<a name="rule-different"></a>
#### different:_field_

Указанное поле _field_ должно отличаться от проверяемого поля.

<a name="rule-digits"></a>
#### digits:_value_

Поле должно быть числовым и иметь точную длину _value_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

Поле должно иметь длину между значениями _min_ и _max_.

<a name="rule-email"></a>
#### email

Поле должно быть отформатировано как адрес электронной почты.

<a name="rule-exists"></a>
#### exists:_table_,_column_

Поле должно существовать в указанной таблице базы данных.

Базовое использование правила exists

```php
'state' => 'exists:states'
```

Указание собственного имени столбца

```php
'state' => 'exists:states,abbreviation'
```

Можно добавить дополнительные условия, которые будут преобразованы в предложения "where" запроса:

```php
'email' => 'exists:staff,email,account_id,1'
```

Передача `NULL` в качестве значения "where" добавляет проверку на `NULL` в базе данных:

```php
'email' => 'exists:staff,email,deleted_at,NULL'
```

<a name="rule-image"></a>
#### image

Файл должен быть изображением (jpeg, png, bmp или gif).

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

Поле должно принимать одно из значений указанного списка.

<a name="rule-integer"></a>
#### integer

Поле должно содержать целочисленное значение.

<a name="rule-ip"></a>
#### ip

Поле должно быть отформатировано как IP-адрес.

<a name="rule-max"></a>
#### max:_value_

Поле должно быть меньше или равно значению _value_. Строки, числа и файлы проверяются так же, как в правиле [`size`](#rule-size).

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

Файл должен иметь MIME-тип, соответствующий одному из перечисленных расширений.

#### Базовое использование правила MIME

```php
'photo' => 'mimes:jpeg,bmp,png'
```

<a name="rule-min"></a>
#### min:_value_

Поле должно иметь минимальное значение _value_. Строки, числа и файлы проверяются так же, как в правиле [`size`](#rule-size).

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

Поле не должно входить в указанный список значений.

<a name="rule-nullable"></a>
#### nullable

Поле может иметь значение `null`. Это особенно полезно при проверке примитивных типов, таких как строки и целые числа, которые могут принимать `null`.

<a name="rule-numeric"></a>
#### numeric

Поле должно содержать числовое значение.

<a name="rule-regex"></a>
#### regex:_pattern_

Поле должно соответствовать указанному регулярному выражению.

**Примечание.** При использовании правила `regex` иногда следует указывать правила массивом, а не через разделитель `|`, особенно если регулярное выражение содержит символ вертикальной черты.

<a name="rule-required"></a>
#### required

Поле должно присутствовать во входных данных.

<a name="rule-required-if"></a>
#### required_if:_field_,_value_,...

Поле должно присутствовать, если поле _field_ равно любому из значений _value_.

<a name="rule-required-unless"></a>
#### required_unless:anotherfield,value,...

Поле должно присутствовать и не быть пустым, если поле anotherfield не равно ни одному из значений value.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

Поле должно присутствовать **только если** присутствует хотя бы одно из указанных полей.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

Поле должно присутствовать **только если** присутствуют все указанные поля.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

Поле должно присутствовать **только когда** отсутствует хотя бы одно из указанных полей.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

Поле должно присутствовать **только когда** отсутствуют все указанные поля.

<a name="rule-same"></a>
#### same:_field_

Значение указанного поля _field_ должно совпадать со значением проверяемого поля.

<a name="rule-size"></a>
#### size:_value_

Поле должно иметь размер, равный значению _value_. Для строк _value_ — количество символов. Для чисел — конкретное числовое значение. Для файлов размер указывается в килобайтах.

<a name="rule-string"></a>
#### string:_value_

Поле должно быть строкой.

<a name="rule-timezone"></a>
#### timezone

Поле должно содержать корректный идентификатор часового пояса согласно функции PHP `timezone_identifiers_list`.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

Поле должно быть уникальным в указанной таблице базы данных. Если параметр `column` не задан, используется имя поля.

Базовое использование правила unique.

```php
'email' => 'unique:users'
```

Указание собственного имени столбца.

```php
'email' => 'unique:users,email_address'
```

Игнорирование конкретного идентификатора.

```php
'email' => 'unique:users,email_address,10'
```

Добавление дополнительных условий.

Можно указать дополнительные условия, которые будут преобразованы в предложения "where" запроса:

```php
'email' => 'unique:users,email_address,NULL,id,account_id,1'
```

В приведённом правиле в проверку уникальности будут включены только строки с `account_id`, равным `1`.

<a name="rule-site-unique"></a>
#### unique_site:_table_,_column_,_except_,_idColumn_

Поле должно быть уникальным в рамках [контекста сайта](../../cms/resources/multisite.md). Определение полностью совпадает с правилом `unique`.

```php
'email' => 'unique_site:users'
```

<a name="rule-url"></a>
#### url

Поле должно быть отформатировано как URL.

::: tip
Это правило использует функцию PHP `filter_var`.
:::

## Условное добавление правил

Иногда требуется выполнять проверку поля **только** если оно присутствует во входном массиве. Для этого добавьте правило `sometimes` в список правил:

```php
$v = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```

В этом примере поле `email` будет проверяться, только если присутствует в массиве `$data`.

#### Сложная условная валидация

Иногда нужно требовать поле только в том случае, если значение другого поля превышает 100. Или, возможно, два поля должны принимать определённые значения, только когда существует третье поле. Добавлять такие правила просто. Сначала создайте экземпляр `Validator` со _статичными правилами_, которые не меняются:

```php
$v = Validator::make($data, [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);
```

Предположим, веб-приложение предназначено для коллекционеров игр. Если пользователь указывает, что у него более 100 игр, нужно объяснить причину: возможно, он управляет магазином или просто любит коллекционировать. Чтобы условно добавить это требование, используйте метод `sometimes` экземпляра `Validator`.

```php
$v->sometimes('reason', 'required|max:500', function($input) {
    return $input->games >= 100;
});
```

Первый аргумент метода `sometimes` — имя поля, которое проверяется условно. Второй аргумент — правила, которые нужно добавить. Если замыкание, переданное третьим аргументом, возвращает `true`, правило будет применено. Метод позволяет без труда строить сложные условные проверки. Можно добавлять условия сразу для нескольких полей:

```php
$v->sometimes(['reason', 'cost'], 'required', function($input) {
    return $input->games >= 100;
});
```

::: tip
Параметр `$input`, передаваемый в замыкание, является экземпляром `Illuminate\Support\Fluent`, поэтому к данным и файлам можно обращаться как к свойствам объекта.
:::

## Проверка массивов

Поля форм, основанные на массивах, также можно проверять без лишних усилий. Используйте «точечную нотацию» для проверки атрибутов в массиве. Например, если входящий HTTP-запрос содержит поле `photos[profile]`, проверить его можно так:

```php
$validator = Validator::make(input(), [
    'photos.profile' => 'required|image',
]);
```

Можно проверять и каждый элемент массива. Например, чтобы убедиться, что каждый адрес электронной почты в массиве уникален, выполните:

```php
$validator = Validator::make(input(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

Аналогично, при указании сообщений об ошибках в языковых файлах можно использовать символ `*`, чтобы одно сообщение применялось к полям-массивам:

```php
'custom' => [
    'person.*.email' => [
        'unique' => 'Each person must have a unique e-mail address',
    ]
],
```

Можно использовать и «массивную нотацию» в правилах. При проверке они автоматически преобразуются в «точечную нотацию».

```php
$validator = Validator::make(input(), [
    'photos[profile]' => 'required|image',
    'person[][email]' => 'email|unique:users',
]);
```

## Пользовательские сообщения об ошибках

При необходимости вместо сообщений по умолчанию можно использовать собственные. Есть несколько способов задать их. Ниже показано, как передать сообщения экземпляру валидатора.

```php
$messages = [
    'required' => 'The :attribute field is required.',
];

$validator = Validator::make($input, $rules, $messages);
```

Заполнитель `:attribute` будет заменён фактическим именем проверяемого поля. Можно использовать и другие заполнители. Ниже приведены дополнительные примеры.

```php
$messages = [
    'same' => 'The :attribute and :other must match.',
    'size' => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute must be between :min - :max.',
    'in' => 'The :attribute must be one of the following types: :values',
];
```

Иногда требуется задать собственное сообщение только для определённого поля. В следующем примере задаётся сообщение для атрибута `email`, когда применяется правило **required**.

```php
$messages = [
    'email.required' => 'We need to know your e-mail address!',
];
```

### Определение сообщений в языковых файлах

В некоторых случаях удобнее разместить собственные сообщения в языковом файле, а не передавать их напрямую в `Validator`. Для этого добавьте сообщения в массив файла локализации плагина **lang/xx/validation.php**.

```php
return  [
    'required' => 'We need to know your e-mail address!',
    'email.required' => 'We need to know your e-mail address!',
];
```

Затем при вызове `Validator::make` используйте `Lang::get`, чтобы подключить собственный файл.

```php
Validator::make($formValues, $validations, Lang::get('acme.blog::validation'));
```

### Переопределение сообщений глобально

Строки сообщений по умолчанию находятся в файле **modules/system/lang/xx/validation.php**. Рекомендуется открыть этот файл и ознакомиться со всеми доступными сообщениями.

Файл содержит массив сообщений для каждого правила валидации. Элемент `custom` предназначен для пользовательских сообщений, использующих соглашение «attribute.rule», а элемент `attributes` хранит пользовательские названия атрибутов.

```php
return [
    'required' => 'The :attribute field is required!',
    // ...

    'custom' => [
        // ...
    ],

    'attributes' => [
        // ...
    ]
];
```

Любое из этих значений можно изменить, создав новый файл в каталоге приложения. Например, для локали `en` создайте файл **app/lang/system/en/validation.php**. Значения из этого файла переопределят значения по умолчанию, при этом достаточно указать только нужные ключи.

```php
return [
    'required' => 'Sorry, we need that field (:attribute) you gave!',

    'attributes' => [
        'email' => 'email address'
    ],
];
```

## Пользовательские правила валидации

Существуют разнообразные полезные правила, однако при необходимости можно определить собственные. Сначала решите, нужно ли регистрировать правило глобально или использовать локальный объект правила.

### Глобально зарегистрированные правила

Глобально зарегистрированное правило можно использовать во всём приложении, зарегистрировав его с тегом и классом правила. Обычно это делается в методе `register` [файла регистрации плагина](../extending.md) с помощью вспомогательного метода `registerValidationRule`.

```php
public function register()
{
    $this->registerValidationRule('uppercase', UppercaseRule::class);
}
```

В этом примере создано правило с тегом **uppercase** и указан класс правила, который становится доступным при определении правил везде в приложении.

```php
Validator::make($data, [
    'shoutout' => 'required|uppercase',
]);
```

#### Определение глобального класса правила

Глобальный класс правила представляет одно переиспользуемое правило валидации для моделей. Минимальное требование — реализовать метод `validate`, определяющий успешность проверки. Дополнительно можно определить метод `message`, возвращающий собственное сообщение об ошибке.

```php
class UppercaseRule
{
    /**
     * validate determines if the validation rule passes.
     * @param string $attribute
     * @param mixed $value
     * @param array $params
     * @return bool
     */
    public function validate($attribute, $value, $params)
    {
        return strtoupper($value) === $value;
    }

    /**
     * message gets the validation error message.
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be uppercase.';
    }
}
```

#### Передача аргументов в правила

Глобальные правила могут принимать аргументы вместе с определением. Например, правило **betwixt** может требовать два значения. Параметры передаются после двоеточия (`:`) и разделяются запятыми (`,`).

```php
$v = Validator::make($data, [
    'name' => 'betwixt:1,6',
]);
```

Параметры передаются в метод `validate` и становятся доступными внутри. Сообщение об ошибке можно модифицировать, определив метод `replace`.

```php
class BetwixtRule
{
    /**
     * validate between start and end parameters.
     */
    public function validate($attribute, $value, $params)
    {
        [$start, $end] = $params;

        return strlen($value) > $start && strlen($value) < $end;
    }

    /**
     * message gets the validation error message.
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be between :start and :end.';
    }

    /**
     * replace defines custom placeholder replacements.
     * @return string
     */
    public function replace($message, $attribute, $rule, $params)
    {
        [$start, $end] = $params;

        $message = str_replace(':start', $start, $message);

        $message = str_replace(':end', $end, $message);

        return $message;
    }
}
```

### Локальные объекты правил

[Документация Laravel по объектам правил](https://laravel.com/docs/6.x/validation#using-rule-objects) подробно описывает, как определить класс правила. Правило должно реализовывать контракт `Illuminate\Contracts\Validation\Rule`, требующий определения метода `passes`.

```php
class LowercaseRule implements \Illuminate\Contracts\Validation\Rule
{
    /**
     * passes checks if the rule is successful
     * @param  string  $attribute
     * @param  mixed  $value
     * @return bool
     */
    public function passes($attribute, $value)
    {
        return strtolower($value) === $value;
    }

    /**
     * message gets the validation error message.
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be lowercase.';
    }
}
```

После определения правила его можно передать сервису `Validator` как экземпляр.

```php
$v = Validator::make($data, [
    'name' => ['required', new LowercaseRule],
]);
```

Правило можно задействовать и в моделях, переопределив метод `beforeValidate`.

```php
public function beforeValidate()
{
    $this->rules['name'] = ['required', new LowercaseRule];
}
```

#### См. также

::: also
* [Laravel Validation Documentation](https://laravel.com/docs/12.x/validation)
:::
