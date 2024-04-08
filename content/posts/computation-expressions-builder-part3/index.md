---
layout: post
# title: "Implementing a CE: Delay and Run"
title: "Реализуя вычислительные выражения: Delay и Run"
# description: "Controlling when functions execute"
description: "Управляем временем выполнения функции"
date: 2013-01-27
nav: thinking-functionally
# seriesId: "Computation Expressions"
seriesId: "Вычислительные выражения"
seriesOrder: 8
---

> In the last few posts we have covered all the basic methods (Bind, Return, Zero, and Combine) needed to create your own computation expression builder.
> In this post, we'll look at some of the extra features needed to make the workflow more efficient, by controlling when expressions get evaluated.

В нескольких последних постах мы покрыли <!-- обсудили --> все базовые методы (Bind, Return, Zero и Combine), нужные для создания вашего собственного построителя вычислительного выражения.

> {{<alertinfo>}}
> Note that the "builder" in the context of a computation expression is not the same as the OO "builder pattern" for constructing and validating objects.
> There is a post on the ["builder pattern" here](../builder-pattern).
> {{</alertinfo>}}

{{<alertinfo>}}
Обратите внимание, что "строитель" или "построитель" в контексте вычислительного выражения --- это не то же самое, что и объектно-ориентированный "паттерн строитель" , нужный для конструирования и валидации объектов.
Пост по "паттерне строитель" доступен [здесь](../builder-pattern).
{{</alertinfo>}}

> ## The problem: avoiding unnecessary evaluations

## Проблема: избавляемся от ненужных вычислений

> Let's say that we have created a "maybe" style workflow as before.
> But this time we want to use the "return" keyword to return early and stop any more processing being done.

Давайте предстаим, что мы создали процесс в стиле "maybe", как раньше.
Но сейчас мы хотим использовать ключевое слово "return" для раннего выхода и остановки любой оставшейся обработки.

> Here is our complete builder class.
> The key method to look at is `Combine`, in which we simply ignore any secondary expressions after the first return.

Здесь наш класс-построитель полностью.
Ключевой метод, на который нужно обратить внимание, это `Combine`, где мы просто игнорируем второе выражение, после того, как возвращаем первое.

```fsharp
type TraceBuilder() =
    member this.Bind(m, f) =
        match m with
        | None ->
            printfn "Bind с None. Выход."
        | Some a ->
            printfn "Bind с Some(%A). Продолжение" a
        Option.bind f m

    member this.Return(x) =
        printfn "Return незавёрнутого %A как Option" x
        Some x

    member this.Zero() =
        printfn "Zero"
        None

    member this.Combine (a,b) =
        printfn "Combine. Сразу возвращаем %A. Игнорируем  %A" a b
        a

    member this.Delay(f) =
        printfn "Delay"
        f()

// создаём экземпляр процесса
let trace = new TraceBuilder()
```

> Let's see how it works by printing something, returning, and then printing something else:

Давайте взглянем как это работает, напечатав что-то одно, вызвав "return", и затем напечатав что-то другое:

```fsharp
trace {
    printfn "Часть 1: до return 1"
    return 1
    printfn "Часть 2: после return"
    } |> printfn "Результат Части 1 без Части 2: %A"
```

> The debugging output should look something like the following, which I have annotated:

Отладочный вывод (который я прокомментировал) должен выглядеть приблизительно так, как ниже:

```text
// первое выражение, перед "return"
Delay
Часть 1: до 1
Return незавёрнутого 1 как Option

// второе выражение, перед закрывающей фигурной скобкой.
Delay
Часть 2: после return
Zero   // здесь zero, потому что нет явного return для этой части

// комбинируем два выражения
Combine. Сразу возвращаем Some 1. Игнорируем <null>

// финальный результат
Результат Части 1 без Части 2: Some 1
```

> We can see a problem here.
> The "Part 2: after return" was printed, even though we were trying to return early.

Здесь мы можем увидеть проблему.
Строка "Часть 2: после return" была напечатана, не смотря на то, что мы пытались сделать ранний возврат. 

> Why?
> Well I'll repeat what I said in the last post: **return and yield do *not* generate an early return from a computation expression**.
> The entire computation expression, all the way to the last curly brace, is *always* evaluated and results in a single value.

Почему?
Хорошо, я повторю то, что говорил в предыдущем посте: **return и yield *не* генерируют код раннего возврата из вычислительного выражения**.
Вычислительное выражение целиком, всё, что до закрывающей фигурной скобки, *всегда* вычисляется и возвращает одно значение.

> This is a problem, because you might get unwanted side effects (such as printing a message in this case) and your code is doing something unnecessary, which might cause performance problems.

Это является проблемой, потому что вам могут быть не нужны побочные эффекты (такие, как печать сообщений в нашем случае) и ваш код делает что-то ненужное, что может привести к проблемам производительности.

> So, how can we avoid evaluating the second part until we need it?

Так как нам избежать вычисления второй части до тех пор, пока она действительно нам не понадобиться.

> ## Introducing "Delay"

## Введение в "Delay"

> The answer to the question is straightforward -- simply wrap part 2 of the expression in a function and only call this function when needed, like this.

Ответ на этот вопрос достаточно прост --- просто оберните вторую часть выражения в функцию и вызывайте её только тогда, когда это нужно, как здесь:

```fsharp
let part2 =
    fun () ->
        printfn "Часть 2: после return"
        // выполняем какие-то действия
        // возвращаем Zero

// вычисляем, только если нужно
if needed then
   let result = part2()
```

> Using this technique, part 2 of the computation expression can be processed completely, but because the expression returns a function, nothing actually *happens* until the function is called.
> But the `Combine` method will never call it, and so the code inside it does not run at all.

Благодая этой технике, часть 2 вычислительного выражения может быть обработана полностью, но из-за того, что выражение возвращает функцию, в действительности ничего *не происходит*, пока функция не будет вызвана.
Но метод `Combine` никогда не вызовет её, так что код внутри неё вообще никогда не выполнится.

> And this is exactly what the `Delay` method is for.
> Any result from `Return` or `Yield` is immediately wrapped in a "delay" function like this, and then you can choose whether to run it or not.

И это в точности то, для чего нужен метод `Delay`.
Любой результат от `Return` или `Yield` немедленно оборачивается в "отложенную" функцию, как мы описаил, и затем вы можете выбрать, запускать её или не запускать.

> Let's change the builder to implement a delay:

Давайте изменим построитель, чтобы реализовать отложенные вычисления:

```fsharp
type TraceBuilder() =
    // другие члены как раньше

    member this.Delay(funcToDelay) =
        let delayed = fun () ->
            printfn "%A - Начало отложенной функции." funcToDelay
            let delayedResult = funcToDelay()
            printfn "%A - Конец отложенной функции. Результат %A" funcToDelay delayedResult
            delayedResult  // возвращаем результат

        printfn "%A - отложена благодаря %A" funcToDelay delayed
        delayed // возвращаем новую функцию
```

> As you can see, the `Delay` method is given a function to execute.
> Previously, we executed it immediately.
> What we're doing now is wrapping this function in another function and returning the delayed function instead.
> I have added a number of trace statements before and after the function is wrapped.

Как вы можете видеть, метод `Delay` получает функцию для выполнения.
Раньше мы вызывали её немедленно.
Что мы делаем сейчас --- оборачиваем эту функцию в другую функцию и возвращаем отложенную функцию вместо первой.
Я добавил несколько трассирующих операторов до и после обёрнутой функции.

> If you compile this code, you can see that the signature of `Delay` has changed.
> Before the change, it returned a concrete value (an option in this case), but now it returns a function.

Если вы компилируете этот код, вы можете видеть, что сигнатура `Delay` изменилась.
До изменения, она возвращала конкретное значение (в данном случае Option), но сейчас она возвращает функцию.

```fsharp
// сигнатура ДО изменения
member Delay : f:(unit -> 'a) -> 'a

// сигнатура ПОСЛЕ изменения
member Delay : f:(unit -> 'b) -> (unit -> 'b)
```

> By the way, we could have implemented `Delay` in a much simpler way, without any tracing, just by returning the same function that was passed in, like this:

Кстати, мы могли бы реализовать `Delay` гораздо проще, без всякой трассировки, просто возвращая ту же самую функцию, которую получили, как здесь:

```fsharp
member this.Delay(f) =
    f
```

> Much more concise!
> But in this case, I wanted to add some detailed tracing information as well.

Гораздо короче!
Но в этом случае я всё-таки захотел добавить немного трассирующей информации.

> Now let's try again:

Попробуем снова:

```fsharp
trace {
    printfn "Часть 1: перед return 1"
    return 1
    printfn "Часть 2: после return"
    } |> printfn "Результат Части 1 без Части 2: %A"
```

> Uh-oh.
> This time nothing happens at all!
> What went wrong?

Ой-ой.
Сейчас вообще ничего ен происходит!
Что пошло не так?

> If we look at the output we see this:

Если мы посмотрим на вывод, то увидим:

```text
Результат Части 1 без Части 2: <fun:Delay@84-5>
```

> Hmmm.
> The output of the whole `trace` expression is now a *function*, not an option.
> Why?
> Because we created all these delays, but we never "undelayed" them by actually calling the function!

Хммм.
Вывод всего выражения `trace` теперь *функция*, а не `Option`.
Потому что мы отложили все эти вычисления, но мы ни разу "не вернули их в работу", действительно вызвав функцию!

> One way to do this is to assign the output of the computation expression to a function value, say `f`, and then evaluate it.

Один способ сделать это --- сделать результатом вычислительного выражения функцию, скажем `f`, и в конце вызвать её.

```fsharp
let f = trace {
    printfn "Часть 1: до return 1"
    return 1
    printfn "Часть 2: после return"
    }
f() |> printfn "Результат Части 1 без Части 2: %A"
```

> This works as expected, but is there a way to do this from inside the computation expression itself?
> Of course there is!

Это работает как ожидается, но есть ли способ сделать это изнутри самого вычислительного выражения?
Конечно, есть!

> ## Introducing "Run"

## Введение в "Run"

> The `Run` method exists for exactly this reason.
> It is called as the final step in the process of evaluating a computation expression, and can be used to undo the delay.

Именно по этой причине существует метод `Run`.
Он вызывается на последнем шаге процесса вычисления вычислительного выражения и моежт быть использован, чтобы запустить отложенные вычисления.

> Here's an implementation:

Вот реализация:

```fsharp
type TraceBuilder() =
    // другие члены как и раньше

    member this.Run(funcToRun) =
        printfn "%A - Run Начало." funcToRun
        let runResult = funcToRun()
        printfn "%A - Run Конец. Результат %A" funcToRun runResult
        runResult // возвращаем результат выполнения отложенной функции
```

> Let's try one more time:

```fsharp
trace {
    printfn "Часть 1: до return 1"
    return 1
    printfn "Часть 2: после return"
    } |> printfn "Результат Части 1 без Части 2: %A"
```

> And the result is exactly what we wanted.
> The first part is evaluated, but the second part is not.
> And the result of the entire computation expression is an option, not a function.

И результат в точности такой, как мы хотели.
Первая часть выполняется, а вторая часть нет
И результат всего вычислительного выражения это `Option`, а не функция.

> ## When is delay called?

## Когда вызывается "Delay"?

> The way that `Delay` is inserted into the workflow is straightforward, once you understand it.

Способ, каким `Delay` вставляется в процесс простой, как только вы его поймёте.

> * The bottom (or innermost) expression is delayed.
> * If this is combined with a prior expression, the output of `Combine` is also delayed.
> * And so on, until the final delay is fed into `Run`.

* Нижнее (самое вложенное) выражение откладывается.
* Если оно комбинируется с предшествующим выражением, вывод `Combine` также откладывается.
* И так далее, пока финальное отложенное вычисление не отправляется в `Run`.

> Using this knowledge, let's review what happened in the example above:

Опираясь на это знание, давайте разберёмся, что случилось в примере выше.

> * The first part of the expression is the print statement plus `return 1`.
> * The second part of the expression is the print statement without an explicit return, which means that `Zero()` is called
> * The `None` from the `Zero` is fed into `Delay`, resulting in a "delayed option", that is, a function that will evaluate to an `option` when called.
> * The option from part 1 and the delayed option from part 2 are combined in `Combine` and the second one is discarded.
> * The result of the combine is turned into another "delayed option".
> * Finally, the delayed option is fed to `Run`, which evaluates it and returns a normal option.

* Первая часть выражения --- это оператор печати плюс `return 1`.
* Вторая часть выражения --- оператора печати без явного `return`, что означает, что вызван метод `Zero()`.
* Значение `None`, полученное из `Zero`, отправляется в `Delay`, возвращая "отложенное значение" `Option`, то есть функцию, которая возвращает `Option`, если её вызывать.
* Опциональное значение из части 1 и отложенное значение из части 2 комбинирутся с помощью `Combine` и второе значение отбрасывается.
* Результат комбинации превращается в другое "отложенное значение".
* В конце, отложенное выражение попадает в `Run`, которое выполяет его и возвращает обычное опцинальное значение.

> Here is a diagram that represents this process visually:

Вот диаграмма, которая представляет этот процесс визуально:

![Delay](./ce_delay.png)

> If we look at the debug trace for the example above, we can see in detail what happened.
> It's a little confusing, so I have annotated it.
> Also, it helps to remember that working *down* this trace is the same as working *up* from the bottom of the diagram above, because the outermost code is run first.

Если мы посмотрим на трассирующий вывод из примера выше, мы можем увидеть в деталях, что произошло.
Это может сбивать с толку, так что я добавил объяснения.
Также, полезно помнить, что работа *вниз* по этой трассировке аналогична работе *вверх* из нижней части диаграммы выше, потому что внешние код выпоняется первым. <!-- движение по трассировке сверху вниз, соответствует движению по диаграмме снизу вверх  -->

```text
// откладываем вычисление всего выражения (результат Combine)
<fun:Pipe #1 input at line 42@43> - отложена благодаря <fun:delayed@23>

// запускаем самое внешнее отложенное выражение (полученное из Combine)
<fun:delayed@23> - Run Начало.
<fun:Pipe #1 input at line 42@43> - Начало отложенной функции.

// результат первого выражения Some(1)
Часть 1: до return 1
Return незавёрнутого 1 как Option

// второе выражение заворачивается в функцию
<fun:Pipe #1 input at line 42@45-1> - отложена благодаря <fun:delayed@23>

// первое и второе выражения комбинируется
Return ранний Some 1. Игнорируем вторую часть: <fun:delayed@23>

// завершено оборачивание всего выражения (результата Combine)
<fun:Pipe #1 input at line 42@43> - Конец отложенной функции. Результат Some 1
<fun:delayed@23> - Run Конец. Результат Some 1

// сейчас резульат Option, а не функция
Результат Части 1 без Части 2: Some 1
```

> ## "Delay" changes the signature of "Combine"

## "Delay" изменяет сигнатуру "Combine"

> When `Delay` is introduced into the pipeline like this, it has an effect on the signature of `Combine`.

Когда `Delay` вводится в цепочку вызовов, как в этом примере, это оказывает эффект на сигнутуру `Combine`.

> When we originally wrote `Combine` we were expecting it to handle `options`.
> But now it is handling the output of `Delay`, which is a function.

Когда мы написали `Combine` в начале, вы ожидали, что он обрабатывает опциональный тип.
Но сейчас он обрабатывает вывод `Delay`, который является функцией.

> We can see this if we hard-code the types that `Combine` expects, with `int option` type annotations like this:

Мы увидим это, если явно пропишем типы, которые ожидает `Combine`, и укажем `int option`, как здесь:

```fsharp
member this.Combine (a: int option,b: int option) =
    printfn "Combine. Сразу возвращаем %A. Игнорируем  %A" a b
    a
```

> If this is done, we get an compiler error in the "return" expression:

Если мы сделаем это, мы получим ошибку при возврате выражения:

```fsharp
trace {
    printfn "Part 1: about to return 1"
    return 1
    printfn "Part 2: after return has happened"
    } |> printfn "Result for Part1 without Part2: %A"
```

> The error is:

Ошибка:

> ```text
> error FS0001: This expression was expected to have type
>     int option
> but here has type
>     unit -> 'a
> ```

```text
Error FS0193 : Несоответствие ограничений типов. Тип 
    "unit -> 'a option"    
несовместим с типом
    "int option"
```

> In other words, the `Combine` is being passed a delayed function (`unit -> 'a`), which doesn't match our explicit signature.

Другими словами, в `Combine` передаётся отложенная функция (`unit -> 'a`), которая не совпадает с нашей явноей сигнатурой.

> So what happens when we *do* want to combine the parameters, but they are passed in as a function instead of as a simple value?

Так что происходит, когда мы *хотим* скомбинировать параметры, но они передаются как функции вместо обычных значений?

> The answer is straightforward: just call the function that was passed in to get the underlying value.

Ответ простой: просто вызовите функцию, которая была передана, чтобы получить отложенное значение.

> Let's demonstrate that using the adding example from the previous post.

Давайте продемонстрируем это, используя пример со сложением из предыдущего поста.

```fsharp
type TraceBuilder() =
    // другие члены как разнье

    member this.Combine (m,f) =
        printfn "Combine. Перед получением отложенного параметра %A" f
        let y = f()
        printfn "Combine. После получения отложенного параметра %A. Результат %A" f y

        match m,y with
        | Some a, Some b ->
            printfn "комбинируем %A с %A" a b
            Some (a + b)
        | Some a, None ->
            printfn "комбинируем %A с None" a
            Some a
        | None, Some b ->
            printfn "комбинируем None с %A" b
            Some b
        | None, None ->
            printfn "комбинируем None с None"
            None
```

> In this new version of `Combine`, the *second* parameter is now a function, not an `int option`.
> So to combine them, we must first evaluate the function before doing the combination logic.

В этой новой версии `Combine` *второй* параметр стал функцией, не `int option`.
Так что, комбинируя их, мы должны сначала вызвать фукнци, преджде чем выполнить комбинационную логику.

> If we test this out:

Если мы протестируем этот вывод:

```fsharp
trace {
    return 1
    return 2
    } |> printfn "Результ return с последующим return: %A"
```

> We get the following (annotated) trace:

Мы получем следующую (аннотированный) трассировку:

```text
// entire expression is delayed
<fun:clo@318-69> - Delaying using <fun:delayed@295-6>

// entire expression is run
<fun:delayed@295-6> - Run Start.

// delayed entire expression is run
<fun:clo@318-69> - Starting Delayed Fn.

// first return
Returning a unwrapped 1 as an option

// delaying second return
<fun:clo@319-70> - Delaying using <fun:delayed@295-6>

// combine starts
Combine. Starting second param <fun:delayed@295-6>

    // delayed second return is run inside Combine
    <fun:clo@319-70> - Starting Delayed Fn.
    Returning a unwrapped 2 as an option
    <fun:clo@319-70> - Finished Delayed Fn. Result is Some 2
    // delayed second return is complete

Combine. Finished second param <fun:delayed@295-6>. Result is Some 2
combining 1 and 2
// combine is complete

<fun:clo@318-69> - Finished Delayed Fn. Result is Some 3
// delayed entire expression is complete

<fun:delayed@295-6> - Run End. Result is Some 3
// Run is complete

// final result is printed
Result for return then return: Some 3
```

```text
// откладываем вычисление всего выражения
<fun:Pipe #1 input at line 70@71> - отложена благодаря <fun:delayed@46>

// вычисляем выражеие целиком
<fun:delayed@46> - Run Начало.                                         

// вызываем первую отложенную функцию
<fun:Pipe #1 input at line 70@71> - Начало отложенной функции.         

// первый возврат
Return незавёрнутого 1 как Option                                         

// откладываем вторую отложенную функцию
<fun:Pipe #1 input at line 70@72-1> - отложена благодаря <fun:delayed@46> 

// начинаем комбинирование
Combine. Перед получением отложенного параметра <fun:delayed@46>          

    // вызываем вторую отложенную функцию внутри Combine
    <fun:Pipe #1 input at line 70@72-1> - Начало отложенной функции.          
    Return незавёрнутого 2 как Option                                         
    <fun:Pipe #1 input at line 70@72-1> - Конец отложенной функции. Результат Some 2
    // получили второй результат

Combine. После получения отложенного параметра <fun:delayed@46>. Результат Some 2
комбинируем 1 с 2                                                             
// комбинирование завершено

<fun:Pipe #1 input at line 70@71> - Конец отложенной функции. Результат Some 3
// комбинирование отложенного выражение завершено

<fun:delayed@46> - Run Конец. Результат Some 3   
// выполнение завершено

// печатаем результат
Результ return с последующим return: Some 3   
```

> ## Understanding the type constraints

## Понимание ограничений типов

> Up to now, we have used only our "wrapped type" (e.g. `int option`) and the delayed version (e.g. `unit -> int option`) in the implementation of our builder.

До сих пор в реализации нашего построителя мы использовали только типы-обёртки (такие как `int option`) и их отложенные версии (`unit -> int option`).

> But in fact we can use other types if we like, subject to certain constraints.
> In fact, understanding exactly what the type constraints are in a computation expression can clarify how everything fits together.

Но на самом деле мы можем использовать другие типы, если нам надо, с уётом определённых ограничений.
На самом деле, точное понимание того, какие ограничения типов существуют в вычислительном выражении, может прояснить, как всё друг с другом согласуется.

> For example, we have seen that:

Например, мы видели, что

> * The output of `Return` is passed into `Delay`, so they must have compatible types.
> * The output of `Delay` is passed into the second parameter of `Combine`.
> * The output of `Delay` is also passed into `Run`.

* Вывод `Return` передаётся в `Delay`, поэтому они должны иметь совместимые типы.
* Вывод `Delay` передаётся во второй параметр `Combine`.
* Вывод `Delay` передаётся в `Run`.

> But the output of `Return` does *not* have to be our "public" wrapped type.
> It could be an internally defined type instead.

Но вывод `Return` не должен быть нашим "публичным" типом-обёрткой.
Вместо этого он может быть внутренним определённым типом.

![Delay](./ce_return.png)

> Similarly, the delayed type does not have to be a simple function, it could be any type that satisfies the constraints.

Похожим образом, отложенный тип не обязан быть простой функцией, он может быть любым типом, который удовлетворяет огранчениям.

> So, given a simple set of return expressions, like this:

Итак, возьмём простой набор возвращаемых выражений, как здесь:

```fsharp
    trace {
        return 1
        return 2
        return 3
        } |> printfn "Результат трёх операторов return: %A"
```

> Then a diagram that represents the various types and their flow would look like this:

В этом случае диаграмма, которая представляет различные типы и их взаимодействие, будет выглядеть так:

![Delay](./ce_types.png)

> And to prove that this is valid, here is an implementation with distinct types for `Internal` and `Delayed`:

И чтобы доказать, что это корректно, вот реализация с отдельными типами для `Internal` и `Delayed`:

```fsharp
type Internal = Internal of int option
type Delayed = Delayed of (unit -> Internal)

type TraceBuilder() =
    member this.Bind(m, f) =
        match m with
        | None ->
            printfn "Bind с None. Выход."
        | Some a ->
            printfn "Bind с Some(%A). Продолжение" a
        Option.bind f m

    member this.Return(x) =
        printfn "Return развёрнутого %A как Option" x
        Internal (Some x)

    member this.ReturnFrom(m) =
        printfn "Return завёрнутого (%A) напрямую" m
        Internal m

    member this.Zero() =
        printfn "Zero"
        Internal None

    member this.Combine (Internal x, Delayed g) : Internal =
        printfn "Combine. Начало %A" g
        let (Internal y) = g()
        printfn "Combine. Конец %A. Результат %A" g y
        let o =
            match x,y with
            | Some a, Some b ->
                printfn "комбинируем %A и %A" a b
                Some (a + b)
            | Some a, None ->
                printfn "комбинируем %A с None" a
                Some a
            | None, Some b ->
                printfn "комбинируем None с %A" b
                Some b
            | None, None ->
                printfn "комбинируем None с None"
                None
        // возвращаем новое значение, завёрнутое в Internal
        Internal o

    member this.Delay(funcToDelay) =
        let delayed = fun () ->
            printfn "%A - Delay. Вызов отложенной функции." funcToDelay
            let delayedResult = funcToDelay()
            printfn "%A - Delay. Возврат из отложенной функции. Результат %A" funcToDelay delayedResult
            delayedResult  // возвращаем результат

        printfn "%A - Откладываем используя %A" funcToDelay delayed
        Delayed delayed // возвращаем новую функцию, завёрнутую в Delayed

    member this.Run(Delayed funcToRun) =
        printfn "%A - Run Начало." funcToRun
        let (Internal runResult) = funcToRun()
        printfn "%A - Run Конец. Результат %A" funcToRun runResult
        runResult // возвращаем результат запуска отложенной функции

// создаём экземпляр процесса
let trace = new TraceBuilder()
```

> And the method signatures in the builder class methods look like this:

И сигнатуры методов в классе-построителе выглядят так:

```fsharp
type Internal = | Internal of int option
type Delayed = | Delayed of (unit -> Internal)

type TraceBuilder =
class
  new : unit -> TraceBuilder
  member Bind : m:'a option * f:('a -> 'b option) -> 'b option
  member Combine : Internal * Delayed -> Internal
  member Delay : funcToDelay:(unit -> Internal) -> Delayed
  member Return : x:int -> Internal
  member ReturnFrom : m:int option -> Internal
  member Run : Delayed -> int option
  member Zero : unit -> Internal
end
```

> Creating this artificial builder is overkill of course, but the signatures clearly show how the various methods fit together.

Создание такого искусственного построителя, конечно, слишком сложная задача, но сигнатуры ясно показывают как различные методы взаимодействуют друг с другом.

> ## Summary

## Заключение

> In this post, we've seen that:

В этом посте мы увидели. что

> * You need to implement `Delay` and `Run` if you want to delay execution within a computation expression.
> * Using `Delay` changes the signature of `Combine`.
> * `Delay` and `Combine` can use internal types that are not exposed to clients of the computation expression.

* Вам надо реализовать `Delay` и `Run` если вы хотите отложить выполнение вычислительного выражения.
* Использование `Delay` меняет сигнатуру `Combine`.
* `Delay` и `Combine` могут использовать внутренние типы, которые не доступны клиентам вычислительного выражения.

> The next logical step is wanting to delay execution *outside* a computation expression until you are ready, and that will be the topic on the next but one post.
> But first, we'll take a little detour to discuss method overloads.

Следующий логический шаг это желание отложить выполнение *снаружи* вычислительного выражения до тех пор пока вы не будете готовы, и этот вопрос будет рассмотрен в следующем посте.
Но сначала мы немного отвлечёмся чтобы обсудить перегрузки методов.
