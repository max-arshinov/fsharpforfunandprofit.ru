---
layout: post
# title: "Introducing 'bind'"
title: "Введение в `bind`"
# description: "Steps towards creating our own 'let!' "
description: "Шаги к созданию собственного оператора `let!`"
date: 2013-01-22
nav: thinking-functionally
# seriesId: "Computation Expressions"
seriesId: "Вычислительные выражения"
seriesOrder: 3
---

> In the last post we talked about how we can think of `let` as a nice syntax for doing continuations behind scenes.
> And we introduced a `pipeInto` function that allowed us to add hooks into the continuation pipeline.

В последнем посте мы говорили, что об операторе `let` можно думать, как о приятном синтаксисе для реализации продолжений с дополнительной — подкапотной — работой.

> Now we are ready to look at our first builder method, `Bind`, which formalizes this approach and is the core of any computation expression.

Сейчас мы готовы к тому, чтобы взглянуть на первый метод класса-строителя, `Bind`, который формализует этот подход и является сердцем любого вычислительного выражения.

> {{<alertinfo>}}
> Note that the "builder" in the context of a computation expression is not the same as the OO "builder pattern" for constructing and validating objects.
> There is a post on the ["builder pattern" here](../builder-pattern).
> {{</alertinfo>}}


{{<alertinfo>}}
Обратите внимание, что "класс-строитель" в контексте вычислительных выражений — это не то же самое, что "паттерн строитель", который применияется для конструиварония и валидации объектов.
Чтобы понять разницу, прочтите пост о ["паттерне строитель"](../builder-pattern).
{{</alertinfo>}}


> ### Introducing "Bind"

### Введение в "Bind"

> The [MSDN page on computation expressions](http://msdn.microsoft.com/en-us/library/dd233182.aspx) describes the `let!` expression as syntactic sugar for a `Bind` method. Let's look at this again:

[Страница MSDN о вычислительных выражениях](http://msdn.microsoft.com/en-us/library/dd233182.aspx) описывает `let!` как синтаксический сахар для метода `Bind`. Сравним их ещё раз:

> Here's the `let!` expression documentation, along with a real example:

Вот документация по оператору `let!` вместе с примером использования:

```fsharp
// документация
{| let! pattern = expr in cexpr |}

// пример
let! x = 43 in some expression
```

> And here's the `Bind` method documentation, along with a real example:

А вот документация по методу `Bind`, также с примером использования:

```fsharp
// документация
builder.Bind(expr, (fun pattern -> {| cexpr |}))

// пример
builder.Bind(43, (fun x -> some expression))
```

> Notice a few interesting things about this:

Обратим внимание на несколько интересных аспектов:

> * `Bind` takes two parameters, an expression (`43`) and a lambda.
> * The parameter of the lambda (`x`) is bound to the expression passed in as the first parameter. (In this case at least. More on this later.)
> * The parameters of `Bind` are reversed from the order they are in `let!`.

* `Bind` принимает два параметра: выражение (`43`) и лямбду.
* Параметры лямбды (`x`) связывается с выражением, переданным в качестве первого параметра. (По крайней мере, в этом примере. Подробности позже.)
* Параметры `Bind` записываются в порядке, противоположном их порядку в `let!`.

> So in other words, if we chain a number of `let!` expressions together like this:

Другими словами, если мы запишем подряд несколько операторов `let!` вот так:

```fsharp
let! x = 1
let! y = 2
let! z = x + y
```

> the compiler converts it to calls to `Bind`, like this:

компилятор превратит их в вызовы `Bind` вот так:

```fsharp
Bind(1, fun x ->
Bind(2, fun y ->
Bind(x + y, fun z ->
// и т.д.
```

> I think you can see where we are going with this by now.

У думаю, вы уже видите, к чему я веду.

> Indeed, our `pipeInto` function is exactly the same as the `Bind` method.

И действительно, наша функция `pipeInfo` — это то же самое, что и метод `Bind`.

> This is a key insight: *computation expressions are just a way to create nice syntax for something that we could do ourselves*.

Вот ключевая мысль: *вычислительное выражение — всего лишь упрощённая запись вещей, которые мы и так можем сделать*.

> ### A standalone bind function

### Функция `bind` под микроскопом

> Having a "bind" function like this is actually a standard functional pattern, and it is not dependent on computation expressions at all.

Рассмотренная нами функция `bind` в действительности являтеся стандартным функциональным паттерном и вообще не зависит от вычислительных выражений.

> First, why is it called "bind"? Well, as we've seen, a "bind" function or method can be thought of as feeding an input value to a function. This is known as "[binding](/posts/function-values-and-simple-values/)" a value to the parameter of the function (recall that all functions have only [one parameter](/posts/currying/)).

Во-первых, почему она называется "bind" (прявязать, связывать)? Что ж, как мы видели, функцию или метод "bind" можно рассматривать, как передачу входного значения в функцию. Этот процесс известен, как "[связывание]"(/posts/function-values-and-simple-values/) значения с параметром функции (помним, что в функциональных языках все функции можно привести к виду, когда они получают только [один параметр](/posts/currying/)).

> So when you think of `bind` this this way, you can see that it is similar to piping or composition.

Если смотреть на связыание с этой точки зрения, то оно напоминает конфейер или композицию функциий.

> In fact, you can turn it into an infix operation like this:

В действительности, вы можете превратить его в инфиксный оператор:

```fsharp
let (>>=) m f = pipeInto(m,f)
```

> *By the way, this symbol ">>=" is the standard way of writing bind as an infix operator. If you ever see it used in other F# code, that is probably what it represents.*

*Кстати, символ ">>=" — стандартная запись связывания в виде инфиксного оператора. Если вы когда-нибудь видели её в F#-коде, скорее всего, вы видели именно связывание.*

> Going back to the safe divide example, we can now write the workflow on one line, like this:

Возвращаясь к примеру с безопасным делением, мы можем переписать логику в одну строку:

```fsharp
let divideByWorkflow x y w z =
    x |> divideBy y >>= divideBy w >>= divideBy z
```

> You might be wondering exactly how this is different from normal piping or composition? It's not immediately obvious.

Вам, возможно, интересно, чем именно связывание отличается от обычных конвейера или композиции? Это не так очевидно.

> The answer is twofold:

Ответ здесь двойной:

> * First, the `bind` function has *extra* customized behavior for each situation. It is not a generic function, like pipe or composition.
> * Second, the input type of the value parameter (`m` above) is not necessarily the same as the output type of the function parameter (`f` above), and so one of the things that bind does is handle this mismatch elegantly so that functions can be chained.

* Во-первых, функция `bind` делает дополнительную работу, разную в разных ситуациях. Это не обобщённая функция, как конвейер или композиция.
* Во-вторых, тип входного параметра (`m` выше) не обязательно совпадает с типом результата функции (`f` выше), так что одна из вещей, которую делает `bind` — это элегантная обработка несоответствия типов, в результате которого вызовы можно объединять в цепочку.

> As we will see in the next post, bind generally works with some "wrapper" type. The value parameter might be of `WrapperType<TypeA>`, and then the signature of the function parameter of `bind` function is always `TypeA -> WrapperType<TypeB>`.

Как мы увидим в следующем посте, связывание в целом работает на базе какого-то типа-обёртки. Типом параметра может быть `WrapperType<TypeA>`, а сигнатурой функционального параметра функции `bind` будет `TypeA -> WrapperType<TypeB>`.

> In the particular case of the `bind` for safe divide, the wrapper type is `Option`. The type of the value parameter (`m` above) is `Option<int>` and the signature of the function parameter (`f` above) is `int -> Option<int>`.

В случае `bind` для безопасного деления, типом-обёрткой является `Option`. Тип входного параметра (`m` выше) — `Option<int>`, а сигнатура функционального параметре (`f` выше) — `int -> Option<int>`.

> To see bind used in a different context, here is an example of the logging workflow expressed using a infix bind function:

Чтобы увидеть связывание в разных контекстах, приведём пример логгирования, работающий посредством инфиксной функции `bind`:

```fsharp
let (>>=) m f =
    printfn "expression is %A" m
    f m

let loggingWorkflow =
    1 >>= (+) 2 >>= (*) 42 >>= id
```

> In this case, there is no wrapper type. Everything is an `int`. But even so, `bind` has the special behavior that performs the logging behind the scenes.

В этом случае нет даже типа-обёртки, используется только `int`. Но даже здесь у `bind` есть специальное поведение — логгирование — которое выполняется под капотом.

> ## Option.bind and the "maybe" workflow revisited

## `Option.bind`: ещё раз про обработку опциональных значений

> In the F# libraries, you will see `Bind` functions or methods in many places. Now you know what they are for!

В библеотеке F# вы не раз встретите функции или методы `Bind`. Теперь вы знаете, зачем они нужны!

> A particularly useful one is `Option.bind`, which does exactly what we wrote by hand above, namely

Особенно полезна функция `Option.bind`, которая делает в точности то, что мы написали выше, а именно

> * If the input parameter is `None`, then don't call the continuation function.
> * If the input parameter is `Some`, then do call the continuation function, passing in the contents of the `Some`.

* Если входной параметр имеет значение `None`, она не вызывает функцию-продолжение.
* Если входной параметр имеет значение `Some`, она вызывает функцию-продолжение, передавая ей содержимое `Some`.

> Here was our hand-crafted function:

Так выглядела функция, которую мы написали сами:

```fsharp
let pipeInto (m,f) =
   match m with
   | None ->
       None
   | Some x ->
       x |> f
```

> And here is the implementation of `Option.bind`:

А так выглядит реализация `Option.bind`:

```fsharp
module Option =
    let bind f m =
       match m with
       | None ->
           None
       | Some x ->
           x |> f
```

> There is a moral in this -- don't be too hasty to write your own functions. There may well be library functions that you can reuse.

Вот и мораль — не торопитесь писать свои функции. Может оказаться, что нужные библиотечные функции давно написаны!

> Here is the "maybe" workflow, rewritten to use `Option.bind`:

Вот методы класса-строителя опционального типа, реализованные через `Option.bind`:

```fsharp
type MaybeBuilder() =
    member this.Bind(m, f) = Option.bind f m
    member this.Return(x) = Some x
```

> ## Reviewing the different approaches so far

## Сравнение подходов

> We've used four different approaches for the "safe divide" example so far. Let's put them together side by side and compare them once more.

На данный момент мы использовали четыре различных подхода в примере с "безопасным делением". Давайте ещё раз сравним их строка за строкой.

> *Note: I have renamed the original `pipeInto` function to `bind`, and used `Option.bind` instead of our original custom implementation.*

*Примечание: я переименовал оригинальную функцию `pipeInfo` в `bind` и исользовал `Option.bind` вместо орагинальной самописной реализации.*

> First the original version, using an explicit workflow:

Для начала взглянем на оригинальную версию, явно описывающую весь процесс:

```fsharp
module DivideByExplicit =

    let divideBy bottom top =
        if bottom = 0
        then None
        else Some(top/bottom)

    let divideByWorkflow x y w z =
        let a = x |> divideBy y
        match a with
        | None -> None  // прерываем
        | Some a' ->    // продолжаем
            let b = a' |> divideBy w
            match b with
            | None -> None  // прерываем
            | Some b' ->    // продолжаем
                let c = b' |> divideBy z
                match c with
                | None -> None  // прерываем
                | Some c' ->    // продолжаем
                    // возврат
                    Some c'
    // проверяем
    let good = divideByWorkflow 12 3 2 1
    let bad = divideByWorkflow 12 3 0 1
```

> Next, using our own version of "bind"  (a.k.a. "pipeInto")

Теперь — на версию с самописной функцией `bind` (которую мы называли `pipeInfo`):

```fsharp
module DivideByWithBindFunction =

    let divideBy bottom top =
        if bottom = 0
        then None
        else Some(top/bottom)

    let bind (m,f) =
        Option.bind f m

    let return' x = Some x

    let divideByWorkflow x y w z =
        bind (x |> divideBy y, fun a ->
        bind (a |> divideBy w, fun b ->
        bind (b |> divideBy z, fun c ->
        return' c
        )))

    // test
    let good = divideByWorkflow 12 3 2 1
    let bad = divideByWorkflow 12 3 0 1
```

> Next, using a computation expression:

Далее на версию с вычислительным выражением:

```fsharp
module DivideByWithCompExpr =

    let divideBy bottom top =
        if bottom = 0
        then None
        else Some(top/bottom)

    type MaybeBuilder() =
        member this.Bind(m, f) = Option.bind f m
        member this.Return(x) = Some x

    let maybe = new MaybeBuilder()

    let divideByWorkflow x y w z =
        maybe
            {
            let! a = x |> divideBy y
            let! b = a |> divideBy w
            let! c = b |> divideBy z
            return c
            }

    // test
    let good = divideByWorkflow 12 3 2 1
    let bad = divideByWorkflow 12 3 0 1
```

> And finally, using bind as an infix operation:

И, наконец, на версию с `bind` в качестве инфиксного оператора:

```fsharp
module DivideByWithBindOperator =

    let divideBy bottom top =
        if bottom = 0
        then None
        else Some(top/bottom)

    let (>>=) m f = Option.bind f m

    let divideByWorkflow x y w z =
        x |> divideBy y
        >>= divideBy w
        >>= divideBy z

    // test
    let good = divideByWorkflow 12 3 2 1
    let bad = divideByWorkflow 12 3 0 1
```

> Bind functions turn out to be very powerful. In the next post we'll see that combining `bind` with wrapper types creates an elegant way of passing extra information around in the background.

Функции связывания оказываются очень мощными. В следующем посте мы увидим, как комбинирование `bind` с типами-обёртками даёт элегантную возможность неявно передавать дополнительную информацию.

> ## Exercise: How well do you understand?

## Упражнение: Насколько вы разобрались в материале?

> Before you move on to the next post, why don't you test yourself to see if you have understood everything so far?

Перед тем, как двинуться дальше, почему бы вам не проверить, насколько хорошо вы поняли всё, что мы обсудили к этому моменту?

> Here is a little exercise for you.

Вот для вас небольшое упражнение.

> **Part 1 - create a workflow**

### Часть 1 — реализуйте процесс

> First, create a function that parses a string into a int:

Для начала напишите функцию, которая преобразует строку в целое число:

```fsharp
let strToInt str = ???
```

> and then create your own computation expression builder class so that you can use it in a workflow, as shown below.

и затем — класс-строитель вычислительного выражения, такой, чтобы его можно было использовать в программе, показанной ниже.

```fsharp
let stringAddWorkflow x y z =
    yourWorkflow
        {
        let! a = strToInt x
        let! b = strToInt y
        let! c = strToInt z
        return a + b + c
        }

// проверяем
let good = stringAddWorkflow "12" "3" "2"
let bad = stringAddWorkflow "12" "xyz" "2"
```

> **Part 2 -- create a bind function**

### Часть 2 — напишите функцию `bind`

> Once you have the first part working, extend the idea by adding two more functions:

Как только ваш код заработает, расширьте его, добавлив две новых функции:

```fsharp
let strAdd str i = ???
let (>>=) m f = ???
```

> And then with these functions, you should be able to write code like this:

Теперь, с помощью этих функций вам должно быть нетрудно переписать код в таком стиле:

```fsharp
let good = strToInt "1" >>= strAdd "2" >>= strAdd "3"
let bad = strToInt "1" >>= strAdd "xyz" >>= strAdd "3"
```

> ## Summary ##

## Заключение

> Here's a summary of the points covered in this post:

Вот о чём, в двух словах, мы говорили в этом посте:

> * Computation expressions provide a nice syntax for continuation passing, hiding the chaining logic for us.
> * `bind` is the key function that links the output of one step to the input of the next step.
> * The symbol `>>=` is the standard way of writing bind as an infix operator.

* Вычислительные выражания — это красивый синтаксис для программирования через передачу продолжений, скрывающий от нас вложенность когда.
* `bind` — ключевая функция которая связывает выход, полученный на текущем шаге с входо следующего шага.
* Символ ">>=" — стандартная запись `bind` в виде инфиксного оператора.
