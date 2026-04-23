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

В этом посте мы познакомимся с ключевым преимуществом F#, который использует систему типов, чтобы «сделать недопустимые состояния непредставимыми» (фраза позаимвствована у [Ярона Мински](https://blog.janestreet.com/effective-ml-revisited/)).

В прошлом посте мы упростили тип `Contact` благодаря рефакторингу.
Теперь он выглядит так:

```fsharp
type Contact =
    {
    Name: Name;
    EmailContactInfo: EmailContactInfo;
    PostalContactInfo: PostalContactInfo;
    }
```

Теперь представим, что у нас есть простое бизнес-правило: *«Контакт должен иметь электронный адрес или почтовый адрес»*.
Соответствует ли наш тип этому правилу?

Ответ — нет.
Бизнес-правило подразумевает, что у контакта может быть электронный адрес и не быть почтового, или наоборот.
Но в текущем виде наш тип требует, чтобы у контакта всегда были *оба* адреса.

Решение кажется очевидным — сделать адреса опциональными:

```fsharp
type Contact =
    {
    Name: PersonalName;
    EmailContactInfo: EmailContactInfo option;
    PostalContactInfo: PostalContactInfo option;
    }
```

Но и это решение неправильное.
Теперь контакт может вообще не иметь никаких адресов.
Однако, бизнес-правило утверждает, что в контакте *должен* быть по крайней мере один адрес.

Так какое решение правильное?

## Делаем недопустимые состояния непредставимыми

Если мы тщательно обдумаем бизнес-правило, то выясним, что существуют всего три варианта:

* У контакта есть только электронный адрес
* У контакта есть только почтовый адрес
* У контакта есть и электронный, и почтовый адреса

При такой формулировке правила, решение очевидно — используем тип-объединение с тремя вариантами.

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

Этот дизайн идеально соответствует требованиям.
Все три варианта представлены в явном виде, а четвёртый возможный вариант (когда вообще нет ни электронного, ни почтового адреса) недоступен.

Для варианта «электронный и почтовый адреса» я пока использую обычный тип-кортеж.
Его вполне хватает.

### Конструируем `ContactInfo`

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

Здесь мы написали простую вспомогательную функцию `conctactFromEmail`, чтобы создать новый контакт из имени и электронного адреса.
Однако, электронный адрес может быть неправильным, так функция умеет обрабатывать ошибку.
Поэтому возвращаемый тип — это `Contact option`, а не просто `Contact`.

### Обновляем `ContactInfo`

Если мы хотим добавить почтовый адрес к существующему `ContactInfo`, у нас нет иного выбора, кроме как учесть все возможные варианты:

* Если контакт содержал только электронный адрес, добавляем почтовый адрес.
  Возвращаем вариант `EmailAndPost`.
* Если контакт содержал только почтовый адрес, заменяем этот адрес на новый.
  Возвращаем вариант `PostOnly`.
* Если контакт содержал и электронный, и почтовый адреса, заменяем почтовый адрес на новый.
  Возвращаем вариант `EmailAndPost`. 

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

*ПРЕДУПРЕЖДЕНИЕ: я использую `option.Value`, чтобы извлечь содержимое опционального типа.*
*Так можно делать в учебном коде, но категорически нельзя в продуктовом!*
*Работая с опциональным типом всегда применяйте сопоставление с образцом.*

## Зачем вообще создавать такие сложные типы?

Возможно, сейчас вы думаете, что я всё неоправданно усложнил.
Отвечу вам так:

Во-первых: бизнес-логика сложна *сама по себе*.
Её нельзя упростить.
Если ваш код проще логики, значит, в нём чего-то не хватает.

Во-вторых, логика, представленная в типах, сама себя документирует.
Взглянув на варианты объединения, вы сразу понимаете, в чём заключается бизнес-правило.
Вам не надо читать какой-то другой код.

```fsharp
type ContactInfo =
    | EmailOnly of EmailContactInfo
    | PostOnly of PostalContactInfo
    | EmailAndPost of EmailContactInfo * PostalContactInfo
```

Наконец, если логика представлена типом, любые изменения в бизнес-правиле немедленно сломают код, что, как это ни странно, хорошо.

Последний пункт мы обсудим в следюущем посте.
Возможно, что, пытаясь представить бизнес-логику с помощью типов, вы внезапно узнаете что-то совершенно новое о предметной области.

{{< book_page_ddd_img >}}
