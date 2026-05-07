---
layout: post
title: "Проектирование с помощью типов: Нестроковые типы"
description: "Безопасная работа с целыми и датами"
date: 2013-01-18
nav: thinking-functionally
seriesId: "Проектирование с помощью типов"
seriesOrder: 7
categories: [Types, DDD]
---

В этом цикле мы познакомились с множеством примеров использования SCDU для заворачивания строк.

Нет никаких причин, почему эту технику нельзя применять к другим примитивным типам, такими как числа и даты.
Рассмотрим несколько примеров.

## Объединения с одним вариантом

В большинстве случаев нам не хотелось бы случайно перепутать различные целые числа.
Объекты предметной области могут иметь одно и то же представление, однако, путать их не следует.

Например, у вас могут быть целые числа `OrderId` и `CustomerId`.
Но это не *настоящие* числа.
Например, вы не можете прибавить 42 к `CustomerId`.
И `CustomerId(42)` не равно `OrderId(42)`.
Их, фактически, вообще нельзя сравнивать.

Что ж, размеченные объединения спешат к нам на помощь.

```fsharp
type CustomerId = CustomerId of int
type OrderId = OrderId of int

let custId = CustomerId 42
let orderId = OrderId 42

// ошбка компилятора
printfn "равен ли заказчик заказу? %b" (custId = orderId)
```

Схожим образом вы можете избавиться от путаницы с датами, завернув их в тип.
(`DateTimeKind` пытается сделать что-то подобное, но не всегда надёжно.)

```fsharp
type LocalDttm = LocalDttm of System.DateTime
type UtcDttm = UtcDttm of System.DateTime
```

С такими типами мы можем быть уверены, что передаём в качестве параметра правильные дату и время.
Плюс этот код работает как документация.

```fsharp
let SetOrderDate (d:LocalDttm) =
    () // что-то делаем

let SetAuditTimestamp (d:UtcDttm) =
    () // что-то делаем
```

## Ограничения целых чисел

Также, как мы валидировали и ограничивали `String50` и `ZipCode`, можно валидировать и ограничивать целые числа.

Скажем, система управления запасами может требовать, чтобы некоторые числа всегда были неотрицательными.
Это можно гарантировать, определив тип `NonNegativeInt`.

```fsharp
module NonNegativeInt =
    type T = NonNegativeInt of int

    let create i =
        if (i >= 0 )
        then Some (NonNegativeInt i)
        else None

module InventoryManager =

    // пример использования NonNegativeInt
    let SetStockQuantity (i:NonNegativeInt.T) =
        // установить запас
        ()
```

## Встраивание бизнес-правил в тип

Ранее мы задавались вопросом, может ли имя состоять из 64К символов.
А можно ли на самом деле добавить в корзину 999999 позиций?

![Количество позиций](./AddToCart.png)

Надо ли создавать ограниченный тип для решения этой проблемы?
Посмотрим на реальный код.

Вот очень простая Корзина, использующая стандартный тип `int`, чтобы хранить количество.
При щелчке на нужные кнопки количество увеличивается или уменьшается на единицу.
Видите ли вы ошибку?

```fsharp
module ShoppingCartWithBug =

    let mutable itemQty = 1  // не повторяйте этот трюк дома!

    let incrementClicked() =
        itemQty <- itemQty + 1

    let decrementClicked() =
        itemQty <- itemQty - 1
```

Если нет, возможно, вам следует явно определить ограничения.

Вот та же самая Корзина с ограниченным типом для количества.
Видите ли вы ошибку сейчас?
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

Может быть вам кажется, что всё это слишком сложно для такой тривиальной проблемы.
Но если вы не хотите попасть в [сводку DailyWTF](https://en.wikipedia.org/wiki/The_Daily_WTF), возможно, стоит об этом задуматься.

{{< book_page_ddd >}}

## Ограничения дат

Вам не всегда нужны все возможные даты.
Где то не бывает дат раньше 01.01.1980, а где-то — позднее 01.01.2038 (я использовал 01.01.2038 в качестве максимальной даты, чтобы не писать, в каком порядке идут день и месяц, ведь в разных странах порядок различается).

Как и целые числа, даты тоже полезно ограничивать, чтобы выход за границы обнаруживался на этапе конструирования, а не позже.

```fsharp
type SafeDate = SafeDate of System.DateTime

let create dttm =
    let min = new System.DateTime(1980,1,1)
    let max = new System.DateTime(2038,1,1)
    if dttm < min || dttm > max
    then None
    else Some (SafeDate dttm)
```


## Типы-объединения и единицы измерения

Самое время спросить про [единицы измерения](/posts/units-of-measure/)?
Разве не они должны использоваться в таких сценариях?

Да и нет.
Единицы измерения действительно позволяют не путать числа различных типов.
Более того, они гораздо мощнее объединений, которые мы используем.

В то же время единицы измерения не инкапсулированы и не имеют ограничений.
Кто угодно может создать целое с единицей измерения `<kg>` без минимального и максимального значения.

Чаще всего случаев работают оба подхода.
Например, при использовании стандартной библиотеки .NET часто нужны тайм-ауты, но иногда они заданы в секундах, а иногда — в миллисекундах.
Мне трудно запомнить, где какие нужны.
И я определённо не хочу случайно поставить тайм-аут в 1000 секунд, там, где мне нужен тайм-аут в 1000 миллисекунд.

Чтобы не попасть впросак, я создаю отдельные типы для секунд и миллисекунд.

Вот так это делается с помощью размеченных объединений:

```fsharp
type TimeoutSecs = TimeoutSecs of int
type TimeoutMs = TimeoutMs of int

let toMs (TimeoutSecs secs)  =
    TimeoutMs (secs * 1000)

let toSecs (TimeoutMs ms) =
    TimeoutSecs (ms / 1000)

/// засыпаем на нужное количество миллисекунд
let sleep (TimeoutMs ms) =
    System.Threading.Thread.Sleep ms

/// тайм-аут на указанное число секунд
let commandTimeout (TimeoutSecs s) (cmd:System.Data.IDbCommand) =
    cmd.CommandTimeout <- s
```

А вот так — с помощью единиц измерения:

```fsharp
[<Measure>] type sec
[<Measure>] type ms

let toMs (secs:int<sec>) =
    secs * 1000<ms/sec>

let toSecs (ms:int<ms>) =
    ms / 1000<ms/sec>

/// засыпаем на нужное количество миллисекунд
let sleep (ms:int<ms>) =
    System.Threading.Thread.Sleep (ms * 1<_>)

/// тайм-аут на указанное число секунд
let commandTimeout (s:int<sec>) (cmd:System.Data.IDbCommand) =
    cmd.CommandTimeout <- (s * 1<_>)
```

Что лучше?

Если вам нужна арифметика (сложение, умножение и т. д.), удобнее подход с единицами измерения.
В противном случае SCDU предоставляют больше средств для написания защищённого кода.