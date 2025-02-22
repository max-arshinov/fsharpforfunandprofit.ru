---
layout: post
# title: "Designing with types: Conclusion"
title: "Проектирование с помощью типов: Заключение"
# description: "A before and after comparison"
description: "Сравнение кода до и после"
date: 2013-01-19
nav: thinking-functionally
# seriesId: "Designing with types"
seriesId: "Проектирование с помощью типов"
seriesOrder: 8
categories: [Types, DDD]
---

> In this series, we've looked at some of the ways we can use types as part of the design process, including:

В этом цикле мы познакомились с некоторыми способами, с помощью которых мы можем использовать типы, как часть процесса проектирования, включая:

> * Breaking large structures down into small "atomic" components.
> * Using single case unions to add semantic meaning and validation to key domain types such `EmailAddress` and `ZipCode`.
> * Ensuring that the type system can only represent valid data ("making illegal states unrepresentable").
> * Using types as an analysis tool to uncover hidden requirements
> * Replacing flags and enums with simple state machines
> * Replacing primitive strings with types that guarantee various constraints

* Разбиение больших структур на маленькие «атомарные» компоненты.
* Использование одновариантных объединений для добавления семантического значения и валидации к ключевым типам предметной области, таким как `EmailAddress` и `ZipCode`.
* Гарантия, что система типов может представлять только корректные данные («делаем недопустимые состояния непредставимыми»).
* Использование типов как инструмента анализа для выявления скрытых требований.
* Замена флагов и перечислений простыми конечными автоматами.
* Замена примитивных строк типами, которые гарантируют соответствие различным ограничениям.

> For this final post, let's see them all applied together.

В этом последнем посте давайте рассмотрим их все вместе.

> ## The "before" code ##

## Код «до»

> Here's the original example we started off with in the [first post](/posts/designing-with-types-intro/) in the series:

Вот оригинальный пример, с которого мы начали в [первом посте](../designing-with-types-intro/) цикла^

```fsharp
type Contact =
    {
    FirstName: string;
    MiddleInitial: string;
    LastName: string;

    EmailAddress: string;
    //true if ownership of email address is confirmed
    //true, если приналдежность электронного адреса подтверждена
    IsEmailVerified: bool;

    Address1: string;
    Address2: string;
    City: string;
    State: string;
    Zip: string;
    //true if validated against address service
    //true, если проверен с помощью сервиса проверки адресов
    IsAddressValid: bool;
    }
```

> And how does that compare to the final result after applying all the techniques above?

И как это выглядит по сравнению с конечным результатом, полученным после применения всех техник, описанных выше?

> ## The "after" code ##

## Код «после»

> First, let's start with the types that are not application specific.
> These types could probably be reused in many applications.

Сначала давайте начнём с типов, которые не являются специфичными для приложения.
Вероятно, жти типы можно было бы повторно использовать во многих приложениях.

```fsharp
// ========================================
// WrappedString
// ========================================

/// Common code for wrapped strings
/// Общий код для обёрток над строками
module WrappedString =

    /// An interface that all wrapped strings support
    /// Интерфейс, который поддерживают все обёртки над строками
    type IWrappedString =
        abstract Value : string

    /// Create a wrapped value option
    /// 1) canonicalize the input first
    /// 2) If the validation succeeds, return Some of the given constructor
    /// 3) If the validation fails, return None
    /// Null values are never valid.
    /// Создать опциональное завёрнутое значение
    /// 1) Привести входные данные к каноническому виду
    /// 2) Если валидация прошла, вернуть Some от значения, которое вернул конструктор
    /// 3) Если валидация не прошла, вернуть None
    /// Значения null не считаются валидными.
    let create canonicalize isValid ctor (s:string) =
        if s = null
        then None
        else
            let s' = canonicalize s
            if isValid s'
            then Some (ctor s')
            else None

    /// Apply the given function to the wrapped value
    /// Применить данную функцию к завёрнутому значению
    let apply f (s:IWrappedString) =
        s.Value |> f

    /// Get the wrapped value
    /// Взять завёрнутое значение
    let value s = apply id s

    /// Equality
    /// Равенство
    let equals left right =
        (value left) = (value right)

    /// Comparison
    /// Сравнение
    let compareTo left right =
        (value left).CompareTo (value right)

    /// Canonicalizes a string before construction
    /// * converts all whitespace to a space char
    /// * trims both ends
    /// Приводим строку к каноническому виду перед вызовом конструктора
    /// * конвертируем все пробельные символы в пробелы
    /// * обрезаем слева и справа
    let singleLineTrimmed s =
        System.Text.RegularExpressions.Regex.Replace(s,"\s"," ").Trim()

    /// A validation function based on length
    /// Функция валиадации на основе длины строки
    let lengthValidator len (s:string) =
        s.Length <= len

    /// A string of length 100
    /// Строка длины 100
    type String100 = String100 of string with
        interface IWrappedString with
            member this.Value = let (String100 s) = this in s

    /// A constructor for strings of length 100
    /// Конструктор строк длины 100
    let string100 = create singleLineTrimmed (lengthValidator 100) String100

    /// Converts a wrapped string to a string of length 100
    /// Конвертирует обёртку над строками в строку длины 100
    let convertTo100 s = apply string100 s

    /// A string of length 50
    /// Строка длины 50
    type String50 = String50 of string with
        interface IWrappedString with
            member this.Value = let (String50 s) = this in s

    /// A constructor for strings of length 50
    /// Конструктор строк длины 50
    let string50 = create singleLineTrimmed (lengthValidator 50)  String50

    /// Converts a wrapped string to a string of length 50
    /// Конвертирует обёртку над строками в строку длины 50
    let convertTo50 s = apply string50 s

    /// map helpers
    /// Вспомогательные функции для словарей
    let mapAdd k v map =
        Map.add (value k) v map

    let mapContainsKey k map =
        Map.containsKey (value k) map

    let mapTryFind k map =
        Map.tryFind (value k) map

// ========================================
// Email address (not application specific)
// Электронные адреса (код, не специфичный для приложения)
// ========================================

module EmailAddress =

    type T = EmailAddress of string with
        interface WrappedString.IWrappedString with
            member this.Value = let (EmailAddress s) = this in s

    let create =
        let canonicalize = WrappedString.singleLineTrimmed
        let isValid s =
            (WrappedString.lengthValidator 100 s) &&
            System.Text.RegularExpressions.Regex.IsMatch(s,@"^\S+@\S+\.\S+$")
        WrappedString.create canonicalize isValid EmailAddress

    /// Converts any wrapped string to an EmailAddress
    /// Конвертиуем любую обёртку над строками в EmailAddress
    let convert s = WrappedString.apply create s

// ========================================
// ZipCode (not application specific)
// Почтовый индекс (код, не специфичный для приложения)
// ========================================

module ZipCode =

    type T = ZipCode of string with
        interface WrappedString.IWrappedString with
            member this.Value = let (ZipCode s) = this in s

    let create =
        let canonicalize = WrappedString.singleLineTrimmed
        let isValid s =
            System.Text.RegularExpressions.Regex.IsMatch(s,@"^\d{5}$")
        WrappedString.create canonicalize isValid ZipCode

    /// Converts any wrapped string to a ZipCode
    /// Конвертиуем любую обёртку над строками в ZipCode
    let convert s = WrappedString.apply create s

// ========================================
// StateCode (not application specific)
// Код штата (код, не специфичный для приложения)
// ========================================

module StateCode =

    type T = StateCode  of string with
        interface WrappedString.IWrappedString with
            member this.Value = let (StateCode  s) = this in s

    let create =
        let canonicalize = WrappedString.singleLineTrimmed
        let stateCodes = ["AZ";"CA";"NY"] // и т. д.
        let isValid s =
            stateCodes |> List.exists ((=) s)

        WrappedString.create canonicalize isValid StateCode

    /// Converts any wrapped string to a StateCode
    /// Конвертиуем любую обёртку над строками в StateCode
    let convert s = WrappedString.apply create s

// ========================================
// PostalAddress (not application specific)
// Почтовый адрес (код, не специфичный для приложения)
// ========================================

module PostalAddress =

    type USPostalAddress =
        {
        Address1: WrappedString.String50;
        Address2: WrappedString.String50;
        City: WrappedString.String50;
        State: StateCode.T;
        Zip: ZipCode.T;
        }

    type UKPostalAddress =
        {
        Address1: WrappedString.String50;
        Address2: WrappedString.String50;
        Town: WrappedString.String50;
        PostCode: WrappedString.String50;   // доделать
        }

    type GenericPostalAddress =
        {
        Address1: WrappedString.String50;
        Address2: WrappedString.String50;
        Address3: WrappedString.String50;
        Address4: WrappedString.String50;
        Address5: WrappedString.String50;
        }

    type T =
        | USPostalAddress of USPostalAddress
        | UKPostalAddress of UKPostalAddress
        | GenericPostalAddress of GenericPostalAddress

// ========================================
// PersonalName (not application specific)
// Личное имя (код, не специфичный для приложения)
// ========================================

module PersonalName =
    open WrappedString

    type T =
        {
        FirstName: String50;
        MiddleName: String50 option;
        LastName: String100;
        }

    /// create a new value
    /// создать новое значение
    let create first middle last =
        match (string50 first),(string100 last) with
        | Some f, Some l ->
            Some {
                FirstName = f;
                MiddleName = (string50 middle)
                LastName = l;
                }
        | _ ->
            None

    /// concat the names together
    /// and return a raw string
    /// склеить вместе имя и фамилию
    /// и вернуть простую строку
    let fullNameRaw personalName =
        let f = personalName.FirstName |> value
        let l = personalName.LastName |> value
        let names =
            match personalName.MiddleName with
            | None -> [| f; l |]
            | Some middle -> [| f; (value middle); l |]
        System.String.Join(" ", names)

    /// concat the names together
    /// and return None if too long
    /// склеить вместе имя и фамилию
    /// и вернуть None, если слишком длинная строка
let fullNameOption personalName =
        personalName |> fullNameRaw |> string100

    /// concat the names together
    /// and truncate if too long
    /// склеить вместе имя и фамилию
    /// и усечь если слишком длинная строка
    let fullNameTruncated personalName =
        // helper function
        // вспомогательная функция
        let left n (s:string) =
            if (s.Length > n)
            then s.Substring(0,n)
            else s

        personalName
        |> fullNameRaw  // склеиваем concat
        |> left 100     // усекаем truncate
        |> string100    // заворачиваем wrap
        |> Option.get   // всё вместе даёт результат без ошибок this will always be ok
```

> And now the application specific types.

А теперь типы специфичные для приложения.

```fsharp

// ========================================
// EmailContactInfo -- state machine
// EmailContactInfo -- конечный автомат
// ========================================

module EmailContactInfo =
    open System

    // UnverifiedData = just the EmailAddress
    // UnverifiedData = просто EmailAddress
    type UnverifiedData = EmailAddress.T

    // VerifiedData = EmailAddress plus the time it was verified
    // VerifiedData = EmailAddress плюс дата/время проверки
    type VerifiedData = EmailAddress.T * DateTime

    // set of states
    // множество состояний
    type T =
        | UnverifiedState of UnverifiedData
        | VerifiedState of VerifiedData

    let create email =
        // unverified on creation
        // непроверенный при создании
        UnverifiedState email

    // handle the "verified" event
    // обработать событие "проверен"
    let verified emailContactInfo dateVerified =
        match emailContactInfo with
        | UnverifiedState email ->
            // construct a new info in the verified state
            // конструируем новый объект в проверенном состоянии
            VerifiedState (email, dateVerified)
        | VerifiedState _ ->
            // ignore
            // игнорируем
            emailContactInfo

    let sendVerificationEmail emailContactInfo =
        match emailContactInfo with
        | UnverifiedState email ->
            // send email
            // отправляем письмо
            printfn "отправка письма"
        | VerifiedState _ ->
            // do nothing
            // ничего не делаем
            ()

    let sendPasswordReset emailContactInfo =
        match emailContactInfo with
        | UnverifiedState email ->
            // ignore
            // игнорируем
            ()
        | VerifiedState _ ->
            // ignore
            // игнорируем
            printfn "отправка запроса за сброс пароля"

// ========================================
// PostalContactInfo -- state machine
// PostalContactInfo -- конечный автомат
// ========================================

module PostalContactInfo =
    open System

    // InvalidData = просто PostalAddress
    type InvalidData = PostalAddress.T

    // ValidData = PostalAddress плюс дата/время валидации
    type ValidData = PostalAddress.T * DateTime

    // set of states
    // множество состояний
    type T =
        | InvalidState of InvalidData
        | ValidState of ValidData

    let create address =
        // invalid on creation
        // неправильный при создании
        InvalidState address

    // handle the "validated" event
    // обрабатываем событие "проверен"
    let validated postalContactInfo dateValidated =
        match postalContactInfo with
        | InvalidState address ->
            // construct a new info in the valid state
            // конструируем новый объект в проверенном состоянии
            ValidState (address, dateValidated)
        | ValidState _ ->
            // ignore
            // игнорируем
            postalContactInfo

    let contactValidationService postalContactInfo =
        let dateIsTooLongAgo (d:DateTime) =
            d < DateTime.Today.AddYears(-1)

        match postalContactInfo with
        | InvalidState address ->
            printfn "соединяемся с сервисом проверки адресов (contacting the address validation service)"
        | ValidState (address,date) when date |> dateIsTooLongAgo  ->
            printfn "последняя проверка была слишком давно (last checked a long time ago)"
            printfn "снова соединяемся с сервисом проверки адресов (contacting the address validation service again)"
        | ValidState  _ ->
            printfn "недавно проверен, ничего не делаем (recently checked. Doing nothing)"

// ========================================
// ContactMethod and Contact
// ContactMethod и Contact
// ========================================

type ContactMethod =
    | Email of EmailContactInfo.T
    | PostalAddress of PostalContactInfo.T

type Contact =
    {
    Name: PersonalName.T;
    PrimaryContactMethod: ContactMethod;
    SecondaryContactMethods: ContactMethod list;
    }

```

{{< book_page_ddd_img >}}


> ## Conclusion ##

> Phew!
> The new code is much, much longer than the original code.
> Granted, it has a lot of supporting functions that were not needed in the original version, but even so it seems like a lot of extra work.
> So was it worth it?

Уф!
Новый код гораздо, гораздо длиннее, чем оригинальный код.
Конечно, в нём много поддерживающих функций, которые не были нужны в оригинальной версии, но даже так кажется, что он потребовал много дополнительной работы.
Так стоило ли оно того?

> I think the answer is yes.
> Here are some of the reasons why:

Я думаю, что ответ: да.
Вот несколько причин, почему:

> **The new code is more explicit**

**Новый код более явный**

> If we look at the original example, there was no atomicity between fields, no validation rules, no length constraints, nothing to stop you updating flags in the wrong order, and so on.

Если мы посмотрим на оригинальный пример, там не было ни атомарности полей, ни правил валидации, ни ограничений длины, ничего, что помешало бы вам обновить флаги в неправильном порядке, и т. д.

> The data structure was "dumb" and all the business rules were implicit in the application code.
> Chances are that the application would have lots of subtle bugs that might not even show up in unit tests.
> (*Are you sure the application reset the `IsEmailVerified` flag to false in every place the email address was updated?*)

Структура данных была «тупая» и все бизнес-правила в коде приложения были неявными.
Все шансы, что в приложении будет множество коварных ошибок, которые могут даже не проявиться в модульных тестах.
(*Вы уверены, что приложение сбрасывает флаг `IsEmailVerified` во всех местах, где электронный адрес изменяется?*)

> On the other hand, the new code is extremely explicit about every little detail.
> If I stripped away everything but the types themselves, you would have a very good idea of what the business rules and domain constraints were.

С другой стороны, новый код предельно ясен в отношении каждой мельчайшей детали.
Если бы я удалил всё, кроме непосредственно типов, вы бы <!-- всё равно --> получили хорошее представление о бизнес-правилах и ограничениях предметной области.

> **The new code won't let you postpone error handling**

**Новый код не разрешает откладывать обработку ошибок**

> Writing code that works with the new types means that you are forced to handle every possible thing that could go wrong, from dealing with a name that is too long, to failing to supply a contact method.
> And you have to do this up front at construction time.
> You can't postpone it till later.

Написание кода, который работает с новыми типами, означает, что вам придётся учесть всё, что могло пойти не так, от слишком длинного имени до отсутствия способа связи.
И это нужно сделать заранее, во время конструирования.
Вы не можете отложить это на потом.

> Writing such error handling code can be annoying and tedious, but on the other hand, it pretty much writes itself.
> There is really only one way to write code that actually compiles with these types.

Написание этого кода обработки ошибок может быть раздражающим и скучным, но, с другой стороны, он почти что пишет сам себя.
На самом деле, есть только один способ написать код, который действительно будет компилироваться с этими типами.

> **The new code is more likely to be correct**

**Новый код, скорее всего, будет правильным**

> The *huge* benefit of the new code is that it is probably bug free.
> Without even writing any unit tests, I can be quite confident that a first name will never be truncated when written to a `varchar(50)` in a database, and that I can never accidentally send out a verification email twice.

*Огромное* преимущество нового кода в том, что в нём, вероятно, нет ошибок.
Даже без написания модульных тестов, я могу быть достаточно уверен, что имя никогда не будет усечено до `varchar(50)` при записи в базу, и что я никогда случайно не отправлю письмо для проверки адреса дважды.

> And in terms of the code itself, many of the things that you as a developer have to remember to deal with (or forget to deal with) are completely absent.
> No null checks, no casting, no worrying about what the default should be in a `switch` statement.
> And if you like to use cyclomatic complexity as a code quality metric, you might note that there are only three `if` statements in the entire 350 odd lines.

А что касается самого кода, многие из вещей, которые вам, как разработчику приходится помнить (или забыть), полностью отсутствуют.
Ни проверок на `null`, ни приведения типов, ни беспокоства о том, что доложно быть в ветке `default` оператора `switch`.
И если вам нравиться использовать цикломатическую сложность, как метрику качества кода, вы можете заметить, что здесь всего три оператора `if` на 350 с хвостиком строк.

> **A word of warning...**

**Слово предостережения...** <!-- это немного высокопаный стиль, поэтому предостережения, а не предупреждения -->

> Finally, beware!
> Getting comfortable with this style of type-based design will have an insidious effect on you.
> You will start to develop paranoia whenever you see code that isn't typed strictly enough.
> (*How long should an email address be, exactly?*) and you will be unable to write the simplest python script without getting anxious.
> When this happens, you will have been fully inducted into the cult.
> Welcome!

В заключение, будьте осторожны!
Освоение этого стиля проектирования, основанного на типах, окажет на вас коварное воздействие.
У вас насчнёт разиваться паранойя, всякий раз, когда вы встретите код, который не достаточно строго типизирован
(*Какой точно длины должен быть электронный адрес?*), и вы не сможете написать простейший скрипт на Python, не испытывая при этом беспокойства.
Когда это произойдёт, вы станете полноправным членом культа.
Добро пожаловать!

> *If you liked this series, here is a slide deck that covers many of the same topics. There is [a video as well (here)](/ddd/)*

*Если вам понравился этот цикл, взгляните на слайды, где затронуты многие из этих тем. Есть [также видео (здесь)](/ddd/)*

{{< slideshare A4ay4HQqJgu0Q "domain-driven-design-with-the-f-type-system-functional-londoners-2014" "Domain Driven Design with the F# type System -- F#unctional Londoners 2014" >}}

