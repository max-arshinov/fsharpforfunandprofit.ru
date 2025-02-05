---
layout: post
# title: "Designing with types: Constrained strings"
title: "Проектирование с помощью типов: Ограниченные строки" # так, кажется, не говорят, можно расшифровать: Накладываем ограничения на строки
# description: "Adding more semantic information to a primitive type"
description: "Добавляем больше семантической информации к примитивному типу"
date: 2013-01-17
nav: thinking-functionally
# seriesId: "Designing with types"
seriesId: "Проектирование с помощью типов"
seriesOrder: 6
categories: [Types, DDD]
---

> In a [previous post](/posts/designing-with-types-single-case-dus/), I talked about avoiding using plain primitive strings for email addresses, zip codes, states, etc.
> By wrapping them in a single case union, we could (a) force the types to be distinct and (b) add validation rules.

В [предыдущем посте](/../designing-with-types-single-case-dus/) я говорил о том, что надо избегать использования простых примитивных строк для хранения электронных адресов, почтовых индексов, состояний и т. д.
Оборачивая их в одновариантное объединение мы смогли а) сделать типы ясно различимыми и б) добавить правила валидации.

> In this post, we'll look at whether we can extend that concept to an even more fine grained level.

В этом посте мы узнаем, сможем ли мы расширить эту концепцию до ещё более детального уровня.

> ## When is a string not a string?

## Когда строка — не строка?

> Let's look a simple `PersonalName` type.

Давайте взглянем на простой тип `PersonalName`:

```fsharp
type PersonalName =
    {
    FirstName: string;
    LastName: string;
    }
```

> The type says that the first name is a `string`.
> But really, is that all it is?
> Are there any other constraints that we might need to add to it?

Тип утверждает, что личное имя сотрудника — это `string`.
Но разве это всё, на самом деле?
Есть ли другие ограничения, которые нам может понадобиться наложить на него?

> Well, OK, it must not be null.
> But that is assumed in F#.

Да, хорошо, оно не должно принимать значение `null`.
Но это и так предполагается в F#.

> What about the length of the string?
> Is it acceptable to have a name which is 64K characters long?
> If not, then is there some maximum length allowed?

Что насчёт длины строки?
Допустимо ли иметь имя длиной 64К символов?
Если нет, то существует ли какая-то максимально допустимая длина?

> And can a name contain linefeed characters or tabs?
> Can it start or end with whitespace?

И может ли имя содержать символы перевода строки или табуляции?
Может ли оно начинаться или заканчиваться пробельным символом?

> Once you put it this way, there are quite a lot of constraints even for a "generic" string.
> Here are some of the obvious ones:

Если так подумать, существует довольно много ограничений даже для «просто» строк.
Вот некоторые из очевидных:

> * What is its maximum length?
> * Can it cross over multiple lines?
> * Can it have leading or trailing whitespace?
> * Can it contain non-printing characters?

* Какова её максимальная длина?
* Может ли она содержать переводы строки?
* Может ли она начинаться или заканчиваться пребельным символом?
* Может ли она содержать символы, которые нельзя напечатать?

> ## Should these constraints be part of the domain model?

## Должны ли эти ограничения быть частью предметной области?

> So we might acknowledge that some constraints exist, but should they really be part of the domain model (and the corresponding types derived from it)?
> For example, the constraint that a last name is limited to 100 characters -- surely that is specific to a particular implementation and not part of the domain at all.

Итак, мы можем сознавать, что есть какие-то ограничения, но действительно ли они должны быть частью доменной модели (и соответсвующих типов, которые из неё вытекают)?
Например, ограничение, что фамилия ограничения 100 символами — конечно, относится к конкретной реализации, а не к предметной области в целом.

> I would answer that there is a difference between a logical model and a physical model.
> In a logical model some of these constraints might not be relevant, but in a physical model they most certainly are.
> And when we are writing code, we are always dealing with a physical model anyway.

Я мог бы ответить, что существует различие между логической моделью и физической моделью.
В логической модели некоторые из этих ограничений могут быть неважными, но в физической модели они, безусловно, важны.
А когда мы пишем код, мы в любом случае имеем дело с физической моделью.

> Another reason for incorporating the constraints into the model is that often the model is shared across many separate applications.
> For example, a personal name may be created in a e-commerce application, which writes it into a database table and then puts it on a message queue to be picked up by a CRM application, which in turn calls an email templating service, and so on.

Другая причина для включения ограничений в модель заключается в том, что часто модель совместно используется многими отдельными приложениями.
Например, личное имя может быть создано в приложении электронной коммерции, которое записывает его в таблицу базы данных и затем складываем его в очередь сообщений для обработки приложением CRM, которое в свою очередь вызывает службу шаблонов электронной почты и т. д.

> It is important that all these applications and services have the *same* idea of what a personal name is, including the length and other constraints.
> If the model does not make the constraints explicit, then it is easy to have a mismatch when moving across service boundaries.

Важно, что все эти приложения и сервисы имеют *общую* идею, что такое личное имя, включая длину и другие ограничения.
Если модель не делает ограничения явными, тогда легко получить несовпадение при пересечении границ сервиса.

> For example, have you ever written code that checks the length of a string before writing it to a database?

Например, вы когда-нибудь писали код, который проверяет длину строки перед записью её в базу?

```csharp
void SaveToDatabase(PersonalName personalName)
{
   var first = personalName.First;
   if (first.Length > 50)
   {
        // ensure string is not too long
        // убедиться, что строка не слишком длинная
        first = first.Substring(0,50);
   }

   //save to database
   //сохранить в базу
}
```

> If the string *is* too long at this point, what should you do?
> Silently truncate it?
> Throw an exception?

Если в этом месте строка *оказалась* слишком длинной, что вы должны сделать?
Молча обрезать её?
Бросить исключение?

> A better answer is to avoid the problem altogether if you can.
> By the time the string gets to the database layer it is too late -- the database layer should not be making these kinds of decisions.

Лучший ответ — вообще избежать такой проблемы, если это возможно.
К тому времени, когда строка попадает на уровень базы данных уже слишком поздно — уровень базы данных не должен принимать решений такого типа.

> The problem should be dealt with when the string was *first created*, not when it is *used*.
> In other words, it should have been part of the validation of the string.

Проблема должна быть решена, когда строка была *создана впервые*, а не когда она уже *используется*.
Другими словами, это должно быть частью валидации строки.

> But how can we trust that the validation has been done correctly for all possible paths?
> I think you can guess the answer...

Но как мы можем быть уверены, что валидация была выполнена корректно для всех возможных путей?
Я думаю вы можете догадаться об ответе...

> ## Modeling constrained strings with types

## Моделируем огнариченные строки с помощью типов

> The answer, of course, is to create wrapper types which have the constraints built into the type.

Ответ, конечно, в том, чтобы создать типы-обёртки, которые имеют встроенные в тип ограничения.

> So let's knock up a quick prototype using the single case union technique we used [before](/posts/designing-with-types-single-case-dus/).

Давайте быстро накидаем прототип, использую технику одновариантных объединений, котору мы использовали [раньше](../designing-with-types-single-case-dus/).


```fsharp
module String100 =
    type T = String100 of string
    let create (s:string) =
        if s <> null && s.Length <= 100
        then Some (String100 s)
        else None
    let apply f (String100 s) = f s
    let value s = apply id s

module String50 =
    type T = String50 of string
    let create (s:string) =
        if s <> null && s.Length <= 50
        then Some (String50 s)
        else None
    let apply f (String50 s) = f s
    let value s = apply id s

module String2 =
    type T = String2 of string
    let create (s:string) =
        if s <> null && s.Length <= 2
        then Some (String2 s)
        else None
    let apply f (String2 s) = f s
    let value s = apply id s
```

> Note that we immediately have to deal with the case when the validation fails by using an option type as the result.
> It makes creation more painful, but we can't avoid it if we want the benefits later.

Обратите внимание, что мы сразу имеем дело со случаем, когда валидация закончилась неудачей, используя опциональный тип в качестве результата.
Это делает создание более болезненным, но мы не можем его избежать, если хотим получить преимущества в дальнейшем.

> For example, here is a good string and a bad string of length 2.

Например, вот хорошая строка и плохая строка длины 2.

```fsharp
let s2good = String2.create "CA"
let s2bad = String2.create "California"

match s2bad with
| Some s2 -> // обновляем объект предметной области (update domain object)
| None -> // обрабатываем ошибку (handle error)
```

> In order to use the `String2` value we are forced to check whether it is `Some` or `None` at the time of creation.

Для того, чтобы использовать значение `String2` мы вынуждены проверять, является оно `Some` или `None` на момент создания.

> ### Problems with this design

### Проблемы такого дизайна

> One problem is that we have a lot of duplicated code.
> In practice a typical domain only has a few dozen string types, so there won't be that much wasted code.
> But still, we can probably do better.

Одна их проблем заключается в том, что у нас появляется много повторяющегося кода.
На практике типичная предметная область имеет всего несколько десятков строковых типов, так что будет не так много лишнего кода.
Но всё же мы, возможно, придумаем что-то получше.

> Another more serious problem is that comparisons become harder.
> A `String50` is a different type from a `String100` so that they cannot be compared directly.

Другая более серьёзная проблема заключается в том, что сравние становится сложнее.
Тип `String50` отличен от типа `String100`, так что их нельзя сравнивать напрямую.

```fsharp
let s50 = String50.create "John"
let s100 = String100.create "Smith"

let s50' = s50.Value
let s100' = s100.Value

let areEqual = (s50' = s100')  // ошибка компилятора (compiler error)
```

> This kind of thing will make working with dictionaries and lists harder.

Подобные вещи усложняют работу со словарями и списками.

{{< book_page_pdf >}}

> ### Refactoring

### Рефакторинг

> At this point we can exploit F#'s support for interfaces, and create a common interface that all wrapped strings have to support, and also some standard functions:

На этом этапе мы можем экспулатировать поддержку интерфейсов в F# и создать общий интерфейс, который должны поддерживать все завёрнутые строки, а также некоторые стандартные функции:

```fsharp
module WrappedString =

    /// An interface that all wrapped strings support
    /// Интерфейс, который поддерживают все завёрнутые строки
    type IWrappedString =
        abstract Value : string

    /// Create a wrapped value option
    /// 1) canonicalize the input first
    /// 2) If the validation succeeds, return Some of the given constructor
    /// 3) If the validation fails, return None
    /// Null values are never valid.
    /// Создать завёрнутое опциональное значение
    /// 1) Снача првиводим входные данные в канонический вид
    /// 2) Если проверка прошла успешно, верните Some от результаом, который возвращает переданный конструктор
    /// 3) Если проверка прошла неудачно, верните None
    /// Значения null не могут быть правильными
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
    /// Получить завёрнутое значение
    let value s = apply id s

    /// Equality test
    /// Проверить на равенство
    let equals left right =
        (value left) = (value right)

    /// Comparison
    /// Сравнение
    let compareTo left right =
        (value left).CompareTo (value right)
```

> The key function is `create`, which takes a constructor function and creates new values using it only when the validation passes.

Ключевая функция — это `create`, которая получает функцию-конструктор и создаёт новые значения, используя их только если валидация проходит.

> With this in place it is a lot easier to define new types:

При таком подходе определять новые типы становится намного проще

```fsharp
module WrappedString =

    // ... code from above ...
    // ... код из примера выше ...

    /// Canonicalizes a string before construction
    /// * converts all whitespace to a space char
    /// * trims both ends
    /// Перед конструированием приводим строку в канонический вид
    /// * ковертируем все пробельные символы в пробелы
    /// * обрезаем строку слева и справа
    let singleLineTrimmed s =
        System.Text.RegularExpressions.Regex.Replace(s,"\s"," ").Trim()

    /// A validation function based on length
    /// Функция проверки основанная на длине строки
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
    /// Конвертирует завёрнутую строку в строку длины 100
    let convertTo100 s = apply string100 s

    /// A string of length 50
    /// Строка длины 50
    type String50 = String50 of string with
        interface IWrappedString with
            member this.Value = let (String50 s) = this in s

    /// A constructor for strings of length 50
    /// Конструктор для строк длины 50
    let string50 = create singleLineTrimmed (lengthValidator 50)  String50

    /// Converts a wrapped string to a string of length 50
    /// Конвертирует завёрнутую строку в строку длины 50
    let convertTo50 s = apply string50 s
```

> For each type of string now, we just have to:

Теперь для каждого типа строки нам нужно просто:

> * create a type (e.g. `String100`)
> * an implementation of `IWrappedString` for that type
> * and a public constructor (e.g. `string100`) for that type.

* создать тип (т.е. `String100`)
* реализация `IWrappedString` для данного типа
* и публичный конструктор (т.е. `string100`) для этого типа.

> (In the sample above I have also thrown in a useful `convertTo` to convert from one type to another.)

(В примере выше я также добавил полезную функцию `convertTo`, чтобы конвертировать типы друг в друга.)

> The type is a simple wrapped type as we have seen before.

Тип — это простой тип-обёртка, как мы видели ранее.

> The implementation of the `Value` method of the IWrappedString could have been written using multiple lines, like this:

Реализация метода `Value` из `IWrappedString` может занимать две строки, как здесь:

```fsharp
member this.Value =
    let (String100 s) = this
    s
```

> But I chose to use a one liner shortcut:

Но я предпочитаю использовать короткий одностроный вариант:

```fsharp
member this.Value = let (String100 s) = this in s
```

> The constructor function is also very simple.
> The canonicalize function is `singleLineTrimmed`, the validator function checks the length, and the constructor is the `String100` function (the function associated with the single case, not to be confused with the type of the same name).

Функция-конструктор тоже очень проста.
Функция приведения в каноническую форму — это `singleLineTrimmed`, функция валидации проверяет длину, а конструктор — это функция `String100` (функция, связанная с единственным вариантом <!-- именем варианта -->, не путайте с одноимённым типом).

```fsharp
let string100 = create singleLineTrimmed (lengthValidator 100) String100
```

> If you want to have other types with different constraints, you can easily add them.
> For example you might want to have a `Text1000` type that supports multiple lines and embedded tabs and is not trimmed.

Если вы хотите иметь другие типы с отличающимися ограничениями, вы легко можете их добавить.
Например вы можете захотеть иметь тип `Text1000`, который поддерживает переводы строк, встроенные табуляции и не обрезается.

```fsharp
module WrappedString =

    // ... code from above ...
    // ... код из примера выше ...

    /// A multiline text of length 1000
    /// Текст длины 1000 с переводами строк
    type Text1000 = Text1000 of string with
        interface IWrappedString with
            member this.Value = let (Text1000 s) = this in s

    /// A constructor for multiline strings of length 1000
    /// Конструктор с поддержкой переводов строки длины 1000
    let text1000 = create id (lengthValidator 1000) Text1000
```

> ### Playing with the WrappedString module

### Экспериментируем с модулем `WrappedString`

> We can now play with the module interactively to see how it works:

Мы можем сейчас поэкспериментировать с модулем интерактивно, чтобы понять, как он работает:

```fsharp
let s50 = WrappedString.string50 "abc" |> Option.get
printfn "s50 is %A" s50
let bad = WrappedString.string50 null
printfn "bad is %A" bad
let s100 = WrappedString.string100 "abc" |> Option.get
printfn "s100 is %A" s100

// equality using module function is true
// проверка на равенство с помощью функции модуля возвращет true
printfn "s50 is equal to s100 using module equals? %b" (WrappedString.equals s50 s100)

// equality using Object method is false
// проверка на равество с помощью метода класса Object вовзвращает false
printfn "s50 is equal to s100 using Object.Equals? %b" (s50.Equals s100)

// direct equality does not compile
// прямое сравнение не компилируется
printfn "s50 is equal to s100? %b" (s50 = s100) // ошибка компилятора (compiler error)
```

> When we need to interact with types such as maps that use raw strings, it is easy to compose new helper functions.

Когда нам надо взаимодействовать с такими типами, как словари, которые используют простые строки, легко сконструировать новые вспомогательные функции.

> For example, here are some helpers to work with maps:

Например, вот несколько вспомогательных функций для работы со словарями:

```fsharp
module WrappedString =

    // ... code from above ...
    // ... код из примера выше ...

    /// map helpers
    /// вспомогательные функции для словарей
    let mapAdd k v map =
        Map.add (value k) v map

    let mapContainsKey k map =
        Map.containsKey (value k) map

    let mapTryFind k map =
        Map.tryFind (value k) map
```

> And here is how these helpers might be used in practice:

А вот как эти функции могут быть использованы на практике:

```fsharp
let abc = WrappedString.string50 "abc" |> Option.get
let def = WrappedString.string100 "def" |> Option.get
let map =
    Map.empty
    |> WrappedString.mapAdd abc "значение для (value for) abc"
    |> WrappedString.mapAdd def "значение для (value for) def"

printfn "Найдено ли abc в словаре (Found abc in map)? %A" (WrappedString.mapTryFind abc map)

let xyz = WrappedString.string100 "xyz" |> Option.get
printfn "Найдено ли xyz в словаре (Found xyz in map)? %A" (WrappedString.mapTryFind xyz map)
```

> So overall, this "WrappedString" module allows us to create nicely typed strings without interfering too much.
> Now let's use it in a real situation.

Итак, подводя итог, этот модуль `WrappedString` позволям нам создавать хорошо типизированные строки без большой мороки.
Теперь давайте попробуем его в реальной ситуации.

> ## Using the new string types in the domain

## Использование новых строковых типов в предметной области

> Now we have our types, we can change the definition of the `PersonalName` type to use them.

Теперь, когда у нас есть наши типы, мы можем изменить определение типа `PersonalName` с их использованием.

```fsharp
module PersonalName =
    open WrappedString

    type T =
        {
        FirstName: String50;
        LastName: String100;
        }

    /// create a new value
    /// создаём новое значение
    let create first last =
        match (string50 first),(string100 last) with
        | Some f, Some l ->
            Some {
                FirstName = f;
                LastName = l;
                }
        | _ ->
            None
```

> We have created a module for the type and added a creation function that converts a pair of strings into a `PersonalName`.

Мы создали модуль для типа и добавили функцию создания, которая конвертирует пару строк в `PersonalName`.

> Note that we have to decide what to do if *either* of the input strings are invalid.
> Again, we cannot postpone the issue till later, we have to deal with it at construction time.

Обратите внимание, что должны решить, что делать, если *хотя бы* одна из входных строк неправильная.
И снова, вы не можем откладывать этот вопрос на потом, нам должны решить его во время конструирования.

> In this case we use the simple approach of creating an option type with None to indicate failure.

В этом случае мы используем простой подход, создавая опциональный тип со значением `None`, чтобы сообщить о неудаче.

> Here it is in use:

Вот как это используется:

```fsharp
let name = PersonalName.create "John" "Smith"
```

> We can also provide additional helper functions in the module.

Мы тажке может предоставить дополинтельные вспомогательные функции в модуле.

> Let's say, for example, that we want to create a `fullname` function that will return the first and last names joined together.

Скажем, например, что мы хотим создать функцию `fullName` которая возвращает склеенные вместе имя и фамилию.

> Again, more decisions to make.

Снова придётся принять дополнительные решения.

> * Should we return a raw string or a wrapped string?
>   The advantage of the latter is that the callers know exactly how long the string will be, and it will be compatible with other similar types.
> * If we do return a wrapped string (say a `String100`), then how do we handle the the case when the combined length is too long?
>   (It could be up to 151 chars, based on the length of the first and last name types.).
>   We could either return an option, or force a truncation if the combined length is too long.

* Должны ли мы возвращать простую строку или завёрнутую строку?
  Преимущество последнего в том, что вызывающая сторона точно знает, какой длины будет строка, и это будет совместимо с другими подобными типами.
* Если мы возвращаем завёрнутую строку (например `String100`), как тогда мы должны обрабатывать случай, когда длина склееной строки оказывается слишком большой?
  (Она может быть до 151 символа длиной, исходя из длины типов для имени и фамилии).
  Мы могли бы или вернуть опциональный результат, или принудительно укоротить, если длина склееной строки окажется слишком большой.

> Here's code that demonstrates all three options.

Вот код, который демонстрирует все три возможности:

```fsharp
module PersonalName =

    // ... code from above ...
    // ... код из примера выше ...

    /// concat the first and last names together
    /// and return a raw string
    /// склеиваем имя и фамилию вместе
    /// и возвращаем простую строку
    let fullNameRaw personalName =
        let f = personalName.FirstName |> value
        let l = personalName.LastName |> value
        f + " " + l

    /// concat the first and last names together
    /// and return None if too long
    /// склеиваем имя и фамилию вместе
    /// и возвращаем None если строка слишком длинная
    let fullNameOption personalName =
        personalName |> fullNameRaw |> string100

    /// concat the first and last names together
    /// and truncate if too long
    /// склеиваем имя и фамилию вместе
    /// и укорачиваем строку, если она слишком длинная
    let fullNameTruncated personalName =
        // helper function
        // вспомогательная функция
        let left n (s:string) =
            if (s.Length > n)
            then s.Substring(0,n)
            else s

        personalName
        |> fullNameRaw  // склеить (concat)
        |> left 100     // укоротить (truncate)
        |> string100    // завернуть (wrap)
        |> Option.get   // здесь всеегда будет значение (this will always be ok)

```

> Which particular approach you take to implementing `fullName` is up to you.
> But it demonstrates a key point about this style of type-oriented design: these decisions have to be taken *up front*, when creating the code.
> You cannot postpone them till later.

Какой именно подход к реализации `fillName` выбирать, решать вам.
Важно понять ключевой принцип такого стиля типо-ориентированного дизайна: эти решения должны быть приняты *заранее*, при написании кода.
Вы не можете откладывать их на потом.

> This can be very annoying at times, but overall I think it is a good thing.

Иногда это сильно раздражает, но в целом, я думаю, это хороший подход.

> ## Revisiting the email address and zip code types

## Новый взгляд на типы электронного адреса и почтового индекса

> We can use this WrappedString module to reimplement the `EmailAddress` and `ZipCode` types.

Мы можем использовать этот модуль `WrappedString` для повторной реализации типов `EmailAddress` и `ZipCode`.

```fsharp
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
    /// Конвертирует любую завёрнтую строку в EmailAddress
    let convert s = WrappedString.apply create s

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
    /// Конвертирует любую завёрнутую строку в ZipCode
    let convert s = WrappedString.apply create s
```

> ## Other uses of wrapped strings

## Другие применения завёрнутых строк

> This approach to wrapping strings can also be used for other scenarios where you don't want to mix string types together accidentally.

Такой подход с заворачиванием строк может тажке быть полезен для других сценариев, где вы не хотите случайно перепутать строковые типы вместе.

> One case that leaps to mind is ensuring safe quoting and unquoting of strings in web applications.

Один из примеров, которые приходят на ум — обеспечение экранирования строк и обратной операции в веб-приложениях.

> For example, let's say that you want to output a string to HTML.
> Should the string be escaped or not?
> If it is already escaped, you want to leave it alone but if it is not, you do want to escape it.

Например, скажем, что вы хотите вывести строку в HTML.
Должна ли эта строка быть экранированной или нет?
Если она уже экранирована, вы захотите оставить её в покое, но если нет, вы захотите экранировать её.

> This can be a tricky problem.
> Joel Spolsky discusses using a naming convention [here](http://www.joelonsoftware.com/articles/Wrong.html), but of course, in F#, we want a type-based solution instead.

Это может оказаться сложной проблемой.
Джоэл Спольски обсуждаем использование соглашений об именах [здесь](http://www.joelonsoftware.com/articles/Wrong.html), но конечно, в F# мы хотим вместого решение, основанное на типах.

> A type-based solution will probably use a type for "safe" (already escaped) HTML strings (`HtmlString` say), and one for safe Javascript strings (`JsString`), one for safe SQL strings (`SqlString`), etc.
> Then these strings can be mixed and matched safely without accidentally causing security issues.

Решение, основанное на типах будет, вероятно, использовать один тип для «безопасных» (уже экранированных) строк HTML (скажем `HtmlString`), ещё один для безопасных строк JavaScript (`JsString`) и ещё один для безопасных строк SQL (`SqlString`) и т. д.
После этого эти строки можно будет безопасно смешивать и сопоставлять, не вызывая случайно проблем безопасности.

> I won't create a solution here (and you will probably be using something like Razor anyway), but if you are interested you can read about a [Haskell approach here](http://blog.moertel.com/articles/2006/10/18/a-type-based-solution-to-the-strings-problem) and a [port of that to F#](http://stevegilham.blogspot.co.uk/2011/12/approximate-type-based-solution-to.html).

Я не буду приводить здесь решение (и вы, вероятно, всё равно будете использовать что-то вроде Razor), но если вам интересно, вы можете прочитать о [подходе, приянятом в Haskell здесь](http://blog.moertel.com/articles/2006/10/18/a-type-based-solution-to-the-strings-problem) и о его [версии, портированной на F#](http://stevegilham.blogspot.co.uk/2011/12/approximate-type-based-solution-to.html).

> ## Update ##

## Обновление

> Many people have asked for more information on how to ensure that constrained types such as `EmailAddress` are only created through a special constructor that does the validation.
> So I have created a [gist here](https://gist.github.com/swlaschin/54cfff886669ccab895a) that has some detailed examples of other ways of doing it.

Многие люди спрашивают меня больше рассказать о том, как гарантировать, что ограниченные типы, такие как `EmailAddress` создаются только с помощью специального конструктора, который включает валидацию.
Так что я написал [фрагмент кода здесь](https://gist.github.com/swlaschin/54cfff886669ccab895a), который содержит некоторые подробные примеры других способов добиться этого.

{{< book_page_ddd_img >}}