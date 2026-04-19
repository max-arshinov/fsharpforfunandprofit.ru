---
layout: post
title: "Проектирование с помощью типов: одновариантные типы-объединения"
description: "Привносим смысл в примитивные типы"
date: 2013-01-13
nav: thinking-functionally
seriesId: "Проектирование с помощью типов"
seriesOrder: 2
categories: [Types, DDD]
---

> At the end of the previous post, we had values for email addresses, zip codes, etc., defined like this:

Мы завершили предыдущую главу, описав набор полей для электронных адресов, почтовых индексов и прочих компонентов типа `Contact`. 

```fsharp

EmailAddress: string;
State: string;
Zip: string;
```

> These are all defined as simple strings.
> But really, are they just strings?
> Is an email address interchangeable with a zip code or a state abbreviation?

Все они определены как обычные строки.
Но на самом ли деле они — просто строки?
Можно ли заменить электронный адрес почтовым индексом или названием штата?

> In a domain driven design, they are indeed distinct things, not just strings.
> So we would ideally like to have lots of separate types for them so that they cannot accidentally be mixed up.

В предметно-ориентированном проектировании это не просто строки, а совершенно разные значения.
Нам не помешали бы разные типы для этих значений, чтобы их нельзя было перепутать.

> This has been [known as good practice](http://codemonkeyism.com/never-never-never-use-string-in-java-or-at-least-less-often/) for a long time, but in languages like C# and Java it can be painful to create hundred of tiny types like this, leading to the so called ["primitive obsession"](http://sourcemaking.com/refactoring/primitive-obsession) code smell.

Такая практика довольно давно считается [хорошей](http://codemonkeyism.com/never-never-never-use-string-in-java-or-at-least-less-often/), однако в языках наподобие C# и Java, создание сотен крошечных типов — муторное занятие.
Ей следуют немногие программисты и это приводит к коду «с душком», известным как «одержимость примитивами».

> But F# there is no excuse!
> It is trivial to create simple wrapper types.

Но в F# для такого подхода нет оправданий!
Здесь создание простых типов-обёрток — тривиальное дело.

> ## Wrapping primitive types

## Заворачивая примитивные типы

> The simplest way to create a separate type is to wrap the underlying string type inside another type.
> We can do it using single case union types, like so:

Простейший способ создать отдельный тип — поместить строку внутрь одновариантного объединения:

```fsharp
type EmailAddress = EmailAddress of string
type ZipCode = ZipCode of string
type StateCode = StateCode of string
```

> or alternatively, we could use record types with one field, like this:

Другой несложный способ — создать запись с одним строковым полем:

```fsharp
type EmailAddress = { EmailAddress: string }
type ZipCode = { ZipCode: string }
type StateCode = { StateCode: string}
```

> Both approaches can be used to create wrapper types around a string or other primitive type, so which way is better?

Оба подхода можно использовать для создания типов-обёрток над примитивными типами, так что возникает вопрос — какой способ лучше?

> The answer is generally the single case discriminated union.
> It is much easier to "wrap" and "unwrap", as the "union case" is actually a proper constructor function in its own right.
> Unwrapping can be done using inline pattern matching.

Единственный правильный ответ: одновариантное размеченное объединение.
Он гораздо проще для «заворачивания» и «разворачивания», так как единственный размеченный вариант является одновременно и заворачивающей функцией.
А разворачивание выполняется с помощью встраиваемого сопоставления с образцом.

> Here's some examples of how an `EmailAddress` type might be constructed and deconstructed:

Вот несколько примеров того, как завернуть строку в `EmailAddress`, а затем извлечь обратно:

```fsharp
type EmailAddress = EmailAddress of string

// используем название варианта как функцию
"a" |> EmailAddress
["a"; "b"; "c"] |> List.map EmailAddress

// встраиваемое сопоставление с образцом
let a' = "a" |> EmailAddress
let (EmailAddress a'') = a''

let addresses =
    ["a"; "b"; "c"]
    |> List.map EmailAddress

let addresses' =
    addresses
    |> List.map (fun (EmailAddress e) -> e)
```

> You can't do this as easily using record types.

Сделать что-то подобное с помощью записей не выйдет.

> So, let's refactor the code again to use these union types.
> It now looks like this:

Что ж, добавим типы-обёртки в наш код.
Теперь он выглядит так:

```fsharp
type PersonalName =
    {
    FirstName: string;
    MiddleInitial: string option;
    LastName: string;
    }

type EmailAddress = EmailAddress of string

type EmailContactInfo =
    {
    EmailAddress: EmailAddress;
    IsEmailVerified: bool;
    }

type ZipCode = ZipCode of string
type StateCode = StateCode of string

type PostalAddress =
    {
    Address1: string;
    Address2: string;
    City: string;
    State: StateCode;
    Zip: ZipCode;
    }

type PostalContactInfo =
    {
    Address: PostalAddress;
    IsAddressValid: bool;
    }

type Contact =
    {
    Name: PersonalName;
    EmailContactInfo: EmailContactInfo;
    PostalContactInfo: PostalContactInfo;
    }
```

> Another nice thing about the union type is that the implementation can be encapsulated with module signatures, as we'll discuss below.

Приятное дополнение: реализация функций может быть инкапсулирована с помощью сигнатур модуля.
Что это такое и как работает, обсудим чуть позже.

> ## Naming the "case" of a single case union

## Наименование «варианта» одновариантного объединения

> In the examples above we used the same name for the case as we did for the type:

В примерах выше мы использовали одно и то же имя и для типа, и для его единственного варианта:

```fsharp
type EmailAddress = EmailAddress of string
type ZipCode = ZipCode of string
type StateCode = StateCode of string
```

> This might seem confusing initially, but really they are in different scopes, so there is no naming collision.
> One is a type, and one is a constructor function with the same name.

Сначала это может ставить в тупик, но на самом деле эти имена находятся в разных областях видимости и не мешают друг другу.
Одно из них — это тип, а другое — функция-конструктор.

> So if you see a function signature like this:

Если вы видите сигнатуру функции наподобие этой:

```fsharp
val f: string -> EmailAddress
```

> this refers to things in the world of types, so `EmailAddress` refers to the type.

знайте, что она относится к миру типов, поскольку `EmailAddress` — это имя типа.

> On the other hand, if you see some code like this:

С другой стороны, если вы видите такой код:

```fsharp
let x = EmailAddress y
```

> this refers to things in the world of values, so `EmailAddress` refers to the constructor function.

знайте, что он относится к миру значений, поскольку `EmailAddress` — имя функции-конструктора.

> ## Constructing single case unions

## Конструирование одновариантных объединений

> For values that have special meaning, such as email addresses and zip codes, generally only certain values are allowed.
> Not every string is an acceptable email or zip code.

Обычно для величин, имеющих особый смысл, таких как электронные адреса и почтовые индексы, доступны далеко не все значения.
Не всякая строка может быть электронным адресом или почтовым индексом.

> This implies that we will need to do validation at some point, and what better point than at construction time?
> After all, once the value is constructed, it is immutable, so there is no worry that someone might modify it later.

Это значит, что в какой-то момент нам потребуется валидация.
А какой момент может быть лучше, чем время создания переменной?
Помимо прочего, нельзя изменить переменную после создания, так что можно не беспокоиться, что кто-то модифицирует её в будущем.

> Here's how we might extend the above module with some constructor functions:

Вот как мы можем расширить наш модуль, добавив несколько функций-конструкторов:

```fsharp
... типы как выше ...

let CreateEmailAddress (s:string) =
    if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
        then Some (EmailAddress s)
        else None

let CreateStateCode (s:string) =
    let s' = s.ToUpper()
    let stateCodes = ["AZ";"CA";"NY"] // и т. д.
    if stateCodes |> List.exists ((=) s')
        then Some (StateCode s')
        else None
```

> We can test the constructors now:

Протестируем конструкторы:

```fsharp
CreateStateCode "CA"
CreateStateCode "XX"

CreateEmailAddress "a@example.com"
CreateEmailAddress "example.com"
```

> ## Handling invalid input in a constructor ###

## Обработка недопустимых входных данных в конструкторе

> With these kinds of constructor functions, one immediate challenge is the question of how to handle invalid input.
> For example, what should happen if I pass in "abc" to the email address constructor?

Теперь, имея функции-конструкторы с проверкой, мы можем обрабатывать недопустимые входные данные.
Но как?
Например, как быть, если в конструктор электронного адреса я передам `"abc"`?

> There are a number of ways to deal with it.

Есть несколько способов ответить на этот вопрос.

> First, you could throw an exception.
> I find this ugly and unimaginative, so I'm rejecting this one out of hand!

Во-первых, можно выбросить исключение.
В функциональном коде так не делают, так что мы не будем его рассматривать.

> Next, you could return an option type, with `None` meaning that the input wasn't valid.
> This is what the constructor functions above do.

Во-вторых, мы можем вернуть значение опционального типа, где `None` означает, что входные данные недопустимы.
Это то, что делают функции-конструкторы в примере выше.

> This is generally the easiest approach.
> It has the advantage that the caller has to explicitly handle the case when the value is not valid.

Обычно это простейший способ.
У него есть преимущество — вызывающая сторона должна явно обработать недопустимое значение.

> For example, the caller's code for the example above might look like:

Скажем, вызывающий код для примера выше может выглядеть так:

```fsharp
match (CreateEmailAddress "a@example.com") with
| Some email -> ... // что-то делаем с email
| None -> ... // игнорируем?
```

> The disadvantage is that with complex validations, it might not be obvious what went wrong.
> Was the email too long, or missing a '@' sign, or an invalid domain?
> We can't tell.

Недостаток способа в том, что при сложных проверках неочевидно, что пошло не так.
Был ли электронный адрес слишком длинным, или забыли символ `'@'`, или дело в неправильном домене?
Узнать нельзя.

> If you do need more detail, you might want to return a type which contains a more detailed explanation in the error case.

Если вам нужно больше подробностей, возвращайте тип с детальным объяснением ошибки.

> The following example uses a `CreationResult` type to indicate the error in the failure case.

Следующий пример использует тип `CreationResult`, чтобы сигнализировать об ошибке.

```fsharp
type EmailAddress = EmailAddress of string
type CreationResult<'T> = Success of 'T | Error of string

let CreateEmailAddress2 (s:string) =
    if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
        then Success (EmailAddress s)
        else Error "Электронный адрес должен содержать символ @"

// тест
CreateEmailAddress2 "example.com"
```

> Finally, the most general approach uses continuations.
> That is, you pass in two functions, one for the success case (that takes the newly constructed email as parameter), and another for the failure case (that takes the error string as parameter).

Наконец, самый общий подход использует функции-продолжения.
Вы передаёте две функции, одна из которых вызывается в случае успеха (получая, как параметр, сконструированный электронный адрес), а другая — в случае неудачи (получая, как параметр, описание ошибки).

```fsharp
type EmailAddress = EmailAddress of string

let CreateEmailAddressWithContinuations success failure (s:string) =
    if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
        then success (EmailAddress s)
        else failure "Электронный адрес должен содержать символ @"
```

> The success function takes the email as a parameter and the error function takes a string.
> Both functions must return the same type, but the type is up to you.

Успешная функция принимает на вход электронный адрес, а ошибочная — строку с ошибкой.
Обе функции должны вернуть один и тот же тип, но выбор типа остаётся за вами.

> Here is a simple example -- both functions do a printf, and return nothing (i.e. unit).

Простой пример — обе функции вызывают `printf` и не возвращают ничего (т.е., значение типа `unit`).

```fsharp
let success (EmailAddress s) = printfn "успешное создание электронного адреса %s" s
let failure  msg = printfn "ошибка при создании электронного адреса: %s" msg
CreateEmailAddressWithContinuations success failure "example.com"
CreateEmailAddressWithContinuations success failure "x@example.com"
```

> With continuations, you can easily reproduce any of the other approaches.
> Here's the way to create options, for example.
> In this case both functions return an `EmailAddress option`.

С помощью функций-продолжений вы можете имитировать другие способы.
Можно вернуть опциональное значение (обе функции возвращают значение типа `EmailAddress option`):

```fsharp
let success e = Some e
let failure _  = None
CreateEmailAddressWithContinuations success failure "example.com"
CreateEmailAddressWithContinuations success failure "x@example.com"
```

> And here is the way to throw exceptions in the error case:

Или бросить исключение в случае ошибки.

```fsharp
let success e = e
let failure _  = failwith "неверный электронный адрес"
CreateEmailAddressWithContinuations success failure "example.com"
CreateEmailAddressWithContinuations success failure "x@example.com"
```

> This code seems quite cumbersome, but in practice you would probably create a local partially applied function that you use instead of the long-winded one.

Этот код кажется несколько громоздким.
На практике вместо длинной функции создают локальную функцию без последнего параметра, то есть частично-применённую.

```fsharp
// создаём частично-применённую функцию
let success e = Some e
let failure _  = None
let createEmail = CreateEmailAddressWithContinuations success failure

// используем частично-применённую функцию
createEmail "x@example.com"
createEmail "example.com"
```

{{< book_page_ddd >}}

> ## Creating modules for wrapper types ###

## Создаём модули для типов-обёрток

> These simple wrapper types are starting to get more complicated now that we are adding validations, and we will probably discover other functions that we want to associate with the type.

Наши простые типы-обёртки стали сложнее, как только мы добавили валидацию, и, возможно, это не единственное усложнение, которое нам потребуется.

> So it is probably a good idea to create a module for each wrapper type, and put the type and its associated functions there.

Тип и обслуживающие его функции, можно инкапсулировать внутри модуля:

```fsharp
module EmailAddress =

    type T = EmailAddress of string

    // заворачиваем
    let create (s:string) =
        if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
            then Some (EmailAddress s)
            else None

    // разворачиваем
    let value (EmailAddress e) = e
```

> The users of the type would then use the module functions to create and unwrap the type.
> For example:

Теперь пользователи вызывают функции модуля, чтобы завернуть или развернуть занчение:

```fsharp
// создаём электронные адреса
let address1 = EmailAddress.create "x@example.com"
let address2 = EmailAddress.create "example.com"

// разворачиваем электронные адреса
match address1 with
| Some e -> EmailAddress.value e |> printfn "the value is %s"
| None -> ()
```

> ## Forcing use of the constructor ###

## Принудительное использование конструктора

> One issue is that you cannot force callers to use the constructor.
> Someone could just bypass the validation and create the type directly.

Остаётся одна проблема: мы не можем заставить пользователя вызывать функцию-конструктор.
Если захочет, он может пропустить валидацию и создать тип напрямую.

> In practice, that tends not to be a problem.
> One simple techinique is to use naming conventions to indicate a "private" type, and provide "wrap" and "unwrap" functions so that the clients never need to interact with the type directly.

На практике это не проблема.
Можно использовать соглашение об именовании, чтобы «сделать» тип закрытым, предоставив функции для «заворачивания» и «разворачивания».
Клиентам никогда не придётся взаимодействовать с типом напрямую.

> Here's an example:

Пример:

```fsharp

module EmailAddress =
    // закрытый тип
    type _T = EmailAddress of string

    // заворачиваем
    let create (s:string) =
        if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
            then Some (EmailAddress s)
            else None

    // разворачиваем
    let value (EmailAddress e) = e
```

> Of course the type is not really private in this case, but you are encouraging the callers to always use the "published" functions.

Конечно, тип в модуле не становится по настоящему закрытым.
Скорее, вы обозначаете вызывающей стороне своё намерение.

> If you really want to encapsulate the internals of the type and force callers to use a constructor function, you can use module signatures.

Если вы действительно хотите инкапсулировать тип и заставить вызывающую сторону использовать функции доступа, используйте сигнатуры модуля.

> Here's a signature file for the email address example:

Пример файла сигнатур для электронного адреса:

```fsharp
// ФАЙЛ: EmailAddress.fsi

module EmailAddress

// инкапсулированный тип
type T

// заворачиваем
val create : string -> T option

// разворачиваем
val value : T -> string
```

> (Note that module signatures only work in compiled projects, not in interactive scripts, so to test this, you will need to create three files in an F# project, with the filenames as shown here.)

(Обратите внимание, что сигнатура модуля работают только в компилируемых проектах, а не в интерактивных скриптах.
Для тестирования вам понадобиться создать три файла в проекте F# с именами, показанными здесь.)

> Here's the implementation file:

Файл реализации:

```fsharp
// FILE: EmailAddress.fs
// ФАЙЛ: EmailAddress.fs

module EmailAddress

// инкапсулированный тип
type T = EmailAddress of string

// заворачиваем
let create (s:string) =
    if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
        then Some (EmailAddress s)
        else None

// разворачиваем
let value (EmailAddress e) = e
```

> And here's a client:

Клиентский код:

```fsharp
// ФАЙЛ: EmailAddressClient.fs

module EmailAddressClient

open EmailAddress

// код работает при использовании публичных функций
let address1 = EmailAddress.create "x@example.com"
let address2 = EmailAddress.create "example.com"

// код, который использует детали реализации, не компилируется
let address3 = T.EmailAddress "bad email"

```

> The type `EmailAddress.T` exported by the module signature is opaque, so clients cannot access the internals.

Тип `EmailAddress.T`, экспортируемый модулем сигнатур, не раскрывает детали реализации, поэтому клиенты не имеют доступа к внутренностям.

> As you can see, this approach enforces the use of the constructor.
> Trying to create the type directly (`T.EmailAddress "bad email"`) causes a compile error.

Как видите, этот способ заставляет использовать конструктор.
Попытка создать тип напрямую (`T.EmailAddress "bad email"`) приведёт к ошибке компиляции.

> ## When to "wrap" single case unions ###

## Когда следует «заворачивать» одновариантные объединения

> Now that we have the wrapper type, when should we construct them?

Разобравшись с типами-обёртками, перейдём к следующему вопросу.
Когда заворачивать значения?

> Generally you only need to at service boundaries (for example, boundaries in a [hexagonal architecture](http://alistair.cockburn.us/Hexagonal+architecture))

Обычно это делают на границах сервиса (например, на границах в [гексагональной архитектуре](http://alistair.cockburn.us/Hexagonal+architecture)).

> In this approach, wrapping is done in the UI layer, or when loading from a persistence layer, and once the wrapped type is created, it is passed in to the domain layer and manipulated "whole", as an opaque type.
> It is surprisingly uncommon that you actually need the wrapped contents directly when working in the domain itself.

При таком подходе заворачивание выполняется на уровне UI или при загрузке данных из хранилища.
Созданный тип-обёртка передаётся на уровень предметной области, где обрабатывается как единое целое.
На удивление редко приходится извлекать завёрнутые значения, обрабатывать и заворачивать обратно в рамках предметной области.

> As part of the construction, it is critical that the caller uses the provided constructor rather than doing its own validation logic.
> This ensures that "bad" values can never enter the domain.

В рамках конструирования критически важно, чтобы вызывающая сторона использовала функцию-конструктор, а не использовала собственную логику проверки.
Это гарантирует, что «плохие» значения никогда не попадут в предметную область.

> For example, here is some code that shows the UI doing its own validation:

Например, вот код, где UI делает свою собственную проверку:

```fsharp
let processFormSubmit () =
    let s = uiTextBox.Text
    if (s.Length < 50)
        then // присвоить электронный адрес объекту предметной области
        else // показать сообщение об ошибке
```

> A better way is to let the constructor do it, as shown earlier.

Гораздо лучше, если проверку сделает конструктор:

```fsharp
let processFormSubmit () =
    let emailOpt = uiTextBox.Text |> EmailAddress.create
    match emailOpt with
    | Some email -> // присвоить электронный адрес объекту предметной области
    | None -> // показать сообщение об ошибке
```

> ## When to "unwrap" single case unions ###

## Когда следует «разворачивать» одновариантные объединения

> And when is unwrapping needed?
> Again, generally only at service boundaries.
> For example, when you are persisting an email to a database, or binding to a UI element or view model.

Хорошо, а когда требуется развернуть значение?
И снова — в целом — на границах сервиса.

Например, когда вы сохраняете электронный адрес в базе данных, или связываете его с элементом UI или моделью представления.

> One tip to avoid explicit unwrapping is to use the continuation approach again, passing in a function that will be applied to the wrapped value.

Чтобы избежать явного разворачивания, можно использовать подход с продолжением, передавая функцию, которая будет вызывана с развёрнутым значением.

> That is, rather than calling the "unwrap" function explicitly:

Вместо явного вызова функции «разворачивания»:

```fsharp
address |> EmailAddress.value |> printfn "значение %s"
```

> You would pass in a function which gets applied to the inner value, like this:

Можно передать функцию, которая применяется к завёрнутому значению:

```fsharp
address |> EmailAddress.apply (printfn "значение %s")
```

> Putting this together, we now have the complete `EmailAddress` module.

Теперь модуль `EmailAddress` выглядит так:

```fsharp
module EmailAddress =

    type _T = EmailAddress of string

    // создаём с помощью продолжения
    let createWithCont success failure (s:string) =
        if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
            then success (EmailAddress s)
            else failure "Email address must contain an @ sign"

    // создаём напрямую
    let create s =
        let success e = Some e
        let failure _  = None
        createWithCont success failure s

    // разворачиваем с помощью продолжения
    let apply f (EmailAddress e) = f e

    // разворачиваем напрямую
    let value e = apply id e

```

> The `create` and `value` functions are not strictly necessary, but are added for the convenience of callers.

Функции `create` и `value` не являются обязательными, они добавлены для удобства вызывающей стороны.

> ## The code so far ###

## Код «целиком»

> Let's refactor the `Contact` code now, with the new wrapper types and modules added.

Проведём рефакторинг кода `Contract`, добавив типы-обёртки и модули.

```fsharp
module EmailAddress =

    type T = EmailAddress of string

    // создаём с помощью продолжения
    let createWithCont success failure (s:string) =
        if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
            then success (EmailAddress s)
            else failure "Email address must contain an @ sign"

    // создаём напрямую
    let create s =
        let success e = Some e
        let failure _  = None
        createWithCont success failure s

    // разворачиваем с помощью продолжения
    let apply f (EmailAddress e) = f e

    // разворачиваем напрямую
    let value e = apply id e

module ZipCode =

    type T = ZipCode of string

    // создаём с помощью продолжения
    let createWithCont success failure  (s:string) =
        if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\d{5}$")
            then success (ZipCode s)
            else failure "Zip code must be 5 digits"

    // создаём напрямую
    let create s =
        let success e = Some e
        let failure _  = None
        createWithCont success failure s

    // разворачиваем с помощью продолжения
    let apply f (ZipCode e) = f e

    // разворачиваем напрямую
    let value e = apply id e

module StateCode =

    type T = StateCode of string

    // создаём с помощью продолжения
    let createWithCont success failure  (s:string) =
        let s' = s.ToUpper()
        let stateCodes = ["AZ";"CA";"NY"] //etc
        if stateCodes |> List.exists ((=) s')
            then success (StateCode s')
            else failure "State is not in list"

    // создаём напрямую
    let create s =
        let success e = Some e
        let failure _  = None
        createWithCont success failure s

    // разворачиваем с помощью продолжения
    let apply f (StateCode e) = f e

    // разворачиваем напрямую
    let value e = apply id e

type PersonalName =
    {
    FirstName: string;
    MiddleInitial: string option;
    LastName: string;
    }

type EmailContactInfo =
    {
    EmailAddress: EmailAddress.T;
    IsEmailVerified: bool;
    }

type PostalAddress =
    {
    Address1: string;
    Address2: string;
    City: string;
    State: StateCode.T;
    Zip: ZipCode.T;
    }

type PostalContactInfo =
    {
    Address: PostalAddress;
    IsAddressValid: bool;
    }

type Contact =
    {
    Name: PersonalName;
    EmailContactInfo: EmailContactInfo;
    PostalContactInfo: PostalContactInfo;
    }

```

> By the way, notice that we now have quite a lot of duplicate code in the three wrapper type modules.
> What would be a good way of getting rid of it, or at least making it cleaner?

Обратите внимание, что в нашем примере много похожего кода в модулях с типами-обёртками.
Есть ли хороший способ избавиться от дублей или, по крайней мере, сделать код чище?

> ## Summary ###

## Заключение

> To sum up the use of discriminated unions, here are some guidelines:

Подведём итоги:

> * Do use single case discriminated unions to create types that represent the domain accurately.
> * If the wrapped value needs validation, then provide constructors that do the validation and enforce their use.
> * Be clear what happens when validation fails.
>   In simple cases, return option types.
>   In more complex cases, let the caller pass in handlers for success and failure.
> * If the wrapped value has many associated functions, consider moving it into its own module.
> * If you need to enforce encapsulation, use signature files.

* Используйте одновариантные размеченные объединения для создания типов,
  точно описывающих предметную область.
* Если обёрнутое значение требует проверки, предоставьте конструкторы с
  проверкой и заставьте пользоваться ими.
* Ясно показывайте, что будет, если проверка завершится неудачей.
  В простых случаях возвращайте опциональные типы.
  В сложных случаях, используйте функции-продолжения.
* Если у завёрнутого значения много связанных функций, поместить их
  в отдельный модуль.
* Если вам нужна инкапсуляция, используйте файлы сигнатур.

> We're still not done with refactoring.
> We can alter the design of types to enforce business rules at compile time -- making illegal states unrepresentable.

Однако, мы не закончили с рефакторингом.
В следующей части сделаем недопустимые состояния непредставимыми.

{{< linktarget "update" >}}

> ## Update ##

## Обновление

> Many people have asked for more information on how to ensure that constrained types such as `EmailAddress` are only created through a special constructor that does the validation.
> So I have created a [gist here](https://gist.github.com/swlaschin/54cfff886669ccab895a) that has some detailed examples of other ways of doing it.

Многие просят подробнее рассказать о том, как гарантировать создание типов навроде `EmailAddress`, только через конструктор с проверкой.
Я создал небольшой [фрагмент](https://gist.github.com/swlaschin/54cfff886669ccab895a) с несколькими примерами, как добиться нужного результата.
