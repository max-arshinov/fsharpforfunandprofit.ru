---
layout: post
# title: "Implementing a CE: Zero and Yield"
title: "Реализуем вычислительные выражения: Zero и Yield"
# description: "Getting started with the basic builder methods"
description: "Начало работы с базовыми методами строителя"
date: 2013-01-25
nav: thinking-functionally
# seriesId: "Computation Expressions"
seriesId: "Вычислительные выражения"
seriesOrder: 6
---

> Having covered bind and continuations, and the use of wrapper types, we're finally ready to take on the full set of methods associated with "builder" classes.

Рассмотрев связывание, продолжения, а также использование типов-обёрток, мы, наконец, готовы к знакомству с полным набором методов, связанных с классами-строителями.

> {{<alertinfo>}}
> Note that the "builder" in the context of a computation expression is not the same as the OO "builder pattern" for
  constructing and validating objects.
> There is a post on the ["builder pattern" here](../builder-pattern).
> {{</alertinfo>}}

{{<alertinfo>}}
Обратите внимание, что "строитель" в контексте вычислительных выражений — это не то же самое, что объектно-ориентированный паттерн "строитель", который исползуется для конструирования и валидации объектов.
{{</alertinfo>}}

> If you look at the [MSDN documentation](http://msdn.microsoft.com/en-us/library/dd233182.aspx), you'll see not just `Bind` and `Return`, but also other strangely named methods like `Delay` and `Zero`.
> What are *they* for?
> That's what this and the next few posts will answer.

Если вы посмотрите в [документацию MSDN](http://msdn.microsoft.com/en-us/library/dd233182.aspx), вы увидите не только `Bind` и `Return`, но и другие методы со странными названиями, наподобие `Delay` и `Zero`.
Для чего *они*?
На этот вопрос ответит эта статья и несколько последующих.

> ## The plan of action

## План действий

> To demonstrate how to create a builder class, we will create a custom workflow which uses all of the possible builder methods.

Чтобы продесмонтрировать, как создать класс-строитель, мы создадим собственный процесс, который использует все возможные методы строителя.

> But rather than starting at the top and trying to explain what these methods mean without context, we'll work from the bottom up, starting with a simple workflow and adding methods only as needed to solve a problem or an error.
> In the process, you'll come to understand how F# processes computation expressions in detail.

Но вместо того, чтобы двигаться сверху вниз, и пытататься объяснить что значат эти методы без контекста, мы будем работать снизу вверх, начиная с простого процесса и добавляя методы, только когда надо справиться с какой-то проблемой или ошибкой.
В процессе, вы придёте к пониманию, как F# обрабаывает вычислительные выражения в деталях. <!-- на низком уровне -->

> The outline of this process is:

Общая схема этого процесса:

> * Part 1: In this first part, we'll look at what methods are needed for a basic workflow.
>   We'll introduce `Zero`, `Yield`, `Combine` and `For`.
> * Part 2: Next, we'll look at how to delay the execution of your code, so that it is only evaluated when needed.
>   We'll introduce `Delay` and `Run`, and look at lazy computations.
> * Part 3: Finally, we'll cover the rest of the methods: `While`, `Using`, and exception handling.

* Часть 1: В первой части мы рассмотрим, какие методы нужны для базового процесса.
  Мы представим `Zero`, `Yield`, `Combine` и `For`.
* Часть 2: Далее, мы рассмотрим, как откладывать вычисления вашего кода, чтобы они выполнялись только тогда, когда нужно.
  Мы представим `Delay` и `Run` и рассмотрим ленивые вычисления.
* Часть 3: Наконец, мы рассмотрим оставшиемся методы: `While`, `Using` и обработку исключений.

> ## Before we get started

## Перед тем, как мы начнём

> Before we dive into creating the workflow, here are some general comments.

Преджде, чем мы погрузимся в создание процесса, приведём несколько общих комментариев.

> ### The documentation for computation expressions

### Документация по вычислительным выражениям

> First, as you might have noticed, the MSDN documentation for computation expressions is meagre at best, and although not inaccurate, can be misleading.
> For example, the signatures of the builder methods are *more* flexible than they appear to be, and this can be used to implement some features that might not be obvious if you work from the documentation alone.
> We will show an example of this later.

Во-первых, как вы, возможно, заметили, документация MSDN по вычислительным выражениям в лучшем случае скудна, и, хотя не является неточной, может вводить в заблуждение.
Например, сигнатуры методов строителя *более* гибкие, чем кажутся, и это может быть использовано для реализации некоторых возможностей, который могут быть неочевидны, если вы работаете только на основании одной документации.
Мы покажем такой пример позже.

> If you want more detailed documentation, there are two sources I can recommend.
> For an detailed overview of the concepts behind computation expressions, a great resource is the [paper "The F# Expression Zoo" by Tomas Petricek and Don Syme](http://tomasp.net/academic/papers/computation-zoo/computation-zoo.pdf).
> And for the most accurate up-to-date technical documentation, you should read the [F# language specification](http://research.microsoft.com/en-us/um/cambridge/projects/fsharp/manual/spec.pdf), which has a section on computation expressions.

Если вам нужна более детальная документация, есть два источника, которые я могу порекомендовать.
Для детального обзора концепций, скрытых за кулисами вычислительных выражений, великолепный источник — [статья "Зоопарк выражений F#" Томаса Петрика и Дона Сайма](http://tomasp.net/academic/papers/computation-zoo/computation-zoo.pdf).
И, в качестве наиболее точной и актуальной технической документации, вы должны прочитать [спецификацию языка F#](http://research.microsoft.com/en-us/um/cambridge/projects/fsharp/manual/spec.pdf), в которой есть раздел про вычислительные выражения.

> ### Wrapped and unwrapped types

### Завёрнутые и незавёрнутые типы

> When you are trying to understand the signatures as documented, remember that what I have been calling the "unwrapped" type is normally written as `'T` and the "wrapped" type is normally written `M<'T>`.
> That is, when you see that the `Return` method has the signature `'T -> M<'T>` it means `Return` takes an unwrapped type and returns a wrapped type.

Когда вы пытаетесь понять сигнатуры в документации, помните, что то, что я называю "незавёрнутым" типом, обычно записывается как `'T`, а то, что называю "завёрнутым" — как `M<'T>`.
То есть, когда вы выидите, что метод `Return` имеет сигнатуру `'T -> M<'T>`, это значит, что `Return` получает незавёрнутый тип и возвращает завёрнутый.

> As I have in the earlier posts in this series, I will continue to use "unwrapped" and "wrapped" to describe the relationship between these types, but as we move forward these terms will be stretched to the breaking point, so I will also start using other terminology, such as "computation type" instead of "wrapped type".
> I hope that when we reach this point, the reason for the change will be clear and understandable.

Как и ранее в постах этой серии, я продолжу использовать "незавёрнутый" и "завёрнутый", чтобы оописать взаимоотношения между этими типами, но по мере того, как мы будем двигаться вперёд, эти термины будут выжаты до предела, так что я также начну использовать другую терминлогию, в частности "вычислительный тип" вместо "завёрнутый тип".
Я надеюсь, что когда мы достигнем этой точки, причина изменения <!-- терминлогии --> станет ясной и понимаемой.

> Also, in my examples, I will generally try to keep things simple by using code such as:

Также в моих примерах я обычно буду пытаться сохранять простоту, используя код наподобие такого:

```fsharp
let! x = ...значение типа-обёртки...
```

> But this is actually an oversimplification.
> To be precise, the "x" can be any *pattern* not just a single value, and the "wrapped type" value can, of course, be an *expression* that evaluates to a wrapped type.
> The MSDN documentation uses this more precise approach.
> It uses "pattern" and "expression" in the definitions, such as `let! pattern = expr in cexpr`.

Но в дествительности это чрезмерное упрощение.
Чтобы быть точным, `x` может быть любым *образцом*, а не просто значением, а значение "завёрнутого типа" может, конечно, быть выражением, которое вычисляет завёрнутый тип.
Документация MSDN использует этот, более точный подход.
Она использует "образец" и "выражение" в определениях, скажем, `let! pattern = expr in cexpr`.

> Here are some examples of using patterns and expressions in a `maybe` computation expression, where `Option` is the wrapped type, and the right hand side expressions are `options`:

Вот несколько примеров использования образцов и выражений в вычислительном выражении `maybe`, где `Option` — это тип-обёртка, а в правой части выражений находятся опциональные значения:

```fsharp
// let! pattern = expr in cexpr
maybe {
    let! x,y = Some(1,2)
    let! head::tail = Some( [1;2;3] )
    // и т.д.
    }
```

> Having said this, I will continue to use the oversimplified examples, so as not to add extra complication to an already complicated topic!

Говоря это, я продолжу использовать чрезмерно упрощённые примеры, чтобы не добавять лишней сложности к теме, которая и так сложная!

> ### Implementing special methods in the builder class (or not)

### Импрементация особых методов в классе-строителе (или нет)

> The MSDN documentation shows that each special operation (such as `for..in`, or `yield`) is translated into one or more calls to methods in the builder class.

Документация MSDN показывает, что каждая особая операция (скажем, `for..in` или `yield`) транслируется в один или больше вызовов методов класса-строителя.

> There is not always a one-to-one correspondence, but generally, to support the syntax for a special operation, you *must* implement a corresponding method in the builder class, otherwise the compiler will complain and give you an error.

Не всегда есть соответствие один-к-одному, но в целом, чтобы поддерживать синтаксис для особой операции, вы *должны* реализовать соответствующий метод в классе-строителе, иначе компилятор будет жаловаться и выдаст вам ошибку.

> On the other hand, you do *not* need to implement every single method if you don't need the syntax.
> For example, we have already implemented the `maybe` workflow quite nicely by only implementing the two methods `Bind` and `Return`.
> We don't need to implement `Delay`, `Use`, and so on, if we don't need to use them.

С другой стороны, вы не *не обязаны* реализовывать каждый отдельный метод, если вам не нужен <!-- соответствующий --> синтаксис.
Например, мы уже достаточно хорошо реализовали процесс `maybe`, написав всего лишь два метода `Bind` и `Return`.
Нам не обязательно реализовывать `Delay`, `Use` и другие методы, если мы не хотим их использовать.

> To see what happens if you have not implemented a method, let's try to use the `for..in..do` syntax in our `maybe` workflow like this:

Чтобы увидеть, что случиться, если вы не реализовали какой-то метод, давайте попробуем синтаксис `for..in..do` в нашем процессе `maybe`, как здесь:

```fsharp
maybe { for i in [1;2;3] do i }
```

> We will get the compiler error:

Вы получим ошибку компиляции.

> ```text
> This control construct may only be used if the computation expression builder defines a 'For' method
> ```

```text
Конструкция данного элемента управления может использоваться только в том случае, если построитель вычислительного выражения определяет метод "For"
```

> Sometimes you get will errors that might be cryptic unless you know what is going on behind the scenes.
> For example, if you forget to put `return` in your workflow, like this:

Иногда вы получите ошибки, которые могут быть критическими, пока вы не знаете, что проиисходит за кулисами.
Например, если вы забыти вставить `return` в свой процесс, как здесь:

```fsharp
maybe { 1 }
```

> You will get the compiler error:

Вы получите ошибку компиляции:

> ```text
> This control construct may only be used if the computation expression builder defines a 'Zero' method
> ```

```text
онструкция данного элемента управления может использоваться только в том случае, если построитель вычислительного выражения определяет метод "Zero"
```

> You might be asking: what is the `Zero` method?
> And why do I need it?
> The answer to that is coming right up.

Вы можете спросить: что за метод `Zero`?
И почему он мне нужен?
Ответ на этот вопрос скоро будет.

> ### Operations with and without '!'

### Операции с и без '!'

> Obviously, many of the special operations come in pairs, with and without a "!" symbol.
> For example: `let` and `let!` (pronounced "let-bang"), `return` and `return!`, `yield` and `yield!` and so on.

Очевидно <!-- бросается в глаза, что --> многие особые операции идут парами: с и без символа "!".
Например: `let` и `let!` (произностися "лет-бэнг"), `return` и `return!`, `yield` и `yield!`, и так далее.

> The difference is easy to remember when you realize that the operations *without* a "!" always have *unwrapped* types on the right hand side, while the ones *with* a "!" always have *wrapped* types.

Различие легко запомнить, когда вы осознаёте, что операции *без* "!" всегда имеют *незавёрнутые* типы в правой части, в то время как операции *с* "!" — завёрнутые типы.

> So for example, using the `maybe` workflow, where `Option` is the wrapped type, we can compare the different syntaxes:

Так, например, используя процесс `maybe`, где `Option` — это тип-обёртка, мы можем сравнить различный синтаксис:

```fsharp
let x = 1           // 1 это "незавёрнутый" тип
let! x = (Some 1)   // Some 1 это "завёрнутый" тип
return 1            // 1 это "незавёрнутый" тип
return! (Some 1)    // Some 1 это "завёрнутый" тип
yield 1             // 1 это "незавёрнутый" тип
yield! (Some 1)     // Some 1 это "завёрнутый" тип
```

> The "!" versions are particularly important for composition, because the wrapped type can be the result of *another* computation expression of the same type.

Версии с "!" особенно важны для композиции, поскольку тип-обёртка может быть результатом *другого* вычислительного выражения того же типа.

```fsharp
let! x = maybe {...)       // "maybe" возвращает "завёрнутый" тип

// связываем с другими процессом такого типа, используя let!
let! aMaybe = maybe {...)  // создаём "завёрнутый" тип
return! aMaybe             // возвращаем его

// связываем два дочерних асинка в родительском асинке, используя let!
let processUri uri = async {
    let! html = webClient.AsyncDownloadString(uri)
    let! links = extractLinks html
    ... etc ...
    }
```

> ## Diving in - creating a minimal implementation of a workflow

## Погружаемся глубже — создаём минимальную реализацию процесса

> Let's start!
> We'll begin by creating a minimal version of the "maybe" workflow (which we'll rename as "trace") with every method instrumented, so we can see what is going on.
> We'll use this as our testbed throughout this post.

Давайте начнём!
Мы начём с создания минимальной версии процесса "maybe" (который мы переименуем в "trace"), где каждый метод оснащён, так что мы можем видеть, что происходит. <!-- фактически, где каждый метод выводит трассирующие сообщения -->

> Here's the code for the first version of the `trace` workflow:

Вот код первой версии процесса `trace`:

```fsharp
type TraceBuilder() =
    member this.Bind(m, f) =
        match m with
        | None ->
            printfn "Связывание с None. Выход."
        | Some a ->
            printfn "Связывание с Some(%A). Продолжение" a
        Option.bind f m

    member this.Return(x) =
        printfn "Заворачивание и возврат незавёрнутого %A." x
        Some x

    member this.ReturnFrom(m) =
        printfn "Возврат завёрнутого значения (%A)." m
        m

// создаём экземпляр процесса
let trace = new TraceBuilder()
```

> Nothing new here, I hope. We have already seen all these methods before.

Здесь нет ничего нового, я надеюсь. Мы уже рассматривали все эти методы раньше.

> Now let's run some sample code through it:

Теперь давайте прогоним несколько примеров кода <!-- через этот процесс -->.

```fsharp
trace {
    return 1
    } |> printfn "Результат 1: %A"

trace {
    return! Some 2
    } |> printfn "Результат 2: %A"

trace {
    let! x = Some 1
    let! y = Some 2
    return x + y
    } |> printfn "Результат 3: %A"

trace {
    let! x = None
    let! y = Some 1
    return x + y
    } |> printfn "Результат 4: %A"
```

> Everything should work as expected, in particular, you should be able to see that the use of `None` in the 4th example caused the next two lines (`let! y = ... return x+y`) to be skipped and the result of the whole expression was `None`.

Всё должно работать, как ожидалось, в частности, вы должны увидеть, что использование `None` в четвёртом примере привело к тому, что следующие две строки (`let y = ... return x+y`) были пропущены и результатом всего выражения стал `None`.

> ## Introducing "do!"

## Введение в "do!"

> Our expression supports `let!`, but what about `do!`?

Наше выражение поддерживает `let!`, но что насчёт `do!`?

> In normal F#, `do` is just like `let`, except that the expression doesn't return anything useful (namely, a unit value).

В нормальном F# `do` — это аналог `let`, за исключением того, что выражение не возвращает ничего полезного (буквально, возвращает значение типа `unit`).

> Inside a computation expression, `do!` is very similar.
> Just as `let!` passes a wrapped result to the `Bind` method, so does `do!`, except that in the case of `do!` the "result" is the unit value, and so a *wrapped* version of unit is passed to the bind method.

В вычислительных выражениях `do!` очень похож.
Как `let!` передаёт обёрнутое значение в метод `Bind`, так и `do!`, за исключением того, что в случае с `do!` результат — это значение типа `unit`, так что в метод `Bind` передаётся "завёрнутая" версия `unit`.

> Here is a simple demonstration using the `trace` workflow:

Вот простая демаонстрация, используя процесс `trace`:

```fsharp
trace {
    do! Some (printfn "...выражение типа unit")
    do! Some (printfn "...ещё одно выражение типа unit")
    let! x = Some (1)
    return x
    } |> printfn "Результ do: %A"
```

> Here is the output:

А вот вывод:

> ```text
> ...expression that returns unit
> Binding with Some(<null>). Continuing
> ...another expression that returns unit
> Binding with Some(<null>). Continuing
> Binding with Some(1). Continuing
> Returning a unwrapped 1 as an option
> Result from do: Some 1
> ```

```text
> ...выражение типа unit
> Связывание с Some(<null>). Продолжение.
> ...ещё одно выражение типа unit
> Связывание с Some(<null>). Продолжение.
> Связывание с Some(1). Продолжение.
> Заворачивание и возврат незавёрнутого 1
> Результат do: Some 1
```

> You can verify for yourself that a `unit option` is being passed to `Bind` as a result of each `do!`.

Вы можете самостоятельно убедиться, что `unit option` передаётся в `Bind`, как результат каждого `do!`.

> ## Introducing "Zero"

## Введение в "Zero"

> What is the smallest computation expression you can get away with? Let's try nothing at all:

Какое наименьшее вычислительное выражение в принципе может зарабоать? Давайте попробуем вообще без ничего:

```fsharp
trace {
    } |> printfn "Результат пустого процесса: %A"
```

> We get an error immediately:

Мы немедленно получаем ошибку:

> ```text
> This value is not a function and cannot be applied
> ```

```text
Это значение не является функцией, и применить его невозможно.
```

> Fair enough.
> If you think about it, it doesn't make sense to have nothing at all in a computation expression.
> After all, its purpose is to chain expressions together.

Достаточно справедливо.
Если вы подумаете об этом, нет смысла в том, чтобы вычислительное выражение содержало хоть что-то.
В конце концов, его цель — объединять цепочку выражений вместе. 

> Next, what about a simple expression with no `let!` or `return`?

Далее, что на счёт простого выражения без `let!` или `return`?

```fsharp
trace {
    printfn "привет мир"
    } |> printfn "Результат простого выражения: %A"
```

> Now we get a different error:

Теперь мы получаем другую ошибку:

> ```text
> This control construct may only be used if the computation expression builder defines a 'Zero' method
> ```

```text
Конструкция данного элемента управления может использоваться только в том случае, если построитель вычислительного выражения определяет метод "Zero"
```

> So why is the `Zero` method needed now but we haven't needed it before?
> The answer is that in this particular case we haven't returned anything explicitly, yet the computation expression as a whole *must* return a wrapped value.
> So what value should it return?

Так почему вдруг метод `Zero` потребовался сейчас, хотя раньше он был не нужен?
Ответ в данном конкретном случае заключается в том, что мы не возвращали ничего в явном виде, хотя вычислительное выражение, в целом, *должно* вернуть какое-то завёрнутое значение.
И какое значение оно должен вернуть?

> In fact, this situation will occur any time the return value of the computation expression has not been explicitly given.
> The same thing happens if you have an `if..then` expression without an else clause.

Фактически, эта ситуация возникает всякий раз, когда возвращаемое значение вычислительного выражения не было предоставлено в явном виде.
Похожая штука происходит, если у ва сесть выражение `if..then` без клаузы `else`.

```fsharp
trace {
    if false then return 1
    } |> printfn "Результат if без else: %A"
```

> In normal F# code, an "if..then" without an "else" would result in a unit value, but in a computation expression, the particular return value must be a member of the wrapped type, and the compiler does not know what value this is.

В нормальном коде F# `if..then` без `else` должен возвращать значение `unit, но в вычислительном выражении конкретное возвращаемое значение должно быть членом типа-обёртки <!-- должно иметь тип-обёртку, оборачивающий тип -->, и компилятор не знает, какое это должно быть значение.

> The fix is to tell the compiler what to use -- and that is the purpose of the `Zero` method.

Исправление состоит в том, чтобы сказать компилятору, что использовать — и это и есть цель метода `Zero`.

> ### What value should you use for Zero?

### Какое значение вы должны использовать для Zero?

> So which value *should* you use for `Zero`?
> It depends on the kind of workflow you are creating.

Так какое значение вы *должны* использовать для `Zero`?
Это зависит от того, какого рода процесс вы создаёте.

> Here are some guidelines that might help:

Вот несколько рекомендаций, которые могут помочь:

> * **Does the workflow have a concept of "success" or "failure"?**
>   If so, use the "failure" value for `Zero`.
>   For example, in our `trace` workflow, we use `None` to indicate failure, and so we can use `None` as the Zero value.
> * **Does the workflow have a concept of "sequential processing"?**
>   That is, in your workflow you do one step and then another, with some processing behind the scenes.
>   In normal F# code, an expression that did not return anything explicitly would evaluate to unit.
>   So to parallel this case, your `Zero` should be the *wrapped* version of unit.
>   For example, in a variant on an option-based workflow, we might use `Some ()` to mean `Zero` (and by the way, this would always be the same as `Return ()` as well).
> * **Is the workflow primarily concerned with manipulating data structures?**
>   If so, `Zero` should be the "empty" data structure.
>   For example, in a "list builder" workflow, we would use the empty list as the Zero value.

* **Есть в процессе концепции "успех" и "неудача"?**
  Если да, используйте значение "неудача" для `Zero`.
  Например, в нашем процессе `trace`, мы используем `None`, чтобы сообщить о неудаче, так что мы можем использовать `None` как значение для `Zero`.
* **Есть ли в процессе концепция "последовательной обработки"?**
  То есть, в своём процессе вы делаете один шаг за другим, с какой-то дололнительной обработкой за кулисами?
  В обычном коде F# выражение, которое не возвращает ничего, будет иметь значение `unit`.
  Чтобы провести параллель с этим случаем, ваш метод `Zero` должен быть *завёрнутой* версией `unit`.
  Например, в случае опционального процесса, мы могли бы использовать `Some ()` для `Zero` (и, кстати, это всегда было бы то же самое, что и `Return ()`).
* **Связан ли процесс в первую очередь с манипулированием структурой данных?**
  Если да, `Zero` должно быть "пустым" экземпляром этой структуры данных.
  Например, в процессе "строитель списков" мы могли бы использовать пустой список в качестве значения `Zero`.

> The `Zero` value also has an important role to play when combining wrapped types.
> So stay tuned, and we'll revisit Zero in the next post.

`Zero` также играет важную роль при комбинировании типов-обёрток.
Оставайтесь на связи: мы вернёмся к `Zero` в следующем посте.

> ### A Zero implementation

### Реализация Zero

> So now let's extend our testbed class with a `Zero` method that returns `None`, and try again.

Теперь давайте расширим наш испытательный класс <!-- буквально: класс-испытательный стенд--> методом `Zero`, который возвращает `None` и попробуем снова.

```fsharp
type TraceBuilder() =
    // другие методы, как раньше
    member this.Zero() =
        printfn "Zero"
        None

// создаём новый экземпляр
let trace = new TraceBuilder()

// проверяем
trace {
    printfn "hello world"
    } |> printfn "Результат простого выражения: %A"

trace {
    if false then return 1
    } |> printfn "Результат if без else: %A"
```

> The test code makes it clear that `Zero` is being called behind the scenes.
> And `None` is the return value for the expression as whole.
> *Note: `None` may print out as `<null>`. You can ignore this.*

Тестовый код делает прозрачным, что `Zero` вызывается за кулисами.
И `None` — это результат всего вычисления в целом.
*Заметьте: `None` может выводиться как `<null>`. На это можно не обращать внимания.*

> ### Do you always need a Zero?

### Всегда ли вам нужен Zero?

> Remember, you *not required* to have a `Zero`, but only if it makes sense in the context of the workflow.
> For example `seq` does not allow zero, but `async` does:

Помните, что вы *не обязаны* добавлять метод `Zeor`, если это не имеет смысла в контексте процесса.
Скажем, `seq` не имеет его, а `async` имеет:

```fsharp
let s = seq {printfn "zero" }    // Ошибка
let a = async {printfn "zero" }  // Работает
```

> ## Introducing "Yield"

## Введение в "Yield"

> In C#, there is a "yield" statement that, within an iterator, is used to return early and then picks up where you left off when you come back.

В C# есть оператор "yield", который используется для раннего возврата значений из итератора, и затем возобновляет работу с того места, где она была прервана.

> And looking at the docs, there is a "yield" available in F# computation expressions as well.
> What does it do?
> Let's try it and see.

И, глядя на документацию <!-- мы видимо, что --> "yield" точно также доступен в вычислительных выражениях F#.
Что он делает?
Давайте поэкспериментируем с ним, и узнаем.

```fsharp
trace {
    yield 1
    } |> printfn "Результат yield: %A"
```

> And we get the error:

И мы получим ошибку:

> ```text
> This control construct may only be used if the computation expression builder defines a 'Yield' method
> ```

```test
Конструкция данного элемента управления может использоваться только в том случае, если построитель вычислительного выражения определяет метод "Yield"
```

> No surprise there.
> So what should the implementation of "yield" method look like?
> The MSDN documentation says that it has the signature `'T -> M<'T>`, which is exactly the same as the signature for the `Return` method.
> It must take an unwrapped value and wrap it.

Никаких сюрпризов.
Так на что должна быть похожа реализация метода "yield"?
Документация MSDN говорит, что он имеет сигнатуру `'T -> M<'T>`, которая в точности совпадает с сигнатурой метода `Return`.
Он долже получить незавёрнутое значение и завернуть его.

> So let's implement it the same way as `Return` and retry the test expression.

Так что сделаем такую же реализацию, как и у метода `Return` и повторим эксперимент.

```fsharp
type TraceBuilder() =
    // другие методы, как раньше

    member this.Yield(x) =
        printfn "Заворачивание и ранний возврат незавёрнутого %A" x
        Some x

// создаём новый экземпляр
let trace = new TraceBuilder()

// проверяем
trace {
    yield 1
    } |> printfn "Результат для yield: %A"
```

> This works now, and it seems that it can be used as an exact substitute for `return`.

Теперь это работает, и выглядит, как полная замена для `return`.

> There is a also a `YieldFrom` method that parallels the `ReturnFrom` method.
> And it behaves the same way, allowing you to yield a wrapped value rather than a unwrapped one.

Существует также метод `YieldFrom` параллельно с методом `ReturnFrom`.
И он ведёт себя точно также, позволяя вам возвращать завёрнутое значение вмето незавёрнутого.

> So let's add that to our list of builder methods as well:

Добавим и этот метод к нашему списку методов:

```fsharp
type TraceBuilder() =
    // другие методы, как раньше

    member this.YieldFrom(m) =
        printfn "Ранний возврат завёрнутого значения (%A)" m
        m

// создаём новый экземпляр
let trace = new TraceBuilder()

// проверяем
trace {
    yield! Some 1
    } |> printfn "Результат yield!: %A"
```

> At this point you might be wondering: if `return` and `yield` are basically the same thing, why are there two different keywords?
> The answer is mainly so that you can enforce appropriate syntax by implementing one but not the other.
> For example, the `seq` expression *does* allow `yield` but *doesn't* allow `return`, while the `async` does allow `return`, but does not allow `yield`, as you can see from the snippets below.

Сейчас вам может быть интересно: если `return` и `yield` делают одно и то же, почему для них предусмотрены разные ключевые слова?
Ответ заключается главным образом в том, что вы можете обеспечить нужный вам синтаксис, реализуя или первый или второй.
Например, выражение `seq` *разрешает* `yield`, но *не разрешает* `return`, в то время, как `async` разрешает `return`, но не разрешает `yield`, как вы можете видеть из фрагментов ниже.

```fsharp
let s = seq {yield 1}    // Работает
let s = seq {return 1}   // Не работает

let a = async {return 1} // Работает
let a = async {yield 1}  // Не работает
```

> In fact, you could create slightly different behavior for `return` vs. `yield`, so that, for example, using `return` stops the rest of the computation expression from being evaluated, while `yield` doesn't.

На самом деле, вы можете создать немного отличающееся поведение для `return` и `yield`, такое, что, например, использование `return` останавливало <!-- отбрасывало --> остаток вычислительного выражения от исполнения, в то время, как `yield` — не останавливало бы.

> More generally, of course, `yield` should be used for sequence/enumeration semantics, while `return` is normally used once per expression.
> (We'll see how `yield` can be used multiple times in the next post.)

В целом, конечно, `yield` должен быть исползован для сематники последовательности/перебора, в то время как `return` обычно используется один раз в конце выражения.
(Мы увидим, как `yield` может быть использован несколько раз в следующем посте).

> ## Revisiting "For"

## Возвращаясь к "For"

> We talked about the `for..in..do` syntax in the last post.
> So now let's revisit the "list builder" that we discussed earlier and add the extra methods.
> We already saw how to define `Bind` and `Return` for a list in a previous post, so we just need to implement the additional methods.

Мы говорили о синтаксисе `for..in..do` в последнем посте.
Давайте сейчас вернёмся к "строителю списков", который мы обсуждали ранее и добавим дополнительные методы.
Мы уже видели, как определить `Bind` и `Return` для списка в предыдущем посте,, так что нам надо всего лишь реализовать дополнительные методы.

> * The `Zero` method just returns an empty list.
> * The `Yield` method can be implemented in the same way as `Return`.
> * The `For` method can be implemented the same as `Bind`.

* Метод `Zero` просто возвращает пустой список.
* Метод `Yield` может быть реализован также, как и `Return`.
* Метод `For` может быть реализован также, как и `Bind`.

```fsharp
type ListBuilder() =
    member this.Bind(m, f) =
        m |> List.collect f

    member this.Zero() =
        printfn "Zero"
        []

    member this.Return(x) =
        printfn "Возврат незавёрнутого %A в виде списка" x
        [x]

    member this.Yield(x) =
        printfn "Ранний возврат незавёрнутого %A в виде списка" x
        [x]

    member this.For(m,f) =
        printfn "Для %A" m
        this.Bind(m,f)

// создаём экземпляр процесса
let listbuilder = new ListBuilder()
```

> And here is the code using `let!`:

Вот код, использующий `let!`:

```fsharp
listbuilder {
    let! x = [1..3]
    let! y = [10;20;30]
    return x + y
    } |> printfn "Результат: %A"
```

> And here is the equivalent code using `for`:

А вот его эквивалент с использованием `for`:

```fsharp
listbuilder {
    for x in [1..3] do
    for y in [10;20;30] do
    return x + y
    } |> printfn "Результат: %A"
```

> You can see that both approaches give the same result.

Вы видите, что оба подхода дают один и тот же вариант.

> ## Summary

## Заключение

> In this post, we've seen how to implement the basic methods for a simple computation expression.

В этом посте мы увидели, как реализовать базовые методы для простого вычислительного выражения.

> Some points to reiterate:

Некоторые момент для повторения <!-- закрепления -->:

> * For simple expressions you don't need to implement all the methods.
> * Things with bangs have wrapped types on the right hand side.
> * Things without bangs have unwrapped types on the right hand side.
> * You need to implement `Zero` if you want a workflow that doesn't explicitly return a value.
> * `Yield` is basically equivalent to `Return`, but `Yield` should be used for sequence/enumeration semantics.
> * `For` is basically equivalent to `Bind` in simple cases.

* Для простых выражений вы не должны реализовывать все методы.
* Методы с восклицательным знаком принимают завёрнутые типы в правой части выражения.
* Методы без восклицательного знакак принимают незавёрнутые типы в правой части выражения.
* Вам надо реализовать `Zero`, если вам нужен процесс, который не возвращает значения в явном виде.
* `Yield`, в целом, является эквивалентом `Return`, но `Yield` должен быть использован для семантики последовательности/перебора.
* `For`, в целом, является эквивалентом `Bind` в простых случаях.

> In the next post, we'll look at what happens when we need to combine multiple values.

В следующем посте мы узнаем, что происходит, когда нам надо комбинировать несколько значений.
