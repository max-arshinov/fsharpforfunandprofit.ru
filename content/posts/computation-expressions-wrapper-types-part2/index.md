---
layout: post
# title: "More on wrapper types"
title: "Подробнее про типы-обёртки"
# description: "We discover that even lists can be wrapper types"
description: "Выясняем, что даже списки могут быть тапами-обёртками"
date: 2013-01-24
nav: thinking-functionally
# seriesId: "Computation Expressions"
seriesId: "Вычислительные выражения"
seriesOrder: 5
---

> In the previous post, we looked at the concept of "wrapper types" and their relation to computation expressions.
> In this post, we'll investigate what types are suitable for being wrapper types.

В прошлом посте мы познакомились с концепцией "типов-обёрток" и с их взаимосвязью с вычислительными выражениями.
В этом посте мы разберёмся, какие типы могут быть использованы в качестве типов-обёрток.

> ## What kinds of types can be wrapper types?

## Какого рода типы могут быть типами-обёртками?

> If every computation expression must have an associated wrapper type, then what kinds of type can be used as wrapper types?
> Are there any special constraints or limitations that apply?

Если каждое вычислительное выражение должно быть ассоциировано с типом-обёрткой, тогда какого рода типы можно использовать в качестве типов-обёрток?
Есть ли у них какие-то особые ограничения или требования?

> There is one general rule, which is:

Есть одно основное правило, которое гласит:

> * **Any type with a generic parameter can be used as a wrapper type**

* **Любой обобщённый тип с параметром может быть использован в качестве типа-обёртки**

> So for example, you can use `Option<T>`, `DbResult<T>`, etc., as wrapper types, as we have seen.
> And you can use wrapper types that restrict the type parameter, such as `Vector<int>`.

Например, как мы видели, можно использовать `Option<T>`, `DbResult<T>` и т.д., как типы-обёртки.

> But what about other generic types like `List<T>` or `IEnumerable<T>`?
> Surely they can't be used?
> Actually, yes, they *can* be used! We'll see how shortly.

Но что насчёт других обобщённых типов, таких как `List<T>` или `IEnumerable<T>`?
Неужели их нельзя использовать?
На самом деле конечно *можно*. И вскоре мы в этом убедимся.

> ## Can non-generic wrapper types work?

## Могут ли работать необобщённые типы-обёртки?

> Is it possible to use a wrapper type that does *not* have a generic parameter?

Можно ли создать тип-обёртку, у которого *нет* обобщённого параметра?

> For example, we saw in an earlier example an attempt to do addition on strings, like this: `"1" + "2"`.
> Can't we be clever and treat `string` as a wrapper type for `int` in this case?
> That would be cool, yes?

Например, в одном из предыдущих примеров, мы попытались реализовать сложение строк вида `"1" + "2"`.
Нельзя ли в этом случае как-нибудь искустно трактовать `string` как тип-обёртку над `int`?
Это было бы здорово, правда?

> Let's try.
> We can use the signatures of `Bind` and `Return` to guide our implementation.

Давайте попробуем.
Мы можем использовать сигнатуры методов `Bind` и `Return`, чтобы нащупать нашу реализацию.

> * `Bind` takes a tuple.
>   The first part of the tuple is the wrapped type (`string` in this case), and the second part of the tuple is a function that takes an unwrapped type and converts it to a wrapped type.
>   In this case, that would be `int -> string`.
> * `Return` takes an unwrapped type (`int` in this case) and converts it to a wrapped type.
>   So in this case, the signature of `Return` would be `int -> string`.

* `Bind` получает кортеж.
  Первая часть кортежа — тип-обёртка (в нашем случае `string`), а вторая часть кортежа — это функция, которая принимает незавёрнутый тип и превращает его в завёрнутый тип.
  В данном случае, её сигнатура будет `int -> string`.
* `Return` получает незавёрнутый тип (в нашем случае `int`) и превращает его в завёрнутый тип.
  И в этом случае, сигнатура метода `Return` будет `int -> string`.

> How does this guide the implementation?

Как это помогает нащупать реализацию?

> * The implementation of the "rewrapping" function, `int -> string`, is easy.
>   It is just "toString" on an int.
> * The bind function has to unwrap a string to an int, and then pass it to the function.
>   We can use `int.Parse` for that.
> * But what happens if the bind function *can't* unwrap a string, because it is not a valid number?
>   In this case, the bind function *must* still return a wrapped type (a string), so we can just return a string such as "error".

* Реализация "заворачивающей" функции с сигнатурой `int -> string` очень проста.
  Это всего лишь "toString" типа `int`.
* Функция связывания должна развернуть значение из `string` в `int` и затем передать его в функцию.
  Для реализации мы можем использовать `int.Parse`.
* Но что произойдёт, если функция связывания *не сможет* извлечь значени из строки, потому что оно не является корректным числом?
  Вы этом случае функция связывания все ещё *должна* вернуть тип-обёртку (`string`), так что мы можем просто вернуть что-то вроде "ошибка".
  
> Here's the implementation of the builder class:

Вот реализация класса-строителя:

```fsharp
type StringIntBuilder() =

    member this.Bind(m, f) =
        let b,i = System.Int32.TryParse(m)
        match b,i with
        | false,_ -> "ошибка"
        | true,i -> f i

    member this.Return(x) =
        sprintf "%i" x

let stringint = new StringIntBuilder()
```

> Now we can try using it:

Теперь мы можем его испытать:

```fsharp
let good =
    stringint {
        let! i = "42"
        let! j = "43"
        return i+j
        }
printfn "хороший результат=%s" good
```

> And what happens if one of the strings is invalid?

А что будет, если одна из строк будет некорректной?

```fsharp
let bad =
    stringint {
        let! i = "42"
        let! j = "xxx"
        return i+j
        }
printfn "плохой результат=%s" bad
```

> That looks really good -- we can treat strings as ints inside our workflow!

Всё это выглядит поистине здорово — мы можем обращаться со строками как с целыми числами внутри нашего процесса!

> But hold on, there is a problem.

Но подождите, здесь есть проблема.

> Let's say we give the workflow an input, unwrap it (with `let!`) and then immediately rewrap it (with `return`) without doing anything else.
> What should happen?

Представим, что мы передадим значение в процесс, развернём его (с помощью `let!`) и затем немедленно завернём (с помощью `return`), не выполняя никаких других действий?
Что случится тогда?

```fsharp
let g1 = "99"
let g2 = stringint {
            let! i = g1
            return i
            }
printfn "g1=%s g2=%s" g1 g2
```

> No problem.
> The input `g1` and the output `g2` are the same value, as we would expect.

Никаких проблем.
Входное значение `g1` и выходное значение `g2` совпадают, как мы могли бы ожидать.

> But what about the error case?

Но что на счёт ошибочного случая?

```fsharp
let b1 = "xxx"
let b2 = stringint {
            let! i = b1
            return i
            }
printfn "b1=%s b2=%s" b1 b2
```

> In this case we have got some unexpected behavior.
> The input `b1` and the output `b2` are *not* the same value.
> We have introduced an inconsistency.

Здесь мы получили не совсем ожидаемое поведение.
Входное значение `b1` и выходное значение `b2` *не* совпадают.
Мы привнесли несоответствие.

> Would this be a problem in practice?
> I don't know.
> But I would avoid it and use a different approach, like options, that are consistent in all cases.

Является ли это проблемой на практике?
Я не знаю.
Но я бы постарался избегать такого подхода, и попробовал бы другой, например, с опциональным типом, который согласован во всех ситуациях.

> ## Rules for workflows that use wrapper types

## Правила для процесса, который использует тип-обёртку

> Here's a question?
> What is the difference between these two code fragments, and should they behave differently?

А вот вам вопрос на засыпку!
Есть ли отличия между этими двумя фрагментами кода, и должны ли они вести себя по разному?

```fsharp
// фрагмент до рефакторинга
myworkflow {
    let wrapped = // какое-то завёрнутое значение
    let! unwrapped = wrapped
    return unwrapped
    }

// фрагмет после рефакторинга
myworkflow {
    let wrapped = // какое-то завёрнутое значение
    return! wrapped
    }
```

> The answer is no, they should not behave differently.
> The only difference is that in the second example, the `unwrapped` value has been refactored away and the `wrapped` value is returned directly.

Ответ — нет, они не должны вести себя по разному.
Единственное отличие во втором примере в том, что значение `unwrapped` было выброшено в результате рефакторинга, и значение `wrapped` возвращается напрямую.

> But as we just saw in the previous section, you can get inconsistencies if you are not careful.
> So, any implementation you create should be sure to follow some standard rules, which are:

Но как мы только что видели в предыдущем разделе, вы можете получить несогласованность, если не будет осторожны.
Так что любая реализация, которую вы создаёте, должна следовать нескольким стандартным правилам, а именно:

> **Rule 1: If you start with an unwrapped value, and then you wrap it (using `return`), then unwrap it (using `bind`), you should always get back the original unwrapped value.**

**Правило 1: Если вы начинаете с незавёрнутого значения, затем заворачиваете его (используя `return`) и снова разворачиваете (используя `bind`), вы должны получить оригинальное незавёрнутое значение.**

> This rule and the next are about not losing information as you wrap and unwrap the values.
> Obviously, a sensible thing to ask, and required for refactoring to work as expected.

Это правило, и следующее — про то, что нельзя терять информацию при заворачивании и разворачивании значений.
Очевиднно, это разумная вещь, которую стоит просить, она требуется, чтобы рефакторинг работал, как надо.

> In code, this would be expressed as something like this:

В коде это можно выразить следующим образом:

```fsharp
myworkflow {
    let originalUnwrapped = something

    // заворачиваем
    let wrapped = myworkflow { return originalUnwrapped }

    // разворачиваем
    let! newUnwrapped = wrapped

    // убеждаемся, что значения совпадают
    assertEqual newUnwrapped originalUnwrapped
    }
```

> **Rule 2: If you start with a wrapped value, and then you unwrap it (using `bind`), then wrap it (using `return`), you should always get back the original wrapped value.**

**Правило 2: Если вы начинаете с завёрнутого значения, затем разворачиваете его (используя `bind`) и снова заворачиваете (используя `return`), вы должны получить оригинальное завёрнутое значение.**

> This is the rule that the `stringInt` workflow broke above.
> As with rule 1, this should obviously be a requirement.

Именно это правило нарушает процесс `stringInt`, описанный ранее.

> In code, this would be expressed as something like this:

В коде это можно выразить следующим образом:

```fsharp
myworkflow {
    let originalWrapped = something

    let newWrapped = myworkflow {

        // разворачиваем
        let! unwrapped = originalWrapped

        // заворачиваем
        return unwrapped
        }

    // убеждаемся, что значения совпадают
    assertEqual newWrapped originalWrapped
    }
```

> **Rule 3: If you create a child workflow, it must produce the same result as if you had "inlined" the logic in the main workflow.**

**Правило 3: Если вы создали дочерний процесс, он должен вренуть то же значение, как если бы вы "встроили" логику в главный процесс.**

> This rule is required for composition to behave properly, and again, "extraction" refactoring will only work correctly if this is true.

Это правило нужно, чтобы композиция вела себя должным образом, и снова, рефакторинг "выделение" будет работать, только если это правило соблюдается.

> In general, you will get this for free if you follow some guidelines (which will be explained in a later post).

В целом, если вы будете следовать некоторым рекомендациям (которые будут объяснены в последующей статье), то получите это бесплатно.

> In code, this would be expressed as something like this:

В коде это можно выразить следующим образом:

```fsharp
// встроенный
let result1 = myworkflow {
    let! x = originalWrapped
    let! y = f x  // какая-то функция с x
    return! g y   // какая-то функция с y
    }

// используя дочерний процесс ("выделение" рефакторинг)
let result2 = myworkflow {
    let! y = myworkflow {
        let! x = originalWrapped
        return! f x // какая-то функция с x
        }
    return! g y     // какая-то функция с y
    }

// убеждаемся, что значения совпадают
assertEqual result1 result2
```

> ## Lists as wrapper types

## Списки как типы-обёртка

> I said earlier that types like `List<T>` or `IEnumerable<T>` can be used as wrapper types.
> But how can this be?
> There is no one-to-one correspondence between the wrapper type and the unwrapped type!

Ранее я утверждал, что типы вроде `List<T>` или `IEnumerable<T>` можно использовать в качестве типов-обёрток.
Но как это возможно?
Оказывается, что нет соответствия между завёрнутым и развёрнутым типом.

> This is where the "wrapper type" analogy becomes a bit misleading.
> Instead, let's go back to thinking of `bind` as a way of connecting the output of one expression with the input of another.

Этот тот случай, когда аналогия с "типом-обёрткой" начинает немного вводить в заблуждение.
Лучше вернёмся назад к методу `bind` как способу соединить выход одного выражения с входом другого.

> As we have seen, the `bind` function "unwraps" the type, and applies the continuation function to the unwrapped value.
> But there is nothing in the definition that says that there has to be only *one* unwrapped value.
> There is no reason that we can't apply the continuation function to each item of the list in turn.

Как мы видели, функция `bind` "разворачивает" тип и применяет функцию продолжения к развёрнутому значению.
Но ничто в определении не говорит о том, что там должно быть только *одно* развёрнутое значение.
Нет причин, по которым мы не можем применить функцию-продолжение к каждому элементу списка по очереди.

> In other words, we should be able to write a `bind` that takes a list and a continuation function, where the continuation function processes one element at a time, like this:

Иначе говоря, мы должны смочь написать `bind`, который принимает список и функцию-продолжение, где фукнция-продолжение обрабатывает по одному элементу за раз, как здесь:

```fsharp
bind( [1;2;3], fun elem -> // выражение с одним элементом )
```

> And with this concept, we should be able to chain some binds together like this:

И с таким пониманием, мы должны смочь объединять вызовы `bind` в цеочку, как здесь:

```fsharp
let add =
    bind( [1;2;3], fun elem1 ->
    bind( [10;11;12], fun elem2 ->
        elem1 + elem2
    ))
```

> But we've missed something important.
> The continuation function passed into `bind` is required to have a certain signature.
> It takes an unwrapped type, but it produces a *wrapped* type.

Но мы упустили кое-что важное.
Функция-продолжение, передаваемая в `bind`, обязана иметь определенную сигнатуру.
Она принимает развёрнутый тип, но возвращает *завёрнутый* тип.

> In other words, the continuation function must *always create a new list* as its result.

Иначе говоря, фукнция-продолжение в качестве результат должна *всегда возвращать новый список*.

```fsharp
bind( [1;2;3], fun elem -> // выражение с одним элементом, возвращающее список )
```

> And the chained example would have to be written like this, with the `elem1 + elem2` result turned into a list:

И пример с цепочкой вызовов нужно записать таким образом, чтобы результат `elem1 + elem2` помещался в список.

```fsharp
let add =
    bind( [1;2;3], fun elem1 ->
    bind( [10;11;12], fun elem2 ->
        [elem1 + elem2] // список!
    ))
```

> So the logic for our bind method now looks like this:

Так что логика для нашего метода `bind` сейчас выглядит следующим образом:

```fsharp
let bind(list,f) =
    // 1) к каждому элементу списка применить f
    // 2) f вернёт список (как того требует сигнатура)
    // 3) результатом будет список списков
```

> We have another issue now. `Bind` itself must produce a wrapped type, which means that the "list of lists" is no good.
> We need to turn them back into a simple "one-level" list.

Теперь у нас другая задача. `Bind` должен возвращать тип-обёртку, что означает, что "список списков" нам не подходит.
Мы должны превратить его обратно в простой "одноуровневый" список.

> But that is easy enough -- there is a list module function that does just that, called `concat`.

Но это довольно просто — в модуле `List` есть функция, которая именно это и делает, она называется `concat`.

> So putting it together, we have this:

Складывая всё вместе, мы получаем:

```fsharp
let bind(list,f) =
    list
    |> List.map f
    |> List.concat

let added =
    bind( [1;2;3], fun elem1 ->
    bind( [10;11;12], fun elem2 ->
//       elem1 + elem2    // неправильно
        [elem1 + elem2]   // правильно: возвращаем список
    ))
```

> Now that we understand how the `bind` works on its own, we can create a "list workflow".

Теперь, когда мы понимаем, как `bind` работает само по себе, мы можем создать "списочный процесс".

> * `Bind` applies the continuation function to each element of the passed in list, and then flattens the resulting list of lists into a one-level list.
>   `List.collect` is a library function that does exactly that.
> * `Return` converts from unwrapped to wrapped.
>   In this case, that just means wrapping a single element in a list.

* `Bind` применяет функцию-продолжение к каждому элементу переданного списка и затем превращает <!-- спрессовывает --> получившийся список списков в одноуровневый список.

```fsharp
type ListWorkflowBuilder() =

    member this.Bind(list, f) =
        list |> List.collect f

    member this.Return(x) =
        [x]

let listWorkflow = new ListWorkflowBuilder()
```

> Here is the workflow in use:

Вот процесс в работе:

```fsharp
let added =
    listWorkflow {
        let! i = [1;2;3]
        let! j = [10;11;12]
        return i+j
        }
printfn "суммы=%A" added

let multiplied =
    listWorkflow {
        let! i = [1;2;3]
        let! j = [10;11;12]
        return i*j
        }
printfn "произведения=%A" multiplied
```

> And the results show that every element in the first collection has been combined with every element in the second collection:

Результаты показывают, что каждый элемент из первой коллекции комбинируется с каждым элементом из второй коллекции:

```fsharp
val added : int list = [11; 12; 13; 12; 13; 14; 13; 14; 15]
val multiplied : int list = [10; 11; 12; 20; 22; 24; 30; 33; 36]
```

> That's quite amazing really.
> We have completely hidden the list enumeration logic, leaving just the workflow itself.

Это и правда достаточно удивительно.
Мы полностью спрятали логику перебора элементов списка, оставив только сам процесс.

> ### Syntactic sugar for "for"

### Синтаксический сахар для "for"

> If we treat lists and sequences as a special case, we can add some nice syntactic sugar to replace `let!` with something a bit more natural.

Если мы будем трактовать списки и последовательности как особый случай, мы сможем добавить немного вкусного синтаксического сахара, чтобы заменить `let!` чем-то более естественным.

> What we can do is replace the `let!` with a `for..in..do` expression:

Что мы можем сделать, это заменить `let!` на выражение `for..in..do`:

```fsharp
// версия с let!
let! i = [1;2;3] in [some expression]

// версия с for..in..do
for i in [1;2;3] do [some expression]
```

> Both variants mean exactly the same thing, they just look different.

Оба варианта означают в точности одно и то же, просто они выглядят по-разному.

> To enable the F# compiler to do this, we need to add a `For` method to our builder class.
> It generally has exactly the same implementation as the normal `Bind` method, but is required to accept a sequence type.

Чтобы разрешить компилятору F# такую обработку, мы должны добавить метод `For` в наш класс-строитель.

```fsharp
type ListWorkflowBuilder() =

    member this.Bind(list, f) =
        list |> List.collect f

    member this.Return(x) =
        [x]

    member this.For(list, f) =
        this.Bind(list, f)

let listWorkflow = new ListWorkflowBuilder()
```

> And here is how it is used:

И вот как это использовать:

```fsharp
let multiplied =
    listWorkflow {
        for i in [1;2;3] do
        for j in [10;11;12] do
        return i*j
        }
printfn "произведения=%A" multiplied
```

> ### LINQ and the "list workflow"

### LINQ и "процесс со типом-списком"

> Does the `for element in collection do` look familiar?
> It is very close to the `from element in collection ...` syntax used by LINQ.
> And indeed LINQ uses basically the same technique to convert from a query expression syntax like `from element in collection ...` to actual method calls behind the scenes.

Правда, конструкция `for element in collection do` выглядит знакомо?
По синтаксису она очень похожа на `from element in collection...`, которая используется в LINQ.
LINQ и в прямь в основе использует такую же технику "закулисной" конвертации из синтасиса выражений запросов вроде `from element in collection ...` в реальные вызовы методов.

> In F#, as we saw, the `bind` uses the `List.collect` function.
> The equivalent of `List.collect` in LINQ is the `SelectMany` extension method.
> And once you understand how `SelectMany`  works, you can implement the same kinds of queries yourself.
> Jon Skeet has written a [helpful blog post](http://codeblog.jonskeet.uk/2010/12/27/reimplementing-linq-to-objects-part-9-selectmany/) explaining this.

В F#, как мы видели, `bind` использует функцию `List.collect`.
Эквивалентом `List.collect` в LINQ является метод расширения `SelectMany`.
И как только вы поймёте, как работает `SelectMany`, вы можете реализовать похожий вид запросов самостоятельно.
Джон Скит написал [полезный пост в своём блоге](http://codeblog.jonskeet.uk/2010/12/27/reimplementing-linq-to-objects-part-9-selectmany/), который объясняет, как это сделать.

> ## The identity "wrapper type"

## Тот же самый "тип-обёртка"

> So we've seen a number of wrapper types in this post, and have said that *every* computation expression *must* have an associated wrapper type.

Что ж, в этом посте мы рассмотрели несколько типов-обёрток, и должны сказать, что *любое" вычислительное выражение *должно* иметь связанный тип-обёртку.

> But what about the logging example in the previous post? There was no wrapper type there.
> There was a `let!` that did things behind the scenes, but the input type was the same as the output type.
> The type was left unchanged.

Но что насчёт примера с логгированием из предыдущего поста? У него не было никакого типа-обёртки.
Там был `let!`, который делал разные штуки где-то там внутри, но входный тип был тем же самым, что и выходной.
Этот тип остался неизменным.

> The short answer to this is that you can treat any type as its own "wrapper".
> But there is another, deeper way to understand this.

Короткий ответ на этот вопрос заключает в том, что каждый тип можно трактовать, как свою собственную "обёртку".
Но есть другое, более глубокое объяснение, чтобы разобраться в этом. 

> Let's step back and consider what a wrapper type definition like `List<T>` really means.

Давайте сделаем шаг назад и разберёмся, что в дейтствительности означает определение типа-обёртки наподобие `List<T>`.

> If you have a type such as `List<T>`, it is in fact not a "real" type at all.
> `List<int>` is a real type, and `List<string>` is a real type.
> But `List<T>` on its own is incomplete.
> It is missing the parameter it needs to become a real type.

Если у вас есть такой тип, как `List<T>`, он, фактически, вообще не является "реальным" типом.
`List<int>` — реальный тип и `List<string>` — реальный тип.
Но `List<T>` сам по себе — неполный.
Есть отсутствующий параметр, который нужен, что он превратился в реальный тип.

> One way to think about `List<T>` is that it is a *function*, not a type.
> It is a function in the abstract world of types, rather than the concrete world of normal values, but just like any function it maps values to other values, except in this case, the input values are types (say `int` or `string`) and the output values are other types (`List<int>` and `List<string>`).
> And like any function it takes a parameter, in this case a "type parameter".
> Which is why the concept that .NET developers call "generics" is known as "[parametric polymorphism](http://en.wikipedia.org/wiki/Parametric_polymorphism)" in computer science terminology.

Один из способов думать о `List<T>` — не как о типе, а как о функции.
Это функция не из конкретного мира нормальных значений, а из абстрактного мира типов, но как и любая другая функция она отображает одни значения в другие значений, за тем исключением, что входные значения это типы (`int` и `string`) и выходные значения — тоже типы (`List<int>` и `List<string>`).
И, как у всякой функции, у неё есть параметре, в нашем случае это "параметр-тип".
Именно поэтому то, что программисты из мира .NET называют "обобщёнными типами", в научной терминологии называется [параметрическим полиморфизмом](http://en.wikipedia.org/wiki/Parametric_polymorphism).

> Once we grasp the concept of functions that generate one type from another type (called "type constructors"), we can see that what we really mean by a "wrapper type" is just a type constructor.

Как только мы ухватим концепцию функций, которые генерируют один тип из другого (они называются "конструкторами типа"), мы увидим, что в действительности "тип-обёртка" является таким конструктором, всего лишь.

> But if a "wrapper type" is just a function that maps one type to another type, surely a function that maps a type to the *same* type fits into this category?
> And indeed it does.
> The "identity" function for types fits our definition and can be used as a wrapper type for computation expressions.

Но если "тип-обёртка" — это всего лишь функция, которая отображает один тип в другой, наверняка функция, которая отображает тип сам в себя <!-- (она называется идентичная) -->, тоже попадает в эту категорию?

> Going back to some real code then, we can define the "identity workflow" as the simplest possible implementation of a workflow builder.

Возвращаясь к реальному коду, мы можем определить "идентичный процесс", как простейшую возможную реализацию строителя процесса.

```fsharp
type IdentityBuilder() =
    member this.Bind(m, f) = f m
    member this.Return(x) = x
    member this.ReturnFrom(x) = x

let identity = new IdentityBuilder()

let result = identity {
    let! x = 1
    let! y = 2
    return x + y
    }
```

> With this in place, you can see that the logging example discussed earlier is just the identity workflow with some logging added in.

Теперь, зная всё это, вы можете считать пример с логгированием, обсуждаемый ранее, обычным идентичным процессом с дополнительной логикой логгирования внутри.

> ## Summary

## Заключение

> Another long post, and we covered a lot of topics, but I hope that the role of wrapper types is now clearer.
> We will see how the wrapper types can be used in practice when we come to look at common workflows such as the "writer workflow" and the "state workflow" later in this series.

Ещё один длинный пост, и мы разобрались с большим количеством тем, но я надеюсь, что роль типов-обёрток теперь ясна.
Мы увидим, как использовать типы-обёртки на практике, когда доберёмся до общих процессов. таких как "процесс-писатель" и "процесс с состоянием" далее в этой серии статей.

> Here's a summary of the points covered in this post:

Вот сводка основных тем, затронутых в этом посте:

> * A major use of computation expressions is to unwrap and rewrap values that are stored in some sort of wrapper type.
> * You can easily compose computation expressions, because the output of a `Return` can be fed to the input of a `Bind`.
> * Every computation expression *must* have an associated wrapper type.
> * Any type with a generic parameter can be used as a wrapper type, even lists.
> * When creating workflows, you should ensure that your implementation conforms to the three sensible rules about wrapping and unwrapping and composition.

* Основное использование вычислительных выражений — разворачивать и заворачивать значения, которые хранятся в одном из типов-обёрток.
* Вы с лёгкостью можете компоновать вычислительные выражения, поскольку выход функции `Return` можно подать на вход функции `Bind`.
* Каждое вычислительное выражение *должно* быть ассоциировано с вычислительным типом.
* Любой тип с обобщённым параметром, может быть использован в качестве типа-обёртки, даже список.
* При создании процесса, вы должны убедиться, что ваша реализация соответствует трём разумным правилам, касающимся заворачивания, разворачивания и композиции.
