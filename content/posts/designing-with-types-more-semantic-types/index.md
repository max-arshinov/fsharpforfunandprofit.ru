---
layout: post
title: "Проектирование с помощью типов: Типизированные строки"
description: "Добавляем больше семантической информации к примитивному типу"
date: 2013-01-17
nav: thinking-functionally
seriesId: "Проектирование с помощью типов"
seriesOrder: 6
categories: [Types, DDD]
---

В одном из [предыдущих постов](/../designing-with-types-single-case-dus/) я писал, что для представления электронных адресов, почтовых индексов, состояний, и т. д., можно использовать не простые примитивные строки, а размеченные объединения с одним вариантом (SCDU, Single Case Descriminated Unions).
Благодаря им можно сделать типы хорошо различимыми и добавить правила валидации.

В этом посте мы узнаем, как сделать этот подход ещё более точным и строгим.

## Когда строка — не строка?

Вот простой тип `PersonalName`:

```fsharp
type PersonalName =
    {
    FirstName: string;
    LastName: string;
    }
```

Тип утверждает, что личное имя сотрудника — это `string`.
Но разве это всё, на самом деле?
Существуют ли другие ограничения, которые можно наложить на строку?

Для начала, она не должна принимать значение `null`.
Впрочем, это и так справедливо для F#.

Что насчёт длины?
Допустимо ли иметь имя длиной 64К символов?
Если нет, существует ли какая-то максимальная длина?

Может ли имя содержать переводы строки или табуляции?
Может ли оно начинаться или заканчиваться пробелом?

Если подумать, есть довольно много ограничений даже для обычной строки.

## Должны ли ограничения быть частью предметной области?

Мы можем сознавать, что существует какие-то ограничения, но действительно ли их надо делать частью предметной области (и соответствующих типов, которые от неё зависят)?
Например, ограничение, что фамилия должна быть не длиннее ста символов, относится к конкретной реализации, а не к предметной области в целом.

Я мог бы сказать о различии между логической моделью и физической моделью.
В логической модели часть ограничений может быть неважной, но в физической модели они, безусловно, важны.
А когда мы пишем код, мы имеем дело с физической моделью.

Ещё одна причина поместить ограничения в модель, заключается в том, что она, зачастую, совместно используется несколькими приложениями.
Скажем, личное имя может быть создано в приложении электронной коммерции, которое записывает его в таблицу БД, а затем отправляет в очередь сообщений системы CRM, которая, в свою очередь, вызывает службу шаблонов электронной почты и т. д.

Важно, чтобы все эти приложения и службы имели *общее* представление о том, что такое личное имя, включая длину и другие ограничения.
Если модель не делает ограничения явными, легко столкнуться с рассогласованием при передажи значения от службы к службе.

Приходилось ли вам писать код, который проверяет длину строки перед записью в базу?

```csharp
void SaveToDatabase(PersonalName personalName)
{
   var first = personalName.First;
   if (first.Length > 50)
   {
        // убедиться, что строка не слишком длинная
        first = first.Substring(0,50);
   }

   //сохранить в базу
}
```

Что делать, если строка *оказалась* слишком длинной?
Молча её обрезать?
Бросить исключение?

Лучший ответ — вообще не попадать в такую ситуацию.
Поздно принимать решение на этапе, когда строка пишется в базу данных.

Проблема должна быть решена, когда строка *впервые создаётся*, а не когда она *используется*.
Иными словами, решение должно быть частью валидации строки.

Но как убедиться, что строка прошла валидацию на всех возможных путях программы?
Думаю, вы знаете ответ...

## Моделируем типизированные строки

Ответ, конечно, в том, чтобы написать типы-обёртки и встроить ограничения в них.

Накидаем прототип, используя [знакомые](../designing-with-types-single-case-dus/) нам одновариантные объединения.

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

Обратите внимание, что мы обрабатываем неудачную валидацию сразу, возвращая в качестве результата опциональный тип.
Мы усложнили создание строки, но, сделав этот код сложнее, мы упростили код, который напишем в будущем.

Примеры хорошей и плохой строки длины 2.

```fsharp
let s2good = String2.create "CA"
let s2bad = String2.create "California"

match s2bad with
| Some s2 -> // обновляем объект предметной области
| None -> // обрабатываем ошибку
```

Чтобы использовать значение `String2`, мы должны проверить его при создании.

### Проблемы подобного дизайна

Одна их проблем заключается в том, что мы получаем много дублирующегося кода.
Впрочем, в типичной предметной области всего несколько десятков строковых типов, так что дублей будет не очень много.
Но мы всё же постараемся избавиться от дублей.

Другая проблема в том, что мы «сломали» сравние строк, а это уже серьёзно.
Типы `String50` и `String100` отличаются друг от друга, так что их нельзя сравнивать напрямую.

```fsharp
let s50 = String50.create "John"
let s100 = String100.create "Smith"

let s50' = s50.Value
let s100' = s100.Value

let areEqual = (s50' = s100')  // ошибка компилятора
```

Всё это усложняет работу со словарями и списками.

{{< book_page_pdf >}}

### Рефакторинг

Воспользуемя тем, что в F# есть интерфейсы и создадим общий интерфейс для типов-обёрток, описав несколько стандартных методов.

```fsharp
module WrappedString =

    /// Интерфейс, который поддерживают все типы-обёртки
    type IWrappedString =
        abstract Value : string

    /// Создать завёрнутое опциональное значение
    /// 1) Сперва привести входные данные к каноническому виду
    /// 2) Если проверка прошла успешно, вернуть Some с результатом конструктора
    /// 3) Если проверка прошла неудачно, вернуть None
    /// Значения null не являются корректными
    let create canonicalize isValid ctor (s:string) =
        if s = null
        then None
        else
            let s' = canonicalize s
            if isValid s'
            then Some (ctor s')
            else None

    /// Применить функцию к завёрнутому значению
    let apply f (s:IWrappedString) =
        s.Value |> f

    /// Вернуть завёрнутое значение
    let value s = apply id s

    /// Проверить на равенство
    let equals left right =
        (value left) = (value right)

    /// Сравнить
    let compareTo left right =
        (value left).CompareTo (value right)
```

Ключевая функция — это `create`, которая получает конструктор и создаёт значения только в случае успешной валидации.

С помощью интерфейса создавать новые типы намного проще:

```fsharp
module WrappedString =

    // ... код из примера выше ...

    /// Перед конструированием приводим строку к каноническому виду:
    /// * ковертируем все пробельные символы в пробелы
    /// * обрезаем строку слева и справа
    let singleLineTrimmed s =
        System.Text.RegularExpressions.Regex.Replace(s,"\s"," ").Trim()

    /// Функция валидации, основанная на длине строки
    let lengthValidator len (s:string) =
        s.Length <= len

    /// Строка длины 100
    type String100 = String100 of string with
        interface IWrappedString with
            member this.Value = let (String100 s) = this in s

    /// Конструктор строк длины 100
    let string100 = create singleLineTrimmed (lengthValidator 100) String100

    /// Конвертирует завёрнутую строку в строку длины 100
    let convertTo100 s = apply string100 s

    /// Строка длины 50
    type String50 = String50 of string with
        interface IWrappedString with
            member this.Value = let (String50 s) = this in s

    /// Конструктор строк длины 50
    let string50 = create singleLineTrimmed (lengthValidator 50)  String50

    /// Конвертирует завёрнутую строку в строку длины 50
    let convertTo50 s = apply string50 s
```

Теперь для каждого вида строк нам нужно:

* создать тип (`String100`)
* реализовать для него `IWrappedString`
* написать публичный конструктор (`string100`).

(В примере выше я также написал полезную функцию `convertTo`, чтобы конвертировать типы друг в друга.)

Тип — это обычный тип-обёртка, с которыми мы уже сталкивались.

Реализация метода `Value` из `IWrappedString` может занимать две строки:

```fsharp
member this.Value =
    let (String100 s) = this
    s
```

но я предпочитаю короткий одностроный вариант:

```fsharp
member this.Value = let (String100 s) = this in s
```

Функция `string100` тоже очень простая.
Она вызывает функцию приведения к каноническому виду — `singleLineTrimmed`, функцию валидации, которая проверяет длину, и функцию-конструктор — `String100`.
Последняя — это конструктор единственного варианта в типе-объединении.
Не путайте её с самим типом, который носит то же имя.

```fsharp
let string100 = create singleLineTrimmed (lengthValidator 100) String100
```

Если вам нужны другие типы с другими ограничениями, их несложно добавить.
Например, можно создать тип `Text1000`, который не урезается, поддерживает переводы строк и встроенные табуляции.

```fsharp
module WrappedString =

    // ... код из примера выше ...

    /// Текст длины 1000 с переводами строк
    type Text1000 = Text1000 of string with
        interface IWrappedString with
            member this.Value = let (Text1000 s) = this in s

    /// Конструктор с поддержкой переводов строки длины 1000
    let text1000 = create id (lengthValidator 1000) Text1000
```

### Экспериментируем с модулем `WrappedString`

Чтобы разобраться, как работает модуль, поэкспериментируем с ним в интерактивном режиме:

```fsharp
let s50 = WrappedString.string50 "abc" |> Option.get
printfn "s50 is %A" s50
let bad = WrappedString.string50 null
printfn "bad is %A" bad
let s100 = WrappedString.string100 "abc" |> Option.get
printfn "s100 is %A" s100

// проверка на равенство с помощью функции модуля возвращет true
printfn "s50 is equal to s100 using module equals? %b" (WrappedString.equals s50 s100)

// проверка на равество с помощью метода класса Object вовзвращает false
printfn "s50 is equal to s100 using Object.Equals? %b" (s50.Equals s100)

// прямое сравнение не компилируется
printfn "s50 is equal to s100? %b" (s50 = s100) // ошибка компилятора
```

Напишем несколько вспомогательных функций для работы с типом `Map`, который использует простые строки.

```fsharp
module WrappedString =

    // ... код из примера выше ...

    /// вспомогательные функции для словарей
    let mapAdd k v map =
        Map.add (value k) v map

    let mapContainsKey k map =
        Map.containsKey (value k) map

    let mapTryFind k map =
        Map.tryFind (value k) map
```

Примеры вызова:

```fsharp
let abc = WrappedString.string50 "abc" |> Option.get
let def = WrappedString.string100 "def" |> Option.get
let map =
    Map.empty
    |> WrappedString.mapAdd abc "значение для ключа abc"
    |> WrappedString.mapAdd def "значение для ключа def"

printfn "Найден ли ключ abc в словаре? %A" (WrappedString.mapTryFind abc map)

let xyz = WrappedString.string100 "xyz" |> Option.get
printfn "Найден ли ключ xyz в словаре? %A" (WrappedString.mapTryFind xyz map)
```

Можно сказать, что модуль `WrappedString` позволяет нам работать с хорошо типизированными строками без особой мороки.
Теперь давайте испытаем его в реальной ситуации.

## Используем новые строковые типы в предметной области

Пользуясь полученными знаниями, поменяем определение типа `PersonalName`.

```fsharp
module PersonalName =
    open WrappedString

    type T =
        {
        FirstName: String50;
        LastName: String100;
        }

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

Мы создали модуль для типа и добавили функцию `create`, которая превращает пару строк в `PersonalName`.

Мы обязаны решить, что делать, если *хотя бы* одна из этих строк неправильная.
Повторю, что этот вопрос нельзя откладыать на потом, с ним надо разобраться во время конструирования.

В данном случае мы используем простой подход, создавая при неудачной валидации опциональный тип со значением `None`.

Применение:

```fsharp
let name = PersonalName.create "John" "Smith"
```

Можно добавить в модуль несколько полезных функций.

Например, функцию `fullName` которая возвращает склеенные вместе имя и фамилию.

Чтобы её написать, нужно принять несколько решений.

* Возвращать простую строку или типизированную?
  Преимущество последней в том, что вызывающая сторона точно знает, какой длины будет строка, что совместимо с другими подобными типами.
* Возвращая типизированную строку (например `String100`), что мы будем делать, если склеенная строка окажется слишком большой?
  (Она может содержать до 151 символа, исходя из длины строк для имени и фамилии).
  Мы могли бы или вернуть опциональный результат или принудительно её укоротить.

Реализация всех трёх вариантов:

```fsharp
module PersonalName =

    // ... код из примера выше ...

    /// склеиваем вместе имя и фамилию
    /// возвращаем простую строку
    let fullNameRaw personalName =
        let f = personalName.FirstName |> value
        let l = personalName.LastName |> value
        f + " " + l

    /// склеиваем вместе имя и фамилию
    /// возвращаем None, если строка слишком длинная
    let fullNameOption personalName =
        personalName |> fullNameRaw |> string100

    /// склеиваем вместе имя и фамилию
    /// укорачиваем строку, если она слишком длинная
    let fullNameTruncated personalName =
        // вспомогательная функция
        let left n (s:string) =
            if (s.Length > n)
            then s.Substring(0,n)
            else s

        personalName
        |> fullNameRaw  // склеить
        |> left 100     // укоротить
        |> string100    // завернуть
        |> Option.get   // здесь всеегда будет значениеok)

```

Какой именно подход к реализации `fillName` выбрать, решать вам.
Но важно понять ключевой принцип такого стиля типо-ориентированного дизайна: все решения надо принимать *заранее*, при написании кода.
Нельзя откладывать их на потом.

Иногда это сильно раздражает, но мне кажется, что в целом это правильный подход.

## Новый взгляд на старые типы: электронный адрес и почтовый индекс

Мы можем использовать модуль `WrappedString` для повторной реализации типов `EmailAddress` и `ZipCode`.

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

    /// Конвертирует любую завёрнутую строку в ZipCode
    let convert s = WrappedString.apply create s
```

## Другие применения типизированных строк

Подход с заворачиванием строк может быть полезен и в других сценариях, если вы не хотите случайно перепутать строковые типы.

Один из примеров, которые приходят на ум — экранирование и разэкранирование строк в веб-приложениях.

Скажем, вы хотите вывести строку в HTML.
Надо ли её экранировать?
Если она уже экранирована, её надо оставить в покое, но если нет, строку надо экранировать.

Это может стать проблемой.
Джоэл Спольски [обсуждал](http://www.joelonsoftware.com/articles/Wrong.html) решение, основанное на именах переменных, но в F# мы, конечно, хотим решение, основанное на типах.

Решение, основанное на типах, вероятно, будет использовать тип для «безопасных» строк HTML (скажем `HtmlString`), тип для безопасных строк JavaScript (`JsString`) и тип для безопасных строк SQL (`SqlString`).
Эти строки можно использовать в одном коде, не опасаясь случайно их перепутать.

Я не стану приводить здесь решение (тем боле, что вы, вероятно, всё равно используете Razor), но если вам интересно, вы можете прочитать о [подходе, приянятом в Haskell](http://blog.moertel.com/articles/2006/10/18/a-type-based-solution-to-the-strings-problem) и его [версии, портированной на F#](http://stevegilham.blogspot.co.uk/2011/12/approximate-type-based-solution-to.html).

## Обновление

Многие читатели просили меня рассказать, как гарантировать создание типизированных строк вроде `EmailAddress` только с помощью валидирующего конструктора.
Я написал [git-фрагмент](https://gist.github.com/swlaschin/54cfff886669ccab895a), который содержит несколько работающих способов.

{{< book_page_ddd_img >}}