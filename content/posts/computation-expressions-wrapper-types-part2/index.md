---
layout: post
title: "Подробнее про типы-обёртки"
description: "Выясняем, что даже списки могут быть типами-обёртками"
date: 2013-01-24
nav: thinking-functionally
seriesId: "Вычислительные выражения"
seriesOrder: 5
---

В прошлом посте мы познакомились с концепцией "типов-обёрток" и с тем, как они связаны с вычислительными выражениями.
В этом посте мы разберёмся, какие типы можно использовать в качестве обёрток.

## Какие типы могут быть обёртками?

Если каждое вычислительное выражение должно быть ассоциировано с типом-обёрткой, какие типы можно использовать в качестве таких обёрток?
Есть ли у них какие-то особые ограничения?

Существует одно основное правило, которое гласит:

* **Любой обобщённый тип с параметром может быть использован в качестве типа-обёртки**

Например, мы видели, что можно использовать `Option<T>`, `DbResult<T>`, и подобные типы, как обёртки.

Что насчёт других обобщённых типов, таких как `List<T>` или `IEnumerable<T>`?
Это типы-коллекции, так что кажется странным, что в них можно *завернуть* какое-то *одно* значение.
На самом деле, они тоже *могут* быть обёртками, и чуть позже мы разберёмся, как это работает.

## Подойдут ли необобщённые типы-обёртки?

Можно ли создать тип-обёртку, у которого *нет* обобщённого параметра?

В одном из предыдущих примеров, мы попытались реализовать сложение строк вида `"1" + "2"`.
Нельзя ли в этом случае трактовать `string` как тип-обёртку над `int`?
Было бы здорово.

Давайте попробуем.
Мы можем использовать сигнатуры методов `Bind` и `Return` в качестве отправной точки.

* `Bind` получает кортеж.
  Первая часть кортежа — тип-обёртка (в нашем случае `string`), а вторая часть кортежа — функция, которая принимает незавёрнутый тип и превращает его в завёрнутый тип.
  В данном случае, её сигнатура будет `int -> string`.
* `Return` получает незавёрнутый тип (в нашем случае `int`) и превращает его в завёрнутый тип.
  И в этом случае, сигнатура `Return` будет `int -> string`.

Теперь, вот что у нас получается:

* Реализация "заворачивающей" функции с сигнатурой `int -> string` превращает любое число в строку.
  Это обычный метод "toString" типа `int`.
* Функция связывания должна развернуть значение из `string` в `int` и затем передать его в функцию.
  Для реализации мы можем использовать `int.Parse`.
* Что произойдёт, если функция связывания *не сможет* извлечь значени из строки, потому что оно не является корректным числом?
  Вы этом случае функция связывания все ещё *должна* вернуть тип-обёртку (`string`), так что мы можем просто вернуть что-то вроде строки "ошибка".
  
Вот реализация соответствующего класса-построителя:

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

Испытания:

```fsharp
let good =
    stringint {
        let! i = "42"
        let! j = "43"
        return i+j
        }
printfn "хороший результат=%s" good
```

А что произойдёт, если одна из строк не окажется числом?

```fsharp
let bad =
    stringint {
        let! i = "42"
        let! j = "xxx"
        return i+j
        }
printfn "плохой результат=%s" bad
```

Это действительно здорово — внутри нашего процесса мы можем обращаться со строками как с числами!

Но подождите, не всё так безоблачно.

Представим, что мы передадим значение в процесс, развернём его (с помощью `let!`) и затем немедленно завернём (с помощью `return`), не выполняя никаких других действий.
Что случится тогда?

```fsharp
let g1 = "99"
let g2 = stringint {
            let! i = g1
            return i
            }
printfn "g1=%s g2=%s" g1 g2
```

Никаких проблем.
Входное значение `g1` и выходное значение `g2` совпадают, как мы и ожидали.

Но что будет в случае ошибки?

```fsharp
let b1 = "xxx"
let b2 = stringint {
            let! i = b1
            return i
            }
printfn "b1=%s b2=%s" b1 b2
```

Здесь мы получили не то, что ожидали.
Входное значение `b1` и выходное значение `b2` *не* совпадают.
У нас появилось несоответствие.

Является ли это проблемой на практике?
Я не знаю.
Но я бы постарался избегать таких ситуаций, и попробовал бы что-нибудь другое, например, опциональный тип, который согласован во всех случаях.

## Правила для процесса, который использует тип-обёртку

А вот вам вопрос на засыпку!
Есть ли отличия между этими двумя фрагментами кода, и должен ли код вести себя по разному?

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

Ответ — нет, они не должны вести себя по разному.
Единственное отличие второго примера заключается в том, что значение `unwrapped` было выброшено в результате рефакторинга, и значение `wrapped` — возвращено напрямую.

Но, как мы только что видели в предыдущем разделе, вы можете получить несогласованность, если будете неосторожны.
Так что любая реализация, которую вы создаёте, должна следовать нескольким стандартным правилам, а именно:

**Правило 1: Если вы начинаете с незавёрнутого значения, затем заворачиваете его (используя `return`) и снова разворачиваете (используя `bind`), вы должны получить оригинальное незавёрнутое значение.**

И это правило, и следующее — про то, что нельзя терять информацию при заворачивании и разворачивании значений.
Очевидно, это разумное правило, которое должно соблюдаться, чтобы рефакторинг "выделить код" работал корректно.

Первое правило в виде кода:

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

**Правило 2: Если вы начинаете с завёрнутого значения, затем разворачиваете его (используя `bind`) и снова заворачиваете (используя `return`), вы должны получить оригинальное завёрнутое значение.**

Процесс `stringInt`, описанный ранее, нарушает именно это правило.

Второе правило в виде кода:

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

**Правило 3: Дочерний процесс должен возвращать тот же результат, как если бы он был "встроен" в основной процесс.**

Это правило требуется, чтобы композиция вела себя должным образом и чтобы рефакторинг "выделить код" продолжал работать.

Если вы будете следовать некоторым рекомендациям (про которые я расскажу в следующем посте), ваш код будет соответсвовать всем правилам автоматически.

А вот пример со встроенным процессом:

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

## Список как тип-обёртка

Ранее я говорил, что типы вроде `List<T>` или `IEnumerable<T>` можно использовать в качестве обёрток.
Но ведь в списке может хранится несколько значений, как же мы можем его "развернуть"?
Оказывается, между завёрнутыми и развёрнутыми типами не должно быть соответствия один-к-одному.

В данном случае аналогия с "обёрткой" немного вводит в заблуждение.
Вернёмся к методу `bind`, соединяющему выход одного выражения с входом другого.

Как мы видели, функция `bind` "разворачивает" тип и применяет функцию продолжения к развёрнутому значению.
Но ничто в определении не говорит о том, что там должно быть только *одно* развёрнутое значение.
Нет причин, по которым мы не можем применить функцию-продолжение к каждому элементу списка по очереди.

Мы всего лишь должны написать `bind` так, чтобы она принимала список и функцию-продолжение, а функция-продолжение обрабатывала по одному элементу за раз:

```fsharp
bind( [1;2;3], fun elem -> // выражение с одним элементом )
```

Следуя этой концепции, мы можем объединять вызовы `bind` в цепочку:

```fsharp
let add =
    bind( [1;2;3], fun elem1 ->
    bind( [10;11;12], fun elem2 ->
        elem1 + elem2
    ))
```

Однако, мы кое-что упустили.
Функция-продолжение, передаваемая в `bind`, обязана иметь определенную сигнатуру.
Она принимает развёрнутый тип, но возвращает *завёрнутый* тип.

Иначе говоря, фукнция-продолжение в качестве результат должна *всегда возвращать новый список*.

```fsharp
bind( [1;2;3], fun elem -> // выражение с одним элементом, возвращающее список )
```

Следовательно, пример с цепочкой вызовов нужно переписать так, чтобы результат `elem1 + elem2` помещался в список.

```fsharp
let add =
    bind( [1;2;3], fun elem1 ->
    bind( [10;11;12], fun elem2 ->
        [elem1 + elem2] // список!
    ))
```

Так что логика для нашего метода `bind` сейчас выглядит следующим образом:

```fsharp
let bind(list,f) =
    // 1) к каждому элементу списка применить f
    // 2) f вернёт список (как того требует сигнатура)
    // 3) результатом будет список списков
```

Теперь у нас другая задача. `Bind` должен возвращать тип-обёртку, а означает, что "список списков" в качестве результата нам не подходит.
Мы должны превратить его обратно в плоский "одноуровневый" список.

Это довольно просто — в модуле `List` есть функция, которая именно это и делает, она называется `concat`.

Сложив всё вместе, получим:

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

Теперь, когда мы понимаем, как работает `bind`, мы можем создать "списочный процесс".

* `Bind` применяет функцию-продолжение к каждому элементу переданного списка и затем превращает получившийся список списков в плоский одноуровневый список.
* `List.collect` — библиотечная функция, которую можно использовать вместо связки `List.map` и `List.concat`.
* `Return` превращает развёрнутое значение в завёрнутое.
  В нашем случае, он просто помещает отдельный элемент в список.

```fsharp
type ListWorkflowBuilder() =

    member this.Bind(list, f) =
        list |> List.collect f

    member this.Return(x) =
        [x]

let listWorkflow = new ListWorkflowBuilder()
```

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

Результаты показывают, что каждый элемент из первой коллекции комбинируется с каждым элементом из второй коллекции:

```fsharp
val added : int list = [11; 12; 13; 12; 13; 14; 13; 14; 15]
val multiplied : int list = [10; 11; 12; 20; 22; 24; 30; 33; 36]
```

Это и правда достаточно удивительно.
Мы полностью спрятали логику перебора элементов списка, оставив только сам процесс.

### Синтаксический сахар для "for"

Трактуя списки и последовательности как особый случай, мы можем добавить немного синтаксического сахара, чтобы заменить `let!` чем-то более естественным.

Например, мы можем заменить `let!` на выражение `for..in..do`:

```fsharp
// версия с let!
let! i = [1;2;3] in [some expression]

// версия с for..in..do
for i in [1;2;3] do [some expression]
```

Оба варианта означают в точности одно и то же, только выглядят по-разному.

Чтобы разрешить компилятору F# такую обработку, мы должны добавить метод `For` в наш класс-построитель.
В общем случае он делает то же, что и `Bind`, но традиционно используется с типами, хранящими последовательности.

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

Вот пример использования:

```fsharp
let multiplied =
    listWorkflow {
        for i in [1;2;3] do
        for j in [10;11;12] do
        return i*j
        }
printfn "произведения=%A" multiplied
```

### LINQ и "процесс с типом-списком"

Правда, конструкция `for element in collection do` выглядит знакомо?
По синтаксису она очень похожа на `from element in collection...`, которая используется в LINQ.
В основе LINQ и правда лежит такая же техника "закулисной" конвертации из синтасиса наподобие `from element in collection ...` в реальные вызовы методов.

В F#, как мы видели, `bind` вызывает функцию `List.collect`.
Эквивалентом `List.collect` в LINQ является метод расширения `SelectMany`.
Как только вы поймёте, как работает `SelectMany`, вы можете реализовать подобный вид запросов самостоятельно.
Джон Скит написал [полезный пост в своём блоге](http://codeblog.jonskeet.uk/2010/12/27/reimplementing-linq-to-objects-part-9-selectmany/), который объясняет, как это сделать.

## Идентичный "тип-обёртка"

К настоящему моменту, мы рассмотрели несколько типов-обёрток, и можем сказать, что *любое* вычислительное выражение *должно* иметь связанный тип-обёртку.

Но как быть с логгированием из предыдущего поста? У него не было никакого типа-обёртки.
Там был `let!`, который где-то внутри делал разные штуки, но входный тип был тем же самым, что и выходной.
Иными словами, завёрнутый тип был идентичен незавёрнутому типу.

Короткий ответ на этот вопрос заключает в том, что каждый тип можно трактовать, как свою собственную "обёртку".
Но есть и другое, более глубокое объяснение.

Давайте вернёмся назад и разберёмся, что в действительности означает определение типа-обёртки, такое как `List<T>`.

Фактически, `List<T>` вообще не является "реальным" типом.
`List<int>` — реальный тип и `List<string>` — реальный тип.
Но сам по себе `List<T>` — неполный.
У него есть параметр, и мы должны его предоставить, чтобы он стал реальным типом.

Можно думать о `List<T>` не как о типе, а как о функции.
Это функция не из конкретного мира обычных значений, а из абстрактного мира типов, но как и любая другая функция она отображает одни значения в другие.
Однако, её входные значения — это типы (`int` и `string`), и выходные значения — тоже типы (`List<int>` и `List<string>`).
Как и у всякой функции, у неё есть параметер, и это как раз "параметр-тип".
Кстати, именно поэтому то, что мы называем "обобщёнными типами", в научных кругах называют [параметрическим полиморфизмом](http://en.wikipedia.org/wiki/Parametric_polymorphism).

Если вы ухватили концепцию функций, которые генерируют один тип из другого (они называются "конструкторами типов"), вы понимаете, что "тип-обёртка" — как раз такой конструктор.

Но если "тип-обёртка" — это всего лишь функция, которая отображает один тип в другой, то наверняка функция, которая отображает тип сам в себя (традиционно она называется *identity*), тоже попадает в эту категорию?

В реальном коде мы можем определить "идентичный процесс", как простейшую возможную реализацию построителя.

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

Теперь, разобравшись во всём, можно считать пример с логгированием обычным идентичным процессом с дополнительной логикой логгирования.

## Итоги

Ещё один длинный пост.
Мы разобрались в большом количестве тем, так что, я надеюсь, теперь вы понимаете, что такое типы-обёртки.
Добравшись до общих процессов, таких как "процесс-писатель" или "процесс с состоянием", мы разберёмся, как использовать типы-обёртки на практике.
Об этом мы поговорим в одном из следующих постов.

Сводка основных тем, которые мы затронули:

* Основное использование вычислительных выражений — разворачивать и заворачивать значения, которые хранятся в типе-обёртке.
* Вычислительные выражения лекго компоновать, поскольку выход функции `Return` можно подать на вход функции `Bind`.
* Каждое вычислительное выражение *должно* быть ассоциировано с вычислительным типом.
* Любой тип с обобщённым параметром, даже список, может быть использован в качестве типа-обёртки.
* При создании процесса, вы должны убедиться, что ваша реализация соответствует трём разумным правилам, касающимся заворачивания, разворачивания и композиции.
