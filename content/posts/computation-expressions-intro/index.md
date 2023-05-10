---
layout: post
# title: "Computation expressions: Introduction"
title: "Вычислительные выражения: Введение"
# description: "Unwrapping the enigma..."
description: "Разгадывая загадку..."
date: 2013-01-20
nav: thinking-functionally
# seriesId: "Computation Expressions"
seriesId: "Вычислительные выражения"
seriesOrder: 1
---

> By popular request, it is time to talk about the mysteries of computation expressions, what they are, and how
> they can be useful in practice (and I will try to avoid using the [forbidden m-word](/about/#banned)).

По многочисленным просьбам, мы поговорим про тайны вычислительных выражений, о том, что они из себя представляют и как могут применяться на практике (и я постараюсь избегать [запрещённого слова на букву М](/about/#banned)).

<!-- Речь про монады -->

> In this series, you'll learn what computation expressions are, how to make your own, and some common patterns
> involving them. In the process, we'll also look at continuations, the bind function, wrapper types, and more.

В этом цикле статей вы узнаете, что такое вычислительные выражения, как их создавать, а также освоите несколько общих паттернов, связанных с ними. В процессе мы также познакомимся с продолжениями, функцией связывания, типами-обёртками и прочим.

> ## Background ##
> Computation expressions seem to have a reputation for being abstruse and difficult to understand.

Кажется, что вычислительные выражениях имеют репутацию заумной штуки, трудной для понимания.

> On one hand, they're easy enough to use. Anyone who has written much F# code has certainly used standard ones
> like `seq{...}` or `async{...}`.

С одной стороны, их достаточно легко использовать. Любой, кто написал достаточно кода на F# наверняка использовал стандартные конструкции, такие как `seq{...}` или `async{...}`. 

> But how do you make a new one of these things? How do they work behind the scenes?

Но как вы можете создать новую похожую конструкцию? Как они работают за кулисами?

> Unfortunately, many explanations seem to make things even more confusing.  There seems to be some sort of
> mental bridge that you have to cross.
> Once you are on the other side, it is all obvious, but to someone on this side, it is baffling.

К сожалению, кажется, что многие объяснения делают весь гораздо более запутанными. Кажется, что существует своеобразный ментальный мост, который вы должны пересечь. Для тех, кто на другой стороне, всё кажется очевидным, но те, кто на этой, сбиты с толку.

> If we turn for guidance to the
> [official MSDN documentation](http://msdn.microsoft.com/en-us/library/dd233182.aspx), it is explicit,
> but quite unhelpful to a beginner.

Если мы обратимся за помощью к [официальной документации MSDN](http://msdn.microsoft.com/en-us/library/dd233182.aspx), мы обнаружим, что она точна, но достаточно бесполезна для начинающего.

> For example, it says that when you see the following code within a computation expression:

В частности, она говорит, что когда мы видим подобный код в вычислительном выражении:

```fsharp
{| let! pattern = expr in cexpr |}
```

то это на самом деле синтаксический сахар для такого вызова:

> it is simply syntactic sugar for this method call:

```fsharp
builder.Bind(expr, (fun pattern -> {| cexpr |}))
```

> But... what does this mean exactly?

Но... что это такое и для чего нужно?

> I hope that by the end of this series, the documentation above will become obvious.  Don't believe me? Read on!

Я надеюсь, что к концу цикла, документация, пример которой приведён выше, станет очевидной. Не верите? Читайте дальше!

> ## Computation expressions in practice ##

## Вычислительные выражения на практике

> Before going into the mechanics of computation expressions, let's look at a few trivial examples that show the
> same code before and after using computation expressions.

Прежде чем погружаться в глубины вычислительных выражений, рассмотрим несколько тривиальных примеров, которые показывают один и тот же код с вычислительными выражениями и без них.

> Let's start with a simple one.  Let's say we have some code, and we want to log each step. So we define a
> little logging function, and call it after every value is created, like so:

Начнём с простого примера. Представим, что у нас есть код и мы хотим логировать каждый шаг.
Мы пишем небольшую функцию логирования, и вызываем её после каждого вычисления.

```fsharp
let log p = printfn "expression is %A" p

let loggedWorkflow =
    let x = 42
    log x
    let y = 43
    log y
    let z = x + y
    log z
    //return
    z
```

> If you run this, you will see the output:

Если вы запустите эту программу, вы увидите:

```text
expression is 42
expression is 43
expression is 85
```

> Simple enough.

Совсем несложно.

> But it is annoying to have to explicitly write all the log statements each time. Is there a way to hide them?

Но раздражает, что всё время приходится вызывать функцию логирования. Существует ли способ спрятать этот вызов?

> Funny you should ask... A computation expression can do that. Here's one that does exactly the same thing.

Хорошо, что вы спросили... Вычислительные выражения помогут справиться с проблемой. Вот код, которые будет делать то же самое.

> First we define a new type called `LoggingBuilder`:

Для начала определим новый тип `LoggingBuilder`:

```fsharp
type LoggingBuilder() =
    let log p = printfn "expression is %A" p

    member this.Bind(x, f) =
        log x
        f x

    member this.Return(x) =
        x
```

> *Don't worry about what the mysterious `Bind` and `Return` are for yet -- they will be explained soon.*

*Пока не беспокойтесь по поводу таинственных `Bind` и `Return` — мы обязательно вернёмся к ним позже.*

{{<alertinfo>}}
> Note that the "builder" in the context of a computation expression is not the same as the OO "builder pattern"
> for constructing and validating objects.
> There is a post on the ["builder pattern" here](../builder-pattern).
{{</alertinfo>}}


{{<alertinfo>}}
> Обратие внимание, что *строитель* (builder) в контексте вычислительных выражений это не то же самое,
> что объектно-ориентированный паттерн *Строитель*, который применяется для создания сложных объектов.

> Про паттерн *Строитель* вы можете прочитать [здесь](../builder-pattern).
{{</alertinfo>}}

> Next we create an instance of the type, `logger` in this case.

Затем мы создадим экземпляр объявленного типа, в нашем случае `logger`.

```fsharp
let logger = new LoggingBuilder()
```

> So with this `logger` value, we can rewrite the original logging example like this:

И теперь, имея переменную `logger`, мы можем переписать оригинальный пример вот так:

```fsharp
let loggedWorkflow =
    logger
        {
        let! x = 42
        let! y = 43
        let! z = x + y
        return z
        }
```

> If you run this, you get exactly the same output, but you can see that the use of the `logger{...}` workflow
> has allowed us to hide the repetitive code.

Запустив код, вы увидите на экране то же самое, но, как вы могли заметить, использование конструкции `logger{...}` позволило нам избавиться от повторяющегося кода.

> ### Safe division ###

### Безопасное деление

> Now let's look at an old chestnut.

Теперь давайте разберёмся с одной бородатой историей.
<!-- Old chestnut — буквально "старый каштан". В переносмном смысле "старая история", "старый анекдот". -->

> Say that we want to divide a series of numbers, one after another, but one of them might be zero. How can we
> handle it? Throwing an exception is ugly.  Sounds like a good match for the `option` type though.

Представим, что нам надо разделить друг на друга несколько чисел, но одно из них может быть равно нулю. Как нам обработать возможную ошибку? Можно выбросить исключения, но это будет выглядеть достаточно уродливо. Скорее, здесь подошёл бы тип `option`.

> First we need to create a helper function that does the division and gives us back an `int option`.
> If everything is OK, we get a `Some` and if the division fails, we get a `None`.

Вначале нам надо написать вспомогательную функцию, которая делит числа друг на друга и возвращает результат типа `int option`. Если всё прошло нормально, мы получаем какое то (`Some`) значение, а если нет — не получаем ничего (`None`).

> Then we can chain the divisions together, and after each division we need to test whether it failed or not,
> and keep going only if it was successful.

Затем мы объединяем деления в цепочку и после каждого деления проверяем результат, продолжая только в случае успеха.

> Here's the helper function first, and then the main workflow:

Сначала напишем вспомогательную функцию, а потом основной код.

```fsharp
let divideBy bottom top =
    if bottom = 0
    then None
    else Some(top/bottom)
```

> Note that I have put the divisor first in the parameter list. This is so we can write an expression like
> `12 |> divideBy 3`, which makes chaining easier.

Обратите внимание, что первым в списке параметров мы поставили делитель. Это позволит нам записывать выражения в виде `12 |> divideBy 3`, благодаря чему мы сможем объединять их в цепочку.

> Let's put it to use. Here is a workflow that attempts to divide a starting number three times:

Теперь используем нашу функцию. Вот код, где начальное значение последовательно делится на три числа.

```fsharp
let divideByWorkflow init x y z =
    let a = init |> divideBy x
    match a with
    | None -> None  // останавливаемся
    | Some a' ->    // продолжаем
        let b = a' |> divideBy y
        match b with
        | None -> None  // останавливаемся
        | Some b' ->    // продолжаем
            let c = b' |> divideBy z
            match c with
            | None -> None  // останавливаемся
            | Some c' ->    // продолжаем
                // возвращаем результат
                Some c'
```

> And here it is in use:

А здесь мы его используем:

```fsharp
let good = divideByWorkflow 12 3 2 1
let bad = divideByWorkflow 12 3 0 1
```

> The `bad` workflow fails on the third step and returns `None` for the whole thing.

Некорректная цепочка делителей вызовет ошибку на третьем шаге и вернёт `None` как результат всего выражения.

> It is very important to note that the *entire workflow* has to return an `int option` as well. It can't just
> return an `int` because what would it evaluate to in the bad case?
> And can you see how the type that we used "inside" the workflow, the option type, has to be the same type that
> comes out finally at the end. Remember this point -- it will crop up again later.

Важно обратить внимание, что *выражение целиком* также должно иметь тип `int option`. Оно не может быть просто целым, потому что тогда непонятно, чему оно должно быть равно в случае ошибки. Как видите, тип, который мы используем внутри цепочки — `option` — это тот же тип, который имеет мы получим в конце. Запомните этот момент — мы вернёмся к нему позже.

> Anyway, this continual testing and branching is really ugly! Does turning it into a computation expression help?

В любом случае, все эти бесконечные проверки и ветвления выглядят поистине ужасно! Смогут ли вычислительные выражения избавить от них?

> Once more we define a new type (`MaybeBuilder`) and make an instance of the type (`maybe`).

Опять определим новый тип (`MaybeBuilder`) и создадим его экземпляр (`maybe`).

```fsharp
type MaybeBuilder() =

    member this.Bind(x, f) =
        match x with
        | None -> None
        | Some a -> f a

    member this.Return(x) =
        Some x

let maybe = new MaybeBuilder()
```

> I have called this one `MaybeBuilder` rather than `divideByBuilder` because the issue of dealing with option
> types this way, using a computation expression, is quite common, and `maybe` is the standard name for this thing.

Я дал название `MaybeBuilder` вместо `divideByBuilder`, потому что проблема с необязательным результатом, которую мы решаем с помощью вычислительного выражения, встречается довольно часто, и слово `maybe` — устояшееся рназвание
для этой штуки.

> So now that we have defined the `maybe` workflow, let's rewrite the original code to use it.

Теперь, когда мы определили шаги для `maybe`, давайте перепишем оригинальный код с их использованием.

```fsharp
let divideByWorkflow init x y z =
    maybe
        {
        let! a = init |> divideBy x
        let! b = a |> divideBy y
        let! c = b |> divideBy z
        return c
        }
```

> Much, much nicer. The `maybe` expression has completely hidden the branching logic!

Выглдяти гораздо приятнее! Выражение `maybe` полностью скрыло ветвления!

> And if we test it we get the same result as before:

И, если мы протестируем код, мы получим тот же результат, что и раньше:

```fsharp
let good = divideByWorkflow 12 3 2 1
let bad = divideByWorkflow 12 3 0 1
```


> ### Chains of "or else" tests

### Цепочка проверок в "ветках else"

> In the previous example of "divide by", we only wanted to continue if each step was successful.

В предыдущем примере с делением, нам достаточно было продолжать вычисления, если очередной шаг был удачным.

> But sometimes it is the other way around. Sometimes the flow of control depends on a series of "or else" tests.
> Try one thing, and if that succeeds, you're done. Otherwise try another thing, and if that fails, try a third
> thing, and so on.

Но иногда нам бывает нужно что-то другое. Иногда поток управления зависит от последовательности проверок в "ветках else".
Проверьте первое условие и, если оно истинно, вы закончили. В противном случае проверьте второе, а если и оно ложно, проверьте третье, и так далее.

> Let's look at a simple example. Say that we have three dictionaries and we want to find the value corresponding
> to a key. Each lookup might succeed or fail, so we need to chain the lookups in a series.

Давайте взглянем на простой пример. Скажем, у нас есть три словаря и мы хотим найти значение по ключу. Каждая проверка может закончиться успехом или неудачей, так что нам надо объединить проверки в цепочку.

```fsharp
let map1 = [ ("1","One"); ("2","Two") ] |> Map.ofList
let map2 = [ ("A","Alice"); ("B","Bob") ] |> Map.ofList
let map3 = [ ("CA","California"); ("NY","New York") ] |> Map.ofList

let multiLookup key =
    match map1.TryFind key with
    | Some result1 -> Some result1   // success
    | None ->   // failure
        match map2.TryFind key with
        | Some result2 -> Some result2 // success
        | None ->   // failure
            match map3.TryFind key with
            | Some result3 -> Some result3  // success
            | None -> None // failure
```

> Because everything is an expression in F# we can't do an early return, we have to cascade all the tests in
> a single expression.

Поскольку в F# всё является выражением, мы не можем прервать вычисления в произвольном месте, нам придётся уложить все проверки в одно большое выражение.

> Here's how this might be used:

Теперь посмотрим, как это можно использовать:

```fsharp
multiLookup "A" |> printfn "Result for A is %A"
multiLookup "CA" |> printfn "Result for CA is %A"
multiLookup "X" |> printfn "Result for X is %A"
```

> It works fine, but can it be simplified?

Работает прекрасно, но можно ли упростить наш код?

> Yes indeed. Here is an "or else" builder that allows us to simplify these kinds of lookups:

Да, определённо. Вот строитель для "веток else", который позволяет упростить такого рода проверки:

```fsharp
type OrElseBuilder() =
    member this.ReturnFrom(x) = x
    member this.Combine (a,b) =
        match a with
        | Some _ -> a  // a succeeds -- use it
        | None -> b    // a fails -- use b instead
    member this.Delay(f) = f()

let orElse = new OrElseBuilder()
```

> Here's how the lookup code could be altered to use it:

А вот так можно переписать код проверок с использованием строителя.

```fsharp
let map1 = [ ("1","One"); ("2","Two") ] |> Map.ofList
let map2 = [ ("A","Alice"); ("B","Bob") ] |> Map.ofList
let map3 = [ ("CA","California"); ("NY","New York") ] |> Map.ofList

let multiLookup key = orElse {
    return! map1.TryFind key
    return! map2.TryFind key
    return! map3.TryFind key
    }
```

> Again we can confirm that the code works as expected.

И снова убедимся, что код работает, как ожидается.

```fsharp
multiLookup "A" |> printfn "Result for A is %A"
multiLookup "CA" |> printfn "Result for CA is %A"
multiLookup "X" |> printfn "Result for X is %A"
```

> ### Asynchronous calls with callbacks

### Асинхронные вызовы с функциями обратного вызова

> Finally, let's look at callbacks.  The standard approach for doing asynchronous operations in .NET is to use a
> [AsyncCallback delegate](http://msdn.microsoft.com/en-us/library/ms228972.aspx) which gets called when the async
> operation is complete.

И в завершение давайте взглянем на фукнции обратного вызова. Стандартным способом выполнения асинхронных вызовов в .NET является использование [делегата AsyncCallback](http://msdn.microsoft.com/en-us/library/ms228972.aspx), который вызывается, когда завершается асинхронная операциия.

> Here is an example of how a web page might be downloaded using this technique:

Вот пример, как можно скачать вебстраницу, используя эту технику:

```fsharp
open System.Net
let req1 = HttpWebRequest.Create("http://fsharp.org")
let req2 = HttpWebRequest.Create("http://google.com")
let req3 = HttpWebRequest.Create("http://bing.com")

req1.BeginGetResponse((fun r1 ->
    use resp1 = req1.EndGetResponse(r1)
    printfn "Downloaded %O" resp1.ResponseUri

    req2.BeginGetResponse((fun r2 ->
        use resp2 = req2.EndGetResponse(r2)
        printfn "Downloaded %O" resp2.ResponseUri

        req3.BeginGetResponse((fun r3 ->
            use resp3 = req3.EndGetResponse(r3)
            printfn "Downloaded %O" resp3.ResponseUri

            ),null) |> ignore
        ),null) |> ignore
    ),null) |> ignore
```

> Lots of calls to `BeginGetResponse` and `EndGetResponse`, and the use of nested lambdas, makes this quite
> complicated to understand. The important code (in this case, just print statements) is obscured by the callback
> logic.

Множество вызовов `BeginGetResponse` и `EndGetResponse`, и вложенные лямбда-функции достаточно трудны для понимания. Важный код (в нашем случае это операторы печати) теряется на фоне логики с обратными вызовами.

> In fact, managing this cascading approach is always a problem in code that requires a chain of callbacks; it has even been called the ["Pyramid of Doom"](http://raynos.github.com/presentation/shower/controlflow.htm?full#PyramidOfDoom) (although [none of the solutions are very elegant](https://web.archive.org/web/20170609232359/http://adamghill.com/callbacks-considered-a-smell/), IMO).

На самом деле, большая вложенность асинхронного кода — известная проблема, у неё даже есть собственное название — ["Пирамида Судьбы"](http://raynos.github.com/presentation/shower/controlflow.htm?full#PyramidOfDoom) (хотя, на мой взгляд, [ни одно из предложенных решений не выглядит достаточно элегантным](https://web.archive.org/web/20170609232359/http://adamghill.com/callbacks-considered-a-smell/)).

> Of course, we would never write that kind of code in F#, because F# has the `async` computation expression built
> in, which both simplifies the logic and flattens the code.

Естественно, нам никогда не придётся писать подобный код на F#, потому что в F# есть встронное вычислительное выражение `async`, которое и упрощает логику и избавляет код от вложенности (делает его плоским).

```fsharp
open System.Net
let req1 = HttpWebRequest.Create("http://fsharp.org")
let req2 = HttpWebRequest.Create("http://google.com")
let req3 = HttpWebRequest.Create("http://bing.com")

async {
    use! resp1 = req1.AsyncGetResponse()
    printfn "Downloaded %O" resp1.ResponseUri

    use! resp2 = req2.AsyncGetResponse()
    printfn "Downloaded %O" resp2.ResponseUri

    use! resp3 = req3.AsyncGetResponse()
    printfn "Downloaded %O" resp3.ResponseUri

    } |> Async.RunSynchronously
```

> We'll see exactly how the `async` workflow is implemented later in this series.

Позже в этом цикле статей мы разберёмся, как устроен процесс `async`.

> ## Summary ##

## Заключение

> So we've seen some very simple examples of computation expressions, both "before" and "after",
> and they are quite representative of the kinds of problems that computation expressions are useful for.

Итак, мы познакомились с несколькими простейшими примерами вычислительных выражений, как "до", так и "после". Примеры неплохо показывают, для решения каких проблем подходят вычислительные выражения.

> * In the logging example, we wanted to perform some side-effect between each step.
> * In the safe division example, we wanted to handle errors elegantly so that we could focus on the happy path.
> * In the multiple dictionary lookup example, we wanted to return early with the first success.
> * And finally, in the async example, we wanted to hide the use of callbacks and avoid the "pyramid of doom".

* В примере с логгированием, нам нужны были побочные эффекты, сопровождавшие каждый шаг вычислений.
* В примере с делением, нам потребовалась элегантная обработка ошибка, чтобы мы сосредоточились на успешном сценарии.
* В примере со словарями мы хотели прервать большое вычисление, как только получали результат.
* И, наконец, в примере с асинхронным кодом, мы хотели спрятать функции обратного вызова и изабавиться от "пирамиды судьбы".

> What all the cases have in common is that the computation expression is "doing something behind the scenes"
> between each expression.

Общим между всеми этими примерами является то, что вычислительные выражения выполняют какую-то работу за сценой, между отдельными шагами.

> If you want a bad analogy, you can think of a computation expression as somewhat like a post-commit hook for SVN
> or git, or a database trigger that gets called on every update.
> And really, that's all that a computation expression is: something that allows you to sneak your own code in to
> be called *in the background*, which in turn allows you to focus on the important code in the foreground.

Если вам нужна плохая аналогия, думайте о вычислительных выражениях, как скриптах, которые выполняются после коммитов в SVN или git: или как о триггерах, которые вызываются после каждого обновления базы данных. В действительности, это именно то, чем вычислительные выражения и являются: конструкцией, которая позволяют вам незаметно выполнять связующий код на заднем плане, чтобы вы могли сфокусироваться на первом плане, где выполняется важный код.

> Why are they called "computation expressions"? Well, it's obviously some kind of expression, so that bit is
> obvious. I believe that the F# team did originally want to call it
> "expression-that-does-something-in-the-background-between-each-let" but for some reason, people thought that was
> a bit unwieldy, so they settled on the shorter name "computation expression" instead.

Почему они называются "вычислительные выражения"? Что же, очевидно, что речь идёт об особом виде выражений, так что по крайней мере с одним словом всё ясно. Я верю, что команда разработчиков F# первоначально хотела дать им название "выражения-которые-делают-что-то-на-заднем-плане-между-каждым-вычислением", но потом они почему-то подумали, что это слишком громоздко и решили вместо этого дать им имя "вычислительные выражения".

> And as to the difference between a "computation expression" and a "workflow", I use *"computation expression"* to
> mean the `{...}` and `let!` syntax, and reserve *"workflow"* for particular implementations where appropriate.
> Not all computation expression implementations are workflows. For example, it is appropriate to talk about the
> "async workflow" or the "maybe workflow", but the "seq workflow" doesn't sound right.

Немного о разнице между "вычислительным выражением" (computation expression) и "процессом" (workflow). Когда я говорю "вычислительное выражение", я подразумеваю синтаксис из `{...}` и `let!`. "Процесс" относится к конкретным реализациям, когда мы будем их обсуждать.
Не все реализации вычислительных выражений являются процессами. Например, можно говорить о "процессе `async`" или "процессе `maybe`", но "процесс `seq`" звучит неправильно.

> In other words, in the following code, I would say that `maybe` is the workflow we are using, and the particular
> chunk of code `{ let! a = .... return c }` is the computation expression.

В примере ниже я бы сказал, что `maybe` — это процесс, который мы используем, а код в фигурных скобках `{ let! a = .... return c }` — вычислительное выражение.

```fsharp
maybe
    {
    let! a = x |> divideBy y
    let! b = a |> divideBy w
    let! c = b |> divideBy z
    return c
    }
```

> You probably want to start creating your own computation expressions now, but first we need to take a short
> detour into continuations. That's up next.


Сейчас вы, возможно, хотите написать своё первое вычислительное выражение, но сначала нам нужно разобраться с продолжениями (continuations). Это тема следующей статьи. 

> *Update on 2015-01-11: I have removed the counting example that used a "state" computation expression. It was too
> confusing and distracted from the main concepts.*

<!-- Думаю, это можно не переводить. -->