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

Хотя F# хорошо подходит для специалистов в научной области и анализе данных, он также будет отличным решением для коммерческой разработки.

Ниже приведены 5 хороших причин, почему вам стоит рассмотреть использование F# в вашем следующем проекте.

## {{<glyphicon glyphicons_030_pencil>}}&nbsp;Выразительность

F# не захламлён [синтаксическим "шумом"](/posts/fvsc-sum-of-squares/) таким как круглые скобки, точки с запятой и так далее.

Вам практически никогда не стоит указывать тип объекта, благодаря мощному [выводу типов](/posts/conciseness-type-inference/).

И, в сравнении с C#, обычно нужно [меньше строк кода](/posts/fvsc-download/), чтобы решить ту же проблему.

```fsharp
// Запись в одну строку
[1..100] |> List.sum |> printfn "sum=%d"

// Отсутствие фигурных скобок, точек с запятой или круглых скобок
let square x = x * x
let sq = square 42

// Простые типы в одну строку
type Person = {First:string; Last:string}

// Сложные типы в несколько строк
type Employee =
  | Worker of Person
  | Manager of Employee list

// Приведение типов
let jdoe = {First="John"; Last="Doe"}
let worker = Worker jdoe
```

----

## {{<glyphicon glyphicons_343_thumbs_up>}}&nbsp;Удобство

Множество привычных задач программирования значительно проще решаются на F#.

Оно включает в себя такие задачи, как создание и использование [сложных типов](/posts/conciseness-type-definitions/), [обработку списков](/posts/conciseness-extracting-boilerplate/), [сравнение и проверки на равенство](/posts/convenience-types/), [конечных автоматов](/posts/designing-with-types-representing-states/), и многое другое.

И, потому что функции являются полноправными объектами, очень легко писать мощный и переиспользуемый код, создавая функции которые имеют [другие функции в качестве входных параметров](/posts/conciseness-extracting-boilerplate/), или таких что [комбинируют существующие функции](/posts/conciseness-functions-as-building-blocks/) для создания новой функциональности.

```fsharp
// автоматическая проверка на равенство и сравнение
type Person = {First:string; Last:string}
let person1 = {First="john"; Last="Doe"}
let person2 = {First="john"; Last="Doe"}
printfn "Equal? %A"  (person1 = person2)

// простая логика IDisposable с ключевым словом "use"
use reader = new StreamReader(..)

// простая композиция функций
let add2times3 = (+) 2 >> (*) 3
let result = add2times3 5
```

----

## {{<glyphicon glyphicons_150_check>}}&nbsp;Правильность


F# имеет [мощную систему типов](/posts/correctness-type-checking/) которая предотвращает множество привычных ошибок, например [исключения для ссылок на null](/posts/the-option-type/#option-is-not-null).

Значения являются [неизменяемыми по умолчанию](/posts/correctness-immutability/), что предотвращает большой пласт ошибок.

В добавок, вы часто можете описать бизнес логику используя [систему типов](/posts/correctness-exhaustive-pattern-matching/) таким образом, что будет практически [невозможно написать неправильный код](/posts/designing-for-correctness/) или смешать [величины измерения](/posts/units-of-measure/), сильно уменьшая потребность в модульных тестах.

```fsharp
// строгая проверка типов
printfn "print string %s" 123 // ошибка компиляции

// все значения являются неизменяемыми по умолчанию
person1.First <- "new name"  // ошибка присвоения

// никогда не нужно делать проверку на null
let makeNewString str =
   //str can always be appended to safely
   //str всегда можно безопасно добавить
   let newString = str + " new!"
   newString

// встраивание бизнес логики в типы
emptyShoppingCart.remove   // ошибка компиляции!

// единицы измерения
let distance = 10<m> + 10<ft> // ошибка!
```

----

## {{<glyphicon glyphicons_054_clock>}}&nbsp;Параллелизм

F# содержит множество встроенных библиотек для помощи, когда более чем одна вещь происходит одновременно.

Асинхронное программирование является [очень простым](/posts/concurrency-async-and-parallel/), так же как и параллелизм.

F# также имеет встроенную [модель акторов](/posts/concurrency-actor-model/), и отличную поддержку для обработки событий и [функционального реактивного программирования](/posts/concurrency-reactive/).

И, конечно же, из-за того что структуры данных являются неизменяемыми по умолчанию, разделение состояния и избежание блокировок значительно легче.

```fsharp
// простая асинхронная логика с использованием ключевого слова "async"
let! result = async {something}

// простой параллелизм
Async.Parallel [ for i in 0..40 ->
      async { return fib(i) } ]

// очереди сообщений
MailboxProcessor.Start(fun inbox-> async{
    let! msg = inbox.Receive()
    printfn "message is: %s" msg
    })
```

----

## {{<glyphicon glyphicons_280_settings>}}&nbsp;Полнота

Хотя в глубине души это функциональный язык, F# поддерживает другие стили, которые не на 100% "чистые", что позволяет значительно легче взаимодействовать с "нечистым" миром интернет сайтов, баз данных, других приложений и так далее.

В частности, F# разработан как гибридный функциональный/ОО язык, по этому он может [практически всё, что можно сделать с C#](/posts/completeness-anything-csharp-can-do/).

Конечно, F# [является частью экосистемы .NET](/posts/completeness-seamless-dotnet-interop/), что предоставляет беспрепятственный доступ ко всем сторонним .NET библиотекам и инструментам.

Он работает на множестве платформ, включая Linux и смартфоны (используя Mono).

Наконец, он хорошо интегрируется с Visual Studio, что значит что вы можете получить отличную поддержку от IDE и IntelliSense, отладчик, и множество плагинов для модульных тестов, контроля исходного кода, и других задач разработки.

На Linux, вместо этого мы можете использовать MonoDevelop IDE.

```fsharp
// нечистый код, когда в этом есть необходимость
let mutable counter = 0

// создание классов и интерфейсов совместимых с C#
type IEnumerator<'a> =
    abstract member Current : 'a
    abstract MoveNext : unit -> bool

// методы расширения
type System.Int32 with
    member this.IsEven = this % 2 = 0

let i=20
if i.IsEven then printfn "'%i' is even" i

// код пользовательского интерфейса
open System.Windows.Forms
let form = new Form(Width = 400, Height = 300,
   Visible = true, Text = "Hello World")
form.TopMost <- true
form.Click.Add (fun args -> printfn "clicked!")
form.Show()
```

----

## Хотите больше подробностей?

Если вы хотите больше информации, [серия постов "Зачем использовать F#?"](/series/why-use-fsharp/) намного полнее раскрывает каждый из этих пунктов.
