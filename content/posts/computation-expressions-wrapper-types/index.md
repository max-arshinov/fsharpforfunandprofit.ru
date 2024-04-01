---
layout: post
title: "Вычислительные выражения и типы-обёртки"
description: "Использование типов, чтобы упростить процесс"
date: 2013-01-23
nav: thinking-functionally
seriesId: "Вычислительные выражения"
seriesOrder: 4
---

В предыдущем посте мы представили процесс "maybe", который позволил нам спрятать мешанину, возникающую при объединении опциональных типов.

Типичное использование процесса "maybe" выглидит как-то так:

```fsharp
let result =
    maybe
        {
        let! anInt = expression of Option<int>
        let! anInt2 = expression of Option<int>
        return anInt + anInt2
        }
```

Как мы отмечали раньше, здесь есть один странный момент:

* В строках с конструкцей `let!`, выражение *справа* от знака равенства имеет тип `int option`, при этом значение *слева* имеет тип `int`.
  `let!` "разворачивает" опциональный тип перед тем, как связать его со значением.
* В строке `return` всё наоборот.
  Возвращаемое выражение имеет тип `int`, но значение всего вычислительного выражения (`result`) имеет тип `int option`.
  То есть, `return` "заворачивает" обычное значение обратно в опциональный тип.

В этом посте мы не раз столкнёмся с подобными наблюдениями, и увидим, что это ведет к одному из основных способов применения вычислительных выражений, а именно: неявному "разворачиванию" и "заворачиванию" значений, хранящихся в каком-то типе-обёртке.

## Другой пример

Посмотрим на другой пример.
Предположим, мы обращаемся к базе данных, и хотим получать результат в виде такого типа-объединения Успех/Ошибка:

```fsharp
type DbResult<'a> =
    | Success of 'a
    | Error of string
```

После объявления мы можем использовать этот тип в наших методах доступа к базе данных.
Вот несколько простых заглушек, которые дадут вам представление, как можно применить тип `DbResult`:

```fsharp
let getCustomerId name =
    if (name = "")
    then Error "ошибка в getCustomerId"
    else Success "Cust42"

let getLastOrderForCustomer custId =
    if (custId = "")
    then Error "ошибка в getLastOrderForCustomer"
    else Success "Order123"

let getLastProductForOrder orderId =
    if (orderId  = "")
    then Error "ошибка в getLastProductForOrder"
    else Success "Product456"
```

Теперь представим, что нам надо выполнить эти вызовы последовательно.
Сначала получаем идентификатор покупателя по имени, затем — заказ по идентификатору покупателя, и, наконец — товар по идентификатору заказа.

Вот самый очевидный способ решить эту задачу.
Вы видите, что на каждом шаге мы вынуждены использовать сопоставление с образцом.

```fsharp
let product =
    let r1 = getCustomerId "Алиса"
    match r1 with
    | Error _ -> r1
    | Success custId ->
        let r2 = getLastOrderForCustomer custId
        match r2 with
        | Error _ -> r2
        | Success orderId ->
            let r3 = getLastProductForOrder orderId
            match r3 with
            | Error _ -> r3
            | Success productId ->
                printfn "Товар %s" productId
                r3
```

Поистине ужасный код.
А ещё здесь логика основного процесса смешалась с логикой обработки ошибок.

Вычислительные выражения спешат на помощь!
Мы можем написать выражение, которое обрабатывает ветвление Успех/Ошибка за кулисами:

```fsharp
type DbResultBuilder() =

    member this.Bind(m, f) =
        match m with
        | Error _ -> m
        | Success a ->
            printfn "\tУдачно: %s" a
            f a

    member this.Return(x) =
        Success x

let dbresult = new DbResultBuilder()
```

{{<alertinfo>}}
Ещё раз обратите внимание, что "строитель" или "построитель" ("builder") в контексте вычислительных выражений — не то же самое, что объектно-оритентированный паттерн "строитиль", который применяется для конструирования и валидации объектов.
Пост о ["паттерне строитель" здесь](../builder-pattern).
{{</alertinfo>}}

> And with this workflow, we can focus on the big picture and write much cleaner code:

Имея такой процесс, мы можем сконцентрироваться на основной задаче и писать гораздо более чистый код:

```fsharp
let product' =
    dbresult {
        let! custId = getCustomerId "Алиса"
        let! orderId = getLastOrderForCustomer custId
        let! productId = getLastProductForOrder orderId
        printfn "Товар %s" productId
        return productId
        }
printfn "%A" product'
```

Если возникнут ошибки, процесс ловко их перехватит и сообщит нам о месте возникновения, как в следующем примере:

```fsharp
let product'' =
    dbresult {
        let! custId = getCustomerId "Алиса"
        let! orderId = getLastOrderForCustomer "" // провоцируем ошибку!
        let! productId = getLastProductForOrder orderId
        printfn "Товар %s" productId
        return productId
        }
printfn "%A" product''
```

## Роль типов-обёрток при работе с процессами

Что ж, мы познакомились с двумя процессами (`maybe` и `dbresult`), и у каждого из которых есть собственный тип-обёртка (`Option<T>` и `DbResult<T>` соответственно).

И это не особые случаи. На самом деле, у *каждого* вычислительного выражения *должен* быть связанный с ним тип-обёртка.
И часто этот тип-обёртка разрабатывается с оглядкой на процесс, которым мы хотим управлять.

Пример выше ясно это демонстрирует.
Тип `DbResult`, который мы создали — больше чем просто тип, возвращающий какие-то значения; в действительности он критически важен для процесса, "сохраняя" его текущее состояние, независимо от того, удачным или ошибочным был очередной шаг.
Используя различные варианты типа-объединения, процесс `dbresult` может незаметно переходить из состояния в состояние, пряча их от нас, и позволяя нам сконцентрироваться на основной задаче.

Позже в этой серии мы научимся проектировать хорошие типы-обёртки, а пока узнаем, как их использовать.

## Bind, Return и типы-обёртки

Ещё раз взглянем на определение методов `Bind` и `Resturn` в вычислительных выражениях.

Начнём с простого — с `Return`.
Сигнатура `Return`, [как написано в MSDN](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/computation-expressions#return), выглядит так:

```fsharp
member Return : 'T -> M<'T>
```

Иными словами, для какого-то типа `T`, метод `Return` просто заворачивает его в тип-обёртку.

*Обратите внимание: В сигнатурах тип-обёртка обычно называется `M`, так что `M<int>` — это тип-обёртка примененный к `int`, а `M<string>` — тип-обёртка примененный к `string`, и так далее.*


Мы видели два примера такого использования.
Процесс `maybe` возвращает `Some`, который является одним из вариантов опционального типа, а процесс `dbresult` возвращает `Success`, который также является одним из вариантов типа `DbResult`.

```fsharp
// Return для процесса maybe
member this.Return(x) =
    Some x

// Return для процесса dbresult
member this.Return(x) =
    Success x
```

Теперь посмотрим на `Bind`.
Сигнатура `Bind`:

```fsharp
member Bind : M<'T> * ('T -> M<'U>) -> M<'U>
```

Она довольно сложная, так что давайте разбираться.
Функция получает на вход кортеж `M<'T> * ('T -> M<'U>)` и возвращает `M<'U>`, где `M<'U>` — это тип-обёртка для типа-параметра `U`.

Кортеж состоит из двух частей:

* `M<'T>` — тип-обёртка с типом параметром `T`, и
* `'T -> M<'U>` — функция которая получает "развёрнутое" значение `T` и возвращает "завёрнутое" значение `U`.

Другими словами, вот что делает `Bind`:

* Берёт "завёрнутое" значение.
* Разворачивает его в соответствии с уникальной и "спрятанной" логикой процесса.
* Затем, возможно, применяет функцию к "развёрнутому" значению, чтобы получить новое "завёрнутое" значение.
* Даже если функция *не* применяется, `Bind` всё равно должен вернуть "завёрнутое" значение `U`.

Учитывая всё это, ещё раз взглянем на методы `Bind`, которые мы написали ранее:

```fsharp
// Bind для процесса maybe
member this.Bind(m,f) =
   match m with
   | None -> None
   | Some x -> f x

// Bind для процесса dbresult
member this.Bind(m, f) =
    match m with
    | Error _ -> m
    | Success x ->
        printfn "\tУдачно: %s" x
        f x
```

Посмотрите на этот код и убедитесь, что вы понимаете, почему эти методы на самом деле следуют шаблону, описанному выше.

На всякий случай вот вам картинка.
Здесь нарисована диаграмма различных типов и функций:

![диаграмма связывания](./bind.png)

* Для `Bind` мы начинаем с завёрнутого значения (на картинке `m`), разворачиваем его в простое значение типа `T` и затем (может быть) применяем к нему функцию `f`, чтобы получить завёрнутое значение типа `U`.
* Для `Return` мы начинаем с обычного значения (на картинке `x`) и просто заворачиваем его.

### Тип-обёртка — обобщённый

Обратите внимание, что все функции используют обобщённые типы (`T` и `U`), за исключением самого типа-обёртки, который везде одинаковый.

В частности, ничто не мешает функции связывания `maybe` принимать на вход `int` и возвращать `Option<string>`, или принимать `string`, а возвращать `Option<bool>`.
Единственное ограничение заключается в том, что она всегда должна возвращать `Option<что-то>`.

Чтобы убедиться в этом, вернёмся к примеру выше, но вместо использования строк в качестве параметров, создадим особые типы для идентификаторов покупателя, заказа и продукта.

Снова начнём с типов, определив `CustomerId` и все прочие:

```fsharp
type DbResult<'a> =
    | Success of 'a
    | Error of string

type CustomerId =  CustomerId of string
type OrderId =  OrderId of int
type ProductId =  ProductId of string
```

Код почти не изменится, за исключением того, что в ветках `Success` появятся новые типы.

```fsharp
let getCustomerId name =
    if (name = "")
    then Error "ошибка в getCustomerId"
    else Success (CustomerId "Cust42")

let getLastOrderForCustomer (CustomerId custId) =
    if (custId = "")
    then Error "ошибка в getLastOrderForCustomer"
    else Success (OrderId 123)

let getLastProductForOrder (OrderId orderId) =
    if (orderId  = 0)
    then Error "ошибка в getLastProductForOrder"
    else Success (ProductId "Product456")
```

Снова "длинная" версия кода.

```fsharp
let product =
    let r1 = getCustomerId "Алиса"
    match r1 with
    | Error e -> Error e
    | Success custId ->
        let r2 = getLastOrderForCustomer custId
        match r2 with
        | Error e -> Error e
        | Success orderId ->
            let r3 = getLastProductForOrder orderId
            match r3 with
            | Error e -> Error e
            | Success productId ->
                printfn "Товар %A" productId
                r3
```

Здесь есть пара моментов, достойных обсуждения:

* Во-первых, функция `printfn` в конце программы использует формат "%A" вместо "%s".
  Это нужно, поскольку тип `ProductId` — не строка, а объединение.
* Второй, более тонкий момент заключается в том, что код в ошибочных ветках кажется избыточным.
  Зачем писать `| Error e -> Error e`?
  Причина в том, что входящая ошибка имеет тип `DbResult<CustomerId>` или `DbResult<OrderId>`, а результат должен быть типа `DbResult<ProductId>`.
  Не смотря на то, что оба `Error` выглядят одинаково, в действительности они имеют разные типы.

И, наконец, класс-построитель, код которого не меняется, за исключением строки `| Error e -> Error e`.

```fsharp
type DbResultBuilder() =

    member this.Bind(m, f) =
        match m with
        | Error e -> Error e
        | Success a ->
            printfn "\tУдача: %A" a
            f a

    member this.Return(x) =
        Success x

let dbresult = new DbResultBuilder()
```

Код самого процесса остался неизменным.

```fsharp
let product' =
    dbresult {
        let! custId = getCustomerId "Алиса"
        let! orderId = getLastOrderForCustomer custId
        let! productId = getLastProductForOrder orderId
        printfn "Товар %A" productId
        return productId
        }
printfn "%A" product'
```

В каждой строке, возвращаемое значение имеет *свой собственный* тип (`DbResult<CustomerId>`,`DbResult<OrderId>`, ...), но, поскольку все они используют тот же тип-обёртку, связывание работает так, как мы и ожидаем.

И, для сравнения — процесс с ошибкой.

```fsharp
let product'' =
    dbresult {
        let! custId = getCustomerId "Алиса"
        let! orderId = getLastOrderForCustomer (CustomerId "") // провоцируем ошибку!
        let! productId = getLastProductForOrder orderId
        printfn "Product is %A" productId
        return productId
        }
printfn "%A" product''
```

## Композиция вычислительных выражений

Мы узнали, что каждое вычислительное выражение *обязано* иметь связанный с ним тип-обёртку.
Этот тип-обёртка используется и в методе `Bind` и в методе `Return`, что даёт нам важную возможность:

* *выход метода `Return` можно подать на вход метода `Bind`*

Иными словами, поскольку процесс возвращает тип-обёртку, и поскольку `let!` получает тип-обёртку, вы можете поместить "дочерний" процесс в правой части выражения `let!`.

Скажем, у вас есть процесс `myworkflow`.
Вы можете написать что-то подобное:

```fsharp
let subworkflow1 = myworkflow { return 42 }
let subworkflow2 = myworkflow { return 43 }

let aWrappedValue =
    myworkflow {
        let! unwrappedValue1 = subworkflow1
        let! unwrappedValue2 = subworkflow2
        return unwrappedValue1 + unwrappedValue2
        }
```

Вы даже можете "встроить" вызовы непосредственно во внешний процесс:

```fsharp
let aWrappedValue =
    myworkflow {
        let! unwrappedValue1 = myworkflow {
            let! x = myworkflow { return 1 }
            return x
            }
        let! unwrappedValue2 = myworkflow {
            let! y = myworkflow { return 2 }
            return y
            }
        return unwrappedValue1 + unwrappedValue2
        }
```

Если вы использовали процесс `async`, то, скорее всего, сталкивались с подобным подходом, поскольку асинхронный процесс обычно содержит другие встроенные асинхронные процессы:

```fsharp
let a =
    async {
        let! x = doAsyncThing  // вложенный процесс
        let! y = doNextAsyncThing x // вложенный процесс
        return x + y
    }
```

## Введение в "ReturnFrom"

Мы используем `return`, чтобы завернуть результат вычислительного выражения.

Но иногда у нас есть функция, которая уже возвращает завёрнутое значение, которое нам надо передать дальше.
`return` для этого не подходит, поскольку он требует сначала развернуть значение.

Решением является варианция `return` которая называется `return!`.
Этот оператор получает на вход *завёрнутый тип* и возвращает его же.

Соответствующий метод в классе "построителе" называется `ReturnFrom`.
Как правило, он просто возвращает завёрнутое значение "как есть" (хотя, конечно, вы всегда можете добавить дополнительную закулисную логику).

Вот вариант процесса "maybe", который показывает, как это можно использовать:

```fsharp
type MaybeBuilder() =
    member this.Bind(m, f) = Option.bind f m
    member this.Return(x) =
        printfn "Оборачивает значение в опциональный тип"
        Some x
    member this.ReturnFrom(m) =
        printfn "Возвращает опциональное значение напрямую"
        m

let maybe = new MaybeBuilder()
```

Вот как это можно использовать:

```fsharp
// возвращаем int
maybe { return 1  }

// возвращаем Option
maybe { return! (Some 2)  }
```

Если вам нужен более реалистичный пример, взгляните, как `return!` используется совместно с `divideBy`:

```fsharp
// используем return
maybe
    {
    let! x = 12 |> divideBy 3
    let! y = x |> divideBy 2
    return y  // возвращаем int
    }

// используем return!
maybe
    {
    let! x = 12 |> divideBy 3
    return! x |> divideBy 2  // возвращаем Option
    }
```

## Итоги

Этот пост рассказал о типах-обёртках и о том, как они связаны с методами `Bind`, `Return` и `ReturnFrom`, являющимися основными методами любого класса-строителя.

В следующем посте мы продолжим изучать типы-обёртки, в том числе рассмотрим списки в качестве таких обёрток.
