---
layout: post

# title: "Discriminated Unions"

title: "Размеченные объединения"

# description: "Adding types together"

description: "Сложение типов друг с другом"
date: 2012-06-06
nav: fsharp-types
#seriesId: "Understanding F# types"
seriesId: "Понимание F# типов"
seriesOrder: 6
categories: [Types]
---


>Tuples and records are examples of creating new types by "multiplying" existing types together.
Кортежи и записи - примеры создания новых типов путем "умножения" существующих типов друг с другом.
>At the beginning of the series, I mentioned that the other way of creating new types was by "summing" existing types.
В начале серии я упомянул, что другим способом создания новых типов было "сложение" существующих типов.
>What does this mean?
Что это значит?  

>Well, let's say that we want to define a function that works with integers OR booleans, maybe to convert them into strings.
Хорошо, давайте предположим, что мы хотим определить функцию, которая работает с целыми числами ИЛИ булевыми значениями, может быть, для того, чтобы конвертировать их в строки.
>But we want to be strict and not accept any other type (such as floats or strings).
Но мы хотим быть строгими и не принимать любые другие типы (например, числа с плавающей запятой или строки).
>Here's a diagram of such as function:
Вот диаграмма такой функции:

![function from int union bool](./fun_int_union_bool.png)

>How could we represent the domain of this function?

Как мы можем представить область значений такой функции?

>What we need is a type that represents all possible integers PLUS all possible booleans.

Нам нужен тип, представляющий все возможные целые числа ПЛЮС все возможные булевые значения.

![int union bool](./int_union_bool.png)

>In other words, a "sum" type.

Другими словами, тип "сумма".

>In this case the new type is the "sum" of the integer type plus the boolean type.

В этом случае новый тип - это "сумма" типов целых чисел плюс булевых значений. 

>In F#, a sum type is called a "discriminated union" type.
В F# тип "сумма" называется типом "размеченного объединения" (англ. discriminated union, рус. различаемое объединение).
>Each component type (called a *union case*) must be tagged with a label (called a *case identifier* or *tag*) so that they can be told apart ("discriminated").
Каждому типу компонента (называемому "вариантом объединения" (англ. union case)) должна быть присвоена метка (называемая "идентификатором варианта" (англ. case identifier) или "ярлыком" (англ. tag)), чтобы их можно отличать друг от друга ("размеченные").
>The labels can be any identifier you like, but must start with an uppercase letter.
Метки могут быть любыми идентификаторами на ваше усмотрение, но должны начинаться с заглавной буквы.

>Here's how we might define the type above:

Здесь показано, как мы можем определить вышеупомянутый тип:

```fsharp
type IntOrBool =
  | I of int
  | B of bool
```

>The "I" and the "B" are just arbitrary labels; we could have used any other labels that were meaningful.

"I" и "B" - это просто произвольные метки; мы могли бы выбрать любые другие метки, которые несут больше смысла.

>For small types, we can put the definition on one line:

Для малых типов мы можем поместить определение в одной строке:

```fsharp
type IntOrBool = I of int | B of bool
```

>The component types can be any other type you like, including tuples, records, other union types, and so on.

Типы компонентов могут быть любыми другими типами по вашему усмотрению, включая кортежи, записи, другие типы объединений и так далее.

```fsharp
type Person = {first:string; last:string}  // определяем тип записи
type IntOrBool = I of int | B of bool

type MixedType =
  | Tup of int * int  // кортеж
  | P of Person       // используем тип записи, определенный выше
  | L of int list     // список целых чисел
  | U of IntOrBool    // используем тип объединения, определенный выше
```

>You can even have types that are recursive, that is, they refer to themselves.

Вы даже можете иметь рекурсивные типы, т.е. типы, ссылающиеся сами на себя.

>This is typically how tree structures are defined.

Это типичный способ определения древовидных структур.

>Recursive types will be discussed in more detail shortly.

Рекурсивные типы будут детально рассмотрены ниже.

>### Sum types vs. C++ unions and VB variants

### Типы суммы vs. C++ объединения и VB варианты.

>At first glance, a sum type might seem similar to a union type in C++ or a variant type in Visual Basic, but there is a key difference.
На первый взгляд, тип суммы может показаться похожим на тип объединения в C++ или тип варианта в Visual Basic, но есть ключевое отличие.
>The union type in C++ is not type-safe and the data stored in the type can be accessed using any of the possible tags.
Тип объединения в C++ не типобезопасный и данные, хранящиеся в таком типе, могут быть получены с использованием любых из возможных ярлыков.
>An F# discriminated union type is safe, and the data can only be accessed one way.
Тип размеченных объединений в F# безопасен, и данные могут быть получены только одним способом.
>It really is helpful to think of it as a sum of two types (as shown in the diagram), rather than as just an overlay of data.
Это действительно полезно - думать об объединении, как о сумме двух типов (как показано на диаграмме), а не как о совмещении данных.

>## Key points about union types

## Ключевые пункты в типах объединения

>Some key things to know about union types are:

Вот некоторые ключевые пункты, которые нужно знать о типах объединения:

>* 	The vertical bar is optional before the first component, so that the following definitions are all equivalent, as you can see by examining the output of the interactive window:

* Вертикальная черта необязательна перед первым компонентом, поэтому все следующие определения эквивалентны, как вы можете увидеть, проверив вывод интерактивного окна:

```fsharp
type IntOrBool = I of int | B of bool     // без начальной черты
type IntOrBool = | I of int | B of bool   // с начальной чертой
type IntOrBool =
   | I of int
   | B of bool      // с начальной чертой на отдельных строках
```

>* 	The tags or labels must start with an uppercase letter.

*  Ярлыки или метки должны начинаться с заглавной буквы. 

>So the following will give an error:

Так что следующее определение выдаст ошибку:

```fsharp
type IntOrBool = int of int| bool of bool
//  error FS0053: Discriminated union cases
//                must be uppercase identifiers
```
<!-- переводить ли текст ошибки? -->

>* 	Other named types (such as `Person` or `IntOrBool`) must be pre-defined outside the union type.
*  Другие именованные типы (такие как `Person` или `IntOrBool`) должны быть определены за пределами типа объединения.
>You can't define them "inline" and write something like this:
Вы не можете "встроить" их определение и написать что-то подобное:

```fsharp
type MixedType =
  | P of  {first:string; last:string}  // ошибка
```

>or

или

```fsharp
type MixedType =
  | U of (I of int | B of bool)  // ошибка
```

>* 	The labels can be any identifier, including the names of the component type themselves, which can be quite confusing if you are not expecting it.
*  Метки могут быть любым идентификатором, включая имена самих типов компонентов, которые могут немного сбивать с толку, если вы этого не ожидаете.
>For example, if the `Int32` and `Boolean` types (from the `System` namespace) were used instead, and the labels were named the same, we would have this perfectly valid definition:
Например, если бы использовались типы `Int32` и `Boolean` (из пространства имен `System`) и метки назывались также, мы бы получили такое полностью допустимое определение:   

```fsharp
open System
type IntOrBool = Int32 of Int32 | Boolean of Boolean
```

>This "duplicate naming" style is actually quite common, because it documents exactly what the component types are.

Такой стиль "дублирующего именования" на самом деле довольно распространен, потому что он точно документирует то, чем являются типы компонентов.

{{< book_page_pdf >}}

>## Constructing a value of a union type

## Конструирование значения типа объединения

>To create a value of a union type, you use a "constructor" that refers to only one of the possible union cases.
Чтобы создать значение типа объединения, вы используете "конструктор", который ссылается только на один из возможных вариантов объединения.
>The constructor then follows the form of the definition, using the case label as if it were a function.
Конструктор следует форме определения, используя метку варианта, как если бы это была функция.
>In the `IntOrBool` example, you would write:
В примере `IntOrBool` вы могли бы написать:

```fsharp
type IntOrBool = I of int | B of bool

let i  = I 99    // используем конструктор "I"
// val i : IntOrBool = I 99

let b  = B true  // используем конструктор "B"
// val b : IntOrBool = B true
```

>The resulting value is printed out with the label along with the component type:

Результирующее значение выводится с меткой вместе с типом компонента:

```fsharp
val [имя значения] : [тип]     = [метка] [печать типа компонента]
val i              : IntOrBool = I       99
val b              : IntOrBool = B       true
```

>If the case constructor has more than one "parameter", you construct it in the same way that you would call a function:

Если конструктор варианта имеет больше, чем один "параметр", вы вызываете его также, как вы вызывали бы функцию:

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

>The case constructors for union types are normal functions, so you can use them anywhere a function is expected.
Конструкторы варианта для типов объединения - обычные функции, так что вы можете использовать их везде, где ожидается функция.
>For example, in `List.map`:
Например, в `List.map`:

```fsharp
type C = Circle of int | Rectangle of int * int

[1..10]
|> List.map Circle

[1..10]
|> List.zip [21..30]
|> List.map Rectangle
```

>### Naming conflicts

### Конфликты именования

>If a particular case has a unique name, then the type to construct will be unambiguous.

Если отдельный вариант имеет уникальное имя, тогда создаваемый тип будет однозначным.

>But what happens if you have two types which have cases with the same labels?

Но что произойдет, если у вас есть два типа, которые имеют варианты с одинаковыми метками?

```fsharp
type IntOrBool1 = I of int | B of bool
type IntOrBool2 = I of int | B of bool
```

>In this case, the last one defined is generally used:

В такой ситуации общим правилом является использование последнего определения:

```fsharp
let x = I 99                // val x : IntOrBool2 = I 99
```

>But it is much better to explicitly qualify the type, as shown:

Но лучше всего явно указывать тип, как показано ниже:

```fsharp
let x1 = IntOrBool1.I 99    // val x1 : IntOrBool1 = I 99
let x2 = IntOrBool2.B true  // val x2 : IntOrBool2 = B true
```

>And if the types come from different modules, you can use the module name as well:

И если типы берутся из разных модулей, вы можете использовать также и имя модуля:

```fsharp
module Module1 =
  type IntOrBool = I of int | B of bool

module Module2 =
  type IntOrBool = I of int | B of bool

module Module3 =
  let x = Module1.IntOrBool.I 99 // val x : Module1.IntOrBool = I 99
```


>### Matching on union types

### Сопоставление типов объединения

>For tuples and records, we have seen that "deconstructing" a value uses the same model as constructing it.
Для кортежей и записей мы увидели, что "деконструирование" значения использует ту же модель, что и его создание.
>This is also true for union types, but we have a complication: which case should we deconstruct?
Это справедливо и для типов объединения, но мы имеем сложность: какой именно вариант должен быть деконструирован?

>This is exactly what the "match" expression is designed for.
Это именно то, для чего разработано выражение "match".
>As you should now realize, the match expression syntax has parallels to how a union type is defined.
Как вы должны были догадаться, синтаксис сопоставления имеет параллели с тем, как определён тип объединения.

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

>Let's analyze what is going on here:

Давайте проанализируем, что здесь происходит:

>* 	Each "branch" of the overall match expression is a pattern expression that is designed to match the corresponding case of the union type.

*  Каждая "ветвь" общего выражения сопоставления - это выражение образца, которое сконструировано, чтобы сопоставлять соответствующий вариант типа объединения.

>* 	The pattern starts with the tag for the particular case, and then the rest of the pattern deconstructs the type for that case in the usual way.

*  Образец начинается с ярлыка конкретного варианта, и затем остальная часть образца деконструирует тип для данного варианта обычным способом.

>* 	The pattern is followed by an arrow "->" and then the code to execute.

*  За образцом следует стрелка "->", а затем код для выполнения.

>## Empty cases

# Пустые варианты (англ. empty cases)<!-- пустые союзы -->

>The label for a union case does not have to have to have any type after it.
Метка варианта объединения не обязательно должна иметь какой-либо тип после нее.
>The following are all valid union types:
Все следующие типы объединения допустимы:

```fsharp
type Directory =
  | Root                   // нет необходимости давать имя корню
  | Subdirectory of string // другие папки должны быть именованы

type Result =
  | Success                // нет необходимости в строке для успешного состояния
  | ErrorMessage of string // сообщение об ошибке необходимо
```

>If *all* the cases are empty, then we have an "enum style" union:

Если *все* варианты пусты, то мы имеем объединение в "стиле перечисления":

```fsharp
type Size = Small | Medium | Large
type Answer = Yes | No | Maybe
```

>Note that this "enum style" union is *not* the same as a true C# enum type, discussed later.

Заметьте, что это объединение в "стиле перечисления" *не* то же самое, что тип перечисления в C#, обсуждаемый ниже.

>To create an empty case, just use the label as a constructor without any parameters:

Чтобы создать пустой вариант, просто используйте метку как конструктор без каких-либо параметров:

```fsharp
let myDir1 = Root
let myDir2 = Subdirectory "bin"

let myResult1 = Success
let myResult2 = ErrorMessage "not found"

let mySize1 = Small
let mySize2 = Medium
```

{{< linktarget "single-case" >}}

>## Single cases

## Одиночные варианты (англ. single cases)

>Sometimes it is useful to create union types with only one case.
Иногда полезно создавать типы объединения с единственным вариантом.
>This might be seem useless, because you don't seem to be adding value.
Это может показаться бесполезным, потому что вы, похоже, не добавляете значения.
>But in fact, this a very useful practice that can enforce type safety*.
Но на самом деле, это очень полезная практика, которая может обеспечить типобезопасноть*. 

{{<footnote "*">}}
>And in a future series we'll see that, in conjunction with module signatures, single case unions can also help with data hiding and capability based security.

И в следующих сериях мы увидим, что в дополнение к сигнатурам модулей одновариантные объединения могут также помочь с сокрытием данных и безопасностью на основе полномочий.

{{</footnote>}}

>For example, let's say that we have customer ids and order ids which are both represented by integers, but that they should never be assigned to each other.

Например, давайте предположим, у вас есть идентификаторы заказчиков и идентификаторы заказов, представленные целыми числами, но они никогда не должны быть присвоены друг другу.

>As we saw before, a type alias approach will not work, because an alias is just a synonym and doesn't create a distinct type.

Как мы видели до этого, подход с присвоением типу псевдонима не будет работать, потому что псевдоним - это просто синоним, и он не создает отдельный тип. 

>Here's how you might try to do it with aliases:

Так вы можете попытаться сделать это с псевдонимами:

```fsharp
type CustomerId = int   // определяем псевдоним типа
type OrderId = int      // определяем другой псевдоним типа

let printOrderId (orderId:OrderId) =
   printfn "The orderId is %i" orderId

//пробуем
let custId = 1          // создаем идентификатор заказчика
printOrderId custId     // Упс!
```

>But even though I explicitly annotated the `orderId` parameter to be of type `OrderId`, I can't ensure that customer ids are not accidentally passed in.

Но даже если я явно указал, что параметр `orderId` имеет тип `OrderId`, я не могу гарантировать, что не будут случайно переданы идентификаторы заказчика.

>On the other hand, if we create simple union types, we can easily enforce the type distinctions.

С другой стороны, если мы создаем простые типы объединений, мы можем легко обеспечить различение типов. 

```fsharp
type CustomerId = CustomerId of int   // определяем тип объединения
type OrderId = OrderId of int         // определяем другой тип объединения

let printOrderId (OrderId orderId) =  // деконструируем в параметре
   printfn "The orderId is %i" orderId

//пробуем
let custId = CustomerId 1             // создаем идентификатор заказчика
printOrderId custId                   // Хорошо! Имеем ошибку компиляции.
```

>This approach is feasible in C# and Java as well, but is rarely used because of the overhead of creating and managing the special classes for each type. 
Такой подход осуществим также в C# и Java, но он редко используется из-за накладных расходов по созданию и управлению специальными классами для каждого типа.
>In F# this approach is lightweight and therefore quite common.  
В F# такой подход легковесен и, следовательно, довольно распространен.

>A convenient thing about single case union types is you can pattern match directly against a value without having to use a full `match-with` expression.

Полезный нюанс у типов объединения с одиночным вариантом - вы можете сопоставлять их напрямую со значением без необходимости использования полного `match-with` выражения.

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

>But a common "gotcha" is that in some cases, the pattern match must have parens around it, otherwise the compiler will think you are defining a function!

Но основная "загвоздка" в том, что в некоторых случаях образец должен быть заключён в скобки, иначе компилятор подумает, что вы определяете функцию!

```fsharp
let custId = CustomerId 1
let (CustomerId customerIdInt) = custId  // Корректное сопоставление с образцом
let CustomerId customerIdInt = custId    // Неверно! Новая функция?
```

>Similarly, if you ever do need to create an enum-style union type with a single case, you will have to start the case with a vertical bar in the type definition; otherwise the compiler will think you are creating an alias.

Аналогично, если вам когда-либо понадобится создать объединение в стиле перечисления с одиночным вариантом, вы должны перед вариантом поставить вертикальную черту в определении типа; иначе компилятор подумает, что вы создаете псевдоним.

```fsharp
type TypeAlias = A     // псевдоним типа!
type SingleCase = | A   // тип объединения с одиночным типом
```


>## Union equality ##

## Сравнение объединений ##

>Like other core F# types, union types have an automatically defined equality operation: two unions are equal if they have the same type and the same case and the values for that case is equal.

Как и другие основные типы в F#, типы объединения имеют автоматически определенную операцию сравнения: два объединения равны, если они имеют одинаковый тип, одинаковый вариант и значения для этого варианта равны.

```fsharp
type Contact = Email of string | Phone of int

let email1 = Email "bob@example.com"
let email2 = Email "bob@example.com"

let areEqual = (email1=email2)
```


>## Union representation ##

## Представление объединений ##

>Union types have a nice default string representation, and can be serialized easily.
Типы объединения имеют приятное строковое представление по умолчанию, и могут быть легко сериализованы.
>But unlike tuples, the ToString() representation is unhelpful.
Но в отличие от кортежей представление через ToString() бесполезно.

```fsharp
type Contact = Email of string | Phone of int
let email = Email "bob@example.com"
printfn "%A" email    // хорошо
printfn "%O" email    // отвратительно!
```
<!-- похоже, что-то поменялось с момента написания статьи, т.к. у меня оба случая выводятся одинаково "хорошо" -->