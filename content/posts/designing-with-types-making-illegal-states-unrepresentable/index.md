---
layout: post
# title: "Designing with types: Making illegal states unrepresentable"
title: "Проектирование с помощью типов: Делаем недопустимые состояния непредставимыми"
# description: "Encoding business logic in types"
description: "Кодирование бизнес-логики в типах"
date: 2013-01-14
nav: thinking-functionally
# seriesId: "Designing with types"
seriesId: "Проектирование с помощью типов"
seriesOrder: 3
categories: [Types, DDD]
---

> In this post, we look at a key benefit of F#, which using the type system to "make illegal states unrepresentable" (a phrase borrowed from [Yaron Minsky](https://blog.janestreet.com/effective-ml-revisited/)).

В этом посте мы познакомимся с ключевым преимуществом F#, который использует систему типов, чтобы «сделать недопустимые состояния непредставимыми» (фраза позаимоствована у [Ярона Мински](https://blog.janestreet.com/effective-ml-revisited/)).

> Let's look at our `Contact` type.
> Thanks to the previous refactoring, it is quite simple:

Давайте взглянем на наш тип `Contact`.
Благодаря предыдущему рефакторингу, он довольно прост:

```fsharp
type Contact =
    {
    Name: Name;
    EmailContactInfo: EmailContactInfo;
    PostalContactInfo: PostalContactInfo;
    }
```

> Now let's say that we have the following simple business rule: *"A contact must have an email or a postal address"*.
> Does our type conform to this rule?

Теперь, представим, что у нас есть такое простое бизнес-правило: *«Контакт должен иметь электронный адрес или почтовый адрес»*.
Соответствует ли наш тип этому правилу?

> The answer is no.
> The business rule implies that a contact might have an email address but no postal address, or vice versa.
> But as it stands, our type requires that a contact must always have *both* pieces of information.

И ответ — нет.
Бизнес-правило подразумевает, что у контакта может быть электронный адрес и не быть почтового, или наоборот.
Но в текущем виде наш тип требует, чтобы у контакта всегда были *обе* части информации.

> The answer seems obvious -- make the addresses optional, like this:

Ответ кажется очевидным — сделаем адреса опциональными, как здесь:

```fsharp
type Contact =
    {
    Name: PersonalName;
    EmailContactInfo: EmailContactInfo option;
    PostalContactInfo: PostalContactInfo option;
    }
```

> But now we have gone too far the other way.
> In this design, it would be possible for a contact to have neither type of address at all.
> But the business rule says that at least one piece of information *must* be present.

Но сейчас мы слишком далеко ушли в противоположную сторону.
В таком дизайне для контакта возможно вообще не иметь никаких адресов.
Но бизнес-правило говорит, что по меньшей мере одна часть информации *должна* быть.

> What's the solution?

Каков же правильный ответ?

> ## Making illegal states unrepresentable

## Делаем недопустимые состояния непредставимыми

> If we think about the business rule carefully, we realize that there are three possibilities:

Если мы внимательно рассмотрим это бизнес-правило, то поймём, что существуют всего три возможности:

> * A contact only has an email address
> * A contact only has a postal address
> * A contact has both a email address and a postal address

* У контакта есть только электронный адрес
* У контакта есть только почтовый адрес
* У контакта есть и электронный, и почтовый адреса

> Once it is put like this, the solution becomes obvious -- use a union type with a case for each possibility.

Если мы сформулируем это так, решение будет очевидным — использовать тип-объединение с вариантом для каждой возможности.

```fsharp
type ContactInfo =
    | EmailOnly of EmailContactInfo
    | PostOnly of PostalContactInfo
    | EmailAndPost of EmailContactInfo * PostalContactInfo

type Contact =
    {
    Name: Name;
    ContactInfo: ContactInfo;
    }
```

> This design meets the requirements perfectly.
> All three cases are explicitly represented, and the fourth possible case (with no email or postal address at all) is not allowed.

Этот дизайн идеально соответствует требованиям.
Все три варианта представлены в явном виде, а четвёртый возможный вариант (когда вообще нет ни электронного, ни почтового адреса) недоступен.

> Note that for the "email and post" case, I just used a tuple type for now.
> It's perfectly adequate for what we need.

Обратите внимание, что для варианта «электронный и почтовый адреса» на данный момент я использовал простой тип-кортеж.
Его достаточно для того, что нам нужно.

> ### Constructing a ContactInfo

### Конструирование `ContactInfo`

> Now let's see how we might use this in practice.
> We'll start by creating a new contact:

Теперь давайте посмотрим, как мы можем использовать это на практике.
Начнём создание нового контакта:

```fsharp
let contactFromEmail name emailStr =
    let emailOpt = EmailAddress.create emailStr
    // handle cases when email is valid or invalid
    // обрабатываем варианты правильного и неправильного электронного адреса
    match emailOpt with
    | Some email ->
        let emailContactInfo =
            {EmailAddress=email; IsEmailVerified=false}
        let contactInfo = EmailOnly emailContactInfo
        Some {Name=name; ContactInfo=contactInfo}
    | None -> None

let name = {FirstName = "A"; MiddleInitial=None; LastName="Smith"}
let contactOpt = contactFromEmail name "abc@example.com"
```

> In this code, we have created a simple helper function `contactFromEmail` to create a new contact by passing in a name and email.
> However, the email might not be valid, so the function has to handle both cases, which it does by returning a `Contact option`, not a `Contact`

В этом коде мы написали простую вспомогательную функцию `conctactFromEmail`, чтобы создать новый контакт, передавая имя и электронный адрес.
Однако, поскольку электронный адрес может быть неправильным, функция обрабатывает оба варианта, что приводит к тому, что возвращаемый тип — это `Contact option`, а не `Contact`.

> ### Updating a ContactInfo

### Обновление `ContactInfo`

> Now if we need to add a postal address to an existing `ContactInfo`, we have no choice but to handle all three possible cases:

Теперь, если мы хотим добавить почтовый адрес к существующему `ContactInfo`, у нас нет другого выбора, кроме как обработать три возможных варианта:

> * If a contact previously only had an email address, it now has both an email address and a postal address, so return a contact using the `EmailAndPost` case.
> * If a contact previously only had a postal address, return a contact using the `PostOnly` case, replacing the existing address.
> * If a contact previously had both an email address and a postal address, return a contact with using the `EmailAndPost` case, replacing the existing address.

* Если контакт раньше содержал только электронный адрес, теперь он содержит и электронный, и почтовый адреса, так что возвращаем контакт, используя вариант `EmailAndPost`.
* Если контакт раньше содержал только почтовый адрес, возвращаем контакт, используя вариант `PostOnly`, заменяя текущий адрес.
* Если контакт раньше содержал и электронный, и почтовый адреса, возвращаем контакт, используя вариант `EmailAndPost`, заменяя текущий адрес. 

> So here's a helper method that updates the postal address.
> You can see how it explicitly handles each case.

Вот вспомогательный метод, который обновляет почтовый адрес.
Вы видите, как он явным образом обрабатывает каждый вариант.

```fsharp
let updatePostalAddress contact newPostalAddress =
    let {Name=name; ContactInfo=contactInfo} = contact
    let newContactInfo =
        match contactInfo with
        | EmailOnly email ->
            EmailAndPost (email,newPostalAddress)
        | PostOnly _ -> // игнорируем текущий адрес (ignore existing address)
            PostOnly newPostalAddress
        | EmailAndPost (email,_) -> // игнорируем текущий адрес (ignore existing address)
            EmailAndPost (email,newPostalAddress)
    // make a new contact
    // создаём новый контакт
    {Name=name; ContactInfo=newContactInfo}
```

> And here is the code in use:

А вот как использовать этот код:

```fsharp
let contact = contactOpt.Value   // см. предупреждение об option.Value ниже (see warning about option.Value below)
let newPostalAddress =
    let state = StateCode.create "CA"
    let zip = ZipCode.create "97210"
    {
        Address =
            {
            Address1= "123 Main";
            Address2="";
            City="Beverly Hills";
            State=state.Value; // см. предупреждение об option.Value ниже (see warning about option.Value below)
            Zip=zip.Value;     // см. предупреждение об option.Value ниже (see warning about option.Value below)
            };
        IsAddressValid=false
    }
let newContact = updatePostalAddress contact newPostalAddress
```

> *WARNING: I am using `option.Value` to extract the contents of an option in this code.
> This is ok when playing around interactively but is extremely bad practice in production code!
> You should always use matching to handle both cases of an option.*

*ПРЕДУПРЕЖДЕНИЕ: я использую в этом коде `option.Value`, чтобы извлечь содержимое опционального типа.*
*Это допустимо в учебном коде, но категорически плохая практика в продуктовом коде!*
*Вы всегда должны использовать сопоставление с образцом для обработки обоих случаев опционального типа.*

> ## Why bother to make these complicated types?

## Зачем вообще создавать такие сложные типы?

> At this point, you might be saying that we have made things unnecessarily complicated.
> I would answer with these points:

Сейчас вы, возможно, скажите, что мы всё неоправданно усложнили.
Я бы ответил вам так:

> First, the business logic *is* complicated.
> There is no easy way to avoid it.
> If your code is not this complicated, you are not handling all the cases properly.

Во-первых: бизнес-логика сложна *сама по себе*.
Нет простого способа избежать этого.
Если ваш код не настолько сложне, значит, он не обрабатывает должным образом все варианты.

> Second, if the logic is represented by types, it is automatically self documenting.
> You can look at the union cases below and immediate see what the business rule is.
> You do not have to spend any time trying to analyze any other code.

Во-вторых, если логика представлена в типах, она автоматически становится само-документируемой.
Вы можете взглянуть на варианты объединения ниже и немедленно понять, в чём заключается бизнес-правило.
Вы не тратите никакого времени, чтобы проанализировать любой другой код.

```fsharp
type ContactInfo =
    | EmailOnly of EmailContactInfo
    | PostOnly of PostalContactInfo
    | EmailAndPost of EmailContactInfo * PostalContactInfo
```

> Finally, if the logic is represented by a type, any changes to the business rules will immediately create breaking changes, which is a generally a good thing.

Наконец, если логика представлена типом, любые изменения в бизнес-правила немедленно сломают код, что, как правило, хорошо.

> In the next post, we'll dig deeper into the last point.
> As you try to represent business logic using types, you may suddenly find that can gain a whole new insight into the domain.

В следующем посте мы подробнее рассмотрим последний пункт.
Может случиться так, что пытаясь представить бизнес-логику с помощью типов, вы внезапно получите совершенно новое представление о предметной области.

{{< book_page_ddd_img >}}
