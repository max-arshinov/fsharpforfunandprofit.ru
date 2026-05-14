---
layout: post
title: "Проектирование с помощью типов: Заключение"
description: "Сравнение кода до и после"
date: 2013-01-19
nav: thinking-functionally
seriesId: "Проектирование с помощью типов"
seriesOrder: 8
categories: [Types, DDD]
---

В этом цикле мы узнали несколько способов использовать типы, как часть процесса проектирования.
В частности:

* Разбиение больших структур на маленькие «атомарные» компоненты.
* Использование одновариантных объединений для добавления семантического значения и валидации к типам предметной области, наподобие `EmailAddress` и `ZipCode`.
* Понимание, что система типов может представлять только корректные данные («делаем недопустимые состояния не представимыми»).
* Использование типов как инструмента анализа для выявления скрытых требований.
* Замена флагов и перечислений простыми конечными автоматами.
* Замена примитивных строк типами, которые гарантируют соответствие различным ограничениям.

В последнем посте цикла, давайте рассмотрим их вместе.

## Код «до»

Оригинальный пример, с которого мы начали в [первом посте](../designing-with-types-intro/) цикла:

```fsharp
type Contact =
    {
    FirstName: string;
    MiddleInitial: string;
    LastName: string;

    EmailAddress: string;
    //true, если электронный адрес подтверждён
    IsEmailVerified: bool;

    Address1: string;
    Address2: string;
    City: string;
    State: string;
    Zip: string;
    //true, если проверен с помощью сервиса проверки адресов
    IsAddressValid: bool;
    }
```

Как он выглядит по сравнению с конечным результатом, полученным после применения всех техник, описанных выше?

## Код «после»

Во-первых, давайте начнём с типов, которые не являются специфичными для приложения.
Эти типы можно использовать повторно в других приложениях.

```fsharp
// ========================================
// WrappedString
// ========================================

/// Общий код для обёрток над строками
module WrappedString =

    /// Интерфейс, который поддерживают все обёртки над строками
    type IWrappedString =
        abstract Value : string

    /// Создаёт опциональное завёрнутое значение
    /// 1) Приводит входные данные к каноническому виду
    /// 2) Если валидация прошла, возвращает Some результата конструктора
    /// 3) Если валидация не прошла, возвращает None
    /// Значения null не считаются валидными.
    let create canonicalize isValid ctor (s:string) =
        if s = null
        then None
        else
            let s' = canonicalize s
            if isValid s'
            then Some (ctor s')
            else None

    /// Применяет функцию к завёрнутому значению
    let apply f (s:IWrappedString) =
        s.Value |> f

    /// Возвращает завёрнутое значение
    let value s = apply id s

    /// Равенство
    let equals left right =
        (value left) = (value right)

    /// Сравнение
    let compareTo left right =
        (value left).CompareTo (value right)

    /// Приводит строку к каноническому виду перед вызовом конструктора
    /// * конвертирует все пробельные символы в пробелы
    /// * обрезает слева и справа
    let singleLineTrimmed s =
        System.Text.RegularExpressions.Regex.Replace(s,"\s"," ").Trim()

    /// Функция валиадации на основе длины строки
    let lengthValidator len (s:string) =
        s.Length <= len

    /// Строка длины 100
    type String100 = String100 of string with
        interface IWrappedString with
            member this.Value = let (String100 s) = this in s

    /// Конструктор строк длины 100
    let string100 = create singleLineTrimmed (lengthValidator 100) String100

    /// Конвертирует обёртку над строками в строку длины 100
    let convertTo100 s = apply string100 s

    /// Строка длины 50
    type String50 = String50 of string with
        interface IWrappedString with
            member this.Value = let (String50 s) = this in s

    /// Конструктор строк длины 50
    let string50 = create singleLineTrimmed (lengthValidator 50)  String50

    /// Конвертирует обёртку над строками в строку длины 50
    let convertTo50 s = apply string50 s

    /// Вспомогательные функции для словарей
    let mapAdd k v map =
        Map.add (value k) v map

    let mapContainsKey k map =
        Map.containsKey (value k) map

    let mapTryFind k map =
        Map.tryFind (value k) map

// ========================================
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

    /// Конвертиует любую обёртку над строками в EmailAddress
    let convert s = WrappedString.apply create s

// ========================================
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

    /// Конвертиует любую обёртку над строками в ZipCode
    let convert s = WrappedString.apply create s

// ========================================
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

    /// Конвертиует любую обёртку над строками в StateCode
    let convert s = WrappedString.apply create s

// ========================================
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

    /// Создаёт новое значение
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

    /// Склеивает вместе имя и фамилию
    /// и возвращает простую строку
    let fullNameRaw personalName =
        let f = personalName.FirstName |> value
        let l = personalName.LastName |> value
        let names =
            match personalName.MiddleName with
            | None -> [| f; l |]
            | Some middle -> [| f; (value middle); l |]
        System.String.Join(" ", names)

    /// Склеивает вместе имя и фамилию
    /// и возвращает None, если слишком длинная строка
let fullNameOption personalName =
        personalName |> fullNameRaw |> string100

    /// Склеивает вместе имя и фамилию
    /// и обрезает слишком длинную строку
    let fullNameTruncated personalName =
        // вспомогательная функция
        let left n (s:string) =
            if (s.Length > n)
            then s.Substring(0,n)
            else s

        personalName
        |> fullNameRaw  // склеиваем
        |> left 100     // обрезаем
        |> string100    // заворачиваем
        |> Option.get   // всё вместе даёт результат без ошибок
```

А теперь — типы, специфичные для приложения.

```fsharp

// ========================================
// EmailContactInfo -- конечный автомат
// ========================================

module EmailContactInfo =
    open System

    // UnverifiedData = просто EmailAddress
    type UnverifiedData = EmailAddress.T

    // VerifiedData = EmailAddress плюс дата/время проверки
    type VerifiedData = EmailAddress.T * DateTime

    // множество состояний
    type T =
        | UnverifiedState of UnverifiedData
        | VerifiedState of VerifiedData

    let create email =
        // непроверенный при создании
        UnverifiedState email

    // обработать событие "проверен"
    let verified emailContactInfo dateVerified =
        match emailContactInfo with
        | UnverifiedState email ->
            // конструируем новый объект в проверенном состоянии
            VerifiedState (email, dateVerified)
        | VerifiedState _ ->
            // игнорируем
            emailContactInfo

    let sendVerificationEmail emailContactInfo =
        match emailContactInfo with
        | UnverifiedState email ->
            // отправляем письмо
            printfn "отправка письма"
        | VerifiedState _ ->
            // ничего не делаем
            ()

    let sendPasswordReset emailContactInfo =
        match emailContactInfo with
        | UnverifiedState email ->
            // игнорируем
            ()
        | VerifiedState _ ->
            printfn "отправка запроса за сброс пароля"

// ========================================
// PostalContactInfo -- конечный автомат
// ========================================

module PostalContactInfo =
    open System

    // InvalidData = просто PostalAddress
    type InvalidData = PostalAddress.T

    // ValidData = PostalAddress плюс дата/время валидации
    type ValidData = PostalAddress.T * DateTime

    // множество состояний
    type T =
        | InvalidState of InvalidData
        | ValidState of ValidData

    let create address =
        // непроверенный при создании
        InvalidState address

    // обрабатываем событие "проверен"
    let validated postalContactInfo dateValidated =
        match postalContactInfo with
        | InvalidState address ->
            // конструируем новый объект в проверенном состоянии
            ValidState (address, dateValidated)
        | ValidState _ ->
            // игнорируем
            postalContactInfo

    let contactValidationService postalContactInfo =
        let dateIsTooLongAgo (d:DateTime) =
            d < DateTime.Today.AddYears(-1)

        match postalContactInfo with
        | InvalidState address ->
            printfn "соединяемся с сервисом проверки адресов"
        | ValidState (address,date) when date |> dateIsTooLongAgo  ->
            printfn "последняя проверка была слишком давно"
            printfn "снова соединяемся с сервисом проверки адресов"
        | ValidState  _ ->
            printfn "недавно проверен, ничего не делаем"

// ========================================
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

## Заключение

Уф!
Новый код гораздо, гораздо длиннее оригинального.
Да, в нём много вспомогательных функций, которых не было в оригинальном коде, но всё равно кажется, что мы потратили на него слишком много сил.
Стоила ли овчинка выделки?

Я думаю, ответ — да.
И вот почему:

**Новый код явно выражает намерения разработчика**

Структура данных была «тупой», а бизнес-правила — неявными.
Все шансы получить множество коварных ошибок, которые даже не проявятся в модульных тестах.
(*Вы уверены, что приложение сбрасывает флаг `IsEmailVerified` везде, где меняется электронный адрес?*)

С другой стороны, новый код предельно ясен в отношении деталей.
Если бы я удалил всё, кроме типов, вы бы всё равно получили чёткое представление о бизнес-правилах и ограничениях предметной области.

**Новый код не разрешает откладывать обработку ошибок**

Написание кода в новом стиле, означает, что вам приходится учитывать всё, что может пойти не так: от слишком длинного имени до отсутствия способа связи.
И это надо делать заранее, при создании объектов.
Вы не можете отложить проверку на потом.

Обработка ошибок может быть скучной и раздражающей, но, с другой стороны, код почти что пишет сам себя.
Потому что есть единственный способ написать код, который действительно будет компилироваться с вашими типами.

**Новый код, скорее всего, будет правильным**

*Огромное* преимущество нового кода в том, что в нём гораздо меньше ошибок.
Даже без написания модульных тестов я уверен, что имя при записи в базу не будет усечено до `varchar(50)`, и что я — даже случайно — не отправлю два раза письмо для проверки адреса.

Что касается самого кода, в нём нет многого из того, что вам, как разработчику, приходится помнить.
Ни проверок на `null`, ни приведения типов, ни беспокойства о том, что должно быть в ветке `default` оператора `switch`.
И если вам нравиться использовать цикломатическую сложность, как метрику качества кода, обратите внимание, что в новом коде всего три оператора `if` на 350 с лишним строк.

**Слова предостережения...**

Будьте осторожны!
Стиль проектирования, основанный на типах, окажет на вас коварное воздействие.
Всякий раз, когда вы встретите недостаточно типизированный код, у вас будет возникать паранойя.
(*Какой точно длины должен быть электронный адрес?*)
Вы не сможете написать простейший скрипт на Python, не испытывая беспокойства.
После этого вы станете полноправным членом культа.
Добро пожаловать!

*Если вам понравился этот цикл, взгляните на слайды, где затронуты многие из этих тем. Есть [также видео (здесь)](/ddd/)*

{{< slideshare A4ay4HQqJgu0Q "domain-driven-design-with-the-f-type-system-functional-londoners-2014" "Domain Driven Design with the F# type System -- F#unctional Londoners 2014" >}}

