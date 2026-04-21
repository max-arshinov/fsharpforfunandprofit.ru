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

Мы завершили предыдущую главу, описав набор полей для электронных адресов, почтовых индексов и прочих компонентов типа `Contact`. 

```fsharp

EmailAddress: string;
State: string;
Zip: string;
```

Все они определены как обычные строки.
Но на самом ли деле они — просто строки?
Можно ли заменить электронный адрес почтовым индексом или названием штата?

В предметно-ориентированном проектировании это не просто строки, а совершенно разные значения.
Чтобы их не перепутать, нам бы не помешали бы разные типы для этих значений.

Эта практика давно считается [хорошей](http://codemonkeyism.com/never-never-never-use-string-in-java-or-at-least-less-often/), однако, в таких языках, как C# и Java, создание сотен крошечных типов — муторное дело.
Ей следуют немногие программисты, что приводит к коду «с душком», известным как «одержимость примитивами».

Но в F# для «одержимости примитивами» нет оправданий!
Здесь создание простых типов-обёрток — тривиальное дело.

## Заворачиваем примитивные типы

Простейший способ создать отдельный тип — поместить строку внутрь одновариантного объединения:

```fsharp
type EmailAddress = EmailAddress of string
type ZipCode = ZipCode of string
type StateCode = StateCode of string
```

Другой несложный способ — создать запись с одним строковым полем:

```fsharp
type EmailAddress = { EmailAddress: string }
type ZipCode = { ZipCode: string }
type StateCode = { StateCode: string}
```

Оба подхода можно использовать для создания обёрток над примитивными типами.
Возникает вопрос — какой из них лучше?

Правильный ответ: одновариантное размеченное объединение.
Он гораздо проще для «заворачивания» и «разворачивания», поскольку единственный размеченный вариант одновременно является и заворачивающей функцией.
А разворачивание выполняется с помощью встраиваемого сопоставления с образцом.

Вот несколько примеров того, как завернуть строку в `EmailAddress`, а затем извлечь обратно:

```fsharp
type EmailAddress = EmailAddress of string

// используем название варианта как функцию
"a" |> EmailAddress
["a"; "b"; "c"] |> List.map EmailAddress

// встраиваемое сопоставление с образцом
let a' = "a" |> EmailAddress
let (EmailAddress a'') = a' // в a'' теперь строка "a"

let addresses =
    ["a"; "b"; "c"]
    |> List.map EmailAddress

let addresses' =
    addresses
    |> List.map (fun (EmailAddress e) -> e)
```

Сделать что-то подобное с помощью записей не выйдет.

Добавим типы-обёртки в наш код.
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

Приятное дополнение: реализация функций может быть инкапсулирована с помощью сигнатур модуля.
Что это такое и как работает, обсудим чуть позже.

## Наименование «варианта» одновариантного объединения

В примерах выше мы использовали одно и то же имя и для типа, и для его единственного варианта:

```fsharp
type EmailAddress = EmailAddress of string
type ZipCode = ZipCode of string
type StateCode = StateCode of string
```

Это немного путает, но на самом деле эти имена находятся в разных зонах видимости и не мешают друг другу.
Одно из них — это тип, а другое — функция-конструктор.

Если вы видите сигнатуру функции наподобие этой:

```fsharp
val f: string -> EmailAddress
```

знайте, что она относится к миру типов, поскольку `EmailAddress` — это имя типа.

С другой стороны, если вы видите такой код:

```fsharp
let x = EmailAddress y
```

знайте, что он относится к миру значений, поскольку `EmailAddress` — имя функции-конструктора.

## Конструируем одновариантные объединения

Часто для величин, имеющих особый смысл — электронных адресов, почтовых индексов — доступны далеко не все значения.
Не всякая строка может быть электронным адресом или почтовым индексом.

Это значит, что в какой-то момент нам потребуется валидация.
А какой момент может быть лучше, чем время создания переменной?
Помимо прочего, после создания переменную нельзя изменить, значит можно не опасаться, что кто-то модифицирует её в будущем.

Вот как расширить наш модуль с помщью валидирующих функций-конструкторов:

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

Протестируем конструкторы:

```fsharp
CreateStateCode "CA"
CreateStateCode "XX"

CreateEmailAddress "a@example.com"
CreateEmailAddress "example.com"
```

## Обрабатываем недопустимые входные данные в конструкторе

Когда у нас есть валидирующие функции-конструкторы, мы можем обрабатывать недопустимые входные данные.
Но как?
Например, как быть, если я вызову конструктор электронного адреса с параметром `"abc"`?

На этот вопрос есть несколько ответов.

Во-первых, можно выбросить исключение.
В функциональном мире так не делают и мы не будем.

Во-вторых, можно вернуть опциональное значение, где `None` означает, что входные данные не прошли валидацию.
Это именно то, что делают функции-конструкторы в примере, который мы только что привели.

Как правило, это простейший способ.
Он обладает преимуществом — вызывающая сторона должна явно обработать недопустимое значение.

Скажем, вызывающий код может выглядеть так:

```fsharp
match (CreateEmailAddress "a@example.com") with
| Some email -> ... // что-то делаем с email
| None -> ... // игнорируем?
```

Недостаток способа в том, что при сложных проверках непонятно, что пошло не так.
Был ли электронный адрес слишком длинным, или мы забыли символ `'@'`, или дело в неправильном домене?
Этого узнать нельзя.

Если вам нужны подробности, возвращайте тип с детальным объяснением ошибки.

Следующий пример возвращает тип `CreationResult`, чтобы сигнализировать об ошибке.

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

Вы передаёте в конструктор две функции.
Одна из них вызывается в случае успеха (получая, как параметр, сконструированный электронный адрес), а другая — в случае неудачи (получая, как параметр, описание ошибки).

```fsharp
type EmailAddress = EmailAddress of string

let CreateEmailAddressWithContinuations success failure (s:string) =
    if System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
        then success (EmailAddress s)
        else failure "Электронный адрес должен содержать символ @"
```

Успешная функция принимает на вход электронный адрес, а ошибочная — строку с ошибкой.
Обе функции должны вернуть один и тот же тип.
Его выбор остаётся за вами.

Простой пример — обе функции вызывают `printf` и не возвращают ничего (т.е. возвращают значение типа `unit`):

```fsharp
let success (EmailAddress s) = printfn "успешное создание электронного адреса %s" s
let failure  msg = printfn "ошибка при создании электронного адреса: %s" msg
CreateEmailAddressWithContinuations success failure "example.com"
CreateEmailAddressWithContinuations success failure "x@example.com"
```

С помощью функций-продолжений можно имитировать другие способы.
Например, можно получить опциональное значение.
В этом случае обе функции должны вернуть значение типа `EmailAddress option`:

```fsharp
let success e = Some e
let failure _  = None
CreateEmailAddressWithContinuations success failure "example.com"
CreateEmailAddressWithContinuations success failure "x@example.com"
```

Можно выбросить исключение в случае ошибки:

```fsharp
let success e = e
let failure _  = failwith "неверный электронный адрес"
CreateEmailAddressWithContinuations success failure "example.com"
CreateEmailAddressWithContinuations success failure "x@example.com"
```

Этот код кажется несколько громоздким.
На практике вместо длинной функции создают локальную функцию без последнего параметра.
Такие функции называют частично-применёнными.

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

## Создаём модули для типов-обёрток

Наши простые типы-обёртки стали сложнее, как только мы добавили валидацию.
Возможно, это не единственное усложнение, которое нам потребуется.

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

Теперь пользователи вызывают функции модуля, чтобы завернуть или развернуть значение:

```fsharp
// создаём электронные адреса
let address1 = EmailAddress.create "x@example.com"
let address2 = EmailAddress.create "example.com"

// разворачиваем электронные адреса
match address1 with
| Some e -> EmailAddress.value e |> printfn "the value is %s"
| None -> ()
```

## Заставляем вызывать конструктор

Осталась одна проблема: мы не можем заставить пользователя вызывать функцию-конструктор.
Если захочет, он может пропустить валидацию и создать тип напрямую.

На практике, это не так страшно.
Можно использовать соглашение об именовании, обозначив тип «якобы закрытым» и предоставив функции для «заворачивания» и «разворачивания».
Клиентам никогда не придётся взаимодействовать с типом напрямую.

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

Конечно, тип в модуле не закрыт по настоящему.
Но вы обозначаете вызывающей стороне своё намерение.

Если вы действительно хотите инкапсулировать тип и заставить вызывающую сторону использовать функции доступа, используйте сигнатуры модуля.

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

(Обратите внимание, что сигнатура модуля работают только в компилируемых проектах, а не в интерактивных скриптах.
Для тестирования вам понадобиться создать три файла в проекте F# с именами, показанными здесь.)

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

Тип `EmailAddress.T`, экспортируемый модулем сигнатур, не раскрывает детали реализации, поэтому клиенты не имеют доступа к его внутренностям.

Как видите, этот способ заставляет использовать конструктор.
Попытка создать тип напрямую (`T.EmailAddress "bad email"`) приведёт к ошибке компиляции.

## Когда следует «заворачивать» одновариантные объединения

Разобравшись с типами-обёртками, перейдём к следующему вопросу.
Когда заворачивать значения?

Обычно это делают на границах сервиса (например, на границах [гексагональной архитектуры](http://alistair.cockburn.us/Hexagonal+architecture)).

При таком подходе заворачивание выполняется на уровне UI или при загрузке данных из хранилища.
Созданный тип-обёртка передаётся на уровень предметной области, где обрабатывается как единое целое.
В рамках предметной области на удивление редко приходится извлекать завёрнутые значения, обрабатывать и заворачивать обратно.

В рамках конструирования критически важно, чтобы вызывающая сторона использовала функцию-конструктор, а не собственную логику проверки.
Это гарантирует, что «плохие» значения не проникнут в предметную область.

Вот, например, код, где UI выполняет собственную проверку:

```fsharp
let processFormSubmit () =
    let s = uiTextBox.Text
    if (s.Length < 50)
        then // присвоить электронный адрес объекту предметной области
        else // показать сообщение об ошибке
```

Гораздо лучше, если проверку сделает конструктор:

```fsharp
let processFormSubmit () =
    let emailOpt = uiTextBox.Text |> EmailAddress.create
    match emailOpt with
    | Some email -> // присвоить электронный адрес объекту предметной области
    | None -> // показать сообщение об ошибке
```

## Когда следует «разворачивать» одновариантные объединения

Хорошо, а когда требуется разворачивать значения?
И снова — в целом — на границах сервиса.

Например, когда вы сохраняете электронный адрес в базу данных, или связываете его с элементом UI или моделью представления.

Чтобы избежать явного разворачивания, можно использовать продолжения, передавая функцию, которая будет вызывана с развёрнутым значением.

Вместо явного вызова функции «разворачивания»:

```fsharp
address |> EmailAddress.value |> printfn "значение %s"
```

передаём функцию, которая применяется к завёрнутому значению:

```fsharp
address |> EmailAddress.apply (printfn "значение %s")
```

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

Функции `create` и `value` не являются обязательными, они добавлены для удобства вызывающей стороны.

## Код «в целом»

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

Обратите внимание, что в нашем примере много похожего кода в модулях с типами-обёртками.
Как думаете, можно ли избавиться от дублей или, по крайней мере, сделать код чище?

## Заключение

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
* Если завёрнутое значение требует проверки, предоставьте валидируцющие
  конструкторы и заставьте их вызывать.
* Ясно показывайте, что будет, если проверка завершится неудачей.
  В простых случаях возвращайте опциональные типы.
  В сложных случаях используйте функции-продолжения.
* Если у завёрнутого значения много связанных функций, поместите их
  в отдельный модуль.
* Если вам нужна инкапсуляция, используйте файлы сигнатур.

Однако, мы не закончили с рефакторингом.
В следующей части займёмся тем, чтобы сделать недопустимые состояния непредставимыми.

{{< linktarget "update" >}}

## Обновление

Многие читатели просят подробнее рассказать о том, как гарантировать создание типов навроде `EmailAddress`, только через конструктор с проверкой.
Для них я сделал небольшой [git-фрагмент](https://gist.github.com/swlaschin/54cfff886669ccab895a) с несколькими примерами.
