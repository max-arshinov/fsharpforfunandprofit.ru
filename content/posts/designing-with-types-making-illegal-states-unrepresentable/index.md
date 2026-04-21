---
layout: post
title: "Проектирование с помощью типов: Делаем недопустимые состояния непредставимыми"
description: "Кодирование бизнес-логики в типах"
date: 2013-01-14
nav: thinking-functionally
seriesId: "Проектирование с помощью типов"
seriesOrder: 3
categories: [Types, DDD]
---

> In this post, we look at a key benefit of F#, which using the type system to "make illegal states unrepresentable" (a phrase borrowed from [Yaron Minsky](https://blog.janestreet.com/effective-ml-revisited/)).


В этом посте мы познакомимся с ключевым преимуществом F#, который использует систему типов, чтобы «сделать недопустимые состояния непредставимыми» (фраза позаимвствована у [Ярона Мински](https://blog.janestreet.com/effective-ml-revisited/)).

> Let's look at our `Contact` type.
> Thanks to the previous refactoring, it is quite simple:

Ещё раз взглянем на тип `Contact`.
Мы его упростили, благодаря рефакторингу из предыдущего поста:

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

Теперь, представим, что у нас есть простое бизнес-правило: *«Контакт должен иметь электронный адрес или почтовый адрес»*.
Соответствует ли наш тип этому правилу?

> The answer is no.
> The business rule implies that a contact might have an email address but no postal address, or vice versa.
> But as it stands, our type requires that a contact must always have *both* pieces of information.

Ответ — нет.
Бизнес-правило подразумевает, что у контакта может быть электронный адрес и не быть почтового, или наоборот.
Но в текущем виде наш тип требует, чтобы у контакта всегда были *оба* адреса.

> The answer seems obvious -- make the addresses optional, like this:

Ответ кажется очевидным — сделать адреса опциональными:

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

Но сейчас мы ушли в противоположную сторону.
В таком виде контакт может вообще не иметь никаких адресов.
Однако, бизнес-правило утверждает, что в контакте *должен* быть по крайней мере один адрес.

> What's the solution?

Так в чём же решение?

> ## Making illegal states unrepresentable

## Делаем недопустимые состояния непредставимыми

> If we think about the business rule carefully, we realize that there are three possibilities:

Если мы тщательно обдумаем бизнес-правило, то выясним, что существуют всего три варианта:

> * A contact only has an email address
> * A contact only has a postal address
> * A contact has both a email address and a postal address

* У контакта есть только электронный адрес
* У контакта есть только почтовый адрес
* У контакта есть и электронный, и почтовый адреса

> Once it is put like this, the solution becomes obvious -- use a union type with a case for each possibility.

При такой формулировке правила, решение очевидно — надо использовать тип-объединение с тремя вариантами.

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

Для варианта «электронный и почтовый адреса» я пока использую обычный тип-кортеж.
Его вполне хватает.

> ### Constructing a ContactInfo

### Конструирование `ContactInfo`

> Now let's see how we might use this in practice.
> We'll start by creating a new contact:

Теперь разберёмся, как использовать наш тип на практике.
Начнём с создания контакта:

```fsharp
let contactFromEmail name emailStr =
    let emailOpt = EmailAddress.create emailStr
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

Здесь мы написали простую вспомогательную функцию `conctactFromEmail`, чтобы создать новый контакт из имени и электронного адреса.
Однако, электронный адрес может быть неправильным, так функция обрабатывает оба варианта.
В результате возвращаемый тип — это `Contact option`, а не просто `Contact`.

> ### Updating a ContactInfo

### Обновление `ContactInfo`

> Now if we need to add a postal address to an existing `ContactInfo`, we have no choice but to handle all three possible cases:

Если мы хотим добавить почтовый адрес к существующему `ContactInfo`, у нас нет иного выбора, кроме как учесть все возможные варианты:

> * If a contact previously only had an email address, it now has both an email address and a postal address, so return a contact using the `EmailAndPost` case.
> * If a contact previously only had a postal address, return a contact using the `PostOnly` case, replacing the existing address.
> * If a contact previously had both an email address and a postal address, return a contact with using the `EmailAndPost` case, replacing the existing address.

* Если контакт содержал только электронный адрес, теперь он содержит и электронный, и почтовый адреса.
  Возвращаем вариант `EmailAndPost`.
* Если контакт содержал только почтовый адрес, заменяем этот адрес на новый.
  Возвращаем вариант `PostOnly`.
* Если контакт содержал и электронный, и почтовый адреса, заменяем почтовый адрес на новый.
  Возвращаем вариант `EmailAndPost`. 

> So here's a helper method that updates the postal address.
> You can see how it explicitly handles each case.

Вот вспомогательный метод, который обновляет почтовый адрес.
Он явным образом обрабатывает каждый вариант.

```fsharp
let updatePostalAddress contact newPostalAddress =
    let {Name=name; ContactInfo=contactInfo} = contact
    let newContactInfo =
        match contactInfo with
        | EmailOnly email ->
            EmailAndPost (email,newPostalAddress)
        | PostOnly _ -> // игнорируем текущий адрес
            PostOnly newPostalAddress
        | EmailAndPost (email,_) -> // игнорируем текущий адрес
            EmailAndPost (email,newPostalAddress)
    // создаём новый контакт
    {Name=name; ContactInfo=newContactInfo}
```

> And here is the code in use:

А вот пример использования:

```fsharp
let contact = contactOpt.Value   // см. предупреждение об option.Value ниже
let newPostalAddress =
    let state = StateCode.create "CA"
    let zip = ZipCode.create "97210"
    {
        Address =
            {
            Address1= "123 Main";
            Address2="";
            City="Beverly Hills";
            State=state.Value; // см. предупреждение об option.Value ниже
            Zip=zip.Value;     // см. предупреждение об option.Value ниже
            };
        IsAddressValid=false
    }
let newContact = updatePostalAddress contact newPostalAddress
```

> *WARNING: I am using `option.Value` to extract the contents of an option in this code.
> This is ok when playing around interactively but is extremely bad practice in production code!
> You should always use matching to handle both cases of an option.*

*ПРЕДУПРЕЖДЕНИЕ: я использую `option.Value`, чтобы извлечь содержимое опционального типа.*
*Так можно делать в учебном коде, но категорически нельзя в продуктовом!*
*Работая с опциональным типом всегда применяйте сопоставление с образцом.*

> ## Why bother to make these complicated types?

## Зачем вообще создавать такие сложные типы?

> At this point, you might be saying that we have made things unnecessarily complicated.
> I would answer with these points:

Возможно, сейчас вы думаете, что я всё неоправданно усложнил.
Отвечу вам так:

> First, the business logic *is* complicated.
> There is no easy way to avoid it.
> If your code is not this complicated, you are not handling all the cases properly.

Во-первых: бизнес-логика сложна *сама по себе*.
Её нельзя упростить.
Если ваш код проще логики, значит, в нём не учтены все варианты.

> Second, if the logic is represented by types, it is automatically self documenting.
> You can look at the union cases below and immediate see what the business rule is.
> You do not have to spend any time trying to analyze any other code.

Во-вторых, логика, представленная в типах, сама себя документирует.
Взглянув на варианты объединения, вы немедленно понимаете, в чём заключается бизнес-правило.
Вам не надо читать какой-то другой код.

```fsharp
type ContactInfo =
    | EmailOnly of EmailContactInfo
    | PostOnly of PostalContactInfo
    | EmailAndPost of EmailContactInfo * PostalContactInfo
```

> Finally, if the logic is represented by a type, any changes to the business rules will immediately create breaking changes, which is a generally a good thing.

Наконец, если логика представлена типом, любые изменения в бизнес-правиле немедленно сломают код, что, как это ни странно, хорошо.

> In the next post, we'll dig deeper into the last point.
> As you try to represent business logic using types, you may suddenly find that can gain a whole new insight into the domain.

Последний пункт мы обсудим в следюущем посте.
Возможно, что, пытаясь представить бизнес-логику с помощью типов, вы внезапно узнаете что-то совершенно новое о предметной области.

{{< book_page_ddd_img >}}
