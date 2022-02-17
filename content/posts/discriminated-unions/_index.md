---
layout: post

# title: "Discriminated Unions"
title: "Размеченные объединения"

# description: "Adding types together"
description: "Сложение типов друг с другом"

date: 2012-06-06
nav: fsharp-types
seriesId: "Understanding F# types"
seriesOrder: 6
categories: [Types]
---

> Tuples and records are examples of creating new types by "multiplying" existing types together.
> At the beginning of the series, I mentioned that the other way of creating new types was by
  "summing" existing types.
> What does this mean?

Кортежи и записи — это примеры создания новых типов путем «умножения» существующих типов. В начале цикла я упоминал, что другим способом создания новых типов является «сложение». Что это означает?  

> Well, let's say that we want to define a function that works with integers OR booleans, maybe to
  convert them into strings.
> But we want to be strict and not accept any other type (such as floats or strings).
> Here's a diagram of such as function:

Что ж, представим, что нам нужна функция, которая умеет конвертировать целые числа ИЛИ булевые значения в строку.
Нам бы хотелось быть строгими и не принимать в качестве параметра никаких других типов, например, чисел с плавающей точкой или строк. Вот диаграмма такой функции:

![функция конвертации из целого или булевого значения](./fun_int_union_bool.png)

> How could we represent the domain of this function?

Как бы мы описали область значений такой функции?

> What we need is a type that represents all possible integers PLUS all possible booleans.

Нам нужен тип, представляющий все возможные целые числа ПЛЮС все возможные булевые значения.

![целое ПЛЮС булевое](./int_union_bool.png)

> In other words, a "sum" type.

Другими словами, тип «сумма».

> In this case the new type is the "sum" of the integer type plus the boolean type.

В нашем случае, новый тип — это «сумма» целого типа и булевого типа. 

> In F#, a sum type is called a "discriminated union" type.
> Each component type (called a *union case*) must be tagged with a label (called a *case
  identifier* or *tag*) so that they can be told apart ("discriminated").
> The labels can be any identifier you like, but must start with an uppercase letter.

В F# тип «сумма» называется "размеченным объединением" (discriminated union).
У каждого компонента — "варианта объединения" (union case) — должна быть своя метка, которую называют "идентификатором варианта" (case identifier) или "ярлыком" (tag). Метки нужны, чтобы отличать варианты друг от друга. Слово *discriminated* в названии как раз и означает *отличимые* или *различимые*. Метки могут быть любыми идентификаторами, какими захотите, но должны начинаться с заглавной буквы.

> Here's how we might define the type above:

Вот как мы можем определить тип, о котором писали выше:

```fsharp
type IntOrBool =
  | I of int
  | B of bool
```

> The "I" and the "B" are just arbitrary labels; we could have used any other labels that were
  meaningful.

Здесь метки — это "I" и "B". Мы могли бы дать им другие, более осмысленные имена.

> For small types, we can put the definition on one line:

Если у нашего типа мало вариантов, мы можем определить его в одну строку:

```fsharp
type IntOrBool = I of int | B of bool
```

> The component types can be any other type you like, including tuples, records, other union types,
  and so on.

Типы компонентов могут быть любыми типами какие вам могут понадобиться: кортежами, записями, другими объединениями и так далее.

```fsharp
type Person = {first:string; last:string}  // определяем тип записи
type IntOrBool = I of int | B of bool

type MixedType =
  | Tup of int * int  // кортеж
  | P of Person       // используем тип записи, определенный выше
  | L of int list     // список целых чисел
  | U of IntOrBool    // используем тип объединения, определенный выше
```

> You can even have types that are recursive, that is, they refer to themselves.

Вы даже можете создавать рекурсивные типы, то есть такие типы, которые ссылаются сами на себя.

> This is typically how tree structures are defined.

Так, например, определяются древовидные структуры.

> Recursive types will be discussed in more detail shortly.

Ниже мы поговорим о рекурсивных типах подробнее.

> ### Sum types vs. C++ unions and VB variants

### Чем тип суммы отличается от объединений C++ и вариантов VB

> At first glance, a sum type might seem similar to a union type in C++ or a variant type in Visual
  Basic, but there is a key difference.
> The union type in C++ is not type-safe and the data stored in the type can be accessed using any
  of the possible tags.
> An F# discriminated union type is safe, and the data can only be accessed one way.
> It really is helpful to think of it as a sum of two types (as shown in the diagram), rather than
  as just an overlay of data.

На первый взгляд, тип суммы кажется похожим на объединения C++ или варианты Visual Basic. Но здесь есть существенное отличие. Объединения в C++ небезопасны с точки зрения типа: данные, хранящиеся внутри, можно извлечь, используя любой ярлык. Размеченные объединения в F# безопасны: данные можно извлечь только одним способом.
О размеченных объединениях полезно думать именно как о сумме двух типов (см. диаграмму), а не как о значениях, которые разделяют одну и ту же память.

> ## Key points about union types

## Ключевые моменты в типах объединения

> Some key things to know about union types are:

Вот некоторые ключевые моменты, которые нужно знать о типах объединения:

>* 	The vertical bar is optional before the first component, so that the following definitions are all
    equivalent, as you can see by examining the output of the interactive window:

* Можно опускать вертикальную черту у самого первого компонента. Все эти определения эквиваленты, в чём можно убедиться в интерактивном окне:

```fsharp
type IntOrBool = I of int | B of bool     // без начальной черты
type IntOrBool = | I of int | B of bool   // с начальной чертой
type IntOrBool =
   | I of int
   | B of bool      // с начальной чертой на отдельных строках
```

> * The tags or labels must start with an uppercase letter. So the following will give an error:

* Названия ярлыков (меток) должны начинаться с заглавной буквы, поэтому такое определение выдаст ошибку:

```fsharp
type IntOrBool = int of int| bool of bool
//  ошибка FS0053: Названия ярлыков в размеченных объединениях
//                 должны начинаться с заглавной буквы
```

> * Other named types (such as `Person` or `IntOrBool`) must be pre-defined outside the union type.
>   You can't define them "inline" and write something like this:


* Другие именованные типы (такие как `Person` или `IntOrBool`) должны быть определены до
  размеченного объединения, их нельзя «встроить» их в само определение:

```fsharp
type MixedType =
  | P of  {first:string; last:string}  // ошибка
```

> or

или

```fsharp
type MixedType =
  | U of (I of int | B of bool)  // ошибка
```

> *	The labels can be any identifier, including the names of the component type themselves, which can be
    quite confusing if you are not expecting it.
>   For example, if the `Int32` and `Boolean` types (from the `System` namespace) were used instead, and
    the labels were named the same, we would have this perfectly valid definition:

*  Метки могут быть любыми идентификаторами, в том числе совпадать с названиями компонентов,
   что поначалу может немного сбивать с толку.
   Например, если бы типы `Int32` и `Boolean` (из пространства имен `System`) использовались бы с
   одноимёнными метками, мы получили бы такое, вполне допустимое определение:   

```fsharp
open System
type IntOrBool = Int32 of Int32 | Boolean of Boolean
```

> This "duplicate naming" style is actually quite common, because it documents exactly what the component types are.

Этот стиль «дублирующего именования» встречается довольно часто, так как он документирует типы компонентов.

{{< book_page_pdf >}}

> ## Constructing a value of a union type

## Конструирование значений типа объединения

> To create a value of a union type, you use a "constructor" that refers to only one of the possible union cases.
> The constructor then follows the form of the definition, using the case label as if it were a function.
> In the `IntOrBool` example, you would write:

Чтобы создать значение типа объединения, используйте «конструкторы», по одному для каждого варианта. По форме конструктор — это функция, чьё имя совпадает с меткой варианта.
В примере с `IntOrBool` вы могли бы написать:

```fsharp
type IntOrBool = I of int | B of bool

let i  = I 99    // используем конструктор "I"
// val i : IntOrBool = I 99

let b  = B true  // используем конструктор "B"
// val b : IntOrBool = B true
```

> The resulting value is printed out with the label along with the component type:

Результирующее значение выводится вместе с меткой сразу за типом компонента:

```fsharp
val [имя значения] : [тип]     = [метка] [печать типа компонента]
val i              : IntOrBool = I       99
val b              : IntOrBool = B       true
```

> If the case constructor has more than one "parameter", you construct it in the same way that you would call a function:

Также, как и функция, конструктор вызывается и тогда, когда у него несколько параметров:

```fsharp
type Person = {first:string; last:string}

type MixedType =
  | Tup of int * int
  | P of Person

let myTup  = Tup (2,99)    // используем конструктор "Tup"
// val myTup : MixedType = Tup (2,99)

let myP  = P {first="Al"; last="Jones"} // используем конструктор "P"
// val myP : MixedType = P {first = "Al";last = "Jones";}
```

> The case constructors for union types are normal functions, so you can use them anywhere a function is expected.
> For example, in `List.map`:

Конструкторы вариантов — это обычные функции, которые вы можете использовать их везде, где ожидаются функции.
Например, в `List.map`:

```fsharp
type C = Circle of int | Rectangle of int * int

[1..10]
|> List.map Circle

[1..10]
|> List.zip [21..30]
|> List.map Rectangle
```

> ### Naming conflicts

### Конфликты именования

> If a particular case has a unique name, then the type to construct will be unambiguous.

Если каждый вариант имеет свою уникальную метку, при вызове конструкторов не возникает никакой неоднозначности.

> But what happens if you have two types which have cases with the same labels?

Но что случится, если у вас есть два типа с одинаковыми метками?

```fsharp
type IntOrBool1 = I of int | B of bool
type IntOrBool2 = I of int | B of bool
```

> In this case, the last one defined is generally used:

В такой ситуации общим правилом является использование последнего определения:

```fsharp
let x = I 99                // val x : IntOrBool2 = I 99
```

> But it is much better to explicitly qualify the type, as shown:

Но ещё лучше явно указывать тип, как показано ниже:

```fsharp
let x1 = IntOrBool1.I 99    // val x1 : IntOrBool1 = I 99
let x2 = IntOrBool2.B true  // val x2 : IntOrBool2 = B true
```

> And if the types come from different modules, you can use the module name as well:

Если типы определены в разных модулях, вы также можете использовать и имя модуля:

```fsharp
module Module1 =
  type IntOrBool = I of int | B of bool

module Module2 =
  type IntOrBool = I of int | B of bool

module Module3 =
  let x = Module1.IntOrBool.I 99 // val x : Module1.IntOrBool = I 99
```

> ### Matching on union types

### Сопоставление типов объединения

> For tuples and records, we have seen that "deconstructing" a value uses the same model as constructing it.
> This is also true for union types, but we have a complication: which case should we deconstruct?

У кортежей и записей сопоставление с образцом выглядит точно также, как и их создание.
Это справедливо и для типов объединения. Правда, здесь у нас возникает сложность, потому типы объединения
имеют несколько конструкторов.

> This is exactly what the "match" expression is designed for.
> As you should now realize, the match expression syntax has parallels to how a union type is defined.

Выражение "match" разработано как раз для этого.
Как вы могли бы догадаться, синтаксис сопоставления похож на синтаксис конструирования.

```fsharp
// определение типа объединения
type MixedType =
  | Tup of int * int
  | P of Person

// "деконструирование" типа объединения
let matcher x =
  match x with
  | Tup (x,y) ->
        printfn "Tuple matched with %i %i" x y
  | P {first=f; last=l} ->
        printfn "Person matched with %s %s" f l

let myTup = Tup (2,99)                 // используем конструктор "Tup"
matcher myTup

let myP = P {first="Al"; last="Jones"} // используем конструктор "P"
matcher myP
```

> Let's analyze what is going on here:

Давайте проанализируем, что здесь происходит:

> *	Each "branch" of the overall match expression is a pattern expression that is designed to match the
    corresponding case of the union type.
> *	The pattern starts with the tag for the particular case, and then the rest of the pattern deconstructs
    the type for that case in the usual way.
> *	The pattern is followed by an arrow "->" and then the code to execute.

* Каждая «ветвь» сопоставления — это выражение образца, которое выглядит также, как соответствюущий
  вариант объединения.
* Образец начинается с метки варианта. Остальная часть образца сопоставляется с типом, относящимуся
  к данному варианту.
* За образцом следует стрелка "->" и затем код для выполнения.

> ## Empty cases

## Пустые варианты (empty cases)

> The label for a union case does not have to have to have any type after it.
> The following are all valid union types:

Метка варианта объединения не обязательно должна иметь какой-то тип.
Все следующие определения допустимы:

```fsharp
type Directory =
  | Root                   // нет необходимости давать имя корню
  | Subdirectory of string // другие папки должны быть именованы

type Result =
  | Success                // нет необходимости в строке для успешного состояния
  | ErrorMessage of string // сообщение об ошибке необходимо
```

> If *all* the cases are empty, then we have an "enum style" union:

Если *все* варианты пусты, у нас получается объединение, похожее на перечисление (enum):

```fsharp
type Size = Small | Medium | Large
type Answer = Yes | No | Maybe
```

> Note that this "enum style" union is *not* the same as a true C# enum type, discussed later.

Обратите внимание, что здесь объединение всего лишь похоже на перечисление. Это не тип перечисления
из C#, который мы обсудим позже.

> To create an empty case, just use the label as a constructor without any parameters:

Чтобы создать пустой вариант, используйте метку как конструктор без каких-либо параметров:

```fsharp
let myDir1 = Root
let myDir2 = Subdirectory "bin"

let myResult1 = Success
let myResult2 = ErrorMessage "not found"

let mySize1 = Small
let mySize2 = Medium
```

{{< linktarget "single-case" >}}

> ## Single cases

## Единственные варианты (single cases)

> Sometimes it is useful to create union types with only one case.
> This might be seem useless, because you don't seem to be adding value.
> But in fact, this a very useful practice that can enforce type safety*.

Иногда полезно создавать типы объединения с единственным вариантом.
Это может показаться бесполезным, потому что вы, кажется, не добавляете ничего нового к существующему значению.
Но на самом деле, это очень полезная практика, которая помогает обеспечить типобезопасноть*. 

{{<footnote "*">}}
> And in a future series we'll see that, in conjunction with module signatures, single case unions can also
  help with data hiding and capability based security.

И в следующих статьях мы увидим, что вместе с сигнатурами модулей, одновариантные объединения помогают в сокрытии данных и безопасности на основе полномочий.
{{</footnote>}}

> For example, let's say that we have customer ids and order ids which are both represented by integers, but
  that they should never be assigned to each other.

Представим, в качестве примера, что у вас есть идентификаторы заказчиков и идентификаторы заказов. И то,
и другое — целые числа, так что мы можем их перепутать, а нам бы этого не хотелось.

> As we saw before, a type alias approach will not work, because an alias is just a synonym and doesn't create
  a distinct type.
> Here's how you might try to do it with aliases:

Мы уже знаем, что подход с созданием псевдонима типа не работает, потому что псевдоним  — всего лишь
синоним и он не создает отдельный тип. 
Вот как вы можете попытаться решить вопрос с помощью псевдонимов:

```fsharp
type CustomerId = int   // определяем псевдоним типа
type OrderId = int      // определяем другой псевдоним типа

let printOrderId (orderId:OrderId) =
   printfn "The orderId is %i" orderId

// пробуем
let custId = 1          // создаем идентификатор заказчика
printOrderId custId     // не получилось!
```

> But even though I explicitly annotated the `orderId` parameter to be of type `OrderId`, I can't ensure that customer ids are not accidentally passed in.

Даже если я явно указал, что параметр `orderId` имеет тип `OrderId`, я не могу гарантировать, что в функцию случайно не попадёт идентификатор заказчика.

> On the other hand, if we create simple union types, we can easily enforce the type distinctions.

А вот с помощью одиночного объединения мы легко обеспечиваем различение типов. 

```fsharp
type CustomerId = CustomerId of int   // определяем тип объединения
type OrderId = OrderId of int         // определяем другой тип объединения

let printOrderId (OrderId orderId) =  // деконструируем в параметре
   printfn "The orderId is %i" orderId

//пробуем
let custId = CustomerId 1             // создаем идентификатор заказчика
printOrderId custId                   // Хорошо! Получаем ошибку компиляции.
```

> This approach is feasible in C# and Java as well, but is rarely used because of the overhead of creating and
  managing the special classes for each type. 
> In F# this approach is lightweight and therefore quite common.

Такой подход возможен и в C#, и Java, но используется он редко, потому что требует дополнительной работы по созданию и сопровождению классов для каждого типа.
А вот в F# не нужно писать много кода, так что этот подход встречается часто.

> A convenient thing about single case union types is you can pattern match directly against a value without having to use a full `match-with` expression.

Полезная подсказка про типы объединения с единственным вариантом — их можно сопоставлять со значением без использования полного выражения `match-with`.

```fsharp
// деконструируем в параметре
let printCustomerId (CustomerId customerIdInt) =
   printfn "The CustomerId is %i" customerIdInt

// или деконструируем явно через выражение let
let printCustomerId2 custId =
   let (CustomerId customerIdInt) = custId  // деконструируем здесь
   printfn "The CustomerId is %i" customerIdInt

// пробуем
let custId = CustomerId 1             // создаем идентификатор заказчика
printCustomerId custId
printCustomerId2 custId
```

> But a common "gotcha" is that in some cases, the pattern match must have parens around it, otherwise the compiler will think you are defining a function!

Здесь основная «загвоздка» в том, что иногда образец надо заключать в скобки, иначе компилятор подумает, что вы определяете функцию!

```fsharp
let custId = CustomerId 1
let (CustomerId customerIdInt) = custId  // Корректное сопоставление с образцом
let CustomerId customerIdInt = custId    // Неверно! Новая функция?
```

> Similarly, if you ever do need to create an enum-style union type with a single case, you will have to start the case with a vertical bar in the type definition; otherwise the compiler will think you are creating an alias.

Аналогично, если вам понадобится создать объединение с единственным пустым вариантом, в определении типа
надо будет поставить вертикальную черту, иначе компилятор подумает, что вы создаете псевдоним.

```fsharp
type TypeAlias = A     // псевдоним типа!
type SingleCase = | A   // тип объединения с единственным вариантов
```

> ## Union equality ##

## Сравнение объединений ##

> Like other core F# types, union types have an automatically defined equality operation: two unions are equal if they have the same type and the same case and the values for that case is equal.

Для типов объединений, также как и для других основных типы F#, компилятор автоматически создаёт функцию сравнения.
Два значения считаются равными, если они у них один и тот же тип, один и тот же вариант, и значения для этого варианта у них равны.

```fsharp
type Contact = Email of string | Phone of int

let email1 = Email "bob@example.com"
let email2 = Email "bob@example.com"

let areEqual = (email1=email2)
```

> ## Union representation ##

## Представление объединений ##

> Union types have a nice default string representation, and can be serialized easily.
> But unlike tuples, the ToString() representation is unhelpful.

У типов объединения приятное встроенное строковое представление, и они легко сериализуются.
Правда, в отличие от кортежей, представление, полученное через ToString(), бесполезно*.

```fsharp
type Contact = Email of string | Phone of int
let email = Email "bob@example.com"
printfn "%A" email    // хорошо
printfn "%O" email    // отвратительно!
```

{{<footnote "*">}}
Примечание переводчика: похоже, с момента написания статьи что-то изменилось, так как сейчас оба случая выводятся одинаково «хорошо».
{{</footnote>}}
