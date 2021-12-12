---
layout: page
title: "Зачем использовать F#?"
description: "Почему вам стоит задуматься об использовании F# в вашем следующем проекте"
nav: why-use-fsharp
hasIcons: 1
image: "/why-use-fsharp/four-concepts2.png"
date: 2020-01-01
GeekdocAnchor: false
---

>Although F# is great for specialist areas such as scientific or data analysis, it is also an excellent choice for enterprise development.
Хотя F# является хорошим <!-- решением\языком --> для специалистов в таких областях как научная или анализ данных, он также является отличным выбором для коммерческой разработки.
<!--Хотя F# хорошо подходит для специалистов в научной области и анализе данных, он также будет отличным решением для коммерческой разработки.-->

>Here are five good reasons why you should consider using F# for  your next project.
Ниже приведены 5 хороших причин, почему вам стоит рассмотреть использование F# в вашем следующем проекте.

## {{<glyphicon glyphicons_030_pencil>}}&nbsp;Conciseness
## {{<glyphicon glyphicons_030_pencil>}}&nbsp;Выразительность

>F# is not cluttered up with [coding "noise"](/posts/fvsc-sum-of-squares/) such as curly brackets, semicolons and so on.
F# не захламлён ["шумом" кодирования](/posts/fvsc-sum-of-squares/) таким как круглые скобки, точки с запятой и так далее.
<!--Лучше просто использовать "шумом", или уже "ненужными конструкциями"--> 

>You almost never have to specify the type of an object, thanks to a powerful [type inference system](/posts/conciseness-type-inference/).
Вам практически никогда не стоит указывать тип объекта, благодаря мощной [системе приведения типов](/posts/conciseness-type-inference/).

>And, compared with C#, it generally takes [fewer lines of code](/posts/fvsc-download/) to solve the same problem.
И, в сравнении с C#, обычно нужно [меньше строк кода](/posts/fvsc-download/) <!-- для того --> чтобы решить аналогичную проблему.

```fsharp
>// one-liners
// Запись в одну строку
<!-- Напрашивается дословный перевод "однострочники" -->
[1..100] |> List.sum |> printfn "sum=%d"

>// no curly braces, semicolons or parentheses
// Отсутствие фигурных скобок, точек с запятой или круглых скобок
let square x = x * x
let sq = square 42

>// simple types in one line
// запись простых типов в одну строку
type Person = {First:string; Last:string}

>// complex types in a few lines
// сложные типы в несколько строк
type Employee =
  | Worker of Person
  | Manager of Employee list

>// type inference
// приведение типов
let jdoe = {First="John"; Last="Doe"}
let worker = Worker jdoe
```

----

## {{<glyphicon glyphicons_343_thumbs_up>}}&nbsp;Convenience
## {{<glyphicon glyphicons_343_thumbs_up>}}&nbsp;Удобство

>Many common programming tasks are much simpler in F#.
Множество привычных задач программирования <!-- выглядят --> значительно проще в F#.

>This includes things like creating and using [complex type definitions](/posts/conciseness-type-definitions/), doing [list processing](/posts/conciseness-extracting-boilerplate/), [comparison and equality](/posts/convenience-types/), [state machines](/posts/designing-with-types-representing-states/), and much more.
Это включает в сябя такие вещи как создание и использование [сложных типов <!--сложных\комплексных определений типов-->](/posts/conciseness-type-definitions/), выполнение [обработок списков](/posts/conciseness-extracting-boilerplate/), [сравнение и проверки на равенство]((/posts/convenience-types/), [конечных автоматов](/posts/designing-with-types-representing-states/), и многое другое.
<!--Тут, `state machine` скорее всего используется как `finite state machine`, а это `конечный автомат`. Страница https://docs.microsoft.com/ru-ru/dotnet/framework/windows-workflow-foundation/state-machine-workflows на русском тоже использует `конечный автомат` -->

>And because functions are first class objects, it is very easy to create powerful and reusable code by creating functions that have [other functions as parameters](/posts/conciseness-extracting-boilerplate/), or that [combine existing functions](/posts/conciseness-functions-as-building-blocks/) to create new functionality.
И, потому что функции являются объектами первого класса, очень легко создать мощный и переиспользуемый код, создавая функции которые имеют [другие функции в качестве входных параметров](/posts/conciseness-extracting-boilerplate/), или таких что [комбинируют существующие функции](/posts/conciseness-functions-as-building-blocks/) для создания новой функциональности.
<!-- `powerful` перевёл как `мощный`, но тут бы лучше подошло `многофункциональный`.-->


```fsharp
>// automatic equality and comparison
// автоматическая проверка на равенство и сравнение
type Person = {First:string; Last:string}
let person1 = {First="john"; Last="Doe"}
let person2 = {First="john"; Last="Doe"}
printfn "Equal? %A"  (person1 = person2)

>// easy IDisposable logic with "use" keyword
// простая логика IDisposable с ключевым словом "use"
use reader = new StreamReader(..)

>// easy composition of functions
// простая композиция функций
let add2times3 = (+) 2 >> (*) 3
let result = add2times3 5
```

----

## {{<glyphicon glyphicons_150_check>}}&nbsp;Correctness
## {{<glyphicon glyphicons_150_check>}}&nbsp;Правильность


>F# has a [powerful type system](/posts/correctness-type-checking/) which prevents many common errors such as [null reference exceptions](/posts/the-option-type/#option-is-not-null).
F# имеет [мощную систему типов](/posts/correctness-type-checking/) которая предупреждает множество привычных ошибок, например [исключения для ссылок на null](/posts/the-option-type/#option-is-not-null).
<!-- `powerful` перевёл как `мощную`, но тут бы лучше подошло `многофункциональный`.-->
<!-- `null reference exception` - `исключение для ссылок на null`, не думаю что `null` стоит переводить, так как это часть языка F#. В словарь добавил два варианта-->

>Values are [immutable by default](/posts/correctness-immutability/), which prevents a large class of errors.
Значения является [неизменяемыми по умолчанию](/posts/correctness-immutability/), что предотвращает большой класс ошибок.

>In addition, you can often encode business logic using the [type system](/posts/correctness-exhaustive-pattern-matching/) itself in such a way that it is actually [impossible to write incorrect code](/posts/designing-for-correctness/) or mix up [units of measure](/posts/units-of-measure/), greatly reducing the need for unit tests.
В добавок, вы часто можете закодировать бизнес логику используя [систему типов](/posts/correctness-exhaustive-pattern-matching/) таким образом, что будет практически [невозможно написать неправильный код](/posts/designing-for-correctness/) или смешать [величины измерения](/posts/units-of-measure/), сильно уменьшая потребность в модульных тестах.
<!--`encode` - закодировать, но тут лучше будет `описать`.-->

```fsharp
>// strict type checking
// строгая проверка типов
printfn "print string %s" 123 //compile error

>// all values immutable by default
// все значения являются неизменяемыми по умолчанию
person1.First <- "new name"  //assignment error

>// never have to check for nulls
// никогда не нужно делать проверку на null
let makeNewString str =
   //str can always be appended to safely
   //str всегда можно безопасно добавить
   let newString = str + " new!"
   newString

>// embed business logic into types
// встраивание бизнес логики в типы
emptyShoppingCart.remove   // compile error!

>// units of measure
// величины измерения
let distance = 10<m> + 10<ft> // error!
```

----


>## {{<glyphicon glyphicons_054_clock>}}&nbsp;Concurrency
## {{<glyphicon glyphicons_054_clock>}}&nbsp;Параллелизм

>F# has a number of built-in libraries to help when more than one thing at a time is happening. 
F# содержит число встроенных библиотек для помощи, когда более чем одна вещь происходит одновременно.

>Asynchronous programming is [very easy](/posts/concurrency-async-and-parallel/), as is parallelism.
Асинхронное программирование является [очень простым](/posts/concurrency-async-and-parallel/), так же как и параллелизм.

>F# also has a built-in [actor model](/posts/concurrency-actor-model/), and excellent support for event handling and [functional reactive programming](/posts/concurrency-reactive/).
F# также имеет встроенную [модель акторов](/posts/concurrency-actor-model/), и отличную поддержку для обработки событий и [функционального реактивного программирования](/posts/concurrency-reactive/).

>And of course, because data structures are immutable by default, sharing state and avoiding locks is much easier.
И, конечно же, из-за того что структуры данных являются неизменяемыми по умолчанию, разделение состояния и избежание блокировок значительно легче.


```fsharp
>// easy async logic with "async" keyword
// простая асинхронная логика с использованием ключевого слова "async"
let! result = async {something}

>// easy parallelism
// простая параллелизация
Async.Parallel [ for i in 0..40 ->
      async { return fib(i) } ]

>// message queues
// очереди сообщений
MailboxProcessor.Start(fun inbox-> async{
    let! msg = inbox.Receive()
    printfn "message is: %s" msg
    })
```

----

## {{<glyphicon glyphicons_280_settings>}}&nbsp;Полнота


>Although it is a functional language at heart, F# does support other styles which are not 100% pure, which makes it much easier to interact with the non-pure world of web sites, databases, other applications, and so on.
Хотя в глубине души это функциональный язык, F# поддерживает другие стили, которые не на 100% "чистые", что позволяет значительно легче взаимодействовать с "нечистым" миром интернет сайтов, баз данных, других приложений, и так далее.

>In particular, F# is designed as a hybrid functional/OO language, so it can do [virtually everything that C# can do](/posts/completeness-anything-csharp-can-do/).
В частности, F# разработан как гибридный функциональный/ОО язык, по этому он может [практически всё, что можно сделать с C#](/posts/completeness-anything-csharp-can-do/). 


>Of course, F# is [part of the .NET ecosystem](/posts/completeness-seamless-dotnet-interop/), which gives you seamless access to all the third party .NET libraries and tools.
Конечно, F# [является частью экосистемы .NET](/posts/completeness-seamless-dotnet-interop/), что предоставляет беспрепятственный доступ ко всем сторонним .NET библиотекам и инструментам.

>It runs on most platforms, including Linux and smart phones (via Mono).
Он работает на множестве платформ, включая Linux и смартфоны (используя Mono).

>Finally, it is well integrated with Visual Studio, which means you get a great IDE with IntelliSense support, a debugger, and many plug-ins for unit tests, source control, and other development tasks.
Наконец, он хорошо интегрируется с Visual Studio, что значит что вы можете получить отличную поддержку от IDE и IntelliSense, отладчик, и множество плагинов для модульных тестов, контроля исходного кода, и других задач разработки.

>Or on Linux, you can use the MonoDevelop IDE instead.
На Linux, вместо этого мы можете использовать MonoDevelop IDE.


```fsharp
>// impure code when needed
// нечистый код, когда в этом есть необходимость
let mutable counter = 0

>// create C# compatible classes and interfaces
// создание классов и интерфейсов совместимых с C#
type IEnumerator<'a> =
    abstract member Current : 'a
    abstract MoveNext : unit -> bool

>// extension methods
// методы расширения
type System.Int32 with
    member this.IsEven = this % 2 = 0

let i=20
if i.IsEven then printfn "'%i' is even" i

>// UI code
// код пользовательского интерфейса
open System.Windows.Forms
let form = new Form(Width = 400, Height = 300,
   Visible = true, Text = "Hello World")
form.TopMost <- true
form.Click.Add (fun args -> printfn "clicked!")
form.Show()
```

----

## Want more details?
## Хотите больше подробностей?

>If you want more information, the ["Why use F#?" series of posts](/series/why-use-fsharp/) covers each of these points in much greater detail.
Если вы хотите больше информации, [серия постов "Зачем использовать F#?"] намного лучше раскрывает каждый из этих пунктов.
<!-- Если вам нужно/если вы хотите; раскрывает каждый из этих пунктов более детально. -->

