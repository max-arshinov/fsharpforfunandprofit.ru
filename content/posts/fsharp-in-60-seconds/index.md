---
layout: post
title: "Синтакс F# за 60 секунд"
description: "Очень быстрый обзор как читать F# код"
date: 2012-04-02
nav: why-use-fsharp
seriesId: "Зачем использовать F#?"
seriesCode: why-use-fsharp
seriesOrder: 2
---

> Here is a very quick overview on how to read F# code for newcomers unfamiliar with the syntax.

Вот небольшая памятка, как читать код на F# — для новичков, не знакомых с синтаксисом языка.

> It is obviously not very detailed but should be enough so that you can read and get the gist of the
> upcoming examples in this series. Don't worry if you don't understand all of it, as I will give more
> detailed explanations when we get to the actual code examples.

Памятка, очевидно, не содержит важных подробностей, но её должно хватить, чтобы уловить суть и разбирать примеры, с которыми вы столкнётесь в следующих статьях. Не переживайте, если не сможете понять всё и сразу — детали я объясню позже, когда мы начнём рассматривать примеры кода.

> The two major differences between F# syntax and a standard C-like syntax are:

Два главных различия между синтаксисом F# и классическим синтаксисом языков-наследников C:

> * Curly braces are not used to delimit blocks of code. Instead, indentation is used (Python is similar this way).
> * Whitespace is used to separate parameters rather than commas.


* Для выделения блоков кода не нужны фигурные скобки. Вместо них, также, как и в Python, используются отступы.
* При определении и вызове функций параметры разделяются не запятыми, а пробелами.

> Some people find the F# syntax off-putting. If you are one of them, consider this quote:

Некоторым людям не нравится синтаксис F#. Если вы один из них, поразмышляйте над этой цитатой:

> > "Optimising your notation to not confuse people in the first 10 minutes of seeing it but to hinder readability ever after is a really bad mistake."
> > (David MacIver, via [a post about Scala syntax](http://rickyclarkson.blogspot.co.uk/2008/01/in-defence-of-0l-in-scala.html)).

> "Большая ошибкая — подстраивать синтаксис под людей, знакомых с языком 10 минут, если этот синтаксис затруднит понимание кода на всё оставшееся время."
> (Девид МакИвер (David MacIver), из [статьи, посвящённой синтаксису Scala](http://rickyclarkson.blogspot.co.uk/2008/01/in-defence-of-0l-in-scala.html)).

> Personally, I think that the F# syntax is very clear and straightforward when you get used to it. In
> many ways, it is simpler than the C# syntax, with fewer keywords and special cases.

Лично я считаю синтаксис F# очень простым и понятным, стоит только к нему привыкнуть. Часто он проще, чем синтаксис C# — в нём меньше ключевых слов и исключений.

> The example code below is a simple F# script that demonstrates most of the concepts that you need on a
> regular basis.

Пример ниже — простой скрипт на F#. Он демонстрирует большинство концепций, с которыми вам предстоит сталкиваться регулярно.

> I would encourage you to test this code interactively and play with it a bit! Either:

Я бы посоветовал вам запустить этот код в интерактивном режиме и немного поиграть с ним! Или:

> * Type this into a F# script file (with .fsx extension) and send it to the interactive window. See the
>   ["installing and using F#"](/installing-and-using/) page for details.
> * Alternatively, try running this code in the interactive window. Remember to always use `;;` at the
>   end to tell the interpreter that you are done entering and ready to evaluate.

* Введите его в файл для скриптов F# (с расширением .fsx), а затем отправьте в окно F# Interactive.
  Подробнее об этом способе рассказано в статье ["Установка и использование F#"](/installing-and-using/).
* В качестве альтернативы, попробуйте запустить код в интерактивном окне. Помните, что для запуска
  надо ввести символы `;;` — так интепретатор поймёт, что вы завершили ввод и код можно выполнять.


```fsharp
// однострочные комментарии начинаются с двойной косой черты
(* многострочные коментарии начинаются с символов (* и завершаются символами *)
они могут быть вложенными *)

// ======== "Переменные" (но не совсем) ==========
// Ключевое слово "let" определяет (неизменяемое) значение
let myInt = 5
let myFloat = 3.14
let myString = "hello"	//обратите внимание, что никаких типов не требуется

// ======== Списки ============
let twoToFive = [2;3;4;5]        // При создании списка в качестве разделителя
                                 // используются точки с запятой.

let oneToFive = 1 :: twoToFive   // :: создает список с новым первым элементом
                                 // Результат [1;2;3;4;5]
let zeroToFive = [0;1] @ twoToFive   // @ соединяет два списка 

// ВАЖНО: запятые никогда не используются в качестве разделителей, только точки с запятой!

// ======== Функции ========
// Ключевое слово "let" также определяет именованную функцию.
let square x = x * x          // Обратите внимание, что круглые скобки не используются.
square 3                      // Вызываем функцию. И снова без скобок.

let add x y = x + y           // не используйте add (x,y)! Скобки придают выражению
                              // другой смысл.
add 2 3                       // Вызываем функцию.

// Чтобы определить многострочную функцию, просто используйте отступы. Точки с запятой не нужны.
let evens list =
   let isEven x = x%2 = 0     // Определяем "isEven" как внутреннюю ("вложенную") функцию
   List.filter isEven list    // List.filter — библиотечная функция с двумя параметрами:
                              // булева функция
                              // и список над которым надо совершить работу

evens oneToFive               // Вызываем функцию.

// Скобки используются, чтобы обозначить порядок выполнения. Здесь, в примере,
// сначала выполнится "map" с двумя аргументами, а затем результат выполнения
// будет передан в функцию "sum".
// Без скобок функция "List.map" будет передана в "List.sum" как аргумент
let sumOfSquaresTo100 =
   List.sum ( List.map square [1..100] )

// Вы можете направить выход одной операции на вход другой, используя "|>"
// Вот та же функция sumOfSquares, написанная с использованием конвейера
let sumOfSquaresTo100piped =
   [1..100] |> List.map square |> List.sum  // функция "square" определена ранее

// Вы можете определять лямбды (анонимные функции), используя ключевое слово "fun"
let sumOfSquaresTo100withFun =
   [1..100] |> List.map (fun x->x*x) |> List.sum

// Функции в F# возвращают значения неявно, так что нет необходимости в ключевом слове "return".
// Результатом функции считается значение её последнего выражения.

// ======== Сопоставление с образцом ========
// Match/with это case/switch на максималках.
let simplePatternMatch =
   let x = "a"
   match x with
    | "a" -> printfn "x is a"
    | "b" -> printfn "x is b"
    | _ -> printfn "x is something else"   // подчеркивание сопоставляется с чем угодно

// Some(..) и None это примерные аналоги типа Nullable
let validValue = Some(99)
let invalidValue = None

// В этом примере, match/with сопоставляется с "Some" и "None",
// и, в то же время, извлекает значение из "Some".
let optionPatternMatch input =
   match input with
    | Some i -> printfn "input is an int=%d" i
    | None -> printfn "input is missing"

optionPatternMatch validValue
optionPatternMatch invalidValue

// ========= Сложные типы данных =========

// Кортежи — это пары значений, тройки значений, и т.д.
// Чтобы разделять значения в кортеже, используются запятые.
let twoTuple = 1,2
let threeTuple = "a",2,true

// Записи содержат именованые поля. В качестве разделителя используется точка с запятой.
type Person = {First:string; Last:string}
let person1 = {First="john"; Last="Doe"}

// Объединение содержит несколько возможных вариантов.
// В качестве разделителя используется вертикальная черта.
type Temp =
  | DegreesC of float
  | DegreesF of float
let temp = DegreesF 98.6

// Типы можно рекурсивно комбинировать сложными способами.
// Здесь, например, объединение, которое содержит список элементов своего же типа:
type Employee =
  | Worker of Person
  | Manager of Employee list
let jdoe = {First="John";Last="Doe"}
let worker = Worker jdoe

// ========= Печать =========
// Функции printf/printfn схожи с
// функциями Console.Write/WriteLine из C#.
printfn "Printing an int %i, a float %f, a bool %b" 1 2.0 true
printfn "A string %s, and something generic %A" "hello" [1;2;3;4]

// все сложные типы имеют встроенную красивую печать
printfn "twoTuple=%A,\nPerson=%A,\nTemp=%A,\nEmployee=%A"
         twoTuple person1 temp worker

// Также существуют функции sprintf/sprintfn для форматированного вывода
// в строку, аналогичные String.Format.
```

Итак, давайте возьмём простой код на F# и сравним его с эквивалентным кодом на C#.
