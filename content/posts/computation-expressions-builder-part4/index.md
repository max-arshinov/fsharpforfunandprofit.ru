---
layout: post
# title: "Implementing a CE: Overloading"
title: "Реализуем вычислительные выражения: Перегрузка"
# description: "Stupid method tricks"
description: "Глупые приёмы"
date: 2013-01-28
nav: thinking-functionally
# seriesId: "Computation Expressions"
seriesId: "Вычислительные выражения"
seriesOrder: 9
---

> In this post, we'll take a detour and look at some tricks you can do with methods in a computation expression builder.

В этом посте мы отвлечёмся от основной темы и взглянем на некоторые трюки, которые вы можете делать с методами в поистроителе вычислительных выражений.

> Ultimately, this detour will lead to a dead end, but I hope the journey might provide some more insight into good practices for designing your own computation expressions.

В конечном итоге этот обходной путь приведёт в тупик, но я надеюсь что путешествие может дать чуть больше прозрений относительно хороших практик при проектировании ваших собственных вычислительных выражений.

> {{<alertinfo>}}
> Note that the "builder" in the context of a computation expression is not the same as the OO "builder pattern" for constructing and validating objects.
> There is a post on the ["builder pattern" here](../builder-pattern).
> {{</alertinfo>}}

{{<alertinfo>}}
Обратите внимание, что "построитель" в контексте вычислительных выражений — это не то же самое, что объектно-ориентированный паттерн "строитель", который исползуется для конструирования и валидации объектов.
{{</alertinfo>}}

> ## An insight: builder methods can be overloaded

## Озарение: методы построиля могут быть перегружены

> At some point, you might have an insight:

В какой-то момент у вас может возникнуть озарение:

> * The builder methods are just normal class methods, and unlike standalone functions, methods can support [overloading with different parameter types](/posts/type-extensions/#method-overloading), which means we can create *different implementations* of any method, as long as the parameter types are different.

* Методы построителя это всего лишь обычные методы класса, и, в отличие от автономной функций, методы поддерживают [перегрузку с отличающимися типами параметров](/posts/type-extensions/#method-overloading), что значит, что мы можем сделать *отличающиеся реализации* любого метода, до тех пор, пока типы параметров отличаются.

> So then you might get excited about this and how it could be used.
> But it turns out to be less useful than you might think.
> Let's look at some examples.

Так что вы можете быть вдохновлены этой возможностью и тем, как её можно использовать.
Но может оказататься, что она менее полезна, чем вы можете подумать.
Давайте посмотрим на несколько примеров.

> ## Overloading "return"

## Перегрузка "Return"

> Say that you have a union type.
> You might consider overloading `Return` or `Yield` with multiple implementations for each union case.

Скажем, у вас есть тип-объединение.
Вы можете решить перегрузить `Return` или `Yield` с разными реализациями для каждого варианта объединения.

> For example, here's a very simple example where `Return` has two overloads:

Например, вот очень простой пример, где `Return` имеет две перегрузки:

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

> And here it is in use:

И вот как это можно использовать:

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

> What's wrong with this, you might think?

Что здесь не так, как вы думаете?

> Well, first, if we go back to the [discussion on wrapper types](/posts/computation-expressions-wrapper-types-part2/), we made the point that wrapper types should be *generic*.
> Workflows should be reusable as much as possible -- why tie the  implementation to any particular primitive type?

Хорошо, во-первых, если мы вернёмся к [обсуждению типов-обёрток](/posts/computation-expressions-wrapper-types-part2/), там мы отметили, что типы-обёртки должны быть *обобщёнными*.
Процессы должны быть повторно используемы настолько, насколько это возможно — зачем привязывать реализацию к какому-то конкретному примитивному типу?

> What that means in this case is that the union type should be resigned to look like this:

В данном случае это означает, что тип объединения должны быть изменён, чтобы выглядеть так:


```fsharp
type SuccessOrError<'a,'b> =
| Success of 'a
| Error of 'b
```

> But as a consequence of the generics, the `Return` method can't be overloaded any more!

Но теперь из-за обобщённых типов, метод `Return` больше не можте быть перегружен!

> Second, it's probably not a good idea to expose the internals of the type inside the expression like this anyway.
> The concept of "success" and "failure" cases is useful, but a better way would be to hide the "failure" case and handle it automatically inside `Bind`, like this:

Во-вторых, возможно в любом случае это не самая хорошая идея раскрывать внутренности типа внутри выражения, как это было сделано.
Концепция вариантов "успеха" и "неудачи" полезна, но лучшим способом было бы скрыть вариант "неудачи" и обрабатывать его автоматически внутри `Bind`, например:

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

> In this approach, `Return` is only used for success, and the failure cases are hidden.

В этом подходе `Return` используется только для "успеха", а варианты "неудачи" спрятаны.

```fsharp
successOrError {
    return 42
    } |> printfn "Результат в случае успеха: %A"

successOrError {
    let! x = Success 1
    return x/0
    } |> printfn "Результат в случае ошибки: %A"
```

> We'll see more of this technique in an upcoming post.

Мы увидим больше примеров применения такой техники в следующем посте.

> ## Multiple Combine implementations

## Множество реализаций "Combine"

> Another time when you might be tempted to overload a method is when implementing `Combine`.

Другой случае, когда у вас может возникнуть соблазн перегрузить метод — это реализация `Combine`.

> Let's revisit the `Combine` method for the `trace` workflow.
> If you remember, in the previous implementation of `Combine`, we just added the numbers together.

Давайте пересмотрим метод `Combine` для процесса `trace`.
Если вы помните, в предыдущей реализации `Combine` мы просто складывали числа.

> But what if we change our requirements, and say that:

Но что если мы изменим наши требованияя, скажем, так:

> * if we yield multiple values in the `trace` workflow, then we want to combine them into a list.

* если мы возвращаем несколько значений в процессе `trace`, мы объединяем их в список.

> A first attempt using combine might look this:

Первая попытка использовать "Combine" может выглядеть так:

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

> In the `Combine` method, we unwrap the value from the passed-in option and combine them into a list wrapped in a `Some` (e.g. `Some [a';b']`).

В методе `Combine` мы разворачиваеем значение из переданного опционального типа с комбинируем их в список-обёртку в `Some` (т.е. `Some [a';b']`).

> For two yields it works as expected:

Для двух операторов возврата это будет работать как ожидалось:

```fsharp
trace {
    yield 1
    yield 2
    } |> printfn "Результат yield и последющий yield: %A"

// Результат yield и последющий yield: Some [1; 2]
```

> And for a yielding a `None`, it also works as expected:

И для возврата `None`, это также работает как ожидалось:

```fsharp
trace {
    yield 1
    yield! None
    } |> printfn "Результат yield и последующий None: %A"

// Результат yield и последующий None: Some [1]
```

> But what happens if there are *three* values to combine? Like this:

Но что случится если есть *три* значения для комбинирования? Как здесь:

```fsharp
trace {
    yield 1
    yield 2
    yield 3
    } |> printfn "Результат yield x 3: %A"
```

> If we try this, we get a compiler error:

Если мы попробуем, то получим ошибку компиляции:

> ```text
> error FS0001: Type mismatch. Expecting a
>     int option
> but given a
>     'a list option
> The type 'int' does not match the type ''a list'
> ```

<!-- Кажется, изменился номер ошибки -->
```text
Error FS0193 : Несоответствие ограничений типов. Тип 
    "int list option"    
несовместим с типом
    "int option"
```

> What is the problem?

В чём проблема?

> The answer is that after combining the 2nd and 3rd values (`yield 2; yield 3`), we get an option containing a *list of ints* or `int list option`.
> The error happens when we attempt to combine the first value (`Some 1`) with the combined value (`Some [2;3]`).
> That is, we are passing a `int list option` as the second parameter of `Combine`, but the first parameter is still a normal `int option`.
> The compiler is telling you that it wants the second parameter to be the same type as the first.

Ответ в том, что после комбинирования 2-го и 3-го значений (`yield 2; yield 3`), мы получаем опциональное значение, содержащиее сописок целых или `int list option`.
Ошибка происходит когда мы пытаетмся комбинировать первое значение (`Some 1`) с комбинированным значением (`Some [2;3]`).
То есть, мы передаём `int list option` в качестве второго параметра `Combine`, но первый параметр всё ещё обычный `int option`.
Компилятор говорит нам, что он хочет, чтобы второй параметр имет такой же тип, как и первый.

> But, here's where we might want use our overloading trick.
> We can create *two* different implementations of `Combine`, with different types for the second parameter, one that takes an `int option` and the other taking an `int list option`.

Но здесь мы можем захотеть использовать наш трюк с перегрузкой.
Мы можем создать *две* различные имплементации `Combine` с различными типами второго параметра — одну, которая принимает `int option` и второую, которая принимает `int list option`.

> So here are the two methods, with different parameter types:

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

> Now if we try combining three results, as before, we get what we expect.

Сейчас если мы попытаемся скомбинировать три результата, как раньше, мы получаем то, что мы ожидали.

```fsharp
trace {
    yield 1
    yield 2
    yield 3
    } |> printfn "Результат yield x 3: %A"

// Результат yield x 3: Some [1; 2; 3]
```

> Unfortunately, this trick has broken some previous code!
> If you try yielding a `None` now, you will get a compiler error.

К сожалению, этот трюк сломал часть предыдущего кода!
Если сейчас вы попробуете вернуть `None`, вы получите ошибку компиляции.

```fsharp
trace {
    yield 1
    yield! None
    } |> printfn "Результа yield с последующим None: %A"
```

> The error is:

Ошибка:

> ```text
> error FS0041: A unique overload for method 'Combine' could not be determined based on type information prior to this program point. A type annotation may be needed.
> ```

```text
Невозможно определить уникальную перегрузку метода "Combine" на основе сведений о типе, заданных до данной точки программы. Возможно, требуется аннотация типа.
```

> But hold on, before you get too annoyed, try thinking like the compiler.
> If you were the compiler, and you were given a `None`, which method would *you* call?

Но подождите, прежде чем впасть в раздражение, попробуйте думать как компилятор.
Если бы вы были компилятором, и вы получили `None`, какой метод *вы* бы вызвали?

> There is no correct answer, because a `None` could be passed as the second parameter to *either* method.
> The compiler does not know where this is a None of type `int list option` (the first method) or a None of type `int option` (the second method).

Не существует правильного ответа, потому что `None` мог бы быть передан, как второй параметр в *оба* метода.
Компилятор не знает, относится ли этот `None` к типу `int list option` (первый метод) или к типу `int option` (второй метод).

> As the compiler reminds us, a type annotation will help, so let's give it one. We'll force the None to be an `int option`.

Как напоминает нам комплиятор, аннотация типа поможет, поэтому давайте её предоставим. Мы заставим `None` быть `int option`.

```fsharp
trace {
    yield 1
    let x:int option = None
    yield! x
    } |> printfn "Результат yield с последующим None: %A"
```

> This is ugly, of course, but in practice might not happen very often.

Это уродливо, конечно, но на практике встречается не слишком часто.

> More importantly, this is a clue that we have a bad design.
> Sometimes the computation expression returns an `'a option` and sometimes it returns an `'a list option`.
> We should be consistent in our design, so that the computation expression always returns the *same* type, no matter how many `yield`s are in it.

Что более важно, это признак того, что у нас плохой дизайн.
Иногда вычислительное выражение возвращает `'a option`, а иногда оно возвращает `'a list option`.
Мы должны быть последовательны в нашем дизайн, так что вычислительное выражение всегда должно иметь *тот же* тип, независимо от того, как много операторов `yield` в нём встретилось.

> That is, if we *do* want to allow multiple `yield`s, then we should use `'a list option` as the wrapper type to begin with rather than just a plain option.
> In this case the `Yield` method would create the list option, and the `Combine` method could be collapsed to a single method again.

То есть, если мы *действительно* хотим разрешить несколько `yield`, тогда мы дожны использовать `'a list option` в качестве типа-обёртки с самого начала, а не только в качестве приятного дополнения.
В этом случае метод `Yield` должен создавать `list option`, и метод `Combine` снова получил бы только одну реализацию.

> Here's the code for our third version:

Вот код нашей третьей версии.

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

> And now the examples work as expected without any special tricks:

Теперь пример работает, как мы ожидали без всяких особых трюков:

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

> Not only is the code cleaner, but as in the `Return` example, we have made our code more generic as well, having gone from a specific type (`int option`) to a more generic type (`'a option`).

Здесь не только чище коод, но и как в случае с примером `Return`, мы также сделали наш код более обобщённым, перейдя от определённого типа (`int option`) к обобщённому типу (`'a option`). 

> ## Overloading "For"

## Перегрузка "For"

> One legitimate case where overloading might be needed is the `For` method.
> Some possible reasons:

Один разумный ваирант, где может потребоваться перегрузка, это метод `For`.

> * You might want to support different kinds of collections (e.g. list *and* `IEnumerable`)
> * You might have a more efficient looping implementation for certain kinds of collections.
> * You might have a "wrapped" version of a list (e.g. LazyList) and you want support looping for both unwrapped and wrapped values.

* Вы можете захотеть поддерживать различные типы коллекций (т.е. `list` *и* `IEnumerable`).
* Вы можете иметь более эффективные реализации цикл для определённых типов коллекций.
* Вы можете иметь "завёрнутую" версию списка (т.е. LazyList) и вы можете захотеть поддерживать обе версии — для завёрнутых и для незавёрнутых значений.

> Here's an example of our list builder that has been extended to support sequences as well as lists:

Вот пример нашего построителя списков, который был расширен, чтобы поддерживать последовательности также, как и списки:

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

> And here is it in use:

И вот как его можно использовать:

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

> If you comment out the second `For` method, you will see the "sequence` example will indeed fail to compile.
> So the overload is needed.

Если вы закомментируете наш второй метод `For`, вы увидите, что пример с последовательностью будет приводить к ошбике компиялции.
Так что перегрузка в данном случае нужна.

> ## Summary

## Заключение

> So we've seen that methods can be overloaded if needed, but be careful at jumping to this solution immediately, because having to doing this may be a sign of a weak design.

Что ж, мы увидели, что методы могут быть перегружены, если нужно, но нужно быть быть осторожным, прежде, чем с головой бросать в подобного рода решения, потому что необходимость в таком коде может быть признаком слабого дизайна.

> In the next post, we'll go back to controlling exactly when the expressions get evaluated, this time using a delay *outside* the builder.

В следующем посте мы вернёмся обратно к точнкому управлению процессом вычисления выражений, используя отложенные вычисления *снаружи* построителя.