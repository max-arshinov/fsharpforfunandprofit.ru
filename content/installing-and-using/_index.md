---
layout: page
# title: "Installing and using F#"
title: "Установка и использование F#"
# description: "Instructions for downloading, installing and using F# with Visual Studio, SharpDevelop and MonoDevelop"
description: "Инструкции по загрузке, установке и использованию F# в Visual Studio, SharpDevelop и MonoDevelop"
nav:
hasComments: 1
image: "/installing-and-using/fsharp_eval2.png"
date: 2014-01-01
---

> The F# compiler is a free and open source tool which is available for Windows, Mac and Linux (via Mono).
Компилятор F# это свободный инструмент с открытым исходным кодом, который доступен для Windows, Max и Linux (благодаря Mono).
> Find out more about F# and how to install it at the [F# Foundation](http://fsharp.org/).
Узнайте больше о F# и о том, как его установить в [Фонде F#](http://fsharp.org/).


> You can use it with an IDE (Visual Studio, MonoDevelop), or with your favorite editor (VS Code and Atom have especially good F# support using [Ionide](http://ionide.io/)), or simply as a standalone command line compiler.
Мы можете использовать его в интегрированных средах разработки (IDE — Visual Studio, MonoDevelop), или в вашем любимом редакторе (VS Code и Atom имеют особенно хорошую поддержку F# благодаря [Ionide](http://ionide.io)), или просто как самостоятельный компилятор в командной строке.

> If you don't want to install anything, you can try the [.NET Fiddle](https://dotnetfiddle.net/) site, which is an interactive environment where you can explore F# in your web browser.
Если вы не хотите ничего устанавливать, вы можете попробовать [.NET Fiddle](https://dotnetfiddle.net/), сайт, который предоставляет интерактивную среду, где вы можете исследовать F# в вашем браузере. <!-- "браузер", мне кажется, уже прочно осел в русском языке. Предложить в словарь. -->
> You should be able to run most of the code on this site there.
Там вы сможете запустить большую часть кода с этого сайта.

> ## Working with the code examples ##
> ## Как работать с примерами кода ##

> Once you have F# installed and running, you can follow along with the code samples.
Как только вы установили и запустили F#, вы можете самостоятельно изучать примеры кода.

> The best way to run the code examples on this site is to type the code into an `.FSX` script file, which you can then send to the F# interactive window for evaluation.
Лушчий способ запускть примеры с этого сайта, это вводить код в файл сценария `.FSX`, который вы затем можете отправить в окно "F# Interactive" для выполнения <!-- в русской студии название окна не переведено, так и осталось F# Interactive -->.
> Alternatively you can type the examples directly into the F# interactive console window.
Либо вы можете вводить примеры непосредственно в консольное окно "F# Interactive".
> I would recommend the script file approach for anything other than one or two lines.
Я бы рекомендовал подход с файлом скрипта для всего, что больше одной или двух строк кода.

> For the longer examples, the code is downloadable from this website -- the links will be in the post.
Код больших примеров доступен для загрузки с этого сайта — ссылки будут в посте.

> Finally, I would encourage you to play with and modify the examples.
Наконец, я бы предложил вам поиграть с примерами и вносить в них изменения.
> If you then get compiler errors, do check out the ["troubleshooting F#"](/troubleshooting-fsharp/) page, which explains the most common problems, and how to fix them.
Если вы получите ошибку компилятора, проверьте страницу ["Исправление ошибок F#"](/troubleshooting-fsharp/), которая объясняет наиболее общие проблемы, и как их исправить.

> ### Contents ###
> ### Содержание ###

> * [Installing F#](#installing-fsharp)
> * Using F# with various tools
>   * [Using F# in Visual Studio](#visual-studio)
>   * [Using F# with Mono on Linux and Mac](#mono-develop)
>   * [Try F# directly in your browser](#browser)
>   * [Using F# in the FSI interactive shell](#interactive-shell)
>   * [Using F# in SharpDevelop](#sharp-develop)
> * [Compilation Errors](#compilation-errors)
> * [Projects and Solutions](#projects-solutions)
> * [Using F# for shell scripts](#shell-scripts)

> * [Установка F#](#installing-fsharp)
> * Использование F# с различными инструментами
>   * [Использование F# с Visual Studio](#visual-studio)
>   * [Использование F# с Mono на Linux и Mac](#mono-develop)
>   * [Попробуйте F# прямо в вашем браузере](#browser)
>   * [Использование F# в интерактивной оболочке FSI](#interactive-shell)
>   * [Использование F# в SharpDevelop](#sharp-develop)
> * [Ошибки компиляции](#compilation-errors)
> * [Проекты и решения](#projects-solutions)
> * [Использование F# в скриптах оболочки](#shell-scripts)

{{< linktarget "installing-fsharp" >}}

> ## Installing F# ##
> ## Установка F# ##

> You can [get F# for multiple platforms here](http://fsxplat.codeplex.com/).
Вы можете [взять F# для множества платформ здесь](http://fsxplat.codeplex.com/).
> Once you have downloaded and installed F#, you might also consider installing the [F# power pack](http://fsharppowerpack.codeplex.com), which provides a number of nice extras, some of which will be referred to in this site.
После того, как вы скачаете и установите F#, вы можете рассмотреть возможность установки [F# power pack](http://fsharppowerpack.codeplex.com), который предоставляет несколько интересных <!-- дословно: приятных, изящных --> расширений, и этот сайт будет ссылаться на некоторые из них.

{{< linktarget "visual-studio" >}}

> ## Using F# in Visual Studio
> ## Использование F# в Visual Studio

> If you are on a Windows platform, using Visual Studio to write F# is strongly recommended, as F# has excellent integration with the IDE, debugger, etc.
Если вы работаете на платформе Windows, использование Visual Studio для разработки на F# настойчиво рекомендуется, так как F# отлично интегрирован с IDE, отладчиком и т.д.

> * If you have Visual Studio 2010 or higher, F# is already included.
> * If you have Visual Studio 2008, you can download an installer for F# from MSDN.
> * If you have neither, you can install the "Visual Studio Integrated Shell" and then install F# into that.

> * Если у вас Visual Studio 2010 или выше, F# уже включён.
> * Если у вас Visual Studio 2008, вы можете скачать установку F# с MSDN.
> * Если у вас ни то, и не другое, вы можете установить "Изолированную оболочку Visual Studio" <!-- так в документации от Microsoft --> и затем установить F# в неё.

> Once you have F# installed, you should create an F# project.
После того, как вы установили F#, вы должны создать проект F#.

![Новый проект](./fsharp_new_project2.png)

> And then create a script (FSX) file to run the examples in.
И затем создать файл скрипта (FSX) для запуска в нём примеров.

![Новый сценарий FSX](./fsharp_new_script2.png)

> Next, make sure the F# interactive window is active (typically via `View > Other Windows > F# Interactive`).
Далее, убедитесь в том, что интерактивное окно F# активно (обычно через `Вид > Другие окна > F# Interactive`).

> Using a script file is the easiest way to experiment with F#; simply type in some code in the script window and evaluate it to see the output in the interactive window below.
Использование файла скрипта это простейший путь <!-- способ --> экспериментировать с F#; просто набираете любой код в окно сценария и запускайте его, чтобы увидеть результат <!-- дословно: "вывод", "то, что этот код выведет" --> в интерактивном окне ниже.
> To evalulate highlighted code, you can:
Чтобы запустить выделенный код, вы можете:

> * Right click to get a context menu and do "send to interactive".
>   Note that if the F# interactive window is not visible, the "send to interactive" menu option will not appear.
> * Use the `Alt+Enter` key combination (but see note below on keyboard mappings).
* Щёлкните правой кнопкой, чтобы вызывать контексное меню и выберите "Выполнение в интерактивном режиме" ("Execute In Interactive").
  <!-- Проверил в английской студии 2019, пункт меню называется именно Execute In Interactive. Кажется, что его переименовали. Русский перевод здесь и далее, такой же, как в русской студии. !-->
  Имейте в виду, что если интерактивное окно F# невидимо, опция меню "Execute In Interactive" будет недоступна.
* Используйте комбинацию клавиш `Alt+Enter` (но смотрите замечание ниже про сопоставления клавиш <!-- Вариант: клавиатурные назначения. Обсудить в словаре. -->).

![Выполнить в интерактивном окне](./send_to_interactive.png)

{{< alerterror >}}
> #### Resharper alert

> #### Предупреждение по поводу Resharper <!-- Оригинальный заголовок короткий, и по стилю соответствует чему-то вроде "Внимание! Resharper", "Осторожно! Resharper" или "Помните про Resharper". -->

> If you have Resharper or other plugins installed, the `Alt+Enter` key combination may be taken.
Если вы установили Resharper или другие расширения <!-- Обсудить в словаре. Кажется, что вариант "плагин" уже прижился в русском языке. -->, комбинация клавиш `Alt+Enter` может быть занята <!-- может быть назначена другой команде -->.
> In this case many people remap the command to `Alt+Semicolon` instead.
В этом случае многие люди вместо неё назначают команду на `Alt+;` <!-- Надо ли писать `Alt+Точка с запятой`? -->

> You can remap the command from the Visual Studio menu `Tools > Options > Keyboard`, and the "Send To Interactive" command is called `EditorContextMenus.CodeWindow.SendToInteractive`.
Вы можете переназначаить команду в меню Visual Studio `Средства > Параметры > Окружение > Клавиатура`, где команда "Выполнение в интерактивном режиме" называется `EditorContextMenus.CodeWindow.ExecuteInInteractive`. <!-- Проверил в Visual Studio 2019. Закладка Keyboard теперь в группе Environment, и команда называется ExecuteInInteractive вместо SendToInteractive. -->
{{< /alerterror >}}

> You can also work directly in the interactive window.
Вы можете также работать непосредственно в интерактивном окне.
> But in this case, you must always terminate a block of code with double semicolons.
Но в этом случае вы должны всегда завершать блок кода двумя точками с запятой.

![Интерактивное окно](./fsharp_interactive2.png)

{{< linktarget "mono-develop" >}}

> ## Using F# with Mono on Linux and Mac
> ## Использование F# с Mono на Linux и Mac

> F# is included in Mono as of the Mono 3.0.2 release.
F# включён в Mono с момента выпуска Mono 3.0.2.
> [Download Mono here](http://www.go-mono.com/mono-downloads/download.html).
> [Скачать Mono здесь](http://www.go-mono.com/mono-downloads/download.html).

> Once you have Mono installed, you can use the MonoDevelop IDE or an editor such as Emacs.
Как только вы установили Mono, вы можете использовать среду разработки MonoDevelop или такой редактор, как Emacs. <!-- по смыслу, скорее, "какой-то из редакторов, например, Emacs". -->

> * [MonoDevelop](http://monodevelop.com/) has integrated support for F#.
>   See [http://addins.monodevelop.com/Project/Index/48](http://addins.monodevelop.com/Project/Index/48).
> * There is an F# mode for Emacs that extends it with syntax highlighting for F#.
>   See the ["F# cross-platform packages and samples"](http://fsxplat.codeplex.com/) page on Codeplex.
* Поддержка F# уже интегрирована в [MonoDevelop](http://monodevelop.com/).
  Смотрите [http://addins.monodevelop.com/Project/Index/48](http://addins.monodevelop.com/Project/Index/48).
* В Emacs есть <!-- существует --> режим F#, который добавляет подсветку синтаксиса F#.
  Смотрите страницу ["F# cross-platform packages and samples"](http://fsxplat.codeplex.com/) на Codeplex.

{{< linktarget "browser" >}}
> ## Try F# in your browser
> ## Попробуйте F# в своём браузере <!-- В Содержании название раздела отличается. Там "try direct in your browser"  -->

> If you don't want to download anything, you can try F# directly from your browser.
Если вы не хотите ничего скачивать, вы можете попробовать F# непосредственно в своём браузере.
> The site is at [try.fsharp.org](https://try.fsharp.org/).
Нужный сайт находится по адресу [try.fsharp.org](https://try.fsharp.org/).
> Note that it does require Silverlight to run.
Обратите внимание, что он требует Silverlight для своей работы. <!-- Проверил: сайт до сих пор существует, но сейчас уже не требует никакого Silverlight. -->

![F# в браузере](./fsharp_web2.png)

{{< linktarget "interactive-shell" >}}
> ## Using F# in the interactive shell
> ## Использование F# в интерактивной оболочке <!-- Словарь: shell — оболочка. -->

> F# has a simple interactive console called FSI.exe that can also be used to run code in.
F# имеет простую интерактивную консоль, именуемую FSI.exe, которая также может быть использована для запуска кода.
> Just as with the interactive window in Visual Studio, you must terminate a block of code with double semicolons.
Точно также, как и в интерактивном окне Visual Studio, вы должны завершать каждый блок кода двумя точками с запятой.

![FSI](./fsharp_fsi2.png)

{{< linktarget "sharp-develop" >}}
> ## Using F# in SharpDevelop
> ## Использование F# в SharpDevelop

> [SharpDevelop](http://www.icsharpcode.net/OpenSource/SD/) has some support for F#.
[SharpDevelop](http://www.icsharpcode.net/OpenSource/SD/) имеет некоторую поддержку F#.
> You can create an F# project, and then within that, create an FSX script file.
Вы можете создать проект на F#, и уже в нём создать файл сценария FSX.
> Then type in some code in the script window and use the context menu to send the code to the interactive window (as shown below).
После этого введите код в окне скрипта и используйте контекстное меню, чтобы отправить код в интерактивное окно (как показано ниже).

![Отправить в интерактивное окно](./fsharp_eval_sharpdevelop2.png)

{{< linktarget "compilation-errors" >}}
> ## Compilation Errors? ##
> ## Ошибки компиляции? ##

> If you have problems getting your own code to compile, the ["troubleshooting F#"](/troubleshooting-fsharp/) page might be helpful.
Если при компиляции вашего кода возникли проблемы, страница ["Исправление ошибок F#"](/troubleshooting-fsharp/) может оказаться полезной.

{{< linktarget "projects-solutions" >}}
> ## Projects and Solutions ##
> ## Проекты и Решения ## <!-- Microsoft Docs предлагает Решение для Solution. Предложить в словарь. -->

> F# uses exactly the same "projects" and "solutions" model that C# does, so if you are familiar with that, you should be able to create an F# executable quite easily.
F# использует точно такую же модель "проектов" и "решений", как и C#, так что если она вам хорошо знакома, вы достаточно легко сможете создать исполняемое приложение на F#.

> To make a file that will be compiled as part of the project, rather than a script file, use the `.fs` extension. `.fsx` files will not be compiled.
Чтобы создать файл, который будет компилироваться, как часть проекта, вместо файлов скрипта используйте файлы с расширенем `.fs` <!-- в оригинале: "вместо файлов используйте расширения `.fs`, что не совсем точно -->. Файлы `.fsx` компилироваться не будут.

> An F# project does have some major differences from C# though:
Тем не менее, проекты F# имеют несколько важных отличий от C#:

> * The F# files are organized *linearly*, not in a hierarchy of folders and subfolders.
>   In fact, there is no "add new folder" option in an F# project!
>   This is not generally a problem, because, unlike C#, an F# file contains more than one class.
>   What might be a whole folder of classes in C# might easily be a single file in F#.
> * The *order of the files in the project is very important*: a "later" F# file can use the public types defined in an "earlier" F# file, but not the other way around.
>   Consequently, you cannot have any circular dependencies between files.
> * You can change the order of the files by right-clicking and doing "Move Up" or "Move Down".
>   Similarly, when creating a new file, you can choose to "Add Above" or "Add Below" an existing file.
* Файлы F# организованы *линейно*, а не в иерархию папок и подпапок. По файту, в проектах на F#
  нет опции "создать папку"! В целом, это не проблема, поскольку, в отличие от C#, файлы в F# могут содержать больше одного класса. То, что может быть целой папкой с классами в C#, в F# может быть одним файлом.
* *Порядок файлов в проекте очень важен*: более "поздние" файлы F# могут использовать
  <!-- обращаться к --> публичные типы, определённые в более "ранних" файлах F#, но не наоборот.
  Как следствие, вы не можете иметь никаких циклических зависимостей между файлами.
* Вы можете изменять порядок файлов по правой кнопке мыши, выполняя "Вверх" ("Move Up") или "Вниз"
  ("Move Down").
  Точно также, создавая новый файл, вы можете выбрать "Добавить выше" ("Add Above") или "Добавить ниже" ("Add Below") существующего файла. <!-- Как переводить пункты меню? У меня нет русской студии! -->

{{< linktarget "shell-scripts" >}}
> ## Shell scripts in F# ##
> ## Сценарии оболочки на F# ##

> You can also use F# as a scripting language, rather than having to compile code into an EXE.
Вы также можете использовать F#, как язык для написания скриптов, вместо того, чтобы компилировать код в EXE <!-- в исполняемый файл -->.
> This is done by using the FSI program, which is not only a console but can also be used to run scripts in the same way that you might use Python or Powershell.
Это делается с помощью программы FSI, которая не только работает в режиме консоли, но также может быть использована для запуска скриптов тем же способом, как вы можете использовать Python или Powershell.

> This is very convenient when you want to quickly create some code without compiling it into a full blown application.
Это очень удобно когда вы хотите быстро запустить какой-то код без компиляции в полноценное приложение.
> The F# build automation system ["FAKE"](https://github.com/fsharp/FAKE) is an example of how useful this can be.
Система автоматизации сборки F# ["FAKE"](https://github.com/fsharp/FAKE) показывает, насколько это может быть полезным.

> To see how you can do this yourself, here is a little example script that downloads a web page to a local file.
Чтобы увидеть, как вы можете сделать это сами, я написал для вас примерчик скрипта, который скачивает <!-- сохраняет --> веб-страницу в локальный файл.
> First create an FSX script file -- call it "`ShellScriptExample.fsx`" -- and paste in the following code.
Сначала создайте файл скрипта FSX — назовём его "`ShellScriptExample.fsx`" — и скопируйте туда этот код.

```fsharp
// ================================
// Описание:
//    скачивает страницу по указанному адресу и сохраняет её в файл,
//    добавляя к его имени дату и время скачивания
//
// Пример вызова:
//    fsi ShellScriptExample.fsx http://google.com google
// ================================

// инструкция "open" добавляет в область видимости указанное пространство имён
open System.Net
open System

// скачать содержимое страницы
let downloadUriToFile url targetfile =
    let req = WebRequest.Create(Uri(url))
    use resp = req.GetResponse()
    use stream = resp.GetResponseStream()
    use reader = new IO.StreamReader(stream)
    let timestamp = DateTime.UtcNow.ToString("yyyy-MM-ddTHH-mm")
    let path = sprintf "%s.%s.html" targetfile timestamp
    use writer = new IO.StreamWriter(path)
    writer.Write(reader.ReadToEnd())
    printfn "finished downloading %s to %s" url path

// При запуске из FSI сначала идёт имя сценария, а затем уже другие аргументы командной строки
match fsi.CommandLineArgs with
    | [| scriptName; url; targetfile |] ->
        printfn "running script: %s" scriptName
        downloadUriToFile url targetfile
    | _ ->
        printfn "USAGE: [url] [targetfile]"
```

> Don't worry about how the code works right now.
Не беспокойтесь прямо сейчас о том, как работает этот код.
> It's pretty crude anyway, and a better example would add error handling, and so on.
В любом случае, он достаточно сырой, и в лучшем примере следовало бы добавить обработку ошибок и подобные вещи.

> To run this script, open a command window in the same directory and type:
Чтобы запустить этот скрипт, откройте командную строку <!-- в оригинале: командное окно --> в том же каталоге, и наберите:

```
fsi ShellScriptExample.fsx http://google.com google_homepage
```

> As you play with the code on this site, you might want to experiment with creating some scripts at the same time.
Пока вы играете с кодом на этом сайте, вы можете заодно поэкспериментировать с созданием скриптов
так точнее будет.
<!-- Дословно: "Пока вы играете с кодом на этом сайте, вы можете в то же время захотеть поэкспериментировать, создавая свои сценарии." -->