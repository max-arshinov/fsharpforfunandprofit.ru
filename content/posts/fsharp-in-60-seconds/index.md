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

Here is a very quick overview on how to read F# code for newcomers unfamiliar with the syntax.
Это очень быстрый обзор, как читать F# код для новичков, незнакомых с синтаксисом.

It is obviously not very detailed but should be enough so that you can read and get the gist of the upcoming examples in this series. Don't worry if you don't understand all of it, as I will give more detailed explanations when we get to the actual code examples.
Очевидно, что он не очень детальный, но достаточный для чтения и чтобы уловить суть предстоящих примеров в этой серии. Не переживайте, если вы не полностью понимаете все из этого, т.к. я дам более детальное объяснение, когда мы перейдем к примерам кода.

The two major differences between F# syntax and a standard C-like syntax are:
Двумя главными различиями между синтаксисом F# и стандартным С-подобным синтаксисом являются:

* Curly braces are not used to delimit blocks of code. Instead, indentation is used (Python is similar this way).
* Фигурные скобки не используются для разделения блоков кода. Вместо этого используются отступы (аналогично Python-у).
* Whitespace is used to separate parameters rather than commas.
* Для разделения параметров используются пробелы, а не запятые.

Some people find the F# syntax off-putting. If you are one of them, consider this quote:
Некоторые люди находят синтаксис F# оталкивающим. Если вы один из них, обратите внимание на эту цитату:

> "Optimising your notation to not confuse people in the first 10 minutes of seeing it but to hinder readability ever after is a really bad mistake."
> "Оптимизация вашей нотации, чтобы она не смущала людей в первые 10 минут, после того как они ее увидят, но которая затруднит читаемость в дальнейшем - это очень большая ошибка."
> (David MacIver, via [a post about Scala syntax](http://rickyclarkson.blogspot.co.uk/2008/01/in-defence-of-0l-in-scala.html)).
> (David MacIver, из [статьи о синтаксисе Scala](http://rickyclarkson.blogspot.co.uk/2008/01/in-defence-of-0l-in-scala.html)).

Personally, I think that the F# syntax is very clear and straightforward when you get used to it. In many ways, it is simpler than the C# syntax, with fewer keywords and special cases.
Лично я считаю что синтаксис F# очень понятен и прост, когда Вы к нему привыкнете. Во многих случаях он проще, чем синтаксис C#, с меньшим количеством ключевых слов и особых случаев.

The example code below is a simple F# script that demonstrates most of the concepts that you need on a regular basis.
Код примера ниже - это простой скрипт на языке F#, который демонстрирует большинство концепций, которые вам нужны регулярно.

I would encourage you to test this code interactively and play with it a bit! Either:
Я бы посоветовал вам протестировать этот код в интерактивном режиме и немного поиграть с ним! Или:

* Type this into a F# script file (with .fsx extension)
* Наберите этот код в файле для скриптов F# (с расширением .fsx)
and send it to the interactive window. See the ["installing and using F#"](/installing-and-using/) page for details.
и отправьте его в F# Interactive. Смотрите ["установка и использование F#"](/installing-and-using/) для деталей.
* Alternatively, try running this code in the interactive window. Remember to always use `;;` at the end to tell
* В качестве альтернативы, попробуйте запустить этот код в интерактивном окне. Не забывайте всегда использовать `;;` в конце чтобы сказать
the interpreter that you are done entering and ready to evaluate.
интерпретатору что вы завершили ввод и его можно исполнять.


```fsharp
// single line comments use a double slash
// однострочные комментарии используют двойную косую черту
(* multi line comments use (* . . . *) pair
(* многострочные коментарии используют пару (* . . . *)

-end of multi line comment- *)
-конец многострочного коментария- *)

// ======== "Variables" (but not really) ==========
// ======== "Переменные" (но не совсем) ==========
// The "let" keyword defines an (immutable) value
// Ключевое слово "let" определяет (неизменяемое) значение
let myInt = 5
let myFloat = 3.14
let myString = "hello"	//note that no types needed
let myString = "hello"	//обратите внимание, что никаких типов не требуется

// ======== Lists ============
// ======== Списки ============
let twoToFive = [2;3;4;5]        // Square brackets create a list with   // Квадратные скобки создают список с
                                 // semicolon delimiters.                // точками с запятой в качестве разделителя.
let oneToFive = 1 :: twoToFive   // :: creates list with new 1st element // :: создает список с новым первым элементом
// Результат [1;2;3;4;5]
// The result is [1;2;3;4;5]
let zeroToFive = [0;1] @ twoToFive   // @ concats two lists              // @ соединяет два списка 

// IMPORTANT: commas are never used as delimiters, only semicolons!
// ВАЖНО: запятые никогда не используются в качестве разделителей, только точки с запятой!

// ======== Functions ========
// ======== Функции ========
// The "let" keyword also defines a named function.
// Ключевое слово "let" так же определяет именованную функцию.
let square x = x * x          // Note that no parens are used.           // Обратите внимание, что круглые скобки не используются.
square 3                      // Now run the function. Again, no parens. // Сейчас запустим функцию. И снова нет скобок.

let add x y = x + y           // don't use add (x,y)! It means something // не используйте add (x,y)! Это означает
                              // completely different.                   // совершенно другое.
add 2 3                       // Now run the function.                   // И запустим функцию.

// to define a multiline function, just use indents. No semicolons needed.
// чтобы определить многострочную функцию, просто используйте отступы. Точки с запятой не нужны.
let evens list =
   let isEven x = x%2 = 0     // Define "isEven" as an inner ("nested") function // Определяем "isEven" в качестве внутренней ("вложенной") функции
   List.filter isEven list    // List.filter is a library function               // List.filter - это библиотечная функция
                              // with two parameters: a boolean function         // с двумя параметрами: булева функция
                              // and a list to work on                           // и список над которым надо совершить работу

evens oneToFive               // Now run the function                            // И запускаем функцию

// You can use parens to clarify precedence. In this example,
// Вы можете использовать скобки для уточнения приоритета. В этом примере,
// do "map" first, with two args, then do "sum" on the result.
// сначала выполни "map" с двумя аргументами первым, потом выполни "sum" над результатом.
// Without the parens, "List.map" would be passed as an arg to List.sum
// Без скобок, "List.map" будет передано как как аргумент List.sum
let sumOfSquaresTo100 =
   List.sum ( List.map square [1..100] )

// You can pipe the output of one operation to the next using "|>"
// Вы можете направить вывод одной операции в следующую используя "|>"
// Here is the same sumOfSquares function written using pipes
// Это та же sumOfSquares функция написанная с использованием конвейера
let sumOfSquaresTo100piped =
   [1..100] |> List.map square |> List.sum  // "square" was defined earlier      // функция "square" определена ранее

// you can define lambdas (anonymous functions) using the "fun" keyword
// Вы можете определить лямбды (анонимные функции) используя ключевое слово "fun"
let sumOfSquaresTo100withFun =
   [1..100] |> List.map (fun x->x*x) |> List.sum

// In F# returns are implicit -- no "return" needed. A function always
// В F# возврат неявный -- нет необходимости в "return". Функция всегда
// returns the value of the last expression used.
// возвращает значение последнего использованного выражения.

// ======== Pattern Matching ========
// ======== Сопоставление с Образцом ========
// Match..with.. is a supercharged case/switch statement.
// Match..with.. это заряженная конструкция case/switch.
let simplePatternMatch =
   let x = "a"
   match x with
    | "a" -> printfn "x is a"
    | "b" -> printfn "x is b"
    | _ -> printfn "x is something else"   // underscore matches anything        // подчеркивание сопоставляется с чем угодно

// Some(..) and None are roughly analogous to Nullable wrappers
// Some(..) и None это примерные аналоги обертки Nullable
let validValue = Some(99)
let invalidValue = None

// In this example, match..with matches the "Some" and the "None",
// В этом примере, match..with сопоставляется с "Some" и "None",
// and also unpacks the value in the "Some" at the same time.
// и одновременно с этим распаковывает значение из "Some".
let optionPatternMatch input =
   match input with
    | Some i -> printfn "input is an int=%d" i
    | None -> printfn "input is missing"

optionPatternMatch validValue
optionPatternMatch invalidValue

// ========= Complex Data Types =========
// ========= Сложные Типы Данных =========

// Tuple types are pairs, triples, etc. Tuples use commas.
// Кортежи - это пары, тройки и т.д. Кортежи используют запятые.
let twoTuple = 1,2
let threeTuple = "a",2,true

// Record types have named fields. Semicolons are separators.
// Записи содержат именованые поля. В качестве разделителя используется точка с запятой.
type Person = {First:string; Last:string}
let person1 = {First="john"; Last="Doe"}

// Union types have choices. Vertical bars are separators.
// Объединение содержит несколько возможных значений.
type Temp =
  | DegreesC of float
  | DegreesF of float
let temp = DegreesF 98.6

// Types can be combined recursively in complex ways.
// Типы можно рекурсивно комбинировать сложными способами.
// E.g. here is a union type that contains a list of the same type:
// Например, здесь используется объединение которое содержит список своего же типа:
type Employee =
  | Worker of Person
  | Manager of Employee list
let jdoe = {First="John";Last="Doe"}
let worker = Worker jdoe

// ========= Printing =========
// ========= Печать =========
// The printf/printfn functions are similar to the
// Функции printf/printfn схожи с
// Console.Write/WriteLine functions in C#.
// функциями Console.Write/WriteLine в C#.
printfn "Printing an int %i, a float %f, a bool %b" 1 2.0 true
printfn "A string %s, and something generic %A" "hello" [1;2;3;4]

// all complex types have pretty printing built in
// все сложные типы имеют встроенную красивую печать
printfn "twoTuple=%A,\nPerson=%A,\nTemp=%A,\nEmployee=%A"
         twoTuple person1 temp worker

// There are also sprintf/sprintfn functions for formatting data
// Также существуют функции sprintf/sprintfn для форматирования данных
// into a string, similar to String.Format.
// в строку, аналогичные String.Format.


```

And with that, let's start by comparing some simple F# code with the equivalent C# code.
Итак, давайте начнем со сравнения некоего простого кода F# с эквивалентным  C# кодом.
