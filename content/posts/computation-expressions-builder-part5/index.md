---
layout: post
title: "Реализуя вычислительные выражения: Добавляем ленивость"
# title: "Implementing a CE: Adding laziness"
# description: "Delaying a workflow externally"
description: "Откладываем вычисления снаружи выражения"
date: 2013-01-29
nav: thinking-functionally
seriesId: "Вычислительные выражения"
seriesOrder: 10
---

> In a [previous post](/posts/computation-expressions-builder-part3/), we saw how to avoid unnecessary evaluation of expressions in a workflow until needed.

В одном из прошлых постов мы разобрались, как избежать вычисления ненужных выражений, пока их значения действительно не понадобятся.

> But that approach was designed for expressions *inside* a workflow.
> What happens if we want to delay the *whole workflow itself* until needed.

Но эта возможность была разработана для выражений *внутри* процесса вычисления.
Что если мы хотим отложить вычисление *всего процесса целиком*, пока не понадобится его значение?

> {{<alertinfo>}}
> Note that the "builder" in the context of a computation expression is not the same as the OO "builder pattern" for constructing and validating objects.
> There is a post on the ["builder pattern" here](../builder-pattern).
> {{</alertinfo>}}

{{<alertinfo>}}
Обратите внимание, что "построитель" в контексте вычислительного выражения — это не то же самое, что и объектно-ориентированный "паттерн строитель", нужный для конструирования и валидации объектов.
{{</alertinfo>}}


> ## The problem

## Проблема

> Here is the code from our "maybe" builder class.
> This code is based on the `trace` builder from the earlier post, but with all the tracing taken out, so that it is nice and clean.

Вот код из нашего класса-построителя "maybe".
Этот код основан на построителе `trace` из ранних постов, но без трассирующих сообщений, так что он сейчас не содержит лишнего кода и легко читается.

```fsharp
type MaybeBuilder() =

    member this.Bind(m, f) =
        Option.bind f m

    member this.Return(x) =
        Some x

    member this.ReturnFrom(x) =
        x

    member this.Zero() =
        None

    member this.Combine (a,b) =
        match a with
        | Some _ -> a  // если значение a существует, пропускаем b
        | None -> b()  // если значение a не существует, запускаем b

    member this.Delay(f) =
        f

    member this.Run(f) =
        f()

// создаём экземпляр процесса
let maybe = new MaybeBuilder()
```

> Before moving on, make sure that you understand how this works.
> If we analyze this using the terminology of the earlier post, we can see that the types used are:

Перед тем, как двигаться дальше, давайте убедимся, что вы понимаете, как работает этот код.
Если мы проанализируем его, используя терминологию предыдущих постов, мы увидим, что здесь используются такие типы:

> * Wrapper type: `'a option`
> * Internal type: `'a option`
> * Delayed type: `unit -> 'a option`

* Тип-обёртка: `'a option`
* Внутренний тип: `'a option`
* Тип отложенного вычисления: `unit -> 'a option`

> Now let's check this code and make sure everything works as expected.

Теперь давайте проверим этот код и убедимся, что всё работает как надо.

```fsharp
maybe {
    printfn "Часть 1: перед return 1"
    return 1
    printfn "Часть 2: после return"
    } |> printfn "Результат Части 1, но не Части 2: %A"

// результат - вторая часть НЕ ВЫПОЛНЯЕТСЯ

maybe {
    printfn "Часть 1: перед return 1"
    return! None
    printfn "Часть 2: после None, продолжаем работать"
    } |> printfn "Результат Части 1, а затем Части 2: %A"

// результат - вторая часть ВЫПОЛНЯЕТСЯ
```

> But what happens if we refactor the code into a child workflow, like this:

Но что случится, если мы поместим этот код в дочерний процесс, как здесь:

```fsharp
let childWorkflow =
    maybe {printfn "Дочерний процесс"}

maybe {
    printfn "Часть 1: перед return 1"
    return 1
    return! childWorkflow
    } |> printfn "Результат Части 1, но не childWorkflow: %A"
```

> The output shows that the child workflow was evaluated even though it wasn't needed in the end.
> This might not be a problem in this case, but in many cases, we may not want this to happen.

Вывод показывает, что дочерний процесс был запущен даже если его результат не был нужен в конце.
Это может не быть проблемой в данном случае, но во многих других случаях мы можем не хотеть, чтобы это происходило.

> So, how to avoid it?

Итак, как этого избежать?

> ## Wrapping the inner type in a delay

## Оборачиваем внутренний тип в Delay

> The obvious approach is to wrap the *entire result of the builder* in a delay function, and then to "run" the result, we just evaluate the delay function.

Очевидный подход — обернуть *результат Построителя целиком* в отложенную функцию, и впоследствии "запускать" результат, мы просто выполняем отложенную функцию.

> So, here's our new wrapper type:

Вот здесь наш новый тип-обёртка:

```fsharp
type Maybe<'a> = Maybe of (unit -> 'a option)
```

> We've replaced a simple `option` with a function that evaluates to an option, and then wrapped that function in a [single case union](/posts/designing-with-types-single-case-dus/) for good measure.

Мы заменили простой тип `option` функцией, которая вычисляет `option`, и затем для наглядности завернули эту функцию в [одновариантное объединение](/posts/designing-with-types-single-case-dus/).

> And now we need to change the `Run` method as well.
> Previously, it evaluated the delay function that was passed in to it, but now it should leave it unevaluated and wrap it in our new wrapper type:

Теперь нам нужно заменить и метод `Run`.
Раньше он выполнял отложенную функцию, которую мы ему передавали, но сейчас он должен оставить её невыполненной и завернуть её в наш новый тип-обёртку:

```fsharp
// до
member this.Run(f) =
    f()

// после
member this.Run(f) =
    Maybe f
```

> *I've forgotten to fix up another method -- do you know which one? We'll bump into it soon!*

*Я забыл поправить ещё один метод — вы догадались, какой? Скоро мы это узнаем!*

> One more thing -- we'll need a way to "run" the result now.

Ещё одна вещь — теперь нам понадобится способ, чтобы "запустить" результат.

```fsharp
let run (Maybe f) = f()
```

> Let's try out our new type on our previous examples:

Давайте испытаем наш новый тип на наших предыдущих примерах:

```fsharp
let m1 = maybe {
    printfn "Часть 1: перед return 1"
    return 1
    printfn "Часть 2: после return 1"
    }
```

> Running this, we get something like this:

Запуская это, мы получаем что-то такое:

```fsharp
val m1 : Maybe<int> = Maybe <fun:m1@123-7>
```

> That looks good; nothing else was printed.

Выглядит нормально; не печатает ничего ненужного.

> And now run it:

И теперь запускаем:

```fsharp
run m1 |> printfn "Результат Части 1, но не Части 2: %A"
```

> and we get the output:

и получаем вывод:

> ```text
> Part 1: about to return 1
> Result for Part1 but not Part2: Some 1
> ```

```text
Часть 1: до return 1
Результат Части 1, но не Части 2: Some 1
```

> Perfect. Part 2 did not run.

Великолепно. Часть 2 не запускалась.

> But we run into a problem with the next example:

Но мы сталкиваемся с проблемой в следующем примере:

```fsharp
let m2 = maybe {
    printfn "Часть 1: перед return None"
    return! None
    printfn "Часть 2: после None, продолжаем выполнение"
    }
```

> Oops!
> We forgot to fix up `ReturnFrom`!
> As we know, that method takes a *wrapped type*, and we have redefined the wrapped type now.

Ой!
Мы забыли поправить `ReturnFrom`!
Как мы знаем, этот метод получает *тип-обёртку*, и сейчас мы должны переопределить этот тип-обёртку.

> Here's the fix:

Вот правка:

```fsharp
member this.ReturnFrom(Maybe f) =
    f()
```

> We are going to accept a `Maybe` from outside, and then immediately run it to get at the option.

Мы собираемся принять `Maybe` снаружи и сразу после этого запустить его, чтобы получить опциональное значение.

> But now we have another problem -- we can't return an explicit `None` anymore in `return! None`, we have to return a `Maybe` type instead.
> How are we going to create one of these?

Но сейчас у нас появилась другая проблема — мы больше не можем вернуть `None` в явном виде в конструкции `return! None`, вместо этого мы должны вернить тип `Maybe`.
Получится ли у нас сделать что-то подобное?

> Well, we could create a helper function that constructs one for us.
> But there is a much simpler answer: you can create a new `Maybe` type by using a `maybe` expression!

Скажем, мы могли бы написать вспомогательную функцию, которая конструирует для нас нужны объект.
Но есть и более простой ответ: мы можем создать новый объект `Maybe` используя выражение `maybe`!

```fsharp
let m2 = maybe {
    return! maybe {printfn "Часть 1: перед return None"}
    printfn "Часть 2: после None, продолжаем"
    }
```

> This is why the `Zero` method is useful.
> With `Zero` and the builder instance, you can create new instances of the type even if they don't do anything.

Вот для чего нужен метод `Zero`.
С помощью `Zero` и экземпляра построителя можно создавать экземпляры типа-обёркти, даже если они ничего не делают.

> But now we have one more error -- the dreaded "value restriction":

Но теперь у нас появилась ещё одна ошибка — страшное "ограничение значения".

> ```text
> Value restriction. The value 'm2' has been inferred to have generic type
> ```

```text
Ограничение значения. Значения 'm2' было выведено, как имеющее общий тип.
```

> The reason why this has happened is that *both* expressions are returning `None`.
> But the compiler does not know what type `None` is.
> The code is using `None` of type `Option<obj>` (presumably because of implicit boxing) yet the compiler knows that the type can be more generic than that.

Причина, по которой это случилось заключается в том, что *оба* выражения возвращают `None`.
Но компилятор не знает, к какому типу относится `None`.
Код использует `None` типа `Option<obj>` (предоположительно из-за неявной упаковки), однако, компилятор знает, что тип может быть более общим.

> There are two fixes.
> One is to make the type explicit:

Есть два способа избавиться от ошибки.
Первый — сделать тип явным:

```fsharp
let m2_int: Maybe<int> = maybe {
    return! maybe {printfn "Часть 1: перед return None"}
    printfn "Часть 2: после None, продолжаем;"
    }
```

> Or we can just return some non-None value instead:

Или вместо `None` мы можем вернёть какое-то значение:

```fsharp
let m2 = maybe {
    return! maybe {printfn "Часть 1: перед return None"}
    printfn "Часть 2: после None, продолжаем;"
    return 1
    }
```

> Both of these solutions will fix the problem.

Оба решения позволяют избавиться от проблемы.

> Now if we run the example, we see that the result is as expected.
> The second part *is* run this time.

Теперь, если мы запустим пример, мы получим тот результат, который и ожидали.
Вторая часть продолжает выполняться.

```fsharp
run m2 |> printfn "Результат Части 1 и последующей Части 2: %A"
```

> The trace output:

```text
Часть 1: перед return None
Часть 2: после None, продолжаем;
Результат Части 1 и последующей Части  2: Some 1
```

> Finally, we'll try the child workflow examples again:

Наконец, снова попытаемся запустить примеры с дочерними процессами:

```fsharp
let childWorkflow =
    maybe {printfn "Дочерний процесс"}

let m3 = maybe {
    printfn "Часть 1: перед return 1"
    return 1
    return! childWorkflow
    }

run m3 |> printfn "Результат Части 1, но без childWorkflow: %A"
```

> And now the child workflow is not evaluated, just as we wanted.

И теперь дочерний процесс не выполняется, как мы и хотели.

> And if we *do* need the child workflow to be evaluated, this works too:

И если нам *надо*, чтобы дочерний процесс выполнился, сработает такой код:

```fsharp
let m4 = maybe {
    return! maybe {printfn "Часть 1: перед return None"}
    return! childWorkflow
    }

run m4 |> printfn "Результат Части 1 и последующего дочернего процесса: %A"
```

> ### Reviewing the builder class

### Обзор класса-построителя

> Let's look at all the code in the new builder class again:

Давайте снова взглянем на код класса-построителя целиком:

```fsharp
type Maybe<'a> = Maybe of (unit -> 'a option)

type MaybeBuilder() =

    member this.Bind(m, f) =
        Option.bind f m

    member this.Return(x) =
        Some x

    member this.ReturnFrom(Maybe f) =
        f()

    member this.Zero() =
        None

    member this.Combine (a,b) =
        match a with
        | Some _' -> a    // Если есть значение a, опускаем значение b
        | None -> b()     // Если нет значения a, запускаем b

    member this.Delay(f) =
        f

    member this.Run(f) =
        Maybe f

// создаём экземпляр процесса
let maybe = new MaybeBuilder()

let run (Maybe f) = f()
```

> If we analyze this new builder using the terminology of the earlier post, we can see that the types used are:

Если мы проанализируем новый построитель в терминах, которые мы использовали в предыдущем посте, от обнаружим, что:

> * Wrapper type: `Maybe<'a>`
> * Internal type: `'a option`
> * Delayed type: `unit -> 'a option`

* Тип-обёртка — это `Maybe<'a>`
* Внутренний тип — это `'a option`
* Тип отложенного вычисления — это `unit -> 'a option`

> Note that in this case it was convenient to use the standard `'a option` as the internal type, because we didn't need to modify `Bind` or `Return` at all.

Заметьте, что в данном случае удобно использовать в качестве внутреннего типа стандартный `'a option`, посольку в этом случае нам не нужно модифицировать ни `Bind`, ни `Return`.

> An alternative design might use `Maybe<'a>` as the internal type as well, which would make things more consistent, but makes the code harder to read.

В качестве альтернативного дизайна можно использовать в качестве внутреннго типа `Maybe<'a>`, что сделает код более согласанным, но в то же время большее сложным для понимания.

> ## True laziness

## Истинная ленивость

> Let's look at a variant of the last example:

Давайте взглянем на немного исправленный последний пример:

```fsharp
let child_twice: Maybe<unit> = maybe {
    let workflow = maybe {printfn "Дочерний процесс"}

    return! maybe {printfn "Часть 1: перед return None"}
    return! workflow
    return! workflow
    }

run child_twice |> printfn "Результат двойного childWorkflow: %A"
```

> What should happen?
> How many times should the child workflow be run?

Что должно произойти?
Сколько раз выполниться дочерний процесс?

> The delayed implementation above does ensure that the child workflow is only be evaluated on demand, but it does not stop it being run twice.

Отложенная реализация, представленная выше, гарантирует, что дочерний процесс будет выполнено только в случае необходимости, но это не значит, что он не будет выполнен дважды.

> In some situations, you might require that the workflow is guaranteed to only run *at most once*, and then cached ("memoized").
> This is easy enough to do using the `Lazy` type that is built into F#.

В некоторых случаях вам важно, чтобы процесс гарантированно запускался *по меньшей мере один раз* с последующим кешированием результата (мемоизацией).
Это достаточно просто сделать, используя тип `Lazy`, который встроен в F#.

> The changes we need to make are:

Изменения, которые нам нужно будет сделать:

> * Change `Maybe` to wrap a `Lazy` instead of a delay
> * Change `ReturnFrom` and `run` to force the evaluation of the lazy value
> * Change `Run` to run the delay from inside a `lazy`

* `Maybe` будет оборачивать `Lazy` вместо отложенной функции
* `ReturnFrom` и `run` будут запускать отложенное вычисление
* `Run` будет запускать отложенное вычисление из `Lazy`

> Here is the new class with the changes:

Вот новый класс с указанными изменениями:

```fsharp
type Maybe<'a> = Maybe of Lazy<'a option>

type MaybeBuilder() =

    member this.Bind(m, f) =
        Option.bind f m

    member this.Return(x) =
        Some x

    member this.ReturnFrom(Maybe f) =
        f.Force()

    member this.Zero() =
        None

    member this.Combine (a,b) =
        match a with
        | Some _' -> a    // Если есть значение a, опускаем значение b
        | None -> b()     // Если нет значения a, запускаем b

    member this.Delay(f) =
        f

    member this.Run(f) =
        Maybe (lazy f())

// создаём экземпляр процесса
let maybe = new MaybeBuilder()

let run (Maybe f) = f.Force()
```

> And if we run the "child twice` code from above, we get:

И, если мы запустим код с двойным дочерним процессом, мы получим:

> ```text
> Part 1: about to return None
> Child workflow
> Result for childWorkflow twice: <null>
> ```

```text
Часть 1: перед return None
Дочерний процесс
Результат двойного childWorkflow: <null>
```

> from which it is clear that the child workflow only ran once.

откуда понятно, что дочерний процесс был запущен только один раз.

> ## Summary: Immediate vs. Delayed vs. Lazy

## Итого: немедленные, отложенные и ленивые вычисления

> On this page, we've seen three different implementations of the `maybe` workflow.
> One that is always evaluated immediately, one that uses a delay function, and one that uses laziness with memoization.

В этой статье мы познакомились с тремя различными реализациями процесса `maybe`.
Одна выполнялась сразу, одна использовала отложенную функцию, и ещё одна — ленивость с мемоизацией.

> So... which approach should you use?

Так... какой же подход нам использовать?

> There is no single "right" answer.
> Your choice depends on a number of things:

На этот вопрос нет единственного "правильного ответа".
Ваш выбор зависит от нескольких вещей:

> * *Is the code in the expression cheap to execute, and without important side-effects?*
>   If so, stick with the first, immediate version.
>   It's simple and easy to understand, and this is exactly what most implementations of the `maybe` workflow use.
> * *Is the code in the expression expensive to execute, might the result vary with each call (e.g. non-deterministic), or are there important side-effects?*
>   If so, use the second, delayed version.
>   This is exactly what most other workflows do, especially those relating to I/O (such as `async`).
> * F# does not attempt to be a purely functional language, so almost all F# code will fall into one of these two categories.
>   But, *if you need to code in a guaranteed side-effect free style, or you just want to ensure that expensive code is evaluated at most once*, then use the third, lazy option.

* *Является ли код недорогим с точки зрения ресурсов, и не имеет ли он существенных побочных эффектов?*
* Если да, берите первую — немедленную — версию.
* Это просто и легко понять, и это именно то, что предоставляют большинство реализаций процесса `maybe`.
* *Является ли код дорогим с точки зрения ресурсов, может ли результат отличаться при последющих запусках (т.е. является код не-детерминированым), или содержит существенные побочные эффекты?*
* Если да, используйте вторую — отложенную — версию.
* Это именно то, что деалет большинство процессов, особенно, относящихся к вводу и выводу (таких как `async`).
* F# не пытается быть чистым функциональным языком, так что почти весь код на F# попадёт в одну из этих двух категорий.
* Но, *если вым нужен код, в котором гарантированно нет побочных эффектов или вы хотите быть уверены, что дорогой с точки зрения ресурсов код выполняется не больше одного раза", тогда используйте третий — ленивый — вариант.

> Whatever your choice, do make it clear in the documentation.
> For example, the delayed vs. lazy implementations appear exactly the same to the client, but they have very different semantics, and the  client code must be written differently for each case.

Чтобы вы ни выбрали, ясно напишите про свой выбор в документации.
Например, отложенная и ленивая реализации с точки зрения клиента выглядтят одинаково, но имеют совершенно разную сематнику, и клиентских код должен быть написан по разному для разных случаев.

> Now that we have finished with delays and laziness, we can go back to the builder methods and finish them off.

Что ж, мы заокнчили с отложенными и ленивыми вычислениями, так что вернёмся к методам построителя и ппокончим с ними.