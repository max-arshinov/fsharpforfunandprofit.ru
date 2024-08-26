---
layout: post
# title: "Implementing a CE: The rest of the standard methods"
title: "Реализуем вычислительные выражения: Оставшиеся стандартные методы"
# description: "Implementing While, Using, and exception handling"
description: "Реализуем While, Using и обработку исключений"
date: 2013-01-30
nav: thinking-functionally
# seriesId: "Computation Expressions"
seriesId: "Вычислительные выражения"
seriesOrder: 11
---

> We're coming into the home stretch now.
> There are only a few more builder methods that need to be covered, and then you will be ready to tackle anything!

Мы выходим на финишную прямую.
Осталось несколько методов класса-построителя, с которыми надо разобраться, и вы готовы к самостоятельному плаванию!

> These methods are:

Вот эти методы:

> * `While` for repetition.
> * `TryWith` and `TryFinally` for handling exceptions.
> * `Use` for managing disposables

* `While` для повторения.
* `TryWith` и `TryFinally` для обработки исключений.
* `Use` для управления освобождаемыми ресурсами.

> Remember, as always, that not all methods need to be implemented.
> If `While` is not relevant to you, don't bother with it.

Помните, что, как и в предыдущих случаях, не все методы требуют реализации.
Если `While` вам не важен, вы можете о нём не беспокоиться.

> One important note before we get started: **all the methods discussed here rely on [delays](/posts/computation-expressions-builder-part3/)** being used.
> If you are not using delay functions, then none of the methods will give the expected results.

Одно важное замечание, прежде чем мы начнём: **все обсуждаемые здесь методы, основаны на использовании [отложенных вычислений](/posts/computation-expressions-builder-part3/)**.
Если вы не используете отложенные функции, ни один из этих методов не даст ожидаемых результатов.

> {{<alertinfo>}}
> Note that the "builder" in the context of a computation expression is not the same as the OO "builder pattern" for constructing and validating objects.
> There is a post on the ["builder pattern" here](../builder-pattern).
> {{</alertinfo>}}

{{<alertinfo>}}
Обратите внимание, что "построитель" в контексте вычислительных выражений — это не то же самое, что объектно-ориентированный паттерн "строитель", который используется для конструирования и валидации объектов.
{{</alertinfo>}}

> ## Implementing "While"

## Реализуем 'While'

> We all know what "while" means in normal code, but what does it mean in the context of a computation expression?
> To understand, we have to revisit the concept of continuations again.

Все мы знаем, что означает "while" в обычном коде, но что он означает в контексте вычислительных выражений?
Чтобы разобраться, нам снова потребуется вернуться к концепции продолжений.

> In previous posts, we saw that a series of expressions is converted into a chain of continuations like this:

В предыдущих постах мы видели, что последовательность выражений можно превратить в цепочку продолжений, как здесь:

```fsharp
Bind(1,fun x ->
   Bind(2,fun y ->
     Bind(x + y,fun z ->
        Return(z)  // или Yield
```

> And this is the key to understanding a "while" loop -- it can be expanded in the same way.

И это ключ к пониманию цикла "while" — он может быть развёрнут похожим образом.

> First, some terminology.
> A while loop has two parts:

Для начала немного терминологии.
Цикл "while" состоит из двух частей:

> * There is a test at the top of the "while" loop which is evaluated each time to determine whether the body should be run.
>   When it evaluates to false, the while loop is "exited".
>   In computation expressions, the test part is known as the **"guard"**.
>   The test function has no parameters, and returns a bool, so its signature is `unit -> bool`, of course.
> * And there is the body of the "while" loop, evaluated each time until the "while" test fails.
>   In computation expressions, this is a delay function that evaluates to a wrapped value. Since the body of the while loop is always the same, the same function is evaluated each time.
>   The body function has no parameters, and returns nothing, and so its signature is just `unit -> wrapped unit`.

* В начале цикла "while" есть проверка, которая каждый раз вычислятеся, чтобы определить, надо ли выполнять тело цикла.
  Если результат вычисления равен `false`, мы «выходим» из цикла.
  В вычислительных выражениях, провека известна, как **охранное выражение**.
  У проверяющей функции нет параметров, она возвращает булево значение, так что её сигнатура — очевидно, `unit -> bool`.
* Также у цикла "while" есть тело, которое выполняется, пока проверка успешно проходит.
  В вычислительных выражениях это отложенная функция, которая принимает завёрнутое значение. Поскольку тело цикла "while" всегда одно и то же, каждый раз вызывается одна и та же функция.
  Функция, реализующая тело, не имеет параметров и ничего не возвращает, поэтому её сигнатура `unit -> wrapped unit`.

> With this in place, we can create pseudo-code for a while loop using continuations:

В этом месте рассуждений, мы можем реализовать цикл "while", основываясь на продолжениях, используя псевдо-код.

```fsharp
// вызываем тестовую функцию
let bool = guard()
if not bool
then
    // выходим из цикла
    return what??
else
    // выполняем тело цикла
    body()

    // возвращаемся к началу цикла

    // вызываем тестовую функцию снова
    let bool' = guard()
    if not bool'
    then
        // выходим из цикла
        return what??
    else
        // выполняем тело цикла снова
        body()

        // возвращаемся к началу цикла

        // вызываем тестовую функцию в третий раз
        let bool'' = guard()
        if not bool''
        then
            // выходим из цикла
            return what??
        else
            // выполняем тело цикла в третий раз
            body()

            // и т.д.
```

> One question that is immediately apparent is: what should be returned when the while loop test fails?
> Well, we have seen this before with `if..then..`, and the answer is of course to use the `Zero` value.

Сразу возникает вопрос: что нужно вернуть если проверка в цикле не сработала?
Что ж, вы встречали подобное, когда обсуждали `if..then..` и ответ, конечно — использовать значение `Zero`.

> The next thing is that the `body()` result is being discarded.
> Yes, it is a unit function, so there is no value to return, but even so, in our expressions, we want to be able to hook into this so we can add behavior behind the scenes.
> And of course, this calls for using the `Bind` function.

Затем мы должны избавиться от результата `body()`.
Да, это функция с типом возврата `unit`, так что возвращать ничего не нужно, но и в этом случае мы хотим каким-то образом встроить в неё собственный код, потому что нам нужны побочные эффекты.
И, конечно, её надо вызывать с помощью `Bind`.

> So here is a revised version of the pseudo-code, using `Zero` and `Bind`:

Вот пересмотренная версия псевдо-кода, методами `Zero` и `Bind`:

```fsharp
// вызываем тестовую функцию
let bool = guard()
if not bool
then
    // выходим из цикла
    return Zero
else
    // выполняем тело цикла
    Bind( body(), fun () ->

        // вызываем тестовую функцию снова
        let bool' = guard()
        if not bool'
        then
            // выходим из цикла
            return Zero
        else
            // выполняем тело цикла снова
            Bind( body(), fun () ->

                // вызываем тестовую функцию в третий раз
                let bool'' = guard()
                if not bool''
                then
                    // выходим из цикла
                    return Zero
                else
                    // выполняем тело цикла в третий раз
                    Bind( body(), fun () ->

                    // и т.д.
```

> In this case, the continuation function passed into `Bind` has a unit parameter, because the `body` function does not have a value.

В нашем случае, функция-продолжение, передаваемая в `Bind`, имеет параметр типа `unit`, поскольку функция `body` не возвращает значения.

> Finally, the pseudo-code can be simplified by collapsing it into a recursive function like this:

В конечном итоге, псевдо-код может быть упрощён путём сворачивания в рекурсивную функцию приблизительно так:

```fsharp
member this.While(guard, body) =
    // вызываем тестовую функцию
    if not (guard())
    then
        // выходим из цикла
        this.Zero()
    else
        // выполняем тело цикла
        this.Bind( body(), fun () ->
            // вызываем рекурсивно
            this.While(guard, body))
```

> And indeed, this is the standard "boiler-plate" implementation for `While` in almost all builder classes.

В действительности, это стандартная «шаблонная» реализация `While` почти во всех классах-построителях.

> It is a subtle but important point that the value of `Zero` must be chosen properly.
> In previous posts, we saw that we could set the value for `Zero` to be `None` or `Some ()` depending on the workflow.
> For `While` to work however, the `Zero` *must be* set to `Some ()` and not `None`, because passing `None` into `Bind` will cause the whole thing to aborted early.

Тонкий, но важный момент заключается в том, что значение `Zero` должно быть выбрано правильно.
В предыдущих постах мы видели, что мы можем использовать для `Zero` и значение `None` и значение `Some ()`, в зависимости от процесса.
Однако, чтобы `While` работал корректно, мы *должны* в качестве `Zero` использовать `Some ()`, а не `None`, потому что передача `None` в `Bind` приведёт к преждевременному завершению цикла.

> Also note that, although this is a recursive function, we didn't need the `rec` keyword.
> It is only needed for standalone functions that are recursive, not methods.

Также обратите внимание, что, не смотря на то, что это рекурсивная функция, мы не используем ключевое слово `rec`.
Оно требуется только для отдельных рекурсивных функций, а не для методов.

> ### "While" in use

### `While`: инструкция по применению

> Let's look at it being used in the `trace` builder.
> Here's the complete builder class, with the `While` method:

Давайте посмотрим, как цикл работает в построителе `build`.
Вот класс-построитель целиком, с методом `While`:

```fsharp
type TraceBuilder() =
    member this.Bind(m, f) =
        match m with
        | None ->
            printfn "Bind с None. Выходим."
        | Some a ->
            printfn "Bind с Some(%A). Продолжаем" a
        Option.bind f m

    member this.Return(x) =
        Some x

    member this.ReturnFrom(x) =
        x

    member this.Zero() =
        printfn "Zero"
        this.Return ()

    member this.Delay(f) =
        printfn "Delay"
        f

    member this.Run(f) =
        f()

    member this.While(guard, body) =
        printfn "While: проверка"
        if not (guard())
        then
            printfn "While: Zero"
            this.Zero()
        else
            printfn "While: цикл"
            this.Bind( body(), fun () ->
                this.While(guard, body))

// создаём экземпляр процесса
let trace = new TraceBuilder()
```

> If you look at the signature for `While`, you will see that the `body` parameter is `unit -> unit option`, that is, a delayed function.
> As noted above, if you don't implement `Delay` properly, you will get unexpected behavior and cryptic compiler errors.

Если вы взглянете на сигнатуру `While`, вы увидите, что параметр `body` имеет тип `unit -> unit option`, то есть это отложенная функция.
Как я писал выше, если вы не реализуете `Delay` должным образом, вы получите неопределённое поведение и загадочные ошибки компилятора.

```fsharp
type TraceBuilder =
    // прочие методы
    member
      While : guard:(unit -> bool) * body:(unit -> unit option) -> unit option

```

> And here is a simple loop using a mutable value that is incremented each time round.

А вот простой цикл, использующий мутабельную переменную, значение которой увеличивается на 1 при каждой итерации.

```fsharp
let mutable i = 1
let test() = i < 5
let inc() = i <- i + 1

let m = trace {
    while test() do
        printfn "i = %i" i
        inc()
    }
```

> ## Handling exceptions with "try..with"

## Обработка исключений с помощью `try..with`

> Exception handling is implemented in a similar way.

Обработка исключений реализуется похожим образом.

> If we look at a `try..with` expression for example, it has two parts:

Взгянув на выражение `try..with`, мы обнаружим, что оно состоит из двух частей:

> * There is the body of the "try", evaluated once.
>   In a computation expressions, this will be a delayed function that evaluates to a wrapped value.
>   The body function has no parameters, and so its signature is just `unit -> wrapped type`.
> * The "with" part handles the exception.
>   It has an exception as a parameters, and returns the same type as the "try" part, so its signature is `exception -> wrapped type`.

* Здесь есть тело `try`, которое выполняется один раз.
  В вычислительных выражениях оно превратится в отложенную функцию, которая
  возвращает завёрнутое значение.
  У функции нет параметров, так что её сигнатура — это `unit -> wrapped type`.
* Часть `with` обрабатываем исключения.
  В качестве параметра она принимает исключение и возвращает тот же тип, что и часть `try`, так что её сигнатура — это `exception -> wrapped type`.

> With this in place, we can create pseudo-code for the exception handler:

Имея это в виду, мы можем создать псевдо-код для обработчика исключений:

```fsharp
try
    let wrapped = delayedBody()
    wrapped  // возвращаем завёрнутое значение
with
| e -> handlerPart e
```

> And this maps exactly to a standard implementation:

И это в точности соответствует стандартной реализации:

```fsharp
member this.TryWith(body, handler) =
    try
        printfn "TryWith Тело"
        this.ReturnFrom(body())
    with
        e ->
            printfn "TryWith Обработка исключения"
            handler e
```

> As you can see, it is common to use pass the returned value through `ReturnFrom` so that it gets the same treatment as other wrapped values.

Как видите, общей практикой для возврата завёрнутого значения является вызов `ReturnFrom`, так что он будет обработан также, как и другие завёрнутые значения.

> Here is an example snippet to test how the handling works:

Вот фрагмент примера для проверки обработчика:

```fsharp
trace {
    try
        failwith "бах!"
    with
    | e -> printfn "Исключение! %s" e.Message
    } |> printfn "Результат %A"
```

> ## Implementing "try..finally"

## Реализуем `try..finally`

> `try..finally` is very similar to `try..with`.

Конструкция `try..finally` очень похожа на `try..with`.

> * There is the body of the "try", evaluated once.
>   The body function has no parameters, and so its signature is `unit -> wrapped type`.
> * The "finally" part is always called.
>   It has no parameters, and returns a unit, so its signature is `unit -> unit`.

* Здесь есть тело `try`, которое выполняется однократно.
  Тело не имеет параметров и его сигнатура — это `unit -> wrapped type`.
* Часть `finally` вызывается всегда.
  У неё нет параметров и она возвращает `unit`, так что её сигнатура — это `unit -> unit`.

> Just as with `try..with`, the standard implementation is obvious.

Как и в случае с `try..with`, стандартная реализация очевидна.

```fsharp
member this.TryFinally(body, compensation) =
    try
        printfn "TryFinally Цикл"
        this.ReturnFrom(body())
    finally
        printfn "TryFinally восстановление"
        compensation()
```

> Another little snippet:

Ещё один фрагментик:

```fsharp
trace {
    try
        failwith "бах!"
    finally
        printfn "ок"
    } |> printfn "Результат %A"
```

> ## Implementing "using"

## Реализуем `using`

> The final method to implement is `Using`.
> This is the builder method for implementing the `use!` keyword.

Последний метод для реализации — это `Using`.
Это метод построителя для реализации ключевого слова `use!`.

> This is what the MSDN documentation says about `use!`:

Вот что документация MSDN говорит об `use!`:

```text
{| use! value = expr in cexpr |}
```

> is translated to:

транслируется в:

```text
builder.Bind(expr, (fun value -> builder.Using(value, (fun value -> {| cexpr |} ))))
```

> In other words, the `use!` keyword triggers both a `Bind` and a `Using`.
> First a `Bind` is done to unpack the wrapped value, and then the unwrapped disposable is passed into `Using` to ensure disposal, with the continuation function as the second parameter.

Иными словами, ключевое слово `use!` запускает как `Bind`, так и `Using`.
Сначала `Bind` распаковывает завёрнутое значение, и затем незавёрнутый освобождаемый объект передаётся в `Using`, для последующего освобождения, вместе с функцией-продолжением в качестве второго параметра.

> Implementing this is straightforward.
> Similar to the other methods, we have a body, or continuation part, of the "using" expression, which is evaluated once.
> This body function has a "disposable" parameter, and so its signature is `#IDisposable -> wrapped type`.

Это довольно просто реализуется.
Как и в других методах, у нас есть тело, или часть-продолжение выражения `Using`, которое выполняется один раз.
У этой функции есть параметр `disposable`, так что её сигнатура — это `#IDisposable -> wrapped type`.

> Of course we want to ensure that the disposable value is always disposed no matter what, so we need to wrap the call to the body function in a `TryFinally`.

Конечно мы хотим быть уверены, что освобождаемое значение освобождается в любом случае, так что нам надо завернуть вызов фукнкции-тела в `TryFinally`.

> Here's a standard implementation:

Вот стандартная реализация:

```fsharp
member this.Using(disposable:#System.IDisposable, body) =
    let body' = fun () -> body disposable
    this.TryFinally(body', fun () ->
        match disposable with
            | null -> ()
            | disp -> disp.Dispose())
```

> Notes:

Замечания:

> * The parameter to `TryFinally` is a `unit -> wrapped`, with a *unit* as the first parameter, so we created a delayed version of the body that is passed in.
> * Disposable is a class, so it could be `null`, and we have to handle that case specially.
>   Otherwise we just dispose it in the "finally" continuation.

* Параметр для `TryFinally` — это `unit -> wrapped`, с `unit` в качестве первого параметра, так что мы создали отложенную версию тела, которую и передаём.
  * Освобождаемое значение — это класс, так что он может быть `null`, и мы должны обрабатывать этот случай отдельно.
  В противном случае мы просто освобождаем его в продолжении `finally`.

> Here's a demonstration of `Using` in action.
> Note that the `makeResource` makes a *wrapped* disposable.
> If it wasn't wrapped, we wouldn't need the special `use!` and could just use a normal `use` instead.

Вот демонстрация `Using` в действии.
Обратите внимание, что `makeResource` создаёт *завёрнутый* освобождаемый объект.
Если он не был завёрнут, нам не нужна специальная версия `use!` и мы можем использовать нормальный оператор `use`.

```fsharp
let makeResource name =
    Some {
    new System.IDisposable with
    member this.Dispose() = printfn "Освобождаем %s" name
    }

trace {
    use! x = makeResource "привет"
    printfn "Освобождаем в use!"
    return 1
    } |> printfn "Результат: %A"
```


> ## Пересмотр `For`

> Finally, we can revisit how `For` is implemented.
> In the previous examples, `For` took a simple list parameter.
> But with `Using` and `While` under our belts, we can change it to accept any `IEnumerable<_>` or sequence.

Напоследок вернёмся к реализации оператора `For`.
В предыдущих примерах `For` принимал простой параметр-список.
Но, имея в запасе `Using` и `While`, мы можем переписать его так, чтобы он принимал любую реализацию `IEnumerable<_>` или последовательность.

> Here's the standard implementation for `For` now:

Вот стандартная реализация для `For`:

```fsharp
member this.For(sequence:seq<_>, body) =
       this.Using(sequence.GetEnumerator(),fun enum ->
            this.While(enum.MoveNext,
                this.Delay(fun () -> body enum.Current)))
 ```

> As you can see, it is quite different from the previous implementation, in order to handle a generic `IEnumerable<_>`.

Как видите, этот код отличается от предыдущих реализаций обработкой обобщённого параметра `IEnumerable<_>`.

> * We explicitly iterate using an `IEnumerator<_>`.
> * `IEnumerator<_>` implements `IDisposable`, so we wrap the enumerator in a `Using`.
> * We use `While .. MoveNext` to iterate.
> * Next, we pass the `enum.Current` into the body function
> * Finally, we delay the call to the body function using `Delay`

* Мы явно итерируем, используя `IEnumerator<_>`.
* `IEnumerator<_>` реализует `IDisposable`, так что мы заворачиваем итератор в `Using`,
* Мы используем `While .. MoveNext` для итерации.
* Далее, мы передаём `enum.Current` в функцию-тело.
* Наконец, мы откладываем вызов функции-тела используя `Delay`.

> ## Complete code without tracing

## Полный код без трассировки

> Up to now, all the builder methods have been made more complex than necessary by the adding of tracing and printing expressions.
> The tracing is helpful to understand what is going on, but it can obscure the simplicity of the methods.

До сих пор наш код был сложнее, чем надо, из-за операторов трассировки и печати.
Трассировка полезна для понимания происходящего, но на затемняет простоту методов.

> So as a final step, let's have a look at the complete code for the "trace" builder class, but this time without any extraneous code at all.
> Even though the code is cryptic, the purpose and implementation of each method should now be familiar to you.

Так что в качестве финального шага бросим взгляд на полный код класса-построителя для `trace`, но на этот раз без всякого постороннего кода.
Несмотря на то, что код достаточно сложный, назначение и реализация каждого метода должны быть вам понятны.

```fsharp
type TraceBuilder() =

    member this.Bind(m, f) =
        Option.bind f m

    member this.Return(x) = Some x

    member this.ReturnFrom(x) = x

    member this.Yield(x) = Some x

    member this.YieldFrom(x) = x

    member this.Zero() = this.Return ()

    member this.Delay(f) = f

    member this.Run(f) = f()

    member this.While(guard, body) =
        if not (guard())
        then this.Zero()
        else this.Bind( body(), fun () ->
            this.While(guard, body))

    member this.TryWith(body, handler) =
        try this.ReturnFrom(body())
        with e -> handler e

    member this.TryFinally(body, compensation) =
        try this.ReturnFrom(body())
        finally compensation()

    member this.Using(disposable:#System.IDisposable, body) =
        let body' = fun () -> body disposable
        this.TryFinally(body', fun () ->
            match disposable with
                | null -> ()
                | disp -> disp.Dispose())

    member this.For(sequence:seq<_>, body) =
        this.Using(sequence.GetEnumerator(),fun enum ->
            this.While(enum.MoveNext,
                this.Delay(fun () -> body enum.Current)))

```

> After all this discussion, the code seems quite tiny now.
> And yet this builder implements every standard method, uses delayed functions.
> A lot of functionality in a just a few lines!

После всех этих обсуждений, код теперь кажется совсем крошечным.
И всё же этот построитель реализует все стандартные методы, включая отложенные функции.
Бездна функциональности всего в нескольких строках!
