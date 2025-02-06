---
layout: post
# title: "Designing with types: Non-string types"
title: "Проектирование с помощью типов: Не-строковые типы"
# description: "Working with integers and dates safely"
description: "Безопасная работа с целыми и датами"
date: 2013-01-18
nav: thinking-functionally
# seriesId: "Designing with types"
seriesId: "Проектирование с помощью типов"
seriesOrder: 7
categories: [Types, DDD]
---

> In this series we've seen a lot of uses of single case discriminated unions to wrap strings.

В этом цикле мы увидели множество примеров использования одновариантных размеченных объединений для заворачивания строк.

> There is no reason why you cannot use this technique with other primitive types, such as numbers and dates.
> Let's look a few examples.

Нет никаких причин, почему вы не можете использовать эту технику с другими примитивными типами, такими как числа и даты.
Давайте взглянем на несколько примеров.

> ## Single case unions

## Объединения с одним вариантом

> In many cases, we want to avoid accidentally mixing up different kinds of integers.
> Two domain objects may have the same representation (using integers) but they should never be confused.

В большинстве случаем нам бы не хотелось случайно перепутать различные виды целых чисел.
Два объекта предметной области могут иметь одно и то же представление (используя целые), но их никогда не следует путать.

> For example, you may have an `OrderId` and a `CustomerId`, both of which are stored as ints.
> But they are not *really* ints. You cannot add 42 to a `CustomerId`, for example.
> And `CustomerId(42)` is not equal to `OrderId(42)`.
> In fact, they should not even be allowed to be compared at all.

Например, у вас могут быть `OrderId` и `CustomerId`, оба их которых хранятся как типы.
Но это не *настоящие* целые. Например, вы не можете прибавить 42 к `CustomerId`.
И `CustomerId(42)` не равно `OrderId(42)`.
Фактически, их в принципе нельзя сравнивать.

> Types to the rescue, of course.

И, конечно, типы приходят на помощь.

```fsharp
type CustomerId = CustomerId of int
type OrderId = OrderId of int

let custId = CustomerId 42
let orderId = OrderId 42

// ошбка компилятора (compiler error)
printfn "равен ли заказчик заказу (cust is equal to order)? %b" (custId = orderId)
```

> Similarly, you might want avoid mixing up semantically different date values by wrapping them in a type.
> (`DateTimeKind` is an attempt at this, but not always reliable.)

Схожим образом, вы можете захотеть избежать путаницы с семантически различными значениями дат, путём заворачивания их в тип.
(`DateTimeKind` пытается сделать это, но не всегда надёжно.)

```fsharp
type LocalDttm = LocalDttm of System.DateTime
type UtcDttm = UtcDttm of System.DateTime
```

> With these types we can ensure that we always pass the right kind of datetime as parameters.
> Plus, it acts as documentation as well.

С такими типами мы можем быть уверены, что мы всегда передаём правильную дату и время в качестве параметра.
Плюс, это работает также работает, как документация.

```fsharp
let SetOrderDate (d:LocalDttm) =
    () // что-то делаем (do something)

let SetAuditTimestamp (d:UtcDttm) =
    () // что-то делаем (do something)
```

> ## Constraints on integers

## Ограничения для целых чисел

> Just as we had validation and constraints on types such as `String50` and `ZipCode`, we can use the same approach when we need to have constraints on integers.

Точно также, как мы делали валидацию и ограничения для таких типов, как `String50` и `ZipCode`, мы можем применить тот же подход, когда нам нужны ограничения для целых чисел.

> For example, an inventory management system or a shopping cart may require that certain types of number are always positive.
> You might ensure this by creating a `NonNegativeInt` type.

Например, система управления запасами или корзина могут требовать, чтобы определённые типы чисел были всегда положительными.
Вы можете обеспечить это, создав тип `NonNegativeInt`.

```fsharp
module NonNegativeInt =
    type T = NonNegativeInt of int

    let create i =
        if (i >= 0 )
        then Some (NonNegativeInt i)
        else None

module InventoryManager =

    // пример использования NonNegativeInt (example of NonNegativeInt in use)
    let SetStockQuantity (i:NonNegativeInt.T) =
        //set stock
        // установить запас
        ()
```

> ## Embedding business rules in the type

## Встраиваение бизнес-правил в тип

> Just as we wondered earlier whether first names could ever be 64K characters long, can you really add 999999 items to your shopping cart?

Точно также, как мы ранее задавались вопросом, могут ли имена в принципе иметь длину 64К символов, можете ли вы действительно добавить 999999 позиций в свою корзину?

> ![State transition diagram: Package Delivery](./AddToCart.png)
![Диаграмма переходов: Доставка посылок](./AddToCart.png)

> Is it worth trying to avoid this issue by using constrained types?
> Let's look at some real code.

Стоит ли пытаться избежать этой проблемы, используя ограниченные типы?
Давайте посмотрим на реальный код.

> Here is a very simple shopping cart manager using a standard `int` type for the quantity.
> The quantity is incremented or decremented when the related buttons are clicked.
> Can you find the obvious bug?

Вот очень простой менеджер корзины, использующий стандартный тип `int` для количества.
Количество увеличивается или уменьшается на единицу при щелчке на соответствующие кнопки.
Можете ли вы найти очевидную ошибку?

```fsharp
module ShoppingCartWithBug =

    let mutable itemQty = 1  // не повторяйте этот трюк дома! (don't do this at home!)

    let incrementClicked() =
        itemQty <- itemQty + 1

    let decrementClicked() =
        itemQty <- itemQty - 1
```

> If you can't quickly find the bug, perhaps you should consider making any constraints more explicit.

Если вы мне можете бытро найти ошибку, возможно, вам следует сделать некоторые ограничения более явными.

> Here is the same simple shopping cart manager using a typed quantity instead.
> Can you find the bug now?
> (Tip: paste the code into a F# script file and run it)

Вот тот же самый менеджер корзины с использованием типа для количества.
Можете ли вы найти ошибку сейчас?
(Подсказка: вставьте код в скриптовый файл F# и запустите.)

```fsharp
module ShoppingCartQty =

    type T = ShoppingCartQty of int

    let initialValue = ShoppingCartQty 1

    let create i =
        if (i > 0 && i < 100)
        then Some (ShoppingCartQty i)
        else None

    let increment t = create (t + 1)
    let decrement t = create (t - 1)

module ShoppingCartWithTypedQty =

    let mutable itemQty = ShoppingCartQty.initialValue

    let incrementClicked() =
        itemQty <- ShoppingCartQty.increment itemQty

    let decrementClicked() =
        itemQty <- ShoppingCartQty.decrement itemQty
```

> You might think this is overkill for such a trivial problem.
> But if you want to avoid being in the DailyWTF, it might be worth considering.

Вы можете подумать, что это слишком сложно для такой тривиальной проблемы.
Но если вы хотите избежать попадания в [сводку DailyWTF](https://en.wikipedia.org/wiki/The_Daily_WTF), возможно, стоит об этом подумать.

{{< book_page_ddd >}}

> ## Constraints on dates

## Ограничения для дат

> Not all systems can handle all possible dates.
> Some systems can only store dates going back to 1/1/1980, and some systems can only go into the future up to 2038 (I like to use 1/1/2038 as a max date to avoid US/UK issues with month/day order).

Не все системы могут обрабатывать все возможные даты.
Некоторые системы могут хранить даты не ранее 01.01.1980, а некоторые — не позднее 2038 года (мне нравится использовать 01.01.2038 в качестве максимальной даты, чтобы избегать вопроса о том, в каком порядке идут месяц и день в США и Великобритании <!-- страны можно убрать, вопрос более общий -->).

> As with integers, it might be useful to have constraints on the valid dates built into the type, so that any out of bound issues are dealt with at construction time rather than later on.

Точно также, как и с целыми, может быть полезно иметь ограничения на правильные даты, встроенные в тип, так что любой выход за границы приводил к проблеме на этапе конструирования, а не позже.

```fsharp
type SafeDate = SafeDate of System.DateTime

let create dttm =
    let min = new System.DateTime(1980,1,1)
    let max = new System.DateTime(2038,1,1)
    if dttm < min || dttm > max
    then None
    else Some (SafeDate dttm)
```


> ## Union types vs. units of measure

## Типы-объединения и единицы измерения

> You might be asking at this point: What about [units of measure](/posts/units-of-measure/)?
> Aren't they meant to be used for this purpose?

Сейчас самое время спросить: а что на счёт [единиц измерения](/posts/units-of-measure/)?
Разве не они должны использоваться для этих целей?

> Yes and no.
> Units of measure can indeed be used to avoid mixing up numeric values of different type, and are much more powerful than the single case unions we've been using.

Да и нет.
Единицы измерения действительно можно использовать, чтобы избежать путаницы с числовыми значениями различных типов, и они гораздо более мощные, чем одновариантные объединения, которые мы использовали.

> On the other hand, units of measure are not encapsulated and cannot have constraints.
> Anyone can create a int with unit of measure `<kg>` say, and there is no min or max value.

С другой стороны, единицы измерения не инкапсулированы и не могут иметь ограничений.
Скажем, кто угодно может создать целое с единицей измерения `<kg>`, у которого не будет минимального или максимального значения.

> In many cases, both approaches will work fine.
> For example, there are many parts of the .NET library that use timeouts, but sometimes the timeouts are set in seconds, and sometimes in milliseconds.
> I often have trouble remembering which is which.
> I definitely don't want to accidentally use a 1000 second timeout when I really meant a 1000 millisecond timeout.

В большинстве случаев, оба подхода прекрасно работают.
Например, есть много частей библиотеки .NET, которые используют тайм-ауты, но иногда тайм-ауты заданы в секундах, а иногда в миллисекундах.
Часто мне трудно запомнить, что есть что.
Я определённо не хочу случайно поставить тайм-аут в 1000 секунду, когда на самом деле мне нужен тайм-аут в 1000 миллисекунд.

> To avoid this scenario, I often like to create separate types for seconds and milliseconds.

Чтобы избежать такого сценария, мне часто нравится создать отдельные типы для секунд и миллисекунд.

> Here's a type based approach using single case unions:

Вот подоход, основанный на типах, использюущий одновариантные объединения.

```fsharp
type TimeoutSecs = TimeoutSecs of int
type TimeoutMs = TimeoutMs of int

let toMs (TimeoutSecs secs)  =
    TimeoutMs (secs * 1000)

let toSecs (TimeoutMs ms) =
    TimeoutSecs (ms / 1000)

/// sleep for a certain number of milliseconds
/// засыпаем на нужное количество миллисекунд
let sleep (TimeoutMs ms) =
    System.Threading.Thread.Sleep ms

/// timeout after a certain number of seconds
/// тайм-аут на указанное число секунд
let commandTimeout (TimeoutSecs s) (cmd:System.Data.IDbCommand) =
    cmd.CommandTimeout <- s
```

> And here's the same thing using units of measure:

А вот то же самое с использованием единиц измерения:

```fsharp
[<Measure>] type sec
[<Measure>] type ms

let toMs (secs:int<sec>) =
    secs * 1000<ms/sec>

let toSecs (ms:int<ms>) =
    ms / 1000<ms/sec>

/// sleep for a certain number of milliseconds
/// засыпаем на нужное количество миллисекунд
let sleep (ms:int<ms>) =
    System.Threading.Thread.Sleep (ms * 1<_>)

/// timeout after a certain number of seconds
/// тайм-аут на указанное число секунд
let commandTimeout (s:int<sec>) (cmd:System.Data.IDbCommand) =
    cmd.CommandTimeout <- (s * 1<_>)
```

> Which approach is better?

Какой подход лучше?

> If you are doing lots of arithmetic on them (adding, multiplying, etc) then the units of measure approach is much more convenient, but otherwise there is not much to choose between them.

Если вам нужно много арифметики с этими числами (сложение, умножение и т. д.), тогда более удобен подход с единицами измерения, в остальном же выбирать между ними не приходится. <!-- не очень ясно, кажется, лучше явно написать, что SCDU лучше -->
