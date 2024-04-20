---
layout: post
title: "Реализуем вычислительные выражения: Combine"
description: "Как вернуть несколько значений за один раз"
date: 2013-01-26
nav: thinking-functionally
seriesId: "Computation Expressions"
seriesOrder: 7
---

В этом посте мы рассмотрим возврат из вычислительного выражения нескольких значений с помощью метода `Combine`.

{{<alertinfo>}}
Обратите внимание, что "построитель" в контексте вычислительных выражений — это не то же самое, что объектно-ориентированный паттерн "строитель", который исползуется для конструирования и валидации объектов.
{{</alertinfo>}}

## Как всё выглядит на данный момент...

На данный момент наш класс-построитель выражений выглядит так:

```fsharp
type TraceBuilder() =
    member this.Bind(m, f) =
        match m with
        | None ->
            printfn "Bind с None. Выход."
        | Some a ->
            printfn "Bind с Some(%A). Продолжение." a
        Option.bind f m

    member this.Return(x) =
        printfn "Return с незавёрнутым %A" x
        Some x

    member this.ReturnFrom(m) =
        printfn "Return с завёрнутым (%A)" m
        m

    member this.Zero() =
        printfn "Zero"
        None

    member this.Yield(x) =
        printfn "Yield с незавёрутым %A" x
        Some x

    member this.YieldFrom(m) =
        printfn "Yield с завёрнутым (%A)" m
        m

// создаём экземпляр процесса
let trace = new TraceBuilder()
```

И на данный момент этот класс прекрасно работает.
Но мы вот-вот столкнёмся с проблемой...

## Проблема с двумя 'yield'

Ранее мы видели как `yield` используется для возврата значений — также, как и `return`.

Обычно `yield` используется не один, а несколько раз, чтобы возвращать значения, например, при перечислении.
Давайте попробуем:

```fsharp
trace {
    yield 1
    yield 2
    } |> printfn "Результат yield и следующего за ним yield: %A"
```

Но — беда — мы получаем сообщение об ошибке:

```text
Конструкция данного элемента управления может использоваться только в том случае, если построитель вычислительного выражения определяет метод "Combine"
```

Если написать `return` вместо `yield`, мы получим такую же ошибку.

```fsharp
trace {
    return 1
    return 2
    } |> printfn "Результат return и следующего за ним return: %A"
```

Эта проблема возникает и в других контекстах.
Например, если мы хотим что-то сделать и затем вернуть результат, как в этом коде:

```fsharp
trace {
    if true then printfn "привет"
    return 1
    } |> printfn "Результат if и следующего за ним return: %A"
```

Мы получим ту же ошибку про отсутствующий метод 'Combine'.

## Понимание проблемы

Так что же здесь происходит?

Чтобы разобраться, давайте снова посмотрим, как вычислительные выражения устроены под капотом.
Мы знаем, что `return` и `yield` — это последний шаг в серии продолжений, как, например, здесь:

```fsharp
Bind(1,fun x ->
   Bind(2,fun y ->
     Bind(x + y,fun z ->
        Return(z)  // или Yield
```

Вы можете думать про `return` и `yield`, как про операторы "сброса" отступов.
Так что, когда мы выполняем `return/yield` и затем снова `return/yield`, мы генерируем вот такой код:

```fsharp
Bind(1,fun x ->
   Bind(2,fun y ->
     Bind(x + y,fun z ->
        Yield(z)
// начинаем новое выражение
Bind(3,fun w ->
   Bind(4,fun u ->
     Bind(w + u,fun v ->
        Yield(v)
```

Но в действительности он может быть упрощён до:

```fsharp
let value1 = какое-то выражение
let value2 = какое-то другое выражение
```

Иными словами, сейчас в нашем вычислительном выражении есть *два* значения.
И теперь возникает очевидный вопрос: как нам объединить эти два значения, чтобы получить общий результат для всего вычислительного выражения?

Это очень важный момент. **Return и yield *не* выполняют ранний выход из вычислительного выражения**.
Всё вычислительное выражение, вплоть до закрывающей фигурной скобки, *всегда* вычисляется и возвращает единственный результат.
Повторю ещё раз: все части выражения *будут вычислены в любом случае* и нет никакого способа этого избежать.
Если мы действительно хотим прервать вычисления, нам придётся написать немного кода и позже мы обсудим, как это сделать.

Но вернёмся к насущному вопросу.
У нас есть два результата выражений: как объединить их в один?

## Введение в "Combine"

Ответ: использовать метод `Combine`, который получает два *завёрнутых" значения и комбинирует из них новое завёрнутое значение.
И именно мы решаем, что означает это *комбинирование*.

В нашем случае, мы имеем дело с `int options`, так что одна из простейших реализаций, которая приходит в голову — это сложить числа.
Каждый параметр, это, конечно, `option` (завёрнутый тип), так что мы должны развернуть их и обработать четыре возможных варианта:

```fsharp
type TraceBuilder() =
    // другие члены, как раньше

    member this.Combine (a,b) =
        match a,b with
        | Some a', Some b' ->
            printfn "Combine %A с %A" a' b'
            Some (a' + b')
        | Some a', None ->
            printfn "Combine %A с None" a'
            Some a'
        | None, Some b' ->
            printfn "Combine None с %A" b'
            Some b'
        | None, None ->
            printfn "Combine None с None"
            None

// создаём новый экземпляр
let trace = new TraceBuilder()
```

Снова запускаем тестовый код:

```fsharp
trace {
    yield 1
    yield 2
    } |> printfn "Результат yield и следующего за ним yield: %A"
```

Теперь мы получаем другое сообщение об ошибке:

```text
Конструкция данного элемента управления может использоваться только в том случае, если построитель вычислительного выражения определяет метод "Delay"
```

> The `Delay` method is a hook that allows you to delay evaluation of a computation expression until needed -- we'll discuss this in detail very soon; but for now, let's create a default implementation:

`Delay` — это особый метод, который позволяет отложить вычисление выражения до тех пор, пока оно не понадобится.
Скоро мы обсудим его в деталях; но сейчас давайте просто создадим реализацию по-умолчанию:

```fsharp
type TraceBuilder() =
    // другие члены как раньше

    member this.Delay(f) =
        printfn "Delay"
        f()

// создаём новый экземпляр
let trace = new TraceBuilder()
```

Снова запустим тестовый код:

```fsharp
trace {
    yield 1
    yield 2
    } |> printfn "Результат yield и следующего за ним yield: %A"
```

И в конце концов мы получаем работающий код.

```text
Delay
Yield с незавёрнутым 1
Delay
Yield с незавёрнутым 2
Combine 1 с 2
Результат yield и следующего за ним yield: Some 3
```

Результат всего процесса — это сумма всех конструкций yield, то есть `Some 3`.

> If we have a "failure" in the workflow (e.g. a `None`), the second yield doesn't occur and the overall result is `Some 1` instead.

Если на первом шаге нашего процесса, результатом окажется "неудача" (т.е. `None`), второй оператор `yield`` не сработает и весь результат окажется равным `Some 1`.

```fsharp
trace {
    yield 1
    let! x = None
    yield 2
    } |> printfn "Результат yield и следующего за ним None: %A"
```

У нас может быть три оператора `yield` вместо двух:

```fsharp
trace {
    yield 1
    yield 2
    yield 3
    } |> printfn "Результат тройного yield: %A"
```

Результат, как вы могли бы предположить, равен `Some 6`.

Мы можем даже попытаться смешивать `yield` и `return` в одном коде.
Не смотря на разницу в синтаксисе, эффект от операторов один и тот же.

```fsharp
trace {
    yield 1
    return 2
    } |> printfn "Результ yield и следующего за ним return: %A"

trace {
    return 1
    return 2
    } |> printfn "Результат return и следующего за ним return: %A"
```

## Использование Combine для генерации последовательностей

Сложение чисел — не настоящая цель `yield`, хотя, возможно, вам понравится идея использовать оператор для конкатенации строк, как в `StringBuilder`.

Но обычно `yield` применяют для генерации последовательностей, и теперь, когда мы знаем, как работает `Combine`, мы можем расширить процесс "ListBuilder" (из предыдущей статьи) нужными методами.

* Метод `Combine` конкатенирует списки.
* Метод `Delay` использует реализацию по умолчанию.

Вот класс целиком:

```fsharp
type ListBuilder() =
    member this.Bind(m, f) =
        m |> List.collect f

    member this.Zero() =
        printfn "Zero"
        []

    member this.Yield(x) =
        printfn "Yield незавёрнутого %A в виде списка" x
        [x]

    member this.YieldFrom(m) =
        printfn "Yield завёрнутого (%A) непосредственно" m
        m

    member this.For(m,f) =
        printfn "For %A" m
        this.Bind(m,f)

    member this.Combine (a,b) =
        printfn "Combine %A и %A" a b
        List.concat [a;b]

    member this.Delay(f) =
        printfn "Delay"
        f()

// создаём экземпляр процесса
let listbuilder = new ListBuilder()
```

А вот его использование:

```fsharp
listbuilder {
    yield 1
    yield 2
    } |> printfn "Результат yield и следующего за ним yield: %A"

listbuilder {
    yield 1
    yield! [2;3]
    } |> printfn "Результат yield и следующего за ним yield! : %A"
```

Более сложный пример с циклом `for` и несколькими `yield`.

```fsharp
listbuilder {
    for i in ["красная";"синяя"] do
        yield i
        for j in ["шапка";"повязка"] do
            yield! [i + " " + j;"-"]
    } |> printfn "Результат for..in..do : %A"
```

И результат:

```text
["красная"; "красная шапка"; "-"; "красная повязка"; "-"; "синяя"; "синяя шапка"; "-"; "синяя повязка"; "-"]
```

Как видите, комбинируя `for..in..do` с `yield`, мы не слишком далеко ушли от встроенного синтаксиса выражений `seq`. Конечно, за исключением того, что `seq` ленивый.

Я настойчиво предлагаю вам поэкспериментировать с этими методами, чтобы вы разобрались, как они работают под капотом.
Как видно из примера выше, к использованию `yield` можно подойти творчески и генерировать с его помощью разного рода нерегулярные списки, а не только самые простые.

*Замечание: Если вам интересно, как работает `While`. Мы обсудим этот метод после того, как разберёмся с `Delay` в следующем посте*.

## Порядок обработки нескольких методов "Combine"

У метода `Combine` всего два параметра.
Что происходит, когда вы хотите скомбинировать больше, чем два значения?
Вот, например, четыре значения для комбинирования:

```fsharp
listbuilder {
    yield 1
    yield 2
    yield 3
    yield 4
    } |> printfn "Результат четырёх yield: %A"
```

Посмотрев на вывод программы, вы увидите, что значения скомбинированы попарно, что, в целом, естественно.

```text
Cobmine [3] и [4]
Cobmine [2] и [3; 4]
Cobmine [1] и [2; 3; 4]
Результат четырёх yield: [1; 2; 3; 4]
```

Тонкий, но важный момент заключается в том, что они комбинируется "в обратном порядке", от последнего знчения к первому.
Сначала "3" комбинируется с "4", затем результат комбинируется с "2", и так далее.

![Combine](./combine.png)

## Комбинирование не для последовательностей

Во втором из наших проблемных примеров, у нас не было последовательности; у нас были два отдельных выражения подряд.

```fsharp
trace {
    if true then printfn "привет"  // выражение 1
    return 1                       // выражение 2
    } |> printfn "Результат комбинирования: %A"
```

Как должны быть скомбинированы эти выражения?

Ответ зависит от того, какие концепции лежат в основании нашего вычислительного процесса.

### Реализуем "Combine" для процессов с "успехом" и "неудачей"

Если процесс основан на концепции "успешных" и "неуспешных" вычислений, то стандартный подход:

* Если первое выражение "успешно" (что бы это ни значило), использовать его значение.
* В противном случае использовать значение второго выражения.

В этом случае метод `Zero` также обычно возвращает значение "неудача".

Такой подход полезен для объединения нескольких выражений в цепочку "или иначе", где итоговым результатом будет результат первого успешного выражения..

```text
если (первое выражение)
или иначе (второе выражение)
или иначе (третье выражение)
```

Например, для процесса `maybe` общей практикой является возврат первого выражения, если у него есть результат, и второго в противом случае, как здесь:

```fsharp
type TraceBuilder() =
    // другие члены как раньше

    member this.Zero() =
        printfn "Zero"
        None  // неудача

    member this.Combine (a,b) =
        printfn "Combine %A и %A" a b
        match a with
        | Some _ -> a  // успех — берём значение a
        | None -> b    // неудача — вместо этого берём значение b

// создаём новый экземпляр
let trace = new TraceBuilder()
```

**Пример: Разбор**

Посмотрим, как работает наш пример с анализом строк:

```fsharp
type IntOrBool = I of int | B of bool

let parseInt s =
    match System.Int32.TryParse(s) with
    | true,i -> Some (I i)
    | false,_ -> None

let parseBool s =
    match System.Boolean.TryParse(s) with
    | true,i -> Some (B i)
    | false,_ -> None

trace {
    return! parseBool "42"  // неудача
    return! parseInt "42"
    } |> printfn "Результат разбора: %A"
```

Получаем результат:

```text
Some (I 42)
```

Как видите, результат первого `return!` равен `None` и он игнорируется.
Конечный результат — результат второго выражения `Some (I 42)`.

**Пример: Поиск в словаре**

Здесь мы попытаемся найти один и тот же ключ в нескольких словарях, и возвратим первое найденное значение:

```fsharp
let map1 = [ ("1","Один"); ("2","Два") ] |> Map.ofList
let map2 = [ ("A","Аня"); ("B","Вова") ] |> Map.ofList

trace {
    return! map1.TryFind "A"
    return! map2.TryFind "A"
    } |> printfn "Результат поиска в словаре: %A"
```

Результат:

```text
Результат поиска в словаре: "Аня"
```

Как видите, первый поиск возвращает `None` и игнорируется.
Результат всего процесса — это результат второго поиска.

Очевидно, эта техника хорошо работает, если вам надо вычислить последовательность выражений с возможным "неудачным" результатом.

### Реализуем "Combine" для процессов с последовательными шагами

Если в основе процесса лежит концепция последовального выполнения, то результатом всего вычисления будет значение последнего выражения, а все предыдущие шаги будут выполнятся только из-за их побочных эффектов.

В обычном F# мы могли бы написать:

```text
выполнить какое-то выражение
выполнить какое-то другое выражение
финальное выражение
```

Или, используя синтаксис с точкой-с-запятой:

```text
выполнить какое-то выражение, выполнить какое-то другое выражение, финальное выражение
```

В обычном F#, каждое выражение (кроме последнего) вычисляется в значение типа `unit`.

Эквивалентным подходом в вычислительных выражениях была бы трактовка каждого выражения (кроме последнего) как *завёрнутого* значения `unit` с передачей его в следующее выражение, пока вы не достигнете последнего выражения.

Это именно то, что делает связывание, поэтому простейшая реалиация — просто переиспользовать метод `Bind`.
Кроме того, чтобы этот подход работал, метод `Zero` должен возвращать завёрнутое значение `unit`.

```fsharp
type TraceBuilder() =
    // другие члены как раньше

    member this.Zero() =
        printfn "Zero"
        this.Return ()  // unit — это не None

    member this.Combine (a,b) =
        printfn "Combine %A и %A" a b
        this.Bind( a, fun ()-> b )

// создаём новый экземпляр
let trace = new TraceBuilder()
```

Отличие от обычного связывния в том, что функция-продолжение имеет параметр типа `unit` и вычисляет `b`.
Отсюда следует, что `a` имеет тип-обёртку с параметром `unit`, то есть в нашем случае `unit option`.

Вот пример последовательной обработки, где `Combine` работает также, как и `Bind`:

```fsharp
trace {
    if true then printfn "привет......."
    if false then printfn ".......мир"
    return 1
    } |> printfn "Результат последовательного комбинирования: %A"
```

 Вот трассировка нашей программы.
 Обратие внимание, что результат всего выражения совпадает с результатом последнего выражения в последовательности. Также, как и в обычном коде F#.

```text
привет.......
Zero
Return с незавёрнутым <null>
Zero
Return с незавёрнутым <null>
Return с незавёрнутым 1
Combine Some null и Some 1
Combine Some null и Some 1
Результат последовательного комбинирования: Some 1
```

### Реализуем "Combine" для процессов, которые строят структуры данных

Наконец, ещё один общий паттерн для процессов — построение структур данных.
Здесь метод `Combine` должен сливать две структуры данных, а метод `Zero` должен возвращать пустую структуру данных, если это вообще возможно.

В примере со "построителем списка" мы использовали именно эту концепцию.

## Рекомендации по смешиванию "Combine" и "Zero"

Мы видели две различные реализации `Combine` для опциональных типов.

* Первая использовала оциональные значения для индикации "успеха/неудачи", где первый успех "побеждает". В этом случае `Zero` должен возвращать `None`.
* Во втором случае речь шла о последовательном выполнении. В этом случае `Zero` определялся как `Some ()`.

Оба варианта прекрасно работают, но было ли это совпадением?
Существуюти ли какие-то рекомендации по правильной реализации `Combine` и `Zero`?

Во-первых, обратите внимание, что `Combine` *не* обязан давать тот же результат, если поменять параметры местами.
То есть `Combine(a,b)` не обязательно имеет то же значение, что и `Combine(b,a)`.
Построитель списков — хороший пример такой реализации.

С другой стороны, есть полезное правило, которое связывает `Zero` и `Combine`.

**Правило: `Combine(a,Zero)` должно быть равно `Combine(Zero,a)`, которое в свою очередь должно быть равно просто `a`.**

Используя аналогию из арифметики, вы можете думать о `Combine`, как о сложении (что не является плохой аналогией — поскольку метод действительно *складывает* два значения).
И `Zero` в этом случае — конечно, число ноль!
Так что правило можно переформулировать:

**Правило: `a + 0` должно быть равно `0 + a`, которое, в свою очередь, должно быть равно `a`, где `+` означает `Combine`, а `0` означает `Zero`.**

Посмотрев на первую реализацию ("успех/неудача") `Combine` для опциональных типов, вы увидите, что она действительно соответсвует этому правилу, точно также, как и вторая реализация ("связывание" с `Some()`).

А вот если мы объединим вторую реализацию `Combine`, с первой реализацией `Zero`, правило сложения больше не будет выполняться, что будет значить, что мы делаем что-то не то.

## "Сombine" без "Bind"

Как и в других случаях, вы не обязаны реализовывать методы, которые вам не нужны.
Так, для процесса, который просто выполняет операции одну за одной, можно создать класс-строитель с методами `Combine`, `Zero` и `Yield`; и без методов `Bind` и `Return`.

Вот пример минимальной работающей реализации:

```fsharp
type TraceBuilder() =

    member this.ReturnFrom(x) = x

    member this.Zero() = Some ()

    member this.Combine (a,b) =
        a |> Option.bind (fun ()-> b )

    member this.Delay(f) = f()

// создаём экземпляр процесса
let trace = new TraceBuilder()
```

Вот как он используется:

```fsharp
trace {
    if true then printfn "привет......."
    if false then printfn ".......мир"
    return! Some 1
    } |> printfn "Результат минимального комбинирования: %A"
```

Сходным образом, если у вас есть процесс, ориентированный на структуру данных, вы можете просто реализовать `Combine` и пару подручных методов.
Вот, например, минимальная реализация класса-строителя для списков.

```fsharp
type ListBuilder() =

    member this.Yield(x) = [x]

    member this.For(m,f) =
        m |> List.collect f

    member this.Combine (a,b) =
        List.concat [a;b]

    member this.Delay(f) = f()

// создаём экземпляр процесса
let listbuilder = new ListBuilder()
```

И даже с минимальной реализацией, мы можем писать код наподобии этого:

```fsharp
listbuilder {
    yield 1
    yield 2
    } |> printfn "Результат: %A"

listbuilder {
    for i in [1..5] do yield i + 2
    yield 42
    } |> printfn "Результат: %A"
```

## Обособленная функция "Combine"

В предыдущей статье мы видели, что функция связывания (`Bind`) иногда используется вне вычислительного процесса и её реализуют в виде оператора `>>=`.

Функция `Combine` также часто используется сама по себе.
В отличие от `Bind`, у неё нет стандартного оператора.
Он завист от того, что именно понимается под комбинированием.

Симметричные операции часто записываются как `++` или `<+>`.
"Лево-сторонняя" комбинация (которая вычисляет второе выражение, только если первое завершилось неудачей), иногда записывается как `<++`.

Вот пример с обособленной лево-сторонней комбинации опциональных значений, переделанный из поиска в словарях.

```fsharp
module StandaloneCombine =

    let combine a b =
        match a with
        | Some _ -> a  // успех — используем a
        | None -> b    // неудача — вместо a используем b

    // создаём инфиксную версию
    let ( <++ ) = combine

    let map1 = [ ("1","Один"); ("2","Два") ] |> Map.ofList
    let map2 = [ ("A","Аня"); ("B","Вова") ] |> Map.ofList

    let result =
        (map1.TryFind "A")
        <++ (map1.TryFind "B")
        <++ (map2.TryFind "A")
        <++ (map2.TryFind "B")
        |> printfn "Результат комбинирования опциональных значений: %A"
```

## Заключение

Что мы узнали о метод `Combine` из этой статьи?

* Вы должны реализовать `Combine` (и `Delay`) если вам нужно комбинировать или "складывать" больше чем одно завёрнутое значение в вычислительных выражениях.
* `Combine` комбинирует значения попарно, от последнего к первому.
* Нет универсальной реализации `Combine`, которая работает во всех случаях — её необходимо подогнать в соответсвии с особенностями процесса.
* Есть осмысленное правило, которое связывает `Combine` с `Zero`.
* `Combine` не требует, чтобы метод `Bind` был реализован.
* `Combine` можно использовать как обособленную функцию.

В следующем посте мы добавим логику для управления временем вычисления выражений и коснёмся ленивых вычислений.