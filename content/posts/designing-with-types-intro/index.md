---
layout: post
# title: "Designing with types: Introduction"
title: "Проектирование с помощью типов: Введение"
# description: "Making design more transparent and improving correctness"
description: "Делая дизайн более прозрачным и увеличивая коректность"
date: 2013-01-12
nav: thinking-functionally
seriesId: "Проектирование с помощью типов"
seriesOrder: 1
categories: [Types, DDD]
---

> In this series, we'll look at some of the ways we can use types as part of the design process.
> In particular, the thoughtful use of types can make a design more transparent and improve correctness at the same time.

В этом цикле мы взглянем на некоторые способы, которые мы можем использовать типы как часть процесса проектирования.
В частности, вдумчивое использование типов может сделать дизайн более прозрачным и увеличить коректность одновременно.

> This series will be focused on the "micro level" of design.
> That is, working at the lowest level of individual types and functions.
> Higher level design approaches, and the associated decisions about using functional or object-oriented style, will be discussed in another series.

Этот цикл будет сконцентрирован на «микро уровне» проектирования.
То есть работая на низшем уровне отдельных типов и функций.
Подходы к проектированию более высокого уровня и связанные с ними решения об использовании функционального или объекто-ориентированного стиля, будут обсуждаться в других циклах.

> Many of the suggestions are also feasible in C# or Java, but the lightweight nature of F# types means that it is much more likely that we will do this kind of refactoring.

Многие предложения осуществимы также в C# и Java, но легковесная природа типов F# делает гораздо более вероятным такого рода рефакторинг.

> ## A basic example ##

## Основной пример

> For demonstration of the various uses of types, I'll work with a very straightforward example, namely a `Contact` type, such as the one below.

Для демонстрации различных вариантов использования типов, я буду работать с очень простым примером, а именно с типом `Contact` (контакт), таким как приведённый ниже.

```fsharp
type Contact =
    {
    FirstName: string;
    MiddleInitial: string;
    LastName: string;

    EmailAddress: string;
    //true if ownership of email address is confirmed
    //истина, если электронный адрес подтверждён
    IsEmailVerified: bool;

    Address1: string;
    Address2: string;
    City: string;
    State: string;
    Zip: string;
    //true if validated against address service
    //истина, если адрес проверен внешней службой проверки адресов
    IsAddressValid: bool;
    }

```

> This seems very obvious -- I'm sure we have all seen something like this many times.
> So what can we do with it?
> How can we refactor this to make the most of the type system?

Это кажется совершенно очевидным — я уверен, мы все много раз видели что-то подобное.
Так что мы можем с этим делать?
Как мы можем рефакторить это, чтобы получить максимум от системы типов?

> ## Creating "atomic" types ##

## Создавая «атомарные» типы

> The first thing to do is to look at the usage pattern of data access and updates.
> For example, would be it be likely that `Zip` is updated without also updating `Address1` at the same time?
> On the other hand, it might be common that a transaction updates `EmailAddress` but not `FirstName`.

Первое, что необходимо сделать — исследовать паттерн использования доступа к данным и обновлений.
Например, можно ли обновить поле `Zip` (почтовый индекс) без одновременного обновления `Address1` (адрес)?
С другой стороны, вполне возможна ситуация, когда транзакция обновляет `EmailAddress` (адрес электронной почты), но не `FirstName` (имя).

> This leads to the first guideline:

Это приводит нас к первому правилу:

> * *Guideline: Use records or tuples to group together data that are required to be consistent (that is "atomic") but don't needlessly group together data that is not related.*

* *Правило: Используйте записи или кортежи, чтобы группировать вместе данные, которые должны быть согласованными (то есть «атомарными»), но не группируйте без необходимости данные, которые не связаны между собой.

> In this case, it is fairly obvious that the three name values are a set, the address values are a set, and the email is also a set.

В данном случае довольно очевидно, что три части именни образуют набор, части адреса образуют набор и электронный адрес также образует набор.

> We have also some extra flags here, such as `IsAddressValid` and `IsEmailVerified`.
> Should these be part of the related set or not?
> Certainly yes for now, because the flags are dependent on the related values.

У нас здесь также есть несколько дополнительных флагов, таких как `IsAddressValid` (является ли адрес правильным?) и `IsEmailVerified` (проверен ли электронный адрес?).
Должны ли они быть частью связанных наборов или нет?
В данном случае, конечно, да, поскольку флаги зависят от значений, к которым они относятся.

> For example, if the `EmailAddress` changes, then `IsEmailVerified` probably needs to be reset to false at the same time.

Например, если меняется `EmailAddress`, тогда, возможно, одновременно стоит сбросить значение флага `IsEmailVerified`.

> For `PostalAddress`, it seems clear that the core "address" part is a useful common type, without the `IsAddressValid` flag.
> On the other hand, the `IsAddressValid` is associated with the address, and will be updated when it changes.

Для <!-- поля -->  `PostalAddress` (почтовый адрес) очевидно, что «адрес» сам по себе является полезным общим типом, без флага `IsAddressFlag`.
С другой стороны, флаг `IsAddressValid` ассоциируется с адресом и должен обновляться, когда арес меняется.

> So it seems that we should create *two* types.
> One is a generic `PostalAddress` and the other is an address in the context of a contact, which we can call `PostalContactInfo`, say.

Поэтому, похоже, нам следует создать *два* типа.
Один — это `PostalAddress` вообще, и второй — адрес в контексте контакта, который, например, можно назвать `PostalContactInfo` (информация о почтовом адресе).


```fsharp
type PostalAddress =
    {
    Address1: string;
    Address2: string;
    City: string;
    State: string;
    Zip: string;
    }

type PostalContactInfo =
    {
    Address: PostalAddress;
    IsAddressValid: bool;
    }
```

> Finally, we can use the option type to signal that certain values, such as `MiddleInitial`, are indeed optional.

В конце мы можем использовать тип `option`, чтобы показать, что некоторые значения, такие как `MiddleInitial` (второе имя) на самом деле необязательные.

```fsharp
type PersonalName =
    {
    FirstName: string;
    // use "option" to signal optionality
    // используем "option" чтобы показать, что это поле необязательное
    MiddleInitial: string option;
    LastName: string;
    }
```

> ## Summary

## Заключение

> With all these changes, we now have the following code:

Со всеми этими изменениями мы получили такой код:

```fsharp
type PersonalName =
    {
    FirstName: string;
    // use "option" to signal optionality
    // используем "option" чтобы показать, что это поле необязательное
    MiddleInitial: string option;
    LastName: string;
    }

type EmailContactInfo =
    {
    EmailAddress: string;
    IsEmailVerified: bool;
    }

type PostalAddress =
    {
    Address1: string;
    Address2: string;
    City: string;
    State: string;
    Zip: string;
    }

type PostalContactInfo =
    {
    Address: PostalAddress;
    IsAddressValid: bool;
    }

type Contact =
    {
    Name: PersonalName;
    EmailContactInfo: EmailContactInfo;
    PostalContactInfo: PostalContactInfo;
    }

```

> We haven't written a single function yet, but already the code represents the domain better.
> However, this is just the beginning of what we can do.

Мы ещё не написали ни одной функции, но код уже лучше представляет предметную область.
Однако, это всего лишь начало того, что мы можем сделать.

> Next up, using single case unions to add semantic meaning to primitive types.

Далее рассмотрим исользование одновариатных объединений, чтобы придания примитивным типам семантического значения.