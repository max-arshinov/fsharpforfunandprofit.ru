---
layout: post
# title: "Designing with types: Single case union types"
title: "Проектирование с помощью типов: Одновариантные типы-объединения"
# description: "Adding meaning to primitive types"
description: "Добавляем смысл примитивным типам"
date: 2013-01-13
nav: thinking-functionally
# seriesId: "Designing with types"
seriesId: "Проектирование с помощью типов"
seriesOrder: 2
categories: [Types, DDD]
---

> At the end of the previous post, we had values for email addresses, zip codes, etc., defined like this:

В конце предыдущего поста у нас появились значения для электронных адресов, почтовых индексов и прочего, определённые подобным образом:

```fsharp

EmailAddress: string;
State: string;
Zip: string;

```

> These are all defined as simple strings.
> But really, are they just strings?
> Is an email address interchangeable with a zip code or a state abbreviation?

Все они определены как простые строки.
Но являются ли они простыми строками на самом деле?
Взаимозаменяем ли электронный адрес с почтовым индексом или аббревиатурой штата?

> In a domain driven design, they are indeed distinct things, not just strings.
> So we would ideally like to have lots of separate types for them so that they cannot accidentally be mixed up.

В предметно-ориентированном программировании это определённо разные вещи, а не просто строки.
Так что в идеале нам было бы здорово иметь несколько различных типов для них, чтобы их нельзя было случайно спутать.

> This has been [known as good practice](http://codemonkeyism.com/never-never-never-use-string-in-java-or-at-least-less-often/) for a long time, but in languages like C# and Java it can be painful to create hundred of tiny types like this, leading to the so called ["primitive obsession"](http://sourcemaking.com/refactoring/primitive-obsession) code smell.

В течение длительного времени эта практика известна, как [хорошая](http://codemonkeyism.com/never-never-never-use-string-in-java-or-at-least-less-often/), но в языках наподобие C# или Java может быть болезненным <!-- муторным, мучительным? --> создавать сотни таких крошечных типов, что приводит к «коду с душком», известным как ["одержимость примитивами"](http://sourcemaking.com/refactoring/primitive-obsession).

> But F# there is no excuse!
> It is trivial to create simple wrapper types.

Но в F# для неё нет оправданий!
Здесь создавать простые типы-обёртки тривиально.

> ## Wrapping primitive types

## Заворачивая примитивные типы

> The simplest way to create a separate type is to wrap the underlying string type inside another type.

Простейший способ создать отдельный тип — поместить простой строковый тип внутрь другого типа.

> We can do it using single case union types, like so:

Мы можем сделать это используя одновариантные типы-объединения, как здесь:

```fsharp
type EmailAddress = EmailAddress of string
type ZipCode = ZipCode of string
type StateCode = StateCode of string
```

> or alternatively, we could use record types with one field, like this:

или, в качестве альтернативы, мы могли бы использовать типы-записи с одним полем, как здесь:

```fsharp
type EmailAddress = { EmailAddress: string }
type ZipCode = { ZipCode: string }
type StateCode = { StateCode: string}
```

> Both approaches can be used to create wrapper types around a string or other primitive type, so which way is better?

Оба подхода могут быть использованы для создания типов-обёрток над строками или другими примитивными типами, так что — какой из способов лучше?

> The answer is generally the single case discriminated union.
> It is much easier to "wrap" and "unwrap", as the "union case" is actually a proper constructor function in its own right.
> Unwrapping can be done using inline pattern matching.

Ответ, как правило, один: одновариантное размеченное объединение.
С его помощью гораздо проще «заворачивать» и «разворачивать» <!-- значение -->, поскольку «вариант объединения» сам по себе является полноценной конструирующий функцией.
Разворачиваение может быть сделано с помощью встроенного сопоставления с образцом.

> Here's some examples of how an `EmailAddress` type might be constructed and deconstructed:

Вот несколько примеров того, как `EmailAddress` может быть сконструирован и деконструирован:

```fsharp
type EmailAddress = EmailAddress of string

// using the constructor as a function
// используем конструктор как функцию
"a" |> EmailAddress
["a"; "b"; "c"] |> List.map EmailAddress

// inline deconstruction
// встроенная деконструкция
let a' = "a" |> EmailAddress
let (EmailAddress a'') = a'

let addresses =
    ["a"; "b"; "c"]
    |> List.map EmailAddress

let addresses' =
    addresses
    |> List.map (fun (EmailAddress e) -> e)
```

> You can't do this as easily using record types.

Сделать это также просто с помощью типов-записей не получится.

> So, let's refactor the code again to use these union types.
> It now looks like this:

Что ж, давайте снова исправим когд, чтобы применить эти типы-объединения.
Сейчас он выглядит так:

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

Другой замечательной вещью, которая касается типа-объединения является та, что реализация может быть инкапсулирована с помощью сигнатур модуля, как мы обсудим ниже.

> ## Naming the "case" of a single case union

## Наименование «варианта» одновариантного объединения

> In the examples above we used the same name for the case as we did for the type:

В примерах выше мы использовали одно и то же имя и для варианта и для типа:

```fsharp
type EmailAddress = EmailAddress of string
type ZipCode = ZipCode of string
type StateCode = StateCode of string
```

> This might seem confusing initially, but really they are in different scopes, so there is no naming collision.
> One is a type, and one is a constructor function with the same name.

В начале это может ставить в тупик, но в действительности они находятся в разных областях видимости, так что противоречий между именами не возинкает.
Одно из них это тип, а другое — конструирующая функция с тем же именем.

> So if you see a function signature like this:

Так что если вы видите подобную сигнатуру функции:

```fsharp
val f: string -> EmailAddress
```

> this refers to things in the world of types, so `EmailAddress` refers to the type.

она относится к штукам из мира типов, поскольку `EmailAddress` — это имя типа.

> On the other hand, if you see some code like this:

С другой стороны, если вы видите код, похожий на этот:

```fsharp
let x = EmailAddress y
```

> this refers to things in the world of values, so `EmailAddress` refers to the constructor function.

он относится к штукам из мира значений, поскольку `EmailAddress` — имя конструирующей функции.

> ## Constructing single case unions

## Конструирование одновариантных объединений

> For values that have special meaning, such as email addresses and zip codes, generally only certain values are allowed.
> Not every string is an acceptable email or zip code.

Для значений со особым смыслом, таких как электронные адреса и почтовые индексы, обычно доступны только определённые значения.
Не каждая строка может быть электронным адресом или почтовым индексом.

> This implies that we will need to do validation at some point, and what better point than at construction time?
> After all, once the value is constructed, it is immutable, so there is no worry that someone might modify it later.

Это приводит нас к тому, что мы должны в какой-то момент выполнить проверку, и какой момент может быть лучше, чем время конструирования?
Помимо прочего, раз уж значение сконструировано, оно неизменяемое, так что можно не беспокоиться, что кто-то изменит его в будущем.

> Here's how we might extend the above module with some constructor functions:

Вот как мы можем расширить вышеприведённый модуль некоторыми конструирующими функциями:

```fsharp

... types as above ...
... типы как выше ...

let CreateEmailAddress (s:string) =
    if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
        then Some (EmailAddress s)
        else None

let CreateStateCode (s:string) =
    let s' = s.ToUpper()
    let stateCodes = ["AZ";"CA";"NY"] //etc
    if stateCodes |> List.exists ((=) s')
        then Some (StateCode s')
        else None
```

> We can test the constructors now:

Теперь мы можем протестировать конструкторы:

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

При использовании подобных конструирующих функций возникает важный вопрос: как обрабатывать недопустимые входные данные.
Например, что должно случиться, если я передам `"abc"` в конструктор электронного адреса?

> There are a number of ways to deal with it.

Есть несколько способов с этим справиться.

> First, you could throw an exception.
> I find this ugly and unimaginative, so I'm rejecting this one out of hand!

Во-первых, можно бросить исключение.
Я считаю это предложение уродливым и невообразимым, поэтому сразу же отвергаю его!

> Next, you could return an option type, with `None` meaning that the input wasn't valid.
> This is what the constructor functions above do.

Затем, вы могли бы вернуть значение `None` опционального типа, означающее, что входные данные оказались недопустимыми.
Это то, что делает конструирующая функция выше.

> This is generally the easiest approach.
> It has the advantage that the caller has to explicitly handle the case when the value is not valid.

Как правило, это простейший способ.
Его преимущество в том, что вызывающая сторона должна явно обработать случай, когда значение недопустимо.

> For example, the caller's code for the example above might look like:

Например, вызывающий код для примера выше может выглядеть так:

```fsharp
match (CreateEmailAddress "a@example.com") with
| Some email -> ... что-то делаем с email
| None -> ... игнорируем?
```

> The disadvantage is that with complex validations, it might not be obvious what went wrong.
> Was the email too long, or missing a '@' sign, or an invalid domain?
> We can't tell.

Недостаток в том, что при сложных проверках, может быть неочевидно, что пошло не так.
Был ли электронный адрес слишком длинным, или забыли символ `'@'`, или дело в неправильном домене?
Мы не можем сказать.

> If you do need more detail, you might want to return a type which contains a more detailed explanation in the error case.

Если вам нужно больше подробностей, мы можете захотеть вернуть тип, который содержит более подробное объяснение в случае ошибки.

> The following example uses a `CreationResult` type to indicate the error in the failure case.

Следующий пример использует тип `CreationResult`, чтобы сигнализировать об ошибке в случае неудачи.

```fsharp
type EmailAddress = EmailAddress of string
type CreationResult<'T> = Success of 'T | Error of string

let CreateEmailAddress2 (s:string) =
    if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
        then Success (EmailAddress s)
        else Error "Электронный адрес должен содержать символ @"

// test
// тест
CreateEmailAddress2 "example.com"
```

> Finally, the most general approach uses continuations.
> That is, you pass in two functions, one for the success case (that takes the newly constructed email as parameter), and another for the failure case (that takes the error string as parameter).

Наконец, самый общий подход использует функции-продолжения.
То есть, вы передаёте две функции, одна из которых будет вызвана в случае успеха (получая сконструированный электронный адрес как параметр), а другая — в случае неудачи (получая строковое описание ошибки как параметр).

```fsharp
type EmailAddress = EmailAddress of string

let CreateEmailAddressWithContinuations success failure (s:string) =
    if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
        then success (EmailAddress s)
        else failure "Электронный адрес должен содержать символ @"
```

> The success function takes the email as a parameter and the error function takes a string.
> Both functions must return the same type, but the type is up to you.

Успешная функция принимает в качестве параметра электронный адрес, а ошибочная функция — принимает строку.
Обе функции должны вернуть один и тот же тип, но выбор типа остаётся за вами.

> Here is a simple example -- both functions do a printf, and return nothing (i.e. unit).

Вот простой пример — обе функции вызывают `printf` и не возвращают ничего (т.е., значение типа `unit`).

```fsharp
let success (EmailAddress s) = printfn "успешное создание электронного адреса %s" s
let failure  msg = printfn "ошибка при создании электронного адреса: %s" msg
CreateEmailAddressWithContinuations success failure "example.com"
CreateEmailAddressWithContinuations success failure "x@example.com"
```

> With continuations, you can easily reproduce any of the other approaches.
> Here's the way to create options, for example.
> In this case both functions return an `EmailAddress option`.

С помощью функций-продолжений вы запросто можете имитировать любой другой подход.
Вот, например, способ создать опциональное значение.
В этом случае обе функции возвращают значение типа `EmailAddress option`.

```fsharp
let success e = Some e
let failure _  = None
CreateEmailAddressWithContinuations success failure "example.com"
CreateEmailAddressWithContinuations success failure "x@example.com"
```

> And here is the way to throw exceptions in the error case:

А вот — способ бросить исключения в случае ошибки.

```fsharp
let success e = e
let failure _  = failwith "неверный электронный адрес"
CreateEmailAddressWithContinuations success failure "example.com"
CreateEmailAddressWithContinuations success failure "x@example.com"
```

> This code seems quite cumbersome, but in practice you would probably create a local partially applied function that you use instead of the long-winded one.

Этот код кажется несколько громоздким, но на практике вы бы наверное создали локальную частично-применённую функцию, которую использовали бы вместо длинной.

```fsharp
// setup a partially applied function
// создаём частично-применённую функцию
let success e = Some e
let failure _  = None
let createEmail = CreateEmailAddressWithContinuations success failure

// use the partially applied function
// используем частично-применённую функцию
createEmail "x@example.com"
createEmail "example.com"
```

{{< book_page_ddd >}}

> ## Creating modules for wrapper types ###

## Создаём модули для типов-обёрток

> These simple wrapper types are starting to get more complicated now that we are adding validations, and we will probably discover other functions that we want to associate with the type.

Эти простые типы-обёртки начинают усложняться сейчас, когда мы добавляем проверки и, возможно, мы обнаружим другие функции, которые мы захотим связать с типом.

> So it is probably a good idea to create a module for each wrapper type, and put the type and its associated functions there.

Так что, кажется, это хорошая идея — создать модуль для каждого типа-обёртки, и поместить тип и связанные функции туда.

```fsharp
module EmailAddress =

    type T = EmailAddress of string

    // wrap
    // заворачиваем
    let create (s:string) =
        if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
            then Some (EmailAddress s)
            else None

    // unwrap
    // разворачиваем
    let value (EmailAddress e) = e
```

> The users of the type would then use the module functions to create and unwrap the type.
> For example:

Тогда бы пользователи типа могли использовать функции модуля, чтобы создавать и разворачивать тип.
Например:

```fsharp

// create email addresses
// создаём электронные адреса
let address1 = EmailAddress.create "x@example.com"
let address2 = EmailAddress.create "example.com"

// unwrap an email address
// разворачиваем электронные адреса
match address1 with
| Some e -> EmailAddress.value e |> printfn "the value is %s"
| None -> ()
```

> ## Forcing use of the constructor ###

## Принудительное использование конструктора

> One issue is that you cannot force callers to use the constructor.
> Someone could just bypass the validation and create the type directly.

Одна из проблем заключается в том, что вы не можете заставить вызывающие функции использовать конструктор.
Кто-то мог бы просто пропустить валидацию и создать тип напрямую.

> In practice, that tends not to be a problem.
> One simple techinique is to use naming conventions to indicate a "private" type, and provide "wrap" and "unwrap" functions so that the clients never need to interact with the type directly.

На практике, это не является проблемой.
Одна простая техника это использование соглашений об именовании, чтобы указать «закрытый» тип и предоставить функции «заворачивания» и «разворачивания», так что клиентам никогда не понадобиться взаимодействовать напрямую с типом.

> Here's an example:

Вот пример:

```fsharp

module EmailAddress =

    // private type
    // закрытый тип
    type _T = EmailAddress of string

    // wrap
    // заворачиваем
    let create (s:string) =
        if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
            then Some (EmailAddress s)
            else None

    // unwrap
    // разворачиваем
    let value (EmailAddress e) = e
```

> Of course the type is not really private in this case, but you are encouraging the callers to always use the "published" functions.

Конечно этот тип не является по настоящему закрытым в данном случае, но вы поощряете вызывающую сторону всегда использовать «публичные» функции.

> If you really want to encapsulate the internals of the type and force callers to use a constructor function, you can use module signatures.

Если вы действительно хотите инкапсулировать внутренности типа и заставить вызывающую сторону использовать конструирующую функцию, вы можете использовать сигнатуры модуля.

> Here's a signature file for the email address example:

Вот пример файла сигнатур для электронного адреса:

```fsharp
// FILE: EmailAddress.fsi
// ФАЙЛ: EmailAddress.fsi

module EmailAddress

// encapsulated type
// инкапсулированный тип
type T

// wrap
// заворачиваем
val create : string -> T option

// unwrap
// разворачиваем
val value : T -> string
```

> (Note that module signatures only work in compiled projects, not in interactive scripts, so to test this, you will need to create three files in an F# project, with the filenames as shown here.)

(Обратите внимание, что сигнатура модуля работают только в компилируемых проектах, а не в интерактивных скриптах, поэтому для тестирования вам понадобиться создать три файла в проекте F#, с именами, показанными здесь.)

> Here's the implementation file:

Вот файл реализации:

```fsharp
// FILE: EmailAddress.fs
// ФАЙЛ: EmailAddress.fs

module EmailAddress

// encapsulated type
// инкапсулированный тип
type T = EmailAddress of string

// wrap
// заворачиваем
let create (s:string) =
    if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
        then Some (EmailAddress s)
        else None

// unwrap
// разворачиваем
let value (EmailAddress e) = e

```

> And here's a client:

А вот — клиент:

```fsharp
// FILE: EmailAddressClient.fs
// ФАЙЛ: EmailAddressClient.fs

module EmailAddressClient

open EmailAddress

// code works when using the published functions
// код работает при использовании публичных функций
let address1 = EmailAddress.create "x@example.com"
let address2 = EmailAddress.create "example.com"

// code that uses the internals of the type fails to compile
// код, который использует внутреннсоти типа, не будет компилироваться
let address3 = T.EmailAddress "bad email"

```

> The type `EmailAddress.T` exported by the module signature is opaque, so clients cannot access the internals.

Тип `EmailAddress.T`, экспортируемый модулем сигнатур, является непрозрачным <!-- не раскрывает деталей реализации -->, поэтому клиенты не могут получить доступ к внутренностям.

> As you can see, this approach enforces the use of the constructor.
> Trying to create the type directly (`T.EmailAddress "bad email"`) causes a compile error.

Как видите, этот подход заставляет использовать конструктор.
Попытка создать тип напрямую (`T.EmailAddress "bad email"`) ведёт к ошибке компиляции.

> ## When to "wrap" single case unions ###

## Когда следует «заворачивать» одновариантные объединения

> Now that we have the wrapper type, when should we construct them?

Теперь, имея тип-обёртку, когда мы должны их конструировать?

> Generally you only need to at service boundaries (for example, boundaries in a [hexagonal architecture](http://alistair.cockburn.us/Hexagonal+architecture))

Обычно это нужно на границах сервиса (например, на границах в [гексагональной архитектуре](http://alistair.cockburn.us/Hexagonal+architecture)).

> In this approach, wrapping is done in the UI layer, or when loading from a persistence layer, and once the wrapped type is created, it is passed in to the domain layer and manipulated "whole", as an opaque type.
> It is surprisingly uncommon that you actually need the wrapped contents directly when working in the domain itself.

В этом подходе, заворачивание выполняется на уровне UI, или на уровне загрузки данных из долговременного хранилища и, как только тип-обёртка создан, он передаётся на уровень предметной области и обрабатывается «целиком», как непрозрачный тип.
На удивление редко вам действительно нужно заворачивать содержимое непосредственно при работе внутри предметной области.

> As part of the construction, it is critical that the caller uses the provided constructor rather than doing its own validation logic.
> This ensures that "bad" values can never enter the domain.

В рамках конструирования, критически важно, чтобы вызывающая сторона использовала предоставленный конструктор, а не выполняла собственную логику проверки.
Это гарантирует, что «плохие» значения никогда не попадут в предметную область.

> For example, here is some code that shows the UI doing its own validation:

Для примера, вот кусок кода, который показывает, как UI делает свою собственную проверку:

```fsharp
let processFormSubmit () =
    let s = uiTextBox.Text
    if (s.Length < 50)
        then // присвоить электронный адрес объекту предметной области (set email on domain object)
        else // показать сообщение об ошибке проверки (show validation error message)
```

> A better way is to let the constructor do it, as shown earlier.

Лучший способ — позволить сделать это конструктору, как показано ранее.

```fsharp
let processFormSubmit () =
    let emailOpt = uiTextBox.Text |> EmailAddress.create
    match emailOpt with
    | Some email -> // присвоить электронный адрес объекту предметной области (set email on domain object)
    | None -> // показать сообщение об ошибке проверки (show validation error message)
```

> ## When to "unwrap" single case unions ###

## Когда следует «разворачивать» одновариантные объединения

> And when is unwrapping needed?
> Again, generally only at service boundaries.
> For example, when you are persisting an email to a database, or binding to a UI element or view model.

А когда требуется разворчивание?
В целом, снова — только на границах сервиса.
Например, когда вы сохраняете электронный адрес в базу данных, или связываете его с элементом UI или моделью представления.

> One tip to avoid explicit unwrapping is to use the continuation approach again, passing in a function that will be applied to the wrapped value.

Один из советов, как избежать явного разворачивания — снова использовать подход с продолжениями, передавая функцию, которая будет применена к завёрнутому значению.

> That is, rather than calling the "unwrap" function explicitly:

То есть, вместо явного вызова функции «разворачивания»:

```fsharp
address |> EmailAddress.value |> printfn "значение %s"
```

> You would pass in a function which gets applied to the inner value, like this:

Вы бы передавали функцию, которая применяется к внутреннему значению, как здесь:

```fsharp
address |> EmailAddress.apply (printfn "значение %s")
```

> Putting this together, we now have the complete `EmailAddress` module.

Собрав всё вместе, получаем полноценный модуль `EmailAddress`.

```fsharp
module EmailAddress =

    type _T = EmailAddress of string

    // create with continuation
    // создаём с помощью продолжения
    let createWithCont success failure (s:string) =
        if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
            then success (EmailAddress s)
            else failure "Email address must contain an @ sign"

    // create directly
    // создаём напрямую
    let create s =
        let success e = Some e
        let failure _  = None
        createWithCont success failure s

    // unwrap with continuation
    // разворачиваем с помощью продолжения
    let apply f (EmailAddress e) = f e

    // unwrap directly
    // разворачиваем напрямую
    let value e = apply id e

```

> The `create` and `value` functions are not strictly necessary, but are added for the convenience of callers.

Функции `create` и `value` не являются обязательными, но добавлены для удобства вызывающей стороны.

> ## The code so far ###

## Код на данный момент

> Let's refactor the `Contact` code now, with the new wrapper types and modules added.

Давайте теперь проведём рефакторинг кода `Contract`, с новыми добавленными типами-обёртками и модулями.

```fsharp
module EmailAddress =

    type T = EmailAddress of string

    // create with continuation
    // создаём с помощью продолжения
    let createWithCont success failure (s:string) =
        if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
            then success (EmailAddress s)
            else failure "Email address must contain an @ sign"

    // create directly
    // создаём напрямую
    let create s =
        let success e = Some e
        let failure _  = None
        createWithCont success failure s

    // unwrap with continuation
    // разворачиваем с помощью продолжения
    let apply f (EmailAddress e) = f e

    // unwrap directly
    // разворачиваем напрямую
    let value e = apply id e

module ZipCode =

    type T = ZipCode of string

    // create with continuation
    // создаём с помощью продолжения
    let createWithCont success failure  (s:string) =
        if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\d{5}$")
            then success (ZipCode s)
            else failure "Zip code must be 5 digits"

    // create directly
    // создаём напрямую
    let create s =
        let success e = Some e
        let failure _  = None
        createWithCont success failure s

    // unwrap with continuation
    // разворачиваем с помощью продолжения
    let apply f (ZipCode e) = f e

    // unwrap directly
    // разворачиваем напрямую
    let value e = apply id e

module StateCode =

    type T = StateCode of string

    // create with continuation
    // создаём с помощью продолжения
    let createWithCont success failure  (s:string) =
        let s' = s.ToUpper()
        let stateCodes = ["AZ";"CA";"NY"] //etc
        if stateCodes |> List.exists ((=) s')
            then success (StateCode s')
            else failure "State is not in list"

    // create directly
    // создаём напрямую
    let create s =
        let success e = Some e
        let failure _  = None
        createWithCont success failure s

    // unwrap with continuation
    // разворачиваем с помощью продолжения
    let apply f (StateCode e) = f e

    // unwrap directly
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

Кстати, обратите внимание что у нас сейчас довольно много дублирующегося кода в трёх модулях с типами-обёртками.
Есть ли хороший способ избавиться от него или по крайней мере сделать его чище?

> ## Summary ###

## Заключение

> To sum up the use of discriminated unions, here are some guidelines:

Суммируем использование размеченных объединения в нескольких рекомендациях:

> * Do use single case discriminated unions to create types that represent the domain accurately.
> * If the wrapped value needs validation, then provide constructors that do the validation and enforce their use.
> * Be clear what happens when validation fails.
>   In simple cases, return option types.
>   In more complex cases, let the caller pass in handlers for success and failure.
> * If the wrapped value has many associated functions, consider moving it into its own module.
> * If you need to enforce encapsulation, use signature files.

* Используйте одновариантные размеченные объединения для создания типов, которые точно представляют предметную область.
* Если обёрнутое значение требует проверки, предоставьте конструкторы, которые выполняют проверку и заставьте пользоваться ими.
* Ясно показывайте, что будет, если проверка завершится неудачей.
  В простых случаях возвращайте опциональные типы.
  В более сложных случаях, пусть вызывающая сторона передаёт обработчики на случай успешной и неудачной проверки.
* Если у обёрнутого значения есть много связанных функций, подумайте над тем, чтобы поместить их в отдельный модуль.
* Если вам необходимо обеспечить инкапсуляцию, используйте файлы сигнатур.

> We're still not done with refactoring.
> We can alter the design of types to enforce business rules at compile time -- making illegal states unrepresentable.

Мы всё ещё не закончили с рефакторингом.
Мы можем изменить дизайн типов, чтобы обеспечить соблюдение бизнес-правил на этапе компиляции — делая недопустимые состояния непредставимыми.

{{< linktarget "update" >}}

> ## Update ##

## Обновление

> Many people have asked for more information on how to ensure that constrained types such as `EmailAddress` are only created through a special constructor that does the validation.
> So I have created a [gist here](https://gist.github.com/swlaschin/54cfff886669ccab895a) that has some detailed examples of other ways of doing it.

Многие просят больше рассказать о том, как гарантировать, что типы с ограничениями, такие как `EmailAddress`? создаются только через специальный конструктор, который выполняет проверку.
Так что я создал небольшой [фрагмент здесь](https://gist.github.com/swlaschin/54cfff886669ccab895a), в котором есть несколько подробных примеров других способов добиться нужного эффекта.