# AMCms Wiki

## Contents

* [Описание](intro.md)
* _[Валидация](validation.md)_

## Валидация форм

Основа для сервиса валидации - библиотека respect/validation и ее имплементация для Slim 3 awurth/slim-validation.


### Базовый синтаксис

Валидация в текущей реализации производится в контроллерах и выглядит примерно так:

```php
$dataForSave = $this->validator->check($request, [
        'email' => 'email',
        'name' => 'required',
    ]);

if ($this->validator->failed()) {
    return $response->withRedirect($this->router->pathFor('form'));
}

$user = User::create($dataForSave);
```

Разработчик может использовать встроенные в AMC правила валидации простым указанием соответствующего правила для каждого параметра HTTP-запроса, как в примере выше. В этом случае в глобальную переменную errors будет записано встроенное сообщение об ошибке ввода в форму. 

Для указания непосредственно в коде контроллера своего собственного сообщения об ошибке, используется следующий способ:

```php
$dataForSave = $this->validator->check($request, [
        'email' => ['email', 'Email must be valid'],
        'name' => 'required',
    ]);
```

Альтернативой может быть редактирование языковых файлов resource/lang/[LOCALE]/validation.php, где для каждого встроенного правила задано сообщение об ошибке вида:

```php
    'email' => ':attr: must be valid email',
```

**:attr:** при выводе будет заменяться на название поля в форме. Если поле в форме называется не "email", а "myEmail", сообщение об ошибке будет выглядеть как **MyEmail must be valid email**.

Если для поля нужно задать несколько правил валидации, их следует задать через запятую. При этом дополнительные параметры следуют за названием правила через двоеточие:

```php
$dataForSave = $this->validator->check($request, [
        'age' => 'digit,min:15,max:90',
    ]);
```

### Продвинутое использование

Если требуется задать правило, которое не является встроенным, но существует в библиотеке respect/validation, его можно задать через замыкание:

```php
use Respect\Validation\Validator as v;

class MyController extends Controller
{
...

$dataForSave = $this->validator->check($request, [
        'password' => [function() { return v::notEmpty(); }, 'Password is required'],
    ]);
```

Правила валидации для этого случая можно подобрать в [документации respect/validation](http://respect.github.io/Validation/docs/validators.html)

Если требуется задать несколько правил и для каждого нужно задать свое сообщение об ошибке, следует использовать такой синтаксис:

```php
$dataForSave = $this->validator->check($request, [
        'email' => ['required,email,unique:users:email', [
                // для правила "required" будет использовано встроенное сообщение
                'email' => 'Email must be valid',
                'unique' => 'This email already in use',
            ]
        ],
        'name' => 'required',
    ]);
```


### Встроенные правила валидации

@todo