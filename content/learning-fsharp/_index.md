---
layout: page
title: "Изучая F#"
description: "Функциональные языки программирования требуют другого подхода"
nav: learning-fsharp
hasComments: 1
date: 2015-01-01
---

>Functional languages are very different from standard imperative languages, and can be quite tricky to get the hang of initially.

Функциональные языки сильно отличаются от стандартных императивных языков, и на начальном этапе их довольно сложно освоить.

>This page offers some tips on how to learn F# effectively.

Эта страница предлагает небольшие подсказки, как учить F# эффективно.

>## Approach learning F# as a beginner ##
## Подходите к изучению F# как новичок ##

>If you have experience in languages such as C# and Java, you have probably found that you can get a pretty good understanding of source code written in other similar languages, even if you aren't familiar with the keywords or the libraries. 

Если у вас есть опыт в таких языках как C# и Java, вы скорее всего заметили, что можете довольно хорошо понимать исходный код, написанный на других похожих языках, даже если вы не знакомы с ключевыми словами или библиотеками.

>This is because all imperative languages use the same way of thinking, and experience in one language can be easily transferred to another.

Это потому, что все императивные языки используют одинаковый способ мышления, и опыт использования одного языка может быть легко перенесён на другой язык.

>If you are like many people, your standard approach to learning a new programming language is to find out how to implement concepts you are already familiar with.

Если вы как большинство людей, ваш стандартный подход к изучению нового языка программирования заключается в поиске способов реализации концепций, которые вам близки.

>You might ask "how do I assign a variable?" or "how do I do a loop?", and with these answers be able to do some basic programming quite quickly.

Вы можете задавать вопросы "Как присвоить переменную?" или "как мне сделать цикл?", и с полученными ответами будете иметь возможность (у вас будет возможность) быстро программировать на базовом уровне.

>When learning F#, **you should not try to bring your old imperative concepts with you**. 

Изучая F#, **вам не стоит нести с собой ваши старые императивные концепции с собой**. 

>In a pure functional language there are no variables, there are no loops, and there are no objects!

В чистых функциональных языках нет переменных, нет циклов, и даже нет объектов!

>Yes, F# is a hybrid language and does support these concepts. 

Да, F# является гибридным языком и поддерживает эти концепции. 

>But you will learn much faster if you start with a beginners mind.

Но вы получите понимание намного быстрее (быстрее поймёте) если вы начнёте с чистого листа.

> ## Change the way you think ##

## Измените свой способ мышления ## 

>It is important to understand that functional programming is not just a stylistic difference; it is a completely different way of thinking about programming, in the way that truly object-oriented programming (in Smalltalk say) is also a different way of thinking from a traditional imperative language such as C.

Важно понимать, что функциональное программирование - не просто стилистическое отличие; это совершенно иной способ мышления о программировании, подобно тому, как настоящее объектно-ориентированное программирование (скажем, в Smalltalk) также отличается от традиционного императивного языка, такого как C.

>F# does allow non-functional styles, and it is tempting to retain the habits you already are familiar with.

F# позволяет использование нефункциональных подходов, и возникает соблазн сохранить уже знакомые привычки.

>You could just use F# in a non-functional way without really changing your mindset, and not realize what you are missing.

Вы можете просто использовать F# в нефункциональном стиле без изменения своего образа мышления и не понимать, чего вам не хватает. 


>To get the most out of F#, and to be fluent and comfortable with functional programming in general, it is critical that you think functionally, not imperatively.

Для того чтобы получить максимальную пользу от F# и свободно владеть функциональным программированием в целом, критически важно мыслить функционально, а не императивно.

> By far the most important thing you can do is to take the time and effort to understand exactly how F# works, especially the core concepts involving functions and the type system.

Безусловно, самой важной целью которую вы можете сделать - потратить время и усилия на то чтобы понять как 
именно работает F#, особенно ключевые идеи, затрагивающие функции и систему типов. 

>So please read and reread the series ["thinking functionally"](/series/thinking-functionally.html) and ["understanding F# types"](/series/understanding-fsharp-types.html), play with the examples, and get comfortable with the ideas before you try to start doing serious coding.

Так что, пожалуйста, прочтите и перечитайте серию ["думать функционально"](/series/thinking-functionally.html) and ["понимание F# типов"](/series/understanding-fsharp-types.html), поиграйтесь с примерами, и освойтесь с идеями прежде чем приступать к серьёзному кодированию. 

>If you don't understand how functions and types work, then you will have a hard time being productive.

Если вы не понимаете как работают функции и типы, тогда вам будет тяжело стать продуктивным.

> ## Dos and Don'ts ##
## Что можно и что нельзя ##

> Here is a list of dos and don'ts that will encourage you to think functionally. 

Вот список того что можно делать и что нельзя, который воодушевит вас думать функционально

> These will be hard at first, but just like learning a foreign language, you have to dive in and force yourself to speak like the locals.

Поначалу это будет сложно, но как и в изучении иностранного языка, вам необходимо броситься в омут с головой и заставить себя говорить как местные.

>* Don't use the `mutable` keyword **at all** as a beginner. 

* Не используйте ключевое слово `mutable` **совсем** на начальном этапе.

>Coding complex functions without the crutch of mutable state will really force you to understand the functional paradigm.

Кодирование сложны фунций без "костылей" изменяемого состояния действительно заставит вас понять функциональную парадигму.

>* Don't use `for` loops or `if-then-else`. 

* Не используйте циклы `for` или `if-then-else`. 

>Use pattern matching for testing booleans and recursing through lists.

Используйте сопоставление с образцом для булевых значений и рекурсивный просмотр списков.

>* Don't use "dot notation". 

* Не используйте точечную нотацию.

>Instead of "dotting into" objects, try to use functions for everything.

Вместо того чтобы "расставлять точки" в объектах, попробуйте везде использовать функции.

>That is, write `String.length "hello"` rather than `"hello".Length`.

То есть, пишите `String.length "hello"` вместо `"hello".Length`

>It might seem like extra work, but this way of working is essential when using pipes and higher order functions like `List.map`.

Это может показаться лишней работой, но этот метод работы (подход) естественнен при использовании конвейеров и функций высшего порядка, таких как `List.map`.

>And don't write your own methods either!

И не пишите свои собственные методы

>See [this post for details](/posts/type-extensions/#downsides-of-methods).

Посмотри [этот пост для подробностей](/posts/type-extensions/#downsides-of-methods).

>* As a corollary, don't create classes.

* Как следствие, не создавайте классы.

>Use only the pure F# types such as tuples, records and unions.

Используйте только чистые F# типы, такие как кортежи, записи и объединения.

>* Don't use the debugger.

* Не используйте отладчик.

>If you have relied on the debugger to find and fix incorrect code, you will get a nasty shock.

Если вы полагались на использование отладчика для нахождения и исправление некорректного кода, вы получите неприятный шок.

>In F#, you will probably not get that far, because the compiler is so much stricter in many ways.

В F#, вы скорее всего не зайдёте так далеко, потому что компилятор гораздо более строгий во многих аспектах.

>And of course, there is no tool to "debug" the compiler and step through its processing. 

И конечно, нет инструмента для "отладки" компилятора и пошагового выполнения его обработки.

>The best tool for debugging compiler errors is your brain, and F# forces you to use it!

Лучшим инструментом для отладки является ваш мозг, и F# заставляет вас его использовать!

>On the other hand:

С другой стороны:

>* Do create lots of "little types", especially union types.

* Создавайте множество "маленьких типов", особенно типов объединения.

>They are lightweight and easy, and their use will help document your domain model and ensure correctness.

Они легковесны и просты, а их использование поможет в документировании вашей модели предметной области и обеспечении корректности.

>* Do understand the `list` and `seq` types and their associated library modules.

* Поймите типы `list` и `seq`, а также ассоциируемые с ними библиотечные модули.

>Functions like `List.fold` and `List.map` are very powerful.

Фунции наподобие `List.fold` и `List.map` являются очень мощными.

>Once you understand how to use them, you will be well on your way to understanding higher order functions in general.

Как только вы поймёте как их использовать, вы будете на правильном пути к понимаю функций высшего порядка в целом.

>* Once you understand the collection modules, try to avoid recursion.

* Как только вы поймёте модули коллекций, попытайтесь избегать рекурсии.

>Recursion can be error prone, and it can be hard to make sure that it is properly tail-recursive.

Рекурсия может быть склонна ошибкам, и может быть сложно удостовериться что она является правильной хвостовой рекурсией.


>When you use `List.fold`, you can never have that problem.

При использовании `List.fold` вы никогда не столкнётесь с этой проблемой.

>* Do use pipe (`|>`) and composition (`>>`) as much as you can.

* Используйте конвейер (`|>`) и композицию (`>>`) настолько часто, насколько возможно.

>This style is much more idiomatic than nested function calls like `f(g(x))`

Этот стиль является более идиоматическим, чем вложенные вызовы функций, наподобие `f(g(x))`.

>* Do understand how partial application works, and try to become comfortable with point-free (tacit) style.

* Поймите как работает частичное применение, и постарайтесь привыкнуть к бесточечному (неявному) стилю.

>* Do develop code incrementally, using the interactive window to test code fragments.

* Разрабатывайте код постепенно, используя интерактивное окно для тестирования фрагментов кода.

>If you blindly create lots of code and then try to compile it all at once, you may end up with many painful and hard-to-debug compilation errors.

Если вы слепо пишите большое количество кода, а затем пытаетесь его скомпилировать, вы можете оказаться в куче болезенных и трудно отлаживаемых ошибках компиляции.

>## Troubleshooting ##

## Устранение неполадок ##

>There are a number of extremely common errors that beginners make, and if you are frustrated about getting your code to compile, please read the ["troubleshooting F#"](/troubleshooting-fsharp/) page.

Есть список чрезвычайно распространённых ошибок совершаемых новичками, и если вы расстроены попытками заставить свой код компилироваться, пожалуйста прочтите ["устранение неполадок F#"](/troubleshooting-fsharp/).
