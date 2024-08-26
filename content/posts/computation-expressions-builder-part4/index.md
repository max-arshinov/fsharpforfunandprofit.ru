---
layout: post
title: "Реализуем вычислительные выражения: Перегрузка"
description: "Глупые приёмы"
date: 2013-01-28
nav: thinking-functionally
seriesId: "Вычислительные выражения"
seriesOrder: 9
---

В этом посте мы отклонимся от основной темы и познакомимся с парой трюков, которые помогут вам разнообразить методы в поистроителе вычислительных выражений.

В конечном итоге наши исследования заведут нас в тупик, но я надеюсь, что путешествие станет источником прозрений, как правильно проектировать вычислительные выражения.

{{<alertinfo>}}
Обратите внимание, что "построитель" в контексте вычислительных выражений — это не то же самое, что объектно-ориентированный паттерн "строитель", который используется для конструирования и валидации объектов.
{{</alertinfo>}}

## Озарение: методы построиля можно перегружать

В какой-то момент у вас может возникнуть озарение:

* Методы построителя — обычные методы класса, и, в отличие от автономной функций, методы поддерживают [перегрузку с отличающимися типами параметров](/posts/type-extensions/#method-overloading). Это значит, что мы можем делать *отличающиеся реализации*, если типы их параметров отличаются.

Возможно, эта идея вас вдохновляет и вы с энтузиазмом придумываете, как можно её использовать.
Однако, может оказататься, что она не настолько полезна, насколько вы думаете.
Взглянем на несколько примеров.

## Перегрузка "Return"

Скажем, у вас есть тип-объединение.
Вы решаете перегрузить `Return` или `Yield` с разными реализациями для каждого варианта объединения.

Вот очень простой пример, где `Return` имеет две перегрузки:

```fsharp
type SuccessOrError =
| Success of int
| Error of string

type SuccessOrErrorBuilder() =

    member this.Bind(m, f) =
        match m with
        | Success s -> f s
        | Error _ -> m

    /// перегрузка для int
    member this.Return(x:int) =
        printfn "Return a success %i" x
        Success x

    /// перегрузка для strings
    member this.Return(x:string) =
        printfn "Return an error %s" x
        Error x

// создаём экземпляр процесса
let successOrError = new SuccessOrErrorBuilder()
```

А вот как его можно использовать:

```fsharp
successOrError {
    return 42
    } |> printfn "Результат в случае успеха: %A"
// Результат в случае успеха: Success 42

successOrError {
    return "error for step 1"
    } |> printfn "Результат в случае ошибки: %A"
// Результат в случае ошибки: Error "error for step 1"
```

Как вы думаете, что здесь не так?

Ну, во-первых, если мы вернёмся к [обсуждению типов-обёрток](/posts/computation-expressions-wrapper-types-part2/), то обнаружим, что типы-обёртки лучше делать *обобщёнными*.
Мы хотим, насколько это возможно, использовать наш код повторно, в том числе, и процессы. Зачем привязывать реализацию к какому-то конкретному примитивному типу?

В нашем случае это значит, что тип объединения должны быть обобщён:


```fsharp
type SuccessOrError<'a,'b> =
| Success of 'a
| Error of 'b
```

Но теперь, из-за обобщённых типов, метод `Return` больше не можте быть перегружен! (В случае, если `'a` и `'b` совпадают, возникнет неоднозначность — *прим. переводчика*.)

Во-вторых, не самая хорошая идея — раскрывать детали реализации типа внутри выражения.
Концепция вариантов "успех" и "неудача" полезна, но лучшим способом было бы скрыть вариант "неудачи" и обрабатывать его автоматически внутри `Bind`:

```fsharp
type SuccessOrError<'a,'b> =
| Success of 'a
| Error of 'b

type SuccessOrErrorBuilder() =

    member this.Bind(m, f) =
        match m with
        | Success s ->
            try
                f s
            with
            | e -> Error e.Message
        | Error _ -> m

    member this.Return(x) =
        Success x

// создаём экземпляр процесса
let successOrError = new SuccessOrErrorBuilder()
```

Здесь `Return` используется только в случае "успеха", а "неудача" спрятана.

```fsharp
successOrError {
    return 42
    } |> printfn "Результат в случае успеха: %A"

successOrError {
    let! x = Success 1
    return x/0
    } |> printfn "Результат в случае ошибки: %A"
```

Больше примеров этой техники мы увидим в следующем посте.

## Множество реализаций "Combine"

Другая ситуация, когда у вас может возникнуть соблазн перегрузить метод — это `Combine`.

Для примера перепишем метод `Combine` для процесса `trace`.
Если вы помните, в предыдущей реализации мы просто складывали числа.

Но что, если мы изменим наши требованияя, скажем, так:

* если мы возвращаем несколько значений в процессе `trace`, мы объединяем их в список.

Первая попытка переписать `Combine` будет выглядеть так:

```fsharp
member this.Combine (a,b) =
    match a,b with
    | Some a', Some b' ->
        printfn "комбинируем %A с %A" a' b'
        Some [a';b']
    | Some a', None ->
        printfn "комбинируем %A с None" a'
        Some [a']
    | None, Some b' ->
        printfn "комбинируем None с %A" b'
        Some [b']
    | None, None ->
        printfn "комбинируем None с None"
        None
```

В методе `Combine` мы разворачиваем значения из переданного опционального типа с комбинируем их в список-обёртку в `Some` (т.е. `Some [a';b']`).

Для двух операторов `yield` всё будет работать, как мы и ожидали:

```fsharp
trace {
    yield 1
    yield 2
    } |> printfn "Результат yield и последющий yield: %A"

// Результат yield и последющий yield: Some [1; 2]
```

И при возврате `None`, всё тоже будет работать как надо:

```fsharp
trace {
    yield 1
    yield! None
    } |> printfn "Результат yield и последующий None: %A"

// Результат yield и последующий None: Some [1]
```

Но что получится, если мы попробуем скомбинировать *три* значения? Например, так:

```fsharp
trace {
    yield 1
    yield 2
    yield 3
    } |> printfn "Результат yield x 3: %A"
```

Внезапно мы получим ошибку компиляции:

```text
Error FS0193 : Несоответствие ограничений типов. Тип 
    "int list option"    
несовместим с типом
    "int option"
```

В чём проблема?

Ответ в том, что после комбинирования 2-го и 3-го значений (`yield 2; yield 3`), мы получаем опциональное значение, содержащиее сописок целых или `int list option`.
Ошибка происходит, когда мы пытаетмся комбинировать первое значение (`Some 1`) с комбинированным значением (`Some [2;3]`).
То есть мы передаём `int list option` в качестве второго параметра `Combine`, но первый параметр всё ещё обычный `int option`.
Компилятор говорит нам, что он хочет, чтобы второй параметр имел такой же тип, как и первый.

Мы снова можем использовать трюк с перегрузкой.
Напишем *две* реализации `Combine` с различными типами второго параметра — одну, которая принимает `int option` и второую, которая принимает `int list option`.

Вот два метода с различными типами параметра:

```fsharp
/// комбинируем с опциональным списком
member this.Combine (a, listOption) =
    match a,listOption with
    | Some a', Some list ->
        printfn "комбинируем %A с %A" a' list
        Some ([a'] @ list)
    | Some a', None ->
        printfn "комбинируем %A с None" a'
        Some [a']
    | None, Some list ->
        printfn "комбинируем None с %A" list
        Some list
    | None, None ->
        printfn "комбинируем None с None"
        None

/// комбинируем с опциональным одиночным значением
member this.Combine (a,b) =
    match a,b with
    | Some a', Some b' ->
        printfn "комбинируем %A с %A" a' b'
        Some [a';b']
    | Some a', None ->
        printfn "комбинируем %A с None" a'
        Some [a']
    | None, Some b' ->
        printfn "комбинируем None с %A" b'
        Some [b']
    | None, None ->
        printfn "комбинируем None с None"
        None
```

Теперь мы можем скомбинировать три значения и получим имено то, что хотели.

```fsharp
trace {
    yield 1
    yield 2
    yield 3
    } |> printfn "Результат yield x 3: %A"

// Результат yield x 3: Some [1; 2; 3]
```

К сожалению, этот трюк сломал часть предыдущего кода!
Если сейчас вы попробуете вернуть `None`, вы получите ошибку компиляции.

```fsharp
trace {
    yield 1
    yield! None
    } |> printfn "Результа yield с последующим None: %A"
```

Ошибка:

```text
Невозможно определить уникальную перегрузку метода "Combine" на основе сведений о типе, заданных до данной точки программы. Возможно, требуется аннотация типа.
```

Прежде чем впасть в раздражение, попробуйте думать как компилятор.
Если бы вы были компилятором, и получили бы `None`, какой бы метод вызвали *вы*?

Правильного ответа нет, потому что `None` можно передать вторым параметром в *оба* метода.
Компилятор не знает, относится ли этот `None` к типу `int list option` (первый метод) или к типу `int option` (второй метод).

Как и говорит комплиятор, нам помогла бы аннотация типа, поэтому давайте её и предоставим. Заставим `None` быть `int option`.

```fsharp
trace {
    yield 1
    let x:int option = None
    yield! x
    } |> printfn "Результат yield с последующим None: %A"
```

Конечно, это уродливо, но на практике встречается не слишком часто.

Гораздо важнее, что подобное уродство — признак плохого дизайна.
Иногда вычислительное выражение возвращает `'a option`, а иногда — `'a list option`.
Мы должны быть последовательны в своём дизайне, так что вычислительное выражение всегда должно иметь *один и тот же* тип, независимо от того, сколько в нём встретилось операторов `yield`.

Если мы *действительно* хотим разрешить сколько угодно операторов `yield`, то в качестве типа-обёртки мы с самого начала дожны использовать `'a list option`.
В этом случае метод `Yield` будет создавать `list option`, а метод `Combine` снова получит только одну реализацию.

Вот третья версия нашего кода.

```fsharp
type TraceBuilder() =
    member this.Bind(m, f) =
        match m with
        | None ->
            printfn "Bind с None. Выход."
        | Some a ->
            printfn "Bind с Some(%A). Продолжаем" a
        Option.bind f m

    member this.Zero() =
        printfn "Zero"
        None

    member this.Yield(x) =
        printfn "Yield незавёрнутого %A" x
        Some [x]

    member this.YieldFrom(m) =
        printfn "Yield завёрнутого (%A)" m
        m

    member this.Combine (a, b) =
        match a,b with
        | Some a', Some b' ->
            printfn "комбинируем %A с %A" a' b'
            Some (a' @ b')
        | Some a', None ->
            printfn "комбинируем %A с None" a'
            Some a'
        | None, Some b' ->
            printfn "комбинируем None с %A" b'
            Some b'
        | None, None ->
            printfn "комбинируем None с None"
            None

    member this.Delay(f) =
        printfn "Delay"
        f()

// создаём экземпляр процесса
let trace = new TraceBuilder()
```

Теперь пример работает так, как ожидается, без всяких особых трюков:

```fsharp
trace {
    yield 1
    yield 2
    } |> printfn "Результат yield с последующим yield: %A"

// Результат yield с последующим yield: Some [1; 2]

trace {
    yield 1
    yield 2
    yield 3
    } |> printfn "Результат yield x 3: %A"

// Результат yield x 3: Some [1; 2; 3]

trace {
    yield 1
    yield! None
    } |> printfn "Результат yield с последующим None: %A"

// Результат yield с последующим None: Some [1]
```

Этот код не только чище, но универсальней, поскольку вместо конкретного типа `int option` мы используем обобщённый тип `'a option`. То же самое ранее нам удалось сделать c методом `Return`.

## Перегрузка "For"

Разумный вариант, где перегрузка может оказаться полезной — метод `For`.
Возможные причины:

* Вы хотите поддерживать различные типы коллекций (т.е. `list` *и* `IEnumerable`).
* Вы хотите реализовать эффективный обход для определённых типов коллекций.
* Вы хотите «обернуть» список в другой тип (скажем, сделать `LazyList`) и собираетесь поддерживать обе версии — завёрнутую и незавёрнутую.

Вот пример построителя списков, который поддерживает не только списки, но и последовательности:

```fsharp
type ListBuilder() =
    member this.Bind(m, f) =
        m |> List.collect f

    member this.Yield(x) =
        printfn "Yield незавёрнутого %A" x
        [x]

    member this.For(m,f) =
        printfn "For %A" m
        this.Bind(m,f)

    member this.For(m:_ seq,f) =
        printfn "For %A используя seq" m
        let m2 = List.ofSeq m
        this.Bind(m2,f)

// создаём экземпляр процесса
let listbuilder = new ListBuilder()
```

А вот как его использовать:

```fsharp
listbuilder {
    let list = [1..10]
    for i in list do yield i
    } |> printfn "Результат для list: %A"

listbuilder {
    let s = seq {1..10}
    for i in s do yield i
    } |> printfn "Результат для seq : %A"
```

Если вы закомментируете второй метод `For`, пример с последовательностью начнёт приводить к ошибке компиляции.
Так что в данном случае перегрузка нужна.

## Заключение

Что ж, мы узнали, что методы можно перегружать, если нужно, но надо быть осторожными, прежде, чем бросаться в подобного рода решения с головой , потому что потребность в таком коде может быть признаком слабого дизайна.

В следующем посте мы вернёмся к основной теме и познакомимся с тонкой настройкой процесса вычисления выражений, используя отложенные вычисления *вне* построителя.