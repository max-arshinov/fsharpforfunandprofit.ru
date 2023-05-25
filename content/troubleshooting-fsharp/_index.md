---
layout: page
# title: "Troubleshooting F#"
title: "Исправление ошибок F#"
# description: "Why won't my code compile?"
description: "Почему мой код не компилируется?"
nav: troubleshooting-fsharp
hasComments: 1
date: 2020-01-01
---

> As the saying goes, "if it compiles, it's correct", but it can be extremely frustrating just trying to get the code to compile at all!
> So this page is devoted to helping you troubleshoot your F# code.

Говорят, что "если код компилируется, то он правильный", но иногда невероятно сложно заставить код даже компилироваться!
Так что эта страница написана, чтобы помочь вам справляться с ошибками в вашем F# коде.

> I will first present some general advice on troubleshooting and some of the most common errors that beginners make.
> After that, I will describe each of the common error messages in detail, and give examples of how they can occur and how to correct them.

Сначала я дам несколько универсальных советов по исправлению и расскажу о самых частых ошибках, которые совершают новички.
Затем я подробно остановлюсь на сообщениях об ошибках, и покажу, из-за чего они возникают, и как их исправить.

> [(Jump to the error numbers)](#NumericErrors)

[(Ошибки по номерам)](#NumericErrors)

> ## General guidelines for troubleshooting ##

## Общие советы по исправлению ошибок ##

> By far the most important thing you can do is to take the time and effort to understand exactly how F# works, especially the core concepts involving functions and the type system.
> So please read and reread the series ["thinking functionally"](/series/thinking-functionally.html) and ["understanding F# types"](/series/understanding-fsharp-types.html), play with the examples, and get comfortable with the ideas before you try to start doing serious coding.
> If you don't understand how functions and types work, then the compiler errors will not make any sense.

Безусловно, важнейшее, что вы можете сделать — это глубоко разобраться в F#, особенно в базовых концепциях, таких как функции и система типов.
Так что, пожалуйста, прочитайте и перечитайте циклы статей о том, как ["мыслить функционально"](/series/thinking-functionally.html) и ["разбираться в типах F#"](/series/understanding-fsharp-types.html), поиграйтесь с примерами, и освойте основные идеи, прежде чем приступать к серьёзным вещам.

> If you are coming from an imperative language such as C#, you may have developed some bad habits by relying on the debugger to find and fix incorrect code.
> In F#, you will probably not get that far, because the compiler is so much stricter in many ways.
> And of course, there is no tool to "debug" the compiler and step through its processing.
> The best tool for debugging compiler errors is your brain, and F# forces you to use it!

Если у вас есть опыт императивного программирования, например на C#, вы можете принести с собой вредные привычки, такие как поиск и исправление ошибок с помощью отладчика. В F# вы, вероятно, просто не доберётесь до отладки, поскольку компилятор здесь гораздо строже. И, конечно, не существует никакого средства «отлаживать» компилятор, пошагово проверяя его работу. Лучшее средство для отладки ошибок компиляции — это ваша голова, и F# заставляет её включить!

> Nevertheless, there are a number of extremely common errors that beginners make, and I will quickly go through them.

Впрочем, есть несколько ошибок, которые новички совершают неприлично часто, так что я быстро по ним пробегусь.

> ### Don't use parentheses when calling a function ###

### Не используйте скобки при вызове функций

> In F#, whitespace is the standard separator for function parameters.
> You will rarely need to use parentheses, and in particular, do not use parentheses when calling a function.

Правильный разделитель параметров функции в F# — это пробел.
Скобки вам потребуются нечасто, так что не пишите их при вызове функций.

```fsharp
let add x y = x + y
let result = add (1 2)  // неправильно
    // ошибка FS0003: это значение не является функцией и не может быть применено
let result = add 1 2    // правильно
```

> ### Don't mix up tuples with multiple parameters ###

### Не путайте кортежи и отдельные параметры функций

> If it has a comma, it is a tuple.
> And a tuple is one object not two.
> So you will get errors about passing the wrong type of parameter, or too few parameters.

Запятыми разделяются значения в кортежах.
И кортеж — это один объект, а не два.
Так что вы получите ошибку о неверном типе параметров или о том, что параметров слишком мало.

```fsharp
addTwoParams (1,2)  // пытаемся передать один кортеж вместо двух параметров
   // ошибка FS0001: ожидалось выражение типа int, обнаружен тип 'a * 'b
```

> The compiler treats `(1,2)` as a generic tuple, which it attempts to pass to "`addTwoParams`".
> Then it complains that the first parameter of `addTwoParams` is an int, and we're trying to pass a tuple.

Компилятор трактует `(1,2)` как кортеж, который он пробует передать в "`addTwoParams`".
В результате он жалуется, что первым параметром `addTwoParams` должно быть целое число, а мы пытаемся передать кортеж.

> If you attempt to pass *two* arguments to a function expecting *one* tuple, you will get another obscure error.

Если вы попытаетесь передать *два* аргумента в функцию, которая ожидает *один* кортеж, вы получите другое неясное сообщение об ошибке.

```fsharp
addTuple 1 2   // пытаемся передать два аргумента вместо одного кортежа
  // ошибка FS0003: это значение не является функцией и не может быть применено
```

> ### Watch out for too few or too many arguments ###

### Остерегайтесь неверного числа аргументов 

> The F# compiler will not complain if you pass too few arguments to a function (in fact "partial application" is an important feature), but if you don't understand what is going on, you will often get strange "type mismatch" errors later.

Компилятор F# не считает ошибкой слишком малое число аргументов функции (в конце концов, «частичное применение» — одно из важнейших свойств языка), но если вы не понимаете, что происходит, вы получите странные сообщения о несовпадении типов чуть дальше в программе.

> Similarly the error for having too many arguments is typically "This value is not a function" rather than a more straightforward error.

Сообщение о слишком большом числе аргументов тоже звучит не очень ясно: "Значение не является функцией".

> The "printf" family of functions is very strict in this respect.
> The argument count must be exact.

В этом отношении очень строгим является семейство функций "printf".
Здесь количество аргументов должно быть точным.

> This is a very important topic -- it is critical that you understand how partial application works.
> See the series ["thinking functionally"](/series/thinking-functionally.html) for a more detailed discussion.

Это очень важная тема — крайне важно разобраться в том, как работает частичное применение.
За подробностями обращайтесь к циклу ["мыслить функционально"](/series/thinking-functionally.html).

> ### Use semicolons for list separators ###

### Используйте точки с запятой, чтобы разделять значения в списках

> In the few places where F# needs an explicit separator character, such as lists and records, the semicolon is used.
> Commas are never used. (Like a broken record, I will remind you that commas are for tuples).

В тех редких случаях, когда F# нуждается в явном символе-разделителе, скажем, в списках и записях, нужно использовать точку с запятой.

```fsharp
let list1 = [1,2,3]    // неправильно! Это список из ОДНОГО кортежа,
                       // который содержит три значения
let list1 = [1;2;3]    // правильно

type Customer = {Name:string, Address: string}  // неправильно
type Customer = {Name:string; Address: string}  // правильно
```

> ### Don't use ! for not or != for not-equal ###

### Не используйте ! как логическое отрицание и != как проверку на неравенство

> The exclamation point symbol is not the "NOT" operator.
> It is the deferencing operator for mutable references. <!-- dereferencing -->
> If you use it by mistake, you will get the following error:

Восклицательный знак — это не оператор логического отрицания.
Это разыменовывающий оператор для изменяемых ссылок.
Если вы случайно его написали, вы получите такую ошибку:

```fsharp
let y = true
let z = !y
// => ошбика FS0001: ожидается выражение типа 'a ref, обнаружено выражение типа bool
```

> The correct construction is to use the "not" keyword.
> Think SQL or VB syntax rather than C syntax.

Вам нужно использовать ключевое слово "not", точно также, как в SQL или VB.

```fsharp
let y = true
let z = not y       // правильно
```

> And for "not equal", use "<>", again like SQL or VB.
Чтобы написать «не равно», используйте "<>" — снова, как в SQL или VB.


```fsharp
let z = 1 <> 2      // правильно
```

> ### Don't use = for assignment ###

### Не используйте = для присваивания

> If you are using mutable values, the assignment operation is written "`<-`".
> If you use the equals symbol you might not even get an error, just an unexpected result.

Оператор присваивания для изменяемых значений выглядит как "`<-`".
Символ равенства может даже не приводить к немедленной ошибке, а просто давать неожиданный результат.

```fsharp
let mutable x = 1
x = x + 1          // вернёт false. x не равно x+1
x <- x + 1         // присвоит x+1 переменной x
```

> ### Watch out for hidden tab characters ###

### Остерегайтесь скрытых символов табуляции

> The indenting rules are very straightforward, and it is easy to get the hang of them.
> But you are not allowed to use tabs, only spaces.

Правила отступов очень просты и их легко запомнить.
Но вам нельзя использовать табуляции, только пробелы.

```fsharp
let add x y =
{tab}x + y
// => ошибка FS1161: нельзя использовать символы табуляции
```

> Be sure to set your editor to convert tabs to spaces.
> And watch out if you are pasting code in from elsewhere.
> If you do run into persistent problems with a bit of code, try removing the whitespace and re-adding it.

Убедитесь, что ваш редактор преобразует табуляцию в пробелы.
И проверяйте код, который вы откуда-то вставляете.
Если какой-то код вызывает у вас проблемы, попробуйте удалить все пробелы, а затем добавить их обратно.

> ### Don't mistake simple values for function values ###

### Не путайте обычные значения с функциональными значениями

> If you are trying to create a function pointer or delegate, watch out that you don't accidentally create a simple value that has already been evaluated.

Если вы пытаетесь создать указатель на функцию или делегат, убедитесь, что вы случайно не создали простое значение, которое уже вычислено.

> If you want a parameterless function that you can reuse, you will need to explicitly pass a unit parameter, or define it as a lambda.

Если вам нужна функция без параметров, которую вы можете вызвать несколько раз, вам надо явно передавать параметр единочного типа `unit`, или определить её как лямбда-функцию.

```fsharp
let reader = new System.IO.StringReader("hello")
let nextLineFn   =  reader.ReadLine()  // неправильно
let nextLineFn() =  reader.ReadLine()  // правильно
let nextLineFn   =  fun() -> reader.ReadLine()  // правильно

let r = new System.Random()
let randomFn   =  r.Next()  // неправильно
let randomFn() =  r.Next()  // правильно
let randomFn   =  fun () -> r.Next()  // правильно
```

> See the series ["thinking functionally"](/series/thinking-functionally.html) for more discussion of parameterless functions.

Подробно функции без параметров обсуждаются в цикле ["мыслить функционально"](/series/thinking-functionally.html).

> ### Tips for troubleshooting "not enough information" errors ###

### Советы по устранению ошибок вида "недостаточно информации"

> The F# compiler is currently a one-pass left-to-right compiler, and so type information later in the program is unavailable to the compiler if it hasn't been parsed yet.

Компилятор F# (по крайней мере, пока) работает в один проход и читает исходный код слева направо, так что информация о типах, которые описаны дальше в программе, комплилятору недоступна — он её ещё не разобрал.

> A number of errors can be caused by this, such as ["FS0072: Lookup on object of indeterminate type"](#FS0072) and ["FS0041: A unique overload for could not be determined"](#FS0041).
> The suggested fixes for each of these specific cases are described below, but there are some general principles that can help if the compiler is complaining about missing types or not enough information.
> These guidelines are:

Часть ошибок может возникнуть именно поэтому, например, ["FS0072: Поиск объекта неопределённого типа"](#FS0072) и ["FS0041: Невозможно определить уникальную перегрузку"](#FS0041). Для каждой из них существуют свои способы исправления, но есть и несколько общих принципов, которые могут помочь, если компилятор жалуется на отсутствие типов или недостаток информации. Вот эти рекомендации:

> * Define things before they are used (this includes making sure the files are compiled in the right order)
> * Put the things that have "known types" earlier than things that have "unknown types".
    In particular, you might be able reorder pipes and similar chained functions so that the typed objects come first.
> * Annotate as needed.
    One common trick is to add annotations until everything works, and then take them away one by one until you have the minimum needed.

* Определяйте все элементы программы до того, как их использовать (и убедитесь, что файлы в проекте компилируется в правильном порядке)
* Размещайте объекты, тип которых известен, до объектов, тип которых неизвестен. Например, можно переупорядочить вызовы в конвейере или в подобной цепочке функций так, чтобы в начале шли объекты с известными типами.
* Аннотируйте по мере необходимости. Одна из общих рекомендаций состоит в том, чтобы добавлять аннотации к коду, пока всё не заработает, а потом убирать их, пока вы не достигнете необходимого минимума.

> Do try to avoid annotating if possible.
> Not only is it not aesthetically pleasing, but it makes the code more brittle.
> It is a lot easier to change types if there are no explicit dependencies on them.

Постарайтесь избегать аннотаций, если это возможно.
Не только потому, что аннотации некрасивы, но и потому, что они повышают хрупкость кода.
Изменять типы намного проще, если ничто в программе не зависит от них явно.

----

> # F# compiler errors

# Ошибки компилятора F# {#NumericErrors}

----

> Here is a list of the major errors that seem to me worth documenting.
> I have not documented any errors that are self explanatory, only those that seem obscure to beginners.

Вот список основных ошибок, которые, как я думаю, стоит задокументировать. Я не документировал очевидные ошибки — только те, которые кажутся непонятными новичкам.

> I will continue to add to the list in the future, and I welcome any suggestions for additions.

Я собираюсь обновлять этот список в будущем, и открыт к вашим предложениям по этому поводу.

> * [FS0001: The type 'X' does not match the type 'Y'](#FS0001)
> * [FS0003: This value is not a function and cannot be applied](#FS0003)
> * [FS0008: This runtime coercion or type test involves an indeterminate type](#FS0008)
> * [FS0010: Unexpected identifier in binding](#FS0010a)
> * [FS0010: Incomplete structured construct](#FS0010b)
> * [FS0013: The static coercion from type X to Y involves an indeterminate type](#FS0013)
> * [FS0020: This expression should have type 'unit'](#FS0020)
> * [FS0030: Value restriction](#FS0030)
> * [FS0035: This construct is deprecated](#FS0035)
> * [FS0039: The field, constructor or member X is not defined](#FS0039)
> * [FS0041: A unique overload for could not be determined](#FS0041)
> * [FS0049: Uppercase variable identifiers should not generally be used in patterns](#FS0049)
> * [FS0072: Lookup on object of indeterminate type](#FS0072)
> * [FS0588: Block following this 'let' is unfinished](#FS0588)

* [FS0001: Тип 'X' не совпадает с типом 'Y'](#FS0001)
* [FS0003: Значение не является функцией и не может быть применено](#FS0003)
* [FS0008: Приведение типа времени выполнения или сопоставление типа приводит к неопределяемому типу](#FS0008)
* [FS0010: Неожиданный идентификатор в привязке](#FS0010a)
* [FS0010: Не полностью структурированная конструкция](#FS0010b)
* [FS0013: Статическое приведение типа X к типу Y приводит к неопределённому типу](#FS0013)
* [FS0020: Выражение должно иметь тип 'unit'](#FS0020)
* [FS0030: Ограничение на значение](#FS0030)
* [FS0035: Эта конструкция устарела](#FS0035)
* [FS0039: Поле, конструктор или член X не определён](#FS0039)
* [FS0041: Невозможно определить уникальную перегрузку](#FS0041)
* [FS0049: В образцах нельзя использовать идентификаторы переменных в верхнем регистре](#FS0049)
* [FS0072: Поиск объекта неопределённого типа](#FS0072)
* [FS0588: Блок, следующий за оператором 'let', не завершён](#FS0588)

> ## FS0001: The type 'X' does not match the type 'Y'

## FS0001: Тип 'X' не совпадает с типом 'Y' {#FS0001}

> This is probably the most common error you will run into.
> It can manifest itself in a wide variety of contexts, so I have grouped the most common problems together with examples and fixes.
> Do pay attention to the error message, as it is normally quite explicit about what the problem is.

Возможно, это самая частая ошибка, с которой вы столкнётесь. Она может проявляться в самых разных ситуациях, так что я сгруппировал основные проблемы, добавив примеры и способы исправления.

> {{<rawtable>}}
> <table class="table table-striped table-bordered table-condensed">
> <thead>
>   <tr>
> 	<th>Error message</th>
> 	<th>Possible causes</th>
>   </tr>
> </thead>
> <tbody>
>   <tr>
> 	<td>The type 'float' does not match the type 'int'</td>
> 	<td><a href="#FS0001A">A. Can't mix floats and ints</a></td>
>   </tr>
>   <tr>
> 	<td>The type 'int' does not support any operators named 'DivideByInt'</td>
> 	<td><a href="#FS0001A">A. Can't mix floats and ints.</a></td>
>   </tr>
>   <tr>
> 	<td>The type 'X' is not compatible with any of the types</td>
> 	<td><a href="#FS0001B">B. Using the wrong numeric type.</a></td>
>   </tr>
>   <tr>
> 	<td>This type (function type) does not match the type (simple type). Note: function types have a arrow in them, like <code>'a -&gt; 'b</code>.</td>
> 	<td><a href="#FS0001C">C. Passing too many arguments to a function.</a></td>
>   </tr>
>   <tr>
> 	<td>This expression was expected to have (function type) but here has (simple type)</td>
> 	<td><a href="#FS0001C">C. Passing too many arguments to a function.</a></td>
>   </tr>
>   <tr>
> 	<td>This expression was expected to have (N part function) but here has (N-1 part function)</td>
> 	<td><a href="#FS0001C">C. Passing too many arguments to a function.</a></td>
>   </tr>
>   <tr>
> 	<td>This expression was expected to have (simple type) but here has (function type)</td>
> 	<td><a href="#FS0001D">D. Passing too few arguments to a function.</a></td>
>   </tr>
>   <tr>
> 	<td>This expression was expected to have (type) but here has (other type)</td>
> 	<td><a href="#FS0001E">E. Straightforward type mismatch.</a><br>
> 	<a href="#FS0001F">F. Inconsistent returns in branches or matches.</a><br>
> 	<a href="#FS0001G">G. Watch out for type inference effects buried in a function.</a><br>
> 	</td>
>   </tr>
>   <tr>
> 	<td>Type mismatch. Expecting a (simple type) but given a (tuple type). Note: tuple types have a star in them, like <code>'a * 'b</code>.</td>
> 	<td><a href="#FS0001H">H. Have you used a comma instead of space or semicolon?</a></td>
>   </tr>
>   <tr>
> 	<td>Type mismatch. Expecting a (tuple type) but given a (different tuple type). </td>
> 	<td><a href="#FS0001I">I. Tuples must be the same type to be compared.</a></td>
>   </tr>
>   <tr>
> 	<td>This expression was expected to have type <code>'a ref</code> but here has type X</td>
> 	<td><a href="#FS0001J">J. Don't use ! as the "not" operator.</a></td>
>   </tr>
>   <tr>
> 	<td>The type (type) does not match the type (other type)</td>
> 	<td><a href="#FS0001K">K. Operator precedence (especially functions and pipes).</a></td>
>   </tr>
>   <tr>
> 	<td>This expression was expected to have type (monadic type) but here has type <code>'b * 'c</code></td>
> 	<td><a href="#FS0001L">L. let! error in computation expressions.</a></td>
>   </tr>
> </tbody>
> </table>
> {{</rawtable>}}

{{<rawtable>}}
<table class="table table-striped table-bordered table-condensed">
<thead>
  <tr>
	<th>Сообщение об ошибке</th>
	<th>Возможные причины</th>
  </tr>
</thead>
<tbody>
  <tr>
	<td>Тип 'float' не совпадает с типом 'int'</td>
	<td><a href="#FS0001A">A. Нельзя смешивать числа с плавающей точкой и целые числа.</a></td>
  </tr>
  <tr>
	<td>Тип 'int' не поддерживает никаких операторов с именем 'DivideByInt'</td>
	<td><a href="#FS0001A">A. Нельзя смешивать числа с плавающей точкой и целые числа.</a></td>
  </tr>
  <tr>
	<td>Тип 'X' не совместим с любым из типов</td>
	<td><a href="#FS0001B">B. Использование неверного численного типа.</a></td>
  </tr>
  <tr>
	<td>Тип (функциональный тип) не совпадает с типом (простой тип). Замечания: функциональные типы содержат стрелку, например, <code>'a -&gt; 'b</code>.</td>
	<td><a href="#FS0001C">C. Слишком много аргументов при вызове функции.</a></td>
  </tr>
  <tr>
	<td>Ожидалось выражение типа (функциональный тип), обнаружен тип (простой тип)</td>
	<td><a href="#FS0001C">C. Слишком много аргументов при вызове функции.</a></td>
  </tr>
  <tr>
	<td>Ожидалось выражение типа (функция с N параметрами), обнаружен тип (функция с N-1 параметрами)</td>
	<td><a href="#FS0001C">C. Слишком много аргументов при вызове функции.</a></td>
  </tr>
  <tr>
	<td>Ожидалось выражение типа (простой тип), обнаружен тип (функциональный тип)</td>
	<td><a href="#FS0001D">D. Слишком мало аргументов при вызове функции.</a></td>
  </tr>
  <tr>
	<td>Ожидалось выражение типа (тип), обнаружен тип (другой тип)</td>
	<td><a href="#FS0001E">E. Простое несоответствие типов.</a><br>
	<a href="#FS0001F">F. Несогласованные типы в ветвлениях или сопоставлениях.</a><br>
	<a href="#FS0001G">G. Остерегайтесь вывода типов внутри функции.</a><br>
	</td>
  </tr>
  <tr>
	<td>Несовпадение типов. Ожидается (простой тип), обнаружен (тип кортежа). Замечание: типы кортежа содержат звёздочку, например, <code>'a * 'b</code>.</td>
	<td><a href="#FS0001H">H. Запятая вместо пробела или точки с запятой?</a></td>
  </tr>
  <tr>
	<td>Несовпадение типов. Ожидается (тип кортежа), обнаружен (другой тип кортежа). </td>
	<td><a href="#FS0001I">I. Кортежи должны быть одного типа, чтобы их можно было сравнивать.</a></td>
  </tr>
  <tr>
	<td>Ожидалось выражение типа <code>'a ref</code>, обнаружен тип X</td>
	<td><a href="#FS0001J">J. Не используйте ! как оператор отрицания.</a></td>
  </tr>
  <tr>
	<td>Тип (тип) не совпадает с типом (другой тип)</td>
	<td><a href="#FS0001K">K. Приоритет операторов (особенно, функций и конвейеров).</a></td>
  </tr>
  <tr>
  <td>Ожидалось выражение типа (монадический тип), обнаружен тип <code>'b * 'c</code></td>
	<td><a href="#FS0001L">L. Ошибка let! в выражениях вычисления.</a></td>
  </tr>
</tbody>
</table>
{{</rawtable>}}

> ### A. Can't mix ints and floats

### A. Нельзя смешивать числа с плавающей точкой и целые числа {#FS0001A}

> Unlike C# and most imperative languages, ints and floats cannot be mixed in expressions.
> You will get a type error if you attempt this:

В отличие от C# и других императивных языков, целые и числа с плавающей точкой нельзя смешивать в выражениях. Вы получите ошибку типа, если попытаетесь сделать так:

```fsharp
1 + 2.0  // неправильно
   // => ошибка FS0001: Тип 'float' не совпадает с типом 'int'
```

> The fix is to cast the int into a `float` first:

Чтобы её исправить, сначала приведите целое к типу `float`:

```fsharp
float 1 + 2.0  // правильно
```

> This issue can also manifest itself in library functions and other places.
> For example, you cannot do "`average`" on a list of ints.

Эта проблема может проявляться при вызове библиотечных функций и в других случаях. Например, вы не можете вычислить среднее арифметическое для списка целых чисел с помощью функции "`average`".

```fsharp
[1..10] |> List.average   // неправильно
   // => ошибка FS0001: Тип 'int' не поддерживает никаких операторов с именем 'DivideByInt'
```

> You must cast each int to a float first, as shown below:

Сначала вы должны привести целые числа к типу `float`, как показано ниже:

```fsharp
[1..10] |> List.map float |> List.average  // правильно
[1..10] |> List.averageBy float  // правильно (использует averageBy)
```

> ### B. Using the wrong numeric type

### B. Использование неверного численного типа {#FS0001B}

> You will get a "not compatible" error when a numeric cast failed.

Вы получите ошибку несовместимости типов в случае некорректного приведения численного типа.

```fsharp
printfn "hello %i" 1.0  // число должно быть целым, а не с плавающим
  // ошибка FS0001: Тип float не совместим с любым из типов byte, int16, int32...
```

> One possible fix is to cast it if appropriate.

Одно из решений состоит в том, чтобы привести выражение к нужному типу самостоятельно, если это имеет смысл.

```fsharp
printfn "hello %i" (int 1.0)
```

> ### C. Passing too many arguments to a function

### C. Слишком много аргументов при вызове функции  {#FS0001C}


```fsharp
let add x y = x + y
let result = add 1 2 3
// ==> ошибка FS0001: Тип ''a -> 'b' не совместим с типом 'int'
```

> The clue is in the error.

Подсказка в сообщении об ошибке.

> The fix is to remove one of the arguments!

Чтобы её исправить, удалите один из аргументов!

> Similar errors are caused by passing too many arguments to `printf`.

Похожие ошибки возникают, если передать слишком много аргументов в `printf`.

```fsharp
printfn "hello" 42
// ==> ошибка FS0001: Ожидалось выражение типа 'a -> 'b, обнаружен тип unit

printfn "hello %i" 42 43
// ==> ошибка FS0001: Несовпадение типов. Ожидается a 'a -> 'b -> 'c, обнаружен 'a -> unit

printfn "hello %i %i" 42 43 44
// ==> ошибка FS0001: Несовпадение типов. Ожидается a  'a -> 'b -> 'c -> 'd, обнаружен 'a -> 'b -> unit
```

> ### D. Passing too few arguments to a function

### D. Слишком мало аргументов при вызове функции {#FS0001D}

> If you do not pass enough arguments to a function, you will get a partial application.
> When you later use it, you get an error because it is not a simple type.

Если вы не передали в функцию достаточно аргументов, вы получите частично применённую функцию.
Когда позже вы её используете, вы получите ошибку, потому что это функция, а не вычисленное значение.

```fsharp
let reader = new System.IO.StringReader("hello");

let line = reader.ReadLine        // неправильно, но компилятор не жалуется
printfn "The line is %s" line     // ошибка компилятора здесь!
// ==> ошибка FS0001: Ожидалось выражение типа string,
//                    обнаружен тип unit -> string
```

> This is particularly common for some .NET library functions that expect a unit parameter, such as `ReadLine` above.

Для некоторых библиотечных функций .NET это особенно характерно. Речь идёт о функциях с единственным параметром типа `unit`, как у `ReadLine` из примера выше.

> The fix is to pass the correct number of parameters.
> Check the type of the result value to make sure that it is indeed a simple type.
> In the `ReadLine` case, the fix is to pass a `()` argument.

Чтобы исправить ошибку, надо передать в функцию правильное число параметров.
Проверяйте тип получившегося значения, чтобы убедиться, что он действительно простой.
В примере с `ReadLine` надо передать значение `()` в качестве аргумента.

```fsharp
let line = reader.ReadLine()      // правильно
printfn "The line is %s" line     // ошибки нет
```


> ### E. Straightforward type mismatch

### E. Простое несоответствие типов {#FS0001E}

> The simplest case is that you have the wrong type, or you are using the wrong type in a print format string.

Простейший случай, когда вы встречаетесь с неправильным типом — строки формата печати.

```fsharp
printfn "hello %s" 1.0
// => ошибка FS0001: Ожидалось выражение типа string,
//                   обнаружен тип float
```

> ### F. Inconsistent return types in branches or matches

### F. Несогласованные типы в ветвлениях или сопоставлениях {#FS0001F}


> A common mistake is that if you have a branch or match expression, then every branch MUST return the same type.
> If not, you will get a type error.

В выражениях ветвления и сопоставления все ветки ДОЛЖНЫ иметь один и тот же тип. Если нет, вы получите ошибку.

```fsharp
let f x =
  if x > 1 then "hello"
  else 42
// => ошибка FS0001: Ожидалось выражение типа string,
//                   обнаружен тип int
```

```fsharp
let g x =
  match x with
  | 1 -> "hello"
  | _ -> 42
// => ошибка FS0001: Ожидалось выражение типа string,
//                   обнаружен тип int
```

> Obviously, the straightforward fix is to make each branch return the same type.

Очевидно, простой способ исправить эту ошибку — возвращать в каждой ветке значения одного типа.

```fsharp
let f x =
  if x > 1 then "hello"
  else "42"

let g x =
  match x with
  | 1 -> "hello"
  | _ -> "42"
```

> Remember that if an "else" branch is missing, it is assumed to return unit, so the "true" branch must also return unit.

Помните, что если ветка "else" опущена, предполагается, что она возвращает `unit`, поэтому "истинная" ветка таже должна возвращать `unit`.

```fsharp
let f x =
  if x > 1 then "hello"
// error FS0001: Ожидалось значение типа unit,
//               обнаружен тип string
```

> If both branches cannot return the same type, you may need to create a new union type that can contain both types.

Если ветви не могут иметь один и тот же тип, создайте новое объединение, которое содержит оба типа.

```fsharp
type StringOrInt = | S of string | I of int  // новое объединение
let f x =
  if x > 1 then S "hello"
  else I 42
```


> ### G. Watch out for type inference effects buried in a function

### G. Остерегайтесь вывода типов внутри функции {#FS0001G}

> A function may cause an unexpected type inference that ripples around your code.
> For example, in the following, the innocent print format string accidentally causes `doSomething` to expect a string.

Функция может повлечь за собой неожиданный вывод типа, который распространится по всему коду. В примере ниже безобидная строка формата приводит к тому, что `doSomething` ожидает строку в качестве параметра.

```fsharp
let doSomething x =
   // что-то делаем
   printfn "x is %s" x
   // делаем что-то ещё

doSomething 1
// => ошибка FS0001: Ожидалось выражение типа string,
//                   обнаружен тип int
```

> The fix is to check the function signatures and drill down until you find the guilty party.
> Also, use the most generic types possible, and avoid type annotations if possible.

Чтобы исправить эту ошибку проверяйте сигнатуры функций, спускаясь всё ниже, пока не найдёте виновницу.
Также, используйте самые общие возможные типы и избегайте аннотаций, если получится.

> ### H. Have you used a comma instead of space or semicolon?

### H. Запятая вместо пробела или точки с запятой? {#FS0001H}

> If you are new to F#, you might accidentally use a comma instead of spaces to separate function arguments:

Если вы новичок в F#, вы можете случайно разделять аргументы функций запятыми вместо пробелов:

```fsharp
// определяем функцию с двумя параметрами
let add x y = x + 1

add(x,y)   // FS0001: Ожидалось выражение типа int,
           // обнаружен тип 'a * 'b
```

> The fix is: don't use a comma!

Как исправить? Просто не используйте запятые!

```fsharp
add x y    // правильно
```

> One area where commas *are* used is when calling .NET library functions.
> These all take tuples as arguments, so the comma form is correct.
> In fact, these calls look just the same as they would from C#:

Одно из мест, где запятые *нужны* — вызов библиотечных функций .NET. Они принимают кортежи
в качестве аргументов, так что здесь запятые уместны. Фактически, эти вызовы выглядят
точно также, как и в C#.


```fsharp
// правильно
System.String.Compare("a","b")

// неправильно
System.String.Compare "a" "b"
```

> ### I. Tuples must be the same type to be compared or pattern matched

### I. Кортежи должны быть одного типа, чтобы их можно было сравнивать или сопоставлять {#FS0001I}

> Tuples with different types cannot be compared.
> Trying to compare a tuple of type `int * int`, with a tuple of type `int * string` results in an error:

Кортежи разных типов сравнивать нельзя. Результатом сравнения кортежа типа `int * int` с кортежем типа `int * string` будет ошибка:

```fsharp
let  t1 = (0, 1)
let  t2 = (0, "hello")
t1 = t2
// => ошибка FS0001: Несоответствие типов. Ожидается тип int * int,
//    обнаружен тип int * string
//    Тип 'int' не сопоставим с типом 'string'
```

> And the length must be the same:
Размер также должен быть одинаковым:

```fsharp
let  t1 = (0, 1)
let  t2 = (0, 1, "hello")
t1 = t2
// => ошибка FS0001: Несоответствие типов. Ожидается тип int * int,
//    обнаружен тип int * int * string
//    Кортежи имеют различающиеся размеры 2 и 3
```

> You can get the same issue when pattern matching tuples during binding:

Вы можете получить похожую ситуацию, привязывая значения с помощью сопоставления с образцом:

```fsharp
let x,y = 1,2,3
// => ошибка FS0001: Несоответствие типов. Ожидается тип 'a * 'b,
//                   обнаружен тип 'a * 'b * 'c
//                   Кортежи имеют различающиеся размеры 2 и 3

let f (x,y) = x + y
let z = (1,"hello")
let result = f z
// => ошибка FS0001: Несоответствие типов. Ожидается тип int * int,
//                   обнаружен тип int * string
//                   Тип 'int' не сопоставим с типом 'string'
```

> ### J. Don't use ! as the "not" operator

### J. Не используйте ! как оператор отрицания {#FS0001J}

> If you use `!` as a "not" operator, you will get a type error mentioning the word "ref".

Если вы используете `!` как оператор отрицания, вы получите ошибку соответствия типов, в котором будет встречаться слово "ref".

```fsharp
let y = true
let z = !y     // неправильно
// => ошибка FS0001: Ошидалось выражение типа 'a ref, обнаружен тип bool
```

> The fix is to use the "not" keyword instead.

Чтобы избавиться от ошибки, используйте ключевое слово "not".

```fsharp
let y = true
let z = not y   // правильно
```

> ### K. Operator precedence (especially functions and pipes)

### K. Приоритет операторов (особенно, функций и конвейеров) {#FS0001K}

> If you mix up operator precedence, you may get type errors.
> Generally, function application is highest precedence compared to other operators, so you get an error in the case below:

Если вы перепутаете приоритет операторов, у вас могут возникнуть ошибки соответствия типов.
В целом, применение функций имеет высший приоритет по сравнению с другими операторами, так что вы получите ошибку в таком коде:

```fsharp
String.length "hello" + "world"
   // => ошибка FS0001:  Тип 'string' не сопоставим с типом 'int'

// что тут действительно происходит
(String.length "hello") + "world"
```

> The fix is to use parentheses.

Чтобы избавиться от ошибки, используйте скобки.


```fsharp
String.length ("hello" + "world")  // правильно
```

> Conversely, the pipe operator is low precedence compared to other operators.

И наоборот, приоритет конвейерного оператора ниже приоритета других опреаторов.

```fsharp
let result = 42 + [1..10] |> List.sum
 // => => ошибка FS0001: Тип ''a list' не сопоставим с типом 'int'

// что тут действительно происходит
let result = (42 + [1..10]) |> List.sum
```

> Again, the fix is to use parentheses.

Снова, чтобы избавиться от ошибки, используйте скобки.

```fsharp
let result = 42 + ([1..10] |> List.sum)
```

> ### L. let! error in computation expressions (monads)

### L. Ошибка let! в вычислительных выражениях (монадах) {#FS0001L}

> Here is a simple computation expression:

Вот простое вычислительное выражение:

```fsharp
type Wrapper<'a> = Wrapped of 'a

type wrapBuilder() =
    member this.Bind (wrapper:Wrapper<'a>) (func:'a->Wrapper<'b>) =
        match wrapper with
        | Wrapped(innerThing) -> func innerThing

    member this.Return innerThing =
        Wrapped(innerThing)

let wrap = new wrapBuilder()
```

> However, if you try to use it, you get an error.

Однако, если вы попытаетесь использовать его, вы получите сообщение об ошибке.

```fsharp
wrap {
    let! x1 = Wrapped(1)   // <== ошибка здесь
    let! y1 = Wrapped(2)
    let z1 = x + y
    return z
    }
// error FS0001: Ожидалось выражения типа Wrapper<'a>,
//               обнаружен тип 'b * 'c
```

> The reason is that "`Bind`" expects a tuple `(wrapper,func)`, not two parameters.
> (Check the signature for bind in the F# documentation).

Причина в том, что "`Bind`" ожидает кортеж `(wrapper,func)`, а не два отдельных параметра.
(Проверьте сигнатуру метода в документации F#).

> The fix is to change the bind function to accept a tuple as its (single) parameter.

Чтобы избавиться от ошибки, измените метод так, чтобы он принимал кортеж в качестве (единственного) параметра.

```fsharp
type wrapBuilder() =
    member this.Bind (wrapper:Wrapper<'a>, func:'a->Wrapper<'b>) =
        match wrapper with
        | Wrapped(innerThing) -> func innerThing
```

> ## FS0003: This value is not a function and cannot be applied

## FS0003: Значение не является функцией и не может быть применено {#FS0003}

> This error typically occurs when passing too many arguments to a function.

Эта ошибка обычно возникает, если в функцию передано слишком много параметров.

```fsharp
let add1 x = x + 1
let x = add1 2 3
// ==>   ошибка FS0003: Значение не является функцией и не может быть применено
```

> It can also occur when you do operator overloading, but the operators cannot be used as prefix or infix.

Она также может возникнуть при перегрузке оператора, когда оператор нельзя использовать в префиксной или инфиксной форме.

```fsharp
let (!!) x y = x + y
(!!) 1 2              // правильно
1 !! 2                // неправильно — !! нельзя использовать в инфиксной форме
// error FS0003: Значение не является функцией и не может быть применено
```

> ## FS0008: This runtime coercion or type test involves an indeterminate type

## FS0008: Приведение во время выполнения или сопоставление типа приводит к неопределяемому типу {#FS0008}

> You will often see this when attempting to use "`:?`" operator to match on a type.

Вы часто будете видеть это сообщение, пытаясь использовать оператор "`:?`" для сопоставления типа.


```fsharp
let detectType v =
    match v with
        | :? int -> printfn "this is an int"
        | _ -> printfn "something else"
// ошибка FS0008: Приведение типа времени выполнения или сопоставление типа `a с int
// приводит к неопределяемому типу, если опираться на информацию, доступную в данной точке программы.
// Сопоставление некоторых типов во время выполнения невозможно. Нужна дополнительная аннотация типа.
```

> The message tells you the problem: "runtime type tests are not allowed on some types".

Сообщение говорит вам о проблеме: "сопоставление некоторых типов во время выполнения невозможно".

> The answer is to "box" the value which forces it into a reference type, and then you can type check it:

Решение состоит в том, чтобы "упаковать" значение, что позволит обращаться с ним, как с другими ссылочными объектами, в частности, проверить его тип:

```fsharp
let detectTypeBoxed v =
    match box v with      // используем "упакованное значение v"
        | :? int -> printfn "this is an int"
        | _ -> printfn "something else"

// проверяем
detectTypeBoxed 1
detectTypeBoxed 3.14
```


> ## FS0010: Unexpected identifier in binding

## FS0010: Неожиданный идентификатор в привязке {#FS0010a}

> Typically caused by breaking the "offside" rule for aligning expressions in a block.

Обычно ошибка возникает из-за неправильного выравнивания выражений в блоке.

<!-- В оригинале используется термин "правило офсайда", которое в русском языке не встречается.
Термин ввёл Питер Лендин в статье 1966 года "Следующие 700 языков программирования".

Формулировка: Any non-whitespace token to the left of the first such token on the previous line is taken to be the start of a new declaration.

Правило определяет языки, где блоки кода определяются отсутпами, а не скобками или ключевыми словами, такими как begin и end.

Перевод правила на русский: Любой токен, не состоящий из пробелов, который находится слева от первого такого же токена в предыдущей строке кода, начинает новую декларацию.

Довольно запутанно. Возможно, похоже на определение офсайда в футболе. -->

```fsharp
//3456789
let f =
  let x=1     // новый отступ в столбце 3
   x+1        // ой! не делайте отступ в столбце 4
              // ошибка FS0010: offside line is at column 3
```

> The fix is to align the code correctly!

Чтобы избавиться от ошибки, выравнивайте код корректно!

> See also [FS0588: Block following this 'let' is unfinished](#FS0588) for another issue caused by alignment.

Также смотрите [FS0588: Блок, следующий за оператором 'let', не завершён](#FS0588), где описана другая ошибка, вызыванная некорректным выравниванием.

> ## FS0010: Incomplete structured construct

## FS0010: Не полностью структурированная конструкция {#FS0010b}

> Often occurs if you are missing parentheses from a class constructor:

Ошибка "Не полностью структурированная конструкция" обычно означает, что вы пропустили скобки при вызове конструктора.

```fsharp
type Something() =
   let field = ()

let x1 = new Something     // ошибка FS0010
let x2 = new Something()   // правильно
```

> Can also occur if you forgot to put parentheses around an operator:

Может также возникнуть, если вы забыли заключить оператор в круглые скобки:

```fsharp
// определяем новый оператор
let (|+) a = -a

|+ 1    // ошибка FS0010:
        // Неожиданный инфиксный оператор

(|+) 1  // Со скобками — всё хорошо!
```

> Can also occur if you are missing one side of an infix operator:

Может также возникнуть, если вы забыли записать один из операндов инфиксного оператора:

```fsharp
|| true  // ошибка FS0010: Неожиданный символ '||'
false || true  // Всё хорошо
```

> Can also occur if you attempt to send a namespace definition to F# interactive.
> The interactive console does not allow namespaces.

Может также возникнуть, если вы пытаетесь задать пространство имён в F# Interactive.
Интерактивная консоль не понимает пространства имён.

```fsharp
namespace Customer  // FS0010: Не полностью структурированная конструкция

// определяем тип
type Person= {First:string; Last:string}
```


> ## FS0013: The static coercion from type X to Y involves an indeterminate type

> ## FS0013: Статическое приведение типа X к типу Y приводит к неопределённому типу {#FS0013}


> This is generally caused by implic

<!-- Раздел не написан в оригинале. Надо, видимо, указать на него Скотту. -->

> ## FS0020: This expression should have type 'unit'

> ## FS0020: Выражение должно иметь тип 'unit' {#FS0020}

> This error is commonly found in two situations:

Эта ошибка обычно возникает в двух случаях:

> * Expressions that are not the last expression in the block
> * Using wrong assignment operator

* Выражения, которые не являются последними выражениями в блоке
* Использование неверного оператора присваивания

> ### FS0020 with expressions that are not the last expression in the block ###

> ### FS0020 из-за выражений, которые не являются последними выражениями в блоке ###

> Only the last expression in a block can return a value.
> All others must return unit.
> So this typically occurs when you have a function in a place that is not the last function.

Возвращать значение может только последнее выражение в блоке. Все остальные должны возвращать значение типа `unit`. Так что ошибка обычно возникает, когда у вас есть функция там в какой-то из строк, кроме последней.

```fsharp
let something =
  2+2               // => FS0020: Это выражение должно иметь тип 'unit'
  "hello"
```

> The easy fix is use `ignore`.
> But ask yourself why you are using a function and then throwing away the answer -- it might be a bug.

Простое исправление заключается в вызове функции `ignore`. Однако, спросите себя, почему вы используете функции, а потом игнорируете её результат — возможно, это ошибка.

```fsharp
let something =
  2+2 |> ignore     // правильно
  "hello"
```

> This also occurs if you think you writing C# and you accidentally use semicolons to separate expressions:

Кроме того, эта ошибка возникает, если вы по привычке из C# случайно используете точку с запятой, чтобы разделять выражения:

```fsharp
// неправильно
let result = 2+2; "hello";

// правильно
let result = 2+2 |> ignore; "hello";
```

> ### FS0020 with assignment ###

> ### FS0020 из-за присваивания ###

> Another variant of this error occurs when assigning to a property.

Другой вариант этой ошибки возникает, когда вы присваивате значение свойству.

>     This expression should have type 'unit', but has type 'Y'.

     Это выражение должно иметь тип 'unit', но имеет тип 'Y'.

> With this error, chances are you have confused the assignment operator "`<-`" for mutable values, with the equality comparison operator "`=`".

В этой ошибке, скорее всего, вы перепутали оператор присваивания "`<-`" для изменяемых значений с оператором сравнения "`=`".

```fsharp
// '=' вместо '<-'
let add() =
    let mutable x = 1
    x = x + 1          // предупреждение FS0020
    printfn "%d" x
```

> The fix is to use the proper assignment operator.

Чтобы избавиться от ошибки, используйте правильный оператор присваивания.

```fsharp
// исправлено
let add() =
    let mutable x = 1
    x <- x + 1
    printfn "%d" x
```


> ## FS0030: Value restriction

> ## FS0030: Ограничение на значение {#FS0030}

> This is related to F#'s automatic generalization to generic types whenever possible.

Эта ошибка связана с автоматическим расширением до обобщённого типа, когда это возможно.

> For example, given:

Например, в примерах:

```fsharp
let id x = x
let compose f g x = g (f x)
let opt = None
```

> F#'s type inference will cleverly figure out the generic types.

Механизм вывода типов F# проявит смекалку и определит обобщённые типы.

```fsharp
val id : 'a -> 'a
val compose : ('a -> 'b) -> ('b -> 'c) -> 'a -> 'c
val opt : 'a option
```

> However in some cases, the F# compiler feels that the code is ambiguous, and, even though it looks like it is guessing the type correctly, it needs you to be more specific:

Но в некоторых случаях компилятор F# посчитает код неоднозначным и, хотя кажется, что он правильно угадывает тип, ему нужно, чтобы вы были более конкретными.

```fsharp
let idMap = List.map id             // ошибка FS0030
let blankConcat = String.concat ""  // ошибка FS0030
```

> Almost always this will be caused by trying to define a partially applied function, and almost always, the easiest fix is to explicitly add the missing parameter:

Почти всегда ошибка будет вызвана попыткой определить частично применённую функцию, и почти всегда простейшее исправление заключается в явном добавлении недостающего параметра.

```fsharp
let idMap list = List.map id list             // правильно
let blankConcat list = String.concat "" list  // правильно
```

> For more details see the MSDN article on ["automatic generalization"](http://msdn.microsoft.com/en-us/library/dd233183%28v=VS.100%29.aspx).

Больше деталей вы найдёте в статье MSDN, посвящённой ["автоматическому обобщению"](http://msdn.microsoft.com/en-us/library/dd233183%28v=VS.100%29.aspx).

> ## FS0035: This construct is deprecated

## FS0035: Эта конструкция устарела {#FS0035}

> F# syntax has been cleaned up over the last few years, so if you are using examples from an older F# book or webpage, you may run into this.
> See the MSDN documentation for the correct syntax.

Синтаксис F# за последние несколько лет сильно упростился, так что если вы используете примеры из старой книги или старой страницы, посвящённой F#, вы можете столкнуться с такой ошибкой.

```fsharp
let x = 10
let rnd1 = System.Random x         // хорошо
let rnd2 = new System.Random(x)    // хорошо
let rnd3 = new System.Random x     // ошибка FS0035
```

> ## FS0039: The field, constructor or member X is not defined

## FS0039: Поле, конструктор или член X не определён {#FS0039}

> This error is commonly found in four situations:

Эта ошибка обычно возникает в одной из четырёх ситуаций.

> * The obvious case where something really isn't defined! And make sure that you don't have a typo or case mismatch either.
> * Interfaces
> * Recursion
> * Extension methods

* Очевидный случай, когда что-то действительно не определено! И убедитесь, что у вас нет опечатки, и вы не перепутали большие и маленькие буквы.
* Интерфейсы
* Рекурсия
* Методы расширения

> ### FS0039 with interfaces ###

> ### FS0039 из-за интерфейсов ###

> In F# all interfaces are "explicit" implementations rather than "implicit".
> (Read the C# documentation on ["explicit interface implementation"](http://msdn.microsoft.com/en-us/library/aa288461%28v=vs.71%29.aspx) for an explanation of the difference).

В F# все интерфейсы должны быть реализованы "явно", в отличие от "неявной" реализации, принятой в C# (см. раздел ["явная реализация интерфейса"](http://msdn.microsoft.com/en-us/library/aa288461%28v=vs.71%29.aspx) где разъясняется отличие).

> The key point is that when a interface member is explicitly implemented, it cannot be accessed through a normal class instance, but only through an instance of the interface, so you have to cast to the interface type by using the `:>` operator.

Суть в том, что когда элемент интерфейса реализован явно, к нему нельзя получить доступ через экземпляр класса, только через экземпляр интерфейса, поэтому вы должны привести объект к типу интерфейса с помощью оператора `:>`.

> Here's an example of a class that implements an interface:

Вот примера класса, который реализует интерфейс.

```fsharp
type MyResource() =
   interface System.IDisposable with
       member this.Dispose() = printfn "disposed"
```

> This doesn't work:

Этот код не работает:

```fsharp
let x = new MyResource()
x.Dispose()  // ошибка FS0039: Поле, конструктор
             // или член 'Dispose' не определён
```

> The fix is to cast the object to the interface, as below:

Чтобы избавиться от ошибки, приведите объект к типу интерфейса, как показано ниже:

```fsharp
// исправлено приведением к System.IDisposable
(x :> System.IDisposable).Dispose()   // правильно

let y =  new MyResource() :> System.IDisposable
y.Dispose()   // правильно
```


> ### FS0039 with recursion ###

> ### FS0039 из-за рекурсии ###

> Here's a standard Fibonacci implementation:

Вот стандартная реализация чисел Фибоначчи:

```fsharp
let fib i =
   match i with
   | 1 -> 1
   | 2 -> 1
   | n -> fib(n-1) + fib(n-2)
```

> Unfortunately, this will not compile:

К сожалению, этот код компилироваться не будет:

>    Error FS0039: The value or constructor 'fib' is not defined

    Ошибка FS0039: Значение или конструктор 'fib' не определён

> The reason is that when the compiler sees 'fib' in the body, it doesn't know about the function because it hasn't finished compiling it yet!

Причина в том, что когда компилятор видит вызов 'fib', он не знает о такой функции, потому что он её ещё не скомпилировал!

> The fix is to use the "`rec`" keyword.

Чтобы избавиться от ошибки, используйте ключевое слово "`rec`".

```fsharp
let rec fib i =
   match i with
   | 1 -> 1
   | 2 -> 1
   | n -> fib(n-1) + fib(n-2)
```

> Note that this only applies to "`let`" functions.
> Member functions do not need this, because the scope rules are slightly different.

Обратите внимание, что оно нужно только при описании функций с помощью "`let`".
Функции-члены в нём не нуждаются, поскольку у них другие правила, касающиеся области видимости.

```fsharp
type FibHelper() =
    member this.fib i =
       match i with
       | 1 -> 1
       | 2 -> 1
       | n -> fib(n-1) + fib(n-2)
```

> ### FS0039 with extension methods ###

> ### FS0039 из-за методов расширения ###

> If you have defined an extension method, you won't be able to use it unless the module is in scope.

Если вы определили метод расширения, вы не сможете его использовать до тех пор, пока не подключите соответствующий модуль.

> Here's a simple extension to demonstrate:

Вот простой пример для демонстрации:

```fsharp
module IntExtensions =
    type System.Int32 with
        member this.IsEven = this % 2 = 0
```

> If you try to use it the extension, you get the FS0039 error:

Если вы попытаетесь использовать метод расширения, вы получите ошибку FS0039:

```fsharp
let i = 2
let result = i.IsEven
    // FS0039: Поле, конструктор или
    // член 'IsEven' не определён
```

> The fix is just to open the `IntExtensions` module.

Чтобы избавиться от ошибки, достаточно подключить модуль `IntExtensions`.

```fsharp
open IntExtensions // вводим методы модуля в область видимости
let i = 2
let result = i.IsEven  // исправлено!
```

> ## FS0041: A unique overload for could not be determined

## FS0041: Невозможно определить уникальную перегрузку {#FS0041}

> This can be caused when calling a .NET library function that has multiple overloads:

Эта ошибка может возникнуть при вызове библиотечной функции .NET с несколькими перегрузками:

```fsharp
let streamReader filename = new System.IO.StreamReader(filename) // FS0041
```

> There a number of ways to fix this. One way is to use an explicit type annotation:

Есть несколько способов избавиться от ошибки. Во-первых, вы можете использовать аннотации типов:

```fsharp
let streamReader filename = new System.IO.StreamReader(filename:string) // работает
```

> You can sometimes use a named parameter to avoid the type annotation:

Иногда вы можете использовать именованные параметры, чтобы избежать аннотации:

```fsharp
let streamReader filename = new System.IO.StreamReader(path=filename) // работает
```

> Or you can try to create intermediate objects that help the type inference, again without needing type annotations:

Или вы можете попытаться создать промежуточные объекты, которые помогают выводу типов и также избавляют нас от аннотаций:

```fsharp
let streamReader filename =
    let fileInfo = System.IO.FileInfo(filename)
    new System.IO.StreamReader(fileInfo.FullName) // работает
```

> ## FS0049: Uppercase variable identifiers should not generally be used in patterns

> ## FS0049: В образцах нельзя использовать идентификаторы переменных в верхнем регистре {#FS0049}

> When pattern matching, be aware of a subtle difference between the pure F# union types which consist of a tag only, and a .NET Enum type.

При сопоставлении с образцом не забывайте о тонкой разнице между размеченными объединениями F#, которые содержат только идентификаторы вариантов, и типом `Enum` из .NET.

> Pure F# union type:

Размеченные объединения из F#:

```fsharp
type ColorUnion = Red | Yellow
let redUnion = Red

match redUnion with
| Red -> printfn "red"     // без проблем
| _ -> printfn "something else"
```

> But with .NET enums you must fully qualify them:

Но перечисления .NET требуют указания полного имени:

```fsharp
type ColorEnum = Green=0 | Blue=1      // перечисление
let blueEnum = ColorEnum.Blue

match blueEnum with
| Blue -> printfn "blue"     // предупреждение FS0049
| _ -> printfn "something else"
```

> The fixed version:

Исправленная версия:

```fsharp
match blueEnum with
| ColorEnum.Blue -> printfn "blue"
| _ -> printfn "something else"
```

> ## FS0072: Lookup on object of indeterminate type

## FS0072: Поиск объекта неопределённого типа {#FS0072}

> This occurs when "dotting into" an object whose type is unknown.

Ошибка возникает, когда вы пытаетесь обратиться к свойству объекта, чей тип неизвестен.

> Consider the following example:

Рассмотрим следующий пример:

```fsharp
let stringLength x = x.Length // ошибка FS0072
```

> The compiler does not know what type "x" is, and therefore does not know if "`Length`" is a valid method.

Компилятор не знает типа "x" и поэтому не может проверить, что "`Length`" является правильным свойством.

<!-- Здесь речь идёт о свойстве, а не о методе. Подозреваю, метод оставлся после переписывания, так что просто исправил слово в переводе. -->

> There a number of ways to fix this.
> The crudest way is to provide an explicit type annotation:

Есть несколько способов избавиться от этой ошибки. Метод "в лоб" — предоставить явную аннотацию типа:

```fsharp
let stringLength (x:string) = x.Length  // правильно
```

> In some cases though, judicious rearrangement of the code can help.
> For example, the example below looks like it should work.
> It's obvious to a human that the `List.map` function is being applied to a list of strings, so why does `x.Length` cause an error?

В некоторых случаях может помочь разумная реорганизация кода.
Скажем, пример ниже, кажется, должен работать.
Программисту очевидно, что функция `List.map` применяется к списку строк, так почему `x.Length` приводит к ошибке?

```fsharp
List.map (fun x -> x.Length) ["hello"; "world"] // ошибка FS0072
```

> The reason is that the F# compiler is currently a one-pass compiler, and so type information present later in the program cannot be used if it hasn't been parsed yet.

Причина заключается в том, что компилятор F# — однопроходный, так что информация о типе, представленная в программе позже, ему не доступна — ведь он о ней ещё не знает.

> Yes, you can always explicitly annotate:

Да, вы всегда можете добавить явную аннотацию:

```fsharp
List.map (fun x:string -> x.Length) ["hello"; "world"] // работает
```

> But another, more elegant way that will often fix the problem is to rearrange things so the known types come first, and the compiler can digest them before it moves to the next clause.

Но есть и другой, более элегантный способ решения проблемы. Надо реорганизовать код так, чтобы типы шли в начале, и комплиятор узнал про них прежде, чем разбирать следующую конструкцию.

```fsharp
["hello"; "world"] |> List.map (fun x -> x.Length)   // работает
```

> It's good practice to avoid explicit type annotations, so this approach is best, if it is feasible.

Считается хорошей практикой избегать явных аннотаций, так что применяйте этот подход всякий раз, когда получается.

> ## FS0588: Block following this 'let' is unfinished

> ## FS0588: Блок, следующий за оператором 'let', не завершён {#FS0588}

> Caused by outdenting an expression in a block, and thus breaking the "offside rule".

Ошибка вызвана неверным выравниваем кода в блоке.

<!-- Выше есть комментарий про "правило офсайда" (правило вне игры). Этот термин не имеет
устойчивого перевода в русском языке.

Предлагаю в переводе вместо нарушения правила говорить о неверных отступах или неверном выравнивании кода. -->

```fsharp
//3456789
let f =
  let x=1    // отступ в 3 пробела
 x+1         // проблема! отступ в 2 пробела!
             // Ошибка FS0588: Блок, следующий за оператором
             // 'let', не завершён
```

> The fix is to align the code correctly.

Чтобы изабвиться от ошибки, выровняйте код правильно.

> See also [FS0010: Unexpected identifier in binding](#FS0010a) for another issue caused by alignment.

Смотрите также раздел [FS0010: Неожиданный идентификатор в привязке](#FS0010a), где рассказано о другой ошибке, вызыванной неправильным выравниваем.