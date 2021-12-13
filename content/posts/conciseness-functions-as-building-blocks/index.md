---
layout: post
>title: "Using functions as building blocks"
title: "Использование функций в качестве строительных блоков"
>description: "Function composition and mini-languages make code more readable"
description: "Композиция функций и мини-языки делают код более читаемым"
date: 2012-04-11
nav: why-use-fsharp
>seriesId: "Why use F#?"
seriesId: "Зачем использовать F#?"
seriesOrder: 11
categories: [Conciseness, Functions]
---

>A well-known principle of good design is to create a set of basic operations and then combine these building blocks in various ways to build up more complex behaviors.
Широко известен такой принцип хорошего дизайна, как определение набора базовых операций и последующее комбинирование этих строительных блоков различными путями для создания более сложных поведений.
>In object-oriented languages, this goal gives rise to a number of implementation approaches such as "fluent interfaces", "strategy pattern", "decorator pattern", and so on.
В объектно-ориентированных языках эта цель порождает ряд подходов к реализации, таких как "флюент интерфейс"(англ. fluent interface, рус. "текучий интерфейс"), "паттерн стратегия", "паттерн декоратор", и т.д.
>In F#, they are all done the same way, via function composition.
В F# для всех этих вещей используют один подход — композицию функций.

>Let's start with a simple example using integers.
Давайте начнем с простого примера с использованием целых чисел.
>Say that we have created some basic functions to do arithmetic:
Допустим, мы создали несколько базовых функций для выполнения арифметических действий:

```fsharp
>// building blocks
// строительные блоки
let add2 x = x + 2
let mult3 x = x * 3
let square x = x * x

>// test
// тест
[1..10] |> List.map add2 |> printfn "%A"
[1..10] |> List.map mult3 |> printfn "%A"
[1..10] |> List.map square |> printfn "%A"
```

>Now we want to create new functions that build on these:
Теперь мы хотим создать новые функции, основанные на них:

```fsharp
>// new composed functions
// новые комбинированные функции
let add2ThenMult3 = add2 >> mult3
let mult3ThenSquare = mult3 >> square
```

>The "`>>`" operator is the composition operator.
Оператор "`>>`" - это оператор композиции.
>It means: do the first function, and then do the second.
Это значит: сначала выполни первую функцию, а потом вторую.

>Note how concise this way of combining functions is.
Обратите внимание как лаконичен этот способ комбинирования функций.
>There are no parameters, types or other irrelevant noise.
Нет никаких параметров, типов и другого неуместного шума.

>To be sure, the examples could also have been written less concisely and more explicitly as:
Для уверенности, можно написать эти примеры менее лаконично и более явно:

```fsharp
let add2ThenMult3 x = mult3 (add2 x)
let mult3ThenSquare x = square (mult3 x)
```

>But this more explicit style is also a bit more cluttered:
Но этот более яный способ также более громоздкий:

>* In the explicit style, the x parameter and the parentheses must be added, even though they don't add to the meaning of the code.
* В явном стиле необходимо добавить параметр x и скобки, не смотря на то, что они не добавляют смысла коду.
>* And in the explicit style, the functions are written back-to-front from the order they are applied.
>In my example of `add2ThenMult3` I want to add 2 first, and then multiply.
>The `add2 >> mult3` syntax makes this visually clearer than `mult3(add2 x)`.
* И в явном стиле функции написаны задом-наперед по отношению к порядку их применения.
В моем примере функции `add2ThenMult3` я хочу сначала прибавить 2, а потом умножить.
Синтаксис `add2 >> mult3` делает это визуально более очевидным, чем `mult3(add2 x)`.

>Now let's test these compositions:
Теперь давайте потестируем эти композиции:

```fsharp
>// test
// тест
add2ThenMult3 5
mult3ThenSquare 5
[1..10] |> List.map add2ThenMult3 |> printfn "%A"
[1..10] |> List.map mult3ThenSquare |> printfn "%A"
```

>## Extending existing functions
## Расширение существующих функций

>Now say that we want to decorate these existing functions with some logging behavior.
Допустим мы хотим декорировать существующие функции логирующим поведением.
>We can compose these as well, to make a new function with the logging built in.
Мы можем также их комбинировать, чтобы создать новые функции, со встроенным логированием.

```fsharp
>// helper functions;
// вспомогательные функции;
>let logMsg msg x = printf "%s%i" msg x; x     //without linefeed
let logMsg msg x = printf "%s%i" msg x; x     //без переноса строки
>let logMsgN msg x = printfn "%s%i" msg x; x   //with linefeed
let logMsgN msg x = printfn "%s%i" msg x; x   //с переносом строки

>// new composed function with new improved logging!
// нвоые комбинированные функции с улучшенным логированием!
let mult3ThenSquareLogged =
   logMsg "before="
   >> mult3
   >> logMsg " after mult3="
   >> square
   >> logMsgN " result="

>// test
// тест
mult3ThenSquareLogged 5
>[1..10] |> List.map mult3ThenSquareLogged //apply to a whole list
[1..10] |> List.map mult3ThenSquareLogged //применить ко всему списку
```

>Our new function, `mult3ThenSquareLogged`, has an ugly name, but it is easy to use and nicely hides the complexity of the functions that went into it.
У нашей новой функции `mult3ThenSquareLogged` ужасное имя, но её легко использовать и она неплохо прячет сложность функций вошедших в её состав.
>You can see that if you define your building block functions well, this composition of functions can be a powerful way to get new functionality.
Вы можете видеть, что если вы хорошо опишите свои функции - строительные блоки, то такая композиция может быть весьма мощным способом получения новой функциональности.

>But wait, there's more!
Но подождите, это еще не все!
>Functions are first class entities in F#, and can be acted on by any other F# code.
В F# функции - это объекты первого класса, и ими можно манипулировать с помощью другого кода на F#.
>Here is an example of using the composition operator to collapse a list of functions into a single operation.
Вот пример использования оператора композиции для схлопывания списка функций в одну операцию.

```fsharp
let listOfFunctions = [
   mult3;
   square;
   add2;
   logMsgN "result=";
   ]

>// compose all functions in the list into a single one
// комбинируем все функции в списке в одну
let allFunctions = List.reduce (>>) listOfFunctions

>//test
//тест
allFunctions 5
```

>## Mini languages
## Мини языки

>Domain-specific languages (DSLs) are well recognized as a technique to create more readable and concise code.
Предметно-ориентированные языки (англ. domain-specific language, DSL) широко известны в качестве техники создания более читаемого и лаконичного кода.
>The functional approach is very well suited for this.
Функциональный подход отлично для этого подходит.

>If you need to, you can go the route of having a full "external" DSL with its own lexer, parser, and so on, and there are various toolsets for F# that make this quite straightforward.
Если Вам необходимо, Вы можете пройти путем создания полностью "внешнего" предметно-ориентированного языка со своим лексером, парсером, и т.д., и существуют различные наборы инструментов для F#, которые сделают эту задачу достаточно простой.

>But in many cases, it is easier to stay within the syntax of F#, and just design a set of "verbs" and "nouns" that encapsulate the behavior we want.
Но во многих случаях проще придерживаться синтаксиса F#, и просто спроектировать набор "глаголов" и "существительных", инкапсулирующих нужное нам поведение.

>The ability to create new types concisely and then match against them makes it very easy to set up fluent interfaces quickly.
Возможность лаконичного создания новых типов и дальнейшего сопоставления с ними значительно упрощает быстрое создание флюент интерфейсов.
>For example, here is a little function that calculates dates using a simple vocabulary.
Например вот небольшая функция, вычисляющая даты используя простой словарь.
>Note that two new enum-style types are defined just for this one function.
Обратите внимание что два новых типа в стиле перечислений были определены исключительно для этой функции.

```fsharp
>// set up the vocabulary
// задаем словарь
type DateScale = Hour | Hours | Day | Days | Week | Weeks
type DateDirection = Ago | Hence

>// define a function that matches on the vocabulary
// создаем функцию, сопоставляющую со словарем
let getDate interval scale direction =
    let absHours = match scale with
                   | Hour | Hours -> 1 * interval
                   | Day | Days -> 24 * interval
                   | Week | Weeks -> 24 * 7 * interval
    let signedHours = match direction with
                      | Ago -> -1 * absHours
                      | Hence ->  absHours
    System.DateTime.Now.AddHours(float signedHours)

>// test some examples
// протестируем на нескольких примерах
let example1 = getDate 5 Days Ago
let example2 = getDate 1 Hour Hence

>// the C# equivalent would probably be more like this:
// эквивалент для C# будет выглядеть примерно так:
// getDate().Interval(5).Days().Ago()
// getDate().Interval(1).Hour().Hence()
```

>The example above only has one "verb", using lots of types for the "nouns".
В примере выше был только один "глагол", использующий несколько "существительных".

>The following example demonstrates how you might build the functional equivalent of a fluent interface with many "verbs".
В следующих примерах демонстрируется как Вы могли бы построить функциональный эквивалент флюент интерфейса со многими "глаголами".

>Say that we are creating a drawing program with various shapes.
Допустим что мы создаем программу для рисования с различными фигурами.
>Each shape has a color, size, label and action to be performed when clicked, and we want a fluent interface to configure each shape.
У каждой фигуры есть цвет, размер, ярлык и действие, выполняемое по клику, и нам нужен флюент интерфейс для настройки каждой фигуры.

>Here is an example of what a simple method chain for a fluent interface in C# might look like:
Вот пример того, как простая цепочка методов флюент интерфейса может выглядеть на C#:

```fsharp
FluentShape.Default
   .SetColor("red")
   .SetLabel("box")
   .OnClick( s => Console.Write("clicked") );
```

>Now the concept of "fluent interfaces" and "method chaining" is really only relevant for object-oriented design.
Однако концепт "флюент интерфейса" и "цепочки методов" имеет смысл только для объектно ориентированного дизайна.
>In a functional language like F#, the nearest equivalent would be the use of the pipeline operator to chain a set of functions together.
В функциональном языке как F# ближайшим эквивалентом будет использование конвейерного оператора для создания цепочки функций.

>Let's start with the underlying Shape type:
Давайте начнем с базового типа Shape:

```fsharp
>// create an underlying type
// создать базовый тип
type FluentShape = {
    label : string;
    color : string;
>    onClick : FluentShape->FluentShape // a function type
    onClick : FluentShape->FluentShape // тип функции
    }
```

>We'll add some basic functions:
Добавим некоторые базовые функции:

```fsharp
let defaultShape =
    {label=""; color=""; onClick=fun shape->shape}

let click shape =
    shape.onClick shape

let display shape =
    printfn "My label=%s and my color=%s" shape.label shape.color
>    shape   //return same shape
    shape   //возвращаем ту же фигуру
```

>For "method chaining" to work, every function should return an object that can be used next in the chain.
Для работы "цепочки вызовов", каждая функция должна возвращать объект, который можно будет использовать дальше по цепочке.
>So you will see that the "`display`" function returns the shape, rather than nothing.
Так что Вы можете видеть, что функция "`display`" возвращает фигуру, вместо ничего.

>Next we create some helper functions which we expose as the "mini-language", and will be used as building blocks by the users of the language.
Далее мы создадим несколько функций-помощников, которые мы представим как "мини-язык", и которые будут использованы пользователями языка как строительные блоки.

```fsharp
let setLabel label shape =
   {shape with FluentShape.label = label}

let setColor color shape =
   {shape with FluentShape.color = color}

>//add a click action to what is already there
//добавить действие по клику к тому, что уже там есть
let appendClickAction action shape =
   {shape with FluentShape.onClick = shape.onClick >> action}
```

>Notice that `appendClickAction` takes a function as a parameter and composes it with the existing click action.
Обратите внимание на то, что `appendClickAction` принимает функцию в качестве параметра и комбинирует её с существующим действием по клику.
>As you start getting deeper into the functional approach to reuse, you start seeing many more "higher order functions" like this, that is, functions that act on other functions.
По мере того, как Вы будете углубляться в функциональный подход к переиспользованию, Вы начнете замечать намного больше "функций высшего порядка" вроде этой, это функции которые совершают действия над другими функциями.
>Combining functions like this is one of the keys to understanding the functional way of programming.
Комбинирование функций таким образом - это один из ключей к пониманию функционального подхода к программированию.

>Now as a user of this "mini-language", I can compose the base helper functions into more complex functions of my own, creating my own function library.
Теперь как пользователь этого "мини-языка", я могу скомбинировать базовые функции-помощники в свои более сложные функции, создавая свою собственную библиотеку функций.
>(In C# this kind of thing might be done using extension methods.)
(В C# это то, чего можно добиться с помощью методов расширений.)

```fsharp
>// Compose two "base" functions to make a compound function.
// Скомбинировать две "базовые" функции для создания сложной функции
let setRedBox = setColor "red" >> setLabel "box"

>// Create another function by composing with previous function.
// Создать новую функцию скомбинировав с предыдущей функцией
>// It overrides the color value but leaves the label alone.
// Она переопределяет значение цвета, но не трогает ярлык.
let setBlueBox = setRedBox >> setColor "blue"

>// Make a special case of appendClickAction
// Создание особого вида appendClickAction
let changeColorOnClick color = appendClickAction (setColor color)
```

>I can then combine these functions together to create objects with the desired behavior.
Далее я могу скомбинировать эти функции вместе чтобы создать объекты с нужным поведением.

```fsharp
>//setup some test values
// подготовим несколько тестовых значений
let redBox = defaultShape |> setRedBox
let blueBox = defaultShape |> setBlueBox

>// create a shape that changes color when clicked
// создадим фигуру, меняющую цвет по клику
redBox
    |> display
    |> changeColorOnClick "green"
    |> click
>   |> display  // new version after the click
    |> display  // новая версия после клика

>// create a shape that changes label and color when clicked
// создадим фигуру, меняющую ярлык и цвет по клику
blueBox
    |> display
    |> appendClickAction (setLabel "box2" >> setColor "green")
    |> click
>   |> display  // new version after the click
    |> display  // новая версия после клика
```

>In the second case, I actually pass two functions to `appendClickAction`, but I compose them into one first.
Во втором случае я на самом деле передаю две функции в `appendClickAction`, но я сперва комбинирую их в одну.
>This kind of thing is trivial to do with a well structured functional library, but it is quite hard to do in C# without having lambdas within lambdas.
Такие вещи очень просто делать с хорошо структурированной библиотекой функций, но это достаточно тяжело сделать в C# без использования лямбд внутри лямбд.

>Here is a more complex example.
Вот более сложный пример.
>We will create a function "`showRainbow`" that, for each color in the rainbow, sets the color and displays the shape.
Мы создадим функцию "`showRainbow`" которая, для каждого цвета радуги задаёт цвет и выводит фигуру.

```fsharp
let rainbow =
    ["red";"orange";"yellow";"green";"blue";"indigo";"violet"]

let showRainbow =
    let setColorAndDisplay color = setColor color >> display
    rainbow
    |> List.map setColorAndDisplay
    |> List.reduce (>>)

>// test the showRainbow function
// тестируем функцию showRainbow
defaultShape |> showRainbow
```

>Notice that the functions are getting more complex, but the amount of code is still quite small.
Обратите внимание на то, что функции становятся более сложными, но количество кода все еще маленькое.
>One reason for this is that the function parameters can often be ignored when doing function composition, which reduces visual clutter.
Одна из причин - это то, что параметры функций часто могут быть игнорированы во время комбинирования функций, что уменьшает визуальный беспорядок.
>For example, the "`showRainbow`" function does take a shape as a parameter, but it is not explicitly shown!
Например функция "`showRainbow`" принимает фигуру в качестве параметра, но это не показано явно!
>This elision of parameters is called "point-free" style and will be discussed further in the ["thinking functionally"](/series/thinking-functionally.html) series.
Эта элизия параметров называется бесточечный стиль(бесточечная нотация, англ. "point-free" style) и будет обсуждаться далее в серии ["думай функционально"](/series/thinking-functionally.html).
