---
layout: post
# title: "Designing with types: Discovering new concepts"
title: "Проектирование с помощью типов: Исследуем новые концепции"
# description: "Gaining deeper insight into the domain"
description: "Углубляем понимание предметной области"
date: 2013-01-15
nav: thinking-functionally
# seriesId: "Designing with types"
seriesId: "Проектирование с помощью типов"
seriesOrder: 4
categories: [Types, DDD]
---

> In the last post, we looked at how we could represent a business rule using types.

В прошлом посте мы разобрались, как можно представить бизенс-правило используя типы.

> The rule was: *"A contact must have an email or a postal address"*.

Правило звучало так: *«У контакта должен быть электронный или почтовый адрес»*.

> And the type we designed was:

А тип, который мы спроектировали, был:

```fsharp
type ContactInfo =
    | EmailOnly of EmailContactInfo
    | PostOnly of PostalContactInfo
    | EmailAndPost of EmailContactInfo * PostalContactInfo
```

> Now let's say that the business decides that phone numbers need to be supported as well.
> The new business rule is: *"A contact must have at least one of the following: an email, a postal address, a home phone, or a work phone"*.

Теперь представим, что бизнес решил, что номера телефонов тоже нуждаются в поддержке.
Новое бизнес-правило: *«У контакта должно быть как минимум что-то из перечисленного: электронный адрес, почтовый адрес, домашний телефон или рабочий телефон»*.

> How can we represent this now?

Как нам сейчас представить эту логику?

> A little thought reveals that there are 15 possible combinations of these four contact methods.
> Surely we don't want to create a union case with 15 choices?
> Is there a better way?

Немного поразмыслив, можно прийти к выводу, что существует 15 возможных комбинаций этих четырёх способов контакта.
Мы ведь не хотим создавать объединение с 15-ю вариантами выбора?
Есть ли лучший способ?

> Let's hold that thought and look at a different but related problem.

Давайте зафиксируем эту мысль и посмотрим на другую — связанную — проблему.

> ## Forcing breaking changes when requirements change

## Изменение требований должно «ломать» код

> Here's the problem.
> Say that you have a contact structure which contains a list of email addresses and also a list of postal addresses, like so:

А проблема такая.
Представим, что у нас есть структура для представления контакта, которая содержит список электронных адресов и список почтовых адресов, вот так:

```fsharp
type ContactInformation =
    {
    EmailAddresses : EmailContactInfo list;
    PostalAddresses : PostalContactInfo list
    }
```

> And, also let's say that you have created a `printReport` function that loops through the information and prints it out in a report:

Представим тажке, что вы создали функцию `printReport`, которая перебирает всю информацию и печатет её в отчёте:

```fsharp
// mock code
// фиктивный код
let printEmail emailAddress =
    printfn "Email Address is %s" emailAddress

// mock code
// фиктивный код
let printPostalAddress postalAddress =
    printfn "Postal Address is %s" postalAddress

let printReport contactInfo =
    let {
        EmailAddresses = emailAddresses;
        PostalAddresses = postalAddresses;
        } = contactInfo
    for email in emailAddresses do
         printEmail email
    for postalAddress in postalAddresses do
         printPostalAddress postalAddress
```

> Crude, but simple and understandable.

Грубо, но просто и понятно.

> Now if the new business rule comes into effect, we might decide to change the structure to have some new lists for the phone numbers.
> The updated structure will now look something like this:

Теперь, если в силу вступает новое бизнес-правило, мы можем решить изменить стурктуру, чтобы в ней появились новые списки для телефонных номеров.
Обновлённая структура теперь будет выглядеть примерно так:

```fsharp
type PhoneContactInfo = string // на данный момент заглушка (dummy for now)

type ContactInformation =
    {
    EmailAddresses : EmailContactInfo list;
    PostalAddresses : PostalContactInfo list;
    HomePhones : PhoneContactInfo list;
    WorkPhones : PhoneContactInfo list;
    }
```

> If you make this change, you also want to make sure that all the functions that process the contact information are updated to handle the new phone cases as well.

Сделав это изменение, вам таже следует убедиться, что в код всех функций, которые обрабатывают контактную информацию, внесены изменения, которые обрабатывают новые телефонные поля.

> Certainly, you will be forced to fix any pattern matches that break.
> But in many cases, you would *not* be forced to handle the new cases.

Безусловно, вам придётся исправить любые сломавшиеся сопоставления с образцом.
Но, во многих случаях вы даже *не* узнаете о необходимости обрабатывать новые варианты.

> For example, here's `printReport` updated to work with the new lists:

Например, вот `printReport`, умеющая работать с новыми списками:

```fsharp
let printReport contactInfo =
    let {
        EmailAddresses = emailAddresses;
        PostalAddresses = postalAddresses;
        } = contactInfo
    for email in emailAddresses do
         printEmail email
    for postalAddress in postalAddresses do
         printPostalAddress postalAddress
```

> Can you see the deliberate mistake?
> Yes, I forgot to change the function to handle the phones.
> The new fields in the record have not caused the code to break at all.
> There is no guarantee that you will remember to handle the new cases.
> It would be all too easy to forget.

Видите, какую ошибку я сделал?
Да, я забыл дописать функцию, чтобы она обрабатывала телефоны.
Новые поля в записи вообще не ломают код.
И нет никаких гарантий, что вы вспомните об обработке новых вариантов.
Об этом слишком легко забыть.

> Again, we have the challenge: can we design types such that these situations cannot easily happen?

И снова перед нами проблема: можно ли спроектировать типы так, чтобы обезопасить себя от подобных ситуаций?

> ## Deeper insight into the domain

## Углублённое понимание предметной области

> If you think about this example a bit more deeply, you will realize that we have missed the forest for the trees.

Если вы глубже погрузитесь в приведённый пример, вы поймёте, что за деревьями мы не увидели леса.

> Our initial concept was: *"to contact a customer, there will be a list of possible emails, and a list of possible addresses, etc"*.

Нашей первоначальной концепцией была: *«для связи с заказчиком, у нас будет список возможных электронных адресов, список возможных почтовых адресов и т. д.»*.

> But really, this is all wrong.
> A much better concept is: *"To contact a customer, there will be a list of contact methods. Each contact method could be an email OR a postal address OR a phone number"*.

Но на самом деле всё это неправильно.
Гораздо более подходящая концепция: *«для связи с заказчиком, у нас будет список способов контакта, где способ контакта — это электронный адрес ИЛИ почтовый адрес ИЛИ телефонный номер»*.

> This is a key insight into how the domain should be modelled.
> It creates a whole new type, a "ContactMethod", which resolves our problems in one stroke.

Это — ключевая идея, как нужно моделировать предметную область.
Она приводит нас к созданию нового типа `ContactMethod`, который одним махом решает наши проблемы.

> We can immediately refactor the types to use this new concept:

Мы немедленно можем рефакторить типы, чтобы применить новую концепцию:

```fsharp
type ContactMethod =
    | Email of EmailContactInfo
    | PostalAddress of PostalContactInfo
    | HomePhone of PhoneContactInfo
    | WorkPhone of PhoneContactInfo

type ContactInformation =
    {
    ContactMethods  : ContactMethod list;
    }
```

> And the reporting code must now be changed to handle the new type as well:

И теперь код отчёта должен быть изменён, чтобы обрабатывать новый тип:

```fsharp
// mock code
// фиктивный код
let printContactMethod cm =
    match cm with
    | Email emailAddress ->
        printfn "Электронный адрес %s" emailAddress
    | PostalAddress postalAddress ->
         printfn "Почтовый адрес %s" postalAddress
    | HomePhone phoneNumber ->
        printfn "Домашний телефон %s" phoneNumber
    | WorkPhone phoneNumber ->
        printfn "Рабочий телефон %s" phoneNumber

let printReport contactInfo =
    let {
        ContactMethods=methods;
        } = contactInfo
    methods
    |> List.iter printContactMethod
```

> These changes have a number of benefits.

Эти правки имеют ряд преимуществ.

> First, from a modelling point of view, the new types represent the domain much better, and are more adaptable to changing requirements.

Для начала, с точки зрения моделирования, новые типы гораздо лучше представляют предметную область и больше приспособлены к изменению требований.

> And from a development point of view, changing the type to be a union means that any new cases that we add (or remove) will break the code in a very obvious way, and it will be much harder to accidentally forget to handle all the cases.

А с точки зрения разработки, изменение типа на объединение означает, что все новые варинаты, которые мы добавим (или удалим), очевидным образом сломают код, и будет гораздо сложнее случайно забыть обработать все варианты.

{{< book_page_ddd >}}

> ## Back to the business rule with 15 possible combinations

## Вернёмся к бизнес-правило с 15-ю возможными комбинациями

> So now back to the original example.
> We left it thinking that, in order to encode the business rule, we might have to create 15 possible combinations of various contact methods.

Итак, вернёмся к исходному примеру.
Тогда мы думали, что для кодирования бизнес-правила нам, возможно, придётся создать 15 возможных комбинаций различных способов связи.

> But the new insight from the reporting problem also affects our understanding of the business rule.

Однако, новый взгляд на проблемы с отчётом, также влияет на наше понимание бизнес-правила.

> With the "contact method" concept in our heads, we can rephase the requirement as: *"A customer must have at least one contact method. A contact method could be an email OR a postal addresses OR a phone number"*.

Держа в голове концепцию «способа связи», мы можем перефразировать требование следующим образом: *«У заказчика должен быть как минимум один способ связи, который может быть электронным адресом ИЛИ почтовым адресом ИЛИ телефонным номером»*.

> So let's redesign the `Contact` type to have a list of contact methods:

Так что давайте перепроектируем тип `Contact` так, чтобы он содержал список способов связи.

```fsharp
type Contact =
    {
    Name: PersonalName;
    ContactMethods: ContactMethod list;
    }
```

> But this is still not quite right.
> The list could be empty.
> How can we enforce the rule that there must be *at least* one contact method?

Но это всё ещё не совсем верно.
Список может быть пустым.
Как мы можем обеспечить соблюдение правила, согласно которому должен быть *как минимум* один способ связи?

> The simplest way is to create a new field that is required, like this:

Простейший способ заключается в том, чтобы добавить новое обязательное поле, как здесь:

```fsharp
type Contact =
    {
    Name: PersonalName;
    PrimaryContactMethod: ContactMethod;
    SecondaryContactMethods: ContactMethod list;
    }
```

> In this design, the `PrimaryContactMethod` is required, and the secondary contact methods are optional, which is exactly what the business rule requires!

В этом дизайне поле `PrimaryConactMethod` (основной способ связи) является обязательным, а дополнительные способы связи — нет, что полностью соответсвует бизнес-правилу!

> And this refactoring too, has given us some insight.
> It may be that the concepts of "primary" and "secondary" contact methods might, in turn, clarify code in other areas, creating a cascading change of insight and refactoring.

Кроме того, этот рефакторинг дал нам новое понимание.
Вполне возможно, что концепция «основного» и «дополнительного» способа связи может, в свою очередь, прояснить код в других местах, вызывав каскадные изменения в понимании и рефакторинге.

> ## Summary

## Заключение

> In this post, we've seen how using types to model business rules can actually help you to understand the domain at a deeper level.

В этом посте мы познакомились с тем, как использование типов для моделирования бизнес-правил, может помочь вам лучше разобраться в предметной области.

> In the *Domain Driven Design* book, Eric Evans devotes a whole section and two chapters in particular (chapters 8 and 9) to discussing the importance of [refactoring towards deeper insight](http://dddcommunity.org/wp-content/uploads/files/books/evans_pt03.pdf).
> The example in this post is simple in comparison, but I hope that it shows that how an insight like this can help improve both the model and the code correctness.

В книге *Предметно-ориентированное проектирование* Эрик Эванс посвятил целый раздел и, в частности, две главы (8 и 9) обсуждению важности [углубленного рефакторинга](http://dddcommunity.org/wp-content/uploads/files/books/evans_pt03.pdf). <!-- углубленный рефакторинг -- термин из русского перевода книги Эванса -->
Пример в этом посте сравнительно прост, но, я надеюсь, он показывает, как углубленное понимание может помочь улучшить как модель, так и правильность кода.

> In the next post, we'll see how types can help with representing fine-grained states.

В следующем посте мы узнаем, как типы могут помочь в представлении детальных состояний.
